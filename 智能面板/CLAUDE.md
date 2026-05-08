# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

摩根智能家居（MOORGEN）演示 UI，单文件实现（`moorgen-home.html`）。无构建系统，无需安装依赖，直接用浏览器打开即可运行。

## 开发方式

**运行：** 用浏览器打开 `moorgen-home.html`（双击或 `open moorgen-home.html`）。

无构建、无 lint、无测试命令，所有代码都在这一个文件里。

## 架构说明

整个应用是一个自包含的 HTML 文件，通过 CDN（unpkg）加载 React 18 和 Babel standalone 进行 JSX 转译，字体使用 Google Fonts。无打包工具、无 npm、无 TypeScript。

**状态管理：** 顶部的 `initialState` 对象通过单个 `useState` 持有全部应用状态（房间、音乐、安防、场景、天气、时间、通知）。所有更新通过 `setState` 或辅助函数 `updateRoom` 完成，无 context、无 reducer。

**页面路由：** 用 `currentTab`（`'overview' | 'room' | 'security'`）加 `currentRoom`（房间 ID 字符串）决定内容区渲染什么，无路由库。

**组件层级：**
- `App` — 顶层，持有全部状态，渲染锁屏或主界面
  - `OverviewDashboard` — 统计卡片、场景选择器、各房间概览
    - `SceneSelector` — 6 个预设场景，批量更新各房间灯光和温度
    - `Sparkline` — 内联 SVG 迷你折线图
    - `AnimatedNumber` — 带缓动的数字动画
  - `RoomDetail` — 单房间控制；各房间专属功能通过 `extraFeatures` 对象映射渲染
    - `Knob` — SVG 拖拽旋钮，控制亮度和温度
    - `CurtainControl` — 窗帘可视化 + 滑块
  - `SecurityPanel` — 布防开关、警报横幅、摄像头网格
  - `MusicPlayer` — 底部固定播放栏，进度通过 `setInterval` 自动推进

**视觉特效（均在文件内实现）：**
- `initParticles()` — canvas 粒子连线网络，组件挂载时启动，卸载时清理
- `addRipple()` — 点击时向 DOM 注入涟漪 span，挂在带 `.ripple-wrap` 类的元素 `onClick` 上
- CSS 动画：shimmer（金色文字流光）、wave-dance（音乐波形柱）、card-glow（激活卡片呼吸光）、scan（摄像头扫描线）、float-up（锁屏粒子上浮）

**锁屏：** PIN 码 `1234` 硬编码。输错后触发 600ms 红色闪烁，然后重置。

**场景系统：** `scenes` 数组定义各房间灯光亮度和目标温度。`applyScene` 批量更新所有房间并将 `state.scene` 设为对应场景 ID 以高亮显示。

## 重要约束

- **无模块系统** — 所有代码在同一个 `<script type="text/babel">` 块内，新组件必须在使用前定义（顺序很重要）。
- **全面使用内联样式** — 几乎所有样式都用 React 内联 style 对象，CSS 类只用于动画、过渡和无法内联的伪元素（`:hover::before`、`::after`）。
- **CDN 依赖** — React、ReactDOM、Babel 均从 `unpkg.com` 加载，首次打开需要联网（之后可走浏览器缓存）。
