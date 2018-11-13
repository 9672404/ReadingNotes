# Spring

## 第1章  Spring概述 

Spring是一个IOC（DI）容器和AOP面向切面编程的框架。

IOC容器是工厂模式。

## 第2章  IOC容器和Bean的配置

### 2.1 IOC 和 DI

IOC 描述的是一种思想，而DI 是对IOC思想的具体实现. 



#### 2.1.1 IOC：控制反转

原来需要自己  通过new关键字创建一个对象，现在有容器创建并将资源推送给需要的组件。

#### 2.1.2 DI：依赖注入

IOC的另一种表述方式：即组件以一些预先定义好的方式(例如：setter 方法)接受来自于容器的资源注入。



### 2.2 Bean配置解析（:star:） 



```java
private String name; //这只叫做成员变量

public String setName() {  //set 后面的值才叫做属性
    this.name = name;
}

```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.0.xsd">

	<!-- 配置bean 
		  配置方式:  基于xml的方式 ,使用的全类名的方式.
		 <bean>: 受Spring管理的一个javaBean对象. 
		 	id:  <bean>的唯一标识. 在整个IOC容器中唯一不重复. 
		 	class: 指定javaBean的全类名. 目的是通过反射创建对象。 
		 		   Class cls = Class.forName("com.atguigu.spring.helloWorld.Person");
		 		   Object obj = cls.newInstance();   必须提供无参数构造器. 
		 <property>: 给对象的属性赋值
		  	name: 指定属性名  , 指定set风格的属性名. 
		  	value:指定属性值 	
	-->
	<bean id="person"  class="com.atguigu.spring.helloWorld.Person">
		<property name="name2" value="HanMeiMei"></property>
	</bean>
	
	<bean id="person1"  class="com.atguigu.spring.helloWorld.Person">
		<property name="name2" value="LiLei"></property>
	</bean>

</beans>

```



