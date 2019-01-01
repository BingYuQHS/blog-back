---
title: shiro知识总结
date: 2018-12-29 10:49:15
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
## shiro初探 ##
### 介绍 ###
shiro安全框架是目前为止作为登录注册最常用的框架，因为它十分的强大简单，提供了认证、授权、加密和会话管理等功能 。

----------
　　shiro能做什么？

　　　　　　　认证：验证用户的身份

　　　　　　　授权：对用户执行访问控制：判断用户是否被允许做某事

　　　　　　　会话管理：在任何环境下使用 Session API，即使没有 Web 或EJB 容器。

　　　　　　　加密：以更简洁易用的方式使用加密功能，保护或隐藏数据防止被偷窥

　　　　　　　Realms：聚集一个或多个用户安全数据的数据源

　　　　　　　单点登录（SSO）功能。

　　　　　　　为没有关联到登录的用户启用 "Remember Me“ 服务

----------

　　Shiro 的四大核心部分

　　　　　　Authentication(身份验证)：简称为“登录”，即证明用户是谁。

　　　　　　Authorization(授权)：访问控制的过程，即决定是否有权限去访问受保护的资源。

　　　　　　Session Management(会话管理)：管理用户特定的会话，即使在非 Web 或 EJB 应用程序。

　　　　　　Cryptography(加密)：通过使用加密算法保持数据安全

----------

　　shiro的三个核心组件：　　　　　

　　　　　　Subject ：正与系统进行交互的人，或某一个第三方服务。所有 Subject 实例都被绑定到（且这是必须的）一个SecurityManager 上。

　　　　　　SecurityManager：Shiro 架构的心脏，用来协调内部各安全组件，管理内部组件实例，并通过它来提供安全管理的各种服务。当 Shiro 与一个 Subject 进行交互时，实质上是幕后的 SecurityManager 处理所有繁重的 Subject 安全操作。

　　　　　　Realms ：本质上是一个特定安全的 DAO。当配置 Shiro 时，必须指定至少一个 Realm 用来进行身份验证和/或授权。Shiro 提供了多种可用的 Realms 来获取安全相关的数据。如关系数据库(JDBC)，INI 及属性文件等。可以定义自己 Realm 实现来代表自定义的数据源。
### 实践一 ###
创建java项目，在src下新建配置文件shiro.ini，定义和安全相关的数据：用户，角色和权限;

	#定义用户
	[users]
	#用户名 zhang3  密码是 12345， 角色是 admin
	zhang3 = 12345, admin 
	#用户名 li4  密码是 abcde， 角色是 产品经理
	li4 = abcde,productManager
	#定义角色
	[roles]
	#管理员什么都能做
	admin = *
	#产品经理只能做产品管理
	productManager = addProduct,deleteProduct,editProduct,updateProduct,listProduct
	#订单经理只能做订单管理
	orderManager = addOrder,deleteOrder,editOrder,updateOrder,listOrder

准备用户类User.java；

	package com.flamingo;
	
	public class User {
	
		/**账号*/
		private String name;
		/**密码*/
		private String password;
	
		public User(String name, String password){
			this.name = name;
			this.password = password;
		}
		public User(){
	
		}
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		public String getPassword() {
			return password;
		}
		public void setPassword(String password) {
			this.password = password;
		}
		
	}
准备3个用户，前两个能在 shiro.ini 中找到，第3个不存在,然后测试登录,接着测试是否包含角色,最后测试是否拥有权限;

注：Subject 在 Shiro 这个安全框架下， Subject 就是当前用户
	package com.flamingo;

	
	import java.util.ArrayList;
	import java.util.List;
	
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.config.IniSecurityManagerFactory;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.util.Factory;
	
	public class TestShiro {
	    public static void main(String[] args) {
	    	//用户
			List<User> users = new ArrayList<>();
	    	User zhang3 = new User("zhang3","12345");
	    	User li4 = new User("li4","abcde");
	    	User wang5 = new User("wang5","wrongpassword");
	    	users.add(zhang3);
	    	users.add(li4);
	    	users.add(wang5);
	
	    	//角色
			List<String> roles = new ArrayList<>();
	    	String roleAdmin = "admin";
	    	String roleProductManager ="productManager";
	    	roles.add(roleAdmin);
	    	roles.add(roleProductManager);
	    	
	    	//权限
			List<String> permits = new ArrayList<>();
	    	String permitAddProduct = "addProduct";
	    	String permitAddOrder = "addOrder";
	    	permits.add(permitAddProduct);
	    	permits.add(permitAddOrder);
	
	    	//登陆每个用户
	    	for (User user : users) {
	    		if(login(user)) 
	    			System.out.printf("%s \t成功登陆，用的密码是 %s\t %n",user.getName(),user.getPassword());
	    		else 
	    			System.out.printf("%s \t成功失败，用的密码是 %s\t %n",user.getName(),user.getPassword());
			}
	
	    	System.out.println("-------分割线------");
	    	
	    	//判断能够登录的用户是否拥有某个角色
	    	for (User user : users) {
	    		for (String role : roles) {
	    			if(login(user)) {
		    			if(hasRole(user, role)) 
		    				System.out.printf("%s\t 拥有角色: %s\t%n",user.getName(),role);
		    			else
		    				System.out.printf("%s\t 不拥有角色: %s\t%n",user.getName(),role);
	    			}
	    		}	
			}
	    	System.out.println("-------分割线------");
	
	    	//判断能够登录的用户，是否拥有某种权限
	    	for (User user : users) {
	    		for (String permit : permits) {
	    			if(login(user)) {
	    				if(isPermitted(user, permit)) 
	    					System.out.printf("%s\t 拥有权限: %s\t%n",user.getName(),permit);
	    				else
	    					System.out.printf("%s\t 不拥有权限: %s\t%n",user.getName(),permit);
	    			}
	    		}	
	    	}
	    }
	
		private static boolean login(User user) {
			Subject subject= getSubject(user);
			//如果已经登录过了，退出
			if(subject.isAuthenticated())
				subject.logout();
			
			//封装用户的数据
			UsernamePasswordToken token = new UsernamePasswordToken(user.getName(), user.getPassword());
			try {
				//将用户的数据token 最终传递到Realm中进行对比
				subject.login(token);
			} catch (AuthenticationException e) {
				//验证错误
				return false;
			}				
			
			return subject.isAuthenticated();
		}
	
		private static boolean hasRole(User user, String role) {
			Subject subject = getSubject(user);
			return subject.hasRole(role);
		}
	
		private static boolean isPermitted(User user, String permit) {
			Subject subject = getSubject(user);
			return subject.isPermitted(permit);
		}
	
		/**
		 * Subject 在 Shiro 这个安全框架下， Subject 就是当前用户
		 * @param user
		 * @return
		 */
		private static Subject getSubject(User user) {
			//加载配置文件，并获取工厂
			Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
			//获取安全管理者实例
			SecurityManager sm = factory.getInstance();
			//将安全管理者放入全局对象
			SecurityUtils.setSecurityManager(sm);
			//全局对象通过安全管理者生成Subject对象
			Subject subject = SecurityUtils.getSubject();
	
			return subject;
		}
	}

结果；

![](https://i.imgur.com/onH2i9R.png)

## shiro 数据库支持 ##
### RBAC 概念 ###
把权限相关的内容放在数据库里；

RBAC 是当下权限系统的设计基础，同时有两种解释：

1. Role-Based Access Control，基于角色的访问控制
即，你要能够删除产品，那么当前用户就必须拥有产品经理这个角色
2. Resource-Based Access Control，基于资源的访问控制
即，你要能够删除产品，那么当前用户就必须拥有删除产品这样的权限
### 表结构 ###
基于 RBAC 概念，就会存在3 张基础表： 用户，角色，权限， 以及 2 张中间表来建立 用户与角色的多对多关系，角色与权限的多对多关系。 用户与权限之间也是多对多关系，但是是通过 角色间接建立的
    DROP DATABASE IF EXISTS shiro;
	CREATE DATABASE shiro DEFAULT CHARACTER SET utf8;
	USE shiro;
	 
	drop table if exists user;
	drop table if exists role;
	drop table if exists permission;
	drop table if exists user_role;
	drop table if exists role_permission;
	 
	create table user (
	  id bigint auto_increment,
	  name varchar(100),
	  password varchar(100),
	  constraint pk_users primary key(id)
	) charset=utf8 ENGINE=InnoDB;
	 
	create table role (
	  id bigint auto_increment,
	  name varchar(100),
	  constraint pk_roles primary key(id)
	) charset=utf8 ENGINE=InnoDB;
	 
	create table permission (
	  id bigint auto_increment,
	  name varchar(100),
	  constraint pk_permissions primary key(id)
	) charset=utf8 ENGINE=InnoDB;
	 
	create table user_role (
	  uid bigint,
	  rid bigint,
	  constraint pk_users_roles primary key(uid, rid)
	) charset=utf8 ENGINE=InnoDB;
	 
	create table role_permission (
	  rid bigint,
	  pid bigint,
	  constraint pk_roles_permissions primary key(rid, pid)
	) charset=utf8 ENGINE=InnoDB;

	//这里基于 Shiro初探中的shiro.ini 文件，插入一样的用户，角色和权限数据。
	INSERT INTO `permission` VALUES (1,'addProduct');
	INSERT INTO `permission` VALUES (2,'deleteProduct');
	INSERT INTO `permission` VALUES (3,'editProduct');
	INSERT INTO `permission` VALUES (4,'updateProduct');
	INSERT INTO `permission` VALUES (5,'listProduct');
	INSERT INTO `permission` VALUES (6,'addOrder');
	INSERT INTO `permission` VALUES (7,'deleteOrder');
	INSERT INTO `permission` VALUES (8,'editOrder');
	INSERT INTO `permission` VALUES (9,'updateOrder');
	INSERT INTO `permission` VALUES (10,'listOrder');
	INSERT INTO `role` VALUES (1,'admin');
	INSERT INTO `role` VALUES (2,'productManager');
	INSERT INTO `role` VALUES (3,'orderManager');
	INSERT INTO `role_permission` VALUES (1,1);
	INSERT INTO `role_permission` VALUES (1,2);
	INSERT INTO `role_permission` VALUES (1,3);
	INSERT INTO `role_permission` VALUES (1,4);
	INSERT INTO `role_permission` VALUES (1,5);
	INSERT INTO `role_permission` VALUES (1,6);
	INSERT INTO `role_permission` VALUES (1,7);
	INSERT INTO `role_permission` VALUES (1,8);
	INSERT INTO `role_permission` VALUES (1,9);
	INSERT INTO `role_permission` VALUES (1,10);
	INSERT INTO `role_permission` VALUES (2,1);
	INSERT INTO `role_permission` VALUES (2,2);
	INSERT INTO `role_permission` VALUES (2,3);
	INSERT INTO `role_permission` VALUES (2,4);
	INSERT INTO `role_permission` VALUES (2,5);
	INSERT INTO `role_permission` VALUES (3,6);
	INSERT INTO `role_permission` VALUES (3,7);
	INSERT INTO `role_permission` VALUES (3,8);
	INSERT INTO `role_permission` VALUES (3,9);
	INSERT INTO `role_permission` VALUES (3,10);
	INSERT INTO `user` VALUES (1,'zhang3','12345');
	INSERT INTO `user` VALUES (2,'li4','abcde');
	INSERT INTO `user_role` VALUES (1,1);
	INSERT INTO `user_role` VALUES (2,2);
### 实践二 ###
#### user ####
	package com.flamingo;

	public class User {
	
		private int id;
		private String name;
		private String password;
	
		public User(String name, String password){
			this.name = name;
			this.password = password;
		}
		public User(){
	
		}
	
		public String getName() {
			return name;
		}
		public void setName(String name) {
			this.name = name;
		}
		public String getPassword() {
			return password;
		}
		public void setPassword(String password) {
			this.password = password;
		}
		public int getId() {
			return id;
		}
		public void setId(int id) {
			this.id = id;
		}
	}
#### DAO ####
DAO提供了和权限相关查询;
	package com.flamingo;
	
	import java.sql.Connection;
	import java.sql.DriverManager;
	import java.sql.PreparedStatement;
	import java.sql.ResultSet;
	import java.sql.SQLException;
	import java.util.HashSet;
	import java.util.Set;
	
	
	public class DAO {
		public DAO() {
			try {
				Class.forName("com.mysql.jdbc.Driver");
			} catch (ClassNotFoundException e) {
				e.printStackTrace();
			}
		}
	
		public Connection getConnection() throws SQLException {
			return DriverManager.getConnection("jdbc:mysql://x.x.x.x:3306/shiro?characterEncoding=UTF-8", "root",
					"123456");
		}
	
		/**
		 * 根据用户名查询密码，这样既能判断用户是否存在，也能判断密码是否正确
		 * @param userName
		 * @return
		 */
		public String getPassword(String userName) {
			String sql = "select password from user where name = ?";
			try (Connection c = getConnection(); PreparedStatement ps = c.prepareStatement(sql);) {
				ps.setString(1, userName);
				ResultSet rs = ps.executeQuery();
				if (rs.next())
					return rs.getString("password");
			} catch (SQLException e) {
				e.printStackTrace();
			}
			return null;
		}
	
		/**
		 * 根据用户名查询此用户有哪些角色，这是3张表的关联
		 * @param userName
		 * @return
		 */
		public Set<String> listRoles(String userName) {
			Set<String> roles = new HashSet<>();
			String sql = "select r.name from user u "
					+ "left join user_role ur on u.id = ur.uid "
					+ "left join role r on r.id = ur.rid "
					+ "where u.name = ?";
			try (Connection c = getConnection(); PreparedStatement ps = c.prepareStatement(sql);) {
				ps.setString(1, userName);
				ResultSet rs = ps.executeQuery();
	
				while (rs.next()) {
					roles.add(rs.getString(1));
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			return roles;
		}
	
		/**
		 * 根据用户名查询此用户有哪些权限，这是5张表的关联
		 * @param userName
		 * @return
		 */
		public Set<String> listPermissions(String userName) {
			Set<String> permissions = new HashSet<>();
			String sql = 
				"select p.name from user u "+
				"left join user_role ru on u.id = ru.uid "+
				"left join role r on r.id = ru.rid "+
				"left join role_permission rp on r.id = rp.rid "+
				"left join permission p on p.id = rp.pid "+
				"where u.name =?";
			
			try (Connection c = getConnection(); PreparedStatement ps = c.prepareStatement(sql);) {
				ps.setString(1, userName);
				ResultSet rs = ps.executeQuery();
				while (rs.next()) {
					permissions.add(rs.getString(1));
				}
			} catch (SQLException e) {
				e.printStackTrace();
			}
			return permissions;
		}
		public static void main(String[] args) {
			System.out.println(new DAO().listRoles("zhang3"));
			System.out.println(new DAO().listRoles("li4"));
			System.out.println(new DAO().listPermissions("zhang3"));
			System.out.println(new DAO().listPermissions("li4"));
		}
	}
#### Realm ####
当应用程序向 Shiro 提供了 账号和密码之后,Shiro 就会问 Realm 这个账号密码是否对， 如果对的话，其所对应的用户拥有哪些角色，哪些权限。Realm 得到了 Shiro 给的用户和密码后，有可能去找 ini 文件，就像Shiro 初探中的 shiro.ini，也可以去找数据库，就如同本知识点中的 DAO 查询信息。Realm才是真正进行用户认证和授权的关键地方。
可以自定义，也有封装好的JdbcRealm；
	package com.flamingo;
	
	import java.util.Set;
	
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.SimpleAuthenticationInfo;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.authz.AuthorizationInfo;
	import org.apache.shiro.authz.SimpleAuthorizationInfo;
	import org.apache.shiro.realm.AuthorizingRealm;
	import org.apache.shiro.subject.PrincipalCollection;
	
	/**
	 *  自定义Realm
	 *  DatabaseRealm 这个类，用户提供，但是不由用户自己调用，而是由 Shiro 去调用。
	 *  就像Servlet的doPost方法，是被Tomcat调用一样
	 */
	public class DatabaseRealm extends AuthorizingRealm {
	
		/**
		 * 授权
		 * @param principalCollection
		 * @return
		 */
		@Override
		protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
			//能进入到这里，表示账号已经通过验证了
			String userName =(String) principalCollection.getPrimaryPrincipal();
			//通过DAO获取角色和权限
			Set<String> permissions = new DAO().listPermissions(userName);
			Set<String> roles = new DAO().listRoles(userName);
			
			//授权对象
			SimpleAuthorizationInfo s = new SimpleAuthorizationInfo();
			//把通过DAO获取到的角色和权限放进去
			s.setStringPermissions(permissions);
			s.setRoles(roles);
			return s;
		}
	
		/**
		 * 验证
		 * @param token
		 * @return
		 * @throws AuthenticationException
		 */
		@Override
		protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
			//获取账号密码
			UsernamePasswordToken t = (UsernamePasswordToken) token;
			String userName= token.getPrincipal().toString();
			String password= new String( t.getPassword());
			//获取数据库中的密码
			String passwordInDB = new DAO().getPassword(userName);
	
			//如果为空就是账号不存在，如果不相同就是密码错误，但是都抛出AuthenticationException，而不是抛出具体错误原因，免得给破解者提供帮助信息
			if(null==passwordInDB || !passwordInDB.equals(password)) 
				throw new AuthenticationException();
			
			//认证信息里存放账号密码, getName() 是当前Realm的继承方法,通常返回当前类名 :getName() == databaseRealm
			System.out.println(getName());
			SimpleAuthenticationInfo a = new SimpleAuthenticationInfo(userName,password,getName());
			return a;
		}
	
	}
#### 修改 shiro.ini ####
DatabaseRealm 这个类，用户提供，但是不由用户自己调用，而是由 Shiro 去调用。 就像Servlet的doPost方法，是被Tomcat调用一样。
那么 Shiro 怎么找到这个 Realm 呢？就需要修改 shiro.ini;

	#配置shiro.ini,使Shiro找到这个Realm
	[main]
	databaseRealm=com.flamingo.DatabaseRealm
	securityManager.realms=$databaseRealm
#### 测试和结果 ####
与实践一 一致

## 加密 ##
### md5加密 ###
	如代码所示 123 用 md5 加密后，得到字符串： 202CB962AC59075B964B07152D234B70
	这个字符串，却无法通过计算，反过来得到源密码是 123.
	这个加密后的字符串就存在数据库里了，下次用户再登陆，输入密码 123， 同样用md5 加密后，再和这个字符串一比较，就知道密码是否正确了。
	如此这样，既能保证用户密码校验的功能，又能保证不暴露密码。
	
	package com.flamingo;
	import org.apache.shiro.crypto.hash.Md5Hash;
	public class TestEncryption {
	    public static void main(String [] args){
	        String password = "123";
	        String encodedPassword = new Md5Hash(password).toString();
	        System.out.println(encodedPassword);
	    }
	}
### 盐 ###
上面讲了md5加密，但是md5加密又有一些缺陷:


1. 如果我的密码是 123,你的也是 123, 那么md5的值是一样的，那么通过比较加密后的字符串，我就可以反推过来，原来你的密码也是123.
2. 与上述相同，虽然 md5 不可逆，但是我可以穷举法呀，我把特别常用的100万或者更多个密码的 md5 值记录下来，比如12345的，abcde的。 相当一部分人用的密码也是这些，那么只要到数据库里一找，也很快就可以知道原密码是多少了。这样看上去也就破解了，至少一部分没有想象中那么安全吧。

为了解决这个问题，引入了盐的概念。 盐是什么意思呢？ 比如炒菜，直接使用md5，就是对食材（源密码）进行炒菜，因为食材是一样的，所以炒出来的味道都一样，可是如果加了不同分量的盐，那么即便食材一样，炒出来的味道也就不一样了。

所以，虽然每次 123 md5 之后都是202CB962AC59075B964B07152D234B70，但是 我加上盐，即 123+随机数，那么md5值不就不一样了吗？ 这个随机数，就是盐，而这个随机数也会在数据库里保存下来，每个不同的用户，随机数也是不一样的。
再就是加密次数，加密一次是202CB962AC59075B964B07152D234B70，我可以加密两次呀，就是另一个数了。 而黑客即便是拿到了加密后的密码，如果不知道到底加密了多少次，也是很难办的。
	package com.flamingo;

	import org.apache.shiro.crypto.SecureRandomNumberGenerator;
	import org.apache.shiro.crypto.hash.SimpleHash;
	
	public class TestEncryption {
	    public static void main(String [] args){
	        String password = "123";
	        String salt = new SecureRandomNumberGenerator().nextBytes().toString();
	        int times = 2;
	        String algorithmName = "md5";
	        String encodedPassword = new SimpleHash(algorithmName,password,salt,times).toString();
	        System.out.printf("password: %s , salt: %s, times: %d, encodedPassword:%s ",password,salt,times,encodedPassword);
	    }
	}

### 数据库调整 ###
加入对加密的支持。 在开始之前，要修改一下user表，加上盐 字段: salt。因盐是随机数，得保留下来，如果不知道盐巴是多少，我们也就没法判断密码是否正确了。

	alter table user add (salt varchar(100) )

	User.java修改：
	public class User {
		private int id;
		private String name;
		private String salt;
		private String password;
		...
	}

### DAO ###
增加两个方法 createUser，getUser
createUser 用于注册，并且在注册的时候，将用户提交的密码加密
getUser 用于取出用户信息，其中不仅仅包括加密后的密码，还包括盐；

	/**
	 *  用于注册，并且在注册的时候，将用户提交的密码加密,
	 * @param name
	 * @param password
	 * @return
	 */
	public String createUser(String name, String password) {
		String sql = "insert into user values(null,?,?,?)";
		String salt = new SecureRandomNumberGenerator().nextBytes().toString(); //盐量随机
		String encodedPassword= new SimpleHash("md5",password,salt,2).toString();
		try (Connection c = getConnection(); PreparedStatement ps = c.prepareStatement(sql);) {
			ps.setString(1, name);
			ps.setString(2, encodedPassword);
			ps.setString(3, salt);
			ps.execute();
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return null;
	}

	/**
	 *  用于取出用户信息，其中不仅仅包括加密后的密码，还包括盐
	 * @param userName
	 * @return
	 */
	public User getUser(String userName) {
		User user = null;
		String sql = "select * from user where name = ?";
		try (Connection c = getConnection(); PreparedStatement ps = c.prepareStatement(sql)) {
			ps.setString(1, userName);
			ResultSet rs = ps.executeQuery();
			if (rs.next()) {
				user = new User();
				user.setId(rs.getInt("id"));
				user.setName(rs.getString("name"));
				user.setPassword(rs.getString("password"));
				user.setSalt(rs.getString("salt"));
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
		return user;
	}

### DatabaseRealm做法一 ###
修改 DatabaseRealm，把用户通过 UsernamePasswordToken 传进来的密码，以及数据库里取出来的 salt 进行加密，加密之后再与数据库里的密文进行比较，判断用户是否能够通过验证。

	package com.flamingo;
 
	import java.util.Set;
	 
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.AuthenticationInfo;
	import org.apache.shiro.authc.AuthenticationToken;
	import org.apache.shiro.authc.SimpleAuthenticationInfo;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.authz.AuthorizationInfo;
	import org.apache.shiro.authz.SimpleAuthorizationInfo;
	import org.apache.shiro.crypto.hash.SimpleHash;
	import org.apache.shiro.realm.AuthorizingRealm;
	import org.apache.shiro.subject.PrincipalCollection;
	 
	public class DatabaseRealm extends AuthorizingRealm {
	
	    /**
	     * 授权
	     * @param principalCollection
	     * @return
	     */
	    @Override
	    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
	         
	        //能进入到这里，表示账号已经通过验证了
	        String userName =(String) principalCollection.getPrimaryPrincipal();
	        //通过DAO获取角色和权限
	        Set<String> permissions = new DAO().listPermissions(userName);
	        Set<String> roles = new DAO().listRoles(userName);
	         
	        //授权对象
	        SimpleAuthorizationInfo s = new SimpleAuthorizationInfo();
	        //把通过DAO获取到的角色和权限放进去
	        s.setStringPermissions(permissions);
	        s.setRoles(roles);
	        return s;
	    }
	
	    /**
	     * 验证
	     * 把用户通过 UsernamePasswordToken|AuthenticationToken传进来的密码，
	     * 以及数据库里取出来的 salt 进行加密，加密之后再与数据库里的密文进行比较，
	     * 判断用户是否能够通过验证。
	     * @param token
	     * @return
	     * @throws AuthenticationException
	     */
	    @Override
	    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
	        System.out.println(this.getCredentialsMatcher());
	        //获取账号密码
	        UsernamePasswordToken t = (UsernamePasswordToken) token;
	        String userName = token.getPrincipal().toString();
	        String password = new String(t.getPassword());
	        //获取数据库中的密码和盐
	        User user = new DAO().getUser(userName);
	        String passwordInDB = user.getPassword();
	        String salt = user.getSalt();
	
	        /** 方法一 */
	        //输入的账号密码和从数据库中拿到的盐用md5加密，对比加密后的字符串与数据库中的密码字符串
	        String passwordEncoded = new SimpleHash("md5",password,salt,2).toString();
	        if(null==user || !passwordEncoded.equals(passwordInDB))
	            throw new AuthenticationException();
	        //认证信息里存放账号密码, getName() 是当前Realm的继承方法,通常返回当前类名 :databaseRealm
	        SimpleAuthenticationInfo a = new SimpleAuthenticationInfo(userName,password,getName());
	
	       return a;
	    }
	 
	}

### DatabaseRealm做法二 ###
#### DatabaseRealm修改 ####
 另一个做法是使用Shiro提供的 HashedCredentialsMatcher 帮我们做。 
在创建 SimpleAuthenticationInfo 的时候，把数据库中取出来的密文以及盐作为参数传递进去

	/**
     * 验证
     * 把用户通过 UsernamePasswordToken|AuthenticationToken传进来的密码，
     * 以及数据库里取出来的 salt 进行加密，加密之后再与数据库里的密文进行比较，
     * 判断用户是否能够通过验证。
     * @param token
     * @return
     * @throws AuthenticationException
     */
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        System.out.println(this.getCredentialsMatcher());
        //获取账号密码
        // UsernamePasswordToken t = (UsernamePasswordToken) token;
        String userName = token.getPrincipal().toString();
        // String password = new String(t.getPassword());
        //获取数据库中的密码和盐
        User user = new DAO().getUser(userName);
        String passwordInDB = user.getPassword();
        String salt = user.getSalt();

        // /** 方法一 */
        // //输入的账号密码和从数据库中拿到的盐用md5加密，对比加密后的字符串与数据库中的密码字符串
        // String passwordEncoded = new SimpleHash("md5",password,salt,2).toString();
        // if(null==user || !passwordEncoded.equals(passwordInDB))
        //     throw new AuthenticationException();
        // //认证信息里存放账号密码, getName() 是当前Realm的继承方法,通常返回当前类名 :databaseRealm
        // SimpleAuthenticationInfo a = new SimpleAuthenticationInfo(userName,password,getName());

        /** 方法二 */
        //认证信息里存放账号密码, getName() 是当前Realm的继承方法,通常返回当前类名 :databaseRealm
        //创建 SimpleAuthenticationInfo 的时候，把数据库中取出来的密文以及盐作为参数传递进去
        //通过shiro.ini里配置的 HashedCredentialsMatcher 进行自动校验
        SimpleAuthenticationInfo a = new SimpleAuthenticationInfo(userName,passwordInDB,ByteSource.Util.bytes(salt),getName());
        return a;
    }

#### shiro.ini ####
另一个做法的DatabaseRealm 中只是提供了密文和盐，那么具体算法怎么指定呢？ 那么就修改shiro.ini：为DatabaseRealm 指定credentialsMatcher，其中就指定了算法是 md5, 次数为2， storedCredentialsHexEncoded 这个表示计算之后以密文为16进制。

这样Shiro就拿着在subject.log() 时传入的UsernamePasswordToken 中的源密码， 数据库里的密文和盐巴，以及配置文件里指定的算法参数，自己去进行相关匹配了

	[main]
	credentialsMatcher=org.apache.shiro.authc.credential.HashedCredentialsMatcher
	credentialsMatcher.hashAlgorithmName=md5
	credentialsMatcher.hashIterations=2
	credentialsMatcher.storedCredentialsHexEncoded=true
	
	databaseRealm=com.flamingo.DatabaseRealm
	databaseRealm.credentialsMatcher=$credentialsMatcher
	securityManager.realms=$databaseRealm
### 测试 ###
	package com.flamingo;

	
	import org.apache.shiro.SecurityUtils;
	import org.apache.shiro.authc.AuthenticationException;
	import org.apache.shiro.authc.UsernamePasswordToken;
	import org.apache.shiro.config.IniSecurityManagerFactory;
	import org.apache.shiro.mgt.SecurityManager;
	import org.apache.shiro.subject.Subject;
	import org.apache.shiro.util.Factory;
	
	public class TestShiro {
	    public static void main(String[] args) {
	    	//这里要释放注释，先注册一个用户
	   		// new DAO().createUser("tom", "123");
	        User user = new User();
	        user.setName("tom");
	        user.setPassword("123");
	        if(login(user)) 
	        	System.out.println("login success");
	        else
	        	System.out.println("login error");
	    }
	
		private static Subject getSubject(User user) {
			//加载配置文件，并获取工厂
			Factory<SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");
			//获取安全管理者实例
			SecurityManager sm = factory.getInstance();
			//将安全管理者放入全局对象
			SecurityUtils.setSecurityManager(sm);
			//全局对象通过安全管理者生成Subject对象
			Subject subject = SecurityUtils.getSubject();
	
			return subject;
		}
	
		private static boolean login(User user) {
			Subject subject= getSubject(user);
			//如果已经登录过了，退出
			if(subject.isAuthenticated())
				subject.logout();
			//封装用户的数据
			UsernamePasswordToken token = new UsernamePasswordToken(user.getName(), user.getPassword());
			try {
				//将用户的数据token 最终传递到Realm中进行对比
				subject.login(token);
			} catch (AuthenticationException e) {
				//验证错误
				return false;
			}
			return subject.isAuthenticated();
		}
	}

