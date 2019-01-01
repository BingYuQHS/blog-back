---
title: (四)spring cloud进阶三——服务消费者（feign）(Finchley版本)
date: 2018-10-16 15:47:59
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# 服务消费者（Feign）(Finchley版本) #
- Feign是一个声明式的伪Http客户端，它使得写http客户端变得更简单。使用feign,只需要创建一个接口并注解。它具有可插拔的注解特性，可使用Feign注解和JAX-RS注解。Feign支持可插拔的编码器和解码器。Feign默认集成了Ribbon，并和Eureka结合，默认实现了负载均衡的效果。
- 和上个教程一样，启动两个service-hi工程的实例【9873，9872端口】
# 创建一个Feign的服务 #
## 创建一个spring boot项目（module） ##
- 参照上个教程的创建方法
## 添加依赖 ##
- 注意依赖名称，新版本：openfeign;老版本：feign![](https://i.imgur.com/TbyXNZe.png)
## 添加配置文件 ##
- 配置文件application.yml![](https://i.imgur.com/x6Amdxw.png)
## 在启动类开启feign的功能 ##
- 在工程的启动类ServiceFeignApplication，加上@EnableFeignClients注解开启Feign的功能。![](https://i.imgur.com/44FKnjL.png)
## 定义feign接口调用其他服务 ##
- 定义一个Feign接口，通过@FeignClient（“服务名”），来指定调用哪个服务，比如在代码中调用了service-hi服务的“hi”接口：(注意，只用定义接口，消费的是其他服务)。![](https://i.imgur.com/BcnyNnf.png)
## 在controller层对外暴露api接口 ##
- 在web层的controller层，对外暴露一个“/hi”的API接口，通过上面定义的Feign客户端SchedualServiceHi来消费服务。![](https://i.imgur.com/gVTSL9a.png)
- 启动程序，运行结果：![](https://i.imgur.com/KTxBBmM.png)
- 结果是浏览器交替显示，即做了负载均衡，交替消费两个运行的实例，与上个教程的效果一致。
- ![](https://i.imgur.com/AwlTkSg.png)刷新后：![](https://i.imgur.com/QlzLi6X.png)