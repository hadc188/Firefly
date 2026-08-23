---
title: Firefly 部署到 Cloudflare：从本地到上线
published: 2026-07-18
updated: 2026-08-23
draft: false
description: 按步骤把 Firefly 部署到 Cloudflare Workers，关联 GitHub 自动构建，push 后自动上线。
image: ""
tags: [Cloudflare, Astro, Firefly, 部署]
category: 技术
lang: ""
pinned: true
author: ""
sourceLink: ""
licenseName: ""
licenseUrl: ""
comment: true
---

根据本站实际部署过程整理，步骤可直接跟着做。命令可复制。

说明：基于 Windows / 已会用终端；主题为 [Firefly](https://github.com/CuteLeaf/Firefly)（Astro 静态博客）。官方主题**没有**网页发文后台，写文章就是在本地改 Markdown，push 后自动上线。

本站仓库示例：

-   GitHub：https://github.com/hadc188/Firefly

-   站点：https://volant.cc.cd


本仓库是官方主题的二改版本；如需使用官方原版，可参考这篇教程：[Firefly 部署教程](https://www.fqzlr.com/posts/firefly-set/windows-firefly)。

如果你 Fork 到了自己的仓库，把下文里的 `hadc188` 换成你的用户名即可。默认分支以 **master** 为例（若是 `main`，全文替换）。

## 一、你要达成什么

-   代码放在 **GitHub**

-   **Cloudflare** 自动：拉代码 → 安装依赖 → 构建 → 上线

-   以后改配置、写文章：本地改完 `git push`，网站自动更新

-   不用自己买服务器也能先用 `*.workers.dev` 访问


整体链路：

```text
本地改代码 → push 到 GitHub → Cloudflare 自动构建 → 域名可访问
```

## 二、前置准备

先确认这几样都有：

1.  **Node.js ≥ 22**（终端执行 `node -v` 有版本号）

2.  **pnpm**（`pnpm -v` 有版本号；没有就：`npm install -g pnpm`）

3.  **Git**（`git -v` 正常）

4.  **GitHub 账号**，并拥有一份 Firefly 代码：

    -   Fork 本站二改版：[hadc188/Firefly](https://github.com/hadc188/Firefly)

5.  **Cloudflare 账号**（[注册/登录](https://dash.cloudflare.com/)）


本地先跑通：

```bash
git clone https://github.com/你的用户名/Firefly.git
cd Firefly
pnpm install
pnpm dev
```

浏览器打开 `http://localhost:4321`，首页正常即可。

## 三、仓库里和部署相关的配置

### 1. 站点地址

改 `src/config/siteConfig.ts` 里的 `site_url`，例如本站：

```ts
site_url: "https://volant.cc.cd",
```

没有正式域名时，可先填 Cloudflare 给的 `*.workers.dev`，绑定域名后再改一次并 push。

### 2. Wrangler 配置

项目根目录需要有 `wrangler.jsonc`（名称可按项目改）。本站大致如下：

```jsonc
{
  "name": "firefly",
  "compatibility_date": "2025-01-01",
  "compatibility_flags": ["nodejs_compat"],
  "assets": {
    "directory": "./dist"
  }
}
```

说明：

-   `assets.directory`：构建产物目录，Firefly 默认是 `dist`

-   纯静态托管时，这样即可；若你后续加了运行时接口，再按官方文档补 `main`、`ASSETS` 绑定等


改完推到 GitHub：

```bash
git add .
git commit -m "chore: 准备 Cloudflare 部署"
git push
```

## 四、Cloudflare 关联 GitHub 并部署

### 1. 创建应用

1.  打开 [Cloudflare 控制台](https://dash.cloudflare.com/)

2.  左侧进入 **Workers 和 Pages**

3.  **创建应用程序** → 连接到 **Git** / **GitHub**

4.  授权后选中仓库：**你自己的 Fork 仓库**

5.  构建相关建议填：


| 项   | 建议值 |
| --- | --- |
| 构建命令 | `pnpm build` |
| 部署命令 | `npx wrangler deploy`（若控制台要求填写） |
| 分支  | `master`（或你的默认分支） |

6.  点部署，等第一次构建跑完（首次构建会报错，属预期，下一小节解决）


### 2. 首次构建会报错，添加构建变量

**配置构建环境变量 `CF_WORKERS=1`（修复报错，必做）**

```text
CF dashboard → Workers & Pages → 选择 firefly 项目 → Settings → Build（或 Builds & deployments）→ Environment variables
```

> 注意：此处为**构建变量**，不是运行时 Secrets

| 变量名 | 值   |
| --- | --- |
| CF\_WORKERS | 1   |

保存变量后，进入 `Deployments` 页面，点击 `Retry deployment` 重新部署；也可推送一条空 commit 触发新构建。配置完成后，`NoAdapterInstalled` 报错会直接消失。

### 3. 验证是否成功

构建状态变绿后，打开项目提供的临时域名（形如 `xxx.workers.dev`）。<br>能打开博客首页，且和本地预览大致一致，说明自动部署已通。

之后每次：

```bash
git add .
git commit -m "更新内容"
git push
```

Cloudflare 会再自动构建上线，一般 1～3 分钟。

## 五、绑定自己的域名（可选）

临时域名能用就可以先写文章；有域名再绑。本站绑定后为 https://volant.cc.cd

1.  进入该 Worker 项目 → **设置 / 触发器** → **自定义域** → 添加你的域名

2.  按提示在域名注册商处添加 **CNAME**（主机 `@` 或 `www`，目标按 CF 页面显示填写）

3.  等待解析生效（几分钟到数小时）

4.  浏览器访问你的域名，能打开且有 HTTPS 小锁即成功


同时把 `siteConfig.ts` 里的 `site_url` 改成正式域名，再 push 一次。

## 六、日常怎么写文章

Firefly **没有**自带的网页后台。文章目录：

```text
src/content/posts/
```

新建 `xxx.md`，开头写 frontmatter，例如：

```yaml
---
title: 文章标题
published: 2026-07-18
description: 一句话简介
tags: [标签1, 标签2]
category: 技术
draft: false
---
```

注意：**日期不要加引号**，写成 `2026-07-18`，不要写成 `"2026-07-18"`，否则部分环境下构建可能报类型错误。

本地预览：

```bash
pnpm dev
```

满意后：

```bash
git add src/content/posts/
git commit -m "docs: 新增文章"
git push origin master
```

等 Cloudflare 构建完成，文章就上线了。

也可使用官方脚手架：

```bash
pnpm new-post 文章文件名
```

## 七、常见问题（避坑）

| 现象  | 处理  |
| --- | --- |
| 本地 `pnpm dev` 打不开 | 看终端是否在跑、端口是否为 4321；依赖是否 `pnpm install` 成功 |
| 构建失败 | 打开 Cloudflare Deployments 看日志；常见是 Node 版本、依赖、或 frontmatter 写错 |
| 构建时报 NoAdapterInstalled | 未配置构建变量 `CF_WORKERS=1`；值必须是数字 1，且放在构建变量（Build → Environment variables），不能放在运行时 Secrets |
| push 了但网站没变 | 确认推的是构建所用分支（如 master）；看部署是否失败 |
| 首页域名/链接不对 | 检查 `siteConfig.ts` 的 `site_url` 是否已改并已 push |
| `published` 类型错误 | frontmatter 日期裸写，不要加引号 |

## 写在最后

部署核心三步：

1.  有自己的 GitHub 仓库（本站：[hadc188/Firefly](https://github.com/hadc188/Firefly)）

2.  本地 `pnpm install` + `pnpm dev` 能预览

3.  Cloudflare 连上该仓库，构建命令用 `pnpm build`，push 即自动上线


写文章就是改 `src/content/posts/` 下的 Markdown，再 `git push`（见第六节）。

-   本站仓库：https://github.com/hadc188/Firefly

-   本站地址：https://volant.cc.cd

-   Firefly 官方源码：https://github.com/CuteLeaf/Firefly

-   Cloudflare 控制台：https://dash.cloudflare.com/
