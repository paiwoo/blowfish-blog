---
title: "VS Code 优化清单"
draft: false
tags: ["VS code"]
---
## 配置
```
// ── 字体与排版 ─────────────────────────
"editor.fontSize": 16,                 // 主编辑区字号，大屏可 18

// ── 主题 & 图标 ───────────────────────
iconTheme - Material Icon Theme

// ── 编辑器行为 ───────────────────────
"editor.formatOnSave": true,

// ── 其他贴心设置 ─────────────────────
"files.autoSave": "onFocusChange",


```

## 插件
| 插件                       | 功能            |
| ------------------------ | ------------- |
| Dracula Theme Official - Dracula Theme             | 主题            |
| Material Icon Theme      | 彩色文件图标        |
| Bracket Pair Colorizer 2 | 彩虹括号          |
| Error Lens               | 把报错直接渲染到行内    |


## 字体放大
仅放大代码区
Ctrl + 滚轮 即可临时放大；永久生效请改 editor.fontSize。
整体 UI 放大
Ctrl + = / Ctrl + - 调整 window.zoomLevel，或菜单 View → Appearance → Zoom。

## 同步配置
登录github账号