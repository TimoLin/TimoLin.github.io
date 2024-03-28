---
title: Paraview中垂直放置文本的一种方法
description: Paraview垂直放置文本
date: 2024-03-27
image: cover.png
categories:
  - Linux
  - CFD
tags:
  - Linux
  - OpenFOAM
  - Paraview
weight: 1
---

## 需求
对于二维云图，Paraview中默认`Y`轴的标题为横向文字（下图左），并且不支持旋转文字的方向，相关讨论可以在这些帖子中看到。
- [Paraview Allow rotation of axes grid title](https://discourse.paraview.org/t/allow-rotation-of-axes-grid-title/9324)
- [Paraview Rotate the Text filter](https://discourse.paraview.org/t/rotate-the-text-filter/4528)

但是有时为了图片更加美观，我们希望`Y`轴的title是垂直放置的，如下图（右）所示。 

![示例云图效果](step0.jpg)

这里有个教程：

{{< youtube id="ZUt0I9XoW2Y" title="Paraview Vertical Text in 2D slice">}}


