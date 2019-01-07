# 基础-zuul

[TOC]

### 1.概念

在微服务/服务网格等组合架构中,服务间通信和服务对外通信的限制是不同的,需要一扇门来管理访问

这扇门就是zuul(网关)

zuul本质就是一个提供负载均衡/反向代理/权限认证的**API Gateway**



##### 1-1.简化了服务调用

微服务架构下后端服务的实例数是动态的,传统模式很难自动扩展

通过引入API Gateway,统一实现相关认证逻辑,简化服务间调用的复杂度



##### 1-2.数据裁剪

通常对于不同的客户端,显示要求是不一致的,比如手机端或web端,存在着低延迟网络环境和高延迟网路环境

使用API Gateway可以对通用性的响应数据进行裁剪以适应不同客户端环境的需求,优化用户体验



##### 1-3.聚合

可以将多个API调用逻辑进行聚合,减少客户端请求数,优化用户体验



##### 1-4.多渠道支持

针对不同的渠道和客户端,提供不同的API Gateway,该模式也被称为前后端分离(Backend for front-end简称**BFF**)



##### 1-5.遗留系统的微服务改造

 API Gateway的模式同样适用于传统的遗留系统逐步改造,通过引入抽象层,逐步使用新的实现替换旧的实现



### 2.快速启动

##### 2-1.引入jar包

```xml
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zuul</artifactId>
</dependency>
```



##### 2-2.启动类

`@EnableZuulProxy`



##### 2-3.properties配置

```properties
spring.application.name=gateway-zuul
server.port=8000
#city为映射标识,path和url匹配一对
zuul.routes.city.path=/c/** #网关访问uri  
zuul.routes.city.url=http://localhost:8081/getDesc2 #网关uri映射url 
```





### 3.快速启动-服务化

##### 概述:

API-Gateway结合Eureka使用,避免了通过url映射的局限性,动态的调用指定提供者的服务



##### 3-1.引入eureka的jar包

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



##### 3-2.properties配置

```properties
spring.application.name=gateway-zull
server.port=8000
zuul.routes.city.path=/d/**
zuul.routes.city.service-id=cityProvider
eureka.client.service-url.defaultZone=http://localhost:8080/eureka/
```



此处service-id是Eureka中的服务名



### 4.快速启动-自动转发机制

zuul以客户端的身份注册到eureka后,默认配置该注册中心下的所有服务



##### 配置

```
#zuul.routes.city.path=/d/**
#zuul.routes.city.service-id=cityProvider
```

注释掉硬编码地址部分



##### 访问方式

```
http://网关IP:网关PORT:Eureka服务名/**
```

