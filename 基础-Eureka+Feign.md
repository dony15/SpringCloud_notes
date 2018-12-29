# 基础-Eureka+Feign

[TOC]

### 1.概念

##### Eureka

SpringCloud-core的核心之一,注册中心的基础功能包,构成如下:

1. 注册中心(也可以同时作为提供者)
2. 提供者
3. 消费者

在微服务中的最大优势是能够更紧密优雅的整合SpringCloud其他组件



##### Feign

1. 声明式webservice客户端远程访问工具,实现http,webservice等访问方式的同时,包含了熔断/负载/重连和超时等功能可配置,防止故障扩散
2. Feign支持JAX-RS标准的注解,也支持可拔插式的编码器和解码器,也可以与Eureka和Ribbon组合使用以支持负载均衡。



### 2.Eureka使用

##### 2-1.注册中心

★引入jar包★

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
 </dependency>
```



★启动类★

`@EnableEurekaServer` 开启Eureka服务端



★配置★

```properties
spring.application.name=eureka #应用名,单机时注册中心可省略,但是集群搭建时必须要有
spring.client.register-with-eureka=false #是否注册到注册中心列表,否
spring.client.fetch-registry=false # 是否需要从注册中心获取列表,否
eureka.client.service-url.defaultzone=http://localhost:8080/eureka/ #指定Eureka作为注册中心的地址
```



> 总结
>
> 1. 简单的单机Eureka注册中心,配置中不牵扯到双热备高可用等配置
> 2. Eureka注册中心有些情况并不需要单独部署,如:Apollo中,Eureka与本身的serviceConfig部署在一起,自己注册自己(启动时会有找不到注册中心异常,启动后会寻找到自身的注册,不需要担心),中小型项目时,可以节省设备资源浪费



##### 2-2.提供者

★引入jar包★

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```



★启动类★

`@EnableDiscoveryClient`开启注册中心客户端(该注解是注册中心通用,如eureka/zk等)



★配置★

```properties
spring.application.name=cityProvider
server.port=8070
eureka.client.service-url.defaultZone=http://localhost:8080/eureka/
```



★代码★

```
@RestController
@RequestMapping("/city")
public class CityController {
	@RequestMapping("/get")
    public String getCityInfo(String name){
        return name+"的信息:这是一个美丽的城市";
    }
}
```



> 总结
>
> 1. Eureka提供者只需要简单的配置即可使用,不需要做特殊的措施
> 2. Eureka提供者会提供其正常的Rest接口到注册中心



##### 2-3.消费者

★引入jar包★

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!-- 客户端负载均衡 -->
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
</dependency>
```



★启动类★

```java
@EnableDiscoveryClient
//...
	@Bean //注入Bean
	@LoadBalanced //开启负载均衡
	RestTemplate restTemplate(){
        return new RestTemplate();
	}
//...
```



★调用★

```java
	@Resource
    private RestTemplate restTemplate;
    @RequestMapping("/getDesc2")
    public String getDesc2(String name){
       String body = restTemplate.getForEntity("http://cityProvider/city/get?name={name}", String.class,name).getBody();
        return body;
    }
```



★RestTemplate方法列表★

用GET/POST/PUT/DELETE四个rest请求方式陈列

```java
//GET
ResponseEntity<T> getForEntity("服务地址",返回类型.class) //返回值常用属性如下
ResponseEntity<T>:statusCode响应状态  statusCodeValue响应码 headers头信息 body消息内容

Class<T> getForObject("服务地址",返回类型.class) //直接按照类型返回消息内容,屏蔽其他无用属性

//POST
ResponseEntity<参数对象> postForEntity("服务地址",参数对象,返回类型.class) //数据封装在参数对象中传递,其他用法同GET
Class<T> postForObject("服务地址",参数对象,返回类型.class) //同getForObject

//PUT
void put("服务地址") //PUT方法没有返回值,请求同GET

//DELETE
void delete("服务地址") //DELETE方法没有返回值,请求同GET
```





> 总结
>
> 1. 使用Ribbon做客户端负载均衡(实时拉取服务列表),Feign底层调用的也是Ribbon处理负载均衡
> 2. Ribbon请求方式类似HttpClient,需要填写目标url(格式: 协议://注册中心服务名/服务请求路径?请求参数)
> 3. 请求参数也可以使用占位符替代,{1}或者直接{参数名}





### 3.Feign使用

##### 3-1.简单调用

★引入jar包★

```xml
 <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```



★启动类★

1. `@EnableDiscoveryClient`Eureka客户端
2. `@EnableFeignClients`Feign客户端



★配置★

```properties
spring.application.name=city-consumers
server.port=8081
eureka.client.register-with-eureka=false #是否注册到注册中心列表,否
eureka.client.service-url.defaultZone=http://localhost:8080/eureka/
feign.hystrix.enabled=true #开启Feign的熔断功能,需要fallback再开启即可
```



★代码★

创建接口

```java
@FeignClient(name = "cityProvider",path ="/city")
public interface CityEureka {
    @RequestMapping("/get")
    public String get(@RequestParam("name")String name);
}
```



★调用★

```java
@RestController
public class CityController {
	@Resource 
	private CityEureka cityEureka

	@RequestMapping("/getDesc")
	public String getDesc(String name){
  	  return cityEureka.get(name);
	}
}
```



##### 3-2.FeignClient属性列表介绍

```properties
name="cityprovider" #注册中心服务的application.name
path="/city" #优雅的rest请求路径,相当于RequestMapping,建议对应提供者类级请求目录,也可以省略
fallback=CityEurekaImpl.class #开启熔断功能后,coding熔断实现类,默认关闭
```



##### 3-3.Hystrix配置

Feign中封装了Hystrix模块

★接口中增加属性★

```java
@FeignClient(name = "cityProvider",path ="/city",fallback=CityEurekaImpl.class)
```



★实现接口★

```java
@Component
public class CityEurekaImpl implements CityEureka {
    @Override
    public String get(String name) {
        return name+"挺好的,别问了,我崩了";
    }
}
```



> 总结
>
> 1. 如果不需要将消费者注册到Eureka,那么关闭其注册提供即可(eureka.client.register-with-eureka=false)
> 2. 结合Feign,大大降低可Eureka消费者的复杂性,通过FeignClient可以对Feign进行局部配置
> 3. 如果需要开启熔断功能,记得在properties中feign.hystrix.enabled=true打开
> 4. Hystrix实现类需要注解@Component,否则Spring无法代理导致初始化失败

