---
layout: post
title: md文件在GitHub page页面不显示的原因
---

>You can add additional posts in the browser on GitHub.com too!
Just hit the + icon in /_posts/ to create new content. 

>Just make sure to include the [front-matter](https://jekyllrb.com/docs/front-matter/) block at the top of each new blog post and make sure the post's filename is in this format: year-month-day-title.md

## 问题现象：

将一篇md文件，添加到_posts目录，在GitHub Page页面一直看不到

## 原因：

md文件的格式没有按照year-month-day-title.md的格式，day和title之间用了下划线，而不是中横线。