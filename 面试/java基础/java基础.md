# 一、Java基础知识

## 1.线程的3种创建方式

(1)重写Thread的run()方法

```java
// 方法1：重写run方法，旧写法
new Thread() {
    @Override
    public void run() {
        System.out.println("线程1");
    }
}.start();

// 重写run方法，lambda表达式写法，() -> {}：表示无参无返回值，(res) -> {return r}：表示有参有返回值
new Thread(() -> {
    System.out.println("线程1");
}).start();
```

(2)Runnable()方法实现

```java
// 方法2：Runnable方法实现
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程2");
    }
}).start();
```

(3)Callable()方法实现

> 与Runnble的区别就是Callble该方法具有返回值。

```java
// 方法3：Callable方法实现，有返回值
Callable<Integer> callable = () -> {
    return 3;
};
FutureTask<Integer> task = new FutureTask<>(callable);
new Thread(task).start();
System.out.println(task.get());
```

## 2.MySQL和SQLServer的区别

(1)公司

MySQL是由瑞典 MySQL AB 公司开发的关系型数据库管理系统，目前属于 Oracle 旗下公司，源码开源。SQL Server 是Microsoft 公司推出的关系型数据库管理系统，源码不对外开放 。  

(2)性能

mysql简单查询 速度快， 最好不要使用函数/join/group等方式查询。  sqlserver的简单查询速度不如mysql，但复杂查询时，性能下降的不明显。

(3)机器配置要求

相对于sqlserver来说，mysql对机器配置要求不高。 一台pd925/2Gram/sataII硬盘//linux2.6内核 的机器可以轻松处理几千万条记录的数据表。 对于sqlserver，我们使用了 双xeon5110/4Gram/raid10(6块sataII硬盘）/win2000ADserver的机器，数据表也有几千万条记录，结果负荷一高就崩溃了，很不稳定。 

(4)主键自增

mysql有自动增长的数据类型。而sqlserver没有自增类型，需要单独一张表来自定义自增序列号。

## 3.spring事务的失效场景

参考链接：https://blog.csdn.net/Jiangbohao_/article/details/107182301

(1)数据库存储引擎不支持事务

如mysql中的MyISAM引擎不支持事务，而InnoDB引擎才支持事务。

(2)非public修饰方法使用

@Transactional用在public方法上才会生效，否则会导致事务失效。

(3)未被Spring管理

若方法所在的类没有注入spring容器中，这个类就不会被spring管理，这样事务就会失效了。

如下情况就会失效：

```java
// @Service
public class UserServiceImpl implements UserService {
    @Transactional
    public void insertUser(User user) {
    }
}  
```

(4)方法间调用

A方法调用B方法，A没有事务，B设置了事务，会导致B方法事务失效。

(5)异常被处理掉了

在整个事务中使用了try catch，没有将异常抛出，会直接导致事务失效。如下情况就会导致事务失效。

```java 
@Transactional
public void method() {
    try {
        // 更新数据
    } catch (Exception e) {
        return;
    }
}
```

(6)异常类型错误

```java
@Service
public class UserServiceImpl implements UserService {
    @Transactional
    public void updateOrder(User user) {
        try {
            // 更新数据
        } catch (Exception e) {
            throw new Exception("更新错误");
        }
    }
}
```

这样事务也是不生效的，因为默认回滚的是：RuntimeException，如果你想触发其他异常的回滚，需要在注解上配置一下，如：@Transactional(rollbackFor = Exception.class)这个配置仅限于 Throwable 异常类及其子类。 

> 为什么会失效呢？因为Spring在扫描Bean的时候会自动为标注了@Transactional注解的类生成一个代理类（proxy）,当有注解的方法被调用的时候，实际上是代理类调用的，代理类在调用之前会开启事务，执行事务的操作，但是同类中的方法互相调用，相当于this.B()，此时的B方法并非是代理类调用，而是直接通过原有的Bean直接调用，所以注解会失效。

## 3.取余和取模的区别

> 取余和取模是极其相似，常常会让人以为两者是同一样性质，其实不然。在符号相同时，两者不会冲突，符号不同时就会有以下区别：

- 取余：向0舍入。
- 取模：向负无穷舍入。

### 符号相同：

比如：9 / 5会有两个商1和2。

9 = 5 * 1 + 4或9 = 5 * 2 + (-1)，因为是向0舍入，取前者计算结果，9 余 5 = 4，9 模 5 = 4。

### 符号不同：

比如：9 / (-5)会有两个商-1和-2。

9 = (-5) * (-1) + 4或9 = (-5) * (-2) + (-1)，9 余 -5 = 4，9 模 -5 = -1。

### 原则定义：

- 取余：rem(x, y) = x - y * fix(x / y)。
- 取模：mod(x, y) = x - y * floor(x / y)。

fix()向0取整，floor()向负无穷取整。

以x = 9，y = -5为例：

- fix(9, -5) = -1, floor(9, -5) = -2。

- rem(9, -5) = 4，mod(9, -5) = -1。

## 4.Java8新特性

java8新特性主要有：速度提升（红黑树）、Lambda表达式（箭头函数）、方法引用（Integer::compare）、stream工具API、日期工具类优化。

## 5.常用Lamda表达式

```java
// List转Map，并去重key
Map<Integer, User> map = list.stream().collect(Collectors.toMap(User::getId, Function.identity(), (key, value) -> key));

// 添加逗号分隔符
String ids = list.stream().map(User::getId).distinct().collect(Collectors.joining(","));

// 求和
long sum = list.stream().mapToLong(User::getAge).sum();
BigDecimal bigSum = list.stream().map(User::getMoney).reduce(BigDecimal.ZERO, BigDecimal::add);

// 求和存在map中
Map<Integer, Integer> mapSum = list.stream().collect(Collectors.toMap(User::getId, User::getAge, Integer::sum));

// 分组
Map<Integer, List<User>> map = list.stream().collect(Collectors.groupingBy(User::getId, Collectors.toList()));

// 倒序排序
list.sort(Comparator.comparing(User::getId).thenComparing(User::getAge).reversed());
```

## 6.动态代理模式实现AOP

### 6.1静态代理

静态代理，只能代理固定的对象。

```java
interface ClothFactory {

	void produceCloth();
}

class ProxyClothFactory implements ClothFactory {

	private ClothFactory clothFactory;

	public ProxyClothFactory(ClothFactory clothFactory) {
		this.clothFactory = clothFactory;
	}

	@Override
	public void produceCloth() {
		System.out.println("生产前...");
		clothFactory.produceCloth();
		System.out.println("生产后...");
	}
}

class AntaClothFactory implements ClothFactory {

	@Override
	public void produceCloth() {
		System.out.println("生产安踏...");
	}
}

public class StaticProxyTest {
	public static void main(String[] args) {
		// 通过代理工厂去生产安踏
		ClothFactory clothFactory = new ProxyClothFactory(new AntaClothFactory());
		clothFactory.produceCloth();
	}
}
```

### 6.2动态代理，实现AOP

通过Proxy.newProxyInstance实现动态代理。

```java
interface Human {

	String study();

	void eat(String food);

}

class SuperMan implements Human {

	@Override
	public String study() {
		return "我要学习变强";
	}

	@Override
	public void eat(String food) {
		System.out.println("我今天吃" + food);
	}
}

class ProxyFactory {

	// 获取动态代理实例对象
	public static Object getProxyInstance(Object object) {
		// 类加载器、类接口、代理放大类对象
		return Proxy.newProxyInstance(
			object.getClass().getClassLoader(),
			object.getClass().getInterfaces(),
			new MyInvocation(object));
	}
}

// 自定义代理调度器，用于获取被代理对象的方法
class MyInvocation implements InvocationHandler {

	private Object object;

	public MyInvocation(Object object) {
		this.object = object;
	}

	// 通过代理对象掉用方法时，就会自动调用invoke()方法
	@Override
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("执行方法前调用...");
		Object invoke = method.invoke(object, args);
		System.out.println("执行方法后调用...");
		return invoke;
	}
}

public class ProxyTest {

	public static void main(String[] args) {
		// 获取superman的代理对象
		Human proxyInstance = (Human) ProxyFactory.getProxyInstance(new SuperMan());
		System.out.println(proxyInstance.study());
		proxyInstance.eat("饺子");
		System.out.println("----------------------------");
		// 代理衣服工厂
		ClothFactory clothFactory = (ClothFactory) ProxyFactory.getProxyInstance(new AntaClothFactory());
		clothFactory.produceCloth();
	}
}
```



# 二、Spring基础知识

## 1.Bean的生命周期

主要步骤：实例化、初始化、创建和使用、销毁

- 解析xml或注解配置的类，创建Bean定义BeanDefinition（如Bean的作用域是单例或原型，加载模式是不是懒加载）。
- 通过BeanDefinition反射实例化对象。
- 将实例化的Bean填充对象属性。
- 实现Aware相关接口，如BeanNameAware、BeanClassLoaderAware、BeanFactoryAware等接口。
- 初始化前调用BeanPostProcessor实现类，执行方法postProcessorBeforeInitialization方法。
- Bean初始化（配置init-method、实现InitializingBean接口）。
- 初始化后调用BeanPostProcessor实现类，执行方法postProcessorAfterInitialization方法。
- 将Bean放入Map中，供业务使用。
- 销毁Bean（PreDestroy注解、配置destroy-method、调用DisposableBean接口的destroy()方法）。

# 三、MQ基础问题

参考：

博客链接：https://doocs.github.io/advanced-java

CSDN链接：https://blog.csdn.net/lettyisme/article/details/85233008

## 1.为什么用MQ？

> 目的：解耦、异步、削峰

- 解耦：A系统调接口发送数据到BCD系统，若C系统不需要这个数据，或者是E系统需要这个数据，那A系统负责人将忙不过来，导致A系统与其他系统耦合严重。若使用MQ，A系统只需将数据通过MQ发送出去，其他系统需要直接从MQ这里消费，不需要则取消MQ消费即可。A系统压根不用管后面的路程，也不用考虑其他系统是否调用成功、失败超时等情况。
- 异步：A系统需要在本地存库，需要3ms，还要同步数据到BCD系统，需要800ms，总延时过长。使用MQ，发送到本地后发送3条MQ消息的时间的9ms，A系统从接收请求到发送MQ只需要总时长10ms，至于同步数据就让系统去处理了。
- 削峰：A系统平常风平浪静，并发数也就30，一到晚上8点，并发量就达到100k，假如MySQL承受的并发量为20k，一旦这些请求打到MySQL会直接崩溃。这时候可以将请求发送到MQ中，A系统每次拉取20k请求，从而保证了系统的稳定性。

## 2.消息丢失怎么解决？

> 消息丢失可能会发生在：生产者、MQ、消费者

这里通过RabbitMQ来描述：

- 生产者丢失：
  - 生产者在发送数据到MQ时，因为网络故障导致消息丢失。
  - 此时可以选择MQ提供的事务功能，当MQ没有收到消息时，就会发送异常提示给生产者，这时候就可以通过事务进行回滚。
  - 还可以在生产者这开启confirm模式，每次写消息都会分配一个全局唯一的ID，若成功写入消息到MQ中，则MQ会回传一个成功的Ack消息。若写入失败，则回调一个nack接口，告诉生产者可以重试。
  - 一般选择confirm作为解决消息丢失的处理方案，因为事务会导致消息堵塞，而confirm是通过异步来处理，速度较快。
- MQ丢失：
  - 开启MQ持久化。
  - 创建队列的时候开启持久化。
  - 将发送消息的deliveryMode设置为2，表示持久化。
- 消费者丢失：
  - 关闭自动ack回应。
  - 在代码中进行自定义ack。
  - 若一直没有ack，MQ会把这个消息分配给其他消费者消费。

