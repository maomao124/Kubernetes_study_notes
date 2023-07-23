<h1 style="font-size:3.3em;color:skyblue;text-align:center">Kubernetes学习笔记</h1>







# 概述

kubernetes，是一个全新的基于容器技术的分布式架构领先方案。

kubernetes的本质是**一组服务器集群**，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化，主要提供了如下的主要功能：

- **自我修复**：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
- **弹性伸缩**：可以根据需要，自动对集群中正在运行的容器数量进行调整
- **服务发现**：服务可以通过自动发现的形式找到它所依赖的服务
- **负载均衡**：如果一个服务起动了多个容器，能够自动实现请求的负载均衡
- **版本回退**：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
- **存储编排**：可以根据容器自身的需求自动创建存储卷



![image-20230721133519710](img/Kubernetes学习笔记/image-20230721133519710.png)









# 容器编排问题

容器化部署方式给带来很多的便利，但是也会出现一些问题：

- 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
- 当并发访问量变大的时候，怎么样做到横向扩展容器数量



为了解决这些容器编排问题，就产生了一些容器编排的软件：

- **Swarm**：Docker自己的容器编排工具
- **Mesos**：Apache的一个资源统一管控的工具，需要和Marathon结合使用
- **Kubernetes**：Google开源的的容器编排工具





# 组件

一个kubernetes集群主要是由**控制节点(master)**、**工作节点(node)**构成，每个节点上都会安装不同的组件。

* **master节点**：集群的控制平面，负责集群的决策
  * **ApiServer** : 资源操作的唯一入口，接收用户输入的命令，提供认证、授权、API注册和发现等机制
  * **Scheduler** : 负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上
  * **ControllerManager** : 负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等
  * **Etcd **：负责存储集群中各种资源对象的信息
* **node节点**：集群的数据平面，负责为容器提供运行环境
  * **Kubelet** : 负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器
  * **KubeProxy** : 负责提供集群内部的服务发现和负载均衡
  * **Docker** : 负责节点上容器的各种操作



![image-20230721134107085](img/Kubernetes学习笔记/image-20230721134107085.png)





以部署一个nginx服务来说明kubernetes系统各个组件调用关系：

1. 首先要明确，一旦kubernetes环境启动之后，master和node都会将自身的信息存储到etcd数据库中

1. 一个nginx服务的安装请求会首先被发送到master节点的apiServer组件

2. apiServer组件会调用scheduler组件来决定到底应该把这个服务安装到哪个node节点上。在此时，它会从etcd中读取各个node节点的信息，然后按照一定的算法进行选择，并将结果告知apiServer

3. apiServer调用controller-manager去调度Node节点安装nginx服务

4. kubelet接收到指令后，会通知docker，然后由docker来启动一个nginx的pod。pod是kubernetes的最小操作单元，容器必须跑在pod中至此，

5. 一个nginx服务就运行了，如果需要访问nginx，就需要通过kube-proxy来对pod产生访问的代理。这样，外界用户就可以访问集群中的nginx服务了







# kubernetes概念

* **Master**：集群控制节点，每个集群需要至少一个master节点负责集群的管控
* **Node**：工作负载节点，由master分配容器到这些node工作节点上，然后node节点上的docker负责容器的运行
* **Pod**：kubernetes的最小控制单元，容器都是运行在pod中的，一个pod中可以有1个或者多个容器
* **Controller**：控制器，通过它来实现对pod的管理，比如启动pod、停止pod、伸缩pod的数量等等
* **Service**：pod对外服务的统一入口，下面可以维护者同一类的多个pod
* **Label**：标签，用于对pod进行分类，同一类pod会拥有相同的标签
* **NameSpace**：命名空间，用来隔离pod的运行环境



![image-20230721134601277](img/Kubernetes学习笔记/image-20230721134601277.png)

















# 集群环境搭建

## 集群类型

kubernetes集群大体上分为两类：

- 一主多从：一台Master节点和多台Node节点，搭建简单，但是有单机故障风险，适合用于测试环境
- 多主多从：多台Master节点和多台Node节点，搭建麻烦，安全性高，适合用于生产环境



![image-20230722135251688](img/Kubernetes学习笔记/image-20230722135251688.png)







## 部署方式

kubernetes有多种部署方式：

- minikube：一个用于快速搭建单节点kubernetes的工具
- kubeadm：一个用于快速搭建kubernetes集群的工具
- 二进制包 ：从官网下载每个组件的二进制包，依次去安装，此方式对于理解kubernetes组件更加有效







## Windows安装

### 第一步：安装桌面版docker

去Docker官网下载Docker Desktop

https://www.docker.com/



![image-20230722142845141](img/Kubernetes学习笔记/image-20230722142845141.png)



直接安装：https://desktop.docker.com/win/main/amd64/Docker%20Desktop%20Installer.exe?utm_source=docker&utm_medium=webreferral&utm_campaign=dd-smartbutton&utm_location=module





### 第二步：下载k8s-for-docker-desktop

目录：

```sh
git clone https://github.com/AliyunContainerService/k8s-for-docker-desktop.git
```



```sh
PS C:\Users\mao\Desktop> git clone https://github.com/AliyunContainerService/k8s-for-docker-desktop.git
Cloning into 'k8s-for-docker-desktop'...
remote: Enumerating objects: 602, done.
remote: Counting objects: 100% (71/71), done.
remote: Compressing objects: 100% (50/50), done.

Receiving objects: 100% (602/602), 2.63 MiB | 25.00 KiB/s, done.
Resolving deltas: 100% (360/360), done.
PS C:\Users\mao\Desktop>
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> ls


    目录: C:\Users\mao\Desktop\k8s-for-docker-desktop


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         2023/7/22     14:32                images
d-----         2023/7/22     14:32                sample
-a----         2023/7/22     14:32              9 .gitignore
-a----         2023/7/22     14:32            837 getLatestIstio.ps1
-a----         2023/7/22     14:32           1035 images.properties
-a----         2023/7/22     14:32          15570 ingress-nginx-controller.yaml
-a----         2023/7/22     14:32            406 k8s命令
-a----         2023/7/22     14:32            560 kube-system-default.yaml
-a----         2023/7/22     14:32           7933 kubernetes-dashboard.yaml
-a----         2023/7/22     14:32            245 load_images.ps1
-a----         2023/7/22     14:32            305 load_images.sh
-a----         2023/7/22     14:32          15404 README.md
-a----         2023/7/22     14:32          11081 README_en.md


PS C:\Users\mao\Desktop\k8s-for-docker-desktop> cd .\sample\
PS C:\Users\mao\Desktop\k8s-for-docker-desktop\sample> ls


    目录: C:\Users\mao\Desktop\k8s-for-docker-desktop\sample


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----         2023/7/22     14:32            364 apple.yaml
-a----         2023/7/22     14:32            370 banana.yaml
-a----         2023/7/22     14:32            589 ingress.yaml


PS C:\Users\mao\Desktop\k8s-for-docker-desktop\sample>
```





### 第三步：安装k8s所需的镜像

* 如Kubernetes版本为 v1.27.2, 请使用下面命令切换 [v1.27.2 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.27.2) ```git checkout v1.27.2```
* 如Kubernetes版本为 v1.25.9, 请使用下面命令切换 [v1.25.9 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.25.9) ```git checkout v1.25.9```
* 如Kubernetes版本为 v1.25.4, 请使用下面命令切换 [v1.25.4 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.25.4) ```git checkout v1.25.4```
* 如Kubernetes版本为 v1.25.2, 请使用下面命令切换 [v1.25.2 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.25.2) ```git checkout v1.25.2```
* 如Kubernetes版本为 v1.25.0, 请使用下面命令切换 [v1.25.0 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.25.0) ```git checkout v1.25.0```
* 如Kubernetes版本为 v1.24.2, 请使用下面命令切换 [v1.24.2 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.24.2) ```git checkout v1.24.2```
* 如Kubernetes版本为 v1.24.0, 请使用下面命令切换 [v1.24.0 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.24.0) ```git checkout v1.24.0```
* 如Kubernetes版本为 v1.23.4, 请使用下面命令切换 [v1.23.4 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.23.4) ```git checkout v1.23.4```
* 如Kubernetes版本为 v1.22.5, 请使用下面命令切换 [v1.22.5 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.22.5) ```git checkout v1.22.5```
* 如Kubernetes版本为 v1.22.4, 请使用下面命令切换 [v1.22.4 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.22.4) ```git checkout v1.22.4```
* 如Kubernetes版本为 v1.21.5, 请使用下面命令切换 [v1.21.5 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.21.5) ```git checkout v1.21.5```
* 如Kubernetes版本为 v1.21.4, 请使用下面命令切换 [v1.21.4 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.21.4) ```git checkout v1.21.4```
* 如Kubernetes版本为 v1.21.3, 请使用下面命令切换 [v1.21.3 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.21.3) ```git checkout v1.21.3```
* 如Kubernetes版本为 v1.21.2, 请使用下面命令切换 [v1.21.2 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.21.2) ```git checkout v1.21.2```
* 如Kubernetes版本为 v1.21.1, 请使用下面命令切换 [v1.21.1 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.21.1) ```git checkout v1.21.1```
* 如Kubernetes版本为 v1.19.3, 请使用下面命令切换 [v1.19.3 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.19.3) ```git checkout v1.19.3```
* 如Kubernetes版本为 v1.19.2, 请使用下面命令切换 [v1.19.2 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.19.2) ```git checkout v1.19.2```
* 如Kubernetes版本为 v1.18.8, 请使用下面命令切换 [v1.18.8 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.18.8) ```git checkout v1.18.8```
* 如Kubernetes版本为 v1.18.6, 请使用下面命令切换 [v1.18.6 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.18.6) ```git checkout v1.18.6```
* 如Kubernetes版本为 v1.18.3, 请使用下面命令切换 [v1.18.3 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.18.3) ```git checkout v1.18.3```
* 如Kubernetes版本为 v1.16.5, 请使用下面命令切换 [v1.16.5 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.16.5) ```git checkout v1.16.5```
* 如Kubernetes版本为 v1.15.5, 请使用下面命令切换 [v1.15.5 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.15.5) ```git checkout v1.15.5```
* 如Kubernetes版本为 v1.15.4, 请使用下面命令切换 [v1.15.4 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.15.4) ```git checkout v1.15.4```
* 如Kubernetes版本为 v1.14.8, 请使用下面命令切换 [v1.14.8 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.14.8) ```git checkout v1.14.8```
* 如Kubernetes版本为 v1.14.7, 请使用下面命令切换 [v1.14.7 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.14.7) ```git checkout v1.14.7```
* 如Kubernetes版本为 v1.14.6, 请使用下面命令切换 [v1.14.6 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.14.6) ```git checkout v1.14.6```
* 如Kubernetes版本为 v1.14.3, 请使用下面命令切换 [v1.14.3 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.14.3) ```git checkout v1.14.3```
* 如Kubernetes版本为 v1.14.1, 请使用下面命令切换 [v1.14.1 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.14.1) ```git checkout v1.14.1```
* 如Kubernetes版本为 v1.13.0, 请使用下面命令切换 [v1.13.0 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.13.0) ```git checkout v1.13.0```
* 如Kubernetes版本为 v1.10.11, 请使用下面命令切换 [v1.10.11 分支](https://github.com/AliyunContainerService/k8s-for-docker-desktop/tree/v1.10.11) ```git checkout v1.10.11```



查看版本：

![image-20230722210938602](img/Kubernetes学习笔记/image-20230722210938602.png)





切换分支命令：

```sh
git checkout v1.25.4
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> git checkout v1.25.4
Switched to a new branch 'v1.25.4'
branch 'v1.25.4' set up to track 'origin/v1.25.4'.
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```





### 第四步：下载 Kubernetes 所需要的镜像

在 Mac 上执行如下脚本

```bash
./load_images.sh
```



在Windows上，使用 PowerShell

```powershell
 .\load_images.ps1
```



说明: 

* 如果因为安全策略无法执行 PowerShell 脚本，请在 “以管理员身份运行” 的 PowerShell 中执行 ```Set-ExecutionPolicy RemoteSigned``` 命令。 
* 如果需要，可以通过修改 ```images.properties``` 文件自行加载你自己需要的镜像



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> cat .\load_images.ps1
foreach($line in Get-Content .\images.properties) {
    $data = $line.Split('=')
    $key = $data[0];
    $value = $data[1];
    Write-Output "$key=$value"
    docker pull ${value}
    docker tag ${value} ${key}
    docker rmi ${value}
}
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> cat .\images.properties
registry.k8s.io/pause:3.8=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8
registry.k8s.io/kube-controller-manager:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.4
registry.k8s.io/kube-scheduler:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.4
registry.k8s.io/kube-proxy:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.4
registry.k8s.io/kube-apiserver:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.4
registry.k8s.io/etcd:3.5.5-0=registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
registry.k8s.io/coredns/coredns:v1.9.3=registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3
registry.k8s.io/ingress-nginx/controller:v1.6.4=registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.6.4
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.2=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```

```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> .\load_images.ps1
registry.k8s.io/pause:3.8=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8
3.8: Pulling from google_containers/pause
Digest: sha256:9001185023633d17a2f98ff69b6ff2615b8ea02a825adffa40422f51dfdcde9d
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8
registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.8
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/pause@sha256:9001185023633d17a2f98ff69b6ff2615b8ea02a825adffa40422f51dfdcde9d
registry.k8s.io/kube-controller-manager:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.4
v1.25.4: Pulling from google_containers/kube-controller-manager
Digest: sha256:2526315b1c01899eab8b0fb81046083e4571d94433b293f9db124d091df98707
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.4
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager@sha256:2526315b1c01899eab8b0fb81046083e4571d94433b293f9db124d091df98707
registry.k8s.io/kube-scheduler:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.4
v1.25.4: Pulling from google_containers/kube-scheduler
Digest: sha256:840d5b9fc29f4cddef60d832f410e3979dde2b8224cdb76dce0784394c0366a0
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.4
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler@sha256:840d5b9fc29f4cddef60d832f410e3979dde2b8224cdb76dce0784394c0366a0
registry.k8s.io/kube-proxy:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.4
v1.25.4: Pulling from google_containers/kube-proxy
Digest: sha256:1df694ba49eb1263a84c6cb32dd143d09b3e0b6cb0d48fddb3424cc4afe22e49
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.4
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy@sha256:1df694ba49eb1263a84c6cb32dd143d09b3e0b6cb0d48fddb3424cc4afe22e49
registry.k8s.io/kube-apiserver:v1.25.4=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.4
v1.25.4: Pulling from google_containers/kube-apiserver
Digest: sha256:ba9fc1737c5b7857f3e19183d1504ec58df0c50d970e0c008e58e8a13dc11422
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.4
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver:v1.25.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver@sha256:ba9fc1737c5b7857f3e19183d1504ec58df0c50d970e0c008e58e8a13dc11422
registry.k8s.io/etcd:3.5.5-0=registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
3.5.5-0: Pulling from google_containers/etcd
1cd0595314a5: Already exists
b96bf43797c7: Already exists
4ef1d19f8f72: Pull complete
b2d88b38a934: Pull complete
7d5a4e7fdcb1: Pull complete
Digest: sha256:b83c1d70989e1fe87583607bf5aee1ee34e52773d4755b95f5cf5a451962f3a4
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/etcd:3.5.5-0
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/etcd@sha256:b83c1d70989e1fe87583607bf5aee1ee34e52773d4755b95f5cf5a451962f3a4
registry.k8s.io/coredns/coredns:v1.9.3=registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3
v1.9.3: Pulling from google_containers/coredns
d92bdee79785: Pull complete
f2401d57212f: Pull complete
Digest: sha256:8e352a029d304ca7431c6507b56800636c321cb52289686a581ab70aaa8a2e2a
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/coredns:v1.9.3
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/coredns@sha256:8e352a029d304ca7431c6507b56800636c321cb52289686a581ab70aaa8a2e2a
registry.k8s.io/ingress-nginx/controller:v1.6.4=registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.6.4
v1.6.4: Pulling from google_containers/nginx-ingress-controller
c158987b0551: Pull complete
2837aab8bb3a: Pull complete
9147e4e88744: Pull complete
5d640e521f0d: Pull complete
88a50e9c640c: Pull complete
4f4fb700ef54: Pull complete
21f50dca696b: Pull complete
be597eeae8e8: Pull complete
12f7c0fc6528: Pull complete
815588d18cc1: Pull complete
b03b28efc8a8: Pull complete
74320839d336: Pull complete
40913e9add4e: Pull complete
d2af2abae592: Pull complete
Digest: sha256:5a49c2996074c1d3a063bde162e6bf62633bfad8dbd1d6c83f876db0d3c35428
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.6.4
registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.6.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller:v1.6.4
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/nginx-ingress-controller@sha256:5a49c2996074c1d3a063bde162e6bf62633bfad8dbd1d6c83f876db0d3c35428
registry.k8s.io/ingress-nginx/kube-webhook-certgen:v1.5.2=registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2
v1.5.2: Pulling from google_containers/kube-webhook-certgen
5dea5ec2316d: Pull complete
30e7849a636a: Pull complete
Digest: sha256:2217d8028431f82afef26b0023ff8b9247971be3964a22e80ef5ca2f36e1a293
Status: Downloaded newer image for registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen:v1.5.2
Untagged: registry.cn-hangzhou.aliyuncs.com/google_containers/kube-webhook-certgen@sha256:2217d8028431f82afef26b0023ff8b9247971be3964a22e80ef5ca2f36e1a293
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```





> 如果在Kubernetes部署的过程中出现问题，可以在 C:\ProgramData\DockerDesktop下的service.txt 查看Docker日志, 在 C:\Users\yourUserName\AppData\Local\Docker下的log.txt 查看Kubernetes日志 

> 如果看到 Kubernetes一直在启动状态，请参考 
>
> * [Issue 3769(comment)](https://github.com/docker/for-win/issues/3769#issuecomment-486046718) 或 [Issue 3649(comment)](https://github.com/docker/for-mac/issues/3649#issuecomment-497441158)
>   * 在macOS上面，执行 ```rm -fr '~/Library/Group\ Containers/group.com.docker/pki'```
>   * 在Windows上面删除 'C:\ProgramData\DockerDesktop\pki' 目录 和 'C:\Users\yourUserName\AppData\Local\Docker\pki' 目录
> * [Issue 1962(comment)](





### 第五步：开启 Kubernetes

此时应该有多个镜像：

```sh
PS C:\Users\mao\Desktop> docker images
REPOSITORY                                           TAG       IMAGE ID       CREATED         SIZE
nginx                                                latest    6efc10a0510f   3 months ago    142MB
registry.k8s.io/ingress-nginx/controller             v1.6.4    7744eedd958f   5 months ago    282MB
registry.k8s.io/kube-apiserver                       v1.25.4   00631e54acba   8 months ago    128MB
registry.k8s.io/kube-proxy                           v1.25.4   2c2bc1864279   8 months ago    61.7MB
registry.k8s.io/kube-scheduler                       v1.25.4   e2d17ec744c1   8 months ago    50.6MB
registry.k8s.io/kube-controller-manager              v1.25.4   8f59f6dfaed6   8 months ago    117MB
registry.k8s.io/etcd                                 3.5.5-0   4694d02f8e61   10 months ago   300MB
registry.k8s.io/pause                                3.8       4873874c08ef   13 months ago   711kB
registry.k8s.io/coredns/coredns                      v1.9.3    5185b96f0bec   14 months ago   48.8MB
redislabs/rebloom                                    latest    66d626dc1387   20 months ago   147MB
centos                                               latest    5d0da3dc9764   22 months ago   231MB
registry.k8s.io/ingress-nginx/kube-webhook-certgen   v1.5.2    a573628e4199   2 years ago     29.7MB
PS C:\Users\mao\Desktop>
```





进入Docker Desktop，点击设置

开启k8s，等待启动

![image-20230722213129734](img/Kubernetes学习笔记/image-20230722213129734.png)





