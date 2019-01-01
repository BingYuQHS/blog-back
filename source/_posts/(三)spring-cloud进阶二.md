---
title: (三)spring-cloud进阶二
date: 2018-10-16 10:43:53
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# 服务消费者（rest+ribbon）(Finchley版本) #
参考：[https://blog.csdn.net/forezp/article/details/81040946](https://blog.csdn.net/forezp/article/details/81040946 "rest+ribbon(Finchley版本)")
- 在微服务架构中，业务都会被拆分成一个独立的服务，服务与服务的通讯是基于http restful的。spring cloud有两种服务调用方式，一种是ribbon+restTemplate,另一种是feign。
- Ribbon用于负载均衡
- RestTemplate用于消费其他服务提供的接口
# 在IEDA下启动多个实例 #
参考：[https://blog.csdn.net/forezp/article/details/76408139](https://blog.csdn.net/forezp/article/details/76408139 "idea下启动多个实例")
- 启动eureka-server工程
- 启动service-hi工程【启动了一个实例】
- 修改service-hi工程的端口（在配置文件中修改），并启动【启动了另一个实例】
- 会发现service-hi在eureka-server注册了2个实例，这就相当于一个小的集群。
- 图解：![](https://i.imgur.com/XJx2OR2.png)![](https://i.imgur.com/GobY1oP.png)![](https://i.imgur.com/v79UfEW.png)![](https://i.imgur.com/iX5oAMH.png)![](https://i.imgur.com/pTETbLr.png)
## 建一个服务消费者 ##
### 创建一个spring boot项目（module） ###
- ![](https://i.imgur.com/EhST8GI.png)![](https://i.imgur.com/j4dOGtv.png)![](https://i.imgur.com/OG29a66.png)![](https://i.imgur.com/jvRD3Hk.png)![](https://i.imgur.com/ukxfFVg.png)
### 添加依赖 ###
- ![](https://i.imgur.com/oytjbRQ.png)![](https://i.imgur.com/He0oPuC.png)
### 添加配置文件 ###
- ![](https://i.imgur.com/TEPnDff.png)
### 在启动类向服务中心注册 ###
- 在工程的启动类中，通过@EnableDiscoveryClient向服务注册中心注册，并且向程序的ioc注入一个bean:restTemplate;并且通过@LoadBalanced注解表明这个restTemplate开启负载均衡的功能。
- 此处注意：spring boot注入restTemplate的方式。
	1. 直接在启动类用new RestTemplate()方式实例化restTemplate(如下图一所示)。
	2. spring boot自动注入了RestTemplateBuilder的实例，利用builder.buid()方法实例化一个RestTemplate对象（如下图二和三所示）。
	3. ![](https://i.imgur.com/L8oV8Cg.png)![](https://i.imgur.com/E7XnZc9.png)![](https://i.imgur.com/getFM2R.png)
### 通过restTemplate来消费其他服务的接口 ###
- 编写测试类HelloService，通过之前注入的ioc的restTemplate来消费service-hi服务的"/hi"接口，此处直接用程序名替代了具体的url地址，在ribbon中它会根据服务名（前提是restTemplate加了负载均衡的注解@Loadbalanced）选着具体的服务实例，根据服务实例在请求的时候会用具体的url替换掉服务名。
- ![](https://i.imgur.com/YywSlFk.png)![](https://i.imgur.com/aaDmO6i.png)
- 运行结果：![](https://i.imgur.com/LFk4sL2.png)![](https://i.imgur.com/T5BLAq9.png)
- 刷新后：浏览器交替显示，说明调用restTemplate.getForObject(“http://SERVICE-HI/hi?name=”+name,String.class)方法时，已经做了负载均衡，访问了不同的端口的服务实例 ![](https://i.imgur.com/MYrdJoP.png)
- 此时的架构：![](https://i.imgur.com/eIMdbMC.png)