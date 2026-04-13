---
title: 网站更新与发布流程
cover: /images/Covers/flowers.jpg
date: 2026-04-13 21:00:00
sticky: 100
tags:
  - Hexo
  - GitHub Pages
  - 网站维护
categories:
  - 学习笔记
mathjax: false
---

**金培晟** *Jarfield*

这篇是给我自己留的站点维护说明。  
目标不是解释 Hexo 原理，而是把“写完内容以后，怎样稳定地同步到网站”固定成一套最短流程。

<!-- more -->

## 1. 当前站点的发布方式

这个仓库现在不是“手动上传 `public/` 目录”的模式，而是：

1. 本地维护 Hexo 源码
2. 推送到 GitHub 仓库
3. GitHub Actions 自动构建
4. GitHub Pages 自动发布

也就是说，真正需要维护的是源码，而不是生成产物。

## 2. 最常用的更新流程

### 2.1 本地修改内容

常见修改位置有：

- 新文章：`source/_posts/`
- 页面：`source/overview/`、`source/topics/`、`source/resources/`
- 样式：`source/css/custom.css`
- 配置：`_config.yml`、`_config.icarus.yml`

### 2.2 本地预览

```bash
npm run server
```

作用：

- 本地启动 Hexo 服务
- 浏览器里检查排版、链接、首页列表和置顶顺序

如果只是改了一篇文章，这一步通常已经足够发现大多数问题。

### 2.3 本地构建检查

```bash
npm run build
```

这一步是为了确认：

- front matter 没写坏
- Markdown 没有触发构建错误
- 主题补丁和样式还能正常工作

如果 build 都没过，就不要往 GitHub 推。

## 3. 正式同步到网站

### 3.1 查看当前改动

```bash
git status
```

先看清楚这次到底改了哪些文件。

### 3.2 提交改动

```bash
git add .
git commit -m "Update site content"
```

如果你想更稳一点，也可以只加相关文件，而不是直接 `git add .`。

例如：

```bash
git add source/_posts/Notes/new-note.md
git commit -m "Add new note"
```

### 3.3 推送到 GitHub

```bash
git push
```

推送以后，不需要再手动跑 `hexo deploy`。

## 4. GitHub 上发生了什么

仓库里已经有工作流文件：

```text
.github/workflows/pages.yml
```

当 `main` 分支有新提交时，GitHub 会自动：

1. 安装依赖
2. 运行 `npm ci`
3. 执行 `npm run build`
4. 上传 `public/`
5. 发布到 GitHub Pages

## 5. 如何确认网站已经更新成功

### 5.1 看 Actions

去 GitHub 仓库的 `Actions` 页面，看 `Deploy Hexo To Pages` 是否成功。

### 5.2 看网站

打开：

```text
https://Jarfield.github.io/
```

检查：

- 首页是否更新
- `/notes/` 里新文章是否出现
- 置顶是否正确
- 新页面链接是否能点开

如果浏览器缓存比较重，强刷一下页面。

## 6. 我之后最常做的几类更新

### 6.1 新增一篇文章

```bash
npm run build
git add source/_posts/Notes/xxx.md
git commit -m "Add xxx note"
git push
```

### 6.2 修改首页或专题页

```bash
npm run build
git add source/index.md source/topics/index.md
git commit -m "Update site pages"
git push
```

### 6.3 修改样式

```bash
npm run build
git add source/css/custom.css
git commit -m "Refine site styles"
git push
```

## 7. 关于置顶文章

现在站内已经支持 `sticky`。

用法：

```yaml
---
title: Example
date: 2026-04-13 12:00:00
sticky: 100
---
```

规则：

- 数值越大，优先级越高
- `/notes/` 列表会优先展示置顶文章
- 右侧 `Recent Updates` 也会优先显示置顶文章

这篇文章本身就是置顶文章，方便以后快速回来看。

## 8. 一份最短速查版

```bash
# 本地预览
npm run server

# 本地构建
npm run build

# 提交并发布
git status
git add .
git commit -m "Update site"
git push
```

## 9. 当前我应该记住的一件事

这个仓库的发布入口已经是 `git push`，不是 `hexo deploy`。  
只要本地 build 正常，推到 `main` 后 GitHub Actions 就会接管后续发布。
