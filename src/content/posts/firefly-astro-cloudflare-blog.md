---
title: 从零搭建 Firefly Astro 个人博客：Git、Cloudflare 与自定义域名
pinned: true
category: daily
image: "/assets/images/bg.jpg"
published: 2026-09-17
description: 记录从静态 HTML 迁移到 Firefly Astro，并完成本地开发、Git 同步、Cloudflare 部署和自定义域名绑定。
tags: [Astro, Firefly, Git, Cloudflare, 建站]
---

这篇文章记录一条完整、可复现的建站路线：
本地运行 Firefly，使用 Markdown 写作，把源码同步到 GitHub，再由 Cloudflare 自动构建并发布，最后绑定自己的域名。文中的 `your-name`、`your-domain.com` 和 `your-repository` 都是占位符。

（基于建站过程中与AI互动总结）

## 先理解架构

```text
浏览器 -> 自定义域名 -> 
Cloudflare DNS/CDN -> Pages 或 Workers -> 
GitHub 源码 -> Astro 构建产物 dist
```

GitHub 保存源码和版本历史，Astro 把 Markdown、配置和组件编译成静态文件，Cloudflare 负责构建、托管和分发。GitHub Pages 也能托管静态站点：用户站点是 `your-name.github.io`，项目站点是 `your-name.github.io/project/`。

本文实际采用 Cloudflare 托管博客；已有的旅行册项目继续作为 GitHub Pages 项目站点独立运行，再从博客导航过去。

## 准备环境

安装 Node.js 22 或更高版本、pnpm、Git 和 VS Code，然后检查：

```bash
node -v
pnpm -v
git --version
```

## 获取 Firefly 并本地预览

```bash
git clone https://github.com/CuteLeaf/Firefly.git
cd Firefly
pnpm install
pnpm dev
```
这一步实际上是把这位大佬的项目存到本地
最终我们的博客更新实际上就是 loop：本地更新 -> 同步到GitHub仓库

打开终端显示的 `http://localhost:4321/`。保存配置或文章后，开发服务器会热更新。生产构建用 `pnpm build`，成功后会生成 `dist/`。

## 项目结构

```text
src/config/       # 站点、导航、个人资料、背景等配置
src/content/      # posts、dynamic、projects
src/components/   # 组件
src/pages/        # 路由
public/           # 原样复制到网站根目录的静态资源
astro.config.mjs  # Astro 配置
wrangler.jsonc    # 仓库可能自带的 Cloudflare 配置
```

需要直接通过 URL 访问的图片或文件，优先放进 `public/`。

## 第一次配置

在 `src/config/siteConfig.ts` 设置最终地址：

```ts
site_url: "https://your-domain.com",
```

它会影响规范链接、RSS、Atom、站点地图和 SEO。紫色主题可从 `hue: 277` 开始尝试；暗色模式使用 `defaultMode: "dark"`；`card.followTheme` 和 `navbar.followTheme` 可让组件跟随主题色。

在 `src/config/backgroundWallpaper.ts` 使用 `public/assets/images/bg.jpg`：

```ts
src: "/assets/images/bg.jpg",
playerEnable: false,
```

`hue` 只决定强调色，不是背景颜色；黑灰底色主要来自暗色模式和主题 CSS。

`profileConfig.ts` 中 `/assets/images/avatar.jpg` 对应 `public/assets/images/avatar.jpg`，邮箱必须是完整的 `mailto:`：

```ts
url: "mailto:your@email.com",
```

导航栏可用 `children` 创建下拉菜单。外部旅行册或仓库链接要加 `external: true`，否则主题可能按站内路由处理并产生 404。

## 写文章：Frontmatter

文章放在 `src/content/posts/`，文件开头和正文之间必须有成对的 `---`：

```md
---
title: 我的第一篇文章
published: 2026-09-17
description: 文章摘要
tags: [建站, Astro]
slug: first-post
---

正文从这里开始。
```

常用字段有 `title`、`published`、`description`、`tags`、`slug`、`draft`、`password`、`passwordHint`、`series` 和 `seriesOrder`。设置 `password` 后文章启用前端密码保护；不加密时不要填写。单回车不一定产生页面换行，分段请空一行，强制换行可在行末加两个空格。普通文章优先使用 `.md`，需要组件时再用 `.mdx`。

## 写项目：严格匹配 Schema

项目放在 `src/content/projects/`。当前 Firefly 文档使用 `image`，链接是 `{ label, icon, value }` 对象数组：

```md
---
title: 我的旅行册
published: 2026-09-17
description: 一份旅行计划
image: "/assets/images/trip.jpg"
status: published
tags: [旅行, 项目]
link:
  - label: 打开旅行册
    icon: material-symbols:menu-book
    value: https://your-name.github.io/chongqing-trip-book/
  - label: 查看仓库
    icon: fa7-brands:github
    value: https://github.com/your-name/your-repository
---

这里是项目 README 正文。
```

`link` 写成字符串、对象使用 `name/url`、把 `image` 写成 `cover`、缺少 `---` 或 YAML 缩进错误，都会在构建时出错。

封面可以使用相对路径、`public/` 根路径或网络 URL：

```yaml
image: "images/cover.webp"
image: "/assets/images/cover.webp"
image: "https://example.com/cover.webp"
```

新手最稳妥的方式是放入 `public/assets/images/` 并使用 `/assets/images/...`。直接访问 `http://localhost:4321/assets/images/cover.webp` 可以验证文件是否真的被公开。若采用 GitHub Pages 项目子路径，还要检查 `base`；自定义根域名通常不需要 `base`。

## 同步到自己的 GitHub 仓库

克隆原作者仓库不会占用自己的仓库。新建空仓库后：

```bash
git remote -v
git remote set-url origin https://github.com/your-name/your-repository.git
git remote -v
git add .
git commit -m "初始化个人博客"
git branch -M main
git push -u origin main
```

首次通过git push提交可能需要配置身份：

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

GitHub HTTPS 推送按提示完成浏览器授权或使用个人访问令牌。
遇到 `rejected` 先检查远程提交，不要未经确认强制推送；`Failed to connect` 通常需要检查网络、代理或 SSH。

以后每次更新就是：

```bash
pnpm dev        #这一步是正常的本地预览
git status      #检查一下是哪些文件改动了
git add .       #增加到缓存区
git commit -m "新增文章" #推送备注，在cloudflare也可以确认
git push                #推送
```

## Cloudflare 部署

Firefly 官方同时支持 Cloudflare Pages 和 Workers：

- Pages：`pnpm install`、`pnpm build`、输出目录 `dist`，静态站点不需要 Astro 适配器。
- Workers：检查仓库已有的 `wrangler.jsonc`，按项目配置使用 Wrangler 部署。

静态博客优先选择 Pages；如果已经创建 Workers 项目，就按 Workers 的实际配置操作，不要把两套配置混在一起。

Pages 控制台流程：连接 GitHub 仓库和 `main` 分支，构建命令填 `pnpm build`，输出目录填 `dist`，安装命令填 `pnpm install`，并将 `NODE_VERSION` 设为 `22` 或更高。

之后每次 `git push` 都会触发构建。默认 `pages.dev` 或 `workers.dev` 地址只是测试入口；构建成功不代表每个网络都能访问它。

## 绑定自定义域名

注册域名时查看续费价格、自动续费和邮箱验证。域名注册商、DNS 托管和 Cloudflare 项目绑定是三个不同环节。

在博客项目的自定义域名设置中添加 `your-domain.com`，需要时再添加 `www.your-domain.com`，等待 DNS 和证书状态正常。确认 `site_url` 使用正式域名后提交并推送一次。

如果 `workers.dev` 在某个网络中打不开但 DNS 正常，先用手机网络或代理交叉测试，区分构建失败、DNS 失败、证书问题和网络可达性。

自定义域名更适合作为正式入口，但不能保证绕过所有地区网络限制。

## 上线检查

- 首页、文章、项目页和图片没有 404。
- RSS、Atom、站点地图使用正式域名。
- 旅行册和技能包外链能打开。
- 加密文章、移动端和根域名/`www` 都测试过。
- Cloudflare 最近一次部署成功。

最终工作流是“写 Markdown → 本地预览 → Git 提交并推送 → Cloudflare 自动构建”。理解每一层的职责，今后更换主题、托管平台或域名时也不会被某个界面绑住。
