# Java学习笔记

## 常用命令

### 单文件编译运行
  
`java main.java`编译
`javac main`运行类文件
有包名编译运行时需加上包名，编译时使用`-encoding`可以设置编码，在`src`目录编译运行。
```
javac com\itranswarp\learnjava\Main.java
//javac -encoding UTF-8 com\itranswarp\learnjava\Main.java
java com.itranswarp.learnjava.Main
```
## 语法

### 基本语法

####  数据类型

byte: -128~127
布尔类型boolean
数组类型int[]
`int[] ns = new int[5];`
`int[] ns = new int[] { 68, 79, 91, 85, 62 };`

#### var关键字

类似C++的auto自动定义变量
`var sb = new StringBuilder();`
实际上会自动变成：
`StringBuilder sb = new StringBuilder();`

#### 输出 
  
```java
double d = 3.1415926;
System.out.printf("%.2f\n", d);
```

#### 输入
  
```java
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in); // 创建Scanner对象
        System.out.print("Input your name: "); // 打印提示
        String name = scanner.nextLine(); // 读取一行输入并获取字符串
        System.out.print("Input your age: "); // 打印提示
        int age = scanner.nextInt(); // 读取一行输入并获取整数
        System.out.printf("Hi, %s, you are %d\n", name, age); // 格式化输出
    }
}
```

#### 类型
 
基本类型：byte，short，int，long，boolean，float，double，char；
引用类型：所有class和interface类型。
引用类型判等要用`.equals()`，`=`相当于检查是否指向同一个对象。


### 面向对象

#### 继承
  
继承关键字：`extends `
在Java中，没有明确写extends的类，编译器会自动加上`extends Object`。
Java只允许一个class继承自一个类。
`super`关键字表示父类（超类）。子类引用父类的字段时，可以用`super.fieldName`
子类构造函数调用父类的构造函数：
```java
class Student extends Person {
    protected int score;

    public Student(String name, int age, int score) {
        super(name, age); // 调用父类的构造方法Person(String, int)
        this.score = score;
    }
}

```
向上转型：把一个子类型安全地变为更加抽象的父类型：
`Person p = new Student();`

#### override（C++的重载）

在继承关系中，子类如果定义了一个与父类方法签名完全相同的方法，被称为覆写（Override）。
加上`@Override`可以让编译器帮助检查是否进行了正确的覆写。希望进行覆写，但是不小心写错了方法签名，编译器会报错。
#### abstract

通过abstract定义的方法是抽象方法，它只有定义，没有实现。抽象方法定义了子类必须实现的接口规范；

定义了抽象方法的class必须被定义为抽象类，从抽象类继承的子类必须实现抽象方法.
```java
abstract class Person {
    public abstract void run();
}
```

#### interface（接口）

如果一个抽象类没有字段，所有方法全部都是抽象方法：
就可以把该抽象类改写为接口：`interface`
Java的接口特指interface的定义，表示一个接口类型和一组方法签名，而编程接口泛指接口规范，如方法签名，数据格式，网络协议等。
一个类可以实现多个`interface`
```java
class Student implements Person, Hello { // 实现了两个interface
    ...
}
```

#### 包作用域
package
包作用域是指一个类允许访问同一个package的没有public、private修饰的class，以及没有public、protected、private修饰的字段和方法。

#### final

用final修饰class可以阻止被继承：
```java
package abc;

// 无法被继承:
public final class Hello {
    private int n = 0;
    protected void hi(int t) {
        long i = t;
    }
}
```
用final修饰method可以阻止被子类覆写：
```java
package abc;

public class Hello {
    // 无法被覆写:
    protected final void hi() {
    }
}
```
用final修饰field可以阻止被重新赋值：

```java
package abc;

public class Hello {
    private final int n = 0;
    protected void hi() {
        this.n = 1; // error!
    }
}
```
用final修饰局部变量可以阻止被重新赋值：
```java
package abc;

public class Hello {
    protected void hi(final int t) {
        t = 1; // error!
    }
}
```

#### 内部类inner Class
```java
// inner class
public class Main {
    public static void main(String[] args) {
        Outer outer = new Outer("Nested"); // 实例化一个Outer
        Outer.Inner inner = outer.new Inner(); // 实例化一个Inner
        inner.hello();
    }
}

class Outer {
    private String name;

    Outer(String name) {
        this.name = name;
    }

    class Inner {
        void hello() {
            System.out.println("Hello, " + Outer.this.name);
        }
    }
}
```
Inner Class除了有一个this指向它自己，还隐含地持有一个Outer Class实例，可以用Outer.this访问这个实例。所以，实例化一个Inner Class不能脱离Outer实例。

#### Java的编译运行过程(简单）：

class,classpath,模块等见[参考资料3.12-3.14](https://liaoxuefeng.com/books/java/oop/basic/classpath-jar/index.html)

[JRE与JDK的区别](https://www.runoob.com/w3cnote/the-different-of-jre-and-jdk.html)

#### Java包装类型

包装类型是基本类型相对应的引用类型。如int对应的引用类型是java.lang.Integer，可直接使用。

#### Javabean

JavaBean是一种符合命名规范的class，它通过getter和setter来定义属性；

我们通常把一组对应的读方法（getter）和写方法（setter）称为属性（property）；

使用Introspector.getBeanInfo()可以获取属性列表。（先`import java.beans.*;`）

### 异常处理

`public byte[] getBytes(String charsetName) throws UnsupportedEncodingException {
    ...
}
`

在方法定义的时候，使用throws Xxx表示该方法可能抛出的异常类型。调用方在调用的时候，必须强制捕获这些异常或者同样声明可能抛出异常，否则编译器会报错。

- 待扩展
### 反射

JVM为每个加载的class及interface创建了对应的Class实例来保存class及interface的所有信息；获取一个class对应的Class实例后，就可以获取该class的所有信息；通过Class实例获取class信息的方法称为反射（Reflection）。

可以通过反射获取**类**的字段(Field)、方法(Method)、构造方法和继承关系，可通过传入类的具体实例得到对应的具体信息。

反射是为了解决在运行期，对某个实例一无所知的情况下，如何调用其方法。
#### 动态代理
...

### 注解
#### 编译检查

@Override - 检查该方法是否是重写方法。如果发现其父类，或者是引用的接口中并没有该方法时，会报编译错误。
@Deprecated - 标记过时方法。如果使用该方法，会报编译警告。
@SuppressWarnings - 指示编译器去忽略注解中声明的警告。
#### 通过反射获取自定义注解
...

### 多线程

#### 创建线程

Java用`Thread`对象表示一个线程，通过调用`start()`启动一个新线程；`start()`方法会在内部自动调用实例的`run()`方法。
可以通过`Thread`派生类或者实现`Runnable`接口的类来创建线程（覆写类`run()`方法）。
`Thread.sleep()`暂停线程，单位为ms；`Thread.setPriority(int n)`设置线程调度优先级。

通过对另一个线程对象调用`join()方法`可以等待其执行结束,例如`t.join()`；

#### 中断线程

对目标线程调用`interrupt()`方法可以请求中断一个线程，目标线程通过检测`isInterrupted()`标志获取自身是否已中断。如果目标线程处于等待状态，该线程会捕获到`InterruptedException`。

线程间共享变量需要使用`volatile`关键字标记，确保每个线程都能读取到更新后的变量值。(`volatile`解决的是可见性问题)

`volatile`关键字的目的是告诉虚拟机：

每次访问变量时，总是获取主内存的最新值；
每次修改变量后，立刻回写到主内存。

#### 守护线程

守护线程是指为其他线程服务的线程。在JVM中，所有非守护线程都执行完毕后，无论有没有守护线程，虚拟机都会自动退出。
在调用`start()`方法前，调用`setDaemon(true)`把该线程标记为守护线程。
守护线程不能持有需要关闭的资源（如打开文件等）。

#### 线程同步


```java
public synchronized void add(int n) { // 锁住this
    count += n;
} // 解锁

```
等价于
```java
public void add(int n) {
    synchronized(this) { // 锁住this
        count += n;
    } // 解锁
}
```

`synchronized(this)`锁住的是对象，同一个对象里只有一个线程能执行`synchronized`修饰的代码段，其他线程可以执行非`synchronized`修饰的代码段。

<details>
<summary>代码展开</summary>

```java
public class Main{
    public static void main(String[] args) {
        System.out.println("使用关键字synchronized");
        Mthreads mt=new Mthreads();
        Thread thread1 = new Thread(mt, "mt1");
        Thread thread2 = new Thread(mt, "mt2");
        Thread thread3 = new Thread(mt, "mt3");
        thread1.start();
        thread2.start();
        thread3.start();
    }
}
class Mthreads implements Runnable{
    private int count;

    public Mthreads() {
        count = 0;
    }

    public void countAdd() {
        synchronized(this) {
            for (int i = 0; i < 5; i ++) {
                try {
                    System.out.println(Thread.currentThread().getName() + ":" + (count++));
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    //非synchronized代码块，未对count进行读写操作，所以可以不用synchronized
    public void printCount() {
        for (int i = 0; i < 5; i ++) {
            try {
                System.out.println(Thread.currentThread().getName() + " count:" + count);
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public synchronized void printCount2() {
        for (int i = 0; i < 5; i ++) {
            try {
                System.out.println(Thread.currentThread().getName() + " count:" + count);
                Thread.sleep(100);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public void run() {
        String threadName = Thread.currentThread().getName();
        if (threadName.equals("mt1")) {
            countAdd();
        } else if (threadName.equals("mt2")) {
            printCount();
        }    
        else if(threadName.equals("mt3")){
            printCount2();
        }
    }
}
```
</details>

这种情况下，mt1和mt3不能同时运行,mt2可以和mt1/mt3同时运行。

我的理解：`synchronized(object)`修饰代码段相当于让代码段运行前得到`object`对应的锁，运行后释放锁。

#### wait和notify

`wait()`方法在当前获取的锁对象上调用，等待时释放锁，唤醒时**试图**重新获得锁。
`notify()/notifyAll()`在锁对象上调用，唤醒一个/所有在该锁对象上等待的线程。

#### 读写锁


`ReadWriteLock`可以实现读写锁，允许多个线程同时读，但只要有一个线程在写，其他线程就必须等待。（悲观锁）

`StampedLock`是一种乐观锁，读的过程中允许写锁写入，有小概率导致数据不一致时用悲观锁处理冲突。
### 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)

[菜鸟教程](https://www.runoob.com/java/java-tutorial.html)