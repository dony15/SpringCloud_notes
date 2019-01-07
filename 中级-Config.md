# 中级-Config

[TOC]

### 1.Eureka整合-Server

##### 引入Eureka的jar包

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



##### 开启Eureka注解

```
@EnableDiscoveryClient
```



##### 注册Eureka

```properties
eureka.client.service-url.defaultZone=http://localhost:8080/eureka/
```



### 2.Eureka整合-Client

##### 引入Eureka的jar包

```xml
<dependency>
     <groupId>org.springframework.cloud</groupId>
     <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



##### 开启Eureka注解

```
@EnableDiscoveryClient
```



##### 配置

```properties
eureka.client.service-url.defaultZone=http://localhost:8080/eureka/ #注册调用Eureka

spring.cloud.config.discovery.service-id=config-server #调用Eureka服务
spring.cloud.config.discovery.enabled=true #开启Config服务发现支持

spring.cloud.config.name=config-dev
spring.cloud.config.profile=base,base1
spring.cloud.config.label=master
```



