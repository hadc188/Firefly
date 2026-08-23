# Volant

个人博客，基于 [Firefly](https://github.com/CuteLeaf/Firefly) 主题二改。

- 站点：<https://volant.cc.cd>
- 技术栈：Astro + Svelte + Tailwind CSS

## 本地开发

要求 Node.js ≥ 22、pnpm ≥ 9。

```bash
pnpm install
pnpm dev      # http://localhost:4321
pnpm build    # 构建到 dist/
```

## 写文章

文章放在 `src/content/posts/`，Markdown 文件 + frontmatter：

```yaml
---
title: 文章标题
published: 2026-08-23
description: 一句话简介
tags: [标签1, 标签2]
category: 技术
draft: false
---
```

脚手架：`pnpm new-post 文章文件名`

写完 `git push`，Cloudflare 自动构建上线。

## 部署

Cloudflare Workers 托管，GitHub 关联自动构建。需要在 Cloudflare 构建变量中设置 `CF_WORKERS=1`。

详细步骤见部署教程：<https://volant.cc.cd/posts/cf-workers-deploy-guide/>

## 致谢

- [Firefly](https://github.com/CuteLeaf/Firefly) —— 本站使用的主题
- [fuwari](https://github.com/saicaca/fuwari) —— Firefly 的上游模板

[MIT License](./LICENSE)，请保留上游版权声明。
