# 基于JDK动态代理的实现

实现动态代理给计算器方法添加日志的操作

1. 计算方法接口

```java
package com.atguigu.spring.aop.proxy;
/**
 * 算数计算器
 *
 */
public interface ArithmeticCalculator {
	
	public int add(int i ,int j );
	
	public int sub(int i, int j );
	
	public int mul(int i ,int j );
	
	public int div(int i, int j );
}

```

2. 计算方法实现

```java
package com.atguigu.spring.aop.proxy;

public class ArithmeticCalculatorImpl implements ArithmeticCalculator {

	@Override
	public int add(int i, int j) {
		int result = i + j ; 
		return result ; 
	}

	@Override
	public int sub(int i, int j) {
		int result = i - j ; 
		return result ; 
	}

	@Override
	public int mul(int i, int j) {
		int result = i * j ; 
		return result ; 
	}

	@Override
	public int div(int i, int j) {
		int result = i / j ; 
		return result ;
	}
	//public int aaa() {}
	
}	

```

3. 代理对象方式实现动态代理

```java
package com.atguigu.spring.aop.proxy;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * 生成代理对象。
 * 
 * JDK的动态代理:
 * 	 1. Proxy : 是所有动态代理类的父类， 专门用于生成代理类或者是代理对象  
 * 		 	public static Class<?> getProxyClass(ClassLoader loader,
                                         Class<?>... interfaces)
                            用于生成代理类的Class对象. 
                                         
 * 			public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                         InvocationHandler h)
                            用于生成代理对象                            
                                         
 *   2. InvocationHandler :完成动态代理的整个过程.
 *   		public Object invoke(Object proxy, Method method, Object[] args)
        	throws Throwable; 
 *   		
 */
public class ArithmeticCalculatorProxy {
	//动态代理:    目标对象     如何获取代理对象      代理要做什么 
	
	//目标对象
	private  ArithmeticCalculator  target ; 
	
	
	public ArithmeticCalculatorProxy(ArithmeticCalculator target) {
		this.target = target ; 
	}
	
	
	//获取代理对象的方法
	public Object  getProxy() {
		
		//代理对象
		Object  proxy ; 
		
		/**
		 * loader:  ClassLoader对象。 类加载器对象.  帮我们加载动态生成的代理类。 
		 * 
		 * interfaces: 接口们.  提供目标对象的所有的接口.  目的是让代理对象保证与目标对象都有接口中想同的方法. 
		 * 			
		 * h:  InvocationHandler类型的对象. 
		 */
		ClassLoader loader = target.getClass().getClassLoader();
		
		Class []  interfaces = target.getClass().getInterfaces();
		
		
		proxy = Proxy.newProxyInstance(loader, interfaces, new InvocationHandler() {
			/**
			 * invoke:  代理对象调用代理方法， 会回来调用invoke方法。
			 * 
			 * proxy: 代理对象 ， 在invoke方法中一般不会使用. 
			 * 
			 * method: 正在被调用的方法对象. 
			 * 
			 * args:   正在被调用的方法的参数. 
			 */
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				
				//将方法的调用转回到目标对象上. 
				
				//获取方法的名字
				String methodName = method.getName();
				//记录日志
				System.out.println("LoggingProxy==> The method " + methodName+" begin with "+ Arrays.asList(args));
				Object result = method.invoke(target, args);  // 目标对象执行目标方法. 相当于执行ArithmeticCalculatorImpl中的+ - * /
				
				//记录日志
				System.out.println("LoggingProxy==> The method " + methodName  +" ends with :" +result   );
				return result ;
			}
		});
		
		return proxy ;
	}
	
}


/**
 * 模拟底层生成的动态代理类
 */

class $Proxy0  extends Proxy  implements ArithmeticCalculator{

	protected $Proxy0(InvocationHandler h) {
		super(h);
	}

	@Override
	public int add(int i, int j) {
	//	return  super.h.invoke(this,方法对象,方法参数);
		return 0 ;
	}

	@Override
	public int sub(int i, int j) {
		return 0;
	}

	@Override
	public int mul(int i, int j) {
		return 0;
	}

	@Override
	public int div(int i, int j) {
		return 0;
	}
	
}
```

4. 生成代理类的方式实现动态代理

```java
package com.atguigu.spring.aop.proxy;

import java.lang.reflect.Constructor;
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

/**
 * 生成代理对象。
 * 
 * JDK的动态代理:
 * 	 1. Proxy : 是所有动态代理类的父类， 专门用户生成代理类或者是代理对象  
 * 		 	public static Class<?> getProxyClass(ClassLoader loader,
                                         Class<?>... interfaces)
                            用于生成代理类的Class对象. 
                                         
 * 			public static Object newProxyInstance(ClassLoader loader,
                                          Class<?>[] interfaces,
                                         InvocationHandler h)
                            用于生成代理对象                            
                                         
 *   2. InvocationHandler :完成动态代理的整个过程.
 *   		public Object invoke(Object proxy, Method method, Object[] args)
        	throws Throwable; 
 *   		
 */
public class ArithmeticCalculatorProxy2 {
	//动态代理:    目标对象     如何获取代理对象      代理要做什么 
	
	//目标对象
	private  ArithmeticCalculator  target ; 
	
	
	public ArithmeticCalculatorProxy2(ArithmeticCalculator target) {
		this.target = target ; 
	}
	
	
	//获取代理对象的方法
	public Object  getProxy() throws Exception {
		
		//代理对象
		Object  proxy ; 
		
		ClassLoader loader = target.getClass().getClassLoader();
		Class [] interfaces = target.getClass().getInterfaces();
		
		Class proxyClass = Proxy.getProxyClass(loader, interfaces);
		
		//Class  创建对象?   newInstance()
		
		Constructor con = 
				proxyClass.getDeclaredConstructor(InvocationHandler.class);
		
		proxy = con.newInstance(new InvocationHandler() {
			
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				//将方法的调用转回到目标对象上. 
				
				//获取方法的名字
				String methodName = method.getName();
				//记录日志
				System.out.println("LoggingProxy2==> The method " + methodName+" begin with "+ Arrays.asList(args));
				Object result = method.invoke(target, args);  // 目标对象执行目标方法. 相当于执行ArithmeticCalculatorImpl中的+ - * /
				
				//记录日志
				System.out.println("LoggingProxy2==> The method " + methodName  +" ends with :" +result   );
				return result ;
			}
		});
		
		
		return proxy ;
	}
	
}
```

5. 测试方法

```java
package com.atguigu.spring.aop.proxy;

import java.util.Properties;

public class Main {
	
	public static void main(String[] args)  throws Exception{
		//将动态生成的代理类保存下来
		 Properties properties = System.getProperties();
		 properties.put("sun.misc.ProxyGenerator.saveGeneratedFiles", "true");

		
		
		//目标对象
		ArithmeticCalculator target = new ArithmeticCalculatorImpl();
		
		//获取代理对象
		Object  obj = new ArithmeticCalculatorProxy2(target).getProxy();
		
		// 转回具体的类型.
		
		ArithmeticCalculator  proxy = (ArithmeticCalculator) obj ;
		
		System.out.println(proxy.getClass().getName());
		
		//
		int result = proxy.add(1, 1);
		
		System.out.println("Main  Result : " + result );
		
		
		/**
		 * 问题:
		 * 	  1. 代理对象能否转换成目标对象的类型?
		 *    2. 代理对象调用代理方法，为什么会执行 InvocationHandler中的invoke 方法
		 */
		
	}
}

```



以上是基于接口的方式实现动态代理，还可以基于“继承”的方式实现，就是代理类继承目标类。