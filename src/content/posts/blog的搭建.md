---
title: 我的第一篇文章
published: 2026-06-07
description: 搭建自己的博客~
image: https://cdn.jsdelivr.net/gh/JuMoXia/blog-images/img/0aa460ac6aa59cad4236a835cbb93c4a.jpg
tags: [开发]
category: 开发
draft: false
comment: true
---

# 前言

在一个偶然的时候，发现了一位大佬搭建的个人博客，后来就想着自己也搞一个自己的博客。
  下面是博客的搭建过程

## 1. 博客本体搭建

### 1.1 前期准备：Fork 自己喜欢的主题

大家可以在GitHub上搜寻自己喜欢的主题，fork到自己的仓库(赞美Github和愿意公开自己源码的大佬们)
我的是在GitHub上fork[FireFly](https://github.com/CuteLeaf/Firefly)的主题，

### 1.2  本地克隆与清理

- 使用 `git clone` 将 fork 的仓库拉取到本地。
- 删除 `content/posts/` 下所有原作者的文章（`.md`）。

## 2. 本地运行与依赖安装

### 2.1 **安装 Node.js**和 **pnpm**：

```cmd
node -v
npm -v
```
如果显示node和npm的版本号，则安装成功。

### 2.2 **安装项目依赖**：

```cmd
pnpm install
```

### 2.3 **本地预览**：

```cmd
pnpm dev→ 访问 `http://localhost:3000` 确认博客可运行。
``` 

## 3. 推送代码到 GitHub
- 国内访问 GitHub 不稳定，`git push` 频繁出现问题
- **解决方案**：安装并使用 **DevSidecar** 加速工具，开启系统代理和 Git 代理。

## 4.部署到 Vercel

### 4.1 步骤
1.使用 GitHub 账号授权。
2.导入自己的仓库，框架会自己识别为Nuxt.js。
3.默认构建配置。
4.点击Deploy，获得预览域名。
5.每次 git push 会自动触发 Vercel 重新部署。

## 5.绑定自定义域名

### 5.1**购买域名**：

在阿里云购买 自己喜欢的域名(很便宜首年十几块)，并完成实名认证。

### 5.2**域名托管到 Cloudflare**：

- 注册 Cloudflare，添加站点，免费套餐。
- 将阿里云的 DNS 服务器改为 Cloudflare 分配的 NS 地址。
- 等待 DNS 生效（状态变为 Active）。

### 5.3**在 Vercel 中添加域名**：

- 项目 Settings → Domains → 添加域名和子域名。
- 等待状态变为 `Valid Configuration`（绿色）。
  注：添加这条记录时，最右边那个代理状态（Proxy status）的橙色小云朵，需要点灰它，不然 CF 的证书和 Vercel 的证书可能会发生冲突，导致 Vercel 证书申请失败。

### 5.4 **访问验证**：

访问自己购买的域名 成功打开个人博客

## 6.配置图床（GitHub + jsDelivr + PicGo）

### 6.1  **创建**GitHub 公开仓库** `JuMoXia/blog-images`。
### 6.2  **生成**Personal Access Token（经典令牌，勾选 `repo`）。
### 6.3  **安装** **PicGo**，配置 GitHub 图床：
   - 仓库名：`JuMoXia/blog-images`
   - 分支：`main`
   - Token：粘贴 Personal Access Token
   - 存储路径：`img/`
   - 
### 6.4 **测试**：拖拽图片，成功生成 CDN 链接。

### 6.5 **与 Typora 联动**：
- Typora 偏好设置 → 图像 → 上传服务选 PicGo(app)，指定 PicGo.exe 路径。
- 粘贴图片时自动上传并返回 CDN 链接。

## 7.总结
  搭建属于自己的博客也是非常有成就感的
  虽然我的水平很烂，但还好有GitHub上大佬的源码，我只需要改一部分就可以了，赞美大佬