# Kubernetes

## 1.1 应用部署方式演变



在部署应用程序的方式上，主要经历了三个时代：

- 传统部署：互联网早期，会直接将应用程序部署在物理机上



  > 优点：简单，不需要其它技术的参与
  >
  > 缺点：不能为应用程序定义资源使用边界，很难合理地分配计算资源，而且程序之间容易产生影响



- 虚拟化部署：可以在一台物理机上运行多个虚拟机，每个虚拟机都是独立的一个环境



> 优点：程序环境不会相互产生影响，提供了一定程度的安全性
>
> 缺点：增加了操作系统，浪费了部分资源





- 容器化部署：与虚拟化类似，但是共享了操作系统

> 优点：
>
> 可以保证每个容器拥有自己的文件系统、CPU、内存、进程空间等
>
> 运行应用程序所需要的资源都被容器包装，并和底层基础架构解耦
>
> 容器化的应用程序可以跨云服务商、跨Linux操作系统发行版进行部署

<img src=".\imgs\202307031703576.png" alt="img" />



> 容器化部署方式给带来很多的便利，但是也会出现一些问题，比如说：

- 一个容器故障停机了，怎么样让另外一个容器立刻启动去替补停机的容器
- 当并发访问量变大的时候，怎么样做到横向扩展容器数量



> 这些容器管理的问题统称为容器编排问题，为了解决这些容器编排问题，就产生了一些容器编排的软件

- Swarm：Docker自己的容器编排工具
- Mesos：Apache的一个资源统一管控的工具，需要和Marathon结合使用
- Kubernetes：Google开源的的容器编排工具





## 1.2 kubernetes简介

<img src=".\imgs\202307031705568.png" alt="image-20200406232838722" />


kubernetes，是一个全新的基于容器技术的分布式架构领先方案，是谷歌严格保密十几年的秘密武器----Borg系统的一个开源版本，于2014年9月发布第一个版本，2015年7月发布第一个正式版本。

kubernetes的本质是一组服务器集群，它可以在集群的每个节点上运行特定的程序，来对节点中的容器进行管理。目的是实现资源管理的自动化，主要提供了如下的主要功能：

- 自我修复：一旦某一个容器崩溃，能够在1秒中左右迅速启动新的容器
- 弹性伸缩：可以根据需要，自动对集群中正在运行的容器数量进行调整
- 服务发现：服务可以通过自动发现的形式找到它所依赖的服务
- 负载均衡：如果一个服务起动了多个容器，能够自动实现请求的负载均衡
- 版本回退：如果发现新发布的程序版本有问题，可以立即回退到原来的版本
- 存储编排：可以根据容器自身的需求自动创建存储卷



## 1.3 kubernetes组件

一个kubernetes集群主要是由控制节点(master)、**工作节点(node)**构成，每个节点上都会安装不同的组件。

> master：集群的控制平面，负责集群的决策 ( 管理 )



- ApiServer : 资源操作的唯一入口，接收用户输入的命令，提供认证、授权、API注册和发现等机制
- kubeScheduler : 负责集群资源调度，按照预定的调度策略将Pod调度到相应的node节点上
- kubeControllerManager : 负责维护集群的状态，比如程序部署安排、故障检测、自动扩展、滚动更新等
- Etcd ：负责存储集群中各种资源对象的信息，键值类型存储的分布式数据库，提供了基于Raft算法的实现自主的集群高可用
- cloudContollerManager 云控制管理器，第三方平台提供的控制器API对接管理功能

<img src=".\imgs\image-20240422204112224.png" alt="image-20240422204112224" />



> node：集群的数据平面，负责为容器提供运行环境 ( 干活 )


- Kubelet : 负责维护容器的生命周期，即通过控制docker，来创建、更新、销毁容器

- KubeProxy : 负责提供集群内部的服务发现和负载均衡

- container-runtime 容器运行时环境：docker， containerd，CRI-O

  

<img src=".\imgs\202307031705927.png" alt="image-20200406184656917" />



## 1.4 服务分类

- 无状态
  - 代码产品 ：Nginx， Apache
  - 优点：对客户端透明，无依赖关系，可以高效的实现扩容，迁移
  - 缺点：不能存储数据，需要额外的数据服务支持
- 有状态
  - 代码产品：MySQL，Redis
  - 优点：可以独立存储数据，实现数据管理
  - 缺点：集群环境下需要实现主从，数据同步，备份，水平扩容复杂



## 1.5 资源和对象

> Java程序员可以理解为资源就是类，对象就是提供资源new出来的





###  1.5.1资源的分类

- 元数据型资源
  - 对于资源的元数据描述，每一个资源都可以使用元数据的数据
- 集群
  - 作用于集群之上，集群下的所以资源都可以共享使用
- 命名空间
  - 作用于命名空间之上，通常只能在该命名空间范围内使用，个人理解为Scope



#### 元数据

##### Horizontal Pod Autoscaler （HPA）

> Pod自动扩容:可以根据CPU使用率或自定义指标(metrics)自动对 Pod 进行扩/缩空。



- 控制管理器每隔30s(可以通过-horizontal-pod-autoscaler-sync-period修改)查询metrics的资源使用情况
- 支持三种metrics类型
  - 预定义metrics(比如Pod的CPU)以利用率的方式计算
  - 自定义的Pod metrics，以原始值(raw value)的方式计算
  - 自定义的object metrics
  - 支持两种metrics查询方式:Heapster和自定义的REST API
  - 支持多metrics



##### PodTemplate

Pod Template是关于Pod的定义，但是被包含在其他的Kubernetes对象中（例如 Deployment、StatefulSet、
Daemonset等控制器)。控制器通过Pod Template 信息来创建Pod。



##### LimitRange

可以对集群内 Request和limits 的配置做一个全局的统一的限制。相当于批量设置了某一个范围内(某个命名空间)的 Pod 的资源使用限制。



#### 集群级



##### Namespace

> 命名空间是在集群内



##### Node

不像其他的资源(如Pod和Namespace) , Node本质上不是Kubernetes来创建的,Kubernetes只是管理Node 上的资源。虽然可以通过Manifest 创建一个Node对象(如下json所示)，但Kubernetes也只是去检查是杏真的是有这么一个Node，如果检查失败，也不会往上调度Pod。



##### ClusterRole

> 用于集群的权限管理





##### ClusterRoleBinding

> 把权限绑定到用户上 只能往集群资源上绑定





#### 命令空间级



##### Pod

Pod 是可以在 Kubernetes 中创建和管理的、最小的可部署的计算单元。

Pod（就像在鲸鱼荚或者豌豆荚中）是一组（一个或多个） [容器](https://kubernetes.io/zh-cn/docs/concepts/containers/)； 这些容器共享存储、网络、以及怎样运行这些容器的声明。 Pod 中的内容总是并置（colocated）的并且一同调度，在共享的上下文中运行。 Pod 所建模的是特定于应用的 “逻辑主机”，其中包含一个或多个应用容器， 这些容器相对紧密地耦合在一起。 在非云环境中，在相同的物理机或虚拟机上运行的应用类似于在同一逻辑主机上运行的云应用。

除了应用容器，Pod 还可以包含在 Pod 启动期间运行的 [Init 容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/init-containers/)。 你也可以注入[临时性容器](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/ephemeral-containers/)来调试正在运行的 Pod。



###### 副本

每个 Pod 都旨在运行给定应用程序的单个实例。如果希望横向扩展应用程序 （例如，运行多个实例以提供更多的资源），则应该使用多个 Pod，每个实例使用一个 Pod。 在 Kubernetes 中，这通常被称为**副本（Replication）**。 通常使用一种工作负载资源及其[控制器](https://kubernetes.io/zh-cn/docs/concepts/architecture/controller/)来创建和管理一组 Pod 副本。



###### 控制器



#### 适用于无状态服务



###### ReplicationController（RC） 弃用

> 相当于自动扩容和缩容



###### ReplicaSet （RS）

> 相当于自动扩容和缩容 但是可以使用selector 来选择对哪些pod生效 RC的升级



###### deployment

> RS 的升级

- 创建 Replica Set / pod
- 滚动升级和回滚
- 平滑扩容和缩容
- 暂停与恢复deployment



#### 适用于有状态的服务

> 特点

- 稳定的持久化存储
- 稳定的网络标识
- 有序部署，有序扩展
- 有序收缩，有序删除



###### StatefulSet

> Statefulset中每个Pod的DNS格式为 statefulSetName-{0到N-1}serviceName.namespace.svc.cluster.local

- serviceName为Headless service的名字
- 0到N-1为Pod所在的序号，从0开始到N-1
- statefulSetName为Statefulset 的名字
- namespace为服务所在的namespace, Headless Servic禾statefulSet 必须在相同的namespace
- .cluster.local为cluster Domain



headless Service： 对有状态的服务进行DNS管理

volumeClaim Template： 用于创建持久化卷的模板



#### 守护进程

> Daemonset 保证在每个Node 上都运行一个容器副本，常用来部署—些集群的日志、监控或者其他系统管理应用。典型的应用包括:

- 日志收集：比如 fluentd，logsash
- 系统监控：比如Prometheus Node Exporter， collectd ,New Relic agent.Ganglia gmond等
- 系统程序，比如kube-proxy, kube-dns, glusterd, ceph 等



#### 任务/定时任务

###### job

> 一次性任务，运行完后pod销毁，不再重新启动新容器



###### CornJob

> 是在job基础上加上了定时器







#### 服务发现

> service

实现k8s集群内部的网络调用，东西流量



> ingress

k8s里面的服务对外暴露，南北流量

<img src=".\imgs\k8s.md" alt="image-20240428135749391" />



> k8s 架构图



<img src=".\imgs\image-20240428140742186.png" alt="image-20240428140742186" />





#### 配置与存储



##### 存储



###### volume

数据卷,共享Pod中容器使用的数据。用来放持久化的数据，比如数据库数据。



###### CSI

Container Storage lnterface是由来自Kubernetes、Mesos、Docker 等社区成员联合制定的一个行业标准接口规范，旨在将任意存储系统暴露给容器化应用程序。

csI规范定义了存储提供简实现CSI兼容的volume Plugin 的最小操作集和部署建议。CSI规范的主要焦点是声明volume Plugin必须实现的接口。



##### 特殊类型配置



###### configMap

ConfigMap 是一种 API 对象，用来将非机密性的数据保存到键值对中。使用时， [Pod](https://kubernetes.io/zh-cn/docs/concepts/workloads/pods/) 可以将其用作环境变量、命令行参数或者存储卷中的配置文件。

ConfigMap 将你的环境配置信息和[容器镜像](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-image)解耦，便于应用配置的修改。

ConfigMap 是一个让你可以存储其他对象所需要使用的配置的 [API 对象](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/#kubernetes-objects)。 和其他 Kubernetes 对象都有一个 `spec` 不同的是，ConfigMap 使用 `data` 和 `binaryData` 字段。这些字段能够接收键-值对作为其取值。`data` 和 `binaryData` 字段都是可选的。`data` 字段设计用来保存 UTF-8 字符串，而 `binaryData` 则被设计用来保存二进制数据作为 base64 编码的字串。

ConfigMap 的名字必须是一个合法的 [DNS 子域名](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/names#dns-subdomain-names)。

`data` 或 `binaryData` 字段下面的每个键的名称都必须由字母数字字符或者 `-`、`_` 或 `.` 组成。在 `data` 下保存的键名不可以与在 `binaryData` 下出现的键名有重叠。



	###### secret

Secret 是一个主要用来存储密码、token 等一些敏感信息的资源对象。其中，敏感信息是采用 Base64 编码保存起来的。

注意：Base64只是一种编码，不含密钥的，并不安全。默认情况下，Kubernetes Secret 未加密地存储在 API 服务器的底层数据存储（etcd）中。 任何拥有 API 访问权限的人都可以检索或修改 Secret，任何有权访问 etcd 的人也可以。 此外，任何有权限在命名空间中创建 Pod 的人都可以使用该访问权限读取该命名空间中的任何 Secret；
secret安全相关的静态加密等，将在k8s学习-思维导图与学习笔记中的安全部分详细展开。



|              内置类型               |                  用法                  |
| :---------------------------------: | :------------------------------------: |
|               Opaque                |           用户定义的任意数据           |
| kubernetes.io/service-account-token |              服务账号令牌              |
|       kubernetes.io/dockercfg       |      ~/.dockercfg文件的序列化形式      |
|   kubernetes.io/dockerconfigjson    | ~/.docker/config.json 文件的序列化形式 |
|      kubernetes.io/basic-auth       |         用于基本身份认证的凭据         |
|       kubernetes.io/ssh-auth        |        用于 SSH 身份认证的凭据         |
|          kubernetes.io/tls          |   用于 TLS 客户端或者服务器端的数据    |
|    bootstrap.kubernetes.io/token    |            启动引导令牌数据            |




	###### DownwardAPI

Downward API可以通过以下两种方式将Pod信息注入容器内部

1.环境变量：用于单个变量，可以将Pod信息和[Container](https://so.csdn.net/so/search?q=Container&spm=1001.2101.3001.7020)信息注入到容器内部

2.[Volume](https://so.csdn.net/so/search?q=Volume&spm=1001.2101.3001.7020)挂载：通过文件挂载到容器内部




​	
​	
​	

	 ## 2.0 其他

###### Role

Role是一组权限的集合，例如Role可以包含列出Pod权限及列Deployment权限，Role用于给某个Namespace中的资源进行鉴权。



###### RoleBinding

可以使用ClusterRoleBinding在集群级别和所有命名空间中授予权限。下面示例中所定义的ClusterRoleBinding 允许在用户组”manager”中的任何用户都可以读取集群中任何命名空间中的secret





### 1.5.2 资料清单

待补充



## 1.6 对象规约和状态

几乎每个 Kubernetes 对象包含两个嵌套的对象字段，它们负责管理对象的配置： 对象 **`spec`（规约）** 和对象 **`status`（状态）**。 对于具有 `spec` 的对象，你必须在创建对象时设置其内容，描述你希望对象所具有的特征： **期望状态（Desired State）**。

`status` 描述了对象的**当前状态（Current State）**，它是由 Kubernetes 系统和组件设置并更新的。在任何时刻，Kubernetes [控制平面](https://kubernetes.io/zh-cn/docs/reference/glossary/?all=true#term-control-plane) 都一直在积极地管理着对象的实际状态，以使之达成期望状态。

例如，Kubernetes 中的 Deployment 对象能够表示运行在集群中的应用。 当创建 Deployment 时，你可能会设置 Deployment 的 `spec`，指定该应用要有 3 个副本运行。 Kubernetes 系统读取 Deployment 的 `spec`， 并启动我们所期望的应用的 3 个实例 —— 更新状态以与规约相匹配。 如果这些实例中有的失败了（一种状态变更），Kubernetes 系统会通过执行修正操作来响应 `spec` 和 `status` 间的不一致 —— 意味着它会启动一个新的实例来替换。

### 规约

> 相当于你的配置文件的规约spec 比如规定命名，版本

### 状态

> 实际生成的对象会尽力满足你的规约





## 2.0 K8S实战篇



### 2.1搭建k8s集群


#### 搭建方案

- minkube
- kubeadm
- 二进制安装
- 命令行工具



##### 服务器要求

> 学习环境

- 3台服务器，也就是虚拟机
- 最低配置2核，2G内存，20G硬盘
- 最好连网



##### 软件环境

- 操作系统：Cent OS7

- Docker 20 +

- k8s： 1.23.6




##### 安装步骤



###### 1.初始操作

> 关闭防火墙

```shell
[root@localhost ~]# systemctl stop firewalld
[root@localhost ~]# systemctl disable firewalld
[root@localhost ~]# systemctl status firewalld
● firewalld.service - firewalld - dynamic firewall daemon
   Loaded: loaded (/usr/lib/systemd/system/firewalld.service; disabled; vendor preset: enabled)
   Active: inactive (dead)
     Docs: man:firewalld(1)

```



> 关闭selinux

这个命令的意思是将系统的SELinux安全策略从"enforcing"模式修改为"disabled"模式。这将禁用SELinux对系统的安全保护。

```shell
[root@localhost ~]# sed -i 's/enforcing/disabled/' /etc/selinux/config 
setenforce     #临时
```



> 关闭swap

`swap`是一种虚拟内存技术，用于将内存中暂时不需要的数据或程序移动到硬盘上的一个特定分区，以释放内存空间供其他程序使用。当系统内存不足时，操作系统会将一些数据从内存中移动到`swap`分区，从而避免出现内存溢出的情况。`swap`分区通常是在硬盘上预留的一个特定空间，用于作为虚拟内存使用。

关闭完swap后 一定要重启一下虚拟机  reboot 重启虚拟机

```shell
 sed -ri 's/.*swap.*/#&/' /etc/fstab   #永久
 swapoff -a   #临时
```



> 根据规划设计主机名 不能有_和.

```shell
hostnamectl set-hostname k8sMaster
hostnamectl set-hostname k8sNode1
hostnamectl set-hostname k8sNode2
```



> 配置DNS吧 

```shell
cat >> /etc/hosts << EOF

192.168.206.130 k8sMaster

192.168.206.131 k8sNode1

192.168.206.132 k8sNode2

EOF
```



配置生效

```shell
sysctl --system
```



时间同步

```shell
yum install ntpdate -y ntdate time.windows.com
```



注意如果提示没有yun, 安装yum

```shell
 wget http://mirrors.163.com/centos/7/os/x86_64/Packages/python-urlgrabber-3.10-10.el7.noarch.rpm

 wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-cron-3.4.3-168.el7.centos.noarch.rpm

 wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-3.4.3-168.el7.centos.noarch.rpm

 wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-metadata-parser-1.1.4-10.el7.x86_64.rpm

 wget http://mirrors.163.com/centos/7/os/x86_64/Packages/yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm



 rpm -ivh --force --nodeps python-urlgrabber-3.10-10.el7.noarch.rpm 

 rpm -ivh --force --nodeps yum-metadata-parser-1.1.4-10.el7.x86_64.rpm 

 rpm -ivh --force --nodeps yum-3.4.3-168.el7.centos.noarch.rpm yum-plugin-fastestmirror-1.1.31-54.el7_8.noarch.rpm 
 
 
 # 下载163配置仓库repo文件
wget http://mirrors.163.com/.help/CentOS7-Base-163.repo
# 复制下载的文件到配置目录下
 cp CentOS7-Base-163.repo /etc/yum.repos.d/ 
# 切换到配置目录下
cd /etc/yum.repos.d/ 
# 重命名
mv CentOS-Base.repo CentOS-Base.repo.bak 
# 重命名
mv CentOS7-Base-163.repo CentOS-Base.repo

yum clean all 

yum makecache 

yum update
```



> 配置yum源  这个yum源有对应的kubelet 1.23.6

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


yum clean all

yum makecache
```





> 安装docker

```shell
#降级
yum downgrade --setopt=obsoletes=0 -y docker-ce-18.09.9-3.el7 docker-ce-cli-18.09.9-3.el7 containerd.io

# 安装
yum install docker-ce-18.09.9-3.el7
```





>  安装k8s

```shell
yum install -y kubelet-1.23.6 kubeadm-1.23.6 kubectl-1.23.6 
systemctl enable kubelet 设计为开机启动

# 如果没搜索到k8s 配置kubernetes镜像源
echo "[kubernetes]" > /etc/yum.repos.d/kubernetes.repo
echo "name=Kubernetes" >> /etc/yum.repos.d/kubernetes.repo
echo "baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/" >> /etc/yum.repos.d/kubernetes.repo
echo "enabled=1" >> /etc/yum.repos.d/kubernetes.repo
echo "gpgcheck=0" >> /etc/yum.repos.d/kubernetes.repo
cat /etc/yum.repos.d/kubernetes.repo
```



> 初始化k8s

```shell
kubeadm init \
	--apiserver-advertise-address=192.168.206.130 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.23.6 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
	
	
	kubeadm init \
	--apiserver-advertise-address=192.168.206.131 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.23.6 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
	
	kubeadm init \
	--apiserver-advertise-address=192.168.206.132 \
	--image-repository registry.aliyuncs.com/google_containers \
	--kubernetes-version=v1.23.6 \
	--service-cidr=10.96.0.0/12 \
	--pod-network-cidr=10.244.0.0/16
```



解决报错 : docker的cgroup驱动程序默认设置为system。默认情况下Kubernetes cgroup为systemd，我们需要更改Docker cgroup驱动，

```shell
[kubelet-check] The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get "http://localhost:10248/healthz": dial tcp 127.0.0.1:10248: connect: connection refused.

kubeadm reset

vim /etc/docker/daemon.json

{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}

systemctl daemon-reload

systemctl restart docker
```





解决报错

```shell
[ERROR FileContent--proc-sys-net-bridge-bridge-nf-call-iptables]: /proc/sys/net/bridge/bridge-nf-call-iptables contents are not set to 1

vim /etc/sysctl.conf

net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1


 sysctl -p
```



解决报错

```shell
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1

sysctl -w net.ipv4.ip_forward=1
```





> 初始化成功

```shell
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube     # 需要执行
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config  # 需要执行
  sudo chown $(id -u):$(id -g) $HOME/.kube/config  # 需要执行

Alternatively, if you are the root user, you can run:

  export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:
# 谁需要加入这个节点谁执行
kubeadm join 192.168.206.130:6443 --token o16nqg.ftcmrek7in0z0nb5 \
	--discovery-token-ca-cert-hash sha256:a9c341457a3c7c3572fdba72e557d03da13e21136a1970956e54ef1ec13caa24 

kubeadm join 192.168.206.131:6443 --token 6ft7e0.qn66wwvbcooq4z0v \
	--discovery-token-ca-cert-hash sha256:42e9548d9b6cd237b9bea27c206cdb4e529f74032fd740401c8fcb51df0b1830 

kubeadm join 192.168.206.132:6443 --token ebfio9.l2h64cmftn9shsvh \
	--discovery-token-ca-cert-hash sha256:5cf5a76f8f1b198a0160acc5f276641280f9973b6e2d2e75e5f9659a20c04586 


```



###### 2. 加入kubernetes Node

分别在k8s node1和 k8s node2执行

```shell
# 192.168.206.130:6443 master 的ip和端口
kubeadm join 192.168.206.130:6443 --token [master控制台的token] \
	--discovery-token-ca-cert-hash sha256:[master控制台的hash]

```

> 如果master的初始化的token被清空可以通过下面的命令重新申请

```shell
--discovery-token-ca-cert-hash sha256:# 查看当前token
[root@k8smaster ~]# kubeadm token list

#生成新的token
[root@k8smaster ~]# kubeadm token create
4wdr7f.7qdbs17sozs24dln

# 生成 --discovery-token-ca-cert-hash sha256:
[root@k8smaster ~]# openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
a9c341457a3c7c3572fdba72e557d03da13e21136a1970956e54ef1ec13caa24

```

> 进行一个拼接

```shell
kubeadm join 192.168.206.130:6443 --token 4wdr7f.7qdbs17sozs24dln \
	--discovery-token-ca-cert-hash sha256:a9c341457a3c7c3572fdba72e557d03da13e21136a1970956e54ef1ec13caa24

```



> 运行如果报错了

```shell
[root@k8snode1 ~]# kubeadm join 192.168.206.130:6443 --token 4wdr7f.7qdbs17sozs24dln \
> --discovery-token-ca-cert-hash sha256:a9c341457a3c7c3572fdba72e557d03da13e21136a1970956e54ef1ec13caa24
# 报错
[preflight] Running pre-flight checks
error execution phase preflight: [preflight] Some fatal errors occurred:
[ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[ERROR Port-10250]: Port 10250 is in use
[ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`

# 执行 kubeadm reset 

#报错
[ERROR FileContent–proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
#执行 sysctl -w net.ipv4.ip_forward=1
```



> 如何判加入成功

```shell
# 未加入前只有一个
[root@k8smaster ~]# kubectl get nodes
NAME        STATUS     ROLES                  AGE     VERSION
k8smaster   NotReady   control-plane,master   2d18h   v1.23.6

#运行后是两个 当前的状态是NotReady 没有准备好
[root@k8smaster ~]# kubectl get nodes
NAME        STATUS     ROLES                  AGE     VERSION
k8smaster   NotReady   control-plane,master   2d18h   v1.23.6
k8snode1    NotReady   <none> 
```



> 去查看组件的情况都是ok

```shell
# cs 是 ComponentStatus 的缩写
[root@k8smaster ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
etcd-0               Healthy   {"health":"true","reason":""}   
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
[root@k8smaster ~]# kubectl get ComponentStatus

```



> 查看pod

```shell
# 看不见是和命令空间有关系
[root@k8smaster ~]# kubectl get pods
No resources found in default namespace.
#下面有2个状态处于Pending状态 待定状态 需要去下载
[root@k8smaster ~]# kubectl get pods -n kube-system
NAME                                READY   STATUS    RESTARTS       AGE
coredns-6d8c4cb4d-8jg9b             0/1     Pending   0              2d18h
coredns-6d8c4cb4d-zvs96             0/1     Pending   0              2d18h
etcd-k8smaster                      1/1     Running   3 (2d5h ago)   2d18h
kube-apiserver-k8smaster            1/1     Running   2 (2d5h ago)   2d18h
kube-controller-manager-k8smaster   1/1     Running   3 (2d5h ago)   2d18h
kube-proxy-q4tvd                    1/1     Running   0              7m47s
kube-proxy-zflrx                    1/1     Running   0              14m
kube-proxy-zgggd                    1/1     Running   2 (2d5h ago)   2d18h
kube-scheduler-k8smaster            1/1     Running   2 (2d5h ago)   2d18h

```



###### 3.部署 CNI 网络插件

> 下载calico

 ```shell
 # 下载calico配置文件， 在master节点上执行
 [root@k8smaster ~]# cd /opt/
 [root@k8smaster opt]# ls
 cni  containerd  redis-7.0.11  redis-7.0.11.tar.gz  rh
 [root@k8smaster opt]# mkdir k8s
 [root@k8smaster opt]# cd k8s/
 [root@k8smaster k8s]# curl -O https://docs.tigera.io/archive/v3.25/manifests/calico.yaml
 [root@k8smaster k8s]# ls
 calico.yaml
 
 ```



> 修改配置文件

```shell
vim calico.yaml

# 默认是注释起来了
    # - name: CALICO_IPV4POOL_CIDR
#  value: "192.168.0.0/16"


#添加  10.244.0.0/16 这个是和初始化k8s 那个要对应起来 
- name: CALICO_IPV4POOL_CIDR
  value: "10.244.0.0/16" 
  
# 修改网卡信息 eth0是本地网卡名称 不知道自己的网卡名称 可以 ifconfig
- name: IP_AUTODETECTION_METHOD
  value: "interface=ens32"
```



> 去构建

```shell
[root@k8smaster k8s]# grep image calico.yaml 
          image: docker.io/calico/cni:v3.25.0
          imagePullPolicy: IfNotPresent
          image: docker.io/calico/cni:v3.25.0
          imagePullPolicy: IfNotPresent
          image: docker.io/calico/node:v3.25.0
          imagePullPolicy: IfNotPresent
          image: docker.io/calico/node:v3.25.0
          imagePullPolicy: IfNotPresent
          image: docker.io/calico/kube-controllers:v3.25.0
          imagePullPolicy: IfNotPresent
[root@k8smaster k8s]# sed -i 's#docker.io/##g' calico.yaml 
[root@k8smaster k8s]# grep image calico.yaml 
          image: calico/cni:v3.25.0
          imagePullPolicy: IfNotPresent
          image: calico/cni:v3.25.0
          imagePullPolicy: IfNotPresent
          image: calico/node:v3.25.0
          imagePullPolicy: IfNotPresent
          image: calico/node:v3.25.0
          imagePullPolicy: IfNotPresent
          image: calico/kube-controllers:v3.25.0
          imagePullPolicy: IfNotPresent
[root@k8smaster k8s]# kubectl apply -f calico.yaml 
# 查看当前 Init 代表正在初始化  去看看这个容器的详细信息
[root@k8smaster k8s]# kubectl get pods -n kube-system
NAME                                     READY   STATUS                  RESTARTS       AGE
calico-kube-controllers-cd8566cf-5p55k   0/1     Pending                 0              72s
calico-node-rztjk                        0/1     Init:ImagePullBackOff   0              72s
calico-node-wrmqh                        0/1     Init:ImagePullBackOff   0              72s
calico-node-xjgjt                        0/1     Init:ImagePullBackOff   0              72s
coredns-6d8c4cb4d-8jg9b                  0/1     Pending                 0              2d19h
coredns-6d8c4cb4d-zvs96                  0/1     Pending                 0              2d19h
etcd-k8smaster                           1/1     Running                 3 (2d5h ago)   2d19h
kube-apiserver-k8smaster                 1/1     Running                 2 (2d5h ago)   2d19h
kube-controller-manager-k8smaster        1/1     Running                 3 (2d5h ago)   2d19h
kube-proxy-q4tvd                         1/1     Running                 0              35m
kube-proxy-zflrx                         1/1     Running                 0              42m
kube-proxy-zgggd                         1/1     Running                 2 (2d5h ago)   2d19h
kube-scheduler-k8smaster                 1/1     Running                 2 (2d5h ago)   2d19h

#  查看这个pod的信息 没有配置容忍所以无法帮我们部署
[root@k8smaster k8s]# kubectl describe po calico-kube-controllers-cd8566cf-5p55k -n kube-system
Warning  FailedScheduling  44s (x6 over 5m10s)  default-scheduler  0/3 nodes are available: 3 node(s) had taint {node.kubernetes.io/not-ready: }, that the pod didn't tolerate.

```



> 再次查看 Component Status 现在的状态是Healthy 健康的

```shell
[root@k8smaster k8s]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE                         ERROR
scheduler            Healthy   ok                              
controller-manager   Healthy   ok                              
etcd-0               Healthy   {"health":"true","reason":""}   

```



> 再次查看pod 现在的状态 还有两个pod还在初始化

```shell
[root@k8smaster k8s]# kubectl get pods -n kube-system
NAME                                     READY   STATUS                  RESTARTS       AGE
calico-kube-controllers-cd8566cf-5p55k   1/1     Running                 0              10m
calico-node-rztjk                        0/1     Init:ImagePullBackOff   0              10m
calico-node-wrmqh                        0/1     Init:ImagePullBackOff   0              10m
calico-node-xjgjt                        1/1     Running                 0              10m
coredns-6d8c4cb4d-8jg9b                  1/1     Running                 0              2d19h
coredns-6d8c4cb4d-zvs96                  1/1     Running                 0              2d19h
etcd-k8smaster                           1/1     Running                 3 (2d6h ago)   2d19h
kube-apiserver-k8smaster                 1/1     Running                 2 (2d6h ago)   2d19h
kube-controller-manager-k8smaster        1/1     Running                 3 (2d6h ago)   2d19h
kube-proxy-q4tvd                         1/1     Running                 0              45m
kube-proxy-zflrx                         1/1     Running                 0              52m
kube-proxy-zgggd                         1/1     Running                 2 (2d6h ago)   2d19h
kube-scheduler-k8smaster                 1/1     Running                 2 (2d6h ago)   2d19h

```



> 查看pod的详情

```shell
[root@k8smaster k8s]# kubectl describe po calico-node-rztjk -n kube-system
  Warning  Failed     13m                  kubelet            Failed to pull image "calico/cni:v3.25.0": rpc error: code = Unknown desc = Get https://registry-1.docker.io/v2/calico/cni/manifests/sha256:ec0fa42b4d03398995800b44131b200aee2c76354d405d5d91689ec99cc70c56: read tcp 192.168.206.131:33196->54.227.20.253:443: read: connection reset by peer
  
# 如果下不下来就使用
docker pull 容器名称
```



> 全正常情况

```shell
[root@k8smaster k8s]# kubectl get nodes
NAME        STATUS   ROLES                  AGE     VERSION
k8smaster   Ready    control-plane,master   2d19h   v1.23.6
k8snode1    Ready    <none>                 64m     v1.23.6
k8snode2    Ready    <none>                 57m     v1.23.6
[root@k8smaster k8s]# kubectl get po -n kube-system
NAME                                     READY   STATUS    RESTARTS       AGE
calico-kube-controllers-cd8566cf-5p55k   1/1     Running   0              22m
calico-node-rztjk                        1/1     Running   0              22m
calico-node-wrmqh                        1/1     Running   0              22m
calico-node-xjgjt                        1/1     Running   0              22m
coredns-6d8c4cb4d-8jg9b                  1/1     Running   0              2d19h
coredns-6d8c4cb4d-zvs96                  1/1     Running   0              2d19h
etcd-k8smaster                           1/1     Running   3 (2d6h ago)   2d19h
kube-apiserver-k8smaster                 1/1     Running   2 (2d6h ago)   2d19h
kube-controller-manager-k8smaster        1/1     Running   3 (2d6h ago)   2d19h
kube-proxy-q4tvd                         1/1     Running   0              57m
kube-proxy-zflrx                         1/1     Running   0              64m
kube-proxy-zgggd                         1/1     Running   2 (2d6h ago)   2d19h
kube-scheduler-k8smaster                 1/1     Running   2 (2d6h ago)   2d19h

```



###### 4.测试 kubernetes 集群

> 创建部署nginx

```shell
# 创建部署
[root@k8smaster k8s]# kubectl create  deployment nginx --image=nginx
deployment.apps/nginx created
# 暴露端口
[root@k8smaster k8s]# kubectl expose deployment nginx --port=80 --type=NodePort
service/nginx exposed

# 查看这个pod的 交换虚拟通路 （svc交换虚拟通路）
[root@k8smaster k8s]# kubectl get pod,svc
NAME                         READY   STATUS    RESTARTS   AGE
pod/nginx-85b98978db-lgjfs   1/1     Running   0          2m56s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        2d19h
service/nginx        NodePort    10.106.85.217   <none>        80:31290/TCP   99s

# 成功访问
[root@k8smaster k8s]# curl 192.168.206.130:31290
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```



###### 在任意节点使用kubelet

> 当前的所有操作都是在master节点上操作的其余节点无法操作

```shell
[root@k8snode1 ~]# kubectl get nodes
The connection to the server 192.168.206.131:6443 was refused - did you specify the right host or port?

```



> 将master节点中的/etc/kubernetes/admin.conf 复制到其他节点上
>
> root@k8sNode1:   root 代表以root登录，k8sNode1 是配置的/etc/hosts  不然就应该使用IP地址

```shell
[root@k8smaster ~]# scp /etc/kubernetes/admin.conf root@k8sNode1:/etc/kubernetes/
root@k8snode1's password: 
admin.conf   
```



> 在对应的服务上配置环境变量

```shell
echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> ~/.bash_profile  
source ~/.bash_profile
```



> 成功使用kubelet

```shell
[root@k8snode2 kubernetes]# kubectl get nodes
NAME        STATUS   ROLES                  AGE     VERSION
k8smaster   Ready    control-plane,master   2d20h   v1.23.6
k8snode1    Ready    <none>                 128m    v1.23.6
k8snode2    Ready    <none>                 121m    v1.23.6
```



### 2.2 kubectl  命令

详情看官网 [命令行工具 (kubectl) | Kubernetes](https://kubernetes.io/zh-cn/docs/reference/kubectl/)

#### 2.2.1资源类型和别名

- pods                                          po
- deployments                          deploy
- services                                    svc
- namespace                              ns
- nodes                                       no



#### 2.2.2 格式化输出

> 先查询出名称

```
kubectl get deploy
```



> 输出json格式  -o json

```
kubectl get deploy nginx -o yaml
```



> 打印出资源名称 -o name

```
kubectl get deploy nginx -o name
deployment.apps/nginx

```



> 打印出所有信息 -o wide

```
[root@k8snode2 ~]# kubectl get deploy nginx -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES   SELECTOR
nginx   1/1     1            1           28h   nginx        nginx    app=nginx

```



> 打印出json 格式 -o json

```
kubectl get deploy nginx -o json
```



#### 2.2.3 API 概述

官网链接 [API 概述 | Kubernetes](https://kubernetes.io/zh-cn/docs/reference/using-api/)

JSON 和 Protobuf 序列化模式遵循相同的模式更改原则。 以下描述涵盖了这两种格式。

API 版本控制和软件版本控制是间接相关的。 [API 和发布版本控制提案](https://git.k8s.io/sig-release/release-engineering/versioning.md)描述了 API 版本控制和软件版本控制间的关系。

不同的 API 版本代表着不同的稳定性和支持级别。 你可以在 [API 变更文档](https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions) 中查看到更多的不同级别的判定标准。



- Alpha：
  - 版本名称包含 `alpha`（例如：`v1alpha1`）。
  - 内置的 Alpha API 版本默认被禁用且必须在 `kube-apiserver` 配置中显式启用才能使用。
  - 软件可能会有 Bug。启用某个特性可能会暴露出 Bug。
  - 对某个 Alpha API 特性的支持可能会随时被删除，恕不另行通知。
  - API 可能在以后的软件版本中以不兼容的方式更改，恕不另行通知。
  - 由于缺陷风险增加和缺乏长期支持，建议该软件仅用于短期测试集群。

- Beta：

  - 版本名称包含 `beta`（例如：`v2beta3`）。
  - 内置的 Beta API 版本默认被禁用且必须在 `kube-apiserver` 配置中显式启用才能使用 （例外情况是 Kubernetes 1.22 之前引入的 Beta 版本的 API，这些 API 默认被启用）。
  - 内置 Beta API 版本从引入到弃用的最长生命周期为 9 个月或 3 个次要版本（以较长者为准）， 从弃用到移除的最长生命周期为 9 个月或 3 个次要版本（以较长者为准）。
  - 软件被很好的测试过。启用某个特性被认为是安全的。
  - 尽管一些特性会发生细节上的变化，但它们将会被长期支持。

  - 在随后的 Beta 版或 Stable 版中，对象的模式和（或）语义可能以不兼容的方式改变。 当这种情况发生时，将提供迁移说明。 适配后续的 Beta 或 Stable API 版本可能需要编辑或重新创建 API 对象，这可能并不简单。 对于依赖此功能的应用程序，可能需要停机迁移。
  - 该版本的软件不建议生产使用。 后续发布版本可能会有不兼容的变动。 一旦 Beta API 版本被弃用且不再提供服务， 则使用 Beta API 版本的用户需要转为使用后续的 Beta 或 Stable API 版本。

- Stable：
  - 版本名称如 `vX`，其中 `X` 为整数。
  - 特性的 Stable 版本会出现在后续很多版本的发布软件中。 Stable API 版本仍然适用于 Kubernetes 主要版本范围内的所有后续发布， 并且 Kubernetes 的主要版本当前没有移除 Stable API 的修订计划。



可以去官网查看弃用的API

[已弃用 API 的迁移指南 | Kubernetes](https://kubernetes.io/zh-cn/docs/reference/using-api/deprecation-guide/)





pod操作

```
# 查看pods 下
[root@k8smaster ~]# kubectl get pods
NAME                     READY   STATUS    RESTARTS   AGE
nginx-85b98978db-lgjfs   1/1     Running   0          2d3h

# 查看dploy 下
[root@k8smaster ~]# kubectl get deploy
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           2d3h

# 查看service 下
[root@k8smaster ~]# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d23h
nginx        NodePort    10.106.85.217   <none>        80:31290/TCP   2d3h

# 删除deploy下的nginx
[root@k8smaster ~]# kubectl delete deploy nginx
deployment.apps "nginx" deleted

# pods 下的nginx 还在
[root@k8smaster ~]# kubectl get pods
NAME                     READY   STATUS        RESTARTS   AGE
nginx-85b98978db-lgjfs   0/1     Terminating   0          2d3h

# deploy 下的nginx 不在
[root@k8smaster ~]# kubectl get deploy
No resources found in default namespace.
# svc 下还存在
[root@k8smaster ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        4d23h
nginx        NodePort    10.106.85.217   <none>        80:31290/TCP   2d3h

# 删除svc下的nginx
[root@k8smaster ~]# kubectl delete svc nginx
service "nginx" deleted

# svc 下不存在
[root@k8smaster ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   4d23h

# pods 的也没准备好了0/1
[root@k8smaster ~]# kubectl get po
NAME                     READY   STATUS        RESTARTS   AGE
nginx-85b98978db-lgjfs   0/1     Terminating   0          2d3h

# 强行删除pod
kubectl delete pods nginx-85b98978db-lgjfs --grace-period=0 --force
kubectl delete pods nginx-demo --grace-period=0 --force
```





> 自己编写yml配置文件生成pod 结合下方官网

[为容器和 Pod 分配内存资源 | Kubernetes](https://kubernetes.io/zh-cn/docs/tasks/configure-pod-container/assign-memory-resource/)

```y
[root@k8smaster pods]# pwd
/opt/k8s/pods
[root@k8smaster pods]# touch ningx_demo.yaml
[root@k8smaster pods]# ls
ningx_demo.yaml

```



> yaml 的配置文件  人生建议不要在linux中写yaml  

[YAML、YML在线编辑器(格式化校验) - JSON中文网](https://www.json.cn/yaml-editor/)

```yaml
apiVersion: v1 # api 文档版本
kind: Pod # 资料对象类型
metadata:  #Pod相关的元数据，用于描述pod的数据
  name: nginx-demo # Pod的名称
  labels: # 定义pod的标签
   tyep: app
   version: 1.0.0 # 自定义的版本好
  namespace: 'default' # 命名空间配置
spec: # 期望 pod按照这里的配置创建
  containers:  # 对于pod中容器的描述
   - name: nginx # 容器的名称 -代表数字
     image: nginx:1.7.9 # 指定容器的镜像
     imagePullPolicy: IfNotPresent # 如果本地没有就去远程拉
     tartupProbe:   # 启动探针
       httpGet:      # 探针方式基于HTTP请求探针
         path: /api/path    # 请求路径
         port: 80  #请求端口
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成
     command:  # 容器启动执行的命令
     - nginx
     - -g
     - 'daemon off;' # nginx -g daemon off;
     workingDir: /usr/share/nginx/html # 定义容器启动后的工作目录
     ports:
      - name: http # 端口名称
        containerPort: 80 # 容器需要暴露的端口
        protocol : TCP  # 采用的协议
     env: # 环境变量
      - name: JVM_OPTS # 环境变量名称
        value: '-Xms128m -Xmx128m' # 环境变量的值
     resources:
       requests: # 最少需要多少资源
         cpu: 100m #cpu 最多使用一个核心的百分之10
         memory: 128Mi # 内存最少使用 128兆
       limits: # 最多可以使用多少资源
         cpu: 200m # 最多使用一个核心百分之20 
         memory: 256Mi # 256
  restartPolicy: OnFailure # 重启策略 只有失败才会重启


```





```
[root@k8smaster pods]# kubectl create -f ningx_demo.yaml 
pod/nginx-demo created
```



> 报错 nginx 启动报错

```
Failed to create pod sandbox: rpc error: code = Unknown desc = [failed to set up sandbox container "00b6872192203669df47261067fd3ce4df4ba9cbf171262669902030f5887dc7" network for pod "liveness-exec-demo": networkPlugin cni failed to set up pod "liveness-exec-demo_default" network: plugin type="calico" failed (add): error getting ClusterInformation: connection is unauthorized: Unauthorized, failed to clean up sandbox container "00b6872192203669df47261067fd3ce4df4ba9cbf171262669902030f5887dc7" network for pod "liveness-exec-demo": networkPlugin cni failed to teardown pod "liveness-exec-demo_default" network: plugin type="calico" failed (delete): error getting ClusterInformation: connection is unauthorized: Unauthorized]

```

> 可以尝试反复加入和删除

```
[root@k8smaster cni]# pwd
/opt/k8s/cni
[root@k8smaster cni]# ls
calico.yaml


vim calico.yaml        #编辑calico.yaml配置文件加入如下内容      
            - name: IP_AUTODETECTION_METHOD
              value: "interface=eth0"    #eth0是本地网卡名称 不知道自己的网卡名称 可以 ifconfig
 
kubectl apply -f calico.yaml    #更新
```



> 容器运行成功

```sh
# 查看pod的详情信息
[root@k8smaster pods]# kubectl get po -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
nginx-demo   1/1     Running   0          37m   10.244.185.193   k8snode2   <none>           <none>
# 查看在master能不能访问到 k8snode2 的容器
[root@k8smaster pods]# curl 10.244.185.193
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
# 查看路由信息   nginx 的网段是10.244.185
# 10.244.185.192  192.168.206.132 255.255.255.192 UG    0      0        0 tunl0
# 这个和nginx 属于同一网段  网关是 192.168.206.132 这个时候去看node2 节点
[root@k8smaster pods]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.206.2   0.0.0.0         UG    0      0        0 ens33
10.244.16.128   0.0.0.0         255.255.255.192 U     0      0        0 *
10.244.16.129   0.0.0.0         255.255.255.255 UH    0      0        0 cali157f1cec446
10.244.16.130   0.0.0.0         255.255.255.255 UH    0      0        0 cali69fcd6a109a
10.244.16.131   0.0.0.0         255.255.255.255 UH    0      0        0 cali71509f97f61
10.244.185.192  192.168.206.132 255.255.255.192 UG    0      0        0 tunl0
10.244.249.0    192.168.206.131 255.255.255.192 UG    0      0        0 tunl0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.206.0   0.0.0.0         255.255.255.0   U     0      0        0 ens33

```



> node2 的路由
>
> 10.244.185.192  0.0.0.0         255.255.255.192 U     0      0        0 * 是网关所以建立的通信

```
[root@k8snode2 ~]# route -n
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.206.2   0.0.0.0         UG    0      0        0 ens33
10.244.16.128   192.168.206.130 255.255.255.192 UG    0      0        0 tunl0
10.244.185.192  0.0.0.0         255.255.255.192 U     0      0        0 *
10.244.185.193  0.0.0.0         255.255.255.255 UH    0      0        0 cali7543b7f115c
10.244.249.0    192.168.206.131 255.255.255.192 UG    0      0        0 tunl0
169.254.0.0     0.0.0.0         255.255.0.0     U     1002   0        0 ens33
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.122.0   0.0.0.0         255.255.255.0   U     0      0        0 virbr0
192.168.206.0   0.0.0.0         255.255.255.0   U     0      0        0 ens33

```



### 2.3 pod探针

> 容器内应用的监听机制，根据不同的探针来判断容器应用当前的状态



#### 2.3.1探针类型

- Liveness Probe：指示容器是否正在运行。如果存活探测失败，则 kubelet 会杀死容器，并且容器将受到其 重启策略 的影响。如果容器不提供存活探针，则默认状态为 Success。

> 下方这个配置nginx 的配置  startupProbe 探针会成功  livenessProbe探针会失败 因为没有这个路径 index

```yaml
     startupProbe:   # 启动探针
       exec:
         command:
         - sh
         - -c
         - "sleep 3;echo 'success' > /inited;"
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长
     livenessProbe:
       httpGet:    # 探针方式基于HTTP请求探针
         port: 80
         path: /index   # 请求路径
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长
```





- Readiness Probe：指示容器是否准备好服务请求。如果就绪探测失败，端点控制器将从与 Pod 匹配的所有 Service 的端点中删除该 Pod 的 IP 地址。初始延迟之前的就绪状态默认为 Failure。如果容器不提供就绪探针，则默认状态为 Success。

```yaml
readinessProbe:
      httpGet:
        port: 80
        path: /index1.html
      initialDelaySeconds: 1
      periodSeconds: 3
```





- Startuup Probe： 启动探针, 配置了这个探针后，会先禁用其他探针

```yaml
     startupProbe:   # 启动探针
       httpGet:      # 探针方式基于HTTP请求探针
         path: /api/path    # 请求路径 这个路径nginx没有的
         port: 80  #请求端口
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长
```



> 给nginx 配置这个启动探针 查看pod 详情

```yaml
Events:
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  23s               default-scheduler  Successfully assigned default/nginx-demo to k8snode2
  Normal   Pulled     22s               kubelet            Container image "nginx:1.7.9" already present on machine
  Normal   Created    22s               kubelet            Created container nginx
  Normal   Started    22s               kubelet            Started container nginx
  Warning  Unhealthy  3s (x2 over 13s)  kubelet            Startup probe failed: HTTP probe failed with statuscode: 404

```





#### 2.3.2 探针方式

**HTTPGetAction**

kubelet 将 HTTP GET 请求发送到 endpoint，并检查 2xx 或 3xx 响应。我们可以重复使用现有的 HTTP endpoint 或设置轻量级 HTTP 服务器以进行探测（例如，具有 `/healthz` endpoint 的 Express server）。HTTP 探针包含其他额外参数：

- `host`：要连接的主机名（默认值：pod 的 IP）。
- `scheme`：HTTP（默认）或 HTTPS。
- `path`：HTTP/S 服务器上的路径 。
- `httpHeaders`：自定义标头（如果需要标头用于身份验证、CORS 设置等） 。
- `port`：访问服务器的端口名称或端口号。



```
     startupProbe:   # 启动探针
       httpGet:      # 探针方式基于HTTP请求探针
         path: /api/path    # 请求路径 这个路径nginx没有的
         port: 80  #请求端口
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长
```





**TCPSocketAction**

如果仅需要检查是否可以建立 TCP 连接，则可以指定 TCP 探针。如果建立 TCP 连接，则将 Pod 标记为运行状况良好。对于不适合使用 HTTP 探针的 gRPC 或 FTP 服务器，TCP 探针可能会有用。



```
     startupProbe:   # 启动探针
       tcpSocket:
         port: 80  #请求端口
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长
```



**ExecAction**

可以将探针配置为运行 shell 命令。如果命令返回的退出代码为 0，则检查通过，否则 Pod 将被标记为不健康。如果不希望公开 HTTP 服务器与端口，或者希望通过命令检查初始化步骤（例如，检查是否已创建配置文件、运行 CLI 命令），这种类型的探针会很有用。

```
     startupProbe:   # 启动探针
       exec:
         command:
         - sh
         - -c
         - "echo 'success' > /inited;"  
       failureThreshold: 3  # 失败多少次才算失败
       periodSeconds: 10   # 间隔时间
       successThreshold: 1 # 多少次成功算成功
       timeoutSeconds: 5 # 请求超时时长

```

> 验证shell脚本呢是否执行成功   -c 指定容器  -- 后面是命令

```
[root@k8smaster pods]# kubectl exec -it nginx-demo -c nginx -- cat /inited
success

```



#### 2.3.3  参数配置

- `initialDelaySeconds`：启动 liveness、readiness 探针前要等待的秒数。
- `periodSeconds`：检查探针的频率。
- `timeoutSeconds`：将探针标记为超时（未通过运行状况检查）之前的秒数。
- `successThreshold`：探针需要通过的最小连续成功检查数量。
- `failureThreshold`：将探针标记为失败之前的重试次数。对于 liveness 探针，这将导致 Pod 重新启动。对于 readiness 探针，将标记 Pod 为未就绪（unready）。



#### 2.3.4 pod的生命周期

<img src=".\imgs\image-20240518160020315.png" alt="image-20240518160020315" />

**pod的退出流程**

- Endpoint 删除pod 的IP地址
- Pod变成terminating状态
  - 变成删除中的状态后，会给pod一个宽限期，让pod去执行一些清理或者销毁的工作
  - 参数配置
    - terminationGracePeriodSeconds: 30 
      - containers:
- 执行PreStop的指令



> 二级 terminationGracePeriodSeconds

````yaml
spec: # 期望 pod按照这里的配置创建i
  terminationGracePeriodSeconds: 40  # 删除pod 等待的时间 
  containers:  # 对于pod中容器的描述
  
 # 验证成功 40秒 
[root@k8smaster cni]# time kubectl delete po nginx-demo
pod "nginx-demo" deleted

real	0m41.152s
user	0m0.020s
sys	0m0.036s
````





##### 2.3.4.1 preStop 的应用

> 配置文件

```shell
  9 spec: # 期望 pod按照这里的配置创建
 10   containers:  # 对于pod中容器的描述
 11    - name: nginx # 容器的名称 -代表数字
 12      image: nginx:1.7.9 # 指定容器的镜像
 13      imagePullPolicy: IfNotPresent # 如果本地没有就去远程拉
 14      lifecycle: #生命周期配置
 15        postStart:   #生命周期启动阶段 不一定在容器command之前运行
 16          exec:
 17            command:
 18            - sh
 19            - -c
 20            - "echo '<h1>pre stop </h1>' /usr/shar/nginx/html/prestop.html"
 21        preStop:     # 生命周期结束
 22          exec:
 23            command:
 24            - sh
 25            - -c
 26            - "sleep 50; echo 'sleep finished ....' >> /usr/shar/nginx/html/prestop.html"

```



> 验证  前置钩子生效了

```shell
[root@k8smaster pods]# kubectl create -f ningx_prestop.yaml 
[root@k8smaster cni]# kubectl get po -o wide
NAME         READY   STATUS    RESTARTS   AGE   IP               NODE       NOMINATED NODE   READINESS GATES
nginx-demo   1/1     Running   0          18s   10.244.185.195   k8snode2   <none>           <none>
[root@k8smaster cni]# curl 10.244.185.195/prestop.html
<h1>pre stop </h1>

```





### 2.4 label 和 selector

#### 2.4.1 label 标签

> 创建label 标签有两种方式 第一种 配置文件中配置 第二中使用命令 

- 配置文件

```
在各种资源的 sepc.metadata.labels 中进行配置
```





- kubectl

> 临时创建label

```
kubectl labe po [资源名称] key=value

[root@k8smaster ~]# kubectl label po nginx-demo author=wmt
pod/nginx-demo labeled
[root@k8smaster ~]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-demo   1/1     Running   0          20h   author=wmt,tyep=app,version=1.0.0

```



> 修改已经存在的标签

```
kubectl labe po [资源名称] key=value --overwrite

[root@k8smaster ~]# kubectl label po nginx-demo author=wangmengtao --overwrite
pod/nginx-demo labeled
[root@k8smaster ~]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-demo   1/1     Running   0          20h   author=wangmengtao,tyep=app,version=1.0.0

```



> 查看labe

```
# selector 按照label值查找节点
kubectl get po -A -l app=appName

#查看所有节点的labels
[root@k8smaster ~]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-demo   1/1     Running   0          20h   tyep=app,version=1.0.0

```



#### 2.4.2 selector 选择器

> 配置文件

在各种对象配置中 spec.selector 或其他可以写selector的属性中编写



> kubectl    
>
> 如何使用指令选择出对应的资源

```
[root@k8smaster ~]# kubectl get po --show-labels
NAME         READY   STATUS    RESTARTS   AGE   LABELS
nginx-demo   1/1     Running   0          20h   author=wangmengtao,tyep=app,version=1.0.0
[root@k8smaster ~]# kubectl get po -l tyep=app  # 配置敲错了type
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          20h
[root@k8smaster ~]# kubectl get po -l 'version in(1.0.0, 1.1.1, 1.1.2)'    # 跟sql语法一个意思
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          20h

[root@k8smaster ~]# kubectl get po -l  version!=1.1.1,author=wangmengtao #两个条件是与的关系
NAME         READY   STATUS    RESTARTS   AGE
nginx-demo   1/1     Running   0          20h

```



### 2.5 deployment 

> 无状态的应用



#### 2.5.1 创建

```
# 创建一个文件放配置文件
[root@k8smaster deployment]# pwd
/opt/k8s/deployment

# 创建deploy
[root@k8smaster deployment]# kubectl create deploy nginx-deploy --image=nginx:1.7.9
deployment.apps/nginx-deploy created
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           32s

# 关联  deployment是 replicaset 的升级
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           32s
[root@k8smaster deployment]# kubectl get replicaset
NAME                      DESIRED   CURRENT   READY   AGE
nginx-deploy-78d8bf4fd7   1         1         1       2m32s
[root@k8smaster deployment]# kubectl get po
NAME                            READY   STATUS    RESTARTS   AGE
nginx-deploy-78d8bf4fd7-gxl7p   1/1     Running   0          2m39s

#也可以使用 这个命名查看 这里的三个对应的是 RC RS deployment
[root@k8smaster deployment]# kubectl get po,rs,deploy --show-labels
NAME                                READY   STATUS    RESTARTS   AGE   LABELS
pod/nginx-deploy-78d8bf4fd7-gxl7p   1/1     Running   0          52m   app=nginx-deploy,pod-template-hash=78d8bf4fd7

NAME                                      DESIRED   CURRENT   READY   AGE   LABELS
replicaset.apps/nginx-deploy-78d8bf4fd7   1         1         1       52m   app=nginx-deploy,pod-template-hash=78d8bf4fd7

NAME                           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
deployment.apps/nginx-deploy   1/1     1            1           52m   app=nginx-deploy



```



> 获取yaml 配置文件

```yaml
[root@k8smaster deployment]# kubectl get deploy nginx-deploy -o yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    deployment.kubernetes.io/revision: "1"
  creationTimestamp: "2024-05-19T07:10:52Z"
  generation: 1
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
  resourceVersion: "15880"
  uid: 56206a14-29b6-49ac-9a8f-2a322653ac81
spec:
  progressDeadlineSeconds: 600
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        name: nginx
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Always
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
```

 删除不必要的配置

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:  #元信息
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1  #期望副本数
  revisionHistoryLimit: 10  #进行滚动更新后保留的副本数
  selector:  #选择器  用于找到RS
    matchLabels:  # 按照标签匹配
      app: nginx-deploy   #匹配的key 和 value
  strategy:  #更新策略
    rollingUpdate:   #滚动更新策略
      maxSurge: 25%  # 进行滚动更新时， ，更新的个数最多可以超过期望副本数的个数/比例
      maxUnavailable: 25%  #  进行滚动更新时， 最大不可更新比例
    type: RollingUpdate  #更新类型 滚动更新
  template:  #pod模拟
    metadata: # pod 的元信息
      labels:  # pod的标签
        app: nginx-deploy
    spec:  # pod 的期望信息
      containers: #pod容器
      - image: nginx:1.7.9  # 镜像
        imagePullPolicy: IfNotPresent #拉去策略
        name: nginx  # 容器名称
      restartPolicy: Always  #重启策略
      terminationGracePeriodSeconds: 30
```





#### 2.5.2 滚动更新

> 只要修改了 deployment 配置文件中的 template中的属性，才会触发滚动更新

```
 kubectl edit deploy nginx-deploy

[root@k8smaster deployment]# kubectl edit deploy nginx-deploy

```



> 修改副本数量为3

```shell
# 查看 deploy 有3个
[root@k8smaster deployment]# kubectl get deploy   --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE   LABELS
nginx-deploy   3/3     3            3           71m   app=nginx-deploy

# rs 只有一个 一个rs 管理三个deploy
[root@k8smaster deployment]# kubectl get rs --show-labels
NAME                      DESIRED   CURRENT   READY   AGE   LABELS
nginx-deploy-78d8bf4fd7   3         3         3       71m   app=nginx-deploy,pod-template-hash=78d8bf4fd7

# pod有3个  pod-template-hash=78d8bf4fd7 3个pod 的模板都是使用同一个
[root@k8smaster deployment]# kubectl get po --show-labels
NAME                            READY   STATUS    RESTARTS   AGE   LABELS
nginx-deploy-78d8bf4fd7-gxl7p   1/1     Running   0          71m   app=nginx-deploy,pod-template-hash=78d8bf4fd7
nginx-deploy-78d8bf4fd7-kx2p6   1/1     Running   0          49s   app=nginx-deploy,pod-template-hash=78d8bf4fd7
nginx-deploy-78d8bf4fd7-s49sn   1/1     Running   0          49s   app=nginx-deploy,pod-template-hash=78d8bf4fd7

```



> 此时修改  - image: nginx:1.7.9  修改为 1.9.1
>
>  UP-TO-DATE 为3了
>
> 查看rs 版本 有两条

```shell
[root@k8smaster deployment]# kubectl get deploy --show-labels
NAME           READY   UP-TO-DATE   AVAILABLE   AGE    LABELS
nginx-deploy   3/3     3            3           127m   app=nginx-deploy


# 查看rs 版本
[root@k8smaster deployment]# kubectl get rs --show-labels
NAME                      DESIRED   CURRENT   READY   AGE     LABELS
nginx-deploy-754898b577   3         3         3       2m27s   app=nginx-deploy,pod-template-hash=754898b577
nginx-deploy-78d8bf4fd7   0         0         0       128m    app=nginx-deploy,pod-template-hash=78d8bf4fd7


```



#### 2.5.3 回滚

> 查看历史版本

```
[root@k8smaster deployment]# kubectl rollout  history deployment/nginx-deploy
deployment.apps/nginx-deploy 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

```



> 查看这两个版本的区别 

```
# 版本号越大表示越新 
[root@k8smaster deployment]# kubectl rollout  history deployment/nginx-deploy --revision=2
deployment.apps/nginx-deploy with revision #2
Pod Template:
  Labels:	app=nginx-deploy
	pod-template-hash=754898b577
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

[root@k8smaster deployment]# kubectl rollout  history deployment/nginx-deploy --revision=1
deployment.apps/nginx-deploy with revision #1
Pod Template:
  Labels:	app=nginx-deploy
	pod-template-hash=78d8bf4fd7
  Containers:
   nginx:
    Image:	nginx:1.7.9
    Port:	<none>
    Host Port:	<none>
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

```



> 版本回退
>
> 本质就是通过RS的方向进行版本管理的

```
[root@k8smaster deployment]# kubectl rollout undo deployment/nginx-deploy --to-revision=1
deployment.apps/nginx-deploy rolled back
```

 

> 这个参数可以设计最多保留多少个版本

```
 revisionHistoryLimit: 10  #进行滚动更新后保留的副本数
```



#### 2.5.4 扩容和缩容

> 方案一

```shell
[root@k8smaster deployment]# kubectl edit deploy nginx-deploy 
#修改配置文件
spec:
  replicas: 1  #期望副本数
```



> 方案二

```shell
kubectl scale --replicas=6 deploy nginx-deploy

[root@k8smaster deployment]# kubectl scale --replicas=6 deploy nginx-deploy
deployment.apps/nginx-deploy scaled
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   6/6     6            6           158m
[root@k8smaster deployment]# kubectl scale --replicas=1 deploy/nginx-deploy
deployment.apps/nginx-deploy scaled
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   1/1     1            1           158m

```



#### 2.5.5 暂停和恢复

> 暂停

```shell
kubectl rollout  pause  类型  资源名称 
kubectl rollout  pause  deployment  nginx-deployment
```



> 恢复

```
kubectl rollout  resume  类型  资源名称 
kubectl rollout  resume  deployment  nginx-deployment
```





### 2.6 StatefulSet

> 有状态应用



#### 2.5.1 创建

```shell
# 创建一个文件夹 保存配置文件
[root@k8smaster statefulset]# pwd
/opt/k8s/statefulset
[root@k8smaster statefulset]# vim web.yaml
[root@k8smaster statefulset]# kubectl create -f web.yaml 
service/nginx created
statefulset.apps/web created

# 查看 statusfulSet 是否存在
[root@k8smaster statefulset]# kubectl get sts
NAME   READY   AGE
web    0/2     63s

# 查看服务是否存在
[root@k8smaster statefulset]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   26h
nginx        ClusterIP   None         <none>        80/TCP    91s


# 查看容器卷是否存在
[root@k8smaster statefulset]# kubectl get pvc
NAME        STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Pending                                                     9m51s

# 查看存储卷 创建失败了 资源也起不来
[root@k8smaster statefulset]# kubectl describe pvc www-web-0
Name:          www-web-0
Namespace:     default
StorageClass:  
Status:        Pending
Volume:        
Labels:        app=nginx
Annotations:   volume.alpha.kubernetes.io/storage-class: anything
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       web-0
Events:
  Type    Reason         Age                 From                         Message
  ----    ------         ----                ----                         -------
  Normal  FailedBinding  90s (x42 over 11m)  persistentvolume-controller  no persistent volumes available for this claim and no storage class is set


```



> yaml 文件

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels: 
    app: nginx
spec: 
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web  # StatefulSet 对象名称
spec:
  serviceName: "nginx"  # 使用哪一个service 来进行管理dns
  replicas: 2  #期望副本数
  selector:
    matchLabels:
      app: nginx
  template:  #pod模拟
    metadata: # pod 的元信息
      labels:  # pod的标签
        app: nginx
    spec:  # pod 的期望信息
      containers: #pod容器
      - name: nginx
        image: nginx:1.7.9  # 镜像
        ports: # 容器内部要暴露的端口
        - containerPort: 80 #具体暴露端口号
          name: web  #改端口配置的名字
        volumeMounts:  # 数据卷 
        - name: www  # 指定加载容器卷的名字
          mountPath: /usr/share/nginx/html  # 加载到容器中的那个目录
  volumeClaimTemplates:  # 数据卷模板
  - metadata:  #数据卷描述
      name: www
      annotations:
        volume.alpha.kubernetes.io/storage-class: anything
    spec:
      accessModes: ["ReadWriteOnce"]  # 访问模式
      resources:
        requests:
          storage: 1Gi # 存储资源
```



> 去重新编辑yaml 删除容器卷的配置
>
> 删除 svc sts pvc 的资源
>
> 重新创建

```shell
[root@k8smaster statefulset]# kubectl delete svc nginx
service "nginx" deleted
[root@k8smaster statefulset]# kubectl delete sts web
statefulset.apps "web" deleted
[root@k8smaster statefulset]# kubectl delete pvc www-web-0
persistentvolumeclaim "www-web-0" deleted

#创建
[root@k8smaster statefulset]# kubectl create -f web.yaml 
service/nginx created
statefulset.apps/web created

```

> yaml 配置文件

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels: 
    app: nginx
spec: 
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web  # StatefulSet 对象名称
spec:
  serviceName: "nginx"  # 使用哪一个service 来进行管理dns
  replicas: 2  #期望副本数
  selector:
    matchLabels:
      app: nginx
  template:  #pod模拟
    metadata: # pod 的元信息
      labels:  # pod的标签
        app: nginx
    spec:  # pod 的期望信息
      containers: #pod容器
      - name: nginx
        image: nginx:1.7.9  # 镜像
        ports: # 容器内部要暴露的端口
        - containerPort: 80 #具体暴露端口号
          name: web  #改端口配置的名字

```





> 测试其它服务能否访问得到

```shell
[root@k8smaster cni]# kubectl run -it --image busybox dns-test  /bin/sh
If you don't see a command prompt, try pressing enter.
/ # ping web-0.nginx
PING web-0.nginx (10.244.185.204): 56 data bytes
64 bytes from 10.244.185.204: seq=0 ttl=62 time=1.388 ms
64 bytes from 10.244.185.204: seq=1 ttl=62 time=0.554 ms
64 bytes from 10.244.185.204: seq=2 ttl=62 time=0.355 ms

# 查看dns的信息
/ # nslookup web-0.nginx
Server:		10.96.0.10
Address:	10.96.0.10:53

# 退出
exit 
```



#### 2.5.2 扩容缩容

> 扩容

```shell
[root@k8smaster ~]# kubectl scale sts web --replicas=5
statefulset.apps/web scaled
```



> 缩容

```shell
[root@k8smaster cni]# kubectl patch sts web -p '{"spec":{"replicas": 3}}'
statefulset.apps/web patched
```





#### 2.5.3 镜像更新

> 镜像更新，目前还不支持更新image，需要patch来间接实现 
>
> 滚动更新

```shell
# 给镜像打补丁
[root@k8smaster cni]# kubectl patch sts web --type='json' -p='[{"op": replace, "path": "/spec/template/spec/containers/0/image", "value": nginx:1.9.1}]'

# 验证 
[root@k8smaster cni]# kubectl rollout history sts web
statefulset.apps/web 
REVISION  CHANGE-CAUSE
1         <none>
2         <none>

[root@k8smaster cni]# kubectl rollout history sts web --revision=2
statefulset.apps/web with revision #2
Pod Template:
  Labels:	app=nginx
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>

```



> 灰度发布也叫金丝雀发布
>
> 目的：项目上线后产生问题的影响，降到最低

利用滚动更新中的partition 属性，可以实现简易的灰度发布的效果例如我们有5个 pod，如果当前 partition 设置为 3，那么此时滚
动更新时，只会更新那些序号>=3的 pod利用该机制，我们可以通过控制 partition 的值，来决定只更新其中一部分 pod，确认没有问题后再主键增大更新的 pod 数量，最终实现全部 pod 更新。



> 修改配置文件

```shell
kubectl edit sts web
```

> 配置信息

```yaml
  updateStrategy:
    rollingUpdate:
      partition: 0  # 灰度更新
    type: RollingUpdate
```





**删除更新** **OnDelete**

> 删除更新 跟名字一样 只有删除这个资源才会更新， 当edit这个文件中template模板不会生效，必须删除这个pod才会触发更新



```yaml
  updateStrategy:
    type: OnDelete
```



#### 2.5.4 删除

> 删除statefulset 有状态的删除和无状态的删除有些不一样



```shell
# 查看资源 po，sts， svc， pvc  下的资源情况， 本次没有配置pvc 所以没有

[root@k8smaster ~]# kubectl get po
NAME       READY   STATUS    RESTARTS        AGE
dns-test   1/1     Running   1 (2d18h ago)   2d18h
web-0      1/1     Running   0               2d18h
web-1      1/1     Running   0               2d18h
[root@k8smaster ~]# kubectl get sts
NAME   READY   AGE
web    2/2     5d20h
[root@k8smaster ~]# kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d23h
nginx        ClusterIP   None         <none>        80/TCP    5d20h
[root@k8smaster ~]# kubectl get pvc
No resources found in default namespace.
```



- 删除 statefulset 和 headless service

  - 级联删除，删除statefulset 时会同时删除pods

    ```shell
    kubectl delete sts web
    ```

    

  - 非级联删除，删除statefulset 时并不会删除pods

    ```shell
    kubectl delete sts web --cascade=fasle
    ```

    

- 删除service

```shell
kubectl delete svc nginx
```







### 2.7 DaemonSet

> *DaemonSet* 是一个**确保全部或者某些节点上必须运行一个 Pod的工作负载资源（守护进程），当有节点加入集群时， 也会为他们新增一个 Pod。**



- 集群守护进程，如`Kured`、`node-problem-detector`
- 日志收集守护进程，如`fluentd`、`logstash`
- 监控守护进程，如promethues `node-exporter`



> yaml 配置

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
 selector:
    matchLabels:  # 匹配标签
      app: logging
  template:
    metadata:
      labels:
        app: logging
        id: fluentd
      name: fluentd
    spec:
      containers:
      - name: fluentd-es
        image: agilestacks/fluentd-elasticsearch:v1.3.0
        evn:  #环境变量配置
          - name: FLUENTD_ARGS  #环境变量的key
            value: -qq   #环境变量的value
        resources:
          limits:
            memory: 200Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:  # 容器卷
        - name: varlog # 容器卷名字
          mountPath: /var/log # 将数据挂载到容器内的哪个目录
        - name: varlibdockercontainers # 和上面一样
          mountPath: /var/lib/docker/containers
          readOnly: true
      terminationGracePeriodSeconds: 30
      volumes: # 定义容器卷
      - name: varlog
        hostPath:
          path: /var/log  #和docker 的容器卷一样 像vue 的 v-model
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers

```



> 创建对应的yaml文件

```shell
[root@k8smaster daemonset]# pwd
/opt/k8s/daemonset
[root@k8smaster daemonset]# ls
fluentd-es.yaml

# 根据配置文件创建daemonset
[root@k8smaster daemonset]# kubectl create -f fluentd-es.yaml 
daemonset.apps/fluentd created

# 获取ds
[root@k8smaster daemonset]# kubectl get ds
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd   2         2         0       2            0           <none>          73s

# 分别在node1 和 node2上部署了一个

[root@k8smaster daemonset]# kubectl get po -o wide
NAME            READY   STATUS    RESTARTS        AGE     IP               NODE       NOMINATED NODE   READINESS GATES
dns-test        1/1     Running   1 (2d22h ago)   2d22h   10.244.249.9     k8snode1   <none>           <none>
fluentd-dq7km   1/1     Running   0               4m35s   10.244.249.11    k8snode1   <none>           <none>
fluentd-f5lkf   1/1     Running   0               4m35s   10.244.185.211   k8snode2   <none>           <none>

```



#### 2.7.1 节点选择器

> 先查看节点标签信息

```shell
# 获取节点选择器的信息
[root@k8smaster daemonset]# kubectl get nodes --show-labels
NAME        STATUS   ROLES                  AGE    VERSION   LABELS
k8smaster   Ready    control-plane,master   7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8smaster,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
k8snode1    Ready    <none>                 7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8snode1,kubernetes.io/os=linux
k8snode2    Ready    <none>                 7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8snode2,kubernetes.io/os=linux
```



> 添加节点标签

```shell
# 给节点1添加标签
[root@k8smaster daemonset]# kubectl label no k8snode1 type=microservices
node/k8snode1 labeled

# 查看节点标签 node1 有新的标签 type=microservices
[root@k8smaster daemonset]# kubectl get nodes --show-labels
NAME        STATUS   ROLES                  AGE    VERSION   LABELS
k8smaster   Ready    control-plane,master   7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8smaster,kubernetes.io/os=linux,node-role.kubernetes.io/control-plane=,node-role.kubernetes.io/master=,node.kubernetes.io/exclude-from-external-load-balancers=
k8snode1    Ready    <none>                 7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8snode1,kubernetes.io/os=linux,type=microservices
k8snode2    Ready    <none>                 7d3h   v1.23.6   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=k8snode2,kubernetes.io/os=linux

```



> 修改配置文件

```shell
[root@k8smaster daemonset]# kubectl get ds
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
fluentd   2         2         2       2            2           <none>          28m
[root@k8smaster daemonset]# kubectl edit ds fluentd
```



> spec 是在template下面的那个

```yaml
    spec:
      nodeSelector:
        type: microservices
```



> 修改后只剩一个了  并且在node1上

```
[root@k8smaster daemonset]# kubectl get ds
NAME      DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR        AGE
fluentd   1         1         1       1            1           type=microservices   33m
[root@k8smaster daemonset]# kubectl get po -o wide
NAME            READY   STATUS    RESTARTS        AGE     IP               NODE       NOMINATED NODE   READINESS GATES
dns-test        1/1     Running   1 (2d22h ago)   2d22h   10.244.249.9     k8snode1   <none>           <none>
fluentd-glv8q   1/1     Running   0               76s     10.244.249.12    k8snode1   <none>           <none>
web-0           1/1     Running   0               2d22h   10.244.185.209   k8snode2   <none>           <none>
web-1           1/1     Running   0               2d22h   10.244.249.10    k8snode1   <none>           <none>
```



#### 2.7.2 更新

> 通过 edit 打开配置文件 发现默认为滚动更新  kubectl edit ds fluentd

 ```yaml
   updateStrategy:
     rollingUpdate:
       maxSurge: 0
       maxUnavailable: 1
     type: RollingUpdate
 ```



> 但是建议使用 OneDelete

```yaml
  updateStrategy:
    type: OneDelete
```



### 2.8 HPA

> 前面有HPA的介绍



> yaml 配置文件 有资源限制

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        name: nginx
      restartPolicy: Always
      terminationGracePeriodSeconds: 30

```



先创建一个deploy

```
[root@k8smaster deployment]# kubectl apply -f nginx-deploy.yaml 
deployment.apps/nginx-deploy created
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   0/1     1            0           6s

```



> 修改配置文件 加入资源管理

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deploy
  name: nginx-deploy
  namespace: default
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      app: nginx-deploy
  strategy:
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
    type: RollingUpdate
  template:
    metadata:
      labels:
        app: nginx-deploy
    spec:
      containers:
      - image: nginx:1.7.9
        imagePullPolicy: IfNotPresent
        name: nginx
        resources:
          limits:  #定义资源上限
            cpu: 200m
            memory: 128Mi
          requests:
            cpu: 10m #定义资源下限
            memory: 128Mi
      restartPolicy: Always
      terminationGracePeriodSeconds: 30

```



> 替换

```shell
[root@k8smaster deployment]# kubectl replace -f nginx-deploy.yaml 
deployment.apps/nginx-deploy replaced
```



> 执行

```shell
[root@k8smaster deployment]# kubectl autoscale deploy nginx-deploy --cpu-percent=20 --min=2 --max=5
horizontalpodautoscaler.autoscaling/nginx-deploy autoscaled
```



> 此时自动就扩容到了2个了

```shell
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   2/2     2            2           2m40s

```



#### 2.8.1 安装Metrics Server

> Metrics Server是Kubernetes内置自动伸缩管道的一个可伸缩、高效的容器资源度量来源。



Metrics Server不是用于非自动伸缩的目的。例如，不要将其用于将指标转发给监视解决方案，或者作为监视解决方案指标的来源。在这种情况下，请直接从Kubelet /metrics/resource端点收集度量。

Metrics Server提供

- 在大多数集群上工作的单个部署
- 快速自动缩放，每15秒收集一次指标。
- 资源效率，为集群中的每个节点使用1毫秒的CPU内核和2 MB的内存。
- 可扩展支持多达5000个节点群集。
  



> 下载

```shell
# 如果每个wget 使用 yum install -y wget

[root@k8smaster k8s]# mkdir components
[root@k8smaster k8s]# cd components
[root@k8smaster k8s]# wget https://github.com/kubernetes-sigs/metrics-server/releases/download/v0.5.0/components.yaml
[root@k8smaster components]# ls
components.yaml
```



> 编辑这个配置文件

```yaml
containers:
- args:
  - --cert-dir=/tmp
  - --secure-port=443
  - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
  - --kubelet-use-node-status-port
  - --metric-resolution=15s
  - --kubelet-insecure-tls # 加上该启动参数 不验证证书
  image: ccr.ccs.tencentyun.com/mirrors/metrics-server:v0.5.0 # 国内集群，请替换成这个镜像
```



> 查看修改信息

```shell
[root@k8smaster components]# grep image components.yaml 
        image: ccr.ccs.tencentyun.com/mirrors/metrics-server:v0.5.0 # 国内集群，请替换成这个镜像
        imagePullPolicy: IfNotPresent
```





> 部署 metrics-server

```shell
# 修改 components.yaml 之后，执行以下命令，通过 kubectl 一键部署到集群：
[root@k8smaster components]# kubectl apply -f components.yaml 

```



> 查看是否安装成功

```shell
[root@k8smaster components]# kubectl get po --all-namespaces | grep metrics
kube-system   metrics-server-8f68868ff-86gzr           0/1     ContainerCreating   0                 92s
```



> 查看资源分配

```
[root@k8smaster components]# kubectl top pod
NAME                           CPU(cores)   MEMORY(bytes)   
dns-test                       0m           0Mi             
fluentd-glv8q                  1m           57Mi            
nginx-deploy-56696fbb5-bspkh   0m           1Mi             
nginx-deploy-56696fbb5-q4dkj   0m           1Mi             
web-0                          0m           1Mi             
web-1                          0m           1Mi      
```



> 查看这两个nginx中的labels       都有一个 app=nginx-deploy

```shell
[root@k8smaster deployment]# kubectl get po --show-labels
NAME                           READY   STATUS    RESTARTS     AGE    LABELS
dns-test                       1/1     Running   1 (7d ago)   7d     run=dns-test
fluentd-glv8q                  1/1     Running   0            4d1h   app=logging,controller-revision-hash=6c588598d8,id=fluentd,pod-template-generation=2
nginx-deploy-56696fbb5-bspkh   1/1     Running   0            2d1h   app=nginx-deploy,pod-template-hash=56696fbb5
nginx-deploy-56696fbb5-q4dkj   1/1     Running   0            2d1h   app=nginx-deploy,pod-template-hash=56696fbb5
web-0                          1/1     Running   0            7d     app=nginx,controller-revision-hash=web-6bc849cb6b,statefulset.kubernetes.io/pod-name=web-0
web-1                          1/1     Running   0            7d     app=nginx,controller-revision-hash=web-6bc849cb6b,statefulset.kubernetes.io/pod-name=web-1
```



> 创建一个 nginx-svc.yaml  

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  labels:
    app: nginx  
spec:
  selector: 
    app: nginx-deploy  # 这里的选择器选择上面的两个nginx 在service中不需要 mathLabels
  ports:
  - port: 80
    targetPort: 80
    name: web
  type: NodePort
---
```



> 创建pod

```shell
[root@k8smaster deployment]# kubectl apply -f nginx-svc.yaml 
```



> 查看这个service

```shell
[root@k8smaster deployment]# kubectl get svc
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
nginx-svc    NodePort    10.104.59.8   <none>        80:31916/TCP   44s

```



> 查看hap

```shell
[root@k8smaster deployment]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   0%/20%    2         5         2          2d1h
```





> 创建一个死循环去调用
>
> 分别去节点1 和节点2 去调用
>
> while true; do wget -q -O- [ip:port]> /dev/null ; done
>
> 这里的ip为节点内部IP， 端口为80 默认是80

```shell
[root@k8snode1 ~]# while true; do wget -q -O- http://10.104.59.8 > /dev/null ; done
```



> 再次查看hpa

```
[root@k8smaster deployment]# kubectl get hpa
NAME           REFERENCE                 TARGETS   MINPODS   MAXPODS   REPLICAS   AGE
nginx-deploy   Deployment/nginx-deploy   31%/20%   2         5         5          2d1h
```



> 查看deploy

```shell
[root@k8smaster deployment]# kubectl get deploy
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy   5/5     5            5           2d1h
```

