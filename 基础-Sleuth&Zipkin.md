# 基础-Sleuth&Zipkin

[TOC]

### 1.概念

分布式链路追踪系统

##### 组成

1. 数据收集(普通数据收集&异步数据收集)
2. 数据存储(实时数据&全量数据)
3. 数据显示(数据分析&数据挖掘)

##### 单位

1. **追踪单元(trace)**:从request请求到达被追踪系统的边界开始-->response响应到客户端结束
2. **调用记录(span)**:每个trace都会调用多个服务,为了记录调用服务,每次调用服务时都会埋入一个span,多个有序的span组成一个trace



##### Sleuth

**功能:**为服务之间提供链路追踪,通过Sleuth可以很清楚每个服务的调用和耗时

1. **耗时分析:** 通过Sleuth可以很方便的了解到每个采样请求的耗时，从而分析出哪些服务调用比较耗时;	
2. **可视化错误:**对于程序未捕捉的异常，可以通过集成Zipkin服务界面上看到;
3. **链路优化:**对于调用比较频繁的服务，可以针对这些服务实施一些优化措施。



##### ZipKin

**功能:**数据收集/存储/展示/查找



### 2.ZipKinServer快速启动

##### 导入jar包

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-server</artifactId>
        <version>2.11.12</version>
    </dependency>
    <dependency>
        <groupId>io.zipkin.java</groupId>
        <artifactId>zipkin-autoconfigure-ui</artifactId>
        <version>2.11.12</version>
    </dependency>
</dependencies>
```



##### 启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableZipkinServer
public class ZipkinApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZipkinApplication.class, args);
    }
}
```



##### 配置

```properties
eureka.client.serviceUrl.defaultZone=http://localhost:8080/eureka/
spring.application.name=zipkin-server
```



##### 界面

```java
http://localhost:8082/zipkin/
```



##### 注意

> 1. 排除Logger重复依赖(JBOSS日志不要排除)
> 2. 官方文档在SpringBoot2.0移除ZipKin&Sleuth服务端依赖,不太兼容= = .
> 3. ZipKin&Sleuth链路追踪一般注册在Eureka上,但是url是硬编码,根据项目结构调整选择



### 3.ZipKinClient快速启动

##### 导入jar包

```xml
<dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-zipkin</artifactId>
</dependency>
```



##### 配置

```properties
spring.zipkin.base-url=http://localhost:8082 #ZipKin服务端地址
spring.sleuth.sampler.probability=1.0 #采样率0-1.0
```

