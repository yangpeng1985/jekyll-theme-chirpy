---
title: linux服务器安全设置
layout: post
---

注：这是一篇老文章，写于2018年5月12日。

<div class="ez-toc-v2_0_66_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction" id="ez-toc-container"><div class="ez-toc-title-container">目录

<span class="ez-toc-title-toggle">[<span class="ez-toc-js-icon-con"><span class=""><span class="eztoc-hide" style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg class="list-377408" fill="none" height="20px" style="fill: #999;color:#999" viewbox="0 0 24 24" width="20px" xmlns="http://www.w3.org/2000/svg"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg baseprofile="tiny" class="arrow-unsorted-368013" height="10px" style="fill: #999;color:#999" version="1.2" viewbox="0 0 24 24" width="10px" xmlns="http://www.w3.org/2000/svg"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"></path></svg></span></span></span>](#)</span></div><nav>

</nav></div>## <span class="ez-toc-section" id="%E7%BB%99%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7%E6%B7%BB%E5%8A%A0sudo%E6%9D%83%E9%99%90"></span>给普通用户添加sudo权限<span class="ez-toc-section-end"></span>

### <span class="ez-toc-section" id="%E6%B7%BB%E5%8A%A0%E6%99%AE%E9%80%9A%E7%94%A8%E6%88%B7"></span>添加普通用户<span class="ez-toc-section-end"></span>

```
useradd commonuser
passwd commonuser
```

### <span class="ez-toc-section" id="%E4%BD%BF%E7%94%A8root%E8%B4%A6%E5%8F%B7%E6%89%A7%E8%A1%8Cvisudo"></span>使用root账号执行visudo<span class="ez-toc-section-end"></span>

```
[root@www ~]# visudo
....(前面省略)....
root    ALL=(ALL)       ALL  
```

### <span class="ez-toc-section" id="SUDO_%E4%B8%8E_su%E7%9A%84%E5%8C%BA%E5%88%AB"></span>SUDO 与 su的区别<span class="ez-toc-section-end"></span>

sudo命令不需要输入root的密码，只需要输入自己的密码就可以拥有root的权限，su命令需要输入root账号密码；普通用户执行sudo su -可以只输入普通用户密码切到root账号；

### <span class="ez-toc-section" id="sudo_su"></span>sudo su -<span class="ez-toc-section-end"></span>

很多时候我们需要大量运行很多 root 的工作，所以一直使用 sudo 觉得很烦人！那有没有办法使用 sudo 搭配 su ， 一口气将身份转为 root ，而且还用用户自己的口令来变成 root 呢？是有的！而且方法简单的会让你想笑！ 我们创建一个 ADMINS 帐户别名，然后这样做：  
\[root@www ~\]# visudo  
User\_Alias ADMINS = pro1, pro2, pro3, myuser1  
ADMINS ALL=(root) /bin/su -

接下来，上述的 pro1, pro2, pro3, myuser1 这四个人，只要输入『 sudo su - 』并且输入『自己的口令』后， 立刻变成 root 的身份！不但 root 口令不会外流，用户的管理也变的非常方便！ 这也是实务上面多人共管一部主机时常常使用的技巧呢！这样管理确实方便，不过还是要强调一下大前提， 那就是『这些你加入的使用者，全部都是你能够信任的用户』！

From Linux 账号管理与 ACL 权限配置 http://cn.linux.vbird.org/linux_basic/0410accountmanager.php

## <span class="ez-toc-section" id="Linux%E4%BF%AE%E6%94%B9ssh%E7%AB%AF%E5%8F%A322"></span>Linux修改ssh端口22<span class="ez-toc-section-end"></span>

修改端口配置文件,取消Port前的#注释，并将端口22改成65535，可以改成从1025到65536之间的任意一个整数

## <span class="ez-toc-section" id="%E5%85%AC%E7%A7%81%E9%92%A5%E8%AE%A4%E8%AF%81"></span>公私钥认证<span class="ez-toc-section-end"></span>

1. 执行命令ssh-keygen -t rsa -b 4096，根据需要可以输入密码，在~/.ssh目录下生成公钥私钥id\_rsa.pub和id\_rsa
2. copy公钥id\_rsa.pub到连接这台服务器的其他服务器上
3. 注意： copy私钥id\_rsa到本地,以后在本地连接这台服务器可以执行这样的命令, ssh -i id\_rsa account\_name@server\_ip -p port（中间找不到密钥，重新生成了一次，搞混了公钥和私钥，导致死活连不上服务器）
4. 添加公钥到.ssh/authorized\_keys中，cp ~/.ssh/id\_rsa.pub ~/.ssh/authorized\_keys
5. 设置权限 
  - chmod 0700 ~/.ssh/
  - chmod 0600 ~/.ssh/authorized\_keys
6. 保险起见，删除公钥 rm id\_rsa.pub
7. 接着修改ssh配置文件，开启如下内容 
  - RSAAuthentication yes
  - PubkeyAuthentication yes
  - AuthorizedKeysFile .ssh/authorized\_keys
  - 禁止password方式登录 PasswordAuthentication no
  - 禁止root登录 PermitRootLogin no
  - 禁止空密码 PermitEmptyPasswords no
  - 添加允许登录的用户名（在最后加上） AllowUsers abc
  - 重启sshd服务,sudo service sshd restart
8. 在ssh工具，比如xshell，配置证书登录 
  1. 新建连接
  2. 点击Authentication, --Method选择Public Key，点击Browse
  3. 点击导入私钥，需要输入之前生成秘钥时设置的密码；如果出现导入秘钥错误，尝试导入公钥时会出现此问题
9. 以后登录只能使用证书登录，密码无法登录

### <span class="ez-toc-section" id="%E5%85%AC%E9%92%A5%E7%99%BB%E5%BD%95%E5%8E%9F%E7%90%86"></span>公钥登录原理<span class="ez-toc-section-end"></span>

所谓"公钥登录"，原理很简单，就是用户将自己的公钥储存在远程主机上。登录的时候，远程主机会向用户发送一段随机字符串，用户用自己的私钥加密后，再发回来。远程主机用事先储存的公钥进行解密，如果成功，就证明用户是可信的，直接允许登录shell，不再要求密码。