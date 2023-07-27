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



![image-20230723131232812](img/Kubernetes学习笔记/image-20230723131232812.png)







### 第六步：配置 Kubernetes

切换Kubernetes运行上下文至 docker-desktop：

```sh
kubectl config use-context docker-desktop
```

```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl config use-context docker-desktop
Switched to context "docker-desktop".
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```



验证 Kubernetes 集群状态：

```sh
kubectl cluster-info
```

```sh
kubectl get nodes
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl cluster-info
Kubernetes control plane is running at https://kubernetes.docker.internal:6443
CoreDNS is running at https://kubernetes.docker.internal:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```

```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl get nodes
NAME             STATUS   ROLES           AGE     VERSION
docker-desktop   Ready    control-plane   6m39s   v1.25.4
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```





### 第七步：部署 Kubernetes dashboard

命令：

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.5.1/aio/deploy/recommended.yaml
```

或者：

```sh
kubectl apply -f kubernetes-dashboard.yaml
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl apply -f kubernetes-dashboard.yaml
namespace/kubernetes-dashboard created
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```



kubernetes-dashboard.yaml内容如下：

```yaml
# Copyright 2017 The Kubernetes Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

apiVersion: v1
kind: Namespace
metadata:
  name: kubernetes-dashboard

---

apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-certs
  namespace: kubernetes-dashboard
type: Opaque

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-csrf
  namespace: kubernetes-dashboard
type: Opaque
data:
  csrf: ""

---

apiVersion: v1
kind: Secret
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-key-holder
  namespace: kubernetes-dashboard
type: Opaque

---

kind: ConfigMap
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard-settings
  namespace: kubernetes-dashboard

---

kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
rules:
  # Allow Dashboard to get, update and delete Dashboard exclusive secrets.
  - apiGroups: [""]
    resources: ["secrets"]
    resourceNames: ["kubernetes-dashboard-key-holder", "kubernetes-dashboard-certs", "kubernetes-dashboard-csrf"]
    verbs: ["get", "update", "delete"]
    # Allow Dashboard to get and update 'kubernetes-dashboard-settings' config map.
  - apiGroups: [""]
    resources: ["configmaps"]
    resourceNames: ["kubernetes-dashboard-settings"]
    verbs: ["get", "update"]
    # Allow Dashboard to get metrics.
  - apiGroups: [""]
    resources: ["services"]
    resourceNames: ["heapster", "dashboard-metrics-scraper"]
    verbs: ["proxy"]
  - apiGroups: [""]
    resources: ["services/proxy"]
    resourceNames: ["heapster", "http:heapster:", "https:heapster:", "dashboard-metrics-scraper", "http:dashboard-metrics-scraper"]
    verbs: ["get"]

---

kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
rules:
  # Allow Metrics Scraper to get metrics from the Metrics server
  - apiGroups: ["metrics.k8s.io"]
    resources: ["pods", "nodes"]
    verbs: ["get", "list", "watch"]

---

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kubernetes-dashboard
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kubernetes-dashboard
subjects:
  - kind: ServiceAccount
    name: kubernetes-dashboard
    namespace: kubernetes-dashboard

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: kubernetes-dashboard
          image: kubernetesui/dashboard:v2.5.1
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
            # Uncomment the following line to manually specify Kubernetes API server Host
            # If not specified, Dashboard will attempt to auto discover the API server and connect
            # to it. Uncomment only if the default does not work.
            # - --apiserver-host=http://my-address:port
          volumeMounts:
            - name: kubernetes-dashboard-certs
              mountPath: /certs
              # Create on-disk volume to store exec logs
            - mountPath: /tmp
              name: tmp-volume
          livenessProbe:
            httpGet:
              scheme: HTTPS
              path: /
              port: 8443
            initialDelaySeconds: 30
            timeoutSeconds: 30
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      volumes:
        - name: kubernetes-dashboard-certs
          secret:
            secretName: kubernetes-dashboard-certs
        - name: tmp-volume
          emptyDir: {}
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule

---

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  ports:
    - port: 8000
      targetPort: 8000
  selector:
    k8s-app: dashboard-metrics-scraper

---

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: dashboard-metrics-scraper
  name: dashboard-metrics-scraper
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: dashboard-metrics-scraper
  template:
    metadata:
      labels:
        k8s-app: dashboard-metrics-scraper
    spec:
      securityContext:
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: dashboard-metrics-scraper
          image: kubernetesui/metrics-scraper:v1.0.7
          ports:
            - containerPort: 8000
              protocol: TCP
          livenessProbe:
            httpGet:
              scheme: HTTP
              path: /
              port: 8000
            initialDelaySeconds: 30
            timeoutSeconds: 30
          volumeMounts:
          - mountPath: /tmp
            name: tmp-volume
          securityContext:
            allowPrivilegeEscalation: false
            readOnlyRootFilesystem: true
            runAsUser: 1001
            runAsGroup: 2001
      serviceAccountName: kubernetes-dashboard
      nodeSelector:
        "kubernetes.io/os": linux
      # Comment the following tolerations if Dashboard must not be deployed on master
      tolerations:
        - key: node-role.kubernetes.io/master
          effect: NoSchedule
      volumes:
        - name: tmp-volume
          emptyDir: {}
```







### 第八步：检查 kubernetes-dashboard 应用状态

命令：

```sh
kubectl get pod -n kubernetes-dashboard
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl get pod -n kubernetes-dashboard
NAME                                         READY   STATUS              RESTARTS   AGE
dashboard-metrics-scraper-748b4f5b9d-g4h26   0/1     ContainerCreating   0          111s
kubernetes-dashboard-68955f84f4-ccwfd        0/1     ContainerCreating   0          111s
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```





### 第九步：开启 API Server 访问代理

命令：

```sh
kubectl proxy
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl proxy
Starting to serve on 127.0.0.1:8001
```





### 第十步：访问Kubernetes Dashboard

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/





![image-20230723132319419](img/Kubernetes学习笔记/image-20230723132319419.png)







### 第十一步：配置控制台访问令牌

授权`kube-system`默认服务账号：

```sh
kubectl apply -f kube-system-default.yaml
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> cat .\kube-system-default.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-system-default
  labels:
    k8s-app: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: default
    namespace: kube-system

---

apiVersion: v1
kind: Secret
metadata:
  name: default
  namespace: kube-system
  labels:
    k8s-app: kube-system
  annotations:
    kubernetes.io/service-account.name: default
type: kubernetes.io/service-account-token
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl apply -f kube-system-default.yaml
clusterrolebinding.rbac.authorization.k8s.io/kube-system-default created
secret/default created
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```



输入以下命令：

```sh
$TOKEN=((kubectl -n kube-system describe secret default | Select-String "token:") -split " +")[1]
kubectl config set-credentials docker-desktop --token="${TOKEN}"
echo $TOKEN
```



输出token：

```sh
eyJhbGciOiJSUzI1NiIsImtpZCI6IkpYc3BrVTJtR2Q2VlVCd2FsMnlac0FFZ19XVWdqYUtBZU5WV296bkJRMTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImRlZmF1bHQiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiJkZTU2MDE0NC03ZDY1LTQ2YjktOWNiMy05NjBlMzEzOTYyM2YiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZS1zeXN0ZW06ZGVmYXVsdCJ9.RACu6JGOeW-wLAS15iPfZk-y8kQcXKbiKBGEtNP0JL3wRGAiBOKColNlDFlwhJ0UPtaD38hQOzA3SBQl1pxjDIdeRbc2GK0p_OSkPvsnwgCwGLADdXF4eDysLX-DLmmTBNM-rlo6vyoX_xtwvmBssswY5IHbkVT97wbhqXXkK-RCdvf31MHZBSIqI_MorrKGRtWabChy-3iRWfhNCb3egOQ6AsIUrptB9FSm_UFZoBbcLvPoztbBFfQEU-vJHy7iaQFZfMYoWVY0215HrWEJErdkT6nKjsrrwG0_a9NS0X_HUq1ERcxGsRxPICoQv7JsoSSm1NxrAiHuwQbW2oAT-w
```





粘贴到Kubernetes Dashboard

![image-20230723132836037](img/Kubernetes学习笔记/image-20230723132836037.png)



点击登录：

![image-20230723133104537](img/Kubernetes学习笔记/image-20230723133104537.png)









### 第十二步：安装 Ingress

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.2.0/deploy/static/provider/cloud/deploy.yaml
```

或

```shell
kubectl apply -f ingress-nginx-controller.yaml
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl apply -f ingress-nginx-controller.yaml
namespace/ingress-nginx created
serviceaccount/ingress-nginx created
serviceaccount/ingress-nginx-admission created
role.rbac.authorization.k8s.io/ingress-nginx created
role.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrole.rbac.authorization.k8s.io/ingress-nginx created
clusterrole.rbac.authorization.k8s.io/ingress-nginx-admission created
rolebinding.rbac.authorization.k8s.io/ingress-nginx created
rolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx created
clusterrolebinding.rbac.authorization.k8s.io/ingress-nginx-admission created
configmap/ingress-nginx-controller created
service/ingress-nginx-controller created
service/ingress-nginx-controller-admission created
deployment.apps/ingress-nginx-controller created
job.batch/ingress-nginx-admission-create created
job.batch/ingress-nginx-admission-patch created
ingressclass.networking.k8s.io/nginx created
validatingwebhookconfiguration.admissionregistration.k8s.io/ingress-nginx-admission created
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```



验证：

```sh
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
NAMESPACE       NAME                                        READY   STATUS              RESTARTS   AGE
ingress-nginx   ingress-nginx-admission-create-8b2gl        0/1     ImagePullBackOff    0          36s
ingress-nginx   ingress-nginx-admission-patch-pb7b9         0/1     ErrImagePull        0          36s
ingress-nginx   ingress-nginx-controller-5dcb895bcd-hh69q   0/1     ContainerCreating   0          36s
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```





### 第十三步：测试示例应用

部署测试应用：

```sh
kubectl create -f sample/apple.yaml
kubectl create -f sample/banana.yaml
kubectl create -f sample/ingress.yaml
```



```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl create -f sample/apple.yaml
pod/apple-app created
service/apple-service created
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl create -f sample/banana.yaml
pod/banana-app created
service/banana-service created
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl create -f sample/ingress.yaml
Error from server (InternalError): error when creating "sample/ingress.yaml": Internal error occurred: failed calling webhook "validate.nginx.ingress.kubernetes.io": failed to call webhook: Post "https://ingress-nginx-controller-admission.ingress-nginx.svc:443/networking/v1/ingresses?timeout=10s": dial tcp 10.99.181.133:443: connect: connection refused
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```

失败1个



测试示例应用：

```sh
curl -kL http://localhost/apple
apple
curl -kL http://localhost/banana
banana
```





### 第十四步：删除示例应用

```sh
kubectl delete -f sample/apple.yaml
kubectl delete -f sample/banana.yaml
kubectl delete -f sample/ingress.yaml
```





### 第十五步：删除 Ingress

```sh
kubectl delete -f ingress-nginx-controller.yaml
```

```sh
PS C:\Users\mao\Desktop\k8s-for-docker-desktop> kubectl delete -f ingress-nginx-controller.yaml
namespace "ingress-nginx" deleted
serviceaccount "ingress-nginx" deleted
serviceaccount "ingress-nginx-admission" deleted
role.rbac.authorization.k8s.io "ingress-nginx" deleted
role.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
clusterrole.rbac.authorization.k8s.io "ingress-nginx" deleted
clusterrole.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
rolebinding.rbac.authorization.k8s.io "ingress-nginx" deleted
rolebinding.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
clusterrolebinding.rbac.authorization.k8s.io "ingress-nginx" deleted
clusterrolebinding.rbac.authorization.k8s.io "ingress-nginx-admission" deleted
configmap "ingress-nginx-controller" deleted
service "ingress-nginx-controller" deleted
service "ingress-nginx-controller-admission" deleted
deployment.apps "ingress-nginx-controller" deleted
job.batch "ingress-nginx-admission-create" deleted
job.batch "ingress-nginx-admission-patch" deleted
ingressclass.networking.k8s.io "nginx" deleted
validatingwebhookconfiguration.admissionregistration.k8s.io "ingress-nginx-admission" deleted
PS C:\Users\mao\Desktop\k8s-for-docker-desktop>
```













## Linux安装

一个master节点，两个node节点，一共需要3台服务器

* master节点： k8s1
* node节点： k8s2
* node节点：k8s3



### 第一步：检查操作系统的版本

此方式下安装kubernetes集群要求Centos版本要在7.5或之上

```sh
cat /etc/redhat-release
```



```sh
[root@5f4b2177517c /]# cat /etc/redhat-release
CentOS Linux release 8.4.2105
[root@5f4b2177517c /]#
```





### 第二步：时间同步

启动chronyd服务：

```sh
systemctl start chronyd
```



设置chronyd服务开机自启：

```sh
systemctl enable chronyd
```



chronyd服务启动稍等几秒钟，就可以使用date命令验证时间了：

```sh
date
```





### 第三步：禁用iptables和firewalld服务

kubernetes和docker在运行中会产生大量的iptables规则，为了不让系统规则跟它们混淆，直接关闭系统的规则



关闭firewalld服务：

```sh
systemctl stop firewalld
```

```sh
systemctl disable firewalld
```



关闭iptables服务：

```sh
systemctl stop iptables
```

```sh
systemctl disable iptables
```





### 第四步：禁用selinux

 selinux是linux系统下的一个安全服务，如果不关闭它，在安装集群中会产生各种各样的奇葩问题

编辑 /etc/selinux/config 文件，修改SELINUX的值为disabled。注意修改完毕之后需要重启linux服务

设置：

SELINUX=disabled





### 第五步：禁用swap分区

swap分区指的是虚拟内存分区，它的作用是在物理内存使用完之后，将磁盘空间虚拟成内存来使用。启用swap设备会对系统的性能产生非常负面的影响，因此kubernetes要求每个节点都要禁用swap设备。但是如果因为某些原因确实不能关闭swap分区，就需要在集群安装过程中通过明确的参数进行配置说明



编辑分区配置文件/etc/fstab，注释掉swap分区一行。注意修改完毕之后需要重启linux服务。





### 第六步：修改linux的内核参数

修改linux的内核参数，添加网桥过滤和地址转发功能

编辑/etc/sysctl.d/kubernetes.conf文件，添加如下配置：

```
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```



重新加载配置：

```sh
sysctl -p
```



加载网桥过滤模块：

```sh
modprobe br_netfilter
```



查看网桥过滤模块是否加载成功：

```sh
lsmod | grep br_netfilter
```





### 第七步：配置ipvs功能

在kubernetes中service有两种代理模型，一种是基于iptables的，一种是基于ipvs的

ipvs的性能明显要高一些，但是如果要使用它，需要手动载入ipvs模块



安装ipset和ipvsadm(centos)：

```sh
yum install ipset ipvsadmin -y
```



添加需要加载的模块写入脚本文件：

```sh
cat <<EOF >  /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
```



为脚本文件添加执行权限：

```sh
chmod +x /etc/sysconfig/modules/ipvs.modules
```



执行脚本文件：

```sh
/bin/bash /etc/sysconfig/modules/ipvs.modules
```



查看对应的模块是否加载成功：

```sh
lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```





### 第八步：重启服务器

命令：

```sh
reboot
```





### 第九步：安装Docker

切换镜像源：

```sh
wget https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo -O /etc/yum.repos.d/docker-ce.repo
```



查看当前镜像源中支持的docker版本：

```sh
yum list docker-ce --showduplicates
```



```sh
[root@5f4b2177517c /]# yum list docker-ce --showduplicates
Failed to set locale, defaulting to C.UTF-8
Docker CE Stable - x86_64                                                                                        55 kB/s |  49 kB     00:00
Available Packages
docker-ce.x86_64                                                3:19.03.13-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:19.03.14-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:19.03.15-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.0-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.1-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.2-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.3-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.4-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.5-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.6-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.7-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.8-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.9-3.el8                                                 docker-ce-stable
docker-ce.x86_64                                                3:20.10.10-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.11-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.12-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.13-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.14-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.15-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.16-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.17-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.18-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.19-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.20-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.21-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.22-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.23-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:20.10.24-3.el8                                                docker-ce-stable
docker-ce.x86_64                                                3:23.0.0-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.1-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.2-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.3-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.4-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.5-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:23.0.6-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.0-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.1-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.2-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.3-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.4-1.el8                                                  docker-ce-stable
docker-ce.x86_64                                                3:24.0.5-1.el8                                                  docker-ce-stable
[root@5f4b2177517c /]#
```





安装特定版本的docker-ce：

```sh
yum install --setopt=obsoletes=0 docker-ce-20.10.24-3.el8 -y
```



```sh
[root@5f4b2177517c /]# yum install --setopt=obsoletes=0 docker-ce-20.10.24-3.el8 -y
Failed to set locale, defaulting to C.UTF-8
Last metadata expiration check: 0:01:44 ago on Tue Jul 25 02:37:00 2023.
Dependencies resolved.
================================================================================================================================================
 Package                                 Architecture      Version                                            Repository                   Size
================================================================================================================================================
Installing:
 docker-ce                               x86_64            3:20.10.24-3.el8                                   docker-ce-stable             21 M
Upgrading:
 iptables-libs                           x86_64            1.8.4-20.el8                                       baseos                      107 k
 python3-rpm                             x86_64            4.14.3-19.el8                                      baseos                      154 k
 rpm                                     x86_64            4.14.3-19.el8                                      baseos                      543 k
 rpm-build-libs                          x86_64            4.14.3-19.el8                                      baseos                      156 k
 rpm-libs                                x86_64            4.14.3-19.el8                                      baseos                      344 k
Installing dependencies:
 checkpolicy                             x86_64            2.9-1.el8                                          baseos                      348 k
 container-selinux                       noarch            2:2.167.0-1.module_el8.5.0+911+f19012f9            appstream                    54 k
 containerd.io                           x86_64            1.6.21-3.1.el8                                     docker-ce-stable             34 M
 diffutils                               x86_64            3.6-6.el8                                          baseos                      358 k
 docker-ce-cli                           x86_64            1:24.0.5-1.el8                                     docker-ce-stable            7.2 M
 docker-ce-rootless-extras               x86_64            24.0.5-1.el8                                       docker-ce-stable            4.9 M
 fuse-common                             x86_64            3.2.1-12.el8                                       baseos                       21 k
 fuse-overlayfs                          x86_64            1.7.1-1.module_el8.5.0+890+6b136101                appstream                    73 k
 fuse3                                   x86_64            3.2.1-12.el8                                       baseos                       50 k
 fuse3-libs                              x86_64            3.2.1-12.el8                                       baseos                       94 k
 iptables                                x86_64            1.8.4-20.el8                                       baseos                      585 k
 libcgroup                               x86_64            0.41-19.el8                                        baseos                       70 k
 libnetfilter_conntrack                  x86_64            1.0.6-5.el8                                        baseos                       65 k
 libnfnetlink                            x86_64            1.0.1-13.el8                                       baseos                       33 k
 libnftnl                                x86_64            1.1.5-4.el8                                        baseos                       83 k
 libselinux-utils                        x86_64            2.9-5.el8                                          baseos                      243 k
 libslirp                                x86_64            4.4.0-1.module_el8.5.0+890+6b136101                appstream                    70 k
 policycoreutils                         x86_64            2.9-16.el8                                         baseos                      373 k
 policycoreutils-python-utils            noarch            2.9-16.el8                                         baseos                      252 k
 python3-audit                           x86_64            3.0-0.17.20191104git1c2f876.el8                    baseos                       86 k
 python3-libselinux                      x86_64            2.9-5.el8                                          baseos                      283 k
 python3-libsemanage                     x86_64            2.9-6.el8                                          baseos                      127 k
 python3-policycoreutils                 noarch            2.9-16.el8                                         baseos                      2.2 M
 python3-setools                         x86_64            4.3.0-2.el8                                        baseos                      626 k
 rpm-plugin-selinux                      x86_64            4.14.3-19.el8                                      baseos                       77 k
 selinux-policy                          noarch            3.14.3-80.el8_5.2                                  baseos                      636 k
 selinux-policy-targeted                 noarch            3.14.3-80.el8_5.2                                  baseos                       15 M
 slirp4netns                             x86_64            1.1.8-1.module_el8.5.0+890+6b136101                appstream                    51 k
Installing weak dependencies:
 docker-buildx-plugin                    x86_64            0.11.2-1.el8                                       docker-ce-stable             13 M
 docker-compose-plugin                   x86_64            2.20.2-1.el8                                       docker-ce-stable             13 M
Enabling module streams:
 container-tools                                           rhel8

Transaction Summary
================================================================================================================================================
```







### 第十步：修改Docker配置

 Docker在默认情况下使用的Cgroup Driver为cgroupfs，而kubernetes推荐使用systemd来代替cgroupfs

添加一个配置文件：

```sh
mkdir /etc/docker
```



```sh
 cat <<EOF >  /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
EOF
```



```sh
[root@5f4b2177517c /]# cat /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://kn0t2bca.mirror.aliyuncs.com"]
}
[root@5f4b2177517c /]#
```





### 第十一步：启动Docker

```sh
systemctl restart docker
```

```sh
systemctl enable docker
```

或者：

```sh
service docker start
```

或者：

```sh
/etc/init.d/docker start
```







### 第十二步：安装kubernetes组件

编辑/etc/yum.repos.d/kubernetes.repo，添加下面的配置：

```sh
[kubernetes]	
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```



安装kubeadm、kubelet和kubectl：

```sh
yum install kubeadm -y
```

```sh
yum install kubelet -y
```

```sh
yum install kubectl -y
```



配置kubelet的cgroup：

编辑/etc/sysconfig/kubelet，添加下面的配置

```sh
KUBELET_CGROUP_ARGS="--cgroup-driver=systemd"
KUBE_PROXY_MODE="ipvs"
```



设置kubelet开机自启：

```sh
systemctl enable kubelet
```





### 第十三步：准备集群镜像

```sh
kubeadm config images list

images=(
    kube-apiserver:v1.17.4
    kube-controller-manager:v1.17.4
    kube-scheduler:v1.17.4
    kube-proxy:v1.17.4
    pause:3.1
    etcd:3.4.3-0
    coredns:1.6.5
)

for imageName in ${images[@]} ; do
	docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
	docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName 		k8s.gcr.io/$imageName
	docker rmi registry.cn-hangzhou.aliyuncs.com/google_containers/$imageName
done
```





### 第十四步：集群初始化

在`master`节点上执行

创建集群：

```sh
kubeadm init \
	--kubernetes-version=v1.17.4 \
    --pod-network-cidr=10.244.0.0/16 \
    --service-cidr=10.96.0.0/12 \
    --apiserver-advertise-address=192.168.109.100
```



```sh
mkdir -p $HOME/.kube
```

```sh
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```

```sh
chown $(id -u):$(id -g) $HOME/.kube/config
```



在`node`节点上执行

```sh
kubeadm join 192.168.109.100:6443 \ 
	--token 8507uc.o0knircuri8etnw2 \
	--discovery-token-ca-cert-hash \
	sha256:acc37967fb5b0acf39d7598f8a439cc7dc88f439a3f4d0c9cae88e7901b9d3f
```



查看状态：

```sh
kubectl get nodes
```

此时的集群状态为NotReady，这是因为还没有配置网络插件





### 第十五步：安装网络插件

kubernetes支持多种网络插件，比如flannel、calico、canal等等，本次选择flannel

在`master`节点执行：

```sh
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```



修改文件中quay.io仓库为quay-mirror.qiniu.com



使用配置文件启动fannel

```sh
kubectl apply -f kube-flannel.yml
```

























# 资源管理

## 概述

在kubernetes中，所有的内容都抽象为资源，用户需要通过操作资源来管理kubernetes

kubernetes的本质上就是一个集群系统，用户可以在集群中部署各种服务，所谓的部署服务，其实就是在kubernetes集群中运行一个个的容器，并将指定的程序跑在容器中

kubernetes的最小管理单元是pod而不是容器，所以只能将容器放在`Pod`中，而kubernetes一般也不会直接管理Pod，而是通过Pod控制器来管理Pod的

Pod可以提供服务之后，就要考虑如何访问Pod中服务，kubernetes提供了`Service`资源实现这个功能

如果Pod中程序的数据需要持久化，kubernetes还提供了各种`存储`系统



![image-20230725112152247](img/Kubernetes学习笔记/image-20230725112152247.png)







## 资源管理方式

* 命令式对象管理：直接使用命令去操作kubernetes资源
* 命令式对象配置：通过命令配置和配置文件去操作kubernetes资源
* 声明式对象配置：通过apply命令和配置文件去操作kubernetes资源



|      类型      | 操作对象 | 适用环境 |      优点      |               缺点               |
| :------------: | :------: | :------: | :------------: | :------------------------------: |
| 命令式对象管理 |   对象   |   测试   |      简单      | 只能操作活动对象，无法审计、跟踪 |
| 命令式对象配置 |   文件   |   开发   | 可以审计、跟踪 |  项目大时，配置文件多，操作麻烦  |
| 声明式对象配置 |   目录   |   开发   |  支持目录操作  |        意外情况下难以调试        |





### 命令式对象管理

示例命令：`kubectl run nginx-pod --image=nginx:1.17.1 --port=80`

命令语法：`kubectl [command] [type] [name] [flags]`



* **kubectl**：kubernetes集群的命令行工具，通过它能够对集群本身进行管理，并能够在集群上进行容器化应用的安装部署
* **comand**：指定要对资源执行的操作，例如create、get、delete
* **type**：指定资源类型，比如deployment、pod、service
* **name**：指定资源的名称，名称大小写敏感
* **flags**：指定额外的可选参数





kubernetes中所有的内容都抽象为资源，可以通过下面的命令进行查看：

```sh
kubectl api-resources
```



```sh
PS C:\Users\mao\Desktop> kubectl api-resources
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
pods                              po           v1                                     true         Pod
podtemplates                                   v1                                     true         PodTemplate
replicationcontrollers            rc           v1                                     true         ReplicationController
resourcequotas                    quota        v1                                     true         ResourceQuota
secrets                                        v1                                     true         Secret
serviceaccounts                   sa           v1                                     true         ServiceAccount
services                          svc          v1                                     true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
controllerrevisions                            apps/v1                                true         ControllerRevision
daemonsets                        ds           apps/v1                                true         DaemonSet
deployments                       deploy       apps/v1                                true         Deployment
replicasets                       rs           apps/v1                                true         ReplicaSet
statefulsets                      sts          apps/v1                                true         StatefulSet
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                               true         CronJob
jobs                                           batch/v1                               true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1                 true         Lease
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice
events                            ev           events.k8s.io/v1                       true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta2   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta2   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
PS C:\Users\mao\Desktop>
```





namespace / pod的创建和删除：

```sh
PS C:\Users\mao\Desktop> kubectl create namespace dev
namespace/dev created
PS C:\Users\mao\Desktop> kubectl get ns
NAME                   STATUS   AGE
default                Active   4d
dev                    Active   9s
kube-node-lease        Active   4d
kube-public            Active   4d
kube-system            Active   4d
kubernetes-dashboard   Active   4d
PS C:\Users\mao\Desktop> kubectl run pod --image=nginx -n dev
pod/pod created
PS C:\Users\mao\Desktop> kubectl get pod -n dev
NAME   READY   STATUS              RESTARTS   AGE
pod    0/1     ContainerCreating   0          12s
PS C:\Users\mao\Desktop> kubectl delete ns dev
namespace "dev" deleted
```







### 命令式对象配置

命令式对象配置就是使用命令配合配置文件一起来操作kubernetes资源

创建一个nginxpod.yaml：

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev

---

apiVersion: v1
kind: Pod
metadata:
  name: nginxpod
  namespace: dev
spec:
  containers:
  - name: nginx-containers
    image: nginx
```



执行create命令：

```sh
kubectl create -f nginxpod.yaml
```

```sh
PS C:\Users\mao\Desktop> kubectl create -f nginxpod.yaml
namespace/dev created
pod/nginxpod created
PS C:\Users\mao\Desktop>
```

```sh
PS C:\Users\mao\Desktop> kubectl get ns
NAME                   STATUS   AGE
default                Active   4d
dev                    Active   10s
kube-node-lease        Active   4d
kube-public            Active   4d
kube-system            Active   4d
kubernetes-dashboard   Active   4d
PS C:\Users\mao\Desktop> kubectl get pod -n dev
NAME       READY   STATUS    RESTARTS   AGE
nginxpod   1/1     Running   0          15s
PS C:\Users\mao\Desktop>
```



执行get命令：

```sh
PS C:\Users\mao\Desktop> kubectl get -f nginxpod.yaml
NAME            STATUS   AGE
namespace/dev   Active   78s

NAME           READY   STATUS    RESTARTS   AGE
pod/nginxpod   1/1     Running   0          78s
PS C:\Users\mao\Desktop>
```



执行delete命令：

```sh
kubectl delete -f nginxpod.yaml
```

```sh
PS C:\Users\mao\Desktop> kubectl delete -f nginxpod.yaml
namespace "dev" deleted
pod "nginxpod" deleted
PS C:\Users\mao\Desktop>
```

```,sh
PS C:\Users\mao\Desktop> kubectl get -f nginxpod.yaml
Error from server (NotFound): namespaces "dev" not found
Error from server (NotFound): namespaces "dev" not found
PS C:\Users\mao\Desktop> kubectl get pod -n dev
No resources found in dev namespace.
PS C:\Users\mao\Desktop> kubectl get ns
NAME                   STATUS   AGE
default                Active   4d
kube-node-lease        Active   4d
kube-public            Active   4d
kube-system            Active   4d
kubernetes-dashboard   Active   4d
PS C:\Users\mao\Desktop>
```





### 声明式对象配置

