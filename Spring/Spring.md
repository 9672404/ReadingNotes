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



### 2.3 通过类型获取Bean 的方式

```java
//1.创建IOC容器对象
ApplicationContext iocContainer = new ClassPathXmlApplicationContext("helloworld.xml");
//2.根据id值获取bean实例对象
Student Student = (Student) iocContainer.getBean("student");

Student Student = cxt.getBean(Student. class);

Student student = cxt.getBean("student",Student. class);

```



### 2.4 依赖注入的方式

1. 通过bean的setXxx()方法赋值,在实体类里面一定要提供 set 方法。

```xml
<!-- DI依赖注入的方式: set方法注入 -->
<bean id="car" class="com.atguigu.spring.di.Car">
    <property name="brand" value="奥迪"></property>
    <property name="crop" value="一汽"></property>
    <property name="price" value="400000"></property>
</bean>
```

2. 构造器方式

**在写自定义构造器的时候，也把无参构造器一起写上，Spring框架需要使用无参构造器创建对象，再进行赋值操作。**如果不指定在属性配置的时候指定属性类型，会按照相同参数个数的构造器倒序匹配。

```xml
<!-- DI依赖注入的方式: 构造器的方式 
      index:指定参数的位置
   type: 指定参数的类型 
 -->
<bean id="car1" class="com.atguigu.spring.di.Car">
    <constructor-arg value="宝马" index="0"></constructor-arg>
    <constructor-arg value="450000" index="2" type="java.lang.Double"></constructor-arg>
    <constructor-arg value="华晨" index="1"></constructor-arg>
</bean>
```

3. 使用P命名空间

为了简化XML文件的配置，越来越多的XML文件采用属性而非子元素配置信息。Spring从2.5版本开始引入了一个新的p命名空间，可以通过<bean>元素属性的方式配置Bean 的属性。 底层使用set方式注入

```xml
<bean 
	id="studentSuper" 
	class="com.atguigu.helloworld.bean.Student"
	p:studentId="2002" p:stuName="Jerry2016" p:age="18" />

```

4. 引用外部Bean

只能应用IOC容器内的bean

```xml
<bean id="shop" class="com.atguigu.spring.bean.Shop" >
    <property name= "book" ref ="book"/>
</bean >
```

### 2.5 Bean 的作用域

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	
	<!-- 
		bean的作用域:
			singleton: 单例的(默认值), 在整个IOC容器中只能存在一个bean的对象. 而且在IOC
			           容器对象被创建时，就创建单例的bean的对象. 后续每次通过getBean()方法
			           获取bean对象时，返回的都是同一个对象.  

			prototype: 原型的/多例的 ,在整个IOC容器中可有多个bean的对象。 在IOC容器对象被
					   创建时， 不会创建原型的bean的对象。 而是等到没次通过getBean()方法获取
					   bean对象时，才会创建一个新的bean对象返回. 

			request:   一次请求对应一个bean对象
			session:   一次会话对应一个bean对象 
	 -->
	
	<bean id="car" class="com.atguigu.spring.scope.Car" scope="prototype">
		<property name="brand" value="奥迪"></property>
		<property name="price" value="400000"></property>
	</bean>
</beans>
```

### 2.6 Bean的生命周期

**实体类：**

```java
package com.atguigu.spring.lifecycle;

public class Car {
	
	private String brand ; 
	
	private Double price ;
	
	public Car() {
		System.out.println("===>1. 调用构造器创建bean对象 ");
	}
	/**
	 * 初始化方法
	 * 需要通过 init-method来指定初始化方法
	 */
	public void init() {
		System.out.println("===>3. 调用初始化方法");
	}
	
	/**
	 * 销毁方法： IOC容器关闭， bean对象被销毁.
	 */
	public void destroy() {
		System.out.println("===>5. 调用销毁方法");
	}
	
	
	public String getBrand() {
		return brand;
	}

	public void setBrand(String brand) {
		System.out.println("===>2. 调用set方法给对象的属性赋值");
		this.brand = brand;
	}

	public Double getPrice() {
		return price;
	}

	public void setPrice(Double price) {
		this.price = price;
	}

	@Override
	public String toString() {
		return "Car [brand=" + brand + ", price=" + price + "]";
	} 
}

```

 在配置bean时，通过init-method和destroy-method 属性为bean指定初始化和销毁方法

**配置：**

```xml
<bean id="car" class="com.atguigu.spring.lifecycle.Car" 
      init-method="init"  destroy-method="destroy">
    <property name="brand" value="宝马"></property>
    <property name="price" value="450000"></property>
</bean>
```

**测试：**

```java
public class TestLifeCycle {

	@Test
	public void testLifeCycle() {
		ConfigurableApplicationContext ctx = 
				new ClassPathXmlApplicationContext("spring-lifecycle.xml");
		Car car = ctx.getBean("car",Car.class);
		
		System.out.println("===>4. 使用bean对象" + car);
		//关闭容器
		
		ctx.close();
	}

}
```

使用 ConfigurableApplicationContext 初始化容器，才能有close方法。

运行结果：

```
===>1. 调用构造器创建bean对象 
===>2. 调用set方法给对象的属性赋值
postProcessBeforeInitialization
===>3. 调用初始化方法
postProcessAfterInitialization
===>4. 使用bean对象Car [brand=宝马, price=450000.0]
十一月 13, 2018 11:11:42 下午 org.springframework.context.support.AbstractApplicationContext doClose
信息: Closing org.springframework.context.support.ClassPathXmlApplicationContext@4cc77c2e: startup date [Tue Nov 13 23:11:42 CST 2018]; root of context hierarchy
===>5. 调用销毁方法
```

### 2.7 引入外部文件

db.properties 配置文件

```properties
jdbc.driver=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/mysql
jdbc.username=root
jdbc.password=1234
```

需要通过Spring容器进行连接池的初始化

```xml
<!-- 通过引入外部的属性文件配置c3p0连接池 -->
<!-- 引入外部的属性文件 -->
<!-- 1 
   <bean class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
    <property name="location" value="classpath:db.properties"></property>
   </bean>
  -->

<!-- 2  -->
<context:property-placeholder location="classpath:db.properties"/>

<!-- 配置c3p0连接池 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="driverClass" value="${jdbc.driver}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="user" value="${jdbc.username}"></property>
    <property name="password" value="${jdbc.password}"></property>

</bean>
```

### 2.8 Xml自动装配

1. xml 配置文件版 ，需要把自动装配的其他Bean都定义好了，用的比较少。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	
	<!-- Car -->
	<bean id="car1" class="com.atguigu.spring.autowire.Car">
		<property name="brand" value="奔驰"></property>
		<property name="price" value="500000"></property>
	</bean>
	
	<bean id="car2" class="com.atguigu.spring.autowire.Car">
		<property name="brand" value="宝马"></property>
		<property name="price" value="500000"></property>
	</bean>
	
	<!-- Address -->
	<bean id="address" class="com.atguigu.spring.autowire.Address">
		<property name="province" value="山西省"></property>
		<property name="city" value="太原市"></property>
	</bean>
	<!-- Person  : 演示自动装配 
		
		 byName: 使用bean的属性名与IOC容器中<bean>的id值进行匹配. 匹配成功则装配成功. 
		 
		 byType: 使用bean的属性的类型与IOC容器中<bean>的class进行匹配。 如果唯一匹配则装配成功
		                     如果匹配到多个兼容类型的bean。则跑出异常。
	-->
	<bean id="person" class="com.atguigu.spring.autowire.Person" autowire="byType">
		<property name="name" value="Tom"></property>	
	</bean>
</beans>
```

*  根据**类型(byType)**自动装配：将类型匹配的bean作为属性注入到另一个bean中。若IOC容器中有多个与目标bean类型一致的bean，Spring将无法判定哪个bean最合适该属性，所以不能执行自动装配

* 根据**名称(byName)**自动装配：必须将目标bean的名称和属性名设置的完全相同



### 2.9 通过注解装配Bean

常用的注解：

* @Component：标识一个受Spring IOC容器管理的组件
* @Repository ：标识一个持久化组件
* @Service：业务逻辑层组件
* @Controller：控制层组件

使用注解装配Bean

1. 开启注解扫描

```xml
<!-- 组件扫描:  扫描加了注解的类，并管理到IOC容器中 
  base-package: 基包. Spring会扫描指定包以及子包下所有的类，将带有注解的类管理到IOC容器中
 -->
<context:component-scan base-package="com.atguigu.spring.annotation">
</context:component-scan>
```

2. 在需要装配的类上面标注注解 @Controller 等

```java
/**
 * @Cotroller 注解的作用: 
 * 相当于在xml文件中: 
 * <bean id="userController" class="com.atguigu.spring.annotation.controller.UserController">
 * 
 * 注解默认的id值 就是类名首字母小写， 可以在注解中手动指定id值:@Controller(value="id值"),可以简写为:@Controller("id值")
 */
@Controller
public class UserController {
	
	@Autowired
	private UserService userService;
	
	public void  regist() {
		
		userService.handleAddUser();
	}

}
```

















