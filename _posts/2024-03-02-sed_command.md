---
title: sed命令
layout: post
---

注：这是一篇老文章，写于2019年1月20日。

<div class="ez-toc-v2_0_66_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction" id="ez-toc-container"><div class="ez-toc-title-container">目录

<span class="ez-toc-title-toggle">[<span class="ez-toc-js-icon-con"><span class=""><span class="eztoc-hide" style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg class="list-377408" fill="none" height="20px" style="fill: #999;color:#999" viewbox="0 0 24 24" width="20px" xmlns="http://www.w3.org/2000/svg"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg baseprofile="tiny" class="arrow-unsorted-368013" height="10px" style="fill: #999;color:#999" version="1.2" viewbox="0 0 24 24" width="10px" xmlns="http://www.w3.org/2000/svg"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"></path></svg></span></span></span>](#)</span></div><nav>- [替换某个关键词开头的整行内容](http://thinknotes.cn/2024/03/02/sed_command/#%E6%9B%BF%E6%8D%A2%E6%9F%90%E4%B8%AA%E5%85%B3%E9%94%AE%E8%AF%8D%E5%BC%80%E5%A4%B4%E7%9A%84%E6%95%B4%E8%A1%8C%E5%86%85%E5%AE%B9 "替换某个关键词开头的整行内容")
- [替换某个关键词，仅替换该关键词，而不是整行替换](http://thinknotes.cn/2024/03/02/sed_command/#%E6%9B%BF%E6%8D%A2%E6%9F%90%E4%B8%AA%E5%85%B3%E9%94%AE%E8%AF%8D%EF%BC%8C%E4%BB%85%E6%9B%BF%E6%8D%A2%E8%AF%A5%E5%85%B3%E9%94%AE%E8%AF%8D%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AF%E6%95%B4%E8%A1%8C%E6%9B%BF%E6%8D%A2 "替换某个关键词，仅替换该关键词，而不是整行替换")

</nav></div>## <span class="ez-toc-section" id="%E6%9B%BF%E6%8D%A2%E6%9F%90%E4%B8%AA%E5%85%B3%E9%94%AE%E8%AF%8D%E5%BC%80%E5%A4%B4%E7%9A%84%E6%95%B4%E8%A1%8C%E5%86%85%E5%AE%B9"></span>替换某个关键词开头的整行内容<span class="ez-toc-section-end"></span>

```shell
sed -i '/^keyword/cnewkeyword' file.txt
```

-i参数表示直接修改file.txt文件，keyword是要替换的行的关键词，这里使用^keyword的意思是寻找keyword开头的行，然后使用newkeyword替换，  
①注意newkeyword前面有一个字母c是关键，  
②newkeyword后面没有左斜线这个也需要特别注意，  
③这个替换是全局行替换

## <span class="ez-toc-section" id="%E6%9B%BF%E6%8D%A2%E6%9F%90%E4%B8%AA%E5%85%B3%E9%94%AE%E8%AF%8D%EF%BC%8C%E4%BB%85%E6%9B%BF%E6%8D%A2%E8%AF%A5%E5%85%B3%E9%94%AE%E8%AF%8D%EF%BC%8C%E8%80%8C%E4%B8%8D%E6%98%AF%E6%95%B4%E8%A1%8C%E6%9B%BF%E6%8D%A2"></span>替换某个关键词，仅替换该关键词，而不是整行替换<span class="ez-toc-section-end"></span>

```shell
sed -i 's/keyword/newkeyword/g' file.txt
```

-i参数表示直接修改file.txt,后面的g表示替换行内所有keyword为newkeyword,  
如果不加g表示仅替换行内第一个匹配的keyword为newkeyword