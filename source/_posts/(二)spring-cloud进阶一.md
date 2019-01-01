---
title: (二)spring-cloud进阶一
date: 2018-10-16 09:34:42
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# eureka server添加权限 #
- 参考：[注册中心加权限](http://www.cnblogs.com/xiaojunbo/p/7094066.html "注册中心加权限")
## 添加security的相关依赖 ##
- ![](https://i.imgur.com/4qZuOWs.png)
## 配置配置文件 ##
- ![](https://i.imgur.com/bkkLccq.png)
- 在spring boot2.X中，开启security已过时，即`security.basic.enabled: true`配置过时。
- 可以: ![](https://i.imgur.com/zDTWd72.png)![](https://i.imgur.com/XeH2Uxk.png)
- ps:[此时启动注册中心，需要输入用户名和密码，但之前的服务提供者eureka-client不能注册成功，需要注册中心为其添加权限]
# eureka client添加注册权限 #
- 修改client工程的配置文件，设置：
>
		eureka.client.serviceUrl.defaultZone=http://${username}:${password}@${eureka.instance.hostname}:${server.port}/eureka/
- 配置如下：![](https://i.imgur.com/JL4Nt6A.png)
# eureka client注册不上的问题解决 #
- 参考：[https://my.oschina.net/bianxin/blog/1819947](https://my.oschina.net/bianxin/blog/1819947 "Springboot-2.0.2.RELEASE Eureka认证后，服务注册失败问题")
- 问题是Spring Security引起的，查看源码发现CSRF保护默认是开启的，可以禁用掉即可。
- spring boot老版本配置：
>
		security:   
		  basic:
		    enabled: true
		  user:
		    name: admin
		    password: 123456

- 新版本解决方案：添加一个配置类禁用csrf如下：(会发现注入服务不需要密码了，说明失去了验证。)
![添加配置类禁用csrf](https://i.imgur.com/nLNagtM.png)