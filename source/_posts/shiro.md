---
title: shiro
date: 2023/09/13
updated: 2023/09/13
categories:
  - coding
tags:
  - shiro
  - 编程基础
abbrlink: 15277
---
# 1、shiro基本概念

Shiro是Apache组织下的一个开源的安全框架，常常用来做：安全认证、权限管理、会话管理、加密

## 1.1、安全认证

可用于验证用户的身份凭证，如用户名和密码。通过配置 Shiro 的 Realm（领域）来自定义身份验证的逻辑，通过 Subject 对象进行身份验证，可以获取用户的身份信息和执行相关操作

## 1.2、权限管理

可以通过配置 Shiro 的 Realm 来定义用户的角色和权限信息，或者使用注解方式进行权限控制。通过 Subject 对象进行权限检查，可以判断用户是否具有某个角色或权限，并根据结果进行相应的处理

## 1.3、会话管理

shiro 提供了会话管理的功能，用于跟踪用户的登录状态和管理用户的会话信息。Shiro 的会话管理可以基于 Web 容器的 HttpSession，也可以使用自己的非 Web 环境会话管理（redis等）

## 1.4、加密

对密码进行加密和解密操作，以保护用户的密码安全。Shiro 支持常用的加密算法，如MD5、SHA、AES等。你可以根据需求选择适当的加密算法进行使用

# 2、shiro架构

![shiro](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202309131118959.jpg)

## 2.1、主体（Subject）

Subject即主体，外部应用与subject进行交互，subject记录了当前操作用户，将用户的概念理解为当前操作的主体，可能是一个通过浏览器请求的用户，也可能是一个运行的程序。Subject在shiro中是一个接口，接口中定义了很多认证授相关的方法，外部程序通过subject进行认证授，而subject是通过SecurityManager安全管理器进行认证授权


## 2.2、安全管理器（Security Manager）

SecurityManager即安全管理器，对全部的subject进行安全管理，它是shiro的核心，负责对所有的subject进行安全管理。通过SecurityManager可以完成subject的认证、授权等，实质上SecurityManager是通过Authenticator进行认证，通过Authorizer进行授权，通过SessionManager进行会话管理等。
SecurityManager是一个接口，继承了Authenticator, Authorizer, SessionManager这三个接口

## 2.3、认证器（Authenticator）

Authenticator即认证器，对用户身份进行认证，Authenticator是一个接口，shiro提供ModularRealmAuthenticator实现类，通过ModularRealmAuthenticator基本上可以满足大多数需求，也可以自定义认证器


## 2.4、授权器（Authorizer）

Authorizer即授权器，用户通过认证器认证通过，在访问功能时需要通过授权器判断用户是否有此功能的操作权限

## 2.5、领域（Realm）
Realm即领域，相当于datasource数据源，securityManager进行安全认证需要通过Realm获取用户权限数据，比如：如果用户身份数据在数据库那么realm就需要从数据库获取用户身份信息。

> 注意：不要把realm理解成只是从数据源取数据，在realm中还有认证授权校验的相关的代码。

## 2.6、会话管理器（SessionManager）

会话管理器负责管理主体的会话，跟踪用户的登录状态和管理会话数据。
在 Shiro 中，会话管理器可以基于 Web 容器的 HttpSession，也可以使用自己的非 Web 环境会话管理

# 3、认证

在shiro中,用户需要提供principlas(身份)和credentials(证明)给shiro，从而应用能验证用户身份

## 3.1、认证流程

1. 提交身份凭证：用户在应用程序的登录页面或其他身份验证入口提交身份和凭证，例如用户名和密码
2. 创建 Subject 对象：应用程序根据用户提交的身份凭证创建一个 Subject 对象，代表当前与应用程序交互的用户
3. 提交身份凭证给认证器：Subject 对象将身份凭证提交给 Shiro 的认证器（Authenticator）进行验证
4. 认证器验证身份凭证：认证器对身份凭证进行验证，通常是通过比对凭证与存储在数据源中的用户信息进行匹配。认证器可以使用一个或多个 Realm 来获取用户信息并进行验证。Realm 对获取到的用户信息与提交的身份凭证进行比对验证，判断凭证是否有效
5. 结果处理：如果身份验证成功，认证器将成功的身份信息存储在 Subject 对象中，以便后续使用

## 3.2、认证实现
### 3.2.1、自定义Realm
```java

/**
 * 自定义Realm
 */
public class CustomerRealm extends AuthorizingRealm {
    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        System.out.println("==================");
        return null;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {
        //在token中获取 用户名
        String principal = (String) token.getPrincipal();
        System.out.println(principal);

        //实际开发中应当 根据身份信息使用jdbc mybatis查询相关数据库
        //在这里只做简单的演示
        //假设username,password是从数据库获得的信息
        String username="zhangsan";
        String password="123456";
        if(username.equals(principal)){
            //参数1:返回数据库中正确的用户名
            //参数2:返回数据库中正确密码
            //参数3:提供当前realm的名字 this.getName();
            SimpleAuthenticationInfo simpleAuthenticationInfo = new SimpleAuthenticationInfo(principal,password,this.getName());
            return simpleAuthenticationInfo;
        }
        return null;
    }
}
```

### 3.2.2、用定义的Realm进行认证
```java
/**
 * 测试自定义的Realm
 */
public class TestAuthenticatorCusttomerRealm {

    public static void main(String[] args) {
        //1.创建安全管理对象 securityManager
        DefaultSecurityManager defaultSecurityManager = new DefaultSecurityManager();

        //2.给安全管理器设置realm（设置为自定义realm获取认证数据）
        defaultSecurityManager.setRealm(new CustomerRealm());
        //IniRealm realm = new IniRealm("classpath:shiro.ini");

        //3.给安装工具类中设置默认安全管理器
        SecurityUtils.setSecurityManager(defaultSecurityManager);

        //4.获取主体对象subject
        Subject subject = SecurityUtils.getSubject();

        //5.创建token令牌
        UsernamePasswordToken token = new UsernamePasswordToken("zhangsan", "123");
        try {
            subject.login(token);//用户登录
            System.out.println("登录成功~~");
        } catch (UnknownAccountException e) {
            e.printStackTrace();
            System.out.println("用户名错误!!");
        }catch (IncorrectCredentialsException e){
            e.printStackTrace();
            System.out.println("密码错误!!!");
        }
    }
}
```


# 4、授权
## 4.1、授权流程
认证成功后，Shiro会将用户的身份信息保存在Subject对象中。一旦用户通过认证，就可以进行授权操作。授权是基于用户的身份和角色进行的，用于确定用户是否有权进行特定的操作或访问特定的资源。Shiro提供了基于角色的访问控制（Role-Based Access Control）和基于权限的访问控制（Permission-Based Access Control）两种授权方式。授权的核心是Realm。Realm是用于获取安全数据的组件

1. 用户发起访问请求。
2. Shiro的Subject对象获取当前用户的身份信息，并创建相应的Principal对象。
3. Subject将Principal对象传递给SecurityManager进行授权操作。
4. SecurityManager委托Realm获取用户的角色和权限信息。
5. Realm根据Principal对象从数据源（如数据库）中获取用户的角色和权限信息。
6. Realm将获取的角色和权限信息返回给SecurityManager。
7. SecurityManager根据获取的角色和权限信息进行授权判断，确定用户是否有权访问资源。
8. 根据授权结果，SecurityManager返回授权成功或失败的结果给Subject对象。
9. Subject根据授权结果执行相应的操作，允许或拒绝用户访问受限资源。


## 4.2、授权方式
### 4.2.1、基于角色的访问控制

```java
if(subject.hasRole("admin")){
   //操作什么资源
}
```

### 4.2.2、基于资源的访问控制
```java
if(subject.isPermission("user:update:01")){ //资源实例
  //对资源01用户具有修改的权限
}
if(subject.isPermission("user:update:*")){  //资源类型
  //对 所有的资源 用户具有更新的权限
}
```

>权限字符串的规则是：资源标识符：操作：资源实例标识符，意思是对哪个资源的哪个实例具有什么操作，“:”是资源/操作/实例的分割符，权限字符串也可以使用*通配符
>用户创建权限：user:create，或user:create:*
 用户修改实例001的权限：user:update:001
 用户实例001的所有权限：user:*：001


## 4.3、授权实现

### 4.3.1、定义Reaml

```java
/**
 * 使用自定义realm
 * 实现授权操作
 */
public class CustomerMd5Realm extends AuthorizingRealm {

    //授权
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {

        String primaryPrincipal = (String)principals.getPrimaryPrincipal();
        System.out.println("身份信息: "+primaryPrincipal); //用户名

        //根据身份信息 用户名 获取当前用户的角色信息，以及权限信息
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        //假设 admin,user 是从数据库查到的 角色信息
        simpleAuthorizationInfo.addRole("admin");
        simpleAuthorizationInfo.addRole("user");
        //假设 ... 是从数据库查到的 权限信息赋值给权限对象
        simpleAuthorizationInfo.addStringPermission("user:*:01");
        simpleAuthorizationInfo.addStringPermission("prodect:*");//第三个参数为*省略

        return simpleAuthorizationInfo;
    }

    //认证
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) throws AuthenticationException {

    }
}
```

### 4.3.2、授权

```java
public class TestCustomerMd5RealmAuthenicator {

    public static void main(String[] args) {
        //授权
        if (subject.isAuthenticated()){

            //基于角色权限控制
            System.out.println(subject.hasRole("admin"));
            //基于多角色的权限控制
            System.out.println(subject.hasAllRoles(Arrays.asList("admin", "user")));//true
            System.out.println(subject.hasAllRoles(Arrays.asList("admin", "manager")));//false
            //是否具有其中一个角色
            boolean[] booleans = subject.hasRoles(Arrays.asList("admin", "user", "manager"));
            for (boolean aBoolean : booleans) {
                System.out.println(aBoolean);
            }

            System.out.println("====这是一个分隔符====");

            //基于权限字符串的访问控制  资源标识符：操作：资源类型
            //用户具有的权限 user:*:01  prodect:*
            System.out.println("权限:"+subject.isPermitted("user:update:01"));
            System.out.println("权限:"+subject.isPermitted("prodect:update:02"));

            //分别具有哪些权限
            boolean[] permitted = subject.isPermitted("user:*:01", "user:update:02");
            for (boolean b : permitted) {
                System.out.println(b);
            }

            //同时具有哪些权限
            boolean permittedAll = subject.isPermittedAll("prodect:*:01", "prodect:update:03");
            System.out.println(permittedAll);
        }
    }
}
```

# 5、会话管理（cacheManager）

![shiro会话管理](https://yancey-note-img.oss-cn-beijing.aliyuncs.com/202309141528819.png)

## 5.1、shiro会话管理实现(EhCache)

### 5.1.1、引入依赖

```xml
<!--引入shiro和ehcache-->
<dependency>
  <groupId>org.apache.shiro</groupId>
  <artifactId>shiro-ehcache</artifactId>
  <version>1.5.3</version>
</dependency>
```

### 5.1.2、开启缓存

```java
//3.创建自定义realm
@Bean
public Realm getRealm(){
    CustomerRealm customerRealm = new CustomerRealm();
    //修改凭证校验匹配器
    HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
    //设置加密算法为md5
    credentialsMatcher.setHashAlgorithmName("MD5");
    //设置散列次数
    credentialsMatcher.setHashIterations(1024);
    customerRealm.setCredentialsMatcher(credentialsMatcher);
    
    //开启缓存管理器
    customerRealm.setCachingEnabled(true);
    customerRealm.setAuthorizationCachingEnabled(true);
    customerRealm.setAuthorizationCachingEnabled(true);
    customerRealm.setCacheManager(new EhCacheManager());
    return customerRealm;
}
```

# 6、springboot整合shiro
## 6.1、引入依赖

```xml
        <!--增加Shiro的配置-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-web-starter</artifactId>
            <version>1.4.0</version>
        </dependency>
        <!--Thymeleaf模板引擎-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!--在Thymeleaf中使用shiro标签-->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>

```

## 6.2、配置文件

```properties
#开启Shiro配置，默认为true
shiro.enabled=true
#开启ShiroWeb配置，默认为true
shiro.web.enabled=true
#登录地址
shiro.loginUrl=/login
#登录成功地址
shiro.successUrl=/index
#未获授权默认跳转地址
shiro.unauthorizedUrl=/unauthorized
#允许通过URL参数实现会话跟踪，如果网站支持Cookie，可以关闭这个选项。默认为true
shiro.sessionManager.sessionIdUrlRewritingEnabled=true
#是否允许通过Cookie实现会话跟踪，默认为true
shiro.sessionManager.sessionIdCookieEnabled=true

```

## 6.3、配置类

```java
@Configuration
public class ShiroConfig {

    /**
     * 自定义Realm
     * 直接配置了两个用户
     * nihiu=123
     * admin=123
     * 角色分别对应user和admin两个角色
     * @return
     */
    @Bean
    public Realm realm(){
        TextConfigurationRealm realm = new TextConfigurationRealm();
        realm.setUserDefinitions("nihui=123,user
 admin=123,admin");
        realm.setRoleDefinitions("admin=read,write
 user=read");
        return realm;
    }

    /**
     * ShiroFilterChainDefinition
     * @return
     */
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition(){
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        chainDefinition.addPathDefinition("/login","anon");
        chainDefinition.addPathDefinition("/doLogin","anon");
        chainDefinition.addPathDefinition("logout","logout");
        chainDefinition.addPathDefinition("/**","authc");
        return chainDefinition;
    }

    /**
     * 定义一个在Thymeleaf中支持Shiro标签。
     * @return
     */
    @Bean
    public ShiroDialect shiroDialect(){
        return new ShiroDialect();
    }
}
```

## 6.4、登录接口控制类

```java
@Controller
public class UserController {
    @PostMapping("/doLogin")
    public String doLogin(String username, String password, Model model){
        //构造一个UsernamePasswordToken实例获取
        UsernamePasswordToken token = new UsernamePasswordToken(username,password);
        //获取Subject对象访问login方法当有异常的时候抛出异常
        Subject subject = SecurityUtils.getSubject();
        try{
            subject.login(token);
        }catch (AuthenticationException e){
            model.addAttribute("error","用户名密码输入错误");
            return "login";
        }
        return "redirect:/index";
    }
    @RequiresRoles("admin")
    @GetMapping("/admin")
    public String admin(){
        return "admin";
    }

    @RequiresRoles(value = {"admin","user"},logical = Logical.OR)
    @GetMapping("/user")
    public String user(){
        return "user";
    }
}
```


## 6.5、编写页面代码

在对应的目录下面建立五个页面分别表示用户登录页面、首页、用户页面、管理员页面、认证错误页面