# java JUC

在 Java 5.0 提供了 java.util.concurrent （ 简称JUC ） 包，在此包中增加了在并发编程中很常用
的实用工具类， 用于定义类似于线程的自定义子系统，包括线程池、异步 IO 和轻量级任务框架。
提供可调的、灵活的线程池。还提供了设计用于多线程上下文中的 Collection 实现等 



用多线程的目的就是提高效率，如果使用不当会，不仅不能提高效率甚至更低。多线程开销比单线程大，多线程涉及到线程之间调度，Cpu内核切换、包括线程创建、销毁等消耗资源。



## 1. Volatile 关键字 内存可见性 

> 内存可见性（Memory Visibility）是指当某个线程正在使用对象状态而另一个线程在同时修改该状态，需要确保当一个线程修改了对象状态后，其他线程能够看到发生的状态变化。 



* 对于线程共享变量，JVM会对每个线程开辟独立的空间保存共享变量，操作完了再同步到主存，这就造成了多线程之间共享变量的不一致问题。

* 当然可以使用 synchronized 进行同步，对每个线程都进行排队，但是这样并发性非常差。

* 除此之外我们也可以使用一种更加轻量级的 volatile 变量 。

* 原理是调用了计算机内存栅栏，及时把缓存中的数据同步到主存中。（可以理解为就在主存中操作）

* 效率低在，JVM底层有重排序的优化，被volatile 修饰的变量不能重排序。

* 相较于 synchionized 是一种较为轻量级的同步策略。不同点在与
  * volatile 不具备"互斥性"。synchionized  只能被一个线程操作，volatile 可以被所有线程访问。
  * 不能保证变量的"原子性"。

## 2. 原子变量 CAS算法 

```java
package com.atguigu.juc;

import java.util.concurrent.atomic.AtomicInteger;

/*
 * 一、i++ 的原子性问题：i++ 的操作实际上分为三个步骤“读-改-写”
 * 		  int i = 10;
 * 		  i = i++; //10
 * 
 * 		  int temp = i;
 * 		  i = i + 1;
 * 		  i = temp;
 * 
 * 二、原子变量：在 java.util.concurrent.atomic 包下提供了一些原子变量。
 * 		1. volatile 保证内存可见性
 * 		2. CAS（Compare-And-Swap） 算法保证数据变量的原子性
 * 		   CAS 算法是硬件对于并发操作的支持,针对多处理器操作而设计的处理器中的一种特殊指令，用于管理			 对共享数据的并发访问。
 
 * 			CAS 包含了三个操作数：
 * 			1 内存值  V
 * 			2 预估值  A
 * 			3 更新值  B
 * 			当且仅当 V == A 时， V = B; 否则，不会执行任何操作。
 */
public class TestAtomicDemo {

	public static void main(String[] args) {
		AtomicDemo ad = new AtomicDemo();
		
		for (int i = 0; i < 10; i++) {
			new Thread(ad).start();
		}
	}
	
}

class AtomicDemo implements Runnable{
	
//	private volatile int serialNumber = 0;
	
	private AtomicInteger serialNumber = new AtomicInteger(0);

	@Override
	public void run() {
		
		try {
			Thread.sleep(200);
		} catch (InterruptedException e) {
		}
		
		System.out.println(getSerialNumber());
	}
	
	public int getSerialNumber(){
		return serialNumber.getAndIncrement();
	}
}

```

CAS可以理解为无锁算法，不涉及到上下文切换的问题，效率高。



原子变量：

java.util.concurrent.atomic   类的小工具包，支持在单个变量上解除锁的线程安全编程。事实上，此包中的类可
将 volatile 值、字段和数组元素的概念扩展到那些也提供原子条件更新操作的类。 

类 AtomicBoolean、 AtomicInteger、 AtomicLong 和 AtomicReference 的实例各自提供对
相应类型单个变量的访问和更新。每个类也为该类型提供适当的实用工具方法。 

**核心方法： boolean compareAndSet(expectedValue, updateValue) **



##3. 同步容器类 ConcurrentHashMap 锁分段机制 

```java
public class Hashtable<K,V>
    extends Dictionary<K,V>
    implements Map<K,V>, Cloneable, java.io.Serializable {
    ....
    }
```

HashMap 和 HashTable 的异同点：底层都是哈希表，HashTable实现了 synchionized 锁机制。多线程访问HashTable的时候，每次只允许一个线程操作，并行转换成了串行，效率非常差。HashTable的复合操作有线程安全，例如：如果存在则删除等操作。



ConcurrentHashMap 1.8以前使用锁分段机制，ConcurrentLevel 为锁分段级别，默认为16级别，就是把HashMap分为16段 ,（每一段还有独立的链表结构），每一段都有一个独立的锁，好处是多个线程并发访问时，访问不同段的线程不影响，这样增大了并行处理能力。JDK1.8底层改为CAS算法实现。

使用方式 和 HashMap相同。



## 4. CountDownLatch

> 这是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
>
> 当要完成一些运算 ，比如要统计各个接口传过来的数据进行统计，需要保证每个接口都调用完成才执行统计操作。



底层维护了一个变量，这个变量就是保证几个线程都完成，

```java
package com.atguigu.juc;

import java.util.concurrent.CountDownLatch;

/*
 * CountDownLatch ：闭锁，在完成某些运算是，只有其他所有线程的运算全部完成，当前运算才继续执行
 */
public class TestCountDownLatch {

    public static void main(String[] args) {
        final CountDownLatch latch = new CountDownLatch(5);
        LatchDemo ld = new LatchDemo(latch);

        long start = System.currentTimeMillis();

        for (int i = 0; i < 5; i++) {
            new Thread(ld).start();
        }

        try {                //如果不加这里，5个子线程，1个主线程是同时执行，不能计算出使用时间
            latch.await(); //如果其他线程没有完成，需要等待。
        } catch (InterruptedException e) {
        }

        long end = System.currentTimeMillis();

        System.out.println("耗费时间为：" + (end - start));
    }

}

class LatchDemo implements Runnable {

    private CountDownLatch latch;

    public LatchDemo(CountDownLatch latch) {
        this.latch = latch;
    }

    @Override
    public void run() {

        try {
            for (int i = 0; i < 50000; i++) {
                if (i % 2 == 0) {
                    System.out.println(i);
                }
            }
        } finally {
            latch.countDown();  //执行完了，闭锁递减1
        }
    }

}
```

## 5. 实现 Callable 接口 

> Callable 接口类似于 Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是 Runnable 不会返回结果，并且无法抛出经过检查的异常。 Callable 需要依赖FutureTask ， FutureTask 也可以用作闭锁 。

```java
package com.atguigu.juc;

import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.FutureTask;

/*
 * 一、创建执行线程的方式三：实现 Callable 接口。 相较于实现 Runnable 接口的方式，方法可以有返回值，并且可以抛出异常。
 * 
 * 二、执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。  FutureTask 是  Future 接口的实现类
 */
public class TestCallable {
	
	public static void main(String[] args) {
		ThreadDemo td = new ThreadDemo();
		
		//1.执行 Callable 方式，需要 FutureTask 实现类的支持，用于接收运算结果。
		FutureTask<Integer> result = new FutureTask<>(td);
		
		new Thread(result).start();
		
		//2.接收线程运算后的结果
		try {
			Integer sum = result.get();  //FutureTask 可用于 闭锁，底层实现了Runnable接口  										   //当上面线程执行完了才执行取返回结果操作。
			System.out.println(sum);
			System.out.println("------------------------------------");
		} catch (InterruptedException | ExecutionException e) {
			e.printStackTrace();
		}
	}

}

class ThreadDemo implements Callable<Integer>{

	@Override
	public Integer call() throws Exception {
		int sum = 0;
		
		for (int i = 0; i <= 100000; i++) {
			sum += i;
		}
		return sum;
	}
}

//Runnable 创建线程
/*class ThreadDemo implements Runnable{

	@Override
	public void run() {
	}
	
}*/
```



## 6. Lock 同步锁 

> 在 Java 5.0 之前，协调共享对象的访问时可以使用的机制只有 synchronized 和 volatile 。 Java 5.0 后增加了一些新的机制，但并不是一种替代内置锁的方法，而是当内置锁不适用时，作为一种可选择的高级功能。ReentrantLock 实现了 Lock 接口，并提供了与synchronized 相同的互斥性和内存可见性。但相较于
> synchronized 提供了更高的处理锁的灵活性。 

```java
package com.atguigu.juc;

import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/*
 * 一、用于解决多线程安全问题的方式：
 *
 * synchronized:隐式锁
 * 1. 同步代码块
 *
 * 2. 同步方法
 *
 * jdk 1.5 后：
 * 3. 同步锁 Lock
 * 注意：是一个显示锁，需要通过 lock() 方法上锁，必须通过 unlock() 方法进行释放锁
 */
public class TestLock {

    public static void main(String[] args) {
        Ticket ticket = new Ticket();

        new Thread(ticket, "1号窗口").start();
        new Thread(ticket, "2号窗口").start();
        new Thread(ticket, "3号窗口").start();
    }

}

class Ticket implements Runnable {

    private int tick = 100;

    private Lock lock = new ReentrantLock();

    @Override
    public void run() {
        while (true) {

            lock.lock(); //上锁

            try {
                if (tick > 0) {
                    try {
                        Thread.sleep(200);
                    } catch (InterruptedException e) {
                    }

                    System.out.println(Thread.currentThread().getName() + " 完成售票，余票为：" + --tick);
                }
            } finally {
                lock.unlock(); //释放锁  必须保证锁被释放，否则其他线程一直被阻塞
            }
        }
    }

}
```



## 7. 生产者消费者案例&虚假唤醒

```java
package com.atguigu.juc;

/*
 * 生产者和消费者案例
 */
public class TestProductorAndConsumer {

	public static void main(String[] args) {
		Clerk clerk = new Clerk();
		
		Productor pro = new Productor(clerk);
		Consumer cus = new Consumer(clerk);
		
		new Thread(pro, "生产者 A").start();
		new Thread(cus, "消费者 B").start();
		
		new Thread(pro, "生产者 C").start();
		new Thread(cus, "消费者 D").start();
	}
	
}

//店员
class Clerk{
	private int product = 0;
	
	//进货
	public synchronized void get(){//循环次数：0
        
	/*if*/while(product >= 1){//为了避免虚假唤醒问题，应该总是使用在循环中
			System.out.println("产品已满！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
			
		}
      //else{ *****************************
            System.out.println(Thread.currentThread().getName() + " : " + ++product);
			this.notifyAll();
     //}	  *****************************
	}
	
	//卖货
	public synchronized void sale(){//product = 0; 循环次数：0
	/*if*/while(product <= 0){
			System.out.println("缺货！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
		}
	//else{ *****************************
		System.out.println(Thread.currentThread().getName() + " : " + --product);
		this.notifyAll();
    //}	  *******************************
	}
}

//生产者
class Productor implements Runnable{
	private Clerk clerk;

	public Productor(Clerk clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
			}
			
			clerk.get();
		}
	}
}

//消费者
class Consumer implements Runnable{
	private Clerk clerk;

	public Consumer(Clerk clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			clerk.sale();
		}
	}
}
```

问题一：创建、消费不协调问题。

```java
//进货
public synchronized void get(){//循环次数：0
    if(product >= 1){//为了避免虚假唤醒问题，应该总是使用在循环中
        System.out.println("产品已满！");
    }else{
        System.out.println(Thread.currentThread().getName() + " : " + ++product);
    }
}
```

这样写会出现一个问题，两个线程同时执行,没有使用 “等待唤醒机制”，“产品已满”以后生产方法还会一直执行，消费方也有相同的问题，所以要修改成  ，“产品已满 ”后生产线程等待，唤醒消费线程进行消费，“缺货”以后消费线程等待，生产线程唤醒，修改为一下

```java
//店员- 进货
public synchronized void get(){//循环次数：0
    if(product >= 1){
        System.out.println("产品已满！");
        try {
				this.wait();
        	} catch (InterruptedException e) {
        }
        
    }else{
        this.notifyAll();
        System.out.println(Thread.currentThread().getName() + " : " + ++product);
    }
}
```

但是此时的 等待唤醒机制还是有问题的,如果把生产者改成以下，线程睡200ms，再把店员的get()方法改成 >=1  则货满，get() 只生产一个、sale() 只消费一个。

```java
//生产者
class Productor implements Runnable{
	private Clerk clerk;

	public Productor(Clerk clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
			}
			
			clerk.get();
		}
	}
}

-------------------------------------------------------------------
    //店员
class Clerk{
	private int product = 0;
	
	//进货
	public synchronized void get(){//循环次数：0
        
		if(product >= 1){
			System.out.println("产品已满！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
		}else{
            System.out.println(Thread.currentThread().getName() + " : " + ++product);
			this.notifyAll();
    	 }
	}
	
	//卖货
	public synchronized void sale(){//product = 0; 循环次数：0
		if(product <= 0){
			System.out.println("缺货！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
		}else{
            System.out.println(Thread.currentThread().getName() + " : " + --product);
            this.notifyAll();
    	}	 
	}
}
```

会出现程序不会停止的情况，一直等待，生产者有200ms的延迟 （sleep 20 0），消费者运行的快，程序循环20次但是最后没停止。

比如：消费者线程，循环次数剩下 1次 ；生产者线程，循环次数 2次；

某一时刻被消费者线程获取了CPU资源，开始进行消费， 如果这时  product = 0 则消费线程进入到等待状态，等待到 this.wait()的位置，同时释放锁。 （如果被唤醒从wait的位置接下去执行，JVM会记录每个线程执行的位置），生产者的循环次数变为0 ，这时生产者线程进入，这时  product = 0   不满足 product > = 1 则进入 else ,执行 ++product  操作。product = 1 ,然后 notifyAll() 方法结束，释放锁资源。 这时生产者和消费者同时抢锁，如果这时又被消费者线程抢到了锁，就会从wait 位置继续执行，else 块的代码得不到执行（因为没有进 if 判断），这时消费者线程已经循环结束，这时只剩 消费者一个线程 消费者执行 此时 product = 1 判断  product >= 1 则进入 wait，这时就没有线程唤醒创建者线程，就会一直等待，程序结束不了。



去掉 else  就可以解决

```java
    //店员
class Clerk{
	private int product = 0;
	
	//进货
	public synchronized void get(){//循环次数：0
        
		if(product >= 1){
			System.out.println("产品已满！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
            System.out.println(Thread.currentThread().getName() + " : " + ++product);
			this.notifyAll();
	}
	
	//卖货
	public synchronized void sale(){//product = 0; 循环次数：0
		if(product <= 0){
			System.out.println("缺货！");
			
			try {
				this.wait();
			} catch (InterruptedException e) {
			}
            System.out.println(Thread.currentThread().getName() + " : " + --product);
            this.notifyAll(); 
	}
}
```

现在只有 一个生产者 、一个消费者  ，如果都改成 两个 ，就会出现问题。

因为生产者 是 sleep 200ms 所以消费者线程先执行，s1 执行  经过判断 product <= 0 则  wait;   s2 也wait ,释放锁资源，这是生产者线程执行，生产一个产品 product = 1 ，这是 notifyAll()，两个消费者线程都被唤醒，接着从上次的 wait 的地方执行，执行两次 --product  则会出现负数，这就是虚假唤醒。

> As in the one argument version, interrupts and spurious wakeups are possible,  and this method should always be used in a loop: 
>
> ```
>      synchronized (obj) {
>          while (<condition does not hold>)
>              obj.wait();
>          ... // Perform action appropriate to condition
>      }
> ```



官方API中给出  ，wait() 方法 只能被用在循环中。所以把 if 改成  while 就可以了，在下次被唤醒的时候 再 进入到 while 再判断一次。



## 8. Condition 控制线程通信 

> * Condition 接口描述了可能会与锁有关联的条件变量。这些变量在用法上与使用 Object.wait 访问的隐式监视器类似，但提供了更强大的功能。需要特别指出的是，单个 Lock 可能与多个 Condition 对象关联。为了避免兼容性问题， Condition 方法的名称与对应的 Object 版本中的不同。
> * 在 Condition 对象中，与 wait、 notify 和 notifyAll 方法对应的分别是await、 signal 和 signalAll 



用Lock 实现上面的 生产者 消费者问题，一定要把  unlock 放到 finally 里面

```java
package com.atguigu.juc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/*
 * 生产者消费者案例：
 */
public class TestProductorAndConsumerForLock {

	public static void main(String[] args) {
		Clerk clerk = new Clerk();

		Productor pro = new Productor(clerk);
		Consumer con = new Consumer(clerk);

		new Thread(pro, "生产者 A").start();
		new Thread(con, "消费者 B").start();

	}

}

class Clerk {
	private int product = 0;

	private Lock lock = new ReentrantLock();
	private Condition condition = lock.newCondition();

	// 进货
	public void get() {
		lock.lock();

		try {
			if (product >= 1) { 
				System.out.println("产品已满！");

				try {
					condition.await();
				} catch (InterruptedException e) {
				}

			}
			System.out.println(Thread.currentThread().getName() + " : "	+ ++product);

			condition.signalAll();
		} finally {
			lock.unlock();
		}

	}

	// 卖货
	public void sale() {
		lock.lock();

		try {
			if (product <= 0) {
				System.out.println("缺货！");

				try {
					condition.await();
				} catch (InterruptedException e) {
				}
			}

			System.out.println(Thread.currentThread().getName() + " : "	+ --product);

			condition.signalAll();

		} finally {
			lock.unlock();
		}
	}
}

// 生产者
class Productor implements Runnable {

	private Clerk clerk;

	public Productor(Clerk clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			try {
				Thread.sleep(200);
			} catch (InterruptedException e) {
				e.printStackTrace();
			}

			clerk.get();
		}
	}
}

// 消费者
class Consumer implements Runnable {

	private Clerk clerk;

	public Consumer(Clerk clerk) {
		this.clerk = clerk;
	}

	@Override
	public void run() {
		for (int i = 0; i < 20; i++) {
			clerk.sale();
		}
	}

}
```



### 8.1 使用 Condition 实现交替打印

就是使用  Condition  控制线程的 唤醒 和 等待。

```java
package com.atguigu.juc;

import java.util.concurrent.locks.Condition;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

/*
 * 编写一个程序，开启 3 个线程，这三个线程的 ID 分别为 A、B、C，每个线程将自己的 ID 在屏幕上打印 10 遍，要求输出的结果必须按顺序显示。
 *	如：ABCABCABC…… 依次递归
 */
public class TestABCAlternate {
	
	public static void main(String[] args) {
		AlternateDemo ad = new AlternateDemo();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				
				for (int i = 1; i <= 20; i++) {
					ad.loopA(i);
				}
				
			}
		}, "A").start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				
				for (int i = 1; i <= 20; i++) {
					ad.loopB(i);
				}
				
			}
		}, "B").start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				
				for (int i = 1; i <= 20; i++) {
					ad.loopC(i);
					
					System.out.println("-----------------------------------");
				}
				
			}
		}, "C").start();
	}

}

class AlternateDemo{
	
	private int number = 1; //当前正在执行线程的标记
	
	private Lock lock = new ReentrantLock();
	private Condition condition1 = lock.newCondition();
	private Condition condition2 = lock.newCondition();
	private Condition condition3 = lock.newCondition();
	
	/**
	 * @param totalLoop : 循环第几轮
	 */
	public void loopA(int totalLoop){
		lock.lock();
		
		try {
			//1. 判断
			if(number != 1){
				condition1.await();
			}
			
			//2. 打印
			for (int i = 1; i <= 1; i++) {
				System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
			}
			
			//3. 唤醒
			number = 2;
			condition2.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void loopB(int totalLoop){
		lock.lock();
		
		try {
			//1. 判断
			if(number != 2){
				condition2.await();
			}
			
			//2. 打印
			for (int i = 1; i <= 1; i++) {
				System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
			}
			
			//3. 唤醒
			number = 3;
			condition3.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
	public void loopC(int totalLoop){
		lock.lock();
		
		try {
			//1. 判断
			if(number != 3){
				condition3.await();
			}
			
			//2. 打印
			for (int i = 1; i <= 1; i++) {
				System.out.println(Thread.currentThread().getName() + "\t" + i + "\t" + totalLoop);
			}
			
			//3. 唤醒
			number = 1;
			condition1.signal();
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			lock.unlock();
		}
	}
	
}
```



## 9. ReadWriteLock 读写锁 

ReadWriteLock 为读写锁，回一个接口。

> * ReadWriteLock 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁以由多个 reader 线程同时保持。写入锁是独占的。。
>
> *  ReadWriteLock 读取操作通常不会改变共享资源，但执行写入操作时，必须独占方式来获取锁。对于读取操作占多数的数据结构。 ReadWriteLock 能提供比独占锁更高的并发性。而对于只读的数据结构，其中包含的不变性可以完全不需要考虑加锁操作 



```java
package com.atguigu.juc;

import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/*
 * 1. ReadWriteLock : 读写锁
 *
 * 写写/读写 需要“互斥”
 * 读读 不需要互斥
 *
 */
public class TestReadWriteLock {

    public static void main(String[] args) {
        ReadWriteLockDemo rw = new ReadWriteLockDemo();

        new Thread(new Runnable() {

            @Override
            public void run() {
                rw.set((int) (Math.random() * 101));
            }
        }, "Write:").start();


        for (int i = 0; i < 100; i++) {
            new Thread(new Runnable() {

                @Override
                public void run() {
                    rw.get();
                }
            }).start();
        }
    }

}

class ReadWriteLockDemo {

    private int number = 0;

    private ReadWriteLock lock = new ReentrantReadWriteLock();

    //读
    public void get() {
        lock.readLock().lock(); //上锁

        try {
            System.out.println(Thread.currentThread().getName() + " : " + number);
        } finally {
            lock.readLock().unlock(); //释放锁
        }
    }

    //写
    public void set(int number) {
        lock.writeLock().lock();

        try {
            System.out.println(Thread.currentThread().getName());
            this.number = number;
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```



## 10. 线程八锁



```java
package com.atguigu.juc;

/*
 * 题目：判断打印的 "one" or "two" ？
 * 
 * 1. 两个普通同步方法，两个线程，标准打印， 打印? //one  two
 * 2. 新增 Thread.sleep() 给 getOne() ,打印? //one  two
 * 3. 新增普通方法 getThree() , 打印? //three  one   two
 * 4. 两个普通同步方法，两个 Number 对象，打印?  //two  one
 * 5. 修改 getOne() 为静态同步方法，打印?  //two   one
 * 6. 修改两个方法均为静态同步方法，一个 Number 对象?  //one   two
 * 7. 一个静态同步方法，一个非静态同步方法，两个 Number 对象?  //two  one
 * 8. 两个静态同步方法，两个 Number 对象?   //one  two
 * 
 * 线程八锁的关键：
 * ①非静态方法的锁默认为  this,  静态方法的锁为 对应的 Class 实例
 * ②某一个时刻内，只能有一个线程持有锁，无论几个方法。
 */
public class TestThread8Monitor {
	
	public static void main(String[] args) {
		Number number = new Number();
		Number number2 = new Number();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
				number.getOne();
			} 
		}).start();
		
		new Thread(new Runnable() {
			@Override
			public void run() {
//				number.getTwo();
				number2.getTwo();
			}
		}).start();
		
		/*new Thread(new Runnable() {
			@Override
			public void run() {
				number.getThree();
			}
		}).start();*/
		
	}

}

class Number{
	
	public static synchronized void getOne(){//Number.class
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
		}
		
		System.out.println("one");
	}
	
	public synchronized void getTwo(){//this
		System.out.println("two");
	}
	
	public void getThree(){
		System.out.println("three");
	}
	
}
```



## 11.  线程池




>  * 一、线程池：提供了一个线程队列，队列中保存着所有等待状态的线程。避免了创建与销毁额外开销，提高了响应的速度。
>  * 二、线程池的体系结构：
>  * java.util.concurrent.Executor : 负责线程的使用与调度的根接口
>  * |--**ExecutorService 子接口: 线程池的主要接口
>      * |--ThreadPoolExecutor 线程池的实现类
>      * |--ScheduledExecutorService 子接口：负责线程的调度
>          * |--ScheduledThreadPoolExecutor ：继承 ThreadPoolExecutor， 实现 ScheduledExecutorService
>
>     三、工具类 : Executors
>  * ExecutorService newFixedThreadPool() : 创建固定大小的线程池
>  * ExecutorService newCachedThreadPool() : 缓存线程池，线程池的数量不固定，可以根据需求自动的更改数量。
>  * ExecutorService newSingleThreadExecutor() : 创建单个线程池。线程池中只有一个线程
>     *
>  * ScheduledExecutorService newScheduledThreadPool() : 创建固定大小的线程，可以延迟或定时的执行任务。

```java
package com.atguigu.juc;

import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.Callable;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;

public class TestThreadPool {
	
	public static void main(String[] args) throws Exception {
		//1. 创建线程池
		ExecutorService pool = Executors.newFixedThreadPool(5);
		
		List<Future<Integer>> list = new ArrayList<>();
		
		for (int i = 0; i < 10; i++) {
			Future<Integer> future = pool.submit(new Callable<Integer>(){

				@Override
				public Integer call() throws Exception {
					int sum = 0;
					
					for (int i = 0; i <= 100; i++) {
						sum += i;
					}
					
					return sum;
				}
				
			});

			list.add(future);
		}
		
		pool.shutdown();
		
		for (Future<Integer> future : list) {
			System.out.println(future.get());
		}
		
		
		
		/*ThreadPoolDemo tpd = new ThreadPoolDemo();
		
		//2. 为线程池中的线程分配任务
		for (int i = 0; i < 10; i++) {
			pool.submit(tpd);
		}
		
		//3. 关闭线程池
		pool.shutdown();*/
	}
	
//	new Thread(tpd).start();
//	new Thread(tpd).start();

}

class ThreadPoolDemo implements Runnable{

	private int i = 0;
	
	@Override
	public void run() {
		while(i <= 100){
			System.out.println(Thread.currentThread().getName() + " : " + i++);
		}
	}
	
}
```



```java
public class TestScheduledThreadPool {

	public static void main(String[] args) throws Exception {
		ScheduledExecutorService pool = Executors.newScheduledThreadPool(5);
		
		for (int i = 0; i < 5; i++) {
			Future<Integer> result = pool.schedule(new Callable<Integer>(){

				@Override
				public Integer call() throws Exception {
					int num = new Random().nextInt(100);//生成随机数
					System.out.println(Thread.currentThread().getName() + " : " + num);
					return num;
				}
				
			}, 1, TimeUnit.SECONDS);
			
			System.out.println(result.get());
		}
		
		pool.shutdown();
	}
	
}
```



## 12. Fork/Join 框架与线程池的区别 

* Fork/Join 框架：就是在必要的情况下，将一个大任务，进行拆分(fork)成若干个小任务（拆到不可再拆时），再将一个个的小任务运算的结果进行 join 汇总。 

* 采用 “工作窃取”模式（work-stealing）：当执行新的任务时它可以将其拆分分成更小的任务执行，并将小任务加到线程队列中，然后再从一个随机线程的队列中偷一个并把它放在自己的队列中。 

* 相对于一般的线程池实现， fork/join框架的优势体现在对其中包含的任务的处理方式上.在一般的线程池中， 如果一个线程正在执行的任务由于某些原因无法继续运行， 那么该线程会处于等待状态。 而在fork/join框架实现中，如果某个子问题由于等待另外一个子问题的完成而无法继续运行。 那么处理该子问题的线程会主动寻找其他尚未运行的子问题来执行.这种方式减少了线程的等待时间， 提高了性能。 



```java
package com.atguigu.juc;

import java.time.Duration;
import java.time.Instant;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.ForkJoinTask;
import java.util.concurrent.RecursiveTask;
import java.util.stream.LongStream;

import org.junit.Test;

public class TestForkJoinPool {
	
	public static void main(String[] args) {
		Instant start = Instant.now();
		
		ForkJoinPool pool = new ForkJoinPool();
		
		ForkJoinTask<Long> task = new ForkJoinSumCalculate(0L, 50000000000L);
		
		Long sum = pool.invoke(task);
		
		System.out.println(sum);
		
		Instant end = Instant.now();
		
		System.out.println("耗费时间为：" + Duration.between(start, end).toMillis());//166-1996-10590
	}
	
	@Test
	public void test1(){
		Instant start = Instant.now();
		
		long sum = 0L;
		
		for (long i = 0L; i <= 50000000000L; i++) {
			sum += i;
		}
		
		System.out.println(sum);
		
		Instant end = Instant.now();
		
		System.out.println("耗费时间为：" + Duration.between(start, end).toMillis());//35-3142-15704
	}
	
	//java8 新特性
	@Test
	public void test2(){
		Instant start = Instant.now();
		
		Long sum = LongStream.rangeClosed(0L, 50000000000L)
							 .parallel()
							 .reduce(0L, Long::sum);
		
		System.out.println(sum);
		
		Instant end = Instant.now();
		
		System.out.println("耗费时间为：" + Duration.between(start, end).toMillis());//1536-8118
	}

}

class ForkJoinSumCalculate extends RecursiveTask<Long>{

	/**
	 * 
	 */
	private static final long serialVersionUID = -259195479995561737L;
	
	private long start;
	private long end;
	
	private static final long THURSHOLD = 10000L;  //临界值
	
	public ForkJoinSumCalculate(long start, long end) {
		this.start = start;
		this.end = end;
	}

	@Override
	protected Long compute() {
		long length = end - start;
		
		if(length <= THURSHOLD){
			long sum = 0L;
			
			for (long i = start; i <= end; i++) {
				sum += i;
			}
			
			return sum;
		}else{
			long middle = (start + end) / 2;
			
			ForkJoinSumCalculate left = new ForkJoinSumCalculate(start, middle); 
			left.fork(); //进行拆分，同时压入线程队列
			
			ForkJoinSumCalculate right = new ForkJoinSumCalculate(middle+1, end);
			right.fork(); //
			
			return left.join() + right.join();
		}
	}
	
}

```

































































