# SpringCloud入门

## 1、微服务概述

### 1.1 什么是微服务？

>微服务架构风格是将一种将单个应用程序作为一套小型服务开发的方法，每种应用程序都运行在自己的进程中，并与轻量级机制（通常是HTTP资源API）进行通信。这些业务是围绕功能构建的，可以通过全自动部署机制独立部署。这些服务的集中管理最少，可以用不同的编程语言来编写，并使用不同的数据存储技术。

### 1.2 SpringCloud和SpringBoot是什么关系

**SpringBoot专注于快速、方便的开发单个微服务个体，SpringCloud关注全局的服务治理框架。**

### 1.3  SpringCloud 和 Dubbo对比

SpringCloud抛弃了Dubbo的RPC通信，采用的是基于HTTP的REST方式。



## 2、Eureka

```java
@SpringBootApplication  //SpringBoot主启动类
@EnableEurekaServer  //EurekaServer服务器端启动类,接受其它微服务注册进来
public class EurekaServer7001_App {
	
	public static void main(String[] args) {
		SpringApplication.run(EurekaServer7001_App.class, args);
	}

}
```

@EnableEurekaServer  //EurekaServer服务器端启动类,接受其它微服务注册进来

@EnableEurekaClient 在服务提供方的 启动类上标注，表示该项目为Eureka的客户端。



```yml
eureka:
  client:                                                   #客户端注册进eureka服务列表内
    service-url: 
      defaultZone: http://localhost:7001/eureka
  instance: 								#一定对应好层级关系，否则失效
    instance-id: microservicecloud-dept8001   #修改服务显示名称
    prefer-ip-address: true     #访问路径可以显示IP地址
```



> 默认情况下，如果EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例（默认90秒）。但是当网络分区故障发生时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为微服务本身其实是健康的，此时本不应该注销这个微服务。Eureka通过“自我保护模式”来解决这个问题——当EurekaServer节点在短时间内丢失过多客户端时（可能发生了网络分区故障），那么这个节点就会进入自我保护模式。一旦进入该模式，EurekaServer就会保护服务注册表中的信息，不再删除服务注册表中的数据（也就是不会注销任何微服务）。当网络故障恢复后，该Eureka Server节点会自动退出自我保护模式。
>
> 在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。当它收到的心跳数重新恢复到阈值以上时，该Eureka Server节点就会自动退出自我保护模式。它的设计哲学就是宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例。一句话讲解：好死不如赖活着
>
> 综上，自我保护模式是一种应对网络异常的安全保护措施。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留），也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

### 2.1 作为服务注册中心，Eureka 比Zookeeper好在哪？

1. Zookeeper 保证的是CP 。ZK集群如果Lader 节点DWON机了，会重新选举，导致服务整个注册中心集群不可用。

   Eureka 保证 AP：Eureka注册是发现连接失败，则会切换到其他节点，只要有一台Eureka在，就保证服务注册可用，只不过查到的可能不是最新的。

2. Eureka的自我保护机制。

   如果超过15分钟内超过85%的节点都没有正常的心跳，Eureka就认为客户端与注册中心出现了网络故障，会出现以下几种情况。

   * Eureka不再从注册列表中移除因为长时间没有收到心跳而应该过期的服务。
   * Eureka仍然能够接受新服务的注册和查询请求，但是不会同步到其他节点上（即保证当前节点可用）
   * 当网络稳定时，当前实例新的注册信息会被同步到其他节点中。

Eureka 可以很好的应对因为网络故障导致部分节点失去联系的情况，而不会像Zookeeper那样使整个服务瘫痪。



## 3、Ribbon

> Spring Cloud Ribbon是基于Netflix Ribbon实现的一套客户端       负载均衡的工具。
>
> 简单的说，Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法，将Netflix的中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。

分为两种方式：

1. 集中式LB：即在服务的消费方和提供方之间使用独立的LB设施(可以是硬件，如F5, 也可以是软件，如nginx), 由该设施负责把访问请求通过某种策略转发至服务的提供方；
2. 进程内LB：将LB逻辑集成到消费方，消费方从服务注册中心获知有哪些地址可用，然后自己再从这些地址中选择出一个合适的服务器。Ribbon就属于进程内LB，它只是一个类库，集成于消费方进程，消费方通过它来获取到服务提供方的地址。

```java
private static final String REST_URL_PREFIX = "http://MICROSERVICECLOUD-DEPT";
//Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号
```



### 3.1修改负载均衡算法

* 只要实现 IRule 接口就能实现自己的算法。

```java
//在 API的 ConfigBean 类里面添加以下配置
@Bean
public IRule myRule() {
    //return new RandomRule(); //用重新传选择的随机算法，来替换默认的轮询算法。
    return new RetryRule();		//默认使用轮询算法，但是如果有机器down机后，进行重试，重试多次后调用不同，下次就不调用了。
}
```

### 3.2 自定义负载均衡算法

1. 在启动类添加注解

   ```java
   //在启动该微服务的时候就能去加载我们的自定义Ribbon配置类，从而是配置生效。
   @RibbonClient(name = "MICROSERVICECLOUD-DEPT", configuration = MySalfRule.class)
   ```

2. 这个配置类不能放在@ComponentScan 所扫描的当前包以及子包下，否则我们自定义的这个配置类会被所有的Ribbon客户端共享，达不到特殊化定制的目的了。

   因为 SpringBoot主启动类里面有 @SpringBootApplication 注解，包含 @ComponentScan注解，所以不能放到主目录下。


## 4. Frign

区别：Ribbon是面向微服务编程，在Controller里面引用了服务名

​	    Frign是面向接口编程，通过接口+注解的方式获得服务。

使用：只需创建一个接口 ，然后在上面添加注解。

## 5. Hystrix


Hystrix是一个用于处理分布式系统的延迟和容错的开源库，在分布式系统里，许多依赖不可避免的会调用失败，比如超时、异常等，Hystrix能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分布式系统的弹性。

“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控（类似熔断保险丝），向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至雪崩。

服务熔断：一般是某个服务故障或者异常引起，类似现实世界的保险丝，当某个异常条件被触发，直接熔断整个服务，而不是一直等到此服务超时。

服务降级：一般是从整体负荷考虑。就是当某个服务熔断之后，服务器将不再被调用，此时客户端可以自己准备一个本地的fallback回掉，返回一个缺省值。这样做虽然服务水平下降，但是比服务挂掉要好。





















