---
layout: post
title: 使用Cloudflare和GitHub Pages托管个人博客
---

# GitHub Pages 部分

该部分完成后，可以使用默认域名访问，默认域名： github的username.github.io

## 复制代码仓库，修改新代码仓库名称

>Head to [Jekyll Now](https://github.com/barryclark/jekyll-now), and hit the “Fork” button in the top-right corner of the repository to fork a copy of the blog theme to your GitHub account.

>As a GitHub user, you’re entitled to one free “user” website (as opposed to a “project” website), which will live at https://yourusername.github.io. This space is ideal for hosting a Jekyll blog!

>The best part is that you can simply place your unbuilt Jekyll blog on the master branch of your user repository, and GitHub Pages will build the static website and serve it for you. You don’t have to worry about the build process at all — it’s all taken care of.

>Click the “Settings” button in your new forked repository (in the menu on the right), and change the repository’s name to yourusername.github.io, replacing yourusername with your GitHub user name.

一句话总结：首先，点击fork按钮，从Jekyii Now 复制一个代码仓库；然后，在新代码仓库settings中修改仓库名称；接着访问 https://yourusername.github.io 访问你的个人博客。


## 个性化设置

> You can now change your website’s name, description, avatar and other options by editing the _config.yml file

修改代码仓库中的_config.yml文件个性化自己的博客。

代码仓库的任何变动都会触发github使用Jekyii重新构建。

## 发布第一篇博客

>You can add additional posts in the browser on GitHub.com too!
Just hit the + icon in /_posts/ to create new content. 

>Just make sure to include the [front-matter](https://jekyllrb.com/docs/front-matter/) block at the top of each new blog post and make sure the post's filename is in this format: year-month-day-title.md

按照以上格式要求，添加md文件目录_posts下。


# Cloudflare 部分

这里使用 cloudflare 作为个人博客的 CDN，请求先到Cloudflare。

## 将域名解析到Cloudflare

你需要将域名的 DNS 解析指向Cloudflare。具体步骤如下：

  1. 登录到你域名注册商的管理后台。
  2. 找到 DNS 管理功能，将域名的DNS 服务器更改为 Cloudflare 提供的 DNS 服务器。Cloudflare 会提供两组 DNS 服务器地址：

     - `ns1.cloudflare.com`
     - `ns2.cloudflare.com`

## 在Cloudflare中配置DNS记录

在 Cloudflare 面板中，进行以下设置：

1. **添加 CNAME 记录**

   - 在 Cloudflare 中，选择你的域名，进入 "DNS" 设置。

   - 添加一条CNAME记录：
     - **类型**：CNAME
     - **名称**：`www`（也可以是其他子域名）
     - **目标**：`username.github.io`（将 `username` 替换为你的 GitHub 用户名）
     - **TTL**：自动或选择最低值

2. **添加 A 记录**（可选，但推荐）

   - 如果你希望支持裸域名（例如：yourdomain.com，没有 www），需要添加 A 记录，将裸域名指向 GitHub Pages 的 IP 地址。GitHub 提供的 IP 地址如下：

     - `185.199.108.153`
     - `185.199.109.153`
     - `185.199.110.153`
     - `185.199.111.153`

   - 添加 4 条 A 记录，分别指向这 4 个 IP 地址。

# 在GitHub 中启用自定义域名

在仓库 settings -> Pages -> Custom domain，填写自己的域名（你在Cloudflare中设置的自定义域名），save后可以在代码仓库的CNAME文件看到变化。

# 测试

访问网站的同时，F12打开开发者工具，可以看到Server是cloudflare，说明已经成功了。

# 参考

* [Build A Blog With Jekyll And GitHub Pages](https://www.smashingmagazine.com/2014/08/build-blog-jekyll-github-pages/)