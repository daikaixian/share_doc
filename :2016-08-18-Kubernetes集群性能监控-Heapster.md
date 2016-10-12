---
title: Kubernetes集群性能监控--Heapster
date: 2016-08-18 20:28:46
tags: Heapster
categories: DevOps

---

最近也是历经艰辛，搭建起了Kubernetes集群的性能监控工具Heapster,用起来感觉实在高大上。而且慢慢的，熬过了最开始最痛苦的开发转运维的过渡期之后，越来越能get到当运维的快感了。

![Heapster in Kubernetes](http://image.slidesharecdn.com/artemzhurbila-dockerclusterssolit2015-150313153240-conversion-gate01/95/artem-zhurbila-docker-clusters-solit-2015-31-638.jpg?cb=1426261060)

<!-- more -->


## Why Heapster

虽然前期上手K8s很痛苦，但是在体验了几次成功的集群搭建之后，深感这东西确实厉害。厉害之处主要体现在设计理念的优秀和API封装的优雅。简单几行命令即可指挥一大波机器去建起可用性很高的应用集群。然而API封装的越好，作为集群管理人员对内部细节知道的也就越少，集群健康情况和资源使用率等等对于集群管理者也是一个黑盒子。所以有效的利用监控工具可以帮助管理者使得集群达到一个性能更好的状态。

好吧以上是我个人的观点，下面还是看看官方文档的说辞。

> Understanding how an application behaves when deployed is crucial to scaling the application and providing a reliable service. In a Kubernetes cluster, application performance can be examined at many different levels: containers, pods, services, and whole clusters. As part of Kubernetes we want to provide users with detailed resource usage information about their running applications at all these levels. This will give users deep insights into how their applications are performing and where possible application bottlenecks may be found. In comes Heapster, a project meant to provide a base monitoring platform on Kubernetes.

## 踩坑路径

官网给的部署教程可谓十分简单，一行命令就完事了。然而就我的部署经历来看，我觉得大多数同学可能都不会那么一帆风顺的靠一行命令就能收工。以下便是我这次踩过的坑。

### imagePullPolicy

官网给的那几个yaml文件中，指定了需要使用到的镜像文件。按照以前的经验，从gcr拉镜像几乎是拉不动的，所以提前通过时速云那边把镜像拉到了本地仓库中，并完成了tag操作。然而在创建pod和service的时候依然报错PullImageError.仔细看了一下yaml文件，发现有一行imagePullPolicy: Always.这个值代表了每次创建都会去远程仓库拉取最新的镜像，并不care本地是否已经有需要的镜像。

解决这个问题，只需要把imagePullPolicy的值指定为IfNotPresent即可。

### SkyDNS

镜像问题解决后，接下来是DNS的问题。
Heapster由两个pod组成，其中heapster和grafana-influxDB之间的通信需要依赖DNS的自动发现,而不是直接通过ClusterIP。但是我之前并没有在本地的K8s集群上配置SkyDNS,所以又被这个问题卡住转去配置SkyDNS.

关于Kubernetes DNS配置的问题可以参考[DNS in Kubernetes](https://github.com/kubernetes/kubernetes/tree/v1.0.1/cluster/addons/dns)和[Kubernetes部署DNS-centos7](http://www.pangxie.space/docker/735)这两篇文章。

![SkyDNS](http://image.slidesharecdn.com/servicediscoveryopensourcemeetupapril162016-160417092151/95/service-discovery-using-etcd-consul-and-kubernetes-22-638.jpg?cb=1460885346)

### 版本遗留bug

DNS启动成功后，pod的Status依然处于Error状态，查看日志，看到了'The config file /etc/kubernetes/kubeconfig/kubeconfig does not exist'这行错误信息。首先该文件确实不存在，但是为什么需要这个文件了，我也不知道。然后就google看了几乎所有能找到的相关的讨论，因为并不多。尝试了好几个可能的解决办法之后，就快要决定放弃。。。然后在这篇[讨论](https://github.com/kubernetes/heapster/issues/320)的最下面看到一个Kubernetes Member的回答，说这个版本的Heapster已经弃用了，请使用1.0.2版本。。。

怀着一丝希望做了尝试，把yaml中指定的heapster镜像的版本从canary换到了最新的稳定版v1.1.0.然后重新创建pod和service，Status 变成了Running!

### Configuring sources

Status变成Running并不代表应用可以正常访问了，习惯性的看一下log会发现'var/run/secrets/kubernetes.io/serviceaccount/token: No such file or directory'的异常。

这个异常指向的问题大意是说Heapster与K8s Master的连接有两种方式，一种是https的，一种是http的。通过https的方式连接需要使用一些证书，如果找不到这些证书的话就会抛出上述异常。通过http的方式就相对简单一点，当然也不够安全。https的搭建方式可以参考[这篇文章](http://blog.dingmingk.com/blog/kube_monitor.html).

想直接使用http方式来解决问题的则通过[官方文档](https://github.com/kubernetes/heapster/blob/master/docs/source-configuration.md)找到答案。

### Env variables bug

一切就绪后，用NodePort的方式暴露grafana的service ip。然后使用浏览器访问时发现无法正常加载css和png文件。所以整个页面就是下面这个丑样子。

![api error](http://7xsrzn.com1.z0.glb.clouddn.com/enverror.jpg)

经排查是influxdb-grafana-controller.yaml中定义了一个叫做GF_SERVER_ROOT_URL的环境变量。根据官网的解释，直接删掉就好了。

![gf](http://7xsrzn.com1.z0.glb.clouddn.com/gf.png)


### Grafana Tutorial

踩完上面的坑就看到了这个让我兴奋到窒息的页面了。。。

![grafana](http://7xsrzn.com1.z0.glb.clouddn.com/grafana.png)

grafana的使用教程可以参考这个[视频](https://www.youtube.com/watch?v=sKNZMtoSHN4&index=7&list=PLDGkOdUX1Ujo3wHw9-z5Vo12YLqXRjzg2)。


## 总结

这里记下的只是我解决问题的过程。其实很多东西的原理，问题更深层次的原因，我在写的过程中会问自己是不是真的弄懂了，是不是真的深有体会了。答案显然是没有。。。

原因可能有这么几个，第一我不是专业的运维，很多东西理解起来可能需要比较多的基础运维知识做辅助。第二则是要消化的东西太多，我可能会把时间精力集中在自己最感兴趣的那一两个焦点上，其他的只要开始正常Running,也就不想再深入更多。

其实内心觉得这样的状态并不是好的，如果要达成自己心里的野心和期望，或许该更专注，更深入。

不过好在能坚持写博客，多多少少也会逼着自己把一些事弄的更清楚一些。一个月60多块的租赁费肯定有很多办法可以省下来，但是用这60多块钱督促自己坚持写两到三篇技术博客，我觉得还是挺值得的。




