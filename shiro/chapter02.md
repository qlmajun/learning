### shiro授权 ###
---

### 2、授权简介

授权，也叫访问控制，既在应用中谁能访问那些资源，授权中几个关键的对象：主体(Subject)、资源(Resources)、权限(Permission)、角色(Role)。

**主体**

主体，即访问应用的用户，在Shiro中使用Subject代表该用户。用户只有授权后才允许访问相应的资源。

**资源**

在应用中用户可以访问的任何东西，比如访问JSP页面、查看/编辑某些数据、访问某个业务方法、打印文本等等都是资源。用户只要授权后才能访问。

**权限**

安全策略中的原子授权单位，通过权限我们可以表示在应用中用户有没有操作某个资源的权力。Shiro支持粗粒度权限（如用户模块的所有权限）和细粒度权限（操作某个用户的权限，即实例级别的）。

**角色**

角色代表了操作集合，可以理解为权限的集合，一般情况下我们会赋予用户角色而不是权限，即这样用户可以拥有一组权限，赋予权限时比较方便。

*隐式角色：* 即直接通过角色来验证用户有没有操作权限。

*显示角色：* 在程序中通过权限控制谁能访问某个资源，角色聚合一组权限集合。

### 2.1 授权方式

shiro支持三种授权方式:

**编程式**

通过编写if/else授权代码块完成

``` java
Subject subject = SecurityUtils.getSubject();  
if(subject.hasRole(“admin”)) {  
    //有权限  
} else {  
    //无权限  
}   
```

**注解式**

通过在执行的java方法上放置响应的注解完成,没有权限将抛出响应的异常

```java
@RequiresRoles("admin")  
public void hello() {  
    //有权限  
}
```

**JSP/GSP标签**

在JSP/GSP页面通过相应的标签完成

```
<shiro:hasRole name="admin">  
<!— 有权限 —>  
</shiro:hasRole>   
```

### 2.2 授权

#### 2.2.1 基于角色访问的控制(隐式角色)

&nbsp;&nbsp;&nbsp;&nbsp;隐式角色这种方式的缺点就是如果很多地方进行了角色判断，但是有一天不需要了那么就需要修改相应代码把所有相关的地方进行删除；这就是粗粒度造成的问题。

* 1、在ini配置文件配置用户拥有的角色

规则即：“用户名=密码,角色1，角色2”,如果需要在应用中判断用户是否有相应角色，就需要在相应的Realm中返回角色信息，也就是说Shiro不负责维护用户-角色信息，需要应用提供，Shiro只是提供相应的接口方便验证，后续会介绍如何动态的获取用户角色。
```
[users]
warrior=123456,role1,role2
aviator=123456,role1
```

* 2、用例代码

```java
public void hashRole() {

		login("classpath:shiro-role.ini", "warrior", "123456");

		Subject subject = getSubject();

		// 判断拥有角色：role1
		Assert.assertTrue(subject.hasRole("role1"));

		// 判断拥有角色：role1 and role2
		Assert.assertTrue(subject.hasAllRoles(Arrays.asList("role1", "role2")));

		// 判断拥有角色：role1 and role2 and !role3
		boolean[] result = subject.hasRoles(Arrays.asList("role1", "role2", "role3"));

		Assert.assertEquals(true, result[0]);

		Assert.assertEquals(true, result[1]);

		Assert.assertEquals(false, result[2]);
	}
```

Shiro提供了hasRole/hasRole用于判断用户是否拥有某个角色/某些权限；但是没有提供如hashAnyRole用于判断是否有某些权限中的某一个。

```java
public void checkRole() {

  login("classpath:shiro-role.ini", "warrior", "123456");

  Subject subject = getSubject();

  // 断言拥有角色：role1
  subject.checkRole("role1");

  // 断言拥有角色：role1 and role3 失败抛出异常
  subject.checkRoles("role1", "role3");
}
```
Shiro提供的checkRole/checkRoles和hasRole/hasAllRoles不同的地方是它在判断为假的情况下会抛出UnauthorizedException异常。

#### 2.2.2 基于资源的访问控制(显示角色)

* 1、在ini配置文件配置用户拥有的角色及角色-权限关系

规则：“用户名=密码，角色1，角色2”“角色=权限1，权限2”，即首先根据用户名找到角色，然后根据角色再找到权限；即角色是权限集合

```
[users]
warrior=123456,role1,role2
aviator=123456,role1

[roles]
#对资源user拥有create、update权限
role1=user:create,user:update
#对资源user拥有create、delete权限
role2=user:create,user:delete
#对资源user拥有create权限
role3=system:user:create
```

* 2、 测试代码

Shiro提供了isPermitted和isPermittedAll用于判断用户是否拥有某个权限或所有权限，也没有提供如isPermittedAny用于判断拥有某一个权限的接口。

```java
public void isPermitted() {

		login("classpath:shiro-permission.ini", "warrior", "123456");

		Subject subject = getSubject();

		// 判断拥有权限：user:create
		Assert.assertTrue(subject.isPermitted("user:create"));

		// 判断拥有权限：user:update and user:delete
		Assert.assertTrue(subject.isPermittedAll("user:update", "user:delete"));

		// 判断没有权限：user:view
		Assert.assertFalse(subject.isPermitted("user:view"));
	}
```

checkPermission:失败的情况下会抛出UnauthorizedException异常。

```java
public void checkPermission() {

  login("classpath:shiro-permission.ini", "warrior", "123456");

  Subject subject = getSubject();

  // 断言拥有权限：user:create
  subject.checkPermission("user:create");

  // 断言拥有权限：user:delete and user:update
  subject.checkPermissions("user:delete", "user:update");

  // 断言拥有权限：user:view 失败抛出异常
  subject.checkPermission("user:view");
}
```

### 2.3 Permission

#### 2.3.1 字符串通配符权限

规则："资源字符:操作:对象实例Id"即对哪个对象哪个实例可以进行什么操作,默认支持通配符权限字符串，":"表示资源/操作/实例的分割，","表示操作的分割;"\*"表示任意资源/操作/实例。

**1、单个资源单个权限**

用户拥有资源“system:user”的“update”权限。
```java
subject().checkPermissions("system:user:update");  
```

**2、当个资源多个权限**

init 配置文件

```
role41=system:user:update,system:user:delete
```
判断代码

```java
subject().checkPermissions("system:user:update", "system:user:delete");  
```

用户拥有资源“system:user”的“update”和“delete”权限。如上可以简写成：

```
role42="system:user:update,delete"
```

判断代码

```java
subject().checkPermissions("system:user:update,delete");  
```

*注意：* 通过“system:user:update,delete”验证"system:user:update, system:user:delete"是没问题的，但是反过来是规则不成立。


**3、单个资源全部权限**

用户拥有资源“system:user”的“create”、“update”、“delete”和“view”所有权限

init 配置

```
role51="system:user:create,update,delete,view"
```

判断代码

```java
subject().checkPermissions("system:user:create,delete,update:view");  
```

如上形式可以简写成:

```
role52=system:user:*  //推荐写法
role53=system:user   //第二种方式
```

判断代码

```java
subject().checkPermissions("system:user:*");  
subject().checkPermissions("system:user");
```

*注意:* 通过“system:user:\*”验证“system:user:create,delete,update:view”可以，但是反过来是不成立的。

**4、所有资源的全部权限**

init 配置

```
role61=*:view
```

判断代码

```java
subject().checkPermissions("user:view");
```

**5、实例级别的权限**

* **5.1单个实例单个权限**

   对资源user的1实例拥有view权限

	 ```
role71=user:view:1
	 ```

	 判断代码

	 ```java
subject().checkPermissions("user:view:1");  
	 ```

* **5.2单个实例多个权限**

	对资源user的1实例拥有update、delete权限

	```
role72="user:update,delete:1"
	```

	判断代码

	```java
subject().checkPermissions("user:delete,update:1");  
subject().checkPermissions("user:update:1", "user:delete:1");
	```

* **5.3单个实例全部权限**

	对资源user的1实例拥有所有权限

	```
role73=user:*:1
	```

	判断代码

	```java
subject().checkPermissions("user:update:1", "user:delete:1", "user:view:1");
	```

* **5.4所有实例单个权限**

  对资源user的所有实例拥有auth权限

	```
role74=user:auth:*
	```

	判断代码

	```java
subject().checkPermissions("user:view:1", "user:auth:2");  
	```

**6、Shiro对权限字符串缺失部分的处理**

&nbsp;&nbsp;&nbsp;&nbsp; 如“user:view”等价于“user:view:\*”；而“organization”等价于“organization:\*”或者“organization:*:*”。可以这么理解，这种方式实现了前缀匹配。

&nbsp;&nbsp;&nbsp;&nbsp; 另外如“user:\*”可以匹配如“user:delete”、“user:delete”可以匹配如“user:delete:1”、“user:\*:1”可以匹配如“user:view:1”、“user”可以匹配“user:view”或“user:view:1”等。即*可以匹配所有，不加*可以进行前缀匹配；但是如“\*:view”不能匹配“system:user:view”，需要使用“\*:\*:view”，即后缀匹配必须指定前缀（多个冒号就需要多个*来匹配）。


**7、WildcardPermission**

如下两种方式是等价的，因此没什么必要的话使用字符串更方便:

```java
subject().checkPermission("menu:view:1");  
subject().checkPermission(new WildcardPermission("menu:view:1"));
```

**8、性能问题**

&nbsp;&nbsp;&nbsp;&nbsp; 通配符匹配方式比字符串相等匹配来说是更复杂的，因此需要花费更长时间，但是一般系统的权限不会太多，且可以配合缓存来提供其性能，如果这样性能还达不到要求我们可以实现位操作算法实现性能更好的权限匹配。另外实例级别的权限验证如果数据量太大也不建议使用，可能造成查询权限及匹配变慢。

### 2.4 授权流程

![授权流程图] (./image/541e4da3-d1a5-3d13-83a6-b65c3596ee4e.png)

**流程：**

* 1、首先调用Subject.isPermitted*/hasRole*接口，其会委托给SecurityManager，而SecurityManager接着会委托给Authorizer;

* 2、Authorizer是真正的授权者，如果我们调用如isPermitted(“user:view”)，其首先会通过PermissionResolver把字符串转换成相应的Permission实例;

* 3、在进行授权之前，其会调用相应的Realm获取Subject相应的角色/权限用于匹配传入的角色/权限;

* 4、Authorizer会判断Realm的角色/权限是否和传入的匹配，如果有多个Realm，会委托给ModularRealmAuthorizer进行循环判断，如果匹配如isPermitted*/hasRole*会返回true，否则返回false表示授权失败。


**ModularRealmAuthorizer进行多Realm匹配流程：**

1、首先检查相应的Realm是否实现了实现了Authorizer；

2、如果实现了Authorizer，那么接着调用其相应的isPermitted*/hasRole*接口进行匹配；

3、如果有一个Realm匹配那么将返回true，否则返回false

**如果Realm进行授权的话，应该继承AuthorizingRealm，其流程是：**

1.1、如果调用hasRole*，则直接获取AuthorizationInfo.getRoles()与传入的角色比较即可；

1.2、首先如果调用如isPermitted(“user:view”)，首先通过PermissionResolver将权限字符串转换成相应的Permission实例，默认使用WildcardPermissionResolver，即转换为通配符的WildcardPermission；

2、通过AuthorizationInfo.getObjectPermissions()得到Permission实例集合；通过AuthorizationInfo. getStringPermissions()得到字符串集合并通过PermissionResolver解析为Permission实例；然后获取用户的角色，并通过RolePermissionResolver解析角色对应的权限集合（默认没有实现，可以自己提供）;

3、接着调用Permission. implies(Permission p)逐个与传入的权限比较，如果有匹配的则返回true，否则false。

### 2.5 Authorizer、PermissionResolver及RolePermissionResolver

Authorizer的职责是授权(访问控制),是Shiro API的核心入口点，提供了相应的角色/权限判断接口，SecurityManager继承了Authorizer接口，且提供了ModularRealmAuthorizer用于多Realm时的授权匹配，PermissionResolver用于解析字符串到Permission实例，而RolePermissionResolver用于根据角色解析相应的权限集合。

#### 示例

**1、ini文件配置**

```
[main]  
#自定义authorizer  
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer  
#自定义permissionResolver  
#permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver  
permissionResolver=com.shiro.example.chapter2.permission.BitAndWildPermissionResolver
authorizer.permissionResolver=$permissionResolver  
#自定义rolePermissionResolver  
rolePermissionResolver=com.shiro.example.chapter2.permission.MyRolePermissionResolver  
authorizer.rolePermissionResolver=$rolePermissionResolver  

securityManager.authorizer=$authorizer

#自定义realm 一定要放在securityManager.authorizer赋值之后（因为调用setRealms会将realms设置给authorizer，并给各个Realm设置permissionResolver和rolePermissionResolver）
realm=com.shiro.example.chapter2.realm.MyRealm
securityManager.realms=$realm
```

*注：* 设置SecurityManager的realms一定要放在最后，因为在调用SecurityManager.因为调用setRealms时会将reamls设置给authorizer，并为各个Reaml设置permissionResolver和rolePermissionResolver。

**2、自定义BitAndWildPermissionResolver及BitPermission**

BitPermission用于实现位移方式的权限，规则如：

* 权限格式：+资源字符串+权限位+实例ID，以+开头中间通过+分割；权限;

* 权限：0 表示所有权限；1 新增（二进制：0001）、2 修改（二进制：0010）、4 删除（二进制：0100）、8 查看（二进制：1000）;

BitPermission java代码

```java
/***
 * 规则 +资源字符串+权限位+实例ID
 *
 * 以+开头 中间通过+分割
 *
 * 权限： 0 表示所有权限 1 新增 0001 2 修改 0010 4 删除 0100 8 查看 1000
 *
 * 如 +user+10 表示对资源user拥有修改/查看权限
 *
 * 不考虑一些异常情况
 *
 * @author warrior
 *
 */
public class BitPermission implements org.apache.shiro.authz.Permission {

	/** 资源字符串 **/
	private String resourceIdentify;

	/** 权限位 **/
	private int permissionBit;

	/** 实例ID **/
	private String instanceId;

	public BitPermission(String permissionString) {

		String[] array = permissionString.split("\\+");

		if (array.length > 1) {
			resourceIdentify = array[1];
		}

		if (StringUtils.isEmpty(resourceIdentify)) {
			resourceIdentify = "*";
		}

		if (array.length > 2) {
			permissionBit = Integer.valueOf(array[2]);
		}

		if (array.length > 3) {
			instanceId = array[3];
		}

		if (StringUtils.isEmpty(instanceId)) {
			instanceId = "*";
		}

	}

	@Override
	public boolean implies(Permission p) {

		if (!(p instanceof BitPermission)) {
			return false;
		}

		BitPermission other = (BitPermission) p;

		// 资源字符串不是通配符或者没有匹配
		if (!("*".equals(this.resourceIdentify) || this.resourceIdentify.equals(other.resourceIdentify))) {
			return false;
		}

		// 没有匹配权限值
		if (!(this.permissionBit == 0 || (this.permissionBit & other.permissionBit) != 0)) {
			return false;
		}

		// 没有匹配的实例Id
		if (!("*".equals(instanceId) || this.instanceId.equals(other.instanceId))) {
			return false;
		}
		return true;
	}

	@Override
	public String toString() {
		return "BitPermission [resourceIdentify=" + resourceIdentify + ", permissionBit=" + permissionBit
				+ ", instanceId=" + instanceId + "]";
	}
}
```

BitAndWildPermissionResolver实现了PermissionResolver接口，并根据权限字符串是否以“+”开头来解析权限字符串为BitPermission或WildcardPermission。

```java
public class BitAndWildPermissionResolver implements PermissionResolver {

	@Override
	public Permission resolvePermission(String permissionString) {

		if (permissionString.startsWith("+")) {
			return new BitPermission(permissionString);
		}
		return new WildcardPermission(permissionString);
	}
}
```

**3、自定义MyRolePermissionResolver**

RolePermissionResolver用于角色字符串来解析得到权限集合。

简单实现如果用户拥有role1角色，就返回一个"menu:\*"权限：
```java
public class MyRolePermissionResolver implements RolePermissionResolver {

	@Override
	public Collection<Permission> resolvePermissionsInRole(String roleString) {
		if ("role1".equals(roleString)) {
			return Arrays.asList((Permission) new WildcardPermission("menu:*"));
		}
		return null;
	}
}
```

**4、自定义Realm**

```java
public class MyRealm extends AuthorizingRealm {

	@Override
	protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

		SimpleAuthorizationInfo authorizationInfo = new SimpleAuthorizationInfo();

		authorizationInfo.addRole("role1");

		authorizationInfo.addRole("role2");

		authorizationInfo.addObjectPermission(new BitPermission("+user1+10"));

		authorizationInfo.addObjectPermission(new WildcardPermission("user1:*"));

		authorizationInfo.addStringPermission("+user2+10");

		authorizationInfo.addStringPermission("user2:*");

		return authorizationInfo;
	}

	@Override
	protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

		String username = (String) token.getPrincipal(); // 得到用户名

		String password = new String((char[]) token.getCredentials()); // 得到密码

		// 如果用户名错误
		if (!"warrior".equals(username)) {
			throw new UnknownAccountException();
		}

		// 如果密码错误
		if (!"123456".equals(password)) {
			throw new IncorrectCredentialsException();
		}

		// 如果身份认证验证成功，返回一个AuthenticationInfo实现；
		return new SimpleAuthenticationInfo(username, password, getName());
	}
}
```

说明：此处继承的是AuthorizingRealm而不是实现Realm接口，推荐使用AuthorizingRealm。

* AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token),表示获取身份验证信息;

* AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals),表示根据用户身份获取授权信息。

**5、JdbcRealm的使用**

* 1、执行sql插入相关数据

```sql
delete from users;
delete from user_roles;
delete from roles_permissions;
insert into users(username, password, password_salt) values('zhang', '123', null);
insert into user_roles(username, role_name) values('zhang', 'role1');
insert into user_roles(username, role_name) values('zhang', 'role2');
insert into roles_permissions(role_name, permission) values('role1', '+user1+10');
insert into roles_permissions(role_name, permission) values('role1', 'user1:*');
insert into roles_permissions(role_name, permission) values('role1', '+user2+10');
insert into roles_permissions(role_name, permission) values('role1', 'user2:*');
```

* 2、使用shiro-jdbc-authorizer.ini配置文件，需要设置jdbcRealm.permissionsLookupEnabled为true来开启权限查询。

```
[main]
#自定义authorizer
authorizer=org.apache.shiro.authz.ModularRealmAuthorizer
#自定义permissionResolver
#permissionResolver=org.apache.shiro.authz.permission.WildcardPermissionResolver
permissionResolver=com.shiro.example.chapter2.permission.BitAndWildPermissionResolver
authorizer.permissionResolver=$permissionResolver
#自定义rolePermissionResolver
rolePermissionResolver=com.shiro.example.chapter2.permission.MyRolePermissionResolver
authorizer.rolePermissionResolver=$rolePermissionResolver

securityManager.authorizer=$authorizer

#自定义realm 一定要放在securityManager.authorizer赋值之后（因为调用setRealms会将realms设置给authorizer，并给各个Realm设置permissionResolver和rolePermissionResolver）
jdbcRealm=org.apache.shiro.realm.jdbc.JdbcRealm
dataSource=com.alibaba.druid.pool.DruidDataSource
dataSource.driverClassName=com.mysql.jdbc.Driver
dataSource.url=jdbc:mysql://localhost:3306/shiro
dataSource.username=root
#dataSource.password=
jdbcRealm.dataSource=$dataSource
jdbcRealm.permissionsLookupEnabled=true
securityManager.realms=$jdbcRealm
```

**5、测试用例**

```java
public void isPermitted() {

	login("classpath:shiro-authorizer.ini", "warrior", "123456");

	// 判断拥有权限：user:create
	Assert.assertTrue(getSubject().isPermitted("user1:update"));

	Assert.assertTrue(getSubject().isPermitted("user2:update"));

	// 通过二进制位的方式表示权限

	// 新增权限
	Assert.assertTrue(getSubject().isPermitted("+user1+2"));

	// 查看权限
	Assert.assertTrue(getSubject().isPermitted("+user1+8"));

	// 新增及查看
	Assert.assertTrue(getSubject().isPermitted("+user1+10"));

	// 没有删除权限
	Assert.assertFalse(getSubject().isPermitted("+user1+4"));

	// 通过MyRolePermissionResolver解析得到的权限
	Assert.assertTrue(getSubject().isPermitted("menu:view"));

}
```
