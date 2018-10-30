# Java高频面试题

## 1. 自增变量

```java
public static void main(String[] args) {
		int i = 1; //直接赋值操作，把i 存到局部变量表 i = 1
		i = i++;   //因为i++涉及到计算，又因为 i++先入栈后计算,所以先把 i压入到操作数栈，执行i++自					   增操作，把结果2 存放到局部变量表，再执行赋值操作，把操作栈中的i，存放到局				      	 部变量表i的位置  所以 i =1 ,i只是曾经 = 2。
		int j = i++;//如上j = 1,i++后没有给i赋值，局部变量表中的i还是2。
		int k = i + ++i * i++; /*先执行 “=”右边的计算操作，按顺序把变量压如到栈，先是2，然后是 
   								++i,先自增 后入栈，自增后局部变量表i=3,i入栈的是3 ，i++ 先入栈 								后自增，i入栈的是3，然后执行 i++,局部变量表中的 i = 4;此时 操作								   数栈中共有 2+3*3 计算得11,并把结果压缩到操作数栈，再把操作数栈中								的值存放k的局部变量表
    							*/
		System.out.println("i=" + i);   // 4
		System.out.println("j=" + j);   // 1
		System.out.println("k=" + k);   // 11
	}

```

注：此题注意两点 运算时需要先把数据压到操作数栈中，计算的结果赋值给局部变量表,分清操作是栈中是的数据是多少。

* 赋值=，最后计算

* 右边的从左到右加载值依次压入操作数栈

* 实际先算哪个，看运算符优先级

* 自增、自减操作都是直接修改变量的值，不经过操作数栈

* 最后的赋值之前，临时结果也是存储在操作数栈中

## 2. 单例设计模式

单例模式需要满足以下几点

> 1.一个类全局只有一个实例
>
> 2.自己初始化自己
>
> 3.向整个系统提供这个实例



饿汉式：在类加载的时候就创建，不管是否需要。因为类的加载机制，只有一个类加载器去加载这个单例类（双亲委派机制），所以不存在线程安全问题。

```java
//-------------------------------类加载初始化-------------------------------
public class Singleton() {
    Singleton（）{}；//构造方法私有化
    public static final Singleton singleton = new Singleton();
    //直接公开静态方法向外提供对象，静态方法可通过类直接调用。    
}
```

懒汉式：使用的时候创建，延时创建。如果在多线程环境需要解决 线程安全和线程效率的问题

```java
//-------------------------------双重检索-------------------------------

public class Singleton() {
    private Singleton(){} //1.构造方法私有
    
    private static Singleton singleton = null;  //2.使用一个静态变量存储实例化好的对象
    
    public static getInstance() {     //3.写静态方法对外提供单例对象的获取
        
        if(singleton == null) {  //6.因为synchronized是比较重的锁，会引起线程长时间阻塞，所以先加									一层判断，如果对象不为空才进行以下操作。
                synchronized（Singleton.class）{ //5.这时需要同步操作，synchronized的原理就是如果一个线程需要进入synchronized代码块，需要持有判断对象的锁，因为代码块中操作的是 类的实例化，所以以类为标志，一个线程只有得到Singleton类的锁才能进入到实例化操作，否则被阻塞，其他线程只能等当前线程释放锁才能继续进入，如果进入以后判断类的实例不为空，则不需要初始化操作。

                    if (singleton == null) {	  //4.判断对象是否存在，不存在new一个对象，这													   时如果跳过了!=null 判断，还没有new对														象，恰巧另一个线程也跳过了判断，这时候两个												线程会分别new一个对象，一个类会出现多个实例。
                        singleton = new Singleton();
                    }
                }
            }
        }
      
}
```

```java
//-------------------------------内部静态类-------------------------------

public class Singleton() {
    private Singleton(){} //1.构造方法私有
    
    private static class Inner() { //静态类内部，静态类内部不会随外部类的加载而加载，在											调用的时候才由类加载器加载，所以延时创建而且线程安全。
        private static final Singleton SINGLETON = new Singleton();
    }
    public static getInstence() {
        Inner.SINGLETON();
    }
    
}
```

## 3. 类的初始化和实例初始化等

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Java%E9%AB%98%E9%A2%91%E9%9D%A2%E8%AF%95%E9%A2%98/Father%26Son.jpg)



考点：

### 3.1 类的初始化过程。

1. 一个类要创建实例需要先加载并初始化该类。
   * main方法所在的类要先加载并初始化。如果main方法在子类，要先加载子类并初始化。
2. 一个子类要初始化需要先初始化父类。
3. 一个类的初始化就是执行<clint>方法
   * <clint>方法由静态类变量显式赋值代码和静态代码块组成。
   * 静态类变量显式赋值和静态代码块代码从上到下依次执行。
   * <clint>(）方法只执行一次。

> 由上面得出先要加载Son类先要初始化Father类，初始化Father类需要先要执行静态类变量显式的赋值代码
>
> （j = method()  ） 调用 method()  方法，输出 (5) ,然后再执行Father类的静态代码块 输出 (1)；执行完Father的初始化再执行Son的初始化，过程一样，则输出 （10），（6）；



### 3.2 实例的初始化过程。

实例初始化就是执行<init>方法。

1. <init>()方法可能重载有多个，有几个构造器就有几个<init>方法
2. <init>()方法由非静态实例变量显示赋值代码和非静态代码块、对应构造器代码组成
3. 非静态实例变量显示赋值代码和非静态代码块代码从上到下顺序执行，而对应构造器的代码最后执行
4. 每次创建实例对象，调用对应构造器，执行的就是对应的<init>方法
5. <init>方法的首行是super()或super(实参列表)，即对应父类的<init>方法。



**Son类的实例化方法**

> Son son1 = new Son();
>
> 由上面结论得出。
>
> 1. Son的初始化的第一行肯定是  super();         	*执行父类的初始化*，如下
>
> 2. 执行Son的非静态变量的显示赋值
> 3. 非静态代码块
> 4. 然后的无参构造

**Father类的实例化方法**

> Super();		*因为Father的父类为Object对象，不能被初始化。*
>
> 执行Father的非静态变量的显示赋值            *执行的是 i = test() ，理论上调用的是Father的test方法，但实际*						     
>
> 									 *调用的是子类的test()方法，详见 3.3 第2点* 
>
> 非静态代码块
>
> 然后的无参构造



由上过程所以输出的结果为：Father（9）（3）（2），Son （9）（8）（7）

因为执行了两次 Son 初始化 ，最终结果 （5）（1）（10）（6）（9）（3）（2）（9）（8）（7）（9）（3）（2）（9）（8）（7）



### 3.3 方法重写。

①哪些方法不可以被重写

* final方法
* 静态方法				**所以类加载时候父类的method执行的是自己的方法**	

* private等子类中不可见方法

②对象的多态性

* 子类如果重写了父类的方法，通过子类对象调用的一定是子类重写过的代码
* 非静态方法默认的调用对象是this
* this对象在构造器或者说<init>方法中就是正在创建的对象



## 4. 方法的参数传递机制

```java
public class Exam4 {
	public static void main(String[] args) {
		int i = 1;
		String str = "hello";
		Integer num = 200;
		int[] arr = {1,2,3,4,5};
		MyData my = new MyData();
		
		change(i,str,num,arr,my);
		
		System.out.println("i = " + i);
		System.out.println("str = " + str);
		System.out.println("num = " + num);
		System.out.println("arr = " + Arrays.toString(arr));
		System.out.println("my.a = " + my.a);
	}
	public static void change(int j, String s, Integer n, int[] a,MyData m){
		j += 1;
		s += "world";
		n += 1;
		a[0] += 1;
		m.a += 1;
	}
}
class MyData{
	int a = 10;
}
```



考点：

1. 方法的参数传递机制。
2. String、包装类等对象的不可变性。



方法的参数传递机制

①形参是基本数据类型

	传递数据值

②实参是引用数据类型

	传递地址值
	
	特殊的类型：String、包装类等对象不可变性



![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Java%E9%AB%98%E9%A2%91%E9%9D%A2%E8%AF%95%E9%A2%98/%E6%96%B9%E6%B3%95%E7%9A%84%E5%8F%82%E6%95%B0%E4%BC%A0%E9%80%92%E6%9C%BA%E5%88%B6.jpg)



>在类中，存在两个方法，变量在虚拟机栈是按方法分布。
>
>-----------------------------------------------------------------------------------
>
>声明的两次局部变量分别存放在不同的地址。
>
>j+=1; 调用方法时，穿过来基础类型的实参为1，自增 j=2 ，改变的是 j自己的局部变量和i无关。
>
>s+= "world"; 传过来的引用类型实参为Str的地址，操作字符串拼接，会重新开辟一块空间，存放helloworld ,然后s指针指向helloworld 的地址。
>
>n += 1; 和String原理形同，传过来的是地址，n指向了num地址，自增后 n指针指向了自增后的值得地址。
>
>a[0] += 1;同样形参实参传过来的是地址，但是这次操作的是数组中的元素，地址没有变，所以主方法的局部变量可以读到改变后的元素的的值。
>
>m.a += 1 同上。

## 5. 上台阶的走法

编程题：有n步台阶，一次只能上1步或2步，共有多少种走法？

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Java%E9%AB%98%E9%A2%91%E9%9D%A2%E8%AF%95%E9%A2%98/%E9%80%92%E5%BD%92%E6%96%B9%E5%BC%8F.jpg)

```java
public class TestStep{
	@Test
	public void test(){
		long start = System.currentTimeMillis();
		System.out.println(f(100));//165580141
		long end = System.currentTimeMillis();
		System.out.println(end-start);//586ms
	}
	
	//实现f(n)：求n步台阶，一共有几种走法
	public int f(int n){
		if(n<1){
			throw new IllegalArgumentException(n + "不能小于1");
		}
		if(n==1 || n==2){
			return n;
		}
		return f(n-2) + f(n-1);
	}
}
```

递归

* 优点：大问题转化为小问题，可以减少代码量，同时代码精简，可读性好；

* 缺点：递归调用浪费了空间，而且递归太深容易造成堆栈的溢出。



## 6. Spring Bean的作用域的区别

```xml
	<bean id="book" class="com.atguigu.spring.beans.Book" scope="prototype">
	 	<property name="id" value="8"></property>
	</bean>
```

可以通过 Spring配置文件的 <bean> 的 scop属性配置。

* singleton：默认值。当IOC容器一创建就会创建bean的实例，而且是单例的，每次得到的都是同
* prototype：原型的。当IOC容器一创建不再实例化该bean，每次调用getBean方法时再实例化该bean，而且每调用一次创建一个对象
* request：每次请求实例化一个bean
* session：在一次会话中共享一个bean

```java
class SpringTest {

	//01.Spring Bean的作用域之间有什么区别

	//创建IOC容器对象
	ApplicationContext ioc = new ClassPathXmlApplicationContext("beans.xml");
	
	@Test
	void testBook() {
		Book book = (Book) ioc.getBean("book");
		Book book2 = (Book) ioc.getBean("book");
		System.out.println(book==book2);
	}
}
```

## 7. Spring 常用的事务传播属性和隔离级别

传播属性：

![](https://readingnotes.oss-cn-beijing.aliyuncs.com/Java%E9%AB%98%E9%A2%91%E9%9D%A2%E8%AF%95%E9%A2%98/%E4%BA%8B%E5%8A%A1%E7%9A%84%E4%BC%A0%E6%92%AD%E8%A1%8C%E4%B8%BA.png)

Spring 事务的传播行为 有以上几种，常用的就是 Required、newRequired。

Spring 默认使用 Required。

事务传播属性可以在@Transactional注解的propagation属性中定义。



例子：function A里调用了 function B ,两个方法都标注了@Transactional,开启了默认事务，因为事务的传播行为，方法B使用的是方法A的传播属性，此时 B方法中的操作必须同时成功，否则会回滚；如果方法B定义了 newRequired 传播属性，在方法B被调用时开启了新的事务，只要保证单次调用方法B中所有操作都成功，则可以提交数据库，因为事务的隔离性，可以不用管其他次调用是否成功。



隔离级别：

1)        **读未提交**：READ UNCOMMITTED

允许Transaction01读取Transaction02未提交的修改。

2)        **读已提交**：READ COMMITTED

            要求Transaction01只能读取Transaction02已提交的修改。

3)        **可重复读**：REPEATABLE READ

            确保Transaction01可以多次从一个字段中读取到相同的值，即Transaction01执行期间禁止其它事务对这个字段进行更新。

4)        **串行化**：SERIALIZABLE

            确保Transaction01可以多次从一个表中读取到相同的行，在Transaction01执行期间，禁止其它事务对这个表进行添加、更新、删除操作。可以避免任何并发问题，但性能十分低下。



 各个隔离级别解决并发问题的能力见下表

|                  | 脏读 | 不可重复读 | 幻读 |
| ---------------- | ---- | ---------- | ---- |
| READ UNCOMMITTED | 有   | 有         | 有   |
| READ COMMITTED   | 无   | 有         | 有   |
| REPEATABLE READ  | 无   | 无         | 有   |
| SERIALIZABLE     | 无   | 无         | 无   |



通过注解的方式设置事务的传播属性和隔离级别。

```java
@Transactional(propagation=Propagation.REQUIRED,isolation=Isolation.READ_COMMITTED)
```

## 8.SpringMVC如何解决Post请求的中文乱码问题？

解决Post请求乱码：

	在web.xml文件中配置过CharacterEncodingFilter滤器，并设置 过滤器参数：param-name、param-value

	配置 filter-mapping，设置过滤的路径。



```xml
<filter>
    <filter-name>CharacterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>UTF-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>CharacterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



解决get请求乱码：

	在service.xml 文件中 的 connector 配置 URIEncodoing = 'UTF-8'属性。

## 9.MyBatis中实体名和数据库字段名一致

1. 在写sql的时候给字段起别名。
2. 开启驼峰命名法
3. 在Mapper.xml 自定义高级映射，数据库字段名和实体字段名映射。



## 10.git分支相关命令

1. 创建分支：git branch  <分支名>
2. 切换分支：git checkout <分支名>

一步操作：git checkout -b <分支名>



3. 合并分支

   * 切换到master：git checkout master.

   * git merge <分支名>

4. 删除分支

   * 切换到master：git checkout master.
   * git branch -D <分支名>



## 11.Redis在项目实际使用

| 数据类型 | 使用场景                                                     |
| :------: | ------------------------------------------------------------ |
|  String  | 比如说 ，我想知道什么时候封锁一个IP地址。Incrby命令          |
|   Hash   | 存储用户信息【id，name，age】   Hset(key,field,value)   Hset(userKey,id,101)   Hset(userKey,name,admin)   Hset(userKey,age,23)   ----修改案例----   Hget(userKey,id)   Hset(userKey,id,102)   为什么不使用String   类型来存储   Set(userKey,用信息的字符串)   Get(userKey)   不建议使用String 类型 |
|   List   | 实现最新消息的排行，还可以利用List的push命令，将任务存在list集合中，同时使用另一个命令，将任务从集合中取出[pop]。   Redis—list数据类型来模拟消息队列。【电商中的秒杀就可以采用这种方式来完成一个秒杀活动】 |
|   Set    | 特殊之处：可以自动排重。比如说微博中将每个人的好友存在集合(Set)中，   这样求两个人的共通好友的操作。我们只需要求交集即可。 |
|   Zset   | 以某一个条件为权重，进行排序。   京东：商品详情的时候，都会有一个综合排名，还可以按照价格进行排名。 |

   

## 12.HashMap的底层实现

Map是一个接口，主要有 HashMap、LinkedHashMap、TreeMap、HashTable





