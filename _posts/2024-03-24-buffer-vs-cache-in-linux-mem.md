---
title: 通过free命令查看内存时，看到的buffer和cache各表示什么含义？
layout: post
---

free命令显示了系统的可用和已用的物理内存及交换内存的总量，以及内核用到的缓存空间。这些信息是从 /proc/meminfo 中得到的

看下这篇文章中的的内容：

<https://www.cnblogs.com/kevingrace/p/5991604.html>

<div class="ez-toc-v2_0_66_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction" id="ez-toc-container"><div class="ez-toc-title-container">目录

<span class="ez-toc-title-toggle">[<span class="ez-toc-js-icon-con"><span class=""><span class="eztoc-hide" style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg class="list-377408" fill="none" height="20px" style="fill: #999;color:#999" viewbox="0 0 24 24" width="20px" xmlns="http://www.w3.org/2000/svg"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg baseprofile="tiny" class="arrow-unsorted-368013" height="10px" style="fill: #999;color:#999" version="1.2" viewbox="0 0 24 24" width="10px" xmlns="http://www.w3.org/2000/svg"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"></path></svg></span></span></span>](#)</span></div><nav>- [buffers](http://thinknotes.cn/2024/03/24/buffer-vs-cache-in-linux-mem/#buffers "buffers")
- [cache](http://thinknotes.cn/2024/03/24/buffer-vs-cache-in-linux-mem/#cache "cache")
- [buff/cache](http://thinknotes.cn/2024/03/24/buffer-vs-cache-in-linux-mem/#buffcache "buff/cache")
- [used](http://thinknotes.cn/2024/03/24/buffer-vs-cache-in-linux-mem/#used "used")
- [available 可用内存](http://thinknotes.cn/2024/03/24/buffer-vs-cache-in-linux-mem/#available_%E5%8F%AF%E7%94%A8%E5%86%85%E5%AD%98 "available 可用内存")

</nav></div>## <span class="ez-toc-section" id="buffers"></span>buffers<span class="ez-toc-section-end"></span>

内核缓冲区使用的内存，等同于/proc/meminfo中的Buffers

常见的有把内存的数据往磁盘进行写操作

## <span class="ez-toc-section" id="cache"></span>cache<span class="ez-toc-section-end"></span>

页面缓存和Slab分配机制使用的内存，等同于/proc/meminfo中的Cached和Slab

## <span class="ez-toc-section" id="buffcache"></span>buff/cache<span class="ez-toc-section-end"></span>

buffers 与cache 之和

## <span class="ez-toc-section" id="used"></span>used<span class="ez-toc-section-end"></span>

used memory  
total - free - buffers - cache

## <span class="ez-toc-section" id="available_%E5%8F%AF%E7%94%A8%E5%86%85%E5%AD%98"></span>available 可用内存<span class="ez-toc-section-end"></span>

以下内容来源于man free的中文翻译：

等同于/proc/meminfo中的MemAvailable值。  
在内核3.14以上，/proc/meminfo添加了新的指标MemAvailable，3.14之前等于free字段的值，即剩余物理内存大小。

在系统没有发生交换时，预估需要多少available内存才可以启动新的应用程序。这个available字段不同于cache和free字段所提供的数据，它除了要考虑到page cache，还要考虑到当项目在使用时，并不是所有的可回收的内存块都能被回收这一情况。

参考 ： <https://linux.cn/article-9232-1.html>