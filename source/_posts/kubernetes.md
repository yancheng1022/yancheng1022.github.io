---
title: kubernetes
date: 2024/09/1
categories:
  - coding
tags:
  - 运维
  - k8s
  - kubernetes
abbrlink: 17997
---

# 1、kubernetes基础

## 1.1、简介

Kubernetes 是一个开源的容器编排引擎和容器集群管理工具，用来对容器化应用进行自动化部署、 扩缩和管理。
Kubernetes 这个名字源于希腊语，意为“舵手”或“飞行员”。k8s 这个缩写是因为 k 和 s 之间有8个字符。 Google 在 2014 年开源了 Kubernetes 项目

>从V1.24开始，kubernetes默认容器运行使用containerd，不再使用docker

## 1.2、云原生

Kubernetes 是 CNCF 托管的第一个开源项目。因此现在提到云原生，往往我们都把它与kubernetes联系起来。
使用Java、Go、PHP、Python等语言开发的应用我们称之为原生应用，在设计和开发这些应用时，使他们能够运行在云基础设施(或kubernetes)上，从而使应用具备可弹性扩展的能力，我们称之为云原生应用。我们可以将云原生理解为以容器技术为载体、基于微服务架构思想的一套技术体系和方法论

## 1.3、kubernetes架构

一个Kubernetes集群至少包含一个控制平面(control plane)，以及一个或多个工作节点(worker node)。

**控制平面(Control Plane) :** 控制平面负责管理工作节点和维护集群状态。所有任务分配都来自于控制平面。
**工作节点(Worker Node) :** 工作节点负责执行由控制平面分配的请求任务,运行实际的应用和工作负载。

### 1.3.1、控制平面

**kube-apiserver**
如果需要与Kubernetes 集群进行交互，就要通过 API。
`apiserver`是 Kubernetes 控制平面的前端，用于处理内部和外部请求。

**kube-scheduler**
集群状况是否良好？如果需要创建新的容器，要将它们放在哪里？这些是调度程序需要关注的问题。
`scheduler`调度程序会考虑容器集的资源需求（例如 CPU 或内存）以及集群的运行状况。随后，它会将容器集安排到适当的计算节点。

**etcd**
etcd是一个键值对数据库，用于存储配置数据和集群状态信息。

**kube-controller-manager**
控制器负责实际运行集群，`controller-manager`控制器管理器则是将多个控制器功能合而为一，降低了程序的复杂性。
`controller-manager`包含了这些控制器：
- 节点控制器（Node Controller）：负责在节点出现故障时进行通知和响应
- 任务控制器（Job Controller）：监测代表一次性任务的 Job 对象，然后创建 Pods 来运行这些任务直至完成
- 端点控制器（Endpoints Controller）：填充端点（Endpoints）对象（即加入 Service 与 Pod）
- 服务帐户和令牌控制器（Service Account & Token Controllers）：为新的命名空间创建默认帐户和 API 访问令牌

### 1.3.2、Node组件

节点组件会在每个节点上运行，负责维护运行的 Pod 并提供 Kubernetes 运行环境。

**kubelet**
kubelet 会在集群中每个节点（node）上运行。 它保证容器（containers）都运行在 Pod 中。
当控制平面需要在节点中执行某个操作时，kubelet 就会执行该操作。

**kube-proxy**
kube-proxy 是集群中每个节点（node）上运行的网络代理，是实现 Kubernetes 服务（Service） 概念的一部分。kube-proxy 维护节点网络规则和转发流量，实现从集群内部或外部的网络与 Pod 进行网络通信。

**容器运行时（Container Runtime）**
容器运行环境是负责运行容器的软件。Kubernetes 支持许多容器运行环境，例如 containerd、docker或者其他实现了 Kubernetes CRI (容器运行环境接口)的容器。

### 1.3.3、组件关系

![](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240520105058.png)

**cloud-controller-manager**
控制平面还包含一个可选组件cloud-controller-manager。
云控制器管理器（Cloud Controller Manager）允许你将你的集群连接到云提供商的 API 之上， 并将与该云平台交互的组件同与你的集群交互的组件分离开来。
如果在自己的环境中运行 Kubernetes，或者在本地计算机中运行学习环境， 所部署的集群不需要有云控制器管理器。

## 1.4、安装Minikube

Minikube是一个可以本地运行的单机版kubernetes,方便我们学习kubernetes和调试程序。

### 1.4.1、windows下安装

1、下载Minikube：https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe

2、启动 minikube
minikube start --image-mirror-country='cn' --container-runtime=containerd

># 设置使用国内阿里云镜像源
>--image-mirror-country='cn' 
># Minikube默认安装kubernetes v1.25.0，需要将容器运行时设置为containerd运行v1.24 .0及之后版本，都需要此设置。如果不设置会出现错误
--container-runtime=containerd


### 1.4.2、Kubectl

kubectl 是一个kubernetes命令行工具。
kubectl 使用 Kubernetes API 与 Kubernetes 集群的控制面进行通信，可以使用 kubectl 部署应用程序、检查和管理群集资源以及查看日志。

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240520111114.png)

```shell
# 查看节点状态
kubectl get node
kubectl get pod -A
# 进入容器
minikube ssh
# 查看kubelet状态
systemctl status kubelet
```

### 1.4.3、安装Dashboard

minikube dashboard --url --port=63373
不设置--port端口是随机的，每次启动可能会变化。

在浏览器访问（命令行退出后无法访问）：
http://127.0.0.1:63373/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/


## 1.5、使用K3s快速搭建集群

K3s 是一个轻量级的、完全兼容的 Kubernetes 发行版本。非常适合初学者。
K3s将所有 Kubernetes 控制平面组件都封装在单个二进制文件和进程中，文件大小<100M,占用资源更小，且包含了kubernetes运行所需要的部分外部依赖和本地存储提供程序。
K3s提供了离线安装包，安装起来非常方便，可以避免安装过程中遇到各种网络资源访问问题。
K3s特别适用于边缘计算、物联网、嵌入式和ARM移动端场景

### 1.5.1、离线安装k3s集群

K3s集群分为k3s Server(控制平面)和k3s Agent(工作节点)。所有的组件都打包在单个二进制文件中

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240520112800.png)

**运行环境**

- 最低运行要求：内存: 512MB / CPU: 1 核心
- K3s版本：v1.25.0+k3s1

1、准备工作
```shell
# 关闭防火墙，设置selinux（需要联网）
systemctl disable firewalld --now
yum install -y container-selinux selinux-policy-base
yum install -y https://rpm.rancher.io/k3s/latest/common/centos/7/noarch/k3s-selinux-0.2-1.el7_8.noarch.rpm
```

2、下载安装包

下载安装脚本install.sh：https://get.k3s.io/
下载k3s二进制文件：k3s
下载必要的image：离线安装需要的image文件

>这些文件都可以在github仓库中获取：https://github.com/k3s-io/k3s

3、执行安装脚本 

```shell
# 将k3s二进制文件移动到`/usr/local/bin`目录，并添加执行权限
mv k3s /usr/local/bin
chmod +x /usr/local/bin/k3s
# 将镜像移动到/var/lib/rancher/k3s/agent/images/目录（无需解压）
mkdir -p /var/lib/rancher/k3s/agent/images/
cp ./k3s-airgap-images-amd64.tar.gz /var/lib/rancher/k3s/agent/images/
# 在k8s-master节点执行
# -------------
#修改权限
chmod +x install.sh
#离线安装
INSTALL_K3S_SKIP_DOWNLOAD=true ./install.sh
#安装完成后，查看节点状态
kubectl get node
#查看token
cat /var/lib/rancher/k3s/server/node-token
# --------------
# 在k8s-worker1和k8s-worker2节点执行
INSTALL_K3S_SKIP_DOWNLOAD=true \
K3S_URL=https://117.72.75.47:6443 \
K3S_TOKEN=K1067df23d36545a47ad93191b0a902c426822e45c0d269e8b1cad2ac29dfbf9d90::server:da58fdc7ec6353010c580b70279ae367 \
./install.sh
```

## 1.6、pord容器集

Pod 是包含一个或多个容器的容器组，是 Kubernetes 中创建和管理的最小对象。

Pod 有以下特点：
Pod是kubernetes中最小的调度单位（原子单元），Kubernetes直接管理Pod而不是容器。
同一个Pod中的容器总是会被自动安排到集群中的同一节点（物理机或虚拟机）上，并且一起调度。
Pod可以理解为运行特定应用的“逻辑主机”，这些容器共享存储、网络和配置声明(如资源限制)。
每个 Pod 有唯一的 IP 地址。 IP地址分配给Pod，在同一个 Pod 内，所有容器共享一个 IP 地址和端口空间，Pod 内的容器可以使用localhost互相通信。

### 1.6.1、创建和管理pod

```shell
kubectl run mynginx --image=nginx
# 查看Pod
kubectl get pod
# 描述
kubectl describe pod mynginx
# 查看Pod的运行日志
kubectl logs mynginx

# 显示pod的IP和运行节点信息
kubectl get pod -owide
# 使用Pod的ip+pod里面运行容器的端口
curl 10.42.1.3

#在容器中执行
kubectl exec mynginx -it -- /bin/bash

kubectl get po --watch
# -it 交互模式 
# --rm 退出后删除容器，多用于执行一次性任务或使用客户端
kubectl run mynginx --image=nginx -it --rm -- /bin/bash 

# 删除
kubectl delete pod mynginx
# 强制删除
kubectl delete pod mynginx --force
```

## 1.7、Deployment(部署)与ReplicaSet(副本集)

#### 1.7.1、概念和基本操作

**Deployment**是对ReplicaSet和Pod更高级的抽象。它使Pod拥有多副本，自愈，扩缩容、滚动升级等能力。

**ReplicaSet**(副本集)是一个Pod的集合。

它可以设置运行Pod的数量，确保任何时间都有指定数量的 Pod 副本在运行。通常我们不直接使用ReplicaSet，而是在Deployment中声明。

```shell
#创建deployment,部署3个运行nginx的Pod
kubectl create deployment nginx-deployment --image=nginx:1.22 --replicas=3
#查看deployment
kubectl get deploy
#查看replicaSet
kubectl get rs 
#删除deployment
kubectl delete deploy nginx-deployment
```

### 1.7.2、缩放

1、手动缩放

```shell
#将副本数量调整为5
kubectl scale deployment/nginx-deployment --replicas=5
kubectl get deploy
```

2、自动缩放
自动缩放通过增加和减少副本的数量，以保持所有 Pod 的平均 CPU 利用率不超过 75%。
自动伸缩需要声明Pod的资源限制，同时使用 Metrics Server 服务（K3s默认已安装）。


```shell
#自动缩放
kubectl autoscale deployment/nginx-auto --min=3 --max=10 --cpu-percent=75 
#查看自动缩放
kubectl get hpa
#删除自动缩放
kubectl delete hpa nginx-deployment
```

### 1.7.3、滚动更新

```shell
#查看版本和Pod
kubectl get deployment/nginx-deployment -owide
kubectl get pods

#更新容器镜像
kubectl set image deployment/nginx-deployment nginx=nginx:1.23
#滚动更新
kubectl rollout status deployment/nginx-deployment
#查看过程
kubectl get rs --watch
```

### 1.7.4、版本回滚

```shell
#查看历史版本
kubectl rollout history deployment/nginx-deployment
#查看指定版本的信息
kubectl rollout history deployment/nginx-deployment --revision=2
#回滚到历史版本
kubectl rollout undo deployment/nginx-deployment --to-revision=2
```

## 1.8、service（服务）

Service将运行在一组 Pods 上的应用程序公开为网络服务的抽象方法。Service为一组 Pod 提供相同的 DNS 名，并且在它们之间进行负载均衡。Kubernetes 为 Pod 提供分配了IP 地址，但IP地址可能会发生变化。集群内的容器可以通过service名称访问服务，而不需要担心Pod的IP发生变化

Kubernetes Service 定义了这样一种抽象：
逻辑上的一组可以互相替换的 Pod，通常称为微服务。 Service 对应的 Pod 集合通常是通过选择算符来确定的。 
举个例子，在一个Service中运行了3个nginx的副本。这些副本是可互换的，我们不需要关心它们调用了哪个nginx，也不需要关注 Pod的运行状态，只需要调用这个服务就可以了

### 1.8.1、创建service对象

ClusterIP：将服务公开在集群内部。kubernetes会给服务分配一个集群内部的 IP，集群内的所有主机都可以通过这个Cluster-IP访问服务。集群内部的Pod可以通过service名称访问服务。

NodePort：通过每个节点的主机IP 和静态端口（NodePort）暴露服务。 集群的外部主机可以使用节点IP和NodePort访问服务。

ExternalName：将集群外部的网络引入集群内部。

LoadBalancer：使用云提供商的负载均衡器向外部暴露服务。

```shell
# port是service访问端口,target-port是Pod端口
# 二者通常是一样的
kubectl expose deployment/nginx-deployment \
--name=nginx-service --type=ClusterIP --port=80 --target-port=80
# 随机产生主机端口
kubectl expose deployment/nginx-deployment \
--name=nginx-service2 --type=NodePort --port=8080 --target-port=80
```

### 1.8.2、访问service

外部主机访问：192.168.56.109:32296

>1.NodePort端口是随机的，范围为:30000-32767。
 2.集群中每一个主机节点的NodePort端口都可以访问。
 3.如果需要指定端口，不想随机产生，需要使用配置文件来声明。

![image.png](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/20240524133628.png)

```shell
#集群内访问
curl 10.43.65.187:80
#容器内访问
kubectl run nginx-test --image=nginx:1.22 -it --rm -- sh
#
curl nginx-service:80
```