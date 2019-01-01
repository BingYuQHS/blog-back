---
title: hexo+next博客
date: 2018-10-11 13:50:49
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>

## 环境准备 ##
- 安装node.js (npm或淘宝镜像cnpm)
- 安装git 配饰ssh等
- 下载安装hexo  (cnpm install -g hexo)

## 本地搭建hexo静态博客 ##
- 新建文件夹，如Blog
- 进入文件夹右键运行git,输入hexo init(生成hexo模板，翻墙？)
- 运行cnpm install
- 运行hexo server(运行程序，访问本地localhost:4000可看到博客已搭建成功)

## 将博客与Github关联 ##
- 在Github上创建名字为XXX.github.io的项目，XXX为github的用户名
- 打开本地的MyBlog文件夹项目内的_config.yml配置文件，将其中的type设置为git
> 
		deploy:
		  type: git
		  repository: https://github.com/BingYuQHS/BingYuQHS.github.io.git
		  branch: master
- 运行：cnpm install --save hexo-deployer-git
- 运行：hexo clean(删除本地Public目录)
- 运行：hexo g(本地生成静态文件)
- 运行：hexo d(将本地静态文件推送至Github)
- 浏览器访问：https://bingyuqhs.github.io/

## 绑定域名 ##
- 博客已经搭建好，能通过github的域名访问，也可以绑定自己的域名。（ps:具体的方法后面补充）

## 更换主题 ##
- Next主题为例
- 下载主题,下载到Blog目录的themes主题下的next文件夹中
> 
    
		git clone https://github.com/iissnan/hexo-theme-next themes/next
- 打开站点的_config.yml配置文件，修改主题为next(主题项目的文件夹名称)
>
    	theme: next

- 打开主题(不是站点下的)的_config.yml配置文件，选择next主题的样式
>
	    # Scheme Settings
    	# Schemes
    	#scheme: Muse
    	scheme: Mist
    	#scheme: Pisces
    	#scheme: Gemini
- 选择好后，再次部署网站，hexo g、hexo d，查看效果，可在本地hexo server查看后再执行hexo d命令。

## 发布文章 ##
- 创建博文
>
		hexo n "博客名字" (= hexo new "博客名字")
		hexo s --debug   (= hexo server --debug，在本地调样式时使用)
		hexo clean		 (删除本地地Public目录)
		hexo g	        （本地生成静态文件）
		hexo d			 (将本地静态文件推送至Github)
- 执行命令后，会发现在Blog根目录下的source文件夹中的_post文件夹中多了一个 博客名字.md 文件，使用Markdown编辑器打开可进行博文编写。
- 写好博文并且样式无误后，通过hexo g、hexo d 生成、部署网页。随后可以在浏览器中输入域名浏览。(ps:可以hexo server --debug命令进行本地调试修改)

## 寻找图床 ##
- 图床，当博文中有图片时，若是少量图片，可以直接把图片存放在source文件夹中，但这显然不合理的，因为图片会占据大量的存储的空间，加载的时候相对缓慢 ，这时考虑把博文里的图片上传到某一网站，然后获得外部链接，使用Markdown语法``![图片信息](外部链接)``，完成图片的插入，这种网站就被成为图床。
- 常见的简易的图床网站有：[贴图库图片外链](http://www.tietuku.com/, "") 国内算比较好的图床我认为有两个：新浪微博和 [七牛云](https://www.qiniu.com/, "七牛云官网") ，七牛云的使用方法可以参看其他文章。图床最重要的就是稳定速度快，所以在挑选图床的时候一定要仔细，下图是博文插入图片，使用图床外链的示例：

## 个性化设置 ##
### 修改基本站点信息 ###
- 在站点配置文件_config.yml修改基本的站点信息
> 
	    # Site
	    title: Flamingo's Blog
	    subtitle: hs
	    description: RunDouble
	    keywords: 
	    author: hsQin
	    avatar: headicon_link
	    language: zh-Hans
	    timezone: Asia/Shanghai
- 依次是网站标题、副标题、网站描述、作者、网站头像外部链接、网站语言、时区等。
### 修改基本主题信息 ###
- 在主题配置文件_config.yml修改基本的主题信息
> 
	    # Reward
		#reward_comment: Donate comment here
		wechatpay: /images/wechatpay.jpg
		alipay: /images/alipay.jpg
		#bitcoin: /images/bitcoin.png
- 博文打赏的微信、支付宝二维码图片，可以直接把图片放在根目录的source文件夹中，也可以使用图床外链。
> 
	    # Social Links.
		# Usage: `Key: permalink || icon`
		# Key is the link label showing to end users.
		# Value before `||` delimeter is the target permalink.
		# Value after `||` delimeter is the name of FontAwesome icon. If icon (with or without delimeter) is not specified, globe icon will be loaded.
		social:
		  GitHub: https://github.com/bingyuqhs || github
		  #E-Mail: mailto:yourname@gmail.com || envelope
		  #Google: https://plus.google.com/yourname || google
		  #Twitter: https://twitter.com/yourname || twitter
		  #FB Page: https://www.facebook.com/yourname || facebook
		  #VK Group: https://vk.com/yourname || vk
		  #StackOverflow: https://stackoverflow.com/yourname || stack-overflow
		  #YouTube: https://youtube.com/yourname || youtube
		  #Instagram: https://instagram.com/yourname || instagram
		  #Skype: skype:yourname?call|chat || skype
		social_icons:
		  enable: true
		  GitHub: github
		  icons_only: false
		  transition: false
- 社交外链的设置，即在侧栏展示你的个人社交网站信息。
> 
	    # Share
		# This plugin is more useful in China, make sure you known how to use it.
		# And you can find the use guide at official webiste: http://www.jiathis.com/.
		# Warning: JiaThis does not support https.
		#jiathis:
		  ##uid: Get this uid from http://www.jiathis.com/
		#add_this_id:
- 博文分享的插件jiathis,值设置为true。
### 个性化进阶 ###
#### 添加网易云音乐 ####
#### 设置背景 ####
#### 增加菜单条目 ####
## 异地同步博客 ##
- 如何将两台电脑上的博客内容同步，及两台电脑上都可以编辑更新博客？要解决这个问题，首先要清楚博客文件地组成：
>
		node_modules
		public
		scaffolds
		source
		themes
		_config_yml
		db.json
		package.json
		.deploy_git
- 以上是利用hexo生成的博客的全部内容，当我们执行hexo d时，真正被推送到github上的有哪些内容呢？
- 看下guthub上的bingyuqhs.github.io项目，发现里面只有Public目录下的内容。也就是说，博客上呈现的内容其实就是public下的文件内容。那么这个public目录是怎么生成的呢？
- 一开始hexo init的时候是没有public目录的，而运行hexo g命令时，public目录生成了，也就是说hexo g命令就是用来生成博客文件的（会根据_config.yml，source目录文件以及themes目录下文件生成）。运行hexo clean命令时，public目录被删除了。
- 现在，我们知道决定博客显示内容的只有一个Public目录，而public目录又是可以动态生成的，那么其实我们只要在不同电脑上同步可以生成Public目录的文件即可。
- 以下文件及目录是必须要同步的：
>  
		source
		themes
		_config.yml
		db.json
		package.json
		.deploy_git

- 同步的方式有很多种，可以每次更新后都备份到一个地址。可以采用github去备份，新建一个项目用来存放以上文件，每次更新后推送的gihub上，用作备份同步。
- 同步完必须的文件后，怎样在其他电脑上也可以更新博客呢？
> 
		1 下载node.js并安装（官网下载安装），默认会安装npm.
		2 下载安装git(官网下载安装)
		3 下载安装hexo(方法：npm install -g hexo)【要翻墙】
		4 新建一个文件夹，如Blog
		5 进入该文件夹内，右键运行git,输入命令：hexo init(生成hexo模板，可能要翻墙)

- 我们重复一开始搭建博客的步骤，重新生成了一个新的模板，这个模板中包含了hexo生成的一些文件。
> 
		1 git clone我们备份的项目，生成一个文件夹，如BlogData
		2 将Blog里面的node_moudules、scaffolds文件夹复制到BlogData里面。

- 做完这些，从表面上看，两台电脑上的BlogData目录下的文件应该都是一样的了，现在运行hexo g、hexo d试试，如果报错就往下看。
> 
		这是因为.deploy_git没同步，在BlogData目录内运行npm install hexo-deployer-git --save后再次推送即可	
- 总结流程：当我们每次更新Blog内容后，先利用hexo将public推送到github，然后再将其余必须同步的文件利用git推送到github上。