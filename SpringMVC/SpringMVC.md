# SpringMVC

## 第1章 SpringMVC概述

使用SpringMVC框架的步骤。

1. 导入相关包。
2. 在web.xml配置核心控制器

```xml
<!-- Springmvc的前端控制器 / 核心控制器:  DispatcherServlet -->
	<servlet>
		<servlet-name>springDispatcherServlet</servlet-name>
		<servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
		<!-- 给DispatcherServlet配置初始化参数：
				指定Springmvc的核心配置文件
		 -->
		<init-param>
			<param-name>contextConfigLocation</param-name>
			<param-value>classpath:springmvc.xml</param-value>
		</init-param>
		
		<!-- 
			load-on-startup: 设置DispatcherServlet在服务器启动时加载。
				Servlet的创建时机：
					 1. 请求到达以后创建
					 2. 服务器启动即创建
		 -->
		<load-on-startup>1</load-on-startup>
	</servlet>
	<!-- 指定请求的匹配 -->
	<servlet-mapping>
		<servlet-name>springDispatcherServlet</servlet-name>
		<url-pattern>/</url-pattern>
	</servlet-mapping>
```

* 在tomcat启动的时候读取web.xml文件，加载DispatcherServlet ，从类路径下读取springmvc.xml 配置，创建SpringMVC容器。
* url-pattern 为处理的请求的路径。

3. 配置SprinMVC的核心配置文件

```xml
<!--  1. 组件扫描 -->
	<context:component-scan base-package="com.atguigu.springmvc"></context:component-scan>
	<!--  2. 视图解析器
		 工作机制:  prefix + 请求处理方法的返回值 + suffix  =  物理视图路径. 
		 		 /WEB-INF/views/success.jsp 
		WEB-INF: 是服务器内部路径。 不能直接从浏览器端访问该路径下的资源. 但是可以内部转发进行访问
	-->
	<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<property name="prefix" value="/WEB-INF/views/"></property>
		<property name="suffix" value=".jsp"></property>
	</bean>
```

4. 编写请求处理器

```java
@Controller  //标注注解任务是控制器
public class SpringmvcHandler {
    
}
```

5. 编写请求处理的方法并映射请求

```java

/**
 * 请求处理器  / 控制器
 *
 */
@Controller
public class SpringMVCHandler {
	
	/**
	 * 处理客户端的请求:  http://localhost:8888/Springmvc01/hello
	 * 
	 * @RequestMapping: 完成请求 与 请求处理方法的映射. 
	 * 
	 */
	@RequestMapping(value="/hello")
	public String  handleHello() {
		System.out.println("Hello SpringMVC ");
		
		return "success";  // 通过视图解析器解析得到具体的视图， 再转发去往该视图. 
	}
}
```

return 返回的字符串，通过视图解析器 根据配置好的前缀(prefix )、后缀 (suffix)  拼装成一个 具体的静态资源路径，/WEB-INF/views/success.jsp  ，就可以转发到具体的页面。

WEB-INF为服务器内部路径，不能直接用浏览器访问，只能通过内部转发。

6. 处理流程

* 启动Tomcat服务器, 会加载DispatcherServlet， 然后就会读取springmvc.xml,进而创建好的Springmvc容器对象. 

  创建Springmvc容器对象: 组件扫描会 扫描到请求处理器， 以及请求处理中@RequestMapping注解，
  	       				     能得到具体的请求与请求处理器 中方法的映射

* 客户端发送请求: http://localhost:8888/Springmvc01/hello

* 请求到达web.xml中与<url-pattern>进行匹配， 匹配成功，就将请求交给DispatcherServlet

* DispatcherServlet根据请求 与 请求处理方法的映射， 将请求交给具体的请求处理器中的请求处理方法来进行处理

* 请求处理方法处理完请求， 最终方法会返回一个字符串

* 视图解析器根据请求处理方法返回的结果， prefix + returnValue + suffix, 解析生成具体的物理视图路径，再通过转发的方式去往视图。

7. html中的相对路径和绝对路径

```html
<!-- 
    相对路径: 不以/开头的路径 . 相对于当前路径来发送请求. 
    绝对路径: 以/开头的路径 . 直接在 http://localhost:8888 后面拼接请求. 
-->
<a href="hello">Hello SpringMVC</a>
```



## 第2章 @RequestMapping注解

### 2.1 基本用法

@RequestMapping(value="/helloworld",method={RequestMethod.GET,RequestMethod.POST})

* 标记在类上，提供初步的请求映射信息，在项目名 /类的注解名 /方法名  这样找到一个方法。

标记在方法上，就是映射到这个方法。

* 可以设置映射的请求的类型，并可以设置多个。

*  <a></a> 属于 get请求，From 发送的是 Post 请求。

### 2.2 设置请求参数和请求头信息

```java
/**
	 * @RequestMapping  映射请求参数   以及  请求头信息
	 * 
	 * params  : username=tom&age=22
	 * headers
	 */

@RequestMapping(value="/testRequestMappingParamsAndHeaders",
                params= {"username","age=22"},
                headers= {"!Accept-Language"})
public String testRequestMappingParamsAndHeaders() {

    return "success";
}
```

### 2.3 带占位符的URL-@PathVariable

```java
/**
	 * 带占位符的URL
	 * 
	 * 浏览器:  http://localhost:8888/Springmvc01/testPathVariable/admin/1001
	 */
	@RequestMapping(value="/testPathVariable/{name}/{id}")
	public String testPathVariable(@PathVariable("name")String name, @PathVariable("id")Integer id ) {
		System.out.println(name  + " : " + id);
		
		return "success";
	}
```



## 第3章 REST

>  具体说，就是 HTTP 协议里面，四个表示操作方式的动词：GET、POST、PUT、DELETE。
>
> 它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源，PUT 用来更新资源，DELETE 用来删除资源

浏览器 form 表单只支持 GET 与 POST 请求，而DELETE、PUT 等 method 并不支持，Spring3.0 添加了一个过滤器，可以将这些请求转换为标准的 http 方法，使得支持 GET、POST、PUT 与 DELETE 请求。

### 3.1 配置HiddenHttpMethodFilter

在web.xml 配置过滤器

```xml
<!-- 配置REST 过滤器  HiddenHttpMethodFilter
       将满足转换条件的请求进行转换. 
     1. 必须是post请求
     2. 必须要通过_method能获取到一个请求参数值(要转换成的请求方式)
   -->
<filter>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>HiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

过滤器必须是 Post请求才会转换，从提交过来的参数中找，有没有  _method  ,参数，根据 _method 来确定转换的类型。可以通过Form隐藏域提交一个参数。



### 3.2 使用

```html
<!-- 修改一个订单 -->
	<form action="order" method="post">
		<input type="hidden" name="_method" value="PUT"/>
		<input type="submit" value="REST PUT"/>
	</form>
	<br/>
	
	<!-- 添加一个新的订单 -->
	<form action="order" method="post">
		<input type="submit" value="REST POST"/>		
	</form>
	<br/>
	<!-- 删除id为1001的订单 -->
	<form action="order/1001" method="post">
		<!-- 隐藏域· -->
		<input type="hidden" name="_method" value="DELETE"/>
		<input type="submit" value="REST DELETE"/>
	</form>
	<br/>
	<!-- 查询id为1001的订单 -->
	<a href="order/1001">REST GET</a>
	<br/>
```

```java
	/**
	 * REST PUT
	 */
	@RequestMapping(value="/order",method=RequestMethod.PUT)
	public String testRestPUT() {
		System.out.println("REST PUT");
		return "success";
	}
	/**
	 * REST POST
	 */
	@RequestMapping(value="/order",method=RequestMethod.POST)
	public String testRestPOST() {
		System.out.println("REST POST");
		
		return "success";
	}
	
	
	/**
	 * REST DELETE
	 */
	@RequestMapping(value="/order/{id}",method=RequestMethod.DELETE)
	public String testRestDELETE(@PathVariable("id")Integer id ) {
		System.out.println("REST DELETE: " + id );
		return "success";
	}
	
	/**
	 * REST GET
	 */ 
	@RequestMapping(value="/order/{id}",method=RequestMethod.GET)
	public String testRestGET(@PathVariable("id")Integer id ) {
		System.out.println("REST GET: " + id );
		return "success";
	}
```



## 第4章 处理请求数据

### 4.1 @RequestParam

映射请求参数到请求处理方法的形参

```java
/**
	 * @RequestParam  映射请求参数到请求处理方法的形参
	 * 	 1. 如果请求参数名与形参名一致， 则可以省略@RequestParam的指定。
	 * 	 2. @RequestParam 注解标注的形参必须要赋值。 必须要能从请求对象中获取到对应的请求参数。
	 * 		可以使用required来设置为不是必须的。 
	 * 	 3. 可以使用defaultValue来指定一个默认值取代null
	 * 客户端的请求:testRequestParam?username=Tom&age=22
	 */
	@RequestMapping("/testRequestParam")
	public String testRequestParam(@RequestParam("username")String username,
							@RequestParam(value="age",required=false,defaultValue="0")int age ) {
		//web: request.getParameter()    request.getParameterMap()
		
		System.out.println(username + " , " + age);
		return "success";
	}
```



### 4.2 @RequestHeader

使用 @RequestHeader 绑定请求报头的属性值

```java
/**
  * @RequestHeader  映射请求头信息到请求处理方法的形参中
  */

@RequestMapping("testRequestHeader")
public String testRequestHeader(@RequestHeader("Accept-Language")String acceptLanguage) {
    System.out.println("acceptLanguage:" + acceptLanguage);
    return "success";
}
```

### 4.3 @CookieValue

```java
	/**
	 * @CookieValue  映射cookie信息到请求处理方法的形参中
	 */
	@RequestMapping("/testCookieValue")
	public String testCookieValue(@CookieValue("JSESSIONID")String sessionId) {
		System.out.println("sessionid:" + sessionId);
		return "success";
	}
```

### 4.4 使用POJO作为参数

```java
/**
 * Spring MVC 会按请求参数名和 POJO 属性名进行自动匹配， 自动为该对象填充属性值。
 * 支持级联属性
 *                 如：dept.deptId、dept.address.tel 等
 */
@RequestMapping("/testPOJO")
public String testPojo(User user) {
System.out.println("testPojo: " + user);
return "success";

```

```html
<!-- 支持级联的方式 -->
用户省份: <input type="text" name="address.province" />
<br/>
用户城市: <input type="text" name="address.city"/>
<br/>
```



## 第5章 处理响应数据

### 5.1 ModelAndView

控制器处理方法的返回值如果为 ModelAndView, 则其既包含视图信息，也包含模型数据信息

* 添加模型数据:

  MoelAndView addObject(String attributeName, Object attributeValue)

  ModelAndView addAllObject(Map<String, ?> modelMap)

* 设置视图:

  void setView(View view)

  void setViewName(String viewName）

```java
/**
	 * ModelAndView
	 * 结论: Springmvc会把ModelAndView中的模型数据存放到request域对象中.
	 */
	@RequestMapping("/testModelAndView")
	public  ModelAndView  testModelAndView() {
		//模型数据: username=Admin
		ModelAndView mav = new ModelAndView();
		//添加模型数据
		mav.addObject("username", "Admin");
		
		//设置视图信息
		mav.setViewName("success");
		
		return mav ;
	}
```

```html
<body>
	<h1>Success Page.</h1>
	
	username: ${requestScope.username}   <!-- 四个域对象: pageScope  requestScope sessionScope  applicationScope -->
	<br/>
	password: ${requestScope.password}
	<br/>
	loginMsg: ${requestScope.loginMsg }
</body>
```

**结论：Springmvc会把ModelAndView中的模型数据存放到request域对象中.**

### 5.2 Map

```java
	/**
	 * Map
	 * 结论: SpringMVC会把Map中的模型数据存放到request域对象中.
	 *  SpringMVC再调用完请求处理方法后，不管方法的返回值是什么类型，都会处理成一个ModelAndView对象（参考DispatcherServlet的945行）
	 */		
	@RequestMapping("/testMap")
	public String  testMap(Map<String,Object> map ) {
		//模型数据: password=123456
		System.out.println(map.getClass().getName()); //BindingAwareModelMap
		map.put("password", "123456");
		
		return "success";
	}
```



### 5.3 Model

```java
	/**
	 * Model
	 */
	@RequestMapping("/testModel")
	public String testModel(Model model) {
		//模型数据 : loginMsg=用户名或者密码错误
		
		model.addAttribute("loginMsg", "用户名或者密码错误");
		
		return "success";
	}
```



**结论：SpringMVC在调用完请求处理方法后，不管方法的返回值是什么类型，都会处理成一个ModelAndView对象（参考DispatcherServlet的945行）**



## 第6章 视图解析器

视图的作用是渲染模型数据，将模型里的数据以某种形式呈现给客户,主要就是完成转发或者是重定向的操作



* 请求处理方法执行完成后，最终返回一个 ModelAndView 对象。对于那些返回 String，View 或 ModeMap 等类型的处理方法，**Spring MVC** **也会在内部将它们装配成一个 ModelAndView** **对象**，它包含了逻辑名和模型对象的视图

*  Spring MVC 借助**视图解析器**（**ViewResolver**）得到最终的视图对象（View），最终的视图可以是 JSP ，也可能是 Excel、JFreeChart等各种表现形式的视图



**通过视图解析器 解析  ModelAndView 得到View 视图对象。**

### 6.1 重定向

一般 般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理

如果返回的字符串中带 forward: 或redirect: 前缀时，SpringMVC会对他们进行特殊处理：将 forward: 和 redirect: 当成指示符，其后的字符串作为URL 来处理

```java
@RequestMapping("/testRedirect")
public String testRedirect(){
System.out.println("testRedirect");
return "redirect:/index.jsp";
//return "forward:/index.jsp";
}
```

## 第7章 处理Json

```java
/**
	 * 处理Json
	 */
@ResponseBody   // 负责将方法的返回值 转化成json字符串 响应给浏览器端.
@RequestMapping("/testJson")
public Collection<Employee> testJson() {

    Collection<Employee> emps = employeeDao.getAll();

    //Gson  gson.toJson(emps);  out.println(jsonStr);

    return emps ;
}
```

方法返回类型直接是查询结果的类型，在方法上标注 @ResponsBody 注就会转换成Json格式。

是通过 HttpMessageConveter 实现的

HttpMessageConveter  支持： @ResponseBody   ResponseEntity

通过  ResponseEntity实现下载功能

```java
	/**
	 * 使用HttpMessageConveter完成下载功能:
	 * 
	 * 支持  @RequestBody   @ResponseBody   HttpEntity  ResponseEntity
	 * 
	 * 下载的原理:  将服务器端的文件 以流的形式  写到 客户端. 
	 * ResponseEntity: 将要下载的文件数据， 以及响应信息封装到ResponseEntity对象中，浏览器端通过解析
	 * 				       发送回去的响应数据， 就可以进行一个下载操作. 
	 */
	@RequestMapping("/download")
	public ResponseEntity<byte[]> testDownload(HttpSession session ) throws Exception{
		//将要下载的文件读取成一个字节数据
		byte [] imgs ;
		
		ServletContext sc = session.getServletContext();
		
		InputStream in = sc.getResourceAsStream("image/songlaoshi.jpg");
		
		imgs = new byte[in.available()] ; 
		
		in.read(imgs);
		
		//将响应数据  以及一些响应头信息封装到ResponseEntity中
		/*
		 * 参数:
		 * 	1. 发送给浏览器端的数据
		 *  2. 设置响应头
		 *  3. 设置响应码
		 */
		HttpHeaders  headers = new HttpHeaders();
		headers.add("Content-Disposition", "attachment;filename=songlaoshi.jpg");
		
		HttpStatus statusCode = HttpStatus.OK;  // 200
		
		ResponseEntity<byte[]>  re = new ResponseEntity<byte[]>(imgs, headers, statusCode);
	
		
		return re ; 
	}
```

## 第8章 文件上传

1）    Spring MVC 为文件上传提供了直接的支持，这种支持是通过即插即用的 **MultipartResolver** 实现的。 

2）    Spring 用 **Jakarta Commons FileUpload** 技术实现了一个 MultipartResolver 实现类：	**CommonsMultipartResolver**   

3）    Spring MVC 上下文中默认没有装配 MultipartResovler，因此默认情况下不能处理文件的上传工作，如果想使用 Spring 的文件上传功能，需现在上下文中配置 MultipartResolver



1. html:

```java
<form action="upload" method="post" enctype="multipart/form-data"> 
		上传文件:<input type="file" name="uploadFile"/>
		<br/>
		文件描述:<input type="text" name="desc"/>
		<br/>
		<input type="submit" value="上传"/>
	</form>
```

2. 上传方法  通过流的方式

```java
	/**
	 * 文件的上传
	 * 上传的原理:  将本地的文件 上传到 服务器端
	 */
	
	@RequestMapping("/upload")
	public String  testUploadFile(@RequestParam("desc")String desc ,
								  @RequestParam("uploadFile") MultipartFile uploadFile,
								   HttpSession session  ) throws Exception {
		//获取到上传文件的名字
		String uploadFileName = uploadFile.getOriginalFilename();
		//获取输入流
		InputStream in = uploadFile.getInputStream();
		//获取服务器端的uploads文件夹的真实路径。
		ServletContext sc = session.getServletContext();
		
		String realPath = sc.getRealPath("uploads");
		
		File targetFile = new File(realPath + "/" + uploadFileName);
		
		//FileOutputStream  os = new FileOutputStream(targetFile);
		
		//写文件
		int i ; 
		while((i =  in.read())!=-1) {
			os.write(i);
		}
		
		in.close();
		os.close();
		
		return "success";
	}
```

通过MultipartFile提供的方法

```java

	/**
	 * 文件的上传
	 * 上传的原理:  将本地的文件 上传到 服务器端
	 */
	
	@RequestMapping("/upload")
	public String  testUploadFile(@RequestParam("desc")String desc ,
								  @RequestParam("uploadFile") MultipartFile uploadFile,
								   HttpSession session  ) throws Exception {
		//获取到上传文件的名字
		String uploadFileName = uploadFile.getOriginalFilename();
		//获取服务器端的uploads文件夹的真实路径。
		ServletContext sc = session.getServletContext();
		String realPath = sc.getRealPath("uploads");
		File targetFile = new File(realPath + "/" + uploadFileName);	
		uploadFile.transferTo(targetFile);
		return "success";
	}
```

3. springMvc.xml 配置

```xml
 <!-- 配置文件的上传
	    该bean的id值必须是 multipartResolver , 因为springmvc底层会通过该名字到容器中找对应的bean
	  -->
	 <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
	 	<!-- 保证与上传表单所在的Jsp页面的编码一致. -->
	 	<property name="defaultEncoding" value="utf-8"></property>
	 	<property name="maxUploadSize" value="10485760"></property>
	 </bean>
```

## 第9章 拦截器

1. 自定义拦截器，实现 HandlerInterceptor 接口，并实现里面的方法。

```java
/**
 * 自定义拦截器
 */
//@Component
public class MyFirstInterceptor implements HandlerInterceptor {
	/**
	 * 1. 是在DispatcherServlet的939行   在请求处理方法之前执行
	 */
	public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {
		System.out.println("MyFirstInterceptor  preHandle");
		return true;
	}
	/**
	 * 2. 在DispatcherServlet 959行   请求处理方法之后，视图处理之前执行。
	 */
	public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler,
			ModelAndView modelAndView) throws Exception {
		System.out.println("MyFirstInterceptor postHandle");
	}
	/**
	 * 3. 
	 * 	 [1].在DispatcherServlet的 1030行   视图处理之后执行.(转发/重定向后执行)
	 * 	 [2].当某个拦截器的preHandle返回false后，也会执行当前拦截器之前拦截器的afterCompletion
	 *   [3].
	 */
	public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex)
			throws Exception {
		System.out.println("MyFirstInterceptor afterCompletion");
	}
}
```

2. 在spring.xml 配置拦截器

```xml
<!-- 配置拦截器  -->
<mvc:interceptors>
    <!--1. 拦截所有的请求 --> 
    <!-- 如果没有在拦截器上标注 @compent注解，可以配置全类名 -->
    <bean class="com.atguigu.springmvc.interceptor.MyFirstInterceptor"></bean>
    <!-- 如果配置了注解，可使用bean的引用 -->
    <!-- <ref bean="myFirstInterceptor"/> -->
</mvc:interceptors>
```

```xml

<!-- 2. 指定拦截 或者指定不拦截 -->
<mvc:interceptor>
    <mvc:mapping path="/emps"/>
    <mvc:exclude-mapping path="/emps"/>
    <bean class="com.atguigu.springmvc.interceptor.MyFirstInterceptor"></bean>
</mvc:interceptor>

```

3. 多个拦截器的执行顺序 是按照配置的顺序执行。

   多个拦截器的方法执行顺序：preHandle 正序迭代，所以按照配置顺序，先配置先执行；

   ```markdown
   					  postHandle 和 afterCompletion 倒序迭代，先配置后执行。
   ```



## 第10章 运行流程详解

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/SpringMVC/SpringMvc%E8%BF%90%E8%A1%8C%E6%B5%81%E7%A8%8B.png)



> 1）    用户向服务器发送请求，请求被SpringMVC 前端控制器 DispatcherServlet捕获；
>
> 2）    DispatcherServlet对请求URL进行解析，得到请求资源标识符（URI）:
>
> 判断请求URI对应的映射
>
> 	    不存在：
>		
> 		l  再判断是否配置了mvc:default-servlet-handler：
>		
> 		l  如果没配置，则控制台报映射查找不到，客户端展示404错误
>		
> 		l  如果有配置，则执行目标资源（一般为静态资源，如：JS,CSS,HTML）
>		
> 	    存在：
>		
> 		l  执行下面流程
>
> 3）    根据该URI，调用HandlerMapping获得该Handler配置的所有相关的对象（包括Handler对象以及Handler对象对应的拦截器），最后以HandlerExecutionChain对象的形式返回；
>
> 4）    DispatcherServlet 根据获得的Handler，选择一个合适的HandlerAdapter。
>
> 5）    如果成功获得HandlerAdapter后，此时将开始执行拦截器的preHandler(...)方法【正向】
>
> 6）    提取Request中的模型数据，填充Handler入参，开始执行Handler（Controller)方法，处理请求。在填充Handler的入参过程中，根据你的配置，Spring将帮你做一些额外的工作：
>
> ①     HttpMessageConveter： 将请求消息（如Json、xml等数据）转换成一个对象，将对象转换为指定的响应信息
>
> ②     数据转换：对请求消息进行数据转换。如String转换成Integer、Double等
>
> ③     数据根式化：对请求消息进行数据格式化。 如将字符串转换成格式化数字或格式化日期等
>
> ④     数据验证： 验证数据的有效性（长度、格式等），验证结果存储到BindingResult或Error中
>
> 7）    Handler执行完成后，向DispatcherServlet 返回一个ModelAndView对象；
>
> 8）    此时将开始执行拦截器的postHandle(...)方法【逆向】
>
> 9）    根据返回的ModelAndView（此时会判断是否存在异常：如果存在异常，则执行HandlerExceptionResolver进行异常处理）选择一个适合的ViewResolver（必须是已经注册到Spring容器中的ViewResolver)返回给DispatcherServlet，根据Model和View，来渲染视图
>
> 10）在返回给客户端时需要执行拦截器的AfterCompletion方法【逆向】
>
> 11）将渲染结果返回给客户端



## 第11章 Spring整合 SpringMVC

1）  如何启动Spring IOC容器?

* 非WEB环境：  直接在main方法或者是junit测试方法中通过new操作来创建.

* WEB 环境:        我们希望SpringIOC容器在WEB应用服务器启动时就被创建.

                               通过监听器来监听ServletContext对象的创建, 监听到ServletContext对象被创建，就创建SpringIOC容器。 并且将容器对象绑定到ServletContext中， 让所有的web组件能共享到IOC容器对象。



现在需要实现如下需求，SpringMVC 管理 Controller 对象，其他 Service、Dao等由 Spring 管理。

在项目启动的时候会初始化 SprinMVC的  DispatcherServlet 前端控制器，其实是一个Servlet，怎样同时创建Spring的IOC容器呢？？写一个监听器，监听ServletContext 容器的初始化，同时初始化SringIOC容器，监听器实现ServletContextListener 接口，在 contextInitialized 方法里初始化SpringIOC容器，并且为了使SpringMVC容器能调用到IOC容器里的Bean，要把IOC容器绑定到ServletContext中。

```java
public class MyServletContextListener implements ServletContextListener {

    public void contextDestroyed(ServletContextEvent sce)  { 
    }
    
    /**
     * 当监听到ServletContext被创建，则执行该方法
     */
    public void contextInitialized(ServletContextEvent sce)  { 
    	//1. 创建SpringIOC容器对象
    	ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    	
    	//2. 将SpringIOC容器对象绑定到ServletContext中
    	ServletContext sc = sce.getServletContext();
    	
    	sc.setAttribute("applicationContext", ctx);	
    }
}
```



SpringIOC容器初始化时会扫描配置好的Bean进行管理。

就可以在其他Servlet获取Spring管理的Bean

```java
public class HelloServlet extends HttpServlet {

    protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        //访问到SpringIOC容器中的person对象. 
        //从ServletContext对象中获取SpringIOC容器对象
        ServletContext sc = getServletContext();

        ApplicationContext ctx =  (ApplicationContext)sc.getAttribute("applicationContext");

        Person person = ctx.getBean("person",Person.class);

        person.sayHello();
    }

    protected void doPost(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
        doGet(request, response);
    }

}
```

---

实际上以上内容 Spring官方提供了更简洁的方式。

在web.xml 初始化SpringIOC容器,并且可以设置IOC容器启动的配置

```xml
<!-- 初始化SpringIOC容器的监听器 -->
	<context-param>
		<param-name>contextConfigLocation</param-name>
		<param-value>classpath:applicationContext.xml</param-value>
	</context-param>

	<listener>
		<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
	</listener>
```



还是有一个问题，就是  SpringMVC的控制器和SpringIOC容器都会都配置的包路径下的所有Bean进行扫描，这样就初始化了两次，解决方案是在配置包扫描的时候设置 扫描/排除 某个包就可以了。如下。

springmvc.xml

```xml
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu.ss" use-default-filters="false">
    <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```



applicationContext.xml 

```xml
<!-- 组件扫描 -->
<context:component-scan base-package="com.atguigu.ss">
    <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
</context:component-scan>
```

### 11.1 SpringIOC 容器和 SpringMVC IOC 容器的关系

> 1）    在 Spring MVC 配置文件中引用业务层的 Bean
>
> 2）    多个 Spring IOC 容器之间可以设置为父子关系，以实现良好的解耦。
>
> 3）    Spring MVC WEB 层容器可作为 “业务层” Spring 容器的子容器：
>
> 即 WEB 层容器可以引用业务层容器的 Bean，而业务层容器却访问不到 WEB 层容器的 Bean



### 11.2 从SpringMVC中获取SpringIOC容器

```java
@RequestMapping("/hello")
public String  hello(HttpSession session ) {
    userService.hello();

    ServletContext sc = session.getServletContext();

    //SpringIOC容器对象
    //ApplicationContext ctx = (ApplicationContext)sc.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);

    //工具类
    ApplicationContext ctx = WebApplicationContextUtils.getWebApplicationContext(sc);

    Person person = ctx.getBean("person",Person.class);

    person.sayHello();

    return "success";
}
```



重点是使用HttpSession 获取当前的 ServletContext。

Spring是把  IOC容器绑定到ServletContext中，通过getAttribute 获取，否则两个容器之间不能通信。





















