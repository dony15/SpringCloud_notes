# 基础-Config

[TOC]



### 1.概念

SpringCloud集成的分布式配置中心

##### 特点

1. C/S方式使用
2. 即时生效
3. 版本管理
4. 并发查询
5. 支持多种语言



### 2.快速启动Server-git

##### 仓库配置

仓库与配置文件名匹配后,即可直接通过配置文件名访问配置,格式如下:

```
{application}-{profile}.properties
```

如:

1. 创建项目:config
2. 创建路径:dev
3. ★创建配置文件:config-dev-base.properties





##### 引入jar包

```xml
<dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```



##### 启动类

`@EnableConfigServer`



##### properties配置

```properties
spring.cloud.config.server.git.uri=https://github.com/dony15/config
spring.cloud.config.server.git.search-paths=dev
spring.cloud.config.server.git.username=用户名
spring.cloud.config.server.git.password=密码
```



##### 访问格式列表

```
application:应用程序
profile:配置文件
label:git标签

★推荐方式:
[/{label}]/{application}-{profile}.properties
```



##### 测试

访问`http://localhost:7000/master/config-dev`

显示配置详情



★访问`http://localhost:7000/master/config-dev-base1.properties`

即可显示数据



### 3.快速启动Client

##### 引入jar包

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```



##### bootstrap.properties配置

```properties
spring.cloud.config.uri=http://localhost:7000
spring.cloud.config.name=config-dev #填写配置文件与路径相同的部分
spring.cloud.config.profile=base,base1 #填写配置文件与路径不同的部分(多个配置文件逗号分隔)
spring.cloud.config.label=master
```

- spring.application.name：对应{application}部分
- spring.cloud.config.profile：对应{profile}部分
- spring.cloud.config.label：对应git的分支。如果配置中心使用的是本地存储，则该参数无用
- spring.cloud.config.uri：配置中心的具体地址
- spring.cloud.config.discovery.service-id：指定配置中心的service-id，便于扩展为高可用配置集群。



##### 使用

```
@Value(value="${key}")
```



### 4.快速启动Server-native

##### properties配置

```properties
spring.profiles.active=native #开启本地策略
spring.cloud.config.server.native.search-locations=classpath:/config #指定配置文件位置
```

该方案使用本地(指定位置)配置



### 5.快速启动Client-native

##### bootstrap.properties配置

```properties
spring.cloud.config.uri=http://localhost:7000
spring.cloud.config.name=demo-service #填写properties文件名
```

