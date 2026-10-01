---
title: 小动物好玩吗？Laopu buddy 普瑞塞斯桌宠
description: 一个常驻桌面的普瑞塞斯 Q 版挂件，实时感知 DSH 与 Codex 的工作状态，把 AI 的思考、等待、完成翻译成动画和气泡。
published: 2026-10-01
tags: [桌面宠物, AI, DSH, Codex, 工具]
category: 技能包
image: "/assets/images/bg.jpg"
link:
  - label: "下载 v2-baseline"
    value: "https://github.com/SIDIHUANG/Laopu-body/releases/tag/v2-baseline"
  - label: "查看源码"
    value: "https://github.com/SIDIHUANG/Laopu-body"
---

## 关于这个项目

**Presage Pet** 是一个常驻桌面的普瑞塞斯（明日方舟）Q 版挂件。

它同时做两件事：

- **看钱**：显示 DeepSeek / Codex 的用量与余额。
- **看活**：感知 DSH 与 Codex 谁在思考、谁在等你、谁完成了，并把状态翻译成动画和气泡。

**独立应用，不依赖 DSH 是否打开。**

## v2 版本亮点

- **双击 `presage-pet.exe` 就是全部**，不需要 `.bat`，不需要安装任何依赖。
- 启动逻辑从 `.bat` 搬进 exe：预检、单实例、WebView2 profile 自愈、无窗口拉起桥接、退出收尾、`--diag` 诊断。
- 前端逻辑一行未改（`v2/app/src/*.js` 与 v1.1 逐字节相同）。

## 快速开始

1. 点击上方“下载 v2-baseline”按钮，下载 `.zip` 压缩包。
2. 解压到任意目录（非 ASCII 路径也没问题）。
3. 双击 `presage-pet.exe` 即可运行。

**退出快捷键**：`Ctrl+Alt+Q`
**设置快捷键**：`Ctrl+Alt+S`

## 更多文档

- [README-V2.txt](https://github.com/SIDIHUANG/Laopu-body/blob/main/README-V2.txt) — 怎么用 / 怎么看日志 / 出问题怎么办
- [V2-BASELINE.md](https://github.com/SIDIHUANG/Laopu-body/blob/main/V2-BASELINE.md) — 这一版改了什么 / 怎么重建
- [OPEN-ISSUES-V2.md](https://github.com/SIDIHUANG/Laopu-body/blob/main/OPEN-ISSUES-V2.md) — 已知问题