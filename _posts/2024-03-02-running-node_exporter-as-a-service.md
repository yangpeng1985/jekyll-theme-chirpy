---
title: 'Linux系统下注册node exporter为systemd服务'
layout: post
---

注：这是一篇老文章，写于2018年5月31日。

<div class="ez-toc-v2_0_66_1 counter-hierarchy ez-toc-counter ez-toc-grey ez-toc-container-direction" id="ez-toc-container"><div class="ez-toc-title-container">目录

<span class="ez-toc-title-toggle">[<span class="ez-toc-js-icon-con"><span class=""><span class="eztoc-hide" style="display:none;">Toggle</span><span class="ez-toc-icon-toggle-span"><svg class="list-377408" fill="none" height="20px" style="fill: #999;color:#999" viewbox="0 0 24 24" width="20px" xmlns="http://www.w3.org/2000/svg"><path d="M6 6H4v2h2V6zm14 0H8v2h12V6zM4 11h2v2H4v-2zm16 0H8v2h12v-2zM4 16h2v2H4v-2zm16 0H8v2h12v-2z" fill="currentColor"></path></svg><svg baseprofile="tiny" class="arrow-unsorted-368013" height="10px" style="fill: #999;color:#999" version="1.2" viewbox="0 0 24 24" width="10px" xmlns="http://www.w3.org/2000/svg"><path d="M18.2 9.3l-6.2-6.3-6.2 6.3c-.2.2-.3.4-.3.7s.1.5.3.7c.2.2.4.3.7.3h11c.3 0 .5-.1.7-.3.2-.2.3-.5.3-.7s-.1-.5-.3-.7zM5.8 14.7l6.2 6.3 6.2-6.3c.2-.2.3-.5.3-.7s-.1-.5-.3-.7c-.2-.2-.4-.3-.7-.3h-11c-.3 0-.5.1-.7.3-.2.2-.3.5-.3.7s.1.5.3.7z"></path></svg></span></span></span>](#)</span></div><nav>- [add user prometheus](http://thinknotes.cn/2024/03/02/running-node_exporter-as-a-service/#add_user_prometheus "add user prometheus")
- [Running Node Exporter as a Service](http://thinknotes.cn/2024/03/02/running-node_exporter-as-a-service/#Running_Node_Exporter_as_a_Service "Running Node Exporter as a Service")
- [批量注册Node Exporter为系统服务](http://thinknotes.cn/2024/03/02/running-node_exporter-as-a-service/#%E6%89%B9%E9%87%8F%E6%B3%A8%E5%86%8CNode_Exporter%E4%B8%BA%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1 "批量注册Node Exporter为系统服务")
- [使用python脚本进行批量注册操作(需要在root下执行）](http://thinknotes.cn/2024/03/02/running-node_exporter-as-a-service/#%E4%BD%BF%E7%94%A8python%E8%84%9A%E6%9C%AC%E8%BF%9B%E8%A1%8C%E6%89%B9%E9%87%8F%E6%B3%A8%E5%86%8C%E6%93%8D%E4%BD%9C%E9%9C%80%E8%A6%81%E5%9C%A8root%E4%B8%8B%E6%89%A7%E8%A1%8C%EF%BC%89 "使用python脚本进行批量注册操作(需要在root下执行）")
- [备忘](http://thinknotes.cn/2024/03/02/running-node_exporter-as-a-service/#%E5%A4%87%E5%BF%98 "备忘")

</nav></div>## <span class="ez-toc-section" id="add_user_prometheus"></span>add user prometheus<span class="ez-toc-section-end"></span>

1. 在root下执行以下命令，添加新用户prometheus  
  useradd prometheus  
  passwd prometheus  
  su - prometheus  
  mkdir /home/prometheus/.ssh  
  cd /home/prometheus/.ssh;touch authorized\_keys
2. 将xshell存储的服务器中另外一个账号的的公钥内容写入authorized\_keys 
  - 在xshell中查看公钥内容的方法：xshell>工具>用户密钥管理者>选中需要的密钥>属性>公钥
3. 修改新建用户prometheus下目录权限： chmod 0700 ~/.ssh/;chmod 0600 ~/.ssh/authorized\_keys 
  - 如果不修改权限，可能会遇到报错：”所选的用户密钥未在远程主机上注册“，使用root账号查看/var/log/security,看到报错 Authentication refused: bad ownership or modes for file /home/prometheus/.ssh/authorized\_keys
4. 如果在ssh配置了限制登录的用户记得在/etc/ssh/sshd\_config中添加AllowUsers prometheus，然后重启ssh服务

## <span class="ez-toc-section" id="Running_Node_Exporter_as_a_Service"></span>Running Node Exporter as a Service<span class="ez-toc-section-end"></span>

1. To make it easy to start and stop Node Exporter, let us now convert it into a service.
2. Use vi or any other text editor to create a unit configuration file called node\_exporter.service.  
  sudo vi /etc/systemd/system/node\_exporter.service
3. This file should contain the path of the node\_exporter executable, and also specify which user should run the executable. Accordingly, add the following code:
  
  ```
  
  [Unit]
  Description=Node Exporter
  [Service]
  User=prometheus
  ExecStart=/home/prometheus/node_exporter/node_exporter
  [Install]
  WantedBy=default.target
  ```
4. Save the file and exit the text editor.
5. Reload systemd so that it reads the configuration file you just created.  
  sudo systemctl daemon-reload
6. At this point, Node Exporter is available as a service which can be managed using the systemctl command. Enable it so that it starts automatically at boot time.  
  sudo systemctl enable node\_exporter.service
7. You can now either reboot your server, or use the following command to start the service manually:  
  sudo systemctl start node\_exporter.service
8. 参考：<https://www.digitalocean.com/community/tutorials/how-to-use-prometheus-to-monitor-your-centos-7-server>

## <span class="ez-toc-section" id="%E6%89%B9%E9%87%8F%E6%B3%A8%E5%86%8CNode_Exporter%E4%B8%BA%E7%B3%BB%E7%BB%9F%E6%9C%8D%E5%8A%A1"></span>批量注册Node Exporter为系统服务<span class="ez-toc-section-end"></span>

5. 以上命令在root用户下操作
6. 准备文件node\_exporter.service，内容如下
  
  ```
  
  [Unit]
  Description=Node Exporter
  [Service]
  User=prometheus
  ExecStart=/home/prometheus/node_exporter/node_exporter
  [Install]
  WantedBy=default.target
  ```
7. cp node\_exporter.service /etc/systemd/system
8. 执行命令： systemctl daemon-reload
9. 执行命令 systemctl enable node\_exporter.service

## <span class="ez-toc-section" id="%E4%BD%BF%E7%94%A8python%E8%84%9A%E6%9C%AC%E8%BF%9B%E8%A1%8C%E6%89%B9%E9%87%8F%E6%B3%A8%E5%86%8C%E6%93%8D%E4%BD%9C%E9%9C%80%E8%A6%81%E5%9C%A8root%E4%B8%8B%E6%89%A7%E8%A1%8C%EF%BC%89"></span>使用python脚本进行批量注册操作(需要在root下执行）<span class="ez-toc-section-end"></span>

```
#!/usr/bin/env python

import os
import shutil
import subprocess
import sys
import getpass

def node(filename,to_dir):
    if not os.path.exists(filename):
        print(filename + 'is not exists')
        sys.exit(1)
    else:
        shutil.copy(filename,to_dir + '/' + os.path.basename(filename))

def register():
    cmd1 = 'systemctl daemon-reload'
    cmd2 = 'systemctl enable node_exporter.service'
    (status1,output1) = subprocess.getstatusoutput(cmd1)
    (status2,output2) = subprocess.getstatusoutput(cmd2)
    if status1:
        sys.stderr.write(output1)
        sys.exit(1)
    if status2:
        sys.stderr.write(output2)
        sys.exit(1)
def check(filename,to_dir,symlink):
    if os.path.exists(os.path.join(to_dir,os.path.basename(filename))) and os.path.exists(symlink):
        print("register node_exporter as a server success")
    else:
        print("register node_exporter as a server fail")

def main():
    filename = '/home/prometheus/node_exporter.service'
    to_dir = "/etc/systemd/system"
    symlink = '/etc/systemd/system/default.target.wants/node_exporter.service'
    node(filename,to_dir)
    register()
    check(filename,to_dir,symlink)

if __name__ == "__main__":
    if getpass.getuser() != 'root':
        print("the python shell need run as user 'root' ")
        sys.exit(1)
    else:
        main()
```

## <span class="ez-toc-section" id="%E5%A4%87%E5%BF%98"></span>备忘<span class="ez-toc-section-end"></span>

```
systemctl enable node_exporter.service
Created symlink from /etc/systemd/system/default.target.wants/node_exporter.service to /etc/systemd/system/node_exporter.service.
```