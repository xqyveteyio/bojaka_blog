---
title: "Vue 3 接入高德地图（AMap）"
date: 2025-03-27 10:00:00
tags:
  - "Vue3 高德地图集成"
  - "AMap Vue3 教程"
  - "@amap/amap-jsapi-loader 使用"
  - "Vue3 地图组件"
categories:
  - 开发
feature: false
comments: false
abstracts: "在 Vue 3 项目中集成高德地图 JS API：使用 @amap/amap-jsapi-loader 异步加载地图，初始化 3D 地图实例，并通过高德 REST API 实现医院、诊所等 POI 兴趣点的分页搜索与地图展示。"
---

## 前言

本文记录在 Vue3 项目中集成高德地图（AMAP），并调用搜索地点服务。

## 设置地图与搜索服务

下面示例包含地图初始化与请求“医院/诊所”数据的逻辑。

~~~ts
import axios from "axios";
import { ref, onMounted, onUnmounted } from "vue";
import AMapLoader from "@amap/amap-jsapi-loader";

let map = null;
const loading = ref(false);

const getHospital = async () => {
  loading.value = true;
  let hospitalList = [];
  let clinicList = [];
  let page = 1;

  while (true) {
    const { data: res } = await axios.get(
      `https://restapi.amap.com/v3/place/text?keywords=医院&city=430100&offset=50&page=${page}&key=你的Web服务Key&extensions=all`
    );
    hospitalList = hospitalList.concat(res.pois);
    page++;
    if (res.pois.length < 50) {
      break;
    }
  }

  page = 1;
  while (true) {
    const { data: res } = await axios.get(
      `https://restapi.amap.com/v3/place/text?keywords=诊所&city=430100&offset=50&page=${page}&key=你的Web服务Key&extensions=all`
    );
    clinicList = clinicList.concat(res.pois);
    page++;
    if (res.pois.length < 50) {
      break;
    }
  }

  loading.value = false;
  return [hospitalList, clinicList];
};

const initMap = async () => {
  const AMap = await AMapLoader.load({
    key: "你的Web端Key", // 申请好的 Web 端开发者 Key
    version: "2.0",
    plugins: [],
  });

  map = new AMap.Map("container", {
    viewMode: "3D",
    zoom: 11,
    center: [116.397428, 39.90923],
  });

  // await getHospital();
};

onMounted(async () => {
  await initMap();
});

onUnmounted(() => {
  map?.destroy();
});
~~~

## 页面结构与样式

~~~html
<div id="container"></div>
~~~

~~~css
#container {
  width: 100%;
  height: 100%;
}
~~~


原文示例里写死了两把高德 Key。迁到这边时改成了占位符，请换成自己在高德控制台申请的 Web 端 Key 和 Web 服务 Key，不要把 Key 提交进仓库。


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
