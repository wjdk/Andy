# Java学习笔记
## 语法

### 基本语法

- 数据类型

byte: -128~127
布尔类型boolean
数组类型int[]
`int[] ns = new int[5];`
`int[] ns = new int[] { 68, 79, 91, 85, 62 };`

- var关键字

类似C++的auto自动定义变量
`var sb = new StringBuilder();`
实际上会自动变成：
`StringBuilder sb = new StringBuilder();`

- 输出 
  
```
double d = 3.1415926;
System.out.printf("%.2f\n", d);
```

- 输入
  
```
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

### 面向对象

#### 继承
  
继承关键字：`extends `
在Java中，没有明确写extends的类，编译器会自动加上`extends Object`。
Java只允许一个class继承自一个类。
`super`关键字表示父类（超类）。子类引用父类的字段时，可以用`super.fieldName`
子类构造函数调用父类的构造函数：
```
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
```
abstract class Person {
    public abstract void run();
}
```

#### interface（接口）

如果一个抽象类没有字段，所有方法全部都是抽象方法：
就可以把该抽象类改写为接口：`interface`
Java的接口特指interface的定义，表示一个接口类型和一组方法签名，而编程接口泛指接口规范，如方法签名，数据格式，网络协议等。
一个类可以实现多个`interface`
```
class Student implements Person, Hello { // 实现了两个interface
    ...
}
```

#### 包作用域
package
包作用域是指一个类允许访问同一个package的没有public、private修饰的class，以及没有public、protected、private修饰的字段和方法。

#### final

用final修饰class可以阻止被继承：
```
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
```
package abc;

public class Hello {
    // 无法被覆写:
    protected final void hi() {
    }
}
```
用final修饰field可以阻止被重新赋值：

```
package abc;

public class Hello {
    private final int n = 0;
    protected void hi() {
        this.n = 1; // error!
    }
}
```
用final修饰局部变量可以阻止被重新赋值：
```
package abc;

public class Hello {
    protected void hi(final int t) {
        t = 1; // error!
    }
}
```

#### 内部类inner Class
```
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

#### Java的编译运行过程：

class,classpath,模块等见[参考资料3.12-3.14](https://liaoxuefeng.com/books/java/oop/basic/classpath-jar/index.html)

[JRE与JDK的区别](https://www.runoob.com/w3cnote/the-different-of-jre-and-jdk.html)

### JVM


### 参考资料

[廖雪峰的Java教程](https://liaoxuefeng.com/books/java/introduction/index.html)