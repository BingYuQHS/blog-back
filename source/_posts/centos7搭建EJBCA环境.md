---
title: centos7搭建EJBCA环境
date: 2018-11-05 10:30:27
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# 一、介绍 #
## EJBCA说明 ##
EJBCA是一个全功能的CA系统软件，它基于J2EE技术，并提供了一个强大的、高性能并基于组件的CA。EJBCA兼具灵活性和平台独立性，能够独立使用，也能和任何J2EE应用程序集成。
## EJBCA特性 ##
- LGPL开源许可
- 建立在J2EE 1.3(EJB2.0)规范上
- 灵活的、基于组件的体系结构
- 多久CA
- 多个CA和多级CA,在一个EJBCA实例中创建一个或多个完整的基础设施单独运行，或者在任何J2EE应用中集成它
- 简单的安装和配置
- 强大的基于Web的管理界面，并采用了高强度的鉴别算法
- 支持基于命令的管理，并支持脚本等功能
- 支持个人证书申请或证书的批量生产
- 服务器和客户端证书能够采用PKCS12，JKS或者PEM格式导出
- 支持采用Netscape,Mozilla,IE等浏览器直接进行证书申请
- 支持使用开放的API和工具通过其他应用程序申请证书
- 由RA添加的新用户可以通过Email进行提醒
- 对于新用户验证可以采用随机或手工的方式生成密码
- 支持硬件模块，来集成硬件签发系统（如智能卡）
- 支持SCEP
- 支持用特定用户权限和用户组的方式来进行多极化管理
- 对不同类型和内容的证书可以进行证书配置
- 对不同类型的用户可以进行实体配置
- 遵循X509和PKIX（RFC3280）标准
- 支持CRL
- 完全支持OCSP,包括AIA扩展
- CRL生成和基于URL的CRL分发遵循RFC3280,可以在任何SQL数据库中存储证书和CRL（通过应用服务器来处理）
- 可选的多个分发器，以用来在LDAP中发布证书和CRL
- 支持用来为指定用户和证书来恢复私钥的密钥恢复模块
- 基于组件的体系结构，用来在发布证书时采用多种实体授权方式
- 容易集成到大型应用程序中，并集成到业务流程进行了优化
- EJBCA完全采用java编写，能够在任何采用J2EE服务器的平台上运行。开发和测试是在linux和Windows上进行的
# 二、安装 #
## 安装前的准备 ##
- jdk-8u191-linux-x64.rpm[JDK1.8]
- apache-ant-1.9.7-bin.tar.gz
- jboss-eap-6.4.0-installer.jar
- ejbca_ce_6_3_1_1.zip
- mysql
- mysql-connector-java-5.1.27.jar
- centos7
## 系统准备 ##
### 安装JDK1.8 ###
- 检查已安装的jdk,如果有，先删除
>
		rpm -qa|grep java
		rpm -e --nodeps filename

- 从[oracle官网](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html, "jdk-8u191-linux-x64.rpm")下载jdk安装包：`jdk-8u191-linux-x64.rpm`,放在/usr/file/目录下
- 用下面的命令安装，默认安装在/usr/java目录
> 
		rpm -ivh jdk-8u191-linux-x64.rpm

- 配置环境变量
> 
		vi /etc/profile

- 追加如下内容：
> 
		#java conf
		export JAVA_HOME=/usr/java/jdk1.8.0_191
		export PATH=$PATH:$JAVA_HOME/bin
		export CLASSPATH=:$JAVA_HOME/lib

- 使配置立即生效，然后检查安装结果
> 
		source /etc/profile
		java -version

### 安装ant ###
- 从[ant官网](https://archive.apache.org/dist/ant/binaries/, "jdk-8u191-linux-x64.rpm")下载jdk安装包：`apache-ant-1.10.0-bin.tar.gz`,解压
> 
		tar xvf apache-ant-1.10.0-bin.tar.gz -C /usr/java/

- 配置环境变量
> 
		vi /etc/profile

- 追加如下内容：
> 
		#ant conf
		export ANT_HOME=/usr/java/apache-ant-1.10.0
		export PATH=$PATH:$ANT_HOME/bin

- 使配置立即生效，然后检查安装结果
> 
		source /etc/profile
		ant -version

### 修改centos主机名 ###
- 参考:`https://www.jianshu.com/p/39d7000dfa47`
- 要查看主机名相关的设置：
> 
		[root@localhost ~]# hostnamectl
		[root@localhost ~]# hostnamectl status

- 只查看静态、瞬态或灵活主机名，分别使用--static，--transient或--pretty选项
> 
		[root@localhost ~]# hostnamectl --static
		localhost.localdomain
		[root@localhost ~]# hostnamectl --transient
		localhost.localdomain
		[root@localhost ~]# hostnamectl --pretty

- 要同时修改所有三个主机名：静态、瞬态和灵活主机名：
> 
		[root@localhost ~]# hostnamectl set-hostname by.cos7
		[root@localhost ~]# hostnamectl --pretty
		[root@localhost ~]# hostnamectl --static
		by.cos7
		[root@localhost ~]# hostnamectl --transient
		by.cos7

- 就像上面展示的那样，在修改静态/瞬态主机名时，任何特殊字符或空白字符会被移除，而提供的参数中的任何大写字母会自动转化为小写。
一旦修改了静态主机名，/etc/hostname 将被自动更新。然而，/etc/hosts 不会更新以保存所做的修改，所以你每次在修改主机名后一定要手动更新/etc/hosts，之后再重启CentOS 7。否则系统再启动时会很慢
- 手动更新/etc/hosts
> 
		vim /etc/hosts
		#127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		127.0.0.1  by.cos7
		#::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
		::1        by.cos7

- 重启CentOS 7 之后（reboot -f ）
> 
		[root@by ~]# hostname
		by.cos7
		[root@by ~]# hostnamectl
- 如果你只想修改特定的主机名（静态，瞬态或灵活），你可以使用--static，--transient或--pretty选项。例如，要永久修改主机名，你可以修改静态主机名:
> 
		[root@localhost ~]# hostnamectl --static set-hostname by.cos7

- 重启CentOS 7 之后（reboot -f ）
> 
		[root@by ~]# hostnamectl --static
		by.cos7
		[root@by ~]# hostnamectl --transient
		by.cos7
		[root@by ~]# hostnamectl --pretty
		by.cos7
		[root@by ~]# hostname
		by.cos7

- 其实，你不必重启机器以激活永久主机名修改。上面的命令会立即修改内核主机名。
注销并重新登入后在命令行提示来观察新的静态主机名
### 安装mysql ###
- 安装前查看已安装的mysql或mysql库，并删除它们
> 
		rpm -qa|grep -i mysql
		rpm -e --nodeps filename

- 如果重装mysql，查找安装mysql产生的文件，并删除它们
> 
		find / -name mysql
		rm -fr filename

- CentOS7的yum源中默认好像是没有mysql的。为了解决这个问题，我们要先下载mysql的repo源
> 
		$ wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm

- 安装mysql-community-release-el7-5.noarch.rpm包
> 
		$ sudo rpm -ivh mysql-community-release-el7-5.noarch.rpm

- 安装这个包后，会获得两个mysql的yum repo源：/etc/yum.repos.d/mysql-community.repo，/etc/yum.repos.d/mysql-community-source.repo
- 安装mysql
> 
		$ sudo yum install mysql-server

- 如果yum正在使用：强制关闭yum命令`rm -f /var/run/yum.pid`
- 重置密码,重置密码前，首先要登录
> 
		$ mysql -u root

- 登录时有可能报这样的错：ERROR 2002 (HY000): Can‘t connect to local MySQL server through socket ‘/var/lib/mysql/mysql.sock‘ (2)，原因是/var/lib/mysql的访问权限问题。下面的命令把/var/lib/mysql的拥有者改为当前用户：
> 
		$ sudo chown -R mysql:mysql /var/lib/mysql

- 然后，重启服务：
> 
		$ service mysqld restart

- 接下来登录重置密码：
> 
		$ mysql -u root
		mysql > use mysql;
		mysql > update user set password=password('123456') where user='root';
		mysql > exit;

- 需要更改权限才能实现远程连接MYSQL数据库
> 
		# mysql -uroot -p
		Enter password: ******
		mysql> use mysql; (此DB存放MySQL的各种配置信息)
		Database changed
		mysql> select host,user from user; (查看用户的权限情况)
		mysql> select host, user, password from user;
		+-----------+------+-------------------------------------------+
		| host       | user | password                                   |
		+-----------+------+-------------------------------------------+
		| localhost | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
		| 127.0.0.1 | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
		| localhost |       |                                            |
		+-----------+------+-------------------------------------------+
		4 rows in set (0.01 sec)
		由此可以看出，只能以localhost的主机方式访问。
		解决方法：
		mysql> Grant all privileges on *.* to 'root'@'%' identified by '123456' with grant option;
		(%表示是所有的外部机器，如果指定某一台机，就将%改为相应的机器名；‘root’则是指要使用的用户名，)
		mysql> flush privileges;    (运行此句才生效，或者重启MySQL)
		Query OK, 0 rows affected (0.03 sec)
		再次查看。。
		mysql> select host, user, password from user;
		+-----------+------+-------------------------------------------+
		| host       | user | password                                   |
		+-----------+------+-------------------------------------------+
		| localhost | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
		| 127.0.0.1 | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
		| localhost |       |                                            |
		| %          | root | *4ACFE3202A5FF5CF467898FC58AAB1D615029441 |
		+-----------+------+-------------------------------------------+
		4 rows in set (0.01 sec)

### 创建ejbca数据库 ###
- 建数据库：
>
		CREATE DATABASE ejbca CHARACTER SET utf8 COLLATE utf8_general_ci;

- 本机连接centos7中的数据库
> 
		vmware->编辑->虚拟网络编辑器->vmnet8->NAT设置->将虚拟ip地址加上（如80 192.168.171.143：80 TCP）
		关闭防火墙：
		[root@by ~]# systemctl stop firewalld.service
		本机hosts文件（windows/system32/drivers/etc/hosts）配置：
		192.168.171.144  by.cos7

### 为EJBCA新建用户 ###
- 进入root用户(su命令)：(设置pwd:ejbca)
> 
		adduser ca
		passwd ca

- 将安装包的所有者改成ca
> 
		mkdir /opt/ca/
		chown ca:ca /opt/ca
		chown ca:ca /opt/ca/jboss-eap-6.4.0-installer.jar
		chown ca:ca /opt/ca/ejbca_ce_6_3_1_1.zip
		chown ca:ca /opt/ca/mysql-connector-java-5.1.27.jar

## 安装JBoss EAP ##
- 切换用户: `su ca`
- 用java命令安装JBoss EAP：
> 
		java -jar /opt/ca/jboss-eap-6.4.0-installer.jar

- 安装过程会有交互：
> 
		#选择安装语言，默认为英语
		Select language  :
		0:  eng
		1:  chn
		2:  deu
		3:  fra
		4:  jpn
		5:  por
		6:  spa
		Please choose  [0]  :
		
		#确认协议之后是安装路径，默认即可
		Select the installation path:  [/root/EAP-6.4.0]
		
		#随后是部件选项，默认即可
		Select the packs you want to  install:
		
		1    [x]  [Required]      [Red Hat JBoss Enterprise Application Platform]  (542.89  KB)
		2    [x] [AppClient]  (34.24  KB)
		3    [x]  [Required]      [Bin]  (10.99  MB)
		4    [x]  [Required]      [Bundles]  (1.01  MB)
		5    [x] [Docs]  (4.75  MB)
		6    [x]  [Required]      [Domain]  (125.56  KB)
		7    [x]  [Required]      [Domain Shell Scripts]  (17.35  KB)
		8    [x]  [Required]      [Modules]  (147.01  MB)
		9    [x]  [Required]      [Standalone]  (152.77  KB)
		10 [x]  [Required]      [Standalone Shell Scripts]  (14.16  KB)
		11 [x]  [Required]      [Welcome Content]  (2.11  MB)
		12 [x] [Red Hat JBoss Enterprise Application Platform Natives]  (8  KB)
		13 [x] [Native RHEL7 x86_64]  (76  KB)
		14 [x] [Native Utils RHEL7 x86_64]  (53.04  KB)
		15 [x] [Native Webserver RHEL7 x86_64]  (254.97  KB)
		Total Size Required:  167.11  MB
		Press  0  to  confirm your selections
		Please select which packs you want to  install
		
		#随后是jboss用户名和密码（admin:jbossadmin5*）
		Admin username:  [admin]
		Admin password:  []
		*************
		Confirm admin password:  [*************]
		*************
		press  1  to  continue,  2  to  quit,  3  to  redisplay.
		
		#然后是演示文件的安装，我选择no
		Quickstarts
		Red Hat JBoss Enterprise Application Platform comes with  a  series of quickstart examples designed to  help users begin writing applications using the Java EE  6  technologies.  Would you like to  install quickstarts?
		0  [x]  No
		1  [  ]  Yes
		
		#最后是监听端口，默认是8080,
		Socket Bindings
		Configure the socket bindings for  Red Hat JBoss Enterprise Application Platform.
		Select Port Configuration:
		0  [x]  Use  the default  port bindings for  standalone and  domain modes.
		1  [  ]  Configure an offset for  all default  port bindings.
		2  [  ]  Configure custom port bindings.
		
		#紧接着是是否启用IPv6
		If  this  computer is  using  a  pure IPv6 configuration,  please check the box below.
		  [  ]  Enable pure IPv6 configuration
		
		#然后会询问是否启动服务器，先不启动
		Server Launch
		Choose server startup mode:
		0  [x]  Don't  start the server
		1  [  ]  Standalone Mode
		2  [  ]  Domain Mode
		
		#选择日志等级
		Configure the logging levels for  Red Hat JBoss Enterprise Application Platform?
		0  [x]  No
		1  [  ]  Yes
		#选择配置文件
		Configure runtime environment
		There are several additional options for  configuring Red Hat JBoss Enterprise Application Platform now that the server has been installed.  Each  option can be individually chosen,  and  will be configured in  the order displayed upon pressing next.  Would you like to  do  this  now?
		0  [x]  Perform default  configuration
		1  [  ]  Perform advanced configuration

- 经过一系列交互之后就会解包安装：![](https://i.imgur.com/eqaXqq5.jpg)
- 到这里已经完成JBoss EAP的安装，其实他有个基于web的控制面饭，但是服务器只监听127.0.0.1这个IP，如果需要监听其他IP或0.0.0.0，请修改以下文件：
>
		#打开文件进行修改
		[root@ejbca  ~]# vim /opt/ca/EAP-6.4.0/standalone/configuration/standalone.xml
		搜索：/127.0.0.1
		#定位到interface的节点和该节点上面的127.0.0.1，修改监听地址为0.0.0.0
	    <interfaces>
	        <interface  name="management">
	            <inet-address value="${jboss.bind.address.management:0.0.0.0}"/>
	        </interface>
	        <interface  name="public">
	            <inet-address value="${jboss.bind.address:0.0.0.0}"/>
	        </interface>
	        <!--  TODO  -  only show this  if  the jacorb subsystem is  added  -->
	        <interface  name="unsecure">
	            <!--
	              ~  Used for  IIOP sockets in  the standard configuration.
	              ~                  To  secure JacORB you need to  setup SSL
	              -->
	            <inet-address value="${jboss.bind.address.unsecure:0.0.0.0}"/>
	        </interface>
	    </interfaces>	

- 随后使用以下命令启动服务，在这里先不用脚本，因为后面需要重启服务。启动服务后再打开一个SSH窗口进行下面的步骤：
>
		[root@ejbca  ~]# /opt/ca/EAP-6.4.0/bin/standalone.sh
		（ps:此处不会用）顺便记一下关闭服务的命令：ctrl+c或者如下：
		ps -ef|grep jboss
		kill -9 进程号

- 确认JBoss EAP服务启动后即可通过浏览器打开管理界面：访问http:by.cos7:8080/
## 配置jboss的mysql数据源 ##
- 创建目录，然后在该目录下创建module.xml
>
		mkdir -p /opt/ca/EAP-6.4.0/modules/com/mysql/main/
		cd /opt/ca/EAP-6.4.0/modules/com/mysql/main
		vi module.xml

- module.xml内容如下：
>
		<?xml version="1.0" encoding="UTF-8"?>
			<module xmlns="urn:jboss:module:1.0" name="com.mysql">
			<resources>
				<resource-root path="mysql-connector-java-5.1.27.jar"/>
			</resources>
			<dependencies>
				<module name="javax.api"/>
				<module name="javax.transaction.api"/>
				<module name="org.slf4j"/>
			</dependencies>
		</module>

- 将mysql的驱动包`mysql-connector-java-5.1.27.jar`，拷贝到当前目录xxx/mian
>
		cp /home/ca/Downloads/mysql-connector-java-5.1.27.jar ./

- 打开新的shell窗口，运行
>
		[root@by EAP-6.4.0]# /opt/ca/EAP-6.4.0/bin/jboss-cli.sh
		You are disconnected at  the moment.  Type  'connect'  to  connect to  the server or  'help'  for  the list of supported commands.
		[disconnected  /]

- 如果提示disconnected，则需要输入connect并回车，如果一切正常则显示：
>
		[standalone@localhost:9999  /]

- 然后输入以下命令在JBoss中注册mysql驱动：
>
		#注册驱动
		/subsystem=datasources/jdbc-driver=com.mysql.jdbc.Driver:add(driver-name=com.mysql.jdbc.Driver,driver-class-name=com.mysql.jdbc.Driver,driver-module-name=com.mysql,driver-xa-datasource-class-name=com.mysql.jdbc.jdbc.jdbc2.optional.MysqlXADataSource)
		#重新加载
		:reload

- 完成后输入exit回车即可退出。
## 安装ejbca ##
- 　从ejbca官方网站下载ejbca安装包：ejbca_ce_6_3_1_1.zip，放在/usr/file目录，解压，准备修改配置
>
		unzip /usr/file/ejbca_ce_6_3_1_1.zip -d /usr/java
		cd /usr/java
		mv ejbca_ce_6_3_1_1 ejbca-ce-6.3.1.1
		cd /usr/java/ejbca-ce-6.3.1.1/conf/
		# 配置文件目录包含以下文件：
		ll -h

- EJBCA只加载后缀名为properties的文件，否则则加载默认配置。在这里我们需要修改以下文件：
>
		database.properties：数据库配置文件
		ejbca.properties：EJBCA配置文件
		install.properties：安装配置文件
		web.properties：web服务配置文件

- 首先是数据库配置文件，先将文件复制一份然后进行修改:
>
		# 复制
		cp database.properties.sample database.properties
		# 修改
		vim database.properties
		# 修改以下内容
		datasource.jndi-name=jboss/datasources/MySqlDS
		database.name=mysql
		database.url=jdbc:mysql://by.cos7:3306/ejbca?characterEncoding=UTF-8
		database.driver=org.mysql.jdbc.Driver
		database.username=root
		database.password=123456

- 然后复制并修改EJBCA配置文件:
>
		# 复制
		cp ejbca.properties.sample ejbca.properties
		# 修改
		vim ejbca.properties
		# 修改以下内容
		appserver.home=/opt/ca/EAP-6.4.0
		appserver.type=jboss
		approval.defaultrequestvalidity=28800 #保持默认状态
		approval.defaultapprovalvalidity=28800 #保持默认状态
		ejbca.passwordlogrounds=8 #设置密码为8位

- 然后修改安装配置文件：
>
		# 复制
		cp install.properties.sample install.properties
		# 修改
		vim install.properties
		# 修改以下内容
		ca.name=EnginxManagementCA
		ca.dn=CN=TestManagementCA,O=Test,C=CN
		ca.tokentype=soft
		ca.tokenpassword=null
		ca.keyspec=4096
		ca.keytype=RSA
		ca.signaturealgorithm=SHA256WithRSA
		ca.validity=3650
		ca.policy=null
		ca.certificateprofile=ROOTCA #要修改

- 最后是web服务配置文件：
>
		# 复制
		cp web.properties.sample web.properties
		# 修改
		vim web.properties
		# 修改以下内容
		java.trustpassword=changeit  #默认即可
		superadmin.cn=SuperAdmin  #默认即可
		superadmin.dn=CN=${superadmin.cn},O=Test,C=CN
		superadmin.password=ejbca  #默认即可
		superadmin.batch=true
		httpsserver.password=serverpwd  #默认即可
		httpsserver.hostname=by.cos7
		httpsserver.dn=CN=${httpsserver.hostname},O=Test,C=CN
		httpserver.pubhttp=8080 #不修改
		httpserver.pubhttps=8442 #不修改
		httpserver.privhttps=8443 #不修改
## 部署ejbca到jboss ##
- deploy用ant部署，install生成证书，deploy-keystore将证书部署到jboss，前两步所需时间较长，过程中如需输入，请直接回车:
>
		cd /opt/ca/ejbca-ce-6.3.1.1
		ant clean deploy
		ant install #成功后会生成数据库表
		ant deploy-keystore
## 配置jboss的https ##
- 打开新的shell窗口，运行
>
		/opt/ca/EAP-6.4.0/bin/jboss-cli.sh
		如果是“disconnect”状态，运行“connect”，多回车几次，准备运行下面4部分配置

- 第一部分（配置任意主机可访问）
>
		/interface=http:add(inet-address="0.0.0.0")
		/interface=httpspub:add(inet-address="0.0.0.0")
		/interface=httpspriv:add(inet-address="0.0.0.0")
		/socket-binding-group=standard-sockets/socket-binding=http:add(port="8080",interface="http")
		/subsystem=undertow/server=default-server/http-listener=http:add(socket-binding=http)
		/subsystem=undertow/server=default-server/http-listener=http:write-attribute(name=redirect-socket, value="httpspriv")
		:reload

- 第二部分（配置证书））
>
		/core-service=management/security-realm=SSLRealm:add()
		/core-service=management/security-realm=SSLRealm/server-identity=ssl:add(keystore-path="${jboss.server.config.dir}/keystore/keystore.jks", keystore-password="serverpwd", alias="prod-ica1")
		/core-service=management/security-realm=SSLRealm/authentication=truststore:add(keystore-path="${jboss.server.config.dir}/keystore/truststore.jks", keystore-password="changeit")
		/socket-binding-group=standard-sockets/socket-binding=httpspriv:add(port="8443",interface="httpspriv")
		/socket-binding-group=standard-sockets/socket-binding=httpspub:add(port="8442", interface="httpspub")
		:reload


- 第三部分（配置ssl））
>
		/subsystem=undertow/server=default-server/https-listener=httpspriv:add(socket-binding=httpspriv, security-realm="SSLRealm", verify-client=REQUIRED)
		/subsystem=undertow/server=default-server/https-listener=httpspub:add(socket-binding=httpspub, security-realm="SSLRealm")
		:reload

- 第四部分（配置web service）
>
		/system-property=org.apache.tomcat.util.buf.UDecoder.ALLOW_ENCODED_SLASH:add(value=true)
		/system-property=org.apache.catalina.connector.CoyoteAdapter.ALLOW_BACKSLASH:add(value=true)
		/system-property=org.apache.catalina.connector.URI_ENCODING:add(value="UTF-8")
		/system-property=org.apache.catalina.connector.USE_BODY_ENCODING_FOR_QUERY_STRING:add(value=true)
		/subsystem=webservices:write-attribute(name=wsdl-host, value=jbossws.undefined.host)
		/subsystem=webservices:write-attribute(name=modify-wsdl-address, value=true)
		:reload

## 使用ejbca community 6.3.1.1 ##
- 假设ejbca服务器地址：172.17.210.124，编辑windows的“C:\Windows\System32\drivers\etc”下的hosts文件，加入一行`172.17.210.124 by.cos7`
- 拷贝ejbca服务器“/usr/java/ejbca-ce-6.3.1.1/p12/”下的superadmin.p12到windows，双击安装，密码“ejbca”,ejbca提供了两个界面
- 用户界面:`http://by.cos7:8080/ejbca/`
- 管理员界面（需要证书，使用刚才安装的superadmin证书）:`https://by.cos7:8443/ejbca/adminweb/`
- 图片参考：![](https://i.imgur.com/10L0sEU.jpg)![](https://i.imgur.com/1hKKnKm.jpg)![](https://i.imgur.com/TvEsmb5.jpg)![](https://i.imgur.com/jizg2OD.png)![](https://i.imgur.com/4SjRPld.png)
# 三、参考链接 #
- `https://www.jianshu.com/p/56051594fee9`
- `https://www.bbsmax.com/A/xl56xw2Yzr/`
- `https://piggsoft.com/2018/07/11/PKI%E4%BD%93%E7%B3%BB%EF%BC%88%E4%B8%89%EF%BC%89-EJBCA%E5%AE%89%E8%A3%85/`
- (centos7安装mysql)`https://www.cnblogs.com/julyme/p/5969626.html`
- (centos改主机名)`https://www.jianshu.com/p/39d7000dfa47`
- (EJBCA下载)`https://www.ejbca.org/download.html`
- (JBoss EAP) `https://developers.redhat.com/products/eap/download/)`6.4.0
- (JDK下载)`https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html`