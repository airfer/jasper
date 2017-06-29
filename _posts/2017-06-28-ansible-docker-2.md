---
layout: post
cover: 'assets/images/new-pic-17.jpg'
title: '基于Ansible && Docker的分布式系统(下)'
date:   2017-06-28 16:28:00
tags: summerize
subclass: 'post tag-test tag-content'
categories: 'casper'
navigation: True
logo: 'assets/images/ghost.png'
---

<h4>前言：</h4>
在上一篇中，我们主要从Poster入手，然后讲述了为何进行这样的技术选型，分别从docker和ansible两个方面来阐述了原因，本篇的重点在于分析集成于镜像中的Unicorn工程，以及playbook脚本的编写。

<h4>一、Unicorn工程</h4>
之所以叫Unicorn这个名称，是因为在整个系统的构建，其扮演者最重要的角色。Unicorn工程的目录结构如下图所示：

![](/images/unicorn/directory.jpg)

该工程主要分成5个部分，每个模块仅从其字面意思应该就可以知道大概了，这里简单的说一下

- APP   ：这个应用的主目录，主要的功能就是启动worker，接收任务，执行，写入数据
- Conf  : 这个目录下主要放置系统相关的配置文件，其可配置的内容包括broker队列的相关信息，任务类型信息，Hook脚本配置信息，结果输出类型信息等
- Lib   ：该目录主要用于存放共有调用库，目前主要放置三类，一类为broker driver相关库，比如结果写入Mongodb，或者写入Mysql，或者写入文件。第二类为配置文件解析库，用于解析配置文件
- Script: 该目录下主要放置shell脚本和python脚本，供主程序调用
- Log   : 主要放置运行过程中的日志文件

该工程的处理流程如下所示：

![](/images/unicorn/flow.jpg)

将该流程可以简单描述为 加载并解析配置文件 --> 获取broker队列等相关信息 --> 启动worker --> 执行任务  --> 结果数据处理。

其中需要对Script模块进行解释一下，这个模块存在的目的在于worker执行之前，以及执行完写入数据之前对数据进行一些必要的操作，包括清除脏数据，筛出符合条件的数据等。由于采用的是插件话设计，在APP主程序不需要变动的情况下，只需要变动conf文件，以及添加script脚本就可以完成上述操作。这给后续一些数据预处理操作提供了接口。


<h4>二、Ansible的应用</h4>
这个版本区别于上一个版本的地方主要在于Ansible的使用。用一句话概况的话，Ansible使得整个部署更可控，更加规范化。
之前的所有操作，包括物料数据同步、用例case同步、docker管理全部都是通过shell脚本来完成的，这就造成了整个系统和业务的耦合度很高。如果通过Ansible的Playbook脚本的话，将各个部分拆成独立的模块，然后不同的业务分别调用共用的模块，可以很轻松的做到一套服务一套脚本。playbook的处理流程如下所示：

![](/images/unicorn/flow2.jpg)

该流程概况的已经非常清楚了，就不多说了。关于如何使用Ansible来启动docker，其实主要是使用Ansible的docker-image和docker-container模块，下面是playbook脚本中关于启动容器的代码示例：

![](/images/unicorn/code.jpg)

<h4>三、结语</h4>
本篇主要讲述了整个工程的处理流程。原理差不多也就这样了，如果是第一次接触可以查看之前发的第二版分布式系统的相关介绍。很多事情都在发生变化，当前这个版本也有很多需要优化的地方，比如执行节点的调度、容错处理等内容，本篇都没有涉及到。

当分布式遇到容器编排，其实现的方式又存在很大的差别，很多已经成熟的东西我们直接拿来用就好，通过k8s来实现上述的分布式会有更大的优势，特别是运行实例动态调度变化上。技术总是日新月异的变化，我们要做的就是紧跟发展的趋势，别被甩下车。

最后欢迎关注我们的微信公众号"铸盾师"

