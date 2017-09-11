---
layout: post
cover: 'assets/images/new-pic-16.jpg'
title: 'k8s实践日记之安装篇'
date:   2017-05-22 17:45:00
tags:   docker
subclass: 'post tag-test tag-content'
categories: 'casper'
navigation: True
logo: 'assets/images/ghost.png'
---

其实很早就想深入了解一下容器编排工具，但是由于各种各样的事情，被耽搁了很久。最近就抽时间认真学习了一下kubernets（K8S）,虽然对其原理还未深入的了解，但是从初步掌握的情况来看，真的是一个非常牛的框架，本系列就先从安装篇开始（Centos7下）。

<h4>关于标准安装方式</h4>
其实就是使用kubeadm进行安装，k8s的核心组件全部容器化
https://kubernetes.io/docs/getting-started-guides/kubeadm/
安装之前，需要注意的地方：
- 限制条件，在官方文档中放在最后了，就是Limitations部分，先进行着部分的配置
- 修改/etc/hosts文件，添加本机非lo地址和所对应的名称

<h4>安装过程中存在的问题</h4>
安装过程中存在很多问题，一些比较简单的，稍微查找下资料就可以解决，但是有些却要费一些功夫。除了官方提供的安装教程，网上也有很多网友自发编写的，比如下方的安装教程2.
安装教程2：
http://blog.frognew.com/2017/04/kubeadm-install-kubernetes-1.6.html
在安装教程2中介绍的已经很详细了，一般情况下按照教程安装就可以。
但有以下两点需要说明以下：

1、节点网络选择。
关于pod之间网络的选择，我自己选择的是weave，当然是用flannel也可以。说一下我是用weave遇到的问题。目前我使用的k8s 1.6以上版本，所以选择weave网络的时候，也应该选择对应的版本，进行安装。很多网络的教程用的还是k8s 1.5版本，所以切不可照搬。
{% highlight powershell %}
1.5版本：  
kubectl apply -f https://git.io/weave-kube
1.6版本：  
kubectl apply -f https://git.io/weave-kube-1.6
{% endhighlight %}
选择错误的话，kubedns启动存在问题

2、出现的错误信息。
正常启动后，kubedns容器中错误提示如下：
reflector.go:199] k8s.io/dns/vendor/k8s.io/client-go/tools/cache/reflector.go:94: Failed to list *v1.Endpoints: Get https://10.0.0.1:443/api/v1/endpoints?resourceVersion=0: dial tcp 10.0.0.1:443: getsockopt: no route to host

通过查看网友关于此类问题的描述，初步定位于iptables的问题，通过查看iptables可以看到关于网关端口转发被禁，不知道什么原因导致，如果单条记录删除，k8s可自行恢复。初步修复无果。后来听网友推荐，使用清空iptables链，启动正常

解决方案地址：
https://github.com/kubernetes/kubernetes/issues/44750

<h4>最官方Demo实例安装错误</h4>
节点加入成功后，运行demo示例提示namespace错误
{% highlight powershell %}
运行的命令如下：
kubectl create namespace sock-shop
kubectl apply -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"

修改为：
kubectl delete namespace sock-shop
kubectl delete -n sock-shop -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
kubectl create -f "https://github.com/microservices-demo/microservices-demo/blob/master/deploy/kubernetes/complete-demo.yaml?raw=true"
{% endhighlight %}

有关此问题的解决方案链接
https://github.com/microservices-demo/microservices-demo/pull/702
https://github.com/kubernetes/kubernetes.github.io/issues/3214

<h4>安装Heapster 、Influxdb、Garafana</h4>
参考文章：
http://www.jianshu.com/p/60069089c981
需要注意的地方：
- 重新修改端口映射，可以使用外部ip访问
- 添加external ip字段，否则无法访问

英文权威指南：
https://github.com/kubernetes/heapster/blob/master/docs/influxdb.md

<h4>监控部署异常排查</h4>
Heapster的错误日志如下：
E0519 07:06:59.731558       1 reflector.go:203] k8s.io/heapster/metrics/processors/namespace_based_enricher.go:84: Failed to list *api.Namespace: the server does not allow access to the requested resource (get namespaces)

E0519 07:07:00.050170       1 reflector.go:203] k8s.io/heapster/metrics/sources/kubelet/kubelet.go:342: Failed to list *api.Node: the server does not allow access to the requested resource (get nodes)

原因定位：初步判断是RABC角色机制导致的这个问题
错误的解决办法：

{% highlight powershell %}
kubectl create clusterrolebinding add-on-cluster-admin \
    --clusterrole=cluster-admin \
    --serviceaccount=kube-system:default
{% endhighlight %}
关于此问题的英文解决方案说明：
https://github.com/kubernetes/kubeadm/issues/248

<h4>结束语</h4>
本篇文章是kubernets容器编排系列的第一篇文章，主要介绍了安装过程中踩的一些坑，以及一些注意事项。每个人在安装过程中可能遇到的问题都不太一样，遇到问题并不可怕，沉下心来，分析错误的原因，总会找到解决方法。


