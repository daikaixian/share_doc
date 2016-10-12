---
title: 'Docker & Kubernetes Intro'
date: 2016-08-25 18:16:18
categories: Devops
tags: Kubernetes

---


## What is Docker

> Docker is the world's leading software containerization platform


## Container VS VMs

> Containers and virtual machines have similar resource isolation and allocation benefits -- but a different architectural approach allows containers to be more portable and efficient.

![docker](http://cdn4.infoqstatic.com/statics_s2_20160823-0357/resource/articles/docker-core-technology-preview/zh/resources/0731013.jpg)

<!-- more -->

## Orchestration Tools


### Why Need Orchestration

Orchestration : 本意是是指管弦乐中的配器法。

![管弦乐](http://img1.imgtn.bdimg.com/it/u=1680618254,696436313&fm=21&gp=0.jpg)

一场大型的交响乐的表演，需要众多乐手，比如小提琴手，鼓手，小号等等，但是更重要的是需要有一名出色的指挥。

一个大型的互联网应用，需要各式各样的服务器，比如缓存服务器，应用服务器，数据库服务器，负载均衡服务器等等，协同工作。当这里的‘服务器’精简成一个个的container, 为了更高效，更优雅的协同工作，同样需要有一名出色的指挥。Orchestration Tool就是这个Container乐团的指挥。

![网站架构图](http://7xsrzn.com1.z0.glb.clouddn.com/jiagou.png?imageView2/3/w/500/h/500)


### Why Kubernetes

Three Popular Orchestration 

 - [Docker Swarm](https://docs.docker.com/swarm/)
 - [Apache Mesos](http://mesos.apache.org/documentation/latest/)
 - [Google Kubernetes](http://kubernetes.io/)
 
 
|   候选     | 优点   |  缺点  |
|  :----:  |  :----: | :----:  |
| Swarm     | Docker Native Support. |   设计比较简单，作为编排工具还不够产品级，还需要一段时间的发展   |
| Mesos        | 已经成熟，文档齐全  |   无明显缺点,如果非要挑一个，那就是设计理念不如k8s优秀，社区支撑不如k8s强大。|
| Kubernetes   | 基于borg（被验证的大规模集群管理技术，google出品）,社区强大，设计理念优秀，与容器技术结合紧密，为业界所看好，被认为是未来的容器编排技术的主流。 |  学习曲线比较陡峭，文档质量偏低  |


结合米喜的实际情况，我们选择了Kubernetes作为编排工具。详细的技术选型过程参加：[内网资料：部署环境改造技术选型](http://192.168.0.214:8090/pages/viewpage.action?pageId=1507386)


另外，说Kubernetes文档质量低不是随便讲讲的：

![论k8s文档写的有多烂](http://7xsrzn.com1.z0.glb.clouddn.com/fuck8s.jpg-large)
 
### Kubernetes Concept Guide

 - **Cluster** : A cluster is a set of physical or virtual machines and other infrastructure resources used by Kubernetes to run your applications.
 - **Node** : A node is a physical or virtual machine running Kubernetes, onto which pods can be scheduled.
 
 - **Kubectl** : command-line interface to interact with Kubernetes.
 - **Pod** : the smallest deployable units of computing that can be created and managed in Kubernetes.A group of one or more containers is called a pod.
 - **Volume** : just a directory, possibly with some data in it, which is accessible to the containers in a pod.
 - **Labels** :  key-value pairs.
 - **Deployments** : A Deployment object defines a Pod creation template  and desired replica count.
 - **Service** : A service defines a set of pods and a means by which to access them, such as single stable IP address and corresponding DNS name.
 - [More](http://kubernetes.io/docs/user-guide/#concept-guide)
 

## TODO For Medishare


- 基于k8s的部署

![基于k8s米喜架构](http://7xsrzn.com1.z0.glb.clouddn.com/k8sformedishare.png?imageView2/3/w/600/h/600)




- CI & CD

![趋势科技持续部署方案](http://cdn4.infoqstatic.com/statics_s2_20160823-0357/resource/news/2016/08/sunqing-docker-kubernetes-CICD/zh/resources/1.png)




## Related Concepts

- [DevOps](https://zh.wikipedia.org/wiki/DevOps)
- [MicroService](http://martinfowler.com/articles/microservices.html)
- [Continuous Deployment](http://www.infoq.com/cn/articles/continuous-deployment-containers)



## Reference

- [深入浅出Docker（一）：Docker核心技术预览
](http://www.infoq.com/cn/articles/docker-core-technology-preview)
- [Docker编排](http://dockone.io/article/861)
- [FROM CONTAINERS TO CONTAINER ORCHESTRATION](http://thenewstack.io/containers-container-orchestration/)
- [k8s实施阶段性回顾](http://codingwater.org/2016/08/09/%E4%BD%BF%E7%94%A8Kubernetes%E6%94%B9%E9%80%A0%E7%B1%B3%E5%96%9C%E9%83%A8%E7%BD%B2%E7%8E%AF%E5%A2%83-%E9%98%B6%E6%AE%B5%E6%80%A7%E6%80%BB%E7%BB%93/)
- [Kubernetes Volume](http://kubernetes.io/docs/user-guide/volumes/)
- [趋势科技基于Docker和Kubernetes的持续部署实践](http://www.infoq.com/cn/news/2016/08/sunqing-docker-kubernetes-CICD)
- [《Kubernetes与云原生应用》系列之Kubernetes的系统架构与设计理念](http://www.infoq.com/cn/articles/kubernetes-and-cloud-native-applications-part01)
- [基于kubernetes构建Docker集群管理详解](http://blog.liuts.com/post/247/)

## SlideShare

[https://drive.google.com/file/d/0Bx57XsAahdV4NWdDRHdvcC1SdTQ/view](https://drive.google.com/file/d/0Bx57XsAahdV4NWdDRHdvcC1SdTQ/view)


