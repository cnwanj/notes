# 7、助记符
通过javap反编译class文件：
进入到类文件目录下输入：javap -c 类名。
### 相关助记符
- aload_0：
	- 装载了一个引用类型。
	- 第一个字母：a：引用类型，i：整数类型，l：长整型
	- 0：表示第0个变量。
- invokespecial：有init、private、super.method()等，其中\<init>表示初始化方法。
- getstatic：获取静态成员。
- bipush：-128 ~ 127整数范围内（8位带符号的整数，256），放到栈顶。
- sipush：小于-128和大于127的整数范围（16位带符号整数），放到栈顶。
- iconst_m1：表示-1，放到栈顶。
- iconst_0 ~ iconst_5：表示0 ~ 5，放到栈顶。
- ldc：int float String常量，放到栈顶。
- ldc2_w：long double常量，放到栈顶。
# 8、JVM的4种引用级别
如果一个对象存在着指向它的引用，那么这个对象就不会被GC回收？
>答案：不一定，有局限性仅适用于强引用，软引用在内存空间不足时会被GC回收，弱引用只要发生GC就会被回收。

4个强引用分别为：强引用 > 软引用 > 弱引用 > 虚引用。
>- 除了强引用之外，其他三个引用都继承于一个父类Reference\<T>。
>- 软引用（SoftReference）、弱引用（WeakReference）、虚引用（PhantomReference）、此外还有一个：最终引用（FinalReference）。
>- Reference中有一个get方法，用于返回引用对象。
### （1）强引用
>Object obj = new Object();
>约定：引用obj、引用对象new Object();

强引用对象失效的情况有以下两种：
1. 生命周期结束（作用域失效）
```java
public void f() {
	Object obj = new Object();
}// 当方法执行结束后，强引用指向的引用对象（new Object()）会被等待被GC回收。
```
2. 将引用置为空null
```java
public void f() {
	Object obj = new Object();
	// 将obj引用置为null后，引用对象就失去了引用指向，（new Object()）将会被GC回收。
	obj = null;
}
```
>注意：除了以上两个情况以外，其他任何时候GC都不会回收引用对象。
### （2）软引用（SoftReference）
回收机制：当JVM内存不足时，GC会主动的回收软引用对象。
#### 软引用被GC回收例子：
```java
import java.lang.ref.SoftReference;
import java.util.ArrayList;
import java.util.List;

public class Main {

    public static void main(String[] args) throws Exception {
        // 创建一个软引用对象，用了装饰模式：将对象层层包裹
        SoftReference<Object> soft = new SoftReference<>(new Object());
        List<byte[]> list = new ArrayList<>();

        // 开启一个线程，判断软引用对象是否被回收
        new Thread(() -> {
            try {
                // 将线程睡眠100ms
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            // 获取软引用对象，若为null，则表示已被回收（内存不足会被回收）
            if (soft.get() == null) {
                System.out.println("软引用被回收了");
                System.exit(0);
            }
        }).start();

        // 不断的往list集合中添加数据，直到堆溢出
        while (true) {
            if (soft.get() != null)
                // 每次添加1M
                list.add(new byte[1024 * 1024]);
        }
    }
}
```
### （3）弱引用（WeakReference）
回收机制：只要GC执行，就会将弱引用对象进行回收。
```java
import java.lang.ref.WeakReference;

public class Main {
    public static void main(String[] args) {
        // 创建一个弱引用对象
        WeakReference<Object> weak = new WeakReference<>(new Object());
        System.out.println(weak.get() == null ? "已被回收" : "未被回收");
        // 建议GC执行一次回收（存在概率）
        System.gc();
        System.out.println(weak.get() == null ? "已被回收" : "未被回收");
    }
}
```
>**执行结果为：**
>未被回收
>已被回收
### （4）虚引用（PhantomReference）
>又称幻影引用或幽灵引用。

- 是否使用虚引用和引用本身没有任何关系。
- 无法通过虚引用来获取对象本身。
- 通过get()方法获取虚引用对象时为null，get() -> null。
- 虚引用不会单独使用，一般会和引用队列一起使用。
#### 虚引用价值价值：
当GC回收一个虚引用对象时，就会将虚引用放入到引用队列中，之后（当引用出队之后）再去回收该对象。因此，我们可以使用虚引用和引用队列实现。
>在对象被GC之前，会进行一些额外的操作。
>GC => 若有虚引用 => 虚引用入队 => 虚引用出队 => 回收对象

验证虚引用被GC回收前所执行的一系列操作。
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

/**
 * 虚引用进行GC前会发生以下操作：
 * GC -> 虚引用入队 -> 虚引用出队 -> GC回收
 */
public class Main {

    public static void main(String[] args) {
        Object obj = new Object();
        // 创建一个引用队列
        ReferenceQueue<Object> queue = new ReferenceQueue<>();
        // 创建一个虚引用对象
        PhantomReference<Object> phantomObject = new PhantomReference<>(obj, queue);
        // 判断对象是否入队，这里没有入队，输出为null
        System.out.println(queue.poll());
        // 将强引用对象置为null，通过GC后会被回收
        obj = null;
        // 执行了一次GC回收
        System.gc();
        System.out.println("GC执行了...");
        // 执行了GC后对象将会入队，这里输出为对象地址
        System.out.println(queue.poll());

    }
}
```
特殊情况：如果虚引用对象重写了finalize()方法，那么JVM将会延迟虚引用的入队时间。
```java
import java.lang.ref.PhantomReference;
import java.lang.ref.ReferenceQueue;

/**
 * 虚引用进行GC前会发生以下操作：
 * GC -> 虚引用入队 -> 虚引用出队 -> GC回收
 */
public class Main {

    public static void main(String[] args) {
        PhantomObject obj = new PhantomObject();
        // 创建一个引用队列
        ReferenceQueue<PhantomObject> queue = new ReferenceQueue<>();
        // 创建一个虚引用对象
        PhantomReference<PhantomObject> phantomObject = new PhantomReference<>(obj, queue);
        // 判断对象是否入队，这里没有入队，输出为null
        System.out.println(queue.poll());
        // 将强引用对象置为null，通过GC后会被回收
        obj = null;
        // 经过循环后，延迟到了第4次才入队
        for (int i = 0; i < 5; i++) {
            // 执行了一次GC回收
            System.gc();
            System.out.println((i + 1) + " GC执行了...");
            // 执行了GC后对象将会入队，这里输出为对象地址
            System.out.println(queue.poll());
        }
    }
}

class PhantomObject{
    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize即将被GC回收...");
    }
}
```
#### 最终引用（FinalReference）
通过Finalizer来自动回收一些不需要的对象，如构造方法。
```java
// 存在唯一一个实现类Finalizer：
final class Finalizer extends FinalReference<Object>
```
# 9、使用某个引用实现缓存的淘汰策略
- 实现目的：将数据库数据存放到缓存中，当缓存达到90%，需要降到60%。
- 可以通过LRU算法来实现。
- 一般的淘汰策略：根据容量（缓存个数）+ LRU进行淘汰。

通过将对象包装成软引用（在内存不足时会将软引用回收），再存入到缓存中。

```java
import java.lang.ref.SoftReference;
import java.util.HashMap;
import java.util.Map;

/**
 * 本程序的优势：
 * 当JVM内存不足时，GC会自动回收软引用，
 * 因此本程序无需考虑OOM（Out Of Memory）问题。
 */
public class Main1 {

    Map<String, SoftReference<Student>> caches = new HashMap<>();

    public static void main(String[] args) {
        SoftReference<Student> stu = new SoftReference<>(new Student());

    }
    /**
     * 将学生信息放入缓存中
     * @param id
     * @param stu
     */
    void setCaches(String id, Student stu) {
        caches.put(id, new SoftReference<>(new Student()));
    }
    /**
     * 从缓存中获取学生信息
     * @param id
     * @return
     */
    Student getCaches(String id) {
        return caches.get(id) == null ? null : caches.get(id).get();
    }
}
class Student{}
```
# 10、双亲委派
**双亲：**
- JVM自带的加载器（在JVM的内部所包含，c++语言编写）。
- 用户自定义加载器（独立与JVM之外的加载器，java编写）。

加载器描述：
- JVM自带加载器
	- 根加载器（Bootstrap）：加载jdk\jre\lib\rt.jar（包含了平时编写代码大部分的API）；可以指定加载某个jar（-Xbootclasspath = xx.jar）。
	- 扩展类加载器（Extention）：加载jdk\jre\lib\ext\*.jar（包含了jdk中的扩展jar包）；可以指定加载（-Djava.ext.dirs = xx.jar）。
	- 系统加载器/应用加载器（System/App）：加载classpath（自己写的类）；可以指定加载（-Djava.class.path = 类/jar）。
- 自定义加载器
	- 都是该抽象类（java.lang.ClassLoader）的子类

**委派：**
- 当一个加载器要加载类的时候，若自己加载不了。
- 就逐层向上交由双亲去加载，当双亲的某个加载器加载成功后，再向下返回成功。
- 如果所有的双亲和自己都无法加载，则会抛出异常。
![在这里插入图片描述](https://gitee.com/cnwanj/cloud_image/raw/master/image/2020122621201277.png)

### （1）bootstrap根加载器案例：
```java
public class Main {
    public static void main(String[] args) throws Exception {
        // Object是rt.jar包中的类
        Class obj = Class.forName("java.lang.Object");
        ClassLoader bootstrap = obj.getClassLoader();
        // 打印出来是null，所以使用了根加载器
        System.out.println(bootstrap);
    }
}
```
打印出来是null，点进去（getClassLoader）看源码注解如下：
```java
/**
  * Returns the class loader for the class.  Some implementations may use
  * null to represent the bootstrap class loader. This method will return
  * null in such implementations if this class was loaded by the bootstrap
  * class loader.
  * /
```
>翻译过来：
>返回该类的类加载器。一些实现可能使用null表示bootstrap类加载器。如果此类由bootstrap加载，则在此类实现中此方法将返回 null 。

就是用返回null表示使用了bootstrap根加载器。
### （2）app应用加载器案例：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 自定义类是在classpath中
        Class myClass = Class.forName("org.gxuwz.arithmatic.lanqiao.MyClass");
        ClassLoader app = myClass.getClassLoader();
        // 打印出来是"sun.misc.Launcher$AppClassLoader@18b4aac2"，app加载器
        System.out.println(app);
    }
}
class MyClass {}
```
>打印出来的是：
>sun.misc.Launcher$AppClassLoader@18b4aac2

表示使用了AppClassLoader加载器。
注意：
- 定义类加载：最终实际加载类加载器。
- 初始化类加载：首次加载类的加载器。

为什么应用（App）加载器又叫系统（System）加载器：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 获取系统加载器打印结果
        ClassLoader systemClassLoader = ClassLoader.getSystemClassLoader();
        System.out.println(systemClassLoader);

        // 获取应用加载器打印结果
        Class myClass = Class.forName("org.gxuwz.arithmatic.lanqiao.MyClass");
        ClassLoader app = myClass.getClassLoader();
        System.out.println(app);
    }
}
class MyClass {}
```
不管是系统加载器还是应用加载器打印，两次打印结果一致，都是AppClassLoader应用加载器：
>sun.misc.Launcher\$AppClassLoader@18b4aac2
>sun.misc.Launcher$AppClassLoader@18b4aac2

从应用加载器开始逐层获取父加载器：

```java
public class Main {
    public static void main(String[] args) throws Exception {
        // 获取应用加载器app
        Class myClass = Class.forName("org.gxuwz.arithmatic.lanqiao.MyClass");
        ClassLoader app = myClass.getClassLoader();
        System.out.println(app);
        
        // 获取应用加载器的父加载器 -> 扩展加载器Extention
        ClassLoader parent1 = app.getParent();
        System.out.println(parent1);

        // 获取扩展加载器的父加载器 -> 根加载器bootstrap
        ClassLoader parent2 = parent1.getParent();
        System.out.println(parent2);

        // 获取根加载器的父加载器 -> 无
        ClassLoader parent3 = parent2.getParent();
        System.out.println(parent3);
    }
}
class MyClass {}
```
打印结果如下：
应用加载器App -> Extention扩展加载器 -> 根加载器Bootstrap -> 无
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122621581993.png)
### （3） 查看类加载器源码
（1）类加载器根据二进制名称加载（binary names）：
```java
// 正常的包类名
“java.lang.String”
// $表示匿名内部类：JSpinner包下的匿名内部类DefaultEditor
“javax.swing.JSpinner$DefaultEditor”
// FileBuilder内部类中的第一个匿名内部类
“java.security.KeyStoreBuilderBuilderFileBuilder$1”
// URLClassLoader包中的第三个内部类的第一个内部类
“java.net.URLClassLoader$3$1”
```
（2）数组的加载器类型
- 数组的加载器类型和数组元素加载器类型相同。
- 原生类型（不是类，如int、long...）的数组是没有加载器。

类加载器数组描述源码文档：
```java
/**
 * <p> <tt>Class</tt> objects for array classes are not created by class
 * loaders, but are created automatically as required by the Java runtime.
 * The class loader for an array class, as returned by {@link
 * Class#getClassLoader()} is the same as the class loader for its element
 * type; if the element type is a primitive type, then the array class has no
 * class loader.
 * /
```
>大概意思：
>返回的数组类的类加载器与其元素类型的类加载器相同；如果元素类型是原始类型，则数组类没有类加载器。

用一个例子来验证一下：
```java
public class Main {
    public static void main(String[] args) throws Exception {
        MyClass[] cls = new MyClass[1];
        MyClass cl = new MyClass();
        // 数组加载器
        System.out.println(cls.getClass().getClassLoader());
        // 数组元素加载器
        System.out.println(cl.getClass().getClassLoader());

        int[] arr = new int[1];
        // 原始类型数组加载器
        System.out.println(arr.getClass().getClassLoader());
        // 原始类型数组元素加载器
        System.out.println(int.class.getClassLoader());
    }
}
class MyClass {}
```
- 因为原生类型时基本数据类型，不是一个类，所以类加载器不能加载。
- 如果为null，有两种可能，一种是没有加载器，另一种是加载类型为根加载器。

输出结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201226223508415.png)
（3）xx.class类文件可能存在本地，也可能来自网络network或者在运行时动态产生。

```java
/**
 * <p> However, some classes may not originate from a file; they may originate
 * from other sources, such as the network, or they could be constructed by an
 * application.
 * /
```
（4）如果类文件来自与网络，则需要继承ClassLoader父类，并重写findClass()和loadClassData()两个方法。

```java
/**
 * <p> The network class loader subclass must define the methods {@link
 * #findClass <tt>findClass</tt>} and <tt>loadClassData</tt> to load a class
 * from the network.  Once it has downloaded the bytes that make up the class,
 * it should use the method {@link #defineClass <tt>defineClass</tt>} to
 * create a class instance.  A sample implementation is:
 *
 * <blockquote><pre>
 *     class NetworkClassLoader extends ClassLoader {
 *         String host;
 *         int port;
 *
 *         public Class findClass(String name) {
 *             byte[] b = loadClassData(name);
 *             return defineClass(name, b, 0, b.length);
 *         }
 *
 *         private byte[] loadClassData(String name) {
 *             // load the class data from the connection
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote>
 * /
```
### （4）自定义类加载器的实现
需要继承ClassLoader类并重写findClass()和loadClassData()两个方法，findClass()调用loadClassData()方法。
```java
import com.sun.xml.internal.messaging.saaj.util.ByteOutputStream;

import java.io.File;
import java.io.FileInputStream;

public class Main1 extends ClassLoader {

    // 默认构造方法，使用根加载器getSystemClassLoader()
    public Main1() {
        super();
    }

    // 自定义使用的加载器
    public Main1(ClassLoader parent) {
        super(parent);
    }

    public Class findClass(String name) {
        byte[] b = loadClassData(name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassData(String name) {
        name = splitDot("out.production.MyProject_02_arithmetic." + name) + ".class";
        FileInputStream inputStream = null;
        ByteOutputStream outputStream = null;
        byte[] res = null;
        try {
            // 文件输入流获取文件数据
            inputStream = new FileInputStream(new File(name));
            // 获取输出流
            outputStream = new ByteOutputStream();
            // 创建缓冲区
            byte[] buf = new byte[2];
            int len = -1;
            // 循环的将缓冲区输出到输出流中
            while ((len = inputStream.read(buf)) != -1) {
                outputStream.write(buf, 0, len);
            }
            // 返回输出流字节数组
            res =  outputStream.getBytes();
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if (inputStream != null) inputStream.close();
                if (outputStream != null) outputStream.close();
            }catch (Exception e) {
                e.printStackTrace();
            }
        }
        return res;
    }

    // 将"."替换成"/"
    private String splitDot(String s) {
        return s.replace(".", "/");
    }

    public static void main(String[] args) throws Exception {
        Main1 main1 = new Main1();
        Class myClass = main1.loadClass("org.gxuwz.arithmatic.lanqiao.MyClass");
        System.out.println(myClass.getClassLoader());
        MyClass myClass1 = (MyClass)(myClass.newInstance());
        myClass1.hello();
    }
}
class MyClass{
    public void hello() {
        System.out.println("hello...");
    }
}
```
>注意：
>- loadClassData(String name) 是文形式字符串a/b/MyClass.class，并且开头out.production...
>- findClass(String name) 是全类名形式a.b.MyClass，并且开头是：包名.类名.class。

以上代码打印出来的仍然是APP加载器：sun.misc.Launcher$AppClassLoader@18b4aac2，因为MyClass.class类文件在classpath中，所以使用的仍是应用加载器，需要将其移出到外部目录，如：D:/MyClass.class，这样就会触发自定义加载器如下，主要修改了path，且需要将classpath中的类文件删除：
```java
import com.sun.xml.internal.messaging.saaj.util.ByteOutputStream;

import java.io.File;
import java.io.FileInputStream;

public class Main1 extends ClassLoader {

    private String path;

    // 默认构造方法，使用根加载器getSystemClassLoader()
    public Main1() {
        super();
    }

    public Class findClass(String name) {
        byte[] b = loadClassData(name);
        return defineClass(name, b, 0, b.length);
    }

    private byte[] loadClassData(String name) {
        // 若自定义路径不为空，设置自定义路径名称
        if (path != null) {
            // 获取全路径中末尾的类名称
            name = path + name.substring(name.lastIndexOf(".") + 1) + ".class";
        } else {
            name = splitDot("out.production.MyProject_02_arithmetic." + name) + ".class";
        }
        FileInputStream inputStream = null;
        ByteOutputStream outputStream = null;
        byte[] res = null;
        try {
            // 文件输入流获取文件数据
            inputStream = new FileInputStream(new File(name));
            // 获取输出流
            outputStream = new ByteOutputStream();
            // 创建缓冲区
            byte[] buf = new byte[2];
            int len = -1;
            // 循环的将缓冲区输出到输出流中
            while ((len = inputStream.read(buf)) != -1) {
                outputStream.write(buf, 0, len);
            }
            // 返回输出流字节数组
            res =  outputStream.toByteArray();
        }catch (Exception e) {
            e.printStackTrace();
        }finally {
            try {
                if (inputStream != null) inputStream.close();
                if (outputStream != null) outputStream.close();
            }catch (Exception e) {
                e.printStackTrace();
            }
        }
        return res;
    }

    // 将"."替换成"/"
    private String splitDot(String s) {
        return s.replace(".", "/");
    }

    public static void main(String[] args) throws Exception {
        Main1 main1 = new Main1();
        // 自定义类路径，D:/MyClass.class
        main1.path = "D:/";
        Class<?> myClass = main1.loadClass("org.gxuwz.arithmatic.lanqiao.MyClass");
        System.out.println(myClass.getClassLoader());
    }
}
```
输出自定加载器：org.gxuwz.arithmatic.lanqiao.Main1@45ee12a7
### 自定义加载器流程：
loadClass() -> findClass() -> loadClassData()
- 在启动类中通过自定义加载器调用loadClass()。
- loadClass()再调用自定义方法findClass()。
- findClass()再调用loadClassData()。
### 加载器结论：
- 类加载器只会把同一个类加载一次（位置也相同）。
- 先委托AppClassLoader加载，AppClassLoader会在classpath路径中寻找是否存在，若存在则直接加载。
- 否则才有可能（可能还会交给扩展和根加载器）交给自定义加载器加载。
- 双亲委派体系中下层的加载器是引用上层parent加载器，各个加载器之间不是继承关系。

loadClass源码解析：

```java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException {
    synchronized (getClassLoadingLock(name)) {
        // First, check if the class has already been loaded
        Class<?> c = findLoadedClass(name);
        if (c == null) {
            long t0 = System.nanoTime();
            try {
            	// 1.若父加载器不为空，则委托父加载器进行加载
                if (parent != null) {
                    c = parent.loadClass(name, false);
                } else {
                	// 2.若父加载器为空，说明双亲委派调用了顶层加载器（根加载器）
                    c = findBootstrapClassOrNull(name);
                }
            } catch (ClassNotFoundException e) {
                // ClassNotFoundException thrown if class not found
                // from the non-null parent class loader
            }
			// 3.若为null，表示父加载器加载失败，只能由自己加载
            if (c == null) {
                // If still not found, then invoke findClass in order
                // to find the class.
                long t1 = System.nanoTime();
                c = findClass(name);

                // this is the defining class loader; record the stats
                sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                sun.misc.PerfCounter.getFindClasses().increment();
            }
        }
        if (resolve) {
            resolveClass(c);
        }
        return c;
    }
}
```
### 双亲委派机制的优势：
可以防止用户自定义的类和rt.jar中的类重名，而造成混乱。

如下代码是自定义一个java.langMath类（和jdk中的rt.jar重名）：
```java
package java.lang;

public class Math {
	public static void main(String[] args) {
		System.out.println("Math...");
	}
}
```
运行结果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227212942164.png)
>原因：
>因为双亲委派机制，越顶层的加载器优先级越高，根加载器bootstrap是最顶层，优先级最高，则会被优先加载，所以加载的是rt.jar中的java.lang.Math类，而该类不是我们自定义的Math类，因此没有main方法，就会抛出异常。
### 双亲委派特点：
- 若存在继承关系：继承的双方（父类、子类）都必须是同一个加载器，否则报错。
- 如果不存在继承关系：子类加载器可以访问父类的加载器（自定义加载器可以访问App加载器），反之则不行（App加载器不能访问子类加载器）。
# 11、OSGi规范
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201227222607758.png)
### OSGi特点：
1.网状结构。
2.屏蔽掉硬件的异构性。例如：可以将项目部署在网络上，可以在A节点上远程操作B节点。在操作上可以对硬件无感。也可以在A节点上对B节点上的项目进行运维、部署，并且在理想的情况下维护期间不用暂停、重启，天然支持。
# 12、类的卸载
- 系统自带的（系统加载器、扩展加载器、根加载器）：这些加载器加载的类是不会被卸载。
- 用户自定义加载器所加载的类，会被GC卸载。

配置类的加载和卸载参数，将会打印类的加载和卸载情况，如下：
>-XX:+TraceClassLoading 
>-XX:+TraceClassUnloading

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228125254140.png)
### （1）自定义加载器不会卸载类
通过自定义加载器加载D盘下的类文件，然后设置为null，调用GC回收，查看加载器是否卸载类：
```java
public class ClassLoaderUnLoading {
    public static void main(String[] args) throws Exception {
        // 自定义加载器
        MyClassLoader classLoader = new MyClassLoader();
        // 指定D盘类文件
        classLoader.path = "D:/";
        Class<?> aClass = classLoader.loadClass("com.vovhh.AClass");
        System.out.println("加载了");
        aClass = null;
        classLoader = null;
        System.gc();
    }
}
```
自定义加载器在GC后被卸载了，输出结果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228130109472.png)
### （2）系统自带加载器不会卸载类
使用系统加载器加载类：

```java
public class ClassLoaderUnLoading {
    public static void main(String[] args) throws Exception {
        // 自定义加载器
        MyClassLoader classLoader = new MyClassLoader();
        // 指定D盘类文件
        Class<?> aClass = classLoader.loadClass("com.vovhh.AClass");
        System.out.println("加载了：" + aClass.getClassLoader());
        aClass = null;
        classLoader = null;
        System.gc();
    }
}
```
系统自带加载器不会卸载类，输出结果如下所示：
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020122813001683.png)
# 13、JVM检测工具
（1）jsp：查看Java进程（java命令，jdk/bin目录中）。
命令：jps
![在这里插入图片描述](https://gitee.com/cnwanj/cloud_image/raw/master/image/20201228131253714.png)
（2）jstat：只能查看当前时刻的内存情况，新生代、老生代等区域的内存情况。
命令：jstat -gcutil jps进程编号
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228131629571.png)
（3）jmap：查看堆内存使用情况。
命令：jmap -heap jps进程编号
（4）jconsole：图形界面查看内存使用情况。
命令：jconsole 或 jconsole jps进程编号
选择对应的进程，点击连接：
![在这里插入图片描述](https://img-blog.csdnimg.cn/202012281331499.png)
查看内存情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228133345887.png)
切换到“内存”界面，点击右上角的“执行gc”，发现gc回收的内存很少，说明当前进程存在问题，需要优化。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228133815284.png)
（5）jvisualvm：可视化工具查看（可以查看到具体的哪一个实体）。
命令：jvisualvm
步骤：左侧类进程 -> 监视 -> 堆Dump（将内存信息放到硬盘中） -> 查找 -> 最大对象列表
如下所示：
![在这里插入图片描述](https://gitee.com/cnwanj/cloud_image/raw/master/image/20201228135020564.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228135114621.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L1loX3loX25ld19ZaA==,size_16,color_FFFFFF,t_70)
（6）通过VM参数实现：当内存溢出时，自动将溢出时刻的堆内存信息Dump下来。
>-Xmx100m
>-Xms100m
>-XX:+HeapDumpOnOutOfMemoryError

在VM中设置如上参数：最大内存、最小内存、记录堆内存溢出信息。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228140554405.png)
堆内存溢出后会Dump到"java_pid12120.hprof"文件中（一般是在项目根目录）。
![在这里插入图片描述](https://gitee.com/cnwanj/cloud_image/raw/master/image/20201228140501359.png)
通过jvisualvm根据进行装入以上文件：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228141719987.png)
按照之前的步骤查看hprof文件的内存使用情况：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20201228142040623.png)
# 14、GC调优
1. 在已知条件相同的前提下，牺牲低延迟来换取高吞吐量，或者反之。
2. 随着软件硬件的发展，二者都可能提高。
### GC发展史
JVM自身在GC上进行了很多次改进和升级。
JVM默认的GC：CMS GC -> G1 GC(jdk9) -> ZGC(jdk11)
- Serial GC：
串行GC，是一种在单核环境下的串行回收器（单线程），当GC回收的时刻，其他线程必须等待，一般不使用。
- Parallel GC：
在Serial的基础上使用了多线程技术，提高了吞吐量。
- CMS GC：
CMS使用了多线程技术，同时使用了"标记-清除"算法，可以极大的提升了效率（尤其在低延迟上有很大的提升）。jdk9后被逐渐废弃，原因配置繁琐，参数多，对开发者经验要求高，且会产生较多的碎片。
- G1 GC：
jdk9开始默认使用。特点：会将堆内存划分为很多个大小相等的regoin，并会对这些区域状态进行标记，方便在GC时快速定位出空闲的regoin，专门对有垃圾的region进行GC，极大的提高了GC的效率，G1采用"标记-整理"算法。
- Z GC：
jdk11开始提供的全新GC，回收TB级别的垃圾在毫秒范围。
### 案例
如果通过检测工具发现：Minor GC和Full GC都在频繁的回收，如何优化？
- Minor GC回收频繁的原因：新生代中的对象太多（触发Minor GC） -> 短生命周期的对象太多 -> 造成逃逸到老年代的对象变多 -> 新生代多和老年代多会（触发Full GC）。
- 可以尝试调大新生代的最大空间。
- 调大新生代晋升到老年代的年龄，降低短生命周期的对象转移到老年代的概率。