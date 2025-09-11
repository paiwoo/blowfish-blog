---
title: "🚀 unibest 极速起步 · 微信小程序"
tags: [tech, Java]
draft: false
weight: 9
---

# 🚀 unibest 极速起步 · 微信小程序（wot-ui 版）

> 5 分钟跑通开发 → 预览 → 热更全流程；基于目前 GitHub 最活跃的 uni-app 模板之一，集成 Vue3 + TS + Vite5 + UnoCSS + wot-ui，为「一套代码、多端发布」提供现代化解决方案。

---

## 1️⃣ 为什么选择 unibest

| 维度     | 传统 HBuilderX 模式 | unibest 方案                                                |
| -------- | ------------------- | ----------------------------------------------------------- |
| 技术栈   | Vue2 + webpack      | **Vue3 + Vite5 + TypeScript**                               |
| 构建速度 | 分钟级              | **秒级热重载 & 秒级构建**                                   |
| 样式方案 | 手写或 scoped       | **UnoCSS 原子化 + 按需生成**                                |
| UI 组件  | 手动按需引入        | **wot-ui 自动导入，70+ 组件开箱即用**                       |
| 代码质量 | 自行集成规范        | **ESLint + Prettier + Stylelint + husky + commitlint 内置** |
| 多端覆盖 | 需多次配置          | **微信小程序 / H5 / App / 支付宝 / 抖音 / 钉钉 一键发行**   |

此外，unibest 近半年 GitHub 提交频率 **> 300 commits/month**，Issue 响应 < 24h，社区活跃度高，持续迭代中。

---

## 2️⃣ 环境准备

| 工具           | 最低版本 | 推荐获取                                                                        |
| -------------- | -------- | ------------------------------------------------------------------------------- |
| Node.js        | 20.18.0  | `nvm use 20.18.0`                                                               |
| pnpm           | 最新     | `npm i -g pnpm`                                                                 |
| 微信开发者工具 | 最新     | [下载](https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html) |

---

## 3️⃣ 创建项目（base 模板，已内置 wot-ui）

```bash
pnpm create unibest
# 交互选项：选择 base 模板，回车即可
```

---

## 4️⃣ 安装依赖

```bash
cd your-project-name
pnpm i
```

---

## 5️⃣ 启动微信小程序开发模式

```bash
pnpm dev:mp
```

- 监听 `src/` 目录，实时生成 `dist/build/mp-weixin`
- 支持 **秒级热重载**，修改即生效

---

## 6️⃣ 导入微信开发者工具

1. 打开微信开发者工具 → 导入项目
2. 目录选择 `dist/build/mp-weixin`
3. AppID 可填测试号 → 完成
4. 真机扫码或模拟器即时刷新

---

## 7️⃣ 目录速览（已约定式路由）

```
src/
├─ pages/          // 页面文件即路由，无需 pages.json
├─ components/     // 自动全局注册
├─ layouts/        // 类 Nuxt 布局，多页复用
├─ stores/         // Pinia + 持久化
├─ utils/          // 请求封装、拦截器已内置
├─ locales/        // i18n 多语言
└─ app.ts          // 入口
```

---

## 8️⃣ 示例：使用 wot-ui 组件

无需手动引入，模板中直接书写：

```vue
<template>
  <wd-button type="primary" @click="onClick">Hello unibest</wd-button>
</template>

<script setup lang="ts">
const onClick = () => console.log("clicked");
</script>
```

---

## 9️⃣ 构建上线

```bash
# 微信小程序生产包
pnpm build:mp

# 其他平台
pnpm build:h5     # H5
pnpm build:app    # App
```

产物位于 `dist/build/` 对应目录，可直接上传微信公众平台或发行市场。

---

## 🔗 更多资源

- [官方文档](https://unibest.tech)
- [GitHub 仓库](https://github.com/unibest-tech/unibest)
- [wot-ui 组件库](https://wot-design-uni.netlify.app/)

---
