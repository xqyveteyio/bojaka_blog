---
title: "单应性标定：从原理到实现"
date: 2025-12-18 01:00:00
tags:
  - "单应性标定 Homography"
  - "OpenCV 透视变换"
  - "Python 单应性矩阵"
  - "findHomography 使用"
categories:
  - 笔记
feature: false
comments: false
abstracts: "深入讲解计算机视觉中的单应性（Homography）概念：从 3×3 矩阵的数学推导，到用 OpenCV + Python 实现 findHomography 标定与 warpPerspective 透视变换，再到基于 SIFT/ORB 特征点自动匹配的进阶方案，涵盖图像拼接、文档校正等实际应用场景。"
---

## 什么是单应性

单应性（Homography）是计算机视觉中的核心概念，描述了**同一平面**在两个不同视角下的坐标变换关系。简单理解：当你拍摄一张纸，然后换个角度再拍一张，这两张图像之间的像素对应关系就是单应性。

## 核心原理

### 数学表达

单应性用3×3矩阵H表示：

$$
\mathbf{x'} = H \mathbf{x}
$$

其中：
$$
\mathbf{x'} = \begin{bmatrix} x' \\ y' \\ 1 \end{bmatrix}, \quad
H = \begin{bmatrix} 
h_{11} & h_{12} & h_{13} \\ 
h_{21} & h_{22} & h_{23} \\ 
h_{31} & h_{32} & h_{33} 
\end{bmatrix}, \quad
\mathbf{x} = \begin{bmatrix} x \\ y \\ 1 \end{bmatrix}
$$

归一化后得到：
$$
x' = \frac{h_{11}x + h_{12}y + h_{13}}{h_{31}x + h_{32}y + h_{33}}, \quad
y' = \frac{h_{21}x + h_{22}y + h_{23}}{h_{31}x + h_{32}y + h_{33}}
$$

其中$(x,y)$是原图像坐标，$(x',y')$是目标图像坐标。

### 标定原理

1. **收集对应点对**：至少需要4对匹配点
2. **构建线性方程组**：每对点贡献2个方程
3. **求解矩阵H**：使用最小二乘法或SVD分解

## Python实现

### 基础实现

```python
import cv2
import numpy as np
import matplotlib.pyplot as plt

def homography_calibration(src_points, dst_points):
    """
    单应性标定
    
    Args:
        src_points: 源图像中的点 [[x1,y1], [x2,y2], ...]
        dst_points: 目标图像中的对应点
    
    Returns:
        H: 3x3单应性矩阵
    """
    # 转换为numpy数组
    src = np.float32(src_points)
    dst = np.float32(dst_points)
    
    # 计算单应性矩阵
    H = cv2.findHomography(src, dst, cv2.RANSAC)[0]
    
    return H

def apply_homography(image, H, output_size):
    """
    应用单应性变换
    
    Args:
        image: 输入图像
        H: 单应性矩阵
        output_size: 输出图像大小 (width, height)
    
    Returns:
        warped_image: 变换后的图像
    """
    return cv2.warpPerspective(image, H, output_size)

# 示例使用
if __name__ == "__main__":
    # 创建示例数据
    # 假设我们有一个矩形在两个视角下的对应点
    src_points = [[0, 0], [300, 0], [300, 200], [0, 200]]    # 原始矩形
    dst_points = [[50, 30], [250, 20], [280, 180], [20, 190]] # 透视变形后的四边形
    
    # 计算单应性矩阵
    H = homography_calibration(src_points, dst_points)
    print("单应性矩阵:")
    print(H)
    
    # 创建测试图像
    test_image = np.zeros((200, 300, 3), dtype=np.uint8)
    cv2.rectangle(test_image, (50, 50), (250, 150), (0, 255, 0), 3)
    cv2.putText(test_image, 'Original', (100, 100), cv2.FONT_HERSHEY_SIMPLEX, 1, (255, 255, 255), 2)
    
    # 应用变换
    warped = apply_homography(test_image, H, (300, 200))
    
    # 显示结果
    plt.figure(figsize=(12, 4))
    plt.subplot(1, 2, 1)
    plt.imshow(cv2.cvtColor(test_image, cv2.COLOR_BGR2RGB))
    plt.title('原始图像')
    plt.axis('off')
    
    plt.subplot(1, 2, 2)
    plt.imshow(cv2.cvtColor(warped, cv2.COLOR_BGR2RGB))
    plt.title('单应性变换后')
    plt.axis('off')
    
    plt.tight_layout()
    plt.show()
```

### 进阶：特征点匹配实现

```python
def feature_based_homography(img1, img2, feature_type='SIFT'):
    """
    基于特征点匹配的单应性标定
    
    Args:
        img1, img2: 输入图像
        feature_type: 特征检测器类型
    
    Returns:
        H: 单应性矩阵
        matches: 匹配点对
    """
    # 创建特征检测器
    if feature_type == 'SIFT':
        detector = cv2.SIFT_create()
    elif feature_type == 'ORB':
        detector = cv2.ORB_create()
    
    # 检测关键点和描述符
    kp1, des1 = detector.detectAndCompute(img1, None)
    kp2, des2 = detector.detectAndCompute(img2, None)
    
    # 特征匹配
    if feature_type == 'SIFT':
        matcher = cv2.BFMatcher()
        matches = matcher.knnMatch(des1, des2, k=2)
        # 筛选好的匹配
        good_matches = []
        for m, n in matches:
            if m.distance < 0.7 * n.distance:
                good_matches.append(m)
    else:
        matcher = cv2.BFMatcher(cv2.NORM_HAMMING, crossCheck=True)
        good_matches = matcher.match(des1, des2)
        good_matches = sorted(good_matches, key=lambda x: x.distance)
    
    # 提取匹配点坐标
    if len(good_matches) >= 4:
        src_pts = np.float32([kp1[m.queryIdx].pt for m in good_matches]).reshape(-1, 1, 2)
        dst_pts = np.float32([kp2[m.trainIdx].pt for m in good_matches]).reshape(-1, 1, 2)
        
        # 计算单应性矩阵
        H, mask = cv2.findHomography(src_pts, dst_pts, cv2.RANSAC, 5.0)
        return H, good_matches
    else:
        print("匹配点数量不足!")
        return None, None

# 使用示例
def demo_feature_matching():
    # 加载两张图像（需要替换为实际图像路径）
    # img1 = cv2.imread('image1.jpg', cv2.IMREAD_GRAYSCALE)
    # img2 = cv2.imread('image2.jpg', cv2.IMREAD_GRAYSCALE)
    
    # 创建模拟图像进行演示
    img1 = np.zeros((400, 400), dtype=np.uint8)
    cv2.rectangle(img1, (100, 100), (300, 300), 255, 2)
    cv2.circle(img1, (200, 200), 50, 128, -1)
    
    # 创建变换后的图像
    H_demo = np.array([[1.2, 0.3, -50], [-0.1, 1.1, 30], [0.001, 0.002, 1]])
    img2 = cv2.warpPerspective(img1, H_demo, (400, 400))
    
    # 添加一些噪声
    noise = np.random.randint(0, 50, img2.shape, dtype=np.uint8)
    img2 = cv2.add(img2, noise)
    
    # 计算单应性
    H, matches = feature_based_homography(img1, img2, 'ORB')
    
    if H is not None:
        print("检测到的单应性矩阵:")
        print(H)
        print(f"匹配点数量: {len(matches)}")
        
        # 验证变换效果
        warped = cv2.warpPerspective(img1, H, (400, 400))
        
        plt.figure(figsize=(15, 5))
        plt.subplot(1, 3, 1)
        plt.imshow(img1, cmap='gray')
        plt.title('原始图像')
        plt.axis('off')
        
        plt.subplot(1, 3, 2)
        plt.imshow(img2, cmap='gray')
        plt.title('目标图像')
        plt.axis('off')
        
        plt.subplot(1, 3, 3)
        plt.imshow(warped, cmap='gray')
        plt.title('单应性变换结果')
        plt.axis('off')
        
        plt.tight_layout()
        plt.show()

# 运行演示
if __name__ == "__main__":
    demo_feature_matching()
```

## 实际应用场景

1. **图像拼接**：全景照片制作
2. **文档校正**：倾斜文档的几何校正
3. **增强现实**：虚拟物体的透视匹配
4. **相机标定**：内参和外参估计
5. **平面跟踪**：广告牌、书页等平面目标跟踪

## 关键要点

- **平面假设**：只适用于平面场景
- **点数要求**：至少4对对应点
- **鲁棒性**：使用RANSAC处理异常值
- **数值稳定性**：注意矩阵条件数和归一化

单应性标定看似复杂，但本质就是找到两个平面之间的投影变换关系。掌握了基本原理和实现，你就能在各种计算机视觉任务中灵活运用这个强大的工具！


---

> 编辑说明：本文在原文基础上经过 AI 编辑优化。
