---
title: linux面试题
layout: page
---



## linux 下用什么命令能找出内存占用的前 5 名

```shell
ps -eo pid,user,%mem,args --sort=-%mem | head -n 6 | tail -n 5
```

