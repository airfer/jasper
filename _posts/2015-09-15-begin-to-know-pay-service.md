---
layout: post
cover: 'assets/images/new-pic-6.jpg'
title:  支付业务入门及实践
date:   2015-09-15 14:18:00
tags: pay
subclass: 'post tag-test tag-content'
categories: 'casper'
navigation: True
logo: 'assets/images/ghost.png'
---

<h4>前言</h4>

<p>对于一些比较复杂的业务，其业务逻辑是十分复杂的。如果只是通过口耳相授，很难将复杂的逻辑讲清楚，也很难在短时间内让新人上手。对于一个团队来说，业务逻辑知识以及测试自动化技术的沉淀显得尤为重要。现在就业务入门及相关指导进行梳理。</p>

---

<h4>一、整个业务指导流程</h4>

+ 业务介绍
+ 系统架构
+ 业务线
+ 流程相关
+ 人员相关
+ 工作中需要使用的工具
+ 权限申请
+ 测试账号
+ 其他
+ 数据库相关系统
+ 测试上线流程规范

<h4>二、详细介绍（360支付平台为例）</h4>

<h5>1、业务介绍</h5>

<p>360支付平台作为360公司支付基础服务部门，为公司提供支付相关服务支持。现在公司支持公司90%的收入性业务，其业务来源主要有三大部分，收入部分可从公司的财报中解读出来。</p>

+ 互联网增值服务部分，包括游戏、彩票、小说等
+ 广告收入部分，主要为点睛广告平台
+ 自营业务部分，主要有360理财宝、票据以及手机钱包等

<h5>2、系统架构</h5>

系统架构的部分   
![](\images\payment_plantform.jpg "平台架构图")   
平台的架构，主要是按照支付流程进行说明的，支付的逻辑涵盖下单、支付以及支付逻辑的处理。在本文中，并不对所有细节进行阐述

<h5>3、业务线</h5>   

+ 内部业务线：主要是给公司内部业务使用，比如彩票、钱包等
+ 外部业务线：主要是公司以外的公司使用，先阶段还未扩展到外部业务线

<h5>4、流程相关</h5>   

+ 对于需求变更，测试人员参加需求变更评审
+ 首先开发人员，根据测试需求，在提测系统中填写提测单，在提测单中写明测试的要点以及测试类别（功能测试、安全测试或者全部）
+ 根据开发人员的提测要点，编写测试用例，并根据测试需要找开发详细沟通，明确测试要点以及测试范围
+ 根据测试用例进行测试，及时与开发沟通，并将测试进度在每天的测试日报中及时告知开发，以及测试中所遇到的问题（例如环境、沟通等等）
+ 测试结束后，发送测试结束报告，并确认上线版本
+ 如果为APP测试，在版本上线之前需经过APP安全测试，确保版本无安全隐患漏洞，并提交上线申请
+ 等产品上线后，需要进行线上回归，确保无bug

<h5>5、人员相关</h5>   
   
+ 根绝测试的难度以及测试时间，合理的分配人力进行测试。对于较为重要的提测任务，最少需要两个人，进行交叉测试，进一步保证产品质量
+ 对于固定模块，可安排固定的人员进行测试，也可以确定测试常员，以支持不同业务的紧急需求，具体的人员分配测试根据测试需要灵活制定    

<h5>6、工作中使用的工具</h5>   

现在支付平台主要有三款自动化测试工具，其中两款为自动化接口类工具，一款为单元测试工具
   
+ 支付接口测试工具
![](\images\qpay_interface.jpg "平台架构图")  
   
+ 支付底层场景回归工具  
![](\images\qpay_situation.jpg "平台架构图")   

插件化设计：  

![](\images\situation_module1.jpg "平台架构图")  
 
表字段校验规则   

![](\images\situation_module2.jpg "平台架构图")  

+ 基于Gtest的白盒单元测试工具  

<h5>7、权限申请</h5>   

+ 在ops进行相关机器、相关权限的申请   

<h5>8、测试账号</h5>   
   
+ 可从数据库中自己进行提取，也可以自行构造

<h5>9、数据库相关系统</h5>   

+ 在第一章 支付网关逻辑之数据库中已经进行了详细的阐述，具体请到相关章节查看   

<h5>10、测试上线流程规范</h5>   

+ 请参照第4部分流程相关



