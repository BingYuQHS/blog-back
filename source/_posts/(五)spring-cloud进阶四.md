---
title: (五)spring cloud进阶四——断路器(Hystrix)(Finchley版本)
date: 2018-10-25 14:44:28
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# 断路器(Hystrix)(Finchley版本) #
- 在微服务架构中，根据业务来拆分成一个个的服务，服务与服务间可以相互调用(RPC)，在spring cloud可以用RestTemplate+Ribbon和Feign来调用服务。为了保证其高可用，单个服务通常会集群部署。但由于网络或自身原因，服务并不能保证100%可用，如果单个服务出现了问题，调用服务就会出现线程阻塞，此时若有大量的请求涌入，Servlet容器的线程资源就会被消耗完毕，导致服务瘫痪，因为服务与服务之间的依赖性，故障会传播，会对整个微服务系统造成灾难性的严重后果，这就是服务故障的“雪崩”效应。
- 为了解决这个问题，业界提出了断路器模型：![](https://i.imgur.com/Ij52NVO.png)![](https://i.imgur.com/QaS6wog.png)
- 启动eureka-server工程；启动service-hi工程，它的端口为9873。
# 在ribbon中使用断路器 #
改造service-ribbon工程的代码
## 添加hystrix的依赖 ##
- 添加依赖：spring-cloud-starter-netflix-hystrix![](https://i.imgur.com/jFUVvZj.png)
## 在启动类开启断路器 ##
- 在启动类添加@EnableHystrix注解，开启断路器的功能，用于控制servlet容器的线程阻塞。![](https://i.imgur.com/9CE0Zr1.png)
## 改造HelloService类 ##
- 改造HelloService类，在hiService方法上加上@HystrixCommand注解，该注解为该方法创建了熔断器功能，并指定了fallbackMethod熔断方法，此处的熔断方法直接返回一个字符串。![](https://i.imgur.com/uyeUM43.png)
## 启动service-ribbon工程 ##
- 访问：localhost:9871/hi?name=qinhs
- 浏览器显示（注意，若service-hi服务正常启动，该服务的/hi接口能正常访问）：![](https://i.imgur.com/yPDUy1l.png)【ps:该图片是service-hi服务正常启动时】
- 关闭service-hi：当service-hi服务不可用时（如未正常启动），service-ribbon调用service-hi的API接口时，会执行快速失败，直接调用熔断方法（返回字符串），而不是等待响应超时，这就很好地控制了容器地线程阻塞。![](https://i.imgur.com/jcMva5H.png)
# 在feign使用断路器 #
- feign是自带断路器的，在D版本的Spring cloud之后，它没有默认打开。需要在配饰文件中配置打开它，在配置文件加以下代码：
>dd
		feign hystrix enabled=true


## 改造service-feign工程 ##
- Feign自带断路器，只需要在FeignClient的schedualServiceHi（定义的Feign接口）接口的注解（@FeignClient注解）中加上fallback的指定类就行了：![](https://i.imgur.com/7O2ryir.png)
- 注意：SchedualServiceHiHystriclmpl类要实现SchedualserviceHi接口，并使用@Component注解：![](https://i.imgur.com/iVcpoDE.png)
## 启动service-feign工程 ##
- 此时没有启动service-hi工程，启动service-feign工程，出现超时问题：![](https://i.imgur.com/wPLnk7B.png)
- 解决办法：真正原因是忘了加@Component注解，断路器指定的返回方法找不到：![](https://i.imgur.com/iFp2SWu.png)![](https://i.imgur.com/KmfmWRm.png)
- 启动service-hi工程后（证明熔断器起到了作用）![](https://i.imgur.com/LNKpPas.png)