---
title: linux面试题
date: 2025-05-04 20:30:00 +0800
categories: [blog, 技术]
tags: [linux]     # TAG names should always be lowercase
---




## linux 下用什么命令能找出内存占用的前 5 名

```shell
ps -eo pid,user,%mem,args --sort=-%mem | head -n 6 | tail -n 5
```

