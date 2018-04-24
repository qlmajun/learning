### shiro简介及身份验证 ###
---

### 1.1 简介

&nbsp;&nbsp;&nbsp;&nbsp; Apache Shiro是java的一个安全框架。目前使用Apache Shiro的人越来越多，因为它相当简单对比与Spring Security，可能没有Spring Security做的功能强大，但是在实际工作时可能不需要那么复杂的东西，所有使用小而简单的Shiro就足够了。Shiro可以非常容易的开发出足够好的应用，其不仅可以用在JavaSE环境，也可以用于JavaEE环境。Shiro可以帮助我们完成：认证、授权、加密、会话管理、与web集成、缓存等。

&nbsp;&nbsp;&nbsp;&nbsp;Shiro API包含的基本功能如下:

* **Authentication：** 身份认证/登入，验证用户是不是拥有相应的身份;

* **Authorization：** 授权，即权限验证，验证某个已认证的用户是否拥有某个权限，常见的如：验证某个用户是否拥有某个角色。或者细粒度的验证某个用户对某个资源是否具有某个权限;

* **SessionManager：** 会话管理，即用户登入后就是一次的会话，在没退出之前，它的所有信息都会在会话中;

* **Cryptography：** 加密，保护数据的安全性，如密码加密存储到数据库，而不是明文存储;

* **Web Support：** web支持，可以非常容易的集成到web环境中;

* **Caching：** 缓存，比如用户登入后,其用户信息、拥有的角色/权限不必每次去查，这样可以提高效率;

* **Testing：** 提供测试支持;

* **Run As：** 允许一个用户假装为另一个用户（如果他们允许）的身份进行访问;

* **Remember Me：** 记住我，这个是非常常见的功能，即一次登录后，下次再来的话不用登录了;

*注意：Shiro不会去维护用户，用户的权限;这些需要我们自己去设计/提供；然后通过相应的接口注入给Shiro即可。*


&nbsp;&nbsp;&nbsp;&nbsp; 从应用程序的角度来观察如何使用Shiro,器API包含的内容：

* **subject：** 主体,代表了当前“用户”，这个用户不一定是一个具体的人，与当前应用交互的任何东西都是Subject，如网络爬虫，机器人等；即一个抽象概念；所有Subject都绑定到SecurityManager，与Subject的所有交互都会委托给SecurityManager；可以把Subject认为是一个门面；SecurityManager才是实际的执行者;

* **SecurityManager：** 安全管理器；即所有与安全有关的操作都会与SecurityManager交互；且它管理着所有Subject；可以看出它是Shiro的核心，它负责与后边介绍的其他组件进行交互，如果学习过SpringMVC，你可以把它看成DispatcherServlet前端控制器;

* **Relam：** 域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。


### 1.2 身份验证

&nbsp;&nbsp;&nbsp;&nbsp; 即在应用中谁能证明他就是他本人。一般提供如他们的身份ID一些标识信息来表明他就是他本人，如提供身份证，用户名/密码来证明。在shiro中，用户需要提供principals （身份）和credentials（证明）给shiro从而应用能验证用户的身份。

**principals：** 身份，即主体的标识属性，可以是任何东西，如用户名、邮箱等，唯一即可。一个主体可以有多个principals，但只有一个Primary principals，一般是用户名/密码/手机号。

**credentials：** 证明/凭证，即只有主体知道的安全值，如密码/数字证书等。

### 1.2.1 环境准备

添加maven依赖

```xml
<dependencies>  
    <dependency>  
        <groupId>junit</groupId>  
        <artifactId>junit</artifactId>  
        <version>4.9</version>  
    </dependency>  
    <dependency>  
        <groupId>commons-logging</groupId>  
        <artifactId>commons-logging</artifactId>  
        <version>1.1.3</version>  
    </dependency>  
    <dependency>  
        <groupId>org.apache.shiro</groupId>  
        <artifactId>shiro-core</artifactId>  
        <version>1.2.2</version>  
    </dependency>  
</dependencies>   
```

### 1.2.2 登入/退出

* 首先准备一些用户身份/凭据（shiro.ini）

```
[users]
warrior=123456
aviator=123456
```

* 测试代码

```java
public void shiorIniAuthentication() {

		// 获取 SecurityManager 工厂，此处使用 Ini 配置文件初始化 SecurityManager
		Factory<org.apache.shiro.mgt.SecurityManager> factory = new IniSecurityManagerFactory("classpath:shiro.ini");

		// 获取SecurityManager实例，并绑定给SecurityUtils
		org.apache.shiro.mgt.SecurityManager securityManager = factory.getInstance();

		SecurityUtils.setSecurityManager(securityManager);

		// 得到Subject对象及创建用户名/密码身份验证Token(即用户身份/凭证)
		Subject subject = SecurityUtils.getSubject();

		UsernamePasswordToken token = new UsernamePasswordToken("warrior", "123456");

		try {
			// 登入，即身份验证
			subject.login(token);
		} catch (AuthenticationException e) {
			// 身份验证失败
			e.printStackTrace();
		}

		// 断言用户是否登入
		Assert.assertEquals(true, subject.isAuthenticated());

		// 退出登入
		subject.logout();

	}
```

### 1.3 Shiro从从Realm获取安全数据（如用户、角色、权限）

&nbsp;&nbsp;&nbsp;&nbsp; Realm：域，Shiro从从Realm获取安全数据（如用户、角色、权限），就是说SecurityManager要验证用户身份，那么它需要从Realm获取相应的用户进行比较以确定用户身份是否合法；也需要从Realm得到用户相应的角色/权限进行验证用户是否能进行操作；可以把Realm看成DataSource，即安全数据源。如我们之前的ini配置方式将使用org.apache.shiro.realm.text.IniRealm。


### 1.3.1 单Realm配置

* 自定义Realm实现

```java
public class MyRealm1 implements Realm {

	@Override
	public AuthenticationInfo getAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

		// 得到用户名
		String username = (String) token.getPrincipal();

		// 获取密码
		String password = new String((char[]) token.getCredentials());

		if (!"warrior".equals(username)) {
			// 用户名不正确
			throw new UnknownAccountException();
		}

		if (!"123456".equals(password)) {
			// 密码不正确
			throw new IncorrectCredentialsException();
		}

		// 如果身份认证验证成功，返回一个AuthenticationInfo实现
		return new SimpleAuthenticationInfo(username, password, getName());
	}

	@Override
	public String getName() {
		return "myrealm1";
	}

	@Override
	public boolean supports(AuthenticationToken token) {
		// 仅支持UsernamePasswordToken类型的Token
		return token instanceof UsernamePasswordToken;
	}

}
```

* ini配置文件指定自定义Realm实现(shiro-realm.ini)  

```
[main]
#声明一个realm
myRealm1=com.shior.example.chapter1.realm.MyRealm1
#指定securityManager的realms实现
securityManager.realms=$myRealm1
```

### 1.3.2 多Realm配置

ini配置文件（shiro-multi-realm.ini）

```
[main]
#声明一个realm
myRealm1=com.shior.example.chapter1.realm.MyRealm1
myRealm2=com.shior.example.chapter1.realm.MyRealm2
#指定securityManager的realms实现
securityManager.realms=$myRealm1,$myRealm2
```

*说明：*

&nbsp;&nbsp;&nbsp;&nbsp; securityManager会按照realms指定的顺序进行身份认证。此处我们使用显示指定顺序的方式指定了Realm的顺序，如果删除“securityManager.realms=$myRealm1,$myRealm2”，那么securityManager会按照realm声明的顺序进行使用（即无需设置realms属性，其会自动发现），当我们显示指定realm后，其他没有定realm将被忽略，如“securityManager.realms=$myRealm1”，那么myRealm2不会被自动设置进去。

### 1.3.3 JDBC Realm使用

* 新增数据库依赖

```xml
<dependency>
		<groupId>mysql</groupId>
		<artifactId>mysql-connector-java</artifactId>
		<version>5.1.38</version>
</dependency>
<dependency>
		<groupId>com.alibaba</groupId>
		<artifactId>druid</artifactId>
		<version>1.0.26</version>
</dependency>
```

* 到数据库shiro下建三张表：users（用户名/密码）、user_roles（用户/角色）、roles_permissions（角色/权限）

```sql
drop database if exists shiro;
create database shiro;
use shiro;

create table users (
  id bigint auto_increment,
  username varchar(100),
  password varchar(100),
  password_salt varchar(100),
  constraint pk_users primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_users_username on users(username);

create table user_roles(
  id bigint auto_increment,
  username varchar(100),
  role_name varchar(100),
  constraint pk_user_roles primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_user_roles on user_roles(username, role_name);

create table roles_permissions(
  id bigint auto_increment,
  role_name varchar(100),
  permission varchar(100),
  constraint pk_roles_permissions primary key(id)
) charset=utf8 ENGINE=InnoDB;
create unique index idx_roles_permissions on roles_permissions(role_name, permission);

insert into users(username,password)values('warrior','123456');
```

* ini配置（shiro-jdbc-realm.ini）

```
[main]
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiro
dataSource.username=root
#dataSource.password=
jdbcRealm.dataSource=$dataSource
securityManager.realms=$jdbcRealm
```

### 1.4 Authentication与AuthenticationStrategy

Authenticator的职责是验证用户账号，是Shiro API中身份验证核心的入口点：

```java
public AuthenticationInfo authenticate(AuthenticationToken authenticationToken)  
            throws AuthenticationException;
```
如果验证成功，将返回AuthenticationInfo验证信息；此信息中包含了身份及凭证；如果验证失败将抛出相应的AuthenticationException实现。

SecurityManager接口继承了Authenticator,另外还有ModularRealmAuthenticator的实现，其委托给多个Realm进行验证，验证规则通过AuthenticationStrategy接口指定，默认提供的实现：

**FirstSuccessfulStrategy：** 只要有一个Realm验证成功即可，只返回第一个Realm身份验证成功的认证信息，其他的忽略;

**AtLeastOneSuccessfulStrategy：** 只要有一个Realm验证成功即可，和FirstSuccessfulStrategy不同，返回所有Realm身份验证成功的认证信息;

**AllSuccessfulStrategy：** 所有Realm验证成功才算成功，且返回所有Realm身份验证成功的认证信息，如果有一个失败就失败了。
