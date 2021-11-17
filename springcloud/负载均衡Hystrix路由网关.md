# 负载均衡/Hystrix/路由网关

## 负载均衡

#### 使用ribbon技术（基于客户端）

改变我们原来的消费者

pom文件

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-ribbon </artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>
```

配置类

```java
@Configuration
public class Config {

    @Bean
    //加注解
    @LoadBalanced
    public RestTemplate getRestTemplate(){
        return new RestTemplate();
    }

    //新建我们的负载均衡算法
    @Bean
    public IRule myRule(){
        return new RandomRule();
    }
}
```

**主启动类新加@EnableEurekaClient注解**

## 服务熔断/降级

说到这，我们就来说说Hystrix技术，他的功能有：

1. **服务降级**
2. **服务熔断**
3. **服务限流**
4. **接近实时的监控**

### 服务熔断

##### 什么是服务熔断?

**熔断机制是赌赢雪崩效应的一种微服务链路保护机制**。

当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，**进而熔断该节点微服务的调用，快速返回错误的响应信息**。检测到该节点微服务调用响应正常后恢复调用链路。在SpringCloud框架里熔断机制通过Hystrix实现。Hystrix会监控微服务间调用的状况，当失败的调用到一定阀值缺省是**5秒内20次调用失败，就会启动熔断机制**。熔断机制的注解是：`@HystrixCommand`。

pom文件

```xml
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>
```

controller层

```java
@GetMapping("/findbyid/{id}")
    @HystrixCommand(fallbackMethod = "findByIdHystrix")
    public Company findById(@PathVariable int id){
        Company byId = companyService.findById(id);

        if (byId == null){
            throw new RuntimeException("id为空或者不存在");
        }
        return byId;
    }

    public Company findByIdHystrix(@PathVariable int id){
        return new Company().setName("id为空或者不存在(备用方法)");
    }
```

服务熔断解决如下问题：

- 当所依赖的对象不稳定时，能够起到快速失败的目的；
- **快速失败后，能够根据一定的算法动态试探所依赖对象是否恢复。**

**主启动类新增@EnableHystrix**

### 服务降级

#### 服务熔断和降级的区别

- **服务熔断—>服务端**：某个服务超时或异常，引起熔断~，类似于保险丝(自我熔断)
- **服务降级—>客户端**：从整体网站请求负载考虑，当某个服务熔断或者关闭之后，服务将不再被调用，此时在客户端，我们可以准备一个 FallBackFactory ，返回一个默认的值(缺省值)。会导致整体的服务下降，但是好歹能用，比直接挂掉强。
- 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
- 实现方式不太一样，服务降级具有代码侵入性(由控制器完成/或自动降级)，熔断一般称为**自我熔断**。

**熔断，降级，限流**：

限流：限制并发的请求访问量，超过阈值则拒绝；

降级：服务分优先级，牺牲非核心服务（不可用），保证核心服务稳定；从整体负荷考虑；

熔断：依赖的下游服务故障触发熔断，避免引发本系统崩溃；系统自动执行和恢复

### Dashboard 流监控

#### 监控服务

 新建springcloud-consumer-hystrix-dashboard模块 

pom

```xml
<dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>


        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
            <version>2.2.5.RELEASE</version>
        </dependency>
```

**主启动类新增@EnableHystrixDashboard注解**

#### 被监控

首先被监控的页面必须使用hystrix

新增配置类

```java

@Configuration
public class MyConfig {
    @Bean
    public ServletRegistrationBean servletRegistrationBean(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean(new HystrixMetricsStreamServlet());
        servletRegistrationBean.addUrlMappings("/actuator/hystrix.stream");
        return servletRegistrationBean;
    }
}
```

## 路由网关

**什么是zuul?**

Zull包含了对请求的**路由**(用来跳转的)和**过滤**两个最主要功能：

其中**路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础**，而过**滤器功能则负责对请求的处理过程进行干预，是实现请求校验，服务聚合等功能的基础**。Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他服务的消息，也即以后的访问微服务都是通过Zuul跳转后获得。

 **注意**：Zuul 服务最终还是会注册进 Eureka 

pom依赖

```xml
<dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
        <version>2.2.5.RELEASE</version>
    </dependency>
```

yml

```yml
server:
  port: 9527

spring:
  application:
    name: springcloud-zuul


eureka:
  client:
    service-url:
      defaultZone: http://7001.com:7001/eureka/,http://7002.com:7002/eureka/,http://7003.com:7003/eureka/
zuul:
  routes:
    mycompany.serviceId: springcloud-provider
    mycompany.path: /mycompany/**
  ignored-services: springcloud-provider

```

**主启动类上@EnableZuulProxy**