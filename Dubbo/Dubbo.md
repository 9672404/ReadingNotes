# Dubbo

## 1. 分布式基础理论

RPC：远程过程调用，是一种进程间通信的方式。

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Dubbo%E5%85%A5%E9%97%A8/RPC%E9%80%9A%E4%BF%A1%E8%BF%87%E7%A8%8B.png)

A和B先建立起socket连接，A调用B的方法，然调用时传递参数，参数（对象类型）在网络间传递需要序列化，B在接收到A的请求后先反序列化成对象，B处理业务逻辑，并得到一个返回值，先要把结果序列化，返回给A，A要把结果反序列化，A使用返回结果。

整个过程涉及到两个过程：建立socket 和 序列化 ，这两个操作影响RPC框架的性能。



RPC框架：Dubbo，gRPC，Thrift，HSF

## 2. Dubbo核心概念



dubbo提供了3个核心能力：面向接口的远程方法调用，服务自动注册和发现，智能容错和负载均衡。

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Dubbo%E5%85%A5%E9%97%A8/Dubbo%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.png)

1. **服务提供者（Provider）**：暴露服务的服务提供方，服务提供者在启动时，向注册中心注册自己提供的服务。
2. **服务消费者（Consumer）**: 调用远程服务的服务消费方，服务消费者在启动时，向注册中心订阅自己所需的服务，服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
3. **注册中心（Registry）**：注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者
4. **监控中心（Monitor）**：服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心



调用过程：

*  服务容器负责启动，加载，运行服务提供者。

* 服务提供者在启动时，向注册中心注册自己提供的服务。

* 服务消费者在启动时，向注册中心订阅自己所需的服务。

* 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。

* 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
* 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。

## 3. Dubbo的配置项

### 3.1 服务提供者和消费者配置型

服务提供者：

```properties
<!--当前应用的名字  -->
<dubbo:application name="gmall-user"></dubbo:application>
<!--指定注册中心的地址  -->
<dubbo:registry address="zookeeper://118.24.44.169:2181" />
<!--使用dubbo协议，将服务暴露在20880端口  -->
<dubbo:protocol name="dubbo" port="20880" />
<!-- 指定需要暴露的服务 -->
<dubbo:service terface="com.atguigu.gmall.service.UserService"ref="userServiceImpl" />
<!---此处暴露的接口，需要服务提供者有接口的实现->
```

服务消费者：

```properties
<!-- 应用名 -->
<dubbo:application name="gmall-order-web"></dubbo:application>
<!-- 指定注册中心地址 -->
<dubbo:registry address="zookeeper://118.24.44.169:2181" />
<!-- 生成远程服务代理，可以和本地bean一样使用demoService -->
<dubbo:reference id="userService" interface="com.atguigu.gmall.service.UserService"></dubbo:reference>

```

在 Controller中使用 使用dubbo提供的@Reference 注解引用远程服务

### 3.2 一些参数的配置的优先级

* JVM 启动 -D 参数优先，这样可以使用户在部署和启动时进行参数重写，比如在启动时需改变协议的端口。

* XML 次之，如果在 XML 中有配置，则 dubbo.properties 中的相应配置项无效。

* Properties 最后，相当于缺省值，只有 XML 没有配置时，dubbo.properties 的相应配置项才会生效，通常用于共享公共配置，比如应用名。

### 3.3 其他

* 失败自动切换，当出现失败，重试其它服务器，但重试会带来更长延迟。可通过 retries="2" 来设置重试次数(不含第一次)

* 由于网络或服务端不可靠，会导致调用出现一种不确定的中间状态（超时）。为了避免超时导致客户端资源（线程）挂起耗尽，必须设置超时时间。

## 4. 高可用

1.  注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯

### 4.1 集群下的Dubbo的负载均衡策略

1. **Random LoadBalance**   随机，按权重设置随机概率。

2. **RoundRobin LoadBalance**  轮循，按公约后的权重设置轮循比率。

3. **LeastActive LoadBalance** 最少活跃调用数，相同活跃数的随机，活跃数指调用前后计数差。

   使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大。

4. **ConsistentHash LoadBalance**   一致性 Hash，相同参数的请求总是发到同一提供者。





