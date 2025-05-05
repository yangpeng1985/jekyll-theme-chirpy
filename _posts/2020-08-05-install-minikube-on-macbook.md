# 在MacBook Pro 上安装minikube

根据官方文档操作。

## 检查macos是否支持虚拟机技术。

> 若要检查您的 macOS 是否支持虚拟化技术，请运行下面的命令：
>
> ```
> sysctl -a | grep -E --color 'machdep.cpu.features|VMX'
> ```
>
> 如果你在输出结果中看到了 `VMX` （应该会高亮显示）的字眼，说明您的电脑已启用 VT-x 特性。
>

##  安装kubectl。

### 通过curl命令安装kubectl 可执行文件

> 1. 通过以下命令下载 kubectl 的最新版本：
>
>    ```
>    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/darwin/amd64/kubectl
>    ```
>
>    若需要下载特定版本的 kubectl，请将上述命令中的 `$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)` 部分替换成为需要下载的 kubectl 的具体版本即可。
>
>    例如，如果需要下载 v1.18.0 版本在 macOS 系统上,需要使用如下命令：
>
>    ```
>    curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.18.0/bin/darwin/amd64/kubectl
>    ```
>
> 2. 修改所下载的 kubectl 二进制文件为可执行模式。
>
>    ```
>    chmod +x ./kubectl
>    ```
>
> 3. 将 kubectl 可执行文件放置到你的 PATH 目录下。
>
>    ```shell
>    sudo mv ./kubectl /usr/local/bin/kubectl
>    ```
>

## 安装hypervisor

可选的hypervisor有HyperKit、VirutalBox、VMware Fusion，因为对VirutalBox比较熟悉，就选择安装了VirtualBox。

macOS安装VirtualBox时，会因为权限问题安装失败，在系统偏好设置中修改配置，允许安装oracle公司的软件，重新安装就可以安装成功了。

## 安装Minikube

> macOS安装Minikube最简单的方法是使用Homebrew：
> ```shell
> brew install minikube
>```
>

以下是安装过程，非常非常慢。

```
dreamPython@MacBook-Pro killedman.github.io % brew install minikube
==> Downloading https://homebrew.bintray.com/bottles/kubernetes-cli-1.18.4.catalina.bottle.tar.gz
==> Downloading from https://d29vzk4ow07wi7.cloudfront.net/d2c0b141991b8315eaa1891f4f0f2e485cffd11f3e58954b571839dee492
######################################################################## 100.0%
==> Downloading https://homebrew.bintray.com/bottles/minikube-1.11.0.catalina.bottle.tar.gz
==> Downloading from https://d29vzk4ow07wi7.cloudfront.net/458c030e48c868ac3499c8d321656c67ac4b10637d1549f9e2921c5eab4d
##########################################################                81.7%
curl: (18) transfer closed with 4786200 bytes remaining to read
Error: Failed to download resource "minikube"
Download failed: https://homebrew.bintray.com/bottles/minikube-1.11.0.catalina.bottle.tar.gz
dreamPython@MacBook-Pro killedman.github.io % brew install minikube
==> Downloading https://homebrew.bintray.com/bottles/kubernetes-cli-1.18.4.catalina.bottle.tar.gz
Already downloaded: /Users/dreamPython/Library/Caches/Homebrew/downloads/062addf2c80cd07fc72f7d1c41c904b104ea5e816cac0e6f04464e170429d67a--kubernetes-cli-1.18.4.catalina.bottle.tar.gz
==> Downloading https://homebrew.bintray.com/bottles/minikube-1.11.0.catalina.bottle.tar.gz
==> Downloading from https://d29vzk4ow07wi7.cloudfront.net/458c030e48c868ac3499c8d321656c67ac4b10637d1549f9e2921c5eab4d
######################################################################## 100.0%
==> Installing dependencies for minikube: kubernetes-cli
==> Installing minikube dependency: kubernetes-cli
==> Pouring kubernetes-cli-1.18.4.catalina.bottle.tar.gz
Error: The `brew link` step did not complete successfully
The formula built, but is not symlinked into /usr/local
Could not symlink bin/kubectl
Target /usr/local/bin/kubectl
already exists. You may want to remove it:
  rm '/usr/local/bin/kubectl'

To force the link and overwrite all conflicting files:
  brew link --overwrite kubernetes-cli

To list all files that would be deleted:
  brew link --overwrite --dry-run kubernetes-cli

Possible conflicting files are:
/usr/local/bin/kubectl -> /Applications/Docker.app/Contents/Resources/bin/kubectl
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/kubernetes-cli/1.18.4: 232 files, 49.2MB
==> Installing minikube
==> Pouring minikube-1.11.0.catalina.bottle.tar.gz
==> Caveats
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> Summary
🍺  /usr/local/Cellar/minikube/1.11.0: 8 files, 61.1MB
==> Caveats
==> kubernetes-cli
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
==> minikube
Bash completion has been installed to:
  /usr/local/etc/bash_completion.d

zsh completions have been installed to:
  /usr/local/share/zsh/site-functions
dreamPython@MacBook-Pro killedman.github.io %
```



## 安装确认

> 要确认 hypervisor 和 Minikube 均已成功安装，可以运行以下命令来启动本地 Kubernetes 集群
>
> 说明： 若要为 minikube start 设置 --vm-driver，在下面提到 <driver_name> 的地方，用小写字母输入你安装的 hypervisor 的名称。 指定 VM 驱动程序 列举了 --vm-driver 值的完整列表。
>
> 说明： 由于国内无法直接连接 k8s.gcr.io，推荐使用阿里云镜像仓库，在 minikube start 中添加 --image-repository 参数。
> ```shell
> minikube start --driver=virtualbox --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
> ```

```
dreamPython@MacBook-Pro killedman.github.io % minikube start --driver=virtualbox --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
😄  Darwin 10.15.5 上的 minikube v1.11.0
✨  根据用户配置使用 virtualbox 驱动程序
✅  正在使用镜像存储库 registry.cn-hangzhou.aliyuncs.com/google_containers
💿  正在下载 VM boot image...
    > minikube-v1.11.0.iso.sha256: 65 B / 65 B [-------------] 100.00% ? p/s 0s
    > minikube-v1.11.0.iso: 174.99 MiB / 174.99 MiB [ 100.00% 1.37 MiB p/s 2m8s
👍  Starting control plane node minikube in cluster minikube
🔥  Creating virtualbox VM (CPUs=2, Memory=4000MB, Disk=20000MB) ...
🐳  正在 Docker 19.03.8 中准备 Kubernetes v1.18.3…
    > kubelet.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubectl.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubeadm.sha256: 65 B / 65 B [--------------------------] 100.00% ? p/s 0s
    > kubectl: 41.99 MiB / 41.99 MiB [---------------] 100.00% 2.28 MiB p/s 19s
    > kubelet: 108.04 MiB / 108.04 MiB [-------------] 100.00% 4.67 MiB p/s 24s
    > kubeadm: 37.97 MiB / 37.97 MiB [---------------] 100.00% 1.44 MiB p/s 27s
🔎  Verifying Kubernetes components...
🌟  Enabled addons: default-storageclass, storage-provisioner
🏄  完成！kubectl 已经配置至 "minikube"

❗  /usr/local/bin/kubectl is version 1.16.6-beta.0, which may be incompatible with Kubernetes 1.18.3.
💡  You can also use 'minikube kubectl -- get pods' to invoke a matching version
dreamPython@MacBook-Pro killedman.github.io % minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```



## 启用minikube dashboard

```
dreamPython@MacBook-Pro killedman.github.io % minikube dashboard
🔌  正在开启 dashboard ...
🤔  正在验证 dashboard 运行情况 ...
🚀  Launching proxy ...
🤔  正在验证 proxy 运行状况 ...
🎉  Opening http://127.0.0.1:61161/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/ in your default browser...


按快捷键ctrl + C 终止 dashboard，执行minikube stop 停止minikube

^C
dreamPython@MacBook-Pro killedman.github.io % minikube stop
✋  Stopping "minikube" in virtualbox ...
🛑  Node "minikube" stopped.
```





## 参考：

1. https://www.cnblogs.com/moonlight-lin/p/13128702.html
2. https://www.jianshu.com/p/83170301adf8
3. https://gocn.vip/topics/9912
4. http://yangyingming.com/article/458/
5. https://segmentfault.com/a/1190000020394339
6. https://www.jianshu.com/p/48804c8bb250
7. https://kubernetes.io/zh/docs/tasks/tools/install-minikube/



