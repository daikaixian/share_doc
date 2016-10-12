---
title: 使用Kubernetes改造米喜部署环境--阶段性总结
date: 2016-08-09 09:07:31
tags: Kubernetes
categories: DevOps

---

又是一个月没有更新博客了，白白的烧了62块钱的服务器租赁费。倒不是不想更新，实在是因为正在解决一个颗粒度很大的技术问题。已经断断续续被这个问题折磨了快两个月的时间，昨天晚上终于取得了阶段性成果，所以赶紧来post一篇。希望今天能早点下班，毕竟七夕。

![七夕之歌](http://img.xiazaizhijia.com/uploads/2016/0808/20160808105639891.png)


<!-- more -->


## 现状与目标

公司目前项目的开发和测试是在内网的物理机上进行的。体量暂时也还不大，所以预发和生产环境挂在阿里云上。Leader结合公司未来的业务发展需求和目前行业的大趋势（比如Docker,DevOps,MicroService等等），跟我定了这么个任务。希望能在2.0项目上线之前，对预发环境的部署方式进行改造，主要就是使用容器技术将应用容器化，并使用编排工具将这些应用管理起来。

容器技术Docker已经是如雷贯耳，几乎成为容器技术的代名词。所以其实没什么好犹豫的，不过值得一提的是CoreOS推动的一款类Docker的开源容器引擎技术，Rocket.CoreOS这样做的原因是因为他们认为Docker已经背弃了提供“一个标准的容器架构”的初衷.但是就我的需求，务实的选择，当然是Docker.

至于Orchestration Tool.,经过详细的技术选型和调研，最终在Docker Swarm、Apache Mesos和Google Kubernetes之间选择了Kubernetes,作为编排工具来管理Docker.

## 艰难的实施过程

首先我其实是一个开发,一个Dev.但是整个实施过程中，涉及到写代码的工作其实是非常之少的，大多数的工作集中在软件的安装，环境的配置，还有镜像的下载。。。。是的，说起这个镜像的下载我就坐不住，有些镜像翻个墙下载，几MB十几MB还行，有些动不动就是几十MB几百MB，一等就是快一个小时。能下到的也就算了，有些镜像放在gcr.io下面，根本拉不动。没有镜像，所有的操作都会被卡住，心情就是操蛋。好在后面学会了用DaoCloud和时速云之类的东西，总算是逃脱了GFW的拦截。

总结起来就是，我认为这个工作让运维同学来做可能更合适且高效。但是都在讲DevOps，所以也没什么好抱怨的了，就当是扩充技能栈了。

这里罗列一下在实施过程中踩过的坑和做过的事，帮助理清思路：

-  去机房给物理机安装操作系统，配置网络，连接交换机。CentOS上有一个设置网络的GUI工具:nmtui,可以帮很多忙.另外本来打算去划分vlan的，但是电脑连交换机的console口使用超级终端有点问题，所以那天就没有划分，但是两台服务器还是可以通过内网ip互相ssh的。

- 使用grale 将一个spring boot应用打包Docker Image.[详情请戳这里，亲测好用](https://spring.io/guides/gs/spring-boot-docker/)
- 为了在centos上使用minikube,需要先[安装VirtualBox](http://tecadmin.net/install-oracle-virtualbox-on-centos-redhat-and-fedora/),也是个麻烦的过程。踩了yum  update  linux kernel的一些坑

- 一开始在mac本上尝试着跑docker，所以用过docker-machine.后来在centos碰到’Cannot connect to the Docker daemon. Is the docker daemon running on this host?‘的问题的时候，以为在centos上也要安装这个docker-machine.其实只是因为我的docker service挂掉了，service docker restart一下就能解决问题。

- docker官网上教我在centos上安装docker-engine.后来k8s官网教我在centos上安装k8s的时候用'yum install kubernetes',结果报告docker版本冲突。十分困惑不解。后经朋友指导和对docker了解更多之后，才知道docker-engine和docker-machine都是docker项目的分支功能，centos安装docker请认准'yum install docker'

- 因为下载镜像超级慢，所以尝试搭建自己的仓库。尝试过[DTR](https://docs.docker.com/docker-trusted-registry/)和[Registry Server](https://docs.docker.com/registry/deploying/),前者直接失败，后者废了九牛二虎之力终于可以在内网各节点间不使用证书访问了，结果后面因为重装了好几次docker,宣告白弄了。。。。现在的状态依然是没有使用仓库，但想一想后面肯定还是要弄起来的。

- terminal下载东西要翻墙的话，挂个http_proxy是个不错的选择。

- 下载镜像如果嫌太慢或者根本拉不动，可以走DaoCloud或者时速云，拉下来之后重新tag一下，就可以使用了。

- 看过好几个在centos上不靠谱的教你如何搭建k8s集群的文章，找到[这篇十分靠谱的](http://severalnines.com/blog/installing-kubernetes-cluster-minions-centos7-manage-pods-services)。因为前前后后在不同机器上重新安装过好多次，所以可以说是屡试不爽。

- 有一次在一组新的机器上搭建k8s集群，搭建好之后在master上执行‘kubectl get nodes’ 没有任何的return信息。经排查发现是因为之前在这台机器上跑过minikube,把kube-context设置为了minikube,所以在minikube这个上下文信息下get不到node是很正常的。但是也不知道新安装的集群的context是什么，不知道如何切换回来。有一种做法是删除~/.kube/config文件。

- 因为缺乏经验，在将要上线运行的服务器上胡乱的修改了很多配置文件。导致一下软件的基本使用都有问题。比如在使用docker的过程中就碰到诸如'unit docker.socket not found '，'bad
  certificate'之类的问题。google了半天也没有很好的解决，于是说想办法重装吧。但是一个不合格的运维起初以为只需要 'yum  remove
  xxx'就可以完成卸载了。其实并没有，这样的卸载很不干净！于是我想了个粗暴的方法，在根目录下使用find命令，递归去查询所有跟‘docker’有关的文件，确认之后全部删掉再重装，就fix了。当然如果一些配置写进了文件，比如/etc/profile之类的，而且后面又忘记了，就只能随缘了。。。把这个烦恼跟一个做运维的朋友反映过之后，他建议我以后不要在生产服务器上乱尝试，而是找那些不重要的机器，比如阿里云按需付费的云主机，把流程测试通过了，再搬到生产服务器上去，或者预先使用虚拟化的方式对一台主机进行一虚多。拿虚拟机做尝试，发现玩坏了，直接废弃掉，重装一台虚拟机。

-  至于k8s官网上的文档，不得不说写的实在差。跟着文档走，到处走不通的体验实在是糟糕透了，简直是要把我弄疯掉。我自身的原因当然也有一些，但是一个hello-world我真的是从第一次看到到顺利跑完居然中间隔了一个月还多。。。下面这张图应该可以表达众多使用者对k8s官方文档共同的心声吧。
  
   ![论K8s官网的文档写的有多烂](http://7xsrzn.com1.z0.glb.clouddn.com/fuck8s.jpg-large)

- 当被k8s官网文档折磨的死去活来的时候，花了一天的时间把Docker Swarm和Apache Mesos的入门文档都跑了一遍，不要太顺畅。跑完之后做了一个更感性的对比。
  ![对比Mesos,k8s,Swarm](http://7xsrzn.com1.z0.glb.clouddn.com/k8smesos.jpg-large)

- 还有一些方法论性质的感悟。比如这次施工，学习资料基本全英文。感谢自己今年4，5月份在学校养成了好习惯，阅读英文文档除了慢了点，没有什么太大的问题。另外在碰到问题的时候，除了stackoverflow，github的issues讨论列表也是一个可以解决问题的地方。不过更多的是讨论过程，正解命中率相对较低。说到stackoverflow,小小的开心了一下的是首答换来了2个赞同，reputation值终于超过了15，可以给别人的答案点赞了。好爽。

- 说到方法论，Leader给我灌输的‘谋定而后动’应该也要算一个。百度脑图是一个不错的分析工具，帮助理清思路。想清楚了再动手。

- 刚刚说到在google里面找答案。突然又记起来另一个非常非常重要的东西。那就是--日志！日志！日志！回顾这不长的‘运维工作经历’，觉得跟开发最相通的地方就在于发现不对劲了，找日志，然后拿日志中的错误信息当keyword去google.但是linux系统记录日志的方式和我们平时常见的应用系统又有一些差别，碰到异常情况但是找不到具体日志，是我在施工过程中最痛苦的经历之一。要了解linux的脾气，可能还是要用时间来磨炼吧。


## 修整一下再继续 
 
  还有一些零碎的事情，一下子可能想不全了，所以就先吐槽这么多。而且这也才完成阶段性的目标，即将公司的主应用docker化并用k8s集群管理起来。后面还要做的事可多了：

  - 数据库docker化
  - 集群组建方式调优
  - k8s正式上线，测试，预发，生产环境部署方式。实现CI,CD.
  - 实践微服务架构
  - ......

要学的东西就更多了。不过先继续深入理解Docker和K8s应该是首当其冲的工作。
   
![Docker](http://img4.imgtn.bdimg.com/it/u=1326576917,4008009621&fm=21&gp=0.jpg)
     
![K8s](http://img2.imgtn.bdimg.com/it/u=533468645,120501819&fm=21&gp=0.jpg)

