# Java学习笔记

## 常用命令

### 单文件编译运行
  
`java main.java`编译
`javac main`运行类文件
有包名编译运行时需加上包名，编译时使用`-encoding`可以设置编码，在`src`目录编译运行。
```
javac com\itranswarp\learnjava\Main.java && java com.itranswarp.learnjava.Main
javac -encoding UTF-8 com\itranswarp\learnjava\Main.java && java com.itranswarp.learnjava.Main
```
### 在markdown中使用代码展开

<details>
<summary>代码展开</summary>

```Java
    这里填入代码
```
</details>

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
#### Semaphore

`Semaphore`是信号量，用于限制资源被最多N个线程访问。
`semaphore.acquire();`获取，`semaphore.release();`释放。

#### 线程池

`ExecutorService`接口表示线程池

```Java
// 创建固定大小的线程池:
ExecutorService executor = Executors.newFixedThreadPool(3);
// 提交任务:
executor.submit(task1);
executor.submit(task2);
executor.submit(task3);
executor.submit(task4);
executor.submit(task5);
```
FixedThreadPool：线程数固定的线程池；
CachedThreadPool：线程数根据任务动态调整的线程池；
SingleThreadExecutor：仅单线程执行的线程池。
放入ScheduledThreadPool的任务可以定期反复执行。

[Java面试：为什么不建议使用Executors创建线程池？](https://blog.csdn.net/ARPOSPF/article/details/120276045)

#### Future

`Callable`接口，和`Runnable`接口比，它多了一个返回值。

可以用`Future`获取`Callable`任务异步执行的结果，获取会在执行完成前阻塞。

```java
ExecutorService executor = Executors.newFixedThreadPool(4); 
// 定义任务:
Callable<String> task = new Task();
// 提交任务并获得Future:
Future<String> future = executor.submit(task);
// 从Future获取异步执行返回的结果:
String result = future.get(); // 可能阻塞
```

使用`CompletableFuture`可以实现回调、线程的串行化、并行化。

#### ThreadLocal

使用ThreadLocal来设置单个线程独有的上下文（“局部变量”）。

ThreadLocal实例通常总是以静态字段初始化如下：

```Java
static ThreadLocal<User> threadLocalUser = new ThreadLocal<>();
```

典型使用如下: 

```Java
void processUser(user) {
    try {
        threadLocalUser.set(user);
        step1();
        step2();
        log();
    } finally {
        threadLocalUser.remove();
    }
}
...
        User u = threadLocalUser.get();
...
```

`ThreadLocal`本身不会存储任何数据，`ThreadLocal`的`set`方法是将值存储到`Thread`线程本身的`ThreadLocalMap`里面了。
真正的项目开发往往会使用线程池，因此线程执行结束时需要清除上下文信息，需要调用`ThreadLocoal`的`remove`方法。

[threadlocal源码解读](https://github.com/wupeixuan/JDKSourceCode1.8/blob/master/src/java/lang/ThreadLocal.java)

#### 虚拟线程

Java 19引入的虚拟线程是为了解决IO密集型任务的吞吐量，它可以高效通过少数线程去调度大量虚拟线程；
虚拟线程在执行到IO操作或Blocking操作时，会自动切换到其他虚拟线程执行，从而避免当前线程等待，能最大化线程的执行效率。
...

### 函数式编程

#### FunctionalInterface

单抽象方法接口被称为`FunctionalInterface`,可用`@FunctionalInterface`来进行编译检查。
接收`FunctionalInterface`作为参数的时候，可以把实例化的匿名类改写为Lambda表达式；Lambda表达式的参数和返回值类型均可由编译器自动推断。
`FunctionalInterface`还可以传入：
符合方法签名的静态方法；
符合方法签名的实例方法（实例类型被看做第一个参数类型）；
符合方法签名的构造方法（实例类型被看做返回类型）。
注：上述方法传入时用使用方法引用。

#### 方法引用

例：`Main`类中静态方法cmp的引用，用`Main::cmp`表示

> You use lambda expressions to create anonymous methods. Sometimes, however, a lambda expression does nothing but call an existing method. In those cases, it’s often clearer to refer to the existing method by name. Method references enable you to do this; they are compact, easy-to-read lambda expressions for methods that already have a name.

#### Stream

...
### 网络编程
TCP/UDP/SMTP/http/RMI...
复习网络知识时可扩展...

### web开发

#### Servlet

每个`Servlet`类通过注解说明自己能处理的路径;
`HttpServlet`是`Servlet`的子类，通过实现`HttpServlet`子类的`doGet()`,`doPost()`方法处理get/post请求。
`Servlet`是单例类。
在`Servlet`中定义的实例变量会被多个线程同时访问，要注意线程安全；
`HttpServletRequest`和`HttpServletResponse`实例是由`Servlet`容器传入的局部变量，它们只能被当前线程访问；
在`doGet()`或`doPost()`方法中，如果使用了ThreadLocal，但没有清理，那么它的状态很可能会影响到下次的某个请求，因为`Servlet`容器很可能用线程池实现线程复用。

<details>
<summary>登录页面代码示例</summary>

```Java
@WebServlet(urlPatterns = "/signin")
public class SignInServlet extends HttpServlet {
    // 模拟一个数据库:
    private Map<String, String> users = Map.of("bob", "bob123", "alice", "alice123", "tom", "tomcat");

    // GET请求时显示登录页:
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.setContentType("text/html");
        PrintWriter pw = resp.getWriter();
        pw.write("<h1>Sign In</h1>");
        pw.write("<form action=\"/signin\" method=\"post\">");
        pw.write("<p>Username: <input name=\"username\"></p>");
        pw.write("<p>Password: <input name=\"password\" type=\"password\"></p>");
        pw.write("<p><button type=\"submit\">Sign In</button> <a href=\"/\">Cancel</a></p>");
        pw.write("</form>");
        pw.flush();
    }

    // POST请求时处理用户登录:
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String name = req.getParameter("username");
        String password = req.getParameter("password");
        String expectedPassword = users.get(name.toLowerCase());
        if (expectedPassword != null && expectedPassword.equals(password)) {
            // 登录成功:
            req.getSession().setAttribute("user", name);
            resp.sendRedirect("/");
        } else {
            resp.sendError(HttpServletResponse.SC_FORBIDDEN);
        }
    }
}

```
</details>

#### Servlet容器

> 无法在代码中直接通过new创建Servlet实例，必须由Servlet容器（如Tomcat）自动创建Servlet实例；
Servlet容器只会给每个Servlet类创建唯一实例；
Servlet容器会使用多线程执行doGet()或doPost()方法。
```Java
resources.addPreResources(
        new DirResourceSet(resources, "/WEB-INF/classes", new File("target/classes").getAbsolutePath(), "/"));
```
这段代码表示Tomcat会去target/classes目录下去寻找需要加载的`Servlet`类文件

#### Cookie和Session

##### Cookie

通过`HttpServletResponse`设置`cookie`以后，浏览器会在有效期内访问生效路径时带上`cookie`。
浏览器请求的RequestHeaders的Cookie部分如下：
<img width="620" height="124" alt="Image" src="https://github.com/user-attachments/assets/90beceb8-aec4-4a40-ac19-2ec77b4c28e1" />


<details>
<summary>cookie设置</summary>

```Java
@WebServlet(urlPatterns = "/pref")
public class LanguageServlet extends HttpServlet {

    private static final Set<String> LANGUAGES = Set.of("en", "zh");

    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String lang = req.getParameter("lang");
        if (LANGUAGES.contains(lang)) {
            // 创建一个新的Cookie:
            Cookie cookie = new Cookie("lang", lang);
            // 该Cookie生效的路径范围:
            cookie.setPath("/");
            // 该Cookie有效期:
            cookie.setMaxAge(8640000); // 8640000秒=100天
            // 将该Cookie添加到响应:
            resp.addCookie(cookie);
        }
        resp.sendRedirect("/");
    }
}
```
</details>

可以通过`req.getCookies()`获取到请求的Cookie信息。

##### Session
http是无状态协议，但是可以通过建立`session`来保存连接的状态。
可以通过```session=req.getSession()```获取当前会话，调用`session.setAttribute()`和`session.getAttribute()`修改和获取当前会话的属性。
通过Cookie来实现Session机制：
Servelet容器在第一次调用`getSession()`时会创建`Session`；
Servlet容器会自动创建`JSESSIONID`送给浏览器，浏览器请求时会带上`JSESSIONID`;

<details>
<summary>req.getSession()源码（具体实现在request.class)</summary>

```Java
public class Request implements HttpServletRequest {
    ...
    public HttpSession getSession(boolean create) {
        Session session = this.doGetSession(create);
        return session == null ? null : session.getSession();
    }
    ...
    protected Session doGetSession(boolean create) {
        Context context = this.getContext();
        if (context == null) {
            return null;
        } else {
            if (this.session != null && !this.session.isValid()) {
                this.session = null;
            }

            if (this.session != null) {
                return this.session;
            } else {
                Manager manager = context.getManager();
                if (manager == null) {
                    return null;
                } else {
                    if (this.requestedSessionId != null) {
                        try {
                            this.session = manager.findSession(this.requestedSessionId);
                        } catch (IOException e) {
                            if (log.isDebugEnabled()) {
                                log.debug(sm.getString("request.session.failed", new Object[]{this.requestedSessionId, e.getMessage()}), e);
                            } else {
                                log.info(sm.getString("request.session.failed", new Object[]{this.requestedSessionId, e.getMessage()}));
                            }

                            this.session = null;
                        }

                        if (this.session != null && !this.session.isValid()) {
                            this.session = null;
                        }

                        if (this.session != null) {
                            this.session.access();
                            return this.session;
                        }
                    }

                    if (!create) {
                        return null;
                    } else {
                        boolean trackModesIncludesCookie = context.getServletContext().getEffectiveSessionTrackingModes().contains(SessionTrackingMode.COOKIE);
                        if (trackModesIncludesCookie && this.response.getResponse().isCommitted()) {
                            throw new IllegalStateException(sm.getString("coyoteRequest.sessionCreateCommitted"));
                        } else {
                            String sessionId = this.getRequestedSessionId();
                            if (!this.requestedSessionSSL) {
                                if ("/".equals(context.getSessionCookiePath()) && this.isRequestedSessionIdFromCookie()) {
                                    if (context.getValidateClientProvidedNewSessionId()) {
                                        boolean found = false;

                                        for(Container container : this.getHost().findChildren()) {
                                            Manager m = ((Context)container).getManager();
                                            if (m != null) {
                                                try {
                                                    if (m.findSession(sessionId) != null) {
                                                        found = true;
                                                        break;
                                                    }
                                                } catch (IOException var13) {
                                                }
                                            }
                                        }

                                        if (!found) {
                                            sessionId = null;
                                        }
                                    }
                                } else {
                                    sessionId = null;
                                }
                            }

                            this.session = manager.createSession(sessionId);
                            if (this.session != null && trackModesIncludesCookie) {
                                Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(context, this.session.getIdInternal(), this.isSecure());
                                this.response.addSessionCookieInternal(cookie);
                            }

                            if (this.session == null) {
                                return null;
                            } else {
                                this.session.access();
                                return this.session;
                            }
                        }
                    }
                }
            }
        }
    }
    ...
}
```
</details>

通过源码可以发现：
`request`类的`doGetSession()`会首先试图访问缓存的`this.session`；
如果找不到则会通过`manager`寻找请求的`requestedSessionId`寻找对应的`Session`；
如果依旧找不到并且`create`参数为真，会根据请求的`sessionId`创建对应`Session`（请求的`sessionId`可能为空）；
创建完成`Session`以后会在返回时设置浏览器cookie的`SessionId`（对应代码如下）。
```Java
Cookie cookie = ApplicationSessionCookieConfig.createSessionCookie(context, this.session.getIdInternal(), this.isSecure());
this.response.addSessionCookieInternal(cookie);
```

### 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)

[菜鸟教程](https://www.runoob.com/java/java-tutorial.html)