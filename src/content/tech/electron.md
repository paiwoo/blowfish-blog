---
title: "Electron 快速实践"
weight: 4
draft: false
# categories: ["tech"]
tags: ["Electron"]
# series: ["tech"]
# series_order: 4
---

# 📘 Electron 桌面壳最佳实践指南  
*基于 Electron + electron-builder 的「远程网页」打包桌面壳方案*

---

## 🧩 方案概览

1. **纯网页内核** - 远端部署即更新，无需重新发安装包  
2. **官方推荐结构** - 4 个必要文件 + build/ 资源，零硬编码  
3. **全部配置化** - 运行时与打包参数分别集中到 `config.js` 与 `package.json`  
4. **体验加分** - Lottie 加载页、托盘、无菜单、无滚动条、单文件 exe

---

## 📁 最小目录
官方最小目录
```
project/
├─ main.js              // 入口（读取 config.js）
├─ config.js            // 运行时配置（唯一要改的地方）
├─ preload.js           // 无滚动条注入
├─ splash.html          // 本地加载页（居中 Lottie）
├─ build/
│  ├─ icon.ico          // 256×256 多尺寸图标
│  ├─ loading.json      // Lottie 动画
│  └─ lottie.min.js     // Lottie 运行时（本地引入，避免跨域）
└─ package.json         // 全部打包配置集中在此
```

---


## 🔣 核心代码

main.js
```js
const { app, BrowserWindow, Tray, nativeImage, Menu } = require('electron');
const path = require('path');
const cfg = require('./config');

let win;
let tray;

function createWindow() {
  const winIcon = nativeImage.createFromPath(path.join(__dirname, 'build/icon.ico'));
  win = new BrowserWindow({
    width: cfg.width,
    height: cfg.height,
    autoHideMenuBar: cfg.autoHideMenuBar,
    titleBarStyle: cfg.titleBarStyle,
    icon: winIcon,
    webPreferences: { contextIsolation: true, preload: path.join(__dirname, 'preload.js') }
  });

  win.loadFile(path.join(__dirname, 'splash.html'));
  setTimeout(() => win.loadURL(cfg.remoteUrl), cfg.splashDuration);

  // 托盘
  const trayIcon = nativeImage.createFromPath(path.join(__dirname, 'build/icon.ico'));
  tray = new Tray(trayIcon.isEmpty() ? nativeImage.createEmpty() : trayIcon.resize({ width: 16 }));
  tray.setToolTip(cfg.productName || 'MyApp');
  tray.setContextMenu(Menu.buildFromTemplate([
    { label: '显示窗口', click: () => win.show() },
    { label: '退出', click: () => app.quit() }
  ]));
  tray.on('double-click', () => win.show());
}

app.whenReady().then(createWindow);
// 监听应用即将退出事件
app.on('before-quit', () => {
    // 如果托盘实例存在，销毁托盘图标并释放资源
    if (tray) {
        tray.destroy();
        tray = null;
    }
});

// 监听所有窗口关闭事件
app.on('window-all-closed', () => {
    // 如果托盘实例存在，销毁托盘图标并释放资源
    if (tray) {
        tray.destroy();
        tray = null;
    }
    // 可选：窗口全关时退出应用
    app.quit();
});
```

preload.js
```js
window.addEventListener('DOMContentLoaded', () => {
  document.documentElement.style.overflow = 'hidden';
  document.body.style.overflow = 'hidden';
});
```

splash.html（Lottie 居中）
```html
<!doctype html>
<html>
<head>
  <meta charset="utf-8"/>
  <title>Loading</title>
  <style>
    html,body{margin:0;height:100%;overflow:hidden;background:#fff;}
    .box{position:absolute;inset:0;display:flex;align-items:center;justify-content:center;flex-direction:column;}
    #lottie{width:200px;height:200px;}
  </style>
</head>
<body>
  <div class="box"><div id="lottie"></div></div>
  <script src="build/lottie.min.js"></script>
  <script>
    lottie.loadAnimation({container:document.getElementById('lottie'),renderer:'svg',loop:true,autoplay:true,path:'build/loading.json'});
    setTimeout(()=>location.replace(require('./config').remoteUrl),3000);
  </script>
</body>
</html>
```

---

## ⚙️ 运行时配置
（config.js）
```js
module.exports = {
  width: 1200,
  height: 800,
  autoHideMenuBar: true,      // 隐藏菜单栏
  titleBarStyle: 'default',   // 原生标题栏（最稳）
  splashDuration: 3000,       // 加载页停留 ms
  remoteUrl: 'https://xxx.com/' // 主页面远程地址
};
```

> 改完立即生效（开发 & 打包共用）。

---

## 📦 打包配置
集中到（package.json）
```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My Admin",
  "author": "Your Name",
  "license": "MIT",
  "main": "main.js",
  "scripts": {
    "start": "electron .",
    "dist": "electron-builder"
  },
  "devDependencies": {
    "electron": "^28.3.3",
    "electron-builder": "^24.13.3"
  },
  "build": {
    "appId": "com.example.myapp",
    "productName": "MyApp",
    "copyright": "© 2025 Your Name",
    "directories": { "output": "dist" },
    "files": [ "**/*", "build/**/*" ],
    "win": {
      "target": { "target": "nsis", "arch": ["x64"] },
      "icon": "build/icon.ico",
      "sign": null
    },
    "nsis": {
      "oneClick": false,
      "allowToChangeInstallationDirectory": true,
      "artifactName": "${productName}-Setup-${version}.${ext}"
    }
  }
}
```

---

## 🚀 一键打包

```bash
npm install
npm start        # 开发
npm run dist     # 单文件 exe 输出到 dist/
```

产出：`dist/MyApp-Setup-1.0.0.exe`  
- 窗口图标、托盘图标、Lottie 加载页、无菜单、无滚动条全部生效；
- 网页更新 → 部署远端即可，**无需重新发安装包**。

---

## 📝 标题栏选型

| titleBarStyle | 效果（Win/Linux） | 官方建议场景 |
|---------------|-------------------|--------------|
| `default`     | 原生完整标题栏    | 后台/OA/管理后台（最稳） |
| `hidden`      | 整个栏消失        | 自写全套标题栏 |
| `hiddenInset` | 同 hidden 仅 macOS 内缩 | 仅 mac 美观 |
| `customButtonsOnHover` | 悬停才显按钮 | 实验性播放器 |

> 管理后台继续用 `default` 即可；想全品牌定制再换 `hidden` + `-webkit-app-region: drag`。

---

## 🚀 一键打包

```bash
npm install
npm start        # 开发
npm run dist     # 单文件 exe 输出到 dist/
```

产出：`dist/MyApp-Setup-1.0.0.exe`  
- 窗口图标、托盘图标、加载页、无菜单、无滚动条全部生效；
- 网页更新 → 部署远端即可，**无需重新发安装包**。

---

## 🧪 高级加分
（可选集中配置）

| 功能 | 配置位置 | 开关 |
| ---- | -------- | ---- |
| **托盘常驻** | `main.js` 加 6 行（图标 `build/icon.ico` resize 16） |
| **Lottie 加载页** | `splash.html` + `build/loading.json`（已居中） |
| **数字签名** | 购买证书后 `build.win.sign: true` + 证书路径 |
| **自动更新** | 后续加 `electron-updater` 即可 |

---

## 📑 维护清单

1. 改功能 → 只改 `config.js`；
2. 改版本/版权/产品名 → 只改 `package.json` 的 `version` / `build` 块；
3. 换图标 → 替换 `build/icon.ico`（256 多尺寸）；
4. 换加载动画 → 替换 `build/loading.json`。

---
