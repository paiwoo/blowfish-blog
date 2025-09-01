---
title: "Hugo & Blowfish 博客搭建"
weight: 1
draft: false
description: "从零到线上 HTTPS 个人知识库博客。"
# categories: ["tech"]
tags: ["Blog","Hugo","Blowfish"]
# series: ["tech"]
series_order: 4
---
## 1 目标
- 仓库：`blowfish-blog`
- 地址：`https://<user>.github.io/blowfish-blog/`
- 效果：与 [官方 demo](https://blowfish.page/zh-cn/) 一致
- 主题：Blowfish v2（Hugo Module）
- CI：GitHub Actions 自动部署



## 2 环境准备
| 工具 | 下载 & 验证 |
|---|---|
| Git | [git-scm.com](https://git-scm.com/download/win) |
| Hugo extended | [v0.139.3](https://github.com/gohugoio/hugo/releases) |
| Go | [golang.org/dl](https://golang.org/dl/) |

```powershell
hugo version   # 需显示 extended
go version     # 任意目录可用
```

## 3 快速初始化

（D:\blowfish-blog）
```powershell
mkdir D:\blowfish-blog && cd D:\blowfish-blog
git clone --depth=1 https://github.com/nunocoracao/blowfish.git temp
xcopy /E /Y temp\exampleSite\* .
rmdir /S /Q temp
rename exampleSite src
cd src
hugo mod init github.com/<user>/blowfish-blog
hugo mod tidy
```

## 4 基础配置

（src/config/_default）

config.toml
```toml
baseURL = "https://<user>.github.io/blowfish-blog/"
languageCode = "zh-cn"
title = "我的知识库"
```
module.toml
```toml
[[imports]]
path = "github.com/nunocoracao/blowfish/v2"
```

## 5 目录 & 菜单最佳实践
### 5.1 目录树
```
content/
├─ life/daily/_index.md
├─ life/todo/_index.md
├─ life/shopping/_index.md
├─ tech/ai/_index.md
├─ tech/code/_index.md
├─ tech/bug/_index.md
├─ tech/project/_index.md
└─ about.md
```
### 5.2 菜单

（src/config/_default/menus.toml）
```toml
[[main]]
  name = "日常"
  pageRef = "life/daily"
  weight = 10
[[main]]
  name = "便签"
  pageRef = "life/todo"
  weight = 15
[[main]]
  name = "购物"
  pageRef = "life/shopping"
  weight = 20
[[main]]
  name = "AI 踩坑"
  pageRef = "tech/ai"
  weight = 30
[[main]]
  name = "代码片段"
  pageRef = "tech/code"
  weight = 35
[[main]]
  name = "BUG 记录"
  pageRef = "tech/bug"
  weight = 40
[[main]]
  name = "项目归档"
  pageRef = "tech/project"
  weight = 50
[[main]]
  name = "关于"
  pageRef = "about"
  weight = 90
```

快速创建：
```powershell
hugo new life/daily/_index.md
hugo new life/todo/_index.md
hugo new life/shopping/_index.md
hugo new tech/ai/_index.md
hugo new tech/code/_index.md
hugo new tech/bug/_index.md
hugo new tech/project/_index.md
hugo new about.md
```

## 6 本地预览
```powershell
hugo server -D
# http://localhost:1313
```

## 7 GitHub 仓库 & Actions
### 7.1 新建仓库
名称：blowfish-blog
Public
不勾选 README

### 7.2 推送源码
```powershell
cd D:\blowfish-blog
git init
git add .
git commit -m "feat: init blowfish blog"
git branch -M main
git remote add origin https://github.com/<user>/blowfish-blog.git
git push -u origin main
```

### 7.3 GitHub Actions

（.github/workflows/gh-pages.yml）
```yaml  
name: Deploy Hugo to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: write
  pages: write
  id-token: write
concurrency:
  group: pages
  cancel-in-progress: false
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: recursive
          fetch-depth: 0
      - uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      - name: Build
        working-directory: ./src
        run: hugo --gc --minify --baseURL "https://${{ github.repository_owner }}.github.io/blowfish-blog/"
      - uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./src/public
```
### 7.4 开启 Pages
仓库 → Settings → Pages → Source → gh-pages → Save
访问：https://<user>.github.io/blowfish-blog/

## 8 后续写作（任意电脑）
```powershell
git pull origin main
hugo new tech/bug/2025-09-03-xxx.md
# 编辑后
git add .
git commit -m "post: xxx"
git push
```

## 9 更新主题
```powershell
cd src
hugo mod get github.com/nunocoracao/blowfish/v2@latest
hugo mod tidy
git add ../src/go.mod ../src/go.sum
git commit -m "chore: bump blowfish"
git push
```

## 10 TOML 注释

在 VS Code 里，用最少的操作一次性给 TOML 文件的多行内容加/去注释。

推荐做法：快捷键 Toggle Line Comment
| 系统       | 快捷键       |
|------------|--------------|
| Windows / Linux | `Ctrl + /` |
| macOS      | `Cmd + /` |

步骤  
1. **安装 [Even Better TOML](https://marketplace.visualstudio.com/items?itemName=tamasfe.even-better-toml ) 插件（✅ 已安装）**   提升语法高亮与格式化体验。
2. **确认右下角语言模式为 TOML（✅ 已显示）**  
3. 选中需要注释的行。  
4. 按一次快捷键 → 每行前自动加 `#`；再按一次 → 取消注释。

结论   
TOML 本身仅支持 `#` 单行注释。在 VS Code 里，**使用 Toggle Line Comment 快捷键** 是目前最快速、通用的“多行注释”方案；如遇失效，按上方排查即可。

---