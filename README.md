# Astro Doge

一个静态优先的 Astro 6 博客模板。

基于 [Astro](https://astro.build) 和 [Tailwind CSS v4](https://tailwindcss.com) 构建，默认就能提供文章、碎碎念、搜索、RSS、PWA、目录导航等能力；评论、留言板和网页端发布则按需接入，不把每个使用者都绑定到同一种后端方案上。

## 预览

![Astro Doge](./preview.webp)

<details>
  <summary>Lighthouse 全绿，展开查看</summary>

![Lighthouse](./lighthouse.webp)

</details>

## 开箱即用

- 静态优先：默认输出静态站点，部署简单
- 内容系统：Markdown / MDX 文章与碎碎念
- 本地搜索：`Ctrl+K` 呼出搜索框
- 阅读体验：目录导航、图片灯箱、标题锚点、阅读时长、相关文章
- 内容展示：RSS、`robots.txt`、sitemap、外链标记、GitHub Alerts
- 视觉与设备支持：亮暗主题、PWA、移动端适配

## 可选增强

- 评论 UI：文章页评论区与留言板页面
- 网页端发布：`/thoughts/new`
- 这两类功能默认只提供前端页面和交互，不内置后端实现
- 如果你要接入自己的服务端，请遵循 [API Contract](./docs/api-contract.md)
- 如果你只想要纯静态博客，可以直接忽略这些可选功能

## 快速开始

### 1. 获取模板

```bash
git clone https://github.com/dogxii/astro-doge.git my-blog
cd my-blog
```

### 2. 安装依赖

推荐使用 [Bun](https://bun.sh)：

```bash
bun install
```

### 3. 本地开发

```bash
bun dev
```

打开 `http://localhost:4321`。

### 4. 构建检查

```bash
bun run build
```

## 先改这几处

初始化一个你自己的站点时，优先修改下面这些文件：

- `astro.config.mjs`
  设置 `site`，并把 `allowHostnames` 改成你自己的域名
- `src/consts.ts`
  站点名、描述、邮箱、社交链接、项目、技术栈都在这里
- `src/pages/about/index.astro`
  默认 About 页面文案
- `src/components/Header.astro`
  导航链接
- `src/components/Footer.astro`
  页脚署名和链接
- `public/avatar.png`、`public/favicon.ico`、`public/manifest.json`
  头像、图标和 PWA 元数据
- `src/content/posts/`、`src/content/thoughts/`
  删掉示例内容，换成你自己的文章和碎碎念

## 写内容

### 写文章

```bash
bun new:blog my-first-post
```

或者手动在 `src/content/posts/` 下创建 `.md` / `.mdx` 文件：

```md
---
title: 我的第一篇文章
description: 一段摘要
date: 2026-04-23T09:00:00+08:00
slug: my-first-post
cover: /images/my-cover.webp
draft: false
---

正文内容。
```

字段说明：

| 字段          | 类型       | 说明                   |
| ------------- | ---------- | ---------------------- |
| `title`       | `string`   | 标题                   |
| `description` | `string`   | 摘要，可选             |
| `date`        | `ISO 8601` | 发布时间               |
| `slug`        | `string`   | 自定义链接，可选       |
| `cover`       | `string`   | 文章封面图，可选       |
| `draft`       | `boolean`  | 为 `true` 时不参与构建 |

### 写碎碎念

```bash
bun t
```

也可以直接带上文件名和内容：

```bash
bun t random-name "今天天气不错"
```

或者手动在 `src/content/thoughts/` 下创建文件：

```md
---
date: 2026-04-23T09:00:00+08:00
draft: false
tags:
  - dev
  - note
---

今天又学到一个新东西。
```

## 部署

### 纯静态部署

这是模板的默认工作方式：

```bash
bun run build
```

产物在 `dist/`，可以直接部署到：

- Vercel
- Netlify
- Cloudflare Pages
- GitHub Pages

### 可选功能接入

如果你需要评论、留言板或 `/thoughts/new`，你需要自己提供后端接口。

前端默认会请求这些地址：

- `GET /api/comments?slug=...`
- `POST /api/submit-comment`
- `POST /api/add-thought`

接口所需的请求体和响应格式见 [docs/api-contract.md](./docs/api-contract.md)。

如果你不需要这些功能，可以删除这些文件或引用：

- `src/components/Comments.astro`
- `src/pages/messages/index.astro`
- `src/pages/thoughts/new.astro`
- `src/components/Header.astro` 中对应导航项
- `src/pages/[...slug].astro` 中的评论区引用

## 环境变量

`.env.example` 提供的是一套“推荐命名”，用于你自己实现可选 API 时参考。

如果你只运行纯静态博客，可以完全不配置任何环境变量。

如果你要接入评论或网页端发布，建议至少区分这几类变量：

| 变量                                         | 用途                                        |
| -------------------------------------------- | ------------------------------------------- |
| `SITE_URL`                                   | 站点地址，便于 API 生成链接 / 做 CORS       |
| `GITHUB_TOKEN`                               | 访问 GitHub API 的 Token                    |
| `COMMENTS_REPO`                              | 评论数据仓库，例如 `yourname/blog-comments` |
| `OWNER_NAME` / `OWNER_EMAIL` / `OWNER_TOKEN` | 博主身份校验                                |
| `THOUGHT_API_TOKEN`                          | `/thoughts/new` 页面使用的提交口令          |
| `CONTENT_REPO` / `CONTENT_BRANCH`            | 网页端发布要写入的内容仓库与分支            |

## 项目结构

```text
.
├── public/               # 静态资源
├── scripts/              # 创建文章 / 碎碎念的辅助脚本
├── src/
│   ├── components/       # 组件
│   ├── content/          # 示例内容
│   ├── layouts/          # 布局
│   ├── lib/              # 工具函数、remark/rehype 插件
│   ├── pages/            # 页面与路由
│   ├── styles/           # 全局样式
│   ├── consts.ts         # 站点配置
│   └── content.config.ts # 内容集合定义
├── docs/
│   └── api-contract.md   # 可选 API 契约
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

## 致谢

- 主题基于 [astro-nano](https://github.com/markhorn-dev/astro-nano) 演化而来
- 碎碎念和留言板交互设计参考了 [Viki](https://github.com/vikiboss/blog) 的一些思路

## License

[MIT](./LICENSE)
