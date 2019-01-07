# 中级-zuul

[TOC]



### 1.概念

Zuul网关除了自动转发外,还有更多的场景应用:

- 鉴权 shiro/Oauth2.0...
- 流量转发
- 请求统计



### 2.Zuul核心

zuul的核心是Filter,共包括4个阶段

1. PRE 前置生效(请求被路由调用之前拦截)
2. ROUTING 路由到微服务时生效
3. POST 路由到微服务以后生效
4. ERROR 其他阶段发生错误时生效



##### 默认Filter列表

| 类型  | 顺序 | 过滤器                  | 功能                       |
| ----- | ---- | ----------------------- | -------------------------- |
| pre   | -3   | ServletDetectionFilter  | 标记处理Servlet的类型      |
| pre   | -2   | Servlet30WrapperFilter  | 包装HttpServletRequest请求 |
| pre   | -1   | FormBodyWrapperFilter   | 包装请求体                 |
| route | 1    | DebugFilter             | 标记调试标志               |
| route | 5    | PreDecorationFilter     | 处理请求上下文供后续使用   |
| route | 10   | RibbonRoutingFilter     | serviceId请求转发          |
| route | 100  | SimpleHostRoutingFilter | url请求转发                |
| route | 500  | SendForwardFilter       | forward请求转发            |
| post  | 0    | SendErrorFilter         | 处理有错误的请求响应       |
| post  | 1000 | SendResponseFilter      | 处理正常的请求响应         |



##### 禁用指定Filter方式

```properties
zuul.FormBodyWrapperFilter.pre.disable=true
```



### 3.路由过滤Filter

1. 继承 ZuulFilter过滤器
2. 重写4个方法
3. 生成Bean,注入Spring



##### 方法列表

```shell
filterType() #过滤器类型
filterOrder() #过滤器执行顺序
shouldFilter() #是否执行过滤器
run() #过滤器自定义逻辑类
```



##### RequestContext

```shell
RequestContext context= RequestContext.getCurrentContext(); #获取容器上下文对象
#获取对象
HttpServletRequest request=context.getRequest() #获取HttpServletRequest请求对象
HttpServletResponse response=context.getResponse(); #获取HttpServletResponse响应对象

#快捷设置(获取同理)
context.setSendZuulResponse(true) #是否对该请求进行路由
context.setResponseStatusCode(200) #设置响应Code
context.setResponseBody("{'name':'小黑'}") #响应内容
context.set("isSuccess",true) #设置值,isSuccess为是否让下一个Filter看到本Filter状态
```



### 4.路由熔断Hystrix

1. 实现并重写FallbackProvider接口
2. fallbackResponse方法中重写ClientHttpResponse接口



##### 方法列表

```
String getRoute() #要处理的路由
ClientHttpResponse fallbackResponse(String route, Throwable cause) #配置Hystrix
```



##### ClientHttpResponse

```
HttpStatus getStatusCode() #响应状态
int getRawStatusCode() #响应状态码
String getStatusText() #响应状态
close()  #关闭资源
InputStream getBody() #常用 return new ByteArrayInputStream(("服务连接失败").getBytes());
```



### 5.路由重试Retry

因为网络等各种原因,导致服务暂时不可用,Zuul可以帮助我们对服务进行请求重试

1. 引入jar包
2. 配置重试策略



##### 引入jar包

```xml
<dependency>
      <groupId>org.springframework.retry</groupId>
      <artifactId>spring-retry</artifactId>
</dependency>
```



##### 配置重试策略

```properties
#zuul路由重试配置
zuul.retryable=true #是否开启重试
ribbon.MaxAutoRetries=2 #最大重试次数
ribbon.MaxAutoRetriesNextServer=0 #切换相同Server的次数
```

