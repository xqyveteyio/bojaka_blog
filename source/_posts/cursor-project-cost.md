---
title: "用 Cursor Hooks 统计每个项目的 AI 花费"
date: 2026-07-12 18:30:00
tags:
  - "Cursor 花费统计"
  - "Cursor Hooks"
  - "Cursor 每个项目花费"
  - "Cursor 费用监控"
categories:
  - 工具
cover: /images/cursor-project-cost/image.png
feature: true
comments: false
abstracts: "基于 Cursor Hooks 机制，记录每一轮对话（含子请求）的真实计费金额，并按项目分组实时展示，搞清楚钱到底花在了哪个项目上。"
---

Cursor 账单只会给一个总数，但同时开着好几个项目用 Agent 写代码时，很想知道到底是哪个项目在"烧钱"。官方后台目前不提供按项目拆分的账单，所以自己写了一个小工具 [cursor-cost-count](https://github.com/xqyveteyio/cursor-cost-count)，用 Cursor 的 Hooks 机制把每一轮对话和它花的钱对应起来，再按项目汇总展示。

## 思路

Cursor 支持在 `hooks.json` 里配置钩子脚本，在特定事件发生时调用外部命令，并把上下文通过 stdin 传入 JSON payload。用得上的几个事件：

- `beforeSubmitPrompt`：一轮对话开始提交时触发，payload 里带 `conversation_id`、`generation_id`、`model`、`workspace_roots`（也就是项目路径）。
- `stop`：这一轮主对话结束时触发。
- `subagentStart` / `subagentStop`：Agent 内部起的子请求（比如并行搜索）开始/结束时触发。

单靠这些事件拿不到金额，因为 Hooks 本身不包含计费信息。但 Cursor 网页后台有一个用于渲染账单明细的接口：

```
POST https://cursor.com/api/dashboard/get-filtered-usage-events
```

传入起止时间戳就能拿到这段时间内每一条 usage event 的 `conversationId`、`model`、`chargedCents`（精确计费，单位为分）。于是整个方案就是：

1. 用 Hooks 记下这一轮的开始时间、结束时间、`conversation_id` 和项目路径。
2. `stop` 触发后，另起一个进程，等账单入账（通常几秒内），拿 `[开始 - 3s, 结束 + 2s]` 这个时间窗口去查 usage events。
3. 把 `conversationId` 等于本轮的事件记为"主请求"；再结合 `subagentStart/Stop` 还原出的时间区间，把落在区间内、且不属于其他已知会话的事件记为"子请求"（子请求的计费 `conversationId` 和主对话不是同一个）。
4. 主请求 + 子请求的费用相加，就是这一轮真实花掉的钱，连同项目名一起写进 `costs.jsonl`。

这样即使一次提问背后触发了好几个并行子任务，费用也都能归并到这一轮、这个项目上。

## 部署步骤

### 1. 克隆项目并安装依赖

```bash
git clone https://github.com/xqyveteyio/cursor-cost-count.git
cd cursor-cost-count
python3 -m venv venv
source venv/bin/activate
pip install fastapi uvicorn
```

### 2. 拿到登录 Cookie

`main.py` 需要一个能访问账单接口的 Session Token。打开浏览器登录 [cursor.com](https://cursor.com/dashboard)，打开开发者工具的 Network 或 Application 面板，找到 Cookie 里的 `WorkosCursorSessionToken`，复制它的值，通过环境变量传入（不要写进代码或提交到仓库）：

```bash
export CURSOR_SESSION_TOKEN="你的 token"
```

### 3. 配置 Hooks

在项目或全局的 `~/.cursor/hooks.json` 里把四个事件都指向同一个脚本：

```json
{
  "version": 1,
  "hooks": {
    "beforeSubmitPrompt": [
      { "command": "python3 /path/to/cursor-cost-count/main.py" }
    ],
    "stop": [
      { "command": "python3 /path/to/cursor-cost-count/main.py" }
    ],
    "subagentStart": [
      { "command": "python3 /path/to/cursor-cost-count/main.py" }
    ],
    "subagentStop": [
      { "command": "python3 /path/to/cursor-cost-count/main.py" }
    ]
  }
}
```

`main.py` 会根据 `hook_event_name` 自动分发处理逻辑，不需要为每个事件单独写脚本。之后正常用 Cursor 聊天、跑 Agent，每一轮结束都会自动在后台对账，结果追加进 `costs.jsonl`。

### 4. 启动可视化面板

```bash
python3 web_gui.py
```

默认监听 `http://0.0.0.0:8787`，浏览器打开后就是一个实时刷新的仪表盘：顶部是总花费、项目数、总调用次数、今日花费；下面按项目分卡片展示，每张卡片里有该项目的总花费、平均单次花费、按模型拆分的花费占比，以及最近几十条调用的明细表（主请求花费 / 子请求花费 / 总花费 / 子请求数）。`costs.jsonl` 一有新增，页面就会通过 WebSocket 推送更新并高亮对应项目卡片。

![Cursor Cost Dashboard 效果图](/images/cursor-project-cost/image.png)

## 一些细节

- 时间窗口两端各留了几秒缓冲，用来抵消本地时钟和服务端落账的延迟，避免漏算或错算到别的对话。
- 子请求的计费 `conversationId` 和触发它的主对话不是同一个，工具靠"已知的所有主对话 id 集合 + 子请求时间区间 + 模型是否匹配"三个条件筛出真正属于本轮的子请求费用，防止把其他并发窗口的请求混进来。
- 对账逻辑跑在完全独立的子进程里（`start_new_session=True`），不会被 Cursor 对 Hook 脚本的执行超时限制卡死。
- Token 有过期时间，失效后重新从浏览器 Cookie 里取一份替换即可。

这套方案本质上是"用官方前端在用的接口，把散落的账单明细按对话重新拼回去"，不涉及破解或绕过任何限制，纯粹是官方没做按项目统计，自己动手拼一个。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
