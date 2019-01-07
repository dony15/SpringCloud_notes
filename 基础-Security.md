# 基础-Security

[TOC]

### 1.概念

Springboot集成的安全框架,可以快速搭建认证授权系统



### 2.使用

##### 引入jar包

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-security</artifactId>
        </dependency>
```



##### SecurityConfig配置类

该自定义的配置类需要实现`WebSecurityConfigurerAdapter`的方法来完成配置定制



##### HttpSecurity方法列表

主要处理授权的对象

```java
#基础方法
authorizeRequest() //授权请求,http大部分功能都属于该操作
and() //连接,builder风格中两个不同功能的连接

#条件前置方法
antMatchers(filePath) //匹配器,用于匹配路径,然后对该路径做后续功能
anyRequest() //所有请求

#条件方法
permitAll() //无条件授权,对指定匹配器范围进行授权,不需要身份认证
authenticated() //需要身份认证
hasRole() //角色,对匹配器范围进行限制,指定角色允许访问
hasAnyRole() //所有角色,对匹配器进行限制,任意角色允许访问


#登录注销
formLogin() //开启默认认证页面功能
loginPage(filePath) //开启默认认证页面功能后,自定义认证页面(该filePath为表单action地址)
logout() //开启默认注销页面功能
logoutUrl(filePath) //开启默认注销页面功能后,自定义注销页面路径

#登录后功能
successHandler(new AuthenticationSuccessHandler()) //登陆后的handler处理,如session设置
successForwardUrl(filePath) //登录成功后转发地址
defaultSuccessUrl(filePath) //登录成功后跳转地址

#特殊方法
csrf().disable() //csrf功能,禁用csrf,与csrf()连用(如果不禁用,表单数据无法提交)
httpBasic() //httpBasic方式登录,不建议,http1.0时代产物,安全缺陷很大
```



##### AuthenticationManagerBuilder方法列表

主要处理认证的对象

```java
inMemoryAuthentication() //在内存中身份验证,注意有多种验证类型可选,如数据库,LDAP等等

withUser() //创建用户名
password() //创建密码
roles()    //创建一到多个角色
and()  //连接,builder风格中两个不同功能的连接
passwordEncoder(new PasswordEncoder()) //自定义密码加密方式,注意此处5.0以后需要自己实现新的加密方式,便于数据迁移
```



##### 整合代码

SecurityConfig

```java
@Component
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.authorizeRequests().antMatchers("/").permitAll()//指定路径不需要认证
                .antMatchers("/userInfo").hasRole("ADMIN") //访问userInfo需要admin角色
                .anyRequest().hasAnyRole() //其他所有请求需要任意角色
                .and().formLogin()  //默认需要认证时跳转的页面
                .loginPage("/login")  //定义需要认证时跳转的页面
                .permitAll()            //该认证页面不需要认证
                .and().logout()         //默认需要注销的页面
//                .logoutUrl("/logout") //定义需要注销的页面
                .permitAll() //该注销页面不需要认证
                .and().csrf().disable(); //暂时禁用csrf,否则表单无法提交
    }

    //该方法为内存认证,如果需要数据库存储认证,则需要重新实现一系列认证流程
    //https://segmentfault.com/a/1190000013057238  具体查看该手册,类似shiro流程
    @Override
    protected void configure(AuthenticationManagerBuilder auth) throws Exception {
		auth.inMemoryAuthentication().withUser("dony15").password("123456").roles("ADMIN")
                .and().withUser("user").password("123456").roles("USER")
        .and().passwordEncoder(new CustomPasswordEncoder());
    }
```



CustomPasswordEncoder,自定义编码和匹配方式(此处不加密)

```java
public class CustomPasswordEncoder implements PasswordEncoder {
    @Override
    public String encode(CharSequence rawPassword) { 
        System.err.println("rawPassword11111:"+rawPassword.toString());
        return rawPassword.toString(); //编码加密,此处直接toString,不加密
    }

    @Override
    public boolean matches(CharSequence rawPassword, String encodedPassword) {
        System.err.println("rawPassword:"+rawPassword);
        System.err.println("encodedPassword:"+encodedPassword);
        return encodedPassword.equals(rawPassword)?true:false;

    }
}
```



controller,自定义登录界面需要手动设置登录请求,会在SecurityConfig中识别跳转

```java
    @RequestMapping("/login")
    public String login(){
        return "/login.html";
    }
```



