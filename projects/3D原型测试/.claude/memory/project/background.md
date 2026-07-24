---
name: 3d-prototype-background
description: 3D原型测试项目背景——探索 Three.js 在 HTML 原型中的集成
metadata:
  type: project
---

# 项目背景：3D原型测试

## 为什么启动

当前 PM 工作空间的所有原型均为 2D 平面页面（表单、表格、图表）。对于涉及物理空间、数据可视化、全球视角的产品概念，2D 原型传达力有限。

**目标**: 验证 Three.js 集成到 HTML 原型的可行性，确定技术边界和应用场景。

## 探索范围

### 已验证
- Three.js r128 (unpkg CDN) 在 HTML 原型中的集成
- 自定义 ShaderMaterial（日/夜纹理混合、海洋镜面高光、大气散射辉光）
- 程序化贝塞尔曲线 + TubeGeometry 飞线效果
- 流动粒子动画
- 纹理预加载（head preload + LoadingManager 进度条）
- OrbitControls 交互（旋转/缩放/自动旋转）

### 待探索
- GLTF/GLB 外部模型加载
- 城市名称标注（CSS2DRenderer 或 sprite）
- WebP 压缩纹理减少加载时间
- 移动端性能适配

## 成功标准

- 单 HTML 文件直接浏览器打开，无构建步骤 ✓
- 3D 场景与 HTML UI 共存 ✓
- 拖拽旋转 + 滚轮缩放 ✓
- 视觉效果达到展示级水准

**Why:** 2D 原型无法有效传达空间/全球视角的产品概念，需要在原型工具链中验证 3D 能力。
**How to apply:** 产出原型时使用 ShaderMaterial 定制特效，TubeGeometry 制作线条，保持单文件自包含。
