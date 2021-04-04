

# Java基础知识

## 1.线程的3种创建方式

### (1)重写Thread的run()方法

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

### (2)Runnable()方法实现

```java
// 方法2：Runnable方法实现
new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("线程2");
    }
}).start();
```

### (3)Callable()方法实现

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

