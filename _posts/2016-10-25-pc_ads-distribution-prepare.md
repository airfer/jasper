---
layout: post
cover: 'assets/images/cover7.jpg'
title: 基于 Docker 的分布式测试系统构建 (二)
date:   2016-10-25 19:18:00
tags: Docker
subclass: 'post tag-test tag-content'
categories: 'casper'
navigation: True
logo: 'assets/images/ghost.png'
---

**前言**

关于如何运用docker来解决现实问题并非是一个新鲜的问题，关于docker在测试方面的应用也有很多的文章，本篇是在前人的基础上研究了如何结合业务逻辑构建基于docker的分布式测试系统。当然初次研究肯定有很多不完善的地方，希望抛砖引玉，如果在阅读过程中有什么问题，欢迎大家留言交流。废话不多说了，本系列共2篇6个部分，涵盖了从项目开始到结束的所有基本过程。本篇介绍前2个部分，分别为构建测试系统的需求以及搭建docker镜像，希望本篇文章对你有所帮助。

   

> ----  搜狗ADTQ测试组出品

                                                                                                                                                                        
----------

<h5> 一、为何要做这件事 </h5>
现在部门内每次有新的需求上线，都会对之前所有的cases进行回归。cases的数量比较大，每次回归case需要花费很长的时间，部门内现在已有一个分布式的回归工具，是基于Jenkins的api和Shell来做的，虽然实现了基本的分布式的功能，但是也有一些弊端在后续的实践中慢慢显现，主要有以下几个方面：

 1. *case回归的时间很长*。case回归的时间长主要有两个方面的原因造成：第一个原因是分布式节点比较少，这使得分布式的优势并没有完全凸显出来；第二个原因是轮询检查子节点的完成情况，并设置超时的时间，超时时间过短则不能完成收集子节点结果的任务，而超时时间过长则进一步延长了case回归的时间

 2. *分布式子节点新增困难*。当case量上升，需要新增子节点的时候，发现这并非是一件很容易的事情。由于子节点是jenkins的node节点，所以每次新增节点都相当于部署一个jenkins节点，并需要将任务代码rsync到新的节点中。这对于一个新手来说是很不方便的，不利于后期维护。

 3. *结果展示不够直观*。现有的结果展示只有邮件的形式，当运行完成之后通过邮件来通知分布式程序运行的结果，如果你想查看某一个文件中case的执行情况，比如该文件中有多少case成功，或者有多少case失败，抱歉，这并不提供这个功能。你收到的邮件只是一堆失败case的集合。

当然还有一些其他的弊端，但是主要的问题就是以上的三个方面。后续在解决这三个问题的基础上，也完善了其他的功能，比如case的执行粒度、case的执行情况统计等等。所以在充分调研了现有的分布式工具的基础上，我们提出了新的需求，新需求主要有以下几个方面：

 - 回归case的时间要尽可能的短，最起码要比现有的执行时间短
 - case的执行粒度要尽可能的细，现有框架的case是以文件作为最小的执行粒度，而我们要做的是以caseid作为分布调度的基本粒度
 - 回归结果的展示要更直观，现有的框架仅能够完成失败case的收集并邮件通知，我们要做的是分布式结果收集、统计、展示、查询这四个维度

有了需求我们才开始着手新分布式测试工具的开发工作，后续的章节将会从各个方面进行阐述。

<h5>二、构建Docker Images 以及踩过的坑</h5>
-----------------

对于一个刚开始接触docker的新手，一切都是新的，所以就从最开始说起，说说搭建docker镜像的那些事以及踩过的那些坑吧。。。

***1、搭建基础docker镜像***

*Build docker镜像*之前，首先要了解的是你的服务程序是运行在哪个linux版本之下的，是Ubuntu还是Redhat或者其他，尽量选择与现有的平台一致的镜像，如果能具体到linux的版本号就最好了。当然如果你说我的程序是通用类型，具体的版本并不重要，那么选择一个你熟悉的镜像就ok了。比如我的服务端程序是运行在redhat6.5下，那么我会选择Centos6.5的版本作为我的基础镜像。

选择与现有平台一致的镜像有利于后续问题的排查，一般你对服务端程序代码的熟悉程度肯定比不了开发，甚至连一知半解都做不到，如果选择不一致的运行平台，如果出现了问题，你可能无法确定是什么原因导致的，是平台？ or 自有的服务端程序? ,而选择一致性平台则排除了这一问题。

服务端程序以c++为例，都会依赖很多的库，而下载下来的镜像是纯净的系统，所以第一步就是按照程序运行的依赖包。如果开发能给出一个依赖包的列表最好，如果不能，那么尝试着运行程序，将缺少的依赖包记录下来，以便制作dockerfile文件。当你尝试着将所有的依赖包安装完整，并且服务端程序可以正常的启动起来，那么dockerfile文件也就差不多制作完成了。

关于dockerfile文件的基础语法我就不再这里详述了，不懂的 搜狗（这是广告，哈哈...）一下吧。下面是我自己制作的dockerfile文件，或许对你有些帮助

{% raw %}
# Dockfile to install bidding module dependency
# Based on Centos:6.5
 
FROM centos:6.5
 
# to replace the origin CentOS-Base.repo
WORKDIR /etc/yum.repos.d
RUN mv CentOS-Base.repo CentOS-Base.repo.backup
 
# Add new CentOS-Base.repo
ADD CentOS-Base.repo /etc/yum.repos.d/
 
# Install dependency package
 
RUN yum install gcc g++ gcc-c++
RUN yum install curl libcurl-devel
RUN yum install boost boost-devel boost-doc
RUN yum instlal zlib zlib-devel
RUN yum install openssl openssl-devel
 
# Set CXX and CC
 
RUN echo "export CC='gcc -std=gnu99'"    >> /etc/profile
RUN echo "export CXX='g++ -std=gnu++0x'" >> /etc/profile
 
CMD ["/bin/bash"]
{% endraw %}

***2、从安装docker到构建完镜像踩过的那些坑***
从开始接触到后续构建完成，一路走来，跌跌撞撞，幸好问题都已解决，现就碰到的问题与大家分享一下，或许你也正被其中的某一问题所困扰

 *（1）关于docker安装过程中，离线安装的问题（仅限centos系列）:*
docker是可以离线安装的，有离线安装包。如果是离线安装则需要按照cgroup依赖包，[http://mirrors.163.com/centos/6/os/x86_64/Packages/][1]。在这篇文章中写的比较清楚：[http://www.iyunv.com/thread-149007-1-1.html][2]，可根据Centos的版本以及安装包依赖，切记不可照搬照抄。如果是可以上外网，最好还是线上安装吧，比如在Ubuntu下，docker的安装一条命令就可以了：
```shell
curl -sSL https://get.daocloud.io/docker | sh
```


 *（2）关于centos6下升级docker引擎到1.2.1版本*
docker引擎的1.2.1版本集成了新的swarm，所以如果有可能还是使用1.2.1以后版本的docker。但是对于centos6来说，如果要升级到1.2.1版本，则安装的包需要依赖systemd，但Centos6无法安装systemd。

关于这个问题的回答可详细查看：[http://stackoverflow.com/questions/28347694/how-to-install-systemd-on-centos-6-6][3]，其中在这篇文章说的在Centos6中安装docker-engine1.2.1，会出现依赖包缺失的错误。[http://blog.csdn.net/weiguang1017/article/details/52293235][4]，这篇文章也介绍了如何安装，但是我尝试后还是无法在centos6下无法升级到1.2.1以上版本，如果有成功的同学，麻烦告知，谢谢


*（3）关于修改devicemapper引擎的相关参数（最大的坑）*
当你的docker镜像过大时，比如大于10G，那么默认的devicemapper引擎是会报错的，因为docker service初始化的时候，默认分配的存储空间最大也就是10G，所以必须要修改这个参数，并重启docker service。关于修改这个参数，足足搞了一天，差点都放弃了，因为按照网上的资料所有的更改都无法生效，具体方法如下：

第一种方法为service docker start 之前修改，其修改的内容就是增加--storage-opt dm.basesize=30G选项；第二种方法为调用脚本动态增加，具体的操作方法可以参照如下：[http://blog.chinaunix.net/uid-20788636-id-5029770.html][5]。

先说第二种方法，在运行脚本的过程中，出现以下的错误：*resize2fs: Device or resource busy while trying to open /dev/mapper/docker-253:1-1270544-d2d2cef71c86910467c1afdeb79c1a008552f3f9ef9507bb1e04d77f2ad5eac4。*因为此脚本主要是调用resize2f函数来进行动态的扩存，但是由于centos6文件格式的为xfs，对比并不支持，所以此方法行不通。对于centos6来说，还可以统统xfs_growfs来进行扩存，但是对比并不熟悉，研究了一些时间，感觉也没有什么头绪，暂时放弃了。关于resize2f调整rootfs可以参照[http://www.111cn.net/sys/linux/87656.htm][6]

但是对于第一种方法，也是会报错，报错的信息如下：*Error starting daemon: error initializing graphdriver: Unknown option dm.basesize*。所以两种方法现在都无法达到本来预期的目的，当做一些事情的时候，忽然感觉所有的事情都在阻碍你向前进，这个时候应该静下心，努力排查可能出现的错误，找出其中的蛛丝马迹，应该相信你并非是第一个碰到这个题目的人，合理的利用google。经过很久的排查，终于在github的一个issue里面找到了可能存在的问题，该网页如下：[https://github.com/docker/docker/issues/21171][7]。其中的一位同学做了一下的事情：
![](/images/docker/answer.png)

再指定storage-opt的时候，同时也指定了storage-driver。抱着试一试的心态，添加了上述参数，果然可以成功的改变container中rootfs中的大小，算做经验教训吧。其他有用的选项可参考如下示例：DOCKER_STORAGE_OPTIONS="--storage-opt dm.loopdatasize=2000G --storage-opt dm.loopmetadatasize=10G --storage-opt dm.fs=ext4 --storage-opt  dm.basesize=20G"

- dm.loopdatasize=2000G是指存放数据的数据库空间为2t，默认是100g

- dm.loopmetadatasize=10G是存放Metadata数据空间为10g，默认是2g

- dm.fs=ext4是指容器磁盘分区为ext4

- dm.basesize=20G是指容器根分区默认为20g，默认是10g

*（4）关于docker镜像拉取以及squid代理*
现在很多公司的机子都是内网环境，无法连接外网，无法连接外网也就无法从docker的registry中拉取镜像，除非你们公司搭建了私有的registry并且包含你所需要的镜像。如果没有，那么有两种途经来解决:

- 第一种方法，就是先在可访问外网的机子上（一般个人本机可以访问外网）下载所需要的docker镜像，然后通过docker save命令将其保存为本地镜像，然后导出后再通过docker load 命令将其导入到所需要的docker宿主机中。这种方法比较麻烦，但是好在一般都能实现，对于确实无法连接外网的宿主机也不失为一种办法。


- 第二种办法，就是在内网中找一台可以访问外网的机子，然后在该机子上搭建squid代理服务器，docker的宿主机可以通过配置代理来拉取镜像。关于squlid代理服务器运行可以参照一下命令:

{% raw %}
docker run --name squid -d --restart=always \
  --publish 3128:3128 \
  --volume /search/wangyukun/log/cache:/var/spool/squid3 \
  --volume /search/wangyukun/log/squid_log/:/var/log/squid3 \
  sameersbn/squid:3.3.8-19
{% endraw %}

当然前提是你已经通过第一种方法安装了squid镜像，这属于一次辛苦多次收益，哈哈，如果你们内网的所有自己都无法连接外网，那么只能通过第一种方法了。启动squid代理服务后，那么就要docker宿主机上配置代理服务，以centos6举例，修改/etc/sysconfig/docker 配置文件的内容如下：

{% raw %}
other_args="--graph=/search/odin/wangyukun/docker --insecure-registry 10.142.97.235:5000 --storage-driver devicemapper --storage-opt dm.basesize=100G --storage-opt dm.loopdatasize=2000G --storage-opt dm.loopmetadatasize=10G"
HTTP_PROXY=http://your_squid_service_ip:3128
http_proxy=$HTTP_PROXY
HTTPS_PROXY=$HTTP_PROXY
https_proxy=$HTTP_PROXY
export HTTP_PROXY HTTPS_PROXY http_proxy https_proxy
{% endraw %}

这样重启docker service 就可以了。镜像搭建以及走过的坑就先说到这吧，剩下的部分第二篇再续。。。


  [1]: http://mirrors.163.com/centos/6/os/x86_64/Packages/
  [2]: http://www.iyunv.com/thread-149007-1-1.html
  [3]: http://stackoverflow.com/questions/28347694/how-to-install-systemd-on-centos-6-6
  [4]: http://blog.csdn.net/weiguang1017/article/details/52293235
  [5]: http://blog.chinaunix.net/uid-20788636-id-5029770.html
  [6]: http://www.111cn.net/sys/linux/87656.htm
  [7]: https://github.com/docker/docker/issues/21171
