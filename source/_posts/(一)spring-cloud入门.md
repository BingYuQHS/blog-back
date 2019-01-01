---
title: (一)spring cloud入门
date: 2018-10-15 15:41:26
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>

# spring cloud介绍 #

- Spring提供了一系列工具，可以帮助开发人员迅速搭建分布式系统中的公共组件（比如：配置管理，服务发现，断路器，智能路由，微代理，控制总线，一次性令牌，全局锁，主节点选举，分布式session，集群状态）。协助分布式环境中各个系统，为各类服务提供模板性配置。使用spring cloud,开发人员可以搭建实现了这些样板的应用，并且在任何分布式环境下都能工作得很好，小到笔记本电脑，大到数据中心和云平台。
- spring cloud官网定义比较抽象，我们可以从简单的东西开始。Spring Cloud是基于spring Boot的，最适合用于管理Spring Boot创建的各个微服务应用。要管理分布式环境下的各个Spring Boot微服务，必然存在服务注册问题。所以我们先从服务的注册谈起。既然是注册，必然有个管理注册中心的服务，各个在Spring Cloud管理下的Spring Boot应用就是需要注册的client。
- Spring Cloud使用eureka server,然后所以需要访问配置文件的应用都作为一个eureka client注册上去。eureka是一个高可用的组件，它没有后端缓存，每一个实例注册后需要向注册中心发送心跳，在默认情况下eureka server也是一个eureka client,必须要指定一个server。
# spring boot与spring cloud的区别 #
- springboot是spring的一套快速配置的脚手架，可以基于springboot快速开发单个微服务，springcloud是一个基于springboot实现的云应用开发工具。
- spring boot专注于快速，方便集成单个个体，spring cloud是关注全局的服务治理框架
- spring boot使用了默认大于配置的理念，很多集成方案已经帮你选着好了，能不配置就不配置，spring cloud很大一部分是基于springboot来实现的。
- spring boot可以离开spring cloud独立使用开发项目，但spring cloud离不开spring boot,属于依赖关系。
# spring boot与spring cloud的版本匹配 #
- 参考：[spring boot与spring cloud的版本匹配](https://blog.csdn.net/zxl646801924/article/details/80742719)
- ![版本匹配](https://i.imgur.com/19dmDbt.png)
# 实践 #
参考：[spring cloud入门](https://blog.csdn.net/forezp/article/details/81040925)
## 创建springboot项目 ##
- ![](https://i.imgur.com/FE7SQ5P.png)
- ![](https://i.imgur.com/cp4g9Tt.png)
- ![](https://i.imgur.com/fAd5OWZ.png)
- ![](https://i.imgur.com/YqllfbF.png)
- ![](https://i.imgur.com/ze1ttoj.png)
## 添加依赖 ##
- ![](https://i.imgur.com/7fRyhIf.png)
- ![](https://i.imgur.com/Tb8BY9O.png)
- ![](https://i.imgur.com/6Rr2aSz.png)
## 创建服务注册中心 ##
- 创建一个model工程作为服务注册中心，即eureka server,另一个作为eureka client
- 右键工程-->创建model-->选择spring initialir
- ![](https://i.imgur.com/ludOPNn.png)
- ![](https://i.imgur.com/CgGqkxs.png)
- ![](https://i.imgur.com/t7UIfPh.png)
- ![](https://i.imgur.com/iVfuY3f.png)
- ![](https://i.imgur.com/wcQEopY.png)
- 创建完后的工程，其pom.xml继承了父pom文件，并引入spring-cloud-starter-netflix-eureka-server的依赖
- ![](https://i.imgur.com/ohiYQUR.png)
## 启动注册服务中心 ##
- 启动一个服务注册中心，只需要一个注解@EnableEurekaServer，这个注解需要在springboot工程的启动application类上加
- ![](https://i.imgur.com/DHZHgAU.png)
## eureka server的配置文件 ##
- eureka是一个高可用的组件，它没有后端缓存，每一个实例注册之后需要向注册中心发送心跳（因此可以在内存中完成），在默认情况下erureka server也是一个eureka client ,必须要指定一个 server。eureka server的配置文件appication.yml
- ![](https://i.imgur.com/6uXm9qd.png)
- 运行项目后：
- ![](https://i.imgur.com/CHYbFR7.png)
## 创建一个服务提供者 ##
- eureka client,当client向server注册时，它会提供一些元数据，例如主机和端口，URL,主页等。eureka server从每个client实例接收心跳消息。如果心跳超时，则通常将该实例从注册server中删除。
- 创建过程同server类似，创建完pom.xml如下：
- ![](https://i.imgur.com/TdgjX2p.png)
- ![](https://i.imgur.com/7dYWz7h.png)
- ![](https://i.imgur.com/j38QtPC.png)
- ![](https://i.imgur.com/2mORUzM.png)
- 注解@EnableEurekaClient，表明自己是一个eurekaclient，加在项目启动类：
- ![](https://i.imgur.com/gamdgHg.png)
##  eureka client的配置文件 ##
- ![](https://i.imgur.com/CvxV4zv.png)
- 启动工程(server和client)，打开http://localhost:9876 ，即eureka server 的网址
- ![](https://i.imgur.com/lfmT8ut.png)
## 添加一个简单的业务 ##
- ![](https://i.imgur.com/0haqJao.png)
- ![](https://i.imgur.com/vp1NCUo.png)
## Eureka的自我保护模式 ##
- ![](https://i.imgur.com/fbzslmz.png)
- ![](https://i.imgur.com/hwMcmf6.png)