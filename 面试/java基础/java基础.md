

# Java基础知识

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

## 4.分布式、微服务、集群的微妙关系

