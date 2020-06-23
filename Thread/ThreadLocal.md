# 一致性问题

1. 多个主体对同一份数据无法达成共识
2. 分布式一致性问题、并发问题
3. 场景多,问题复杂,难以察觉

# 一致性问题解决办法

1. 排序(锁、互斥量、管程、屏障)
2. 投票(Paxos、Raft)

> 避免方式(ThreadLocal 等)


# 什么是ThreadLocal

> 定义:提供线程局部变量;一个线程局部变量在多个线程中,分别有独立的值(副本)
> 特点:简单(开箱即用)、快速(无额外开销)、安全(线程安全)
> 场景:多线程场景(资源持有、线程一致性、并发计算、线程安全场景)

> 实现原理: Java 中用hash表实现
> 应用范围: 几乎所有提供多线程特性的语言

# ThreadLocal API

* 构造函数 ThreadLocal<T>()
* 初始化 initialValue()
* 访问器 get/set
* 回收 remove

# ThreadLocal 使用场景

1. 线程资源持有
2. 线程资源一致性  
3. 线程安全
4. 分布式计算

# ThreadLocal 优点

1. 持有资源--持有线程资源线程的各部分使用,全局获取,减少编程难度
2. 线程一致--帮助需要保持线程一致的资源(数据库事务)维护一致,降低编程难度
3. 线程安全--帮助只考虑但线程的程序库,无缝向多线程场景迁移
4. 分布式计算--帮助分布式极端场景各个线程累计局部计算结果 


# 基本用法

```
public class ThreadLocalTest {

    public static ThreadLocal<Long> threadLocal = new ThreadLocal<Long>() {  //声明 ThreadLocal 对象通过 new 的方式创建
        @Override
        protected Long initialValue() {  //调用初始化 value 方法  这个方法只有在调用 get 方法的时候才会进行调用
            System.out.println("thread init run...");
            return Thread.currentThread().getId();
        }
    };

    public static void main(String[] args) {
        new Thread() {  //开启一个新的线程获取 ThreadLocal 中的值
            @Override
            public void run() {
                System.out.println("new thread");
                System.out.println(threadLocal.get());
            }
        }.start();
        threadLocal.set(2L);   //调用 set 方法设置 ThreadLocal 中的值
        threadLocal.remove();  //调用 remove 方法移除调 前边设置的 值,发现返回内容为初始化的线程 id
        System.out.println(threadLocal.get());
    }
}
```

# 并发编程下 多线程问题

> 并发编程下存在问题, 多个线程同时进行数据添加的情况下 会造成某些线程竞争添加成重复内容 1000 个线程最终添加的值会少于 1000

```
@RestController
public class TestController {
	static Integer value = 0;  // 
	
	@RequestMapping("/getValue")
    public Integer getValue() {
        return value.get();
    }
	
	@RequestMapping("/addValue")
    public Integer addValue() {
        value++;
        return value;
    }
}
```

## 通过加锁的 解决方案

> 通过 synchorized 进行修饰,不同的线程在并发请求时,所有的请求将变成串行,极大的降低了并发能力

```
@RestController
public class TestController {
	static Integer value = 0; 
	
	synchronized void add (){
		value++;
	}
	
	@RequestMapping("/getValue")
    public Integer getValue() {
        return value.get();
    }
	
	@RequestMapping("/addValue")
    public Integer addValue() {
        add();
        return value;
    }
}
```

