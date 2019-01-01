---
title: EJBCA使用教程
date: 2018-11-05 15:01:40
tags:
---
<blockquote class="blockquote-center">优秀的人，不是不合群，而是他们合群的人里面没有你</blockquote>
# web管理界面 #
## web页面 ##
- 拷贝ejbca服务器“/opt/ca/ejbca-ce-6.3.1.1/p12/”下的superadmin.p12到windows，双击安装，密码“ejbca”,ejbca提供了两个界面。
- 管理员界面(需要证书，使用刚才安装的supeadmin证书)：`https://by.cos7:8443/ejbca/adminweb/`![](https://i.imgur.com/o3c0AcR.png)
- 用户界面：`http://by.cos7:8080/ejbca/`![](https://i.imgur.com/MPIAwpH.png)
- 用户指南：`http://by.cos7:8080/ejbca/doc/userguide.html`
## 用户注册 ##
- ejbca管理员界面，打开`RA Functions->Add End Entity`菜单，填写“Required”列打勾的项。
- 用户模板选择“EMPTY”：![](https://i.imgur.com/wKDtk0Q.png)
- 输入用户名和密码：![](https://i.imgur.com/4mEjZId.png)
- Common name，如果是服务器用证书，这里填域名：![](https://i.imgur.com/VIvMfTK.png)
- 证书信息填写，证书模板选择“ENDUSER”，CA选择“dev”,Token选择“P12 file”:![](https://i.imgur.com/e5DAZKp.png)
- 点击“Add”注册
## 下载证书 ##
- ejbca用户界面，打开“Enroll”-“Create Browser Certificate”菜单：![](https://i.imgur.com/0wNUVFl.png)
- “Key length”选择“2048 bits”;“Cetificate profile”选择“ENDUSER”,点击“Enroll”按钮下载证书：![](https://i.imgur.com/pH3qyIG.png)
## 吊销证书 ##
- ejbca管理员界面，打开“RA Functions”—“Search End Entities”菜单。“Search end entities with status”处下拉框选择“All”，点击右边的“Search”按钮查看用户信息（下图省略其他列）勾选需要吊销的用户，点击表格下方的“Revoke Selected”按钮：![](https://i.imgur.com/4serhaB.png)
## 更新证书 ##
- ejbca管理员界面，打开“RA Functions”—“Search End Entities”菜单。“Search end entities with status”处下拉框选择“All”，点击右边的“Search”按钮查看用户信息（下图省略其他列）![](https://i.imgur.com/zbAzu7x.png)
- 点击需要更新的证书用户的最右边列中的“Edit End Entity”超链接，编辑用户：![](https://i.imgur.com/8oyBfPo.png)
- 设置“Status”为“New”，点击右边的“Save”按钮。然后输入新密码，其他项保持不变，点击页面最下方的“Save”按钮保存设置。
## 根证书 ##
- ejbca用户界面，打开“Retrieve”—“Fetch CA Certificates”菜单，可以下载不同格式的根证书
## 为SSL服务器创建证书配置文件 ##
- 在`CA Functions`下，按`Certificate Profiles`
- 输入最终实体证书配置文件的名称，例如`SSLServerCertificateProfile`，然后按`add`
- 选择`SSLServerCertificateProfile`并按`Edit`
- 在`Validity`下输入365d（1年有效期)
- 在`Key usage`下，选择`Digital Signature`数字签名和`Key encipherment`密钥加密（按住Ctrl键单击以选择多个）
- 取消 `Allow Key Usage Override`
- 选中 `Extended Key Usage`
- 在`Extended Key Usage`下，选择`Server Authentication`服务器身份验证
- 在`Available bit lengths`可用位长度，“1024位”，“2048位”和“4096位”
- 在可用CA `Available CAs`下，选择您的CA`ManagementCA`（用于颁发服务器证书的CA)
- 在`Type`下，选择`End Entity`
- 按`Save`
- 创建新证书配置文件的另一种方法是使用现有配置文件作为模板：在证书配置文件列表中，单击“克隆”固定配置文件SERVER;输入最终实体证书配置文件的名称，例如`SSLServerCertificateProfile`，然后按“从模板创建” `Create from template`;按`save`
## 为SSL服务器创建最终实体配置文件 ##
- 之前应该已在“为SSL服务器创建证书配置文件”部分中为SSL服务器创建了证书配置文件
- 在`RA Functions`下，按`Edit End Entity Profiles`编辑最终实体配置文件。
- 输入最终实体配置文件的名称，例如`SSLServerEndEntityProfile`，然后按“添加”。
- 选择`SSLServerEndEntityProfile`并按`Edit`编辑最终实体配置文件。
- 在主题DN字段`Subject DN Fields`下，选择`O, Organization`，然后按`Add`。
- 在`O，Organization`中，在文本框中输入“EJBCA Edu xxx”，选中`required`复选框并取消`modifiable`可修改复选框。
- 在主题DN字段`Subject DN`下，选择`C, Country`，然后按`Add`。
- 在“C，Country”中，在文本框中输入`CN`china，选中`required`复选框并取消“modifiable”复选框。
- 在主题备用字段`Subject Alternative `下，选择`DNS Name`DNS名称，然后按`Add`
- 取消选中电子邮件域`Email Domain`
- 在默认证书配置文件`Default Certificate Profile`下，选择`SSLServerCertificateProfile`（之前创建）。
- 在可用证书配置文件`Available Certificate Profiles`下，选择`SSLServerEndEntityCertificateProfile`
- 在默认CA`Default CA`下，选择`ManagementCA`（用于颁发服务器证书的CA）。
- 在可用CA`Available CAs`下，选择`ManagementCA`（与上面相同）。
- 在默认令牌`Default Token`下，选择用户生成`User Generated`。
- 在可用标记`Available Tokens`下，选择“User Generated”，“P12”，“JKS”和“PEM”（按住Ctrl单击选择多个）。
- 按`Save`
## 申请tomcate服务器证书 ##
- 用户注册时，证书模板选择“SERVER”,Token选择“JKS file”,其他项不变：![](https://i.imgur.com/vellQeB.png)
- 下载证书时，在ejbca用户界面中，打开“Enroll”—“Create Keystore”菜单，输入用户名与密码，进入下面的页面![](https://i.imgur.com/D2pi1Uf.png)
- “Key length”选择“2048 bits”；“Certificate profile”选择“SERVER”，点击“Enroll”按钮下载证书：![](https://i.imgur.com/9XHTrKd.png)
# ejbca的web service interface #
- ejbca系统装好了，可以管理数字证书了，但是，所有的操作都在ejbca界面中执行，挺麻烦的，因此，最合理的做法是在ejbca之上构建中间层，用户访问中间层提供的服务，中间层调用ejbca的web service interface
## superadmin.jks证书 ##
- ejbca提供的web service需要证书认证，证书格式：jks。对superadmin用户执行更新操作，修改“Token”值为“JKS file”![](https://i.imgur.com/2Za654a.png)
- 保存，然后按照下载普通用户证书的步骤下载superadmin的证书
## 初始化web service连接 ##
- 将web service所需的jar包添加到工程中，这些jar包是下面两个目录下的所以的jar.
>
		/opt/ca/ejbca-ce-6.3.1.1/dist/ejbca-ws-cli/lib
		/opt/ca/ejbca-ce-6.3.1.1/dist/ejbca-ws-cli

- 初始化web service
>	  
    	public void init() {
    		if (!new File(certPath).exists()) return;
    		CryptoProviderTools.installBCProvider();
    		System.setProperty("javax.net.ssl.trustStore", "d:/superadmin.jks");
    		System.setProperty("javax.net.ssl.trustStorePassword", "123456");
    		System.setProperty("javax.net.ssl.keyStore", "d:/superadmin.jks");
    		System.setProperty("javax.net.ssl.keyStorePassword", "123456");
    		QName qname = new QName("http://ws.protocol.core.ejbca.org/", "EjbcaWSService"); 
    		try {
    			EjbcaWSService service = new EjbcaWSService(new URL("https://ca.xmyself.com:8443/ejbca/ejbcaws/ejbcaws?wsdl"), qname);
    			EjbcaWS ejbcaWS = service.getEjbcaWSPort();
    		} catch (Exception e) {
    		}
    	}

- 注意：连接地址只能是域名，因此，连接ejbca的web service的机器要配置hosts`192.168.171.144 by.cos7`
## 数字证书管理 ##
- 查看用户是否已经注册
>
        private boolean isExist(String username) throws Exception {
	    	UserMatch usermatch = new UserMatch();
	    	usermatch.setMatchwith(UserMatch.MATCH_WITH_USERNAME);
	    	usermatch.setMatchtype(UserMatch.MATCH_TYPE_EQUALS);
	    	usermatch.setMatchvalue(username);
	    	try {
	    		List<UserDataVOWS> users = ejbcaWS.findUser(usermatch);
	    		if (users != null && users.size() > 0) {
	    			return true;
	    		} else {
	    			return false;
	    		}
	    	} catch (Exception e) {
	    		throw new Exception("检查用户 " + username() + " 是否存在时出错：" + e.getMessage());
	    	}
    	}

- 用户注册与更新，用的都是editUser()方法，因此要先判断是否存在.
>
    
        public void editUser() throws Exception {
	    	UserDataVOWS userData = new UserDataVOWS();
	    	userData.setUsername("testname");//用户名
	    	userData.setPassword("123456");//密码
	    	userData.setClearPwd(false);//默认
	    	userData.setSubjectDN("CN=" + "testname"
	    			+ ",OU=" + "testou"
	    			+ ",O=" + "testo"
	    			+ ",C=cn"
	    			+ ",telephoneNumber=" + "1234567890"
	    			);//设置唯一甄别名
	     
	    	String pattern = "yyyy-MM-dd HH:mm:ssZZ"; // ISO 8601标准时间格式
	    	userData.setStartTime(DateFormatUtils.format(new Date(),pattern));//证书有效起始日期
	    	userData.setEndTime(DateFormatUtils.format(DateUtils.addDays(new Date(), 100), pattern));//结束日期
	     
	    	userData.setCaName("test");//ca名称，ejbca的名称
	    	userData.setSubjectAltName(null);
	    	userData.setEmail("test@test.com");//邮件地址
	    	userData.setStatus(UserDataVOWS.STATUS_NEW);//状态为new
	    	userData.setTokenType(UserDataVOWS.TOKEN_TYPE_P12);//设置p12格式证书
	    	userData.setEndEntityProfileName("user");//终端实体模板
	    	userData.setCertificateProfileName("user");//证书模板
	    	try {
	    		ejbcaWS.editUser(userData);
	    	} catch (Exception e) {
	    		throw new Exception(e.getMessage());
	    	}
    	}

- 代码中有几处值得注意的，终端实体模板“user”和证书模板“user”需要在ejbca管理员界面中配置，并且终端实体模板“user”中要配置开启“SubjectDN”的属性如CN、OU、O、C、telephoneNumber等，还要允许修改startTime和endTime
- 吊销证书
> 
		
		public void revoke(String username) throws ServiceException {
	    	try {
	    		ejbcaWS.revokeUser(username, RevokedCertInfo.REVOCATION_REASON_UNSPECIFIED, false);
	    	} catch (Exception e) {
	    	}
	    }	

- 创建证书
>
	    private void createCert(String username, String password, String path) throws Exception {
	    	FileOutputStream fileOutputStream = null;
	    	try {
	    		// 创建证书文件
	    		KeyStore ksenv = ejbcaWS.pkcs12Req(username, password, null, "2048", AlgorithmConstants.KEYALGORITHM_RSA);
	    		java.security.KeyStore ks = KeyStoreHelper.getKeyStore(ksenv.getKeystoreData(), "PKCS12", password);
	    		fileOutputStream = new FileOutputStream(path + File.separator + username + ".p12");
	    		ks.store(fileOutputStream, password.toCharArray());
	    		// 创建密码文件
	    		File pwdFile = new File(path + File.separator + username + ".pwd");
	    		pwdFile.createNewFile();
	    		BufferedWriter out = new BufferedWriter(new FileWriter(pwdFile));
	    		out.write(password);
	    		out.flush();
	    		out.close();
	    	} catch (Exception e) {
	    		throw new Exception("用户  " + username + " 证书创建失败：" + e.getMessage());
	    	} finally {
	    		if (fileOutputStream != null) {
	    			try {
	    				fileOutputStream.close();
	    			} catch (IOException e) {
	    			}
	    		}
	    	}
    	}		

- 证书创建在服务器上，用户调用下载证书的接口服务，返回下载地址，因此，这里需要一个下载服务器，下面介绍将nginx配置为下载服务器，文件存放的目录是/var/tmp + /download/
>

    	location ^~ /download/{
		    root   /var/tmp;
		    if ($request_filename ~* ^.*?\.(txt|doc|pdf|rar|gz|zip|docx|exe|xlsx|ppt|pptx)$){
		    	add_header Content-Disposition: 'attachment;';
    		}
    	}

# 参考链接 #
- `https://www.bbsmax.com/A/xl56xw2Yzr/`