# ARTS

## 0622-0628

做了 ART，没有 S，先培养起习惯，后面再加大学习量。

每周寄语: 比的是智力，拼的是体力，赢的是心理。

### A.1

https://mp.weixin.qq.com/s?__biz=MzkzOTAyMjEyMg==&mid=2247483653&idx=1&sn=b27bb772276bbc40c1b236bad681c791#rd

### R.1

这周读了 k8s 官方文档。

我目前的工作(基础架构) 与 k8s 将会越来越密切，同事没有义务做培训，自己提前预习。

k8s 有[官方中文版文档](https://kubernetes.io/zh/docs/home)，看起来不是机翻的，np，据说是因为这项技术在中国的应用更广泛，国外的中小公司基本都直接上云了(像 netfilx 这种在 spring 领域贡献了超级多组件的大公司也全在云上)。

PS: 说实在的云服务商是不值得信任的，阿里云的 sla 是 3 个 9，赔偿也只有代金券而已，[参见](http://terms.aliyun.com/legal-agreement/terms/suit_bu1_ali_cloud/suit_bu1_ali_cloud201909241949_62160.html?spm=a2c4g.11186623.2.11.7ec01d94ooNNA3)

除了 doc，[blog](https://kubernetes.io/zh/blog) 也值得一看。

#### 阅读笔记

只看完了入门 & 概述

初步理解：k8s 就是“软件定义硬件” or “软件定义基础设施”，类似 sdn，将硬件用软件定义后，对硬件的操作(运维)就变成了规范化的、可程序化的，将极大的减轻人肉运维的压力，让几十人管理几千实例成为可能。

##### 杂

docker、lxc、lxd 三种容器，Docker 已经成为容器化的事实标准。参考[文献1](https://www.zhihu.com/question/268288911)、[文献2](https://blog.csdn.net/zhengmx100/article/details/79415742)、[文献3](https://www.upguard.com/articles/docker-vs-lxc)

Kubernetes 版本号格式为 x.y.z，大、小、补丁，小版本 3 个月一发，只维护 9 个月。

k8s 据说有 100w 行代码

##### 本地测试环境搭建

[minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/) or [microk8s](https://microk8s.io/) for mac/linux。microk8s on linux 上是用的 snap，更轻量。但在 mac 上，都是建了个 vm，在 vm 里跑 k8s，都比较费资源。

不要用 win 了

###### minikube

minikube start | stop | status | delete
minikube dashboard | service

###### microk8s

有墙，没安上

##### 总览

* Kubernetes 组件
    * 控制平面组件(除 etcd 都运行在 Master 节点)
        * Master 节点：是指管理集群状态的一组进程的集合，通常这些进程都跑在集群中一个单独的节点上(可以多 master for HA)，称为 master 节点
        * kube-apiserver：控制面的前端，提供 Kubernetes API for kubectl、kubelet、kube-proxy
        * kube-controller-manager：一组 controller 的集合
            * NodeController
            * ReplicationController
            * EndpointsController
            * TokenController
        * kube-scheduler：调度器
        * etcd：作为数据库
    * Node 组件(集群中的每个 Node 节点都运行这两个进程)
        * Node 节点：node 节点是用来运行你的应用和云工作流的机器。master 节点控制所有 node 节点；你很少需要和 node 节点进行直接通信。
        * kubelet：node 节点上的代理，和 master 节点进行通信，保证容器运行
        * kube-proxy：一种网络代理，维护节点上的网络规则。
* Kubernetes 基本对象
    * Pod
    * Service
    * Volume
    * Namespace
* Kubernetes 高级对象 Controller
    * Deployment
    * DaemonSet
    * StatefulSet
    * ReplicaSet
    * Job

![Kubernetes 集群](https://d33wubrfki0l68.cloudfront.net/7016517375d10c702489167e704dcb99e570df85/7bb53/images/docs/components-of-kubernetes.png)

### T.1

1. 为了能更多的获得激励(心理上的)，使得学习可以持续。建议大家[建立自己的公众号](https://mp.weixin.qq.com). 我组织 ARTS 也是希望可以从中获各正向反馈。
    1. 学习不一定要抽出一整块时间，可以在平时的零碎时间刷题、阅读、记录想法，然后发在公众号上(按专辑整理)，周末抽时间整理成一篇汇总就好，这样压力不会太大
        1. share 可能会需要单独的一块时间，一个偷懒的办法是上周 review 的啥，下一周再 share 它。
1. 初期的学习量可以不用太大，先培养起习惯，后面再加大学习量。
1. 分享一个小工具，github.com/mhadam/readable-thrift，用于将二进制 Thrift 消息转化为 json，最近在做公司内 基于 thrift 的 rpc 框架相关工作，遇到了协议二进制解析相关的问题，这个工具帮了大忙。
    1. PS：我分享的这个链接并不是工具的原作者，原作者的 repo 的代码编译不过 (gradle 的，不太懂)，这个人修了一把可以编过
