## 目录

- [一、抽象类](#-----)
    + [注意：](#---)
- [二、接口](#----)
- [三、final 修饰符](#--final----)
  * [1. final 成员变量和局部变量：](#1-final-----------)
  * [2. 可执行“宏替换”的final变量](#2----------final--)
    + [例子：](#---)
  * [3. final修饰的方法](#3-final-----)
  * [4. 继承注意点：](#4-------)
- [四、this 关键字](#--this----)





# 一、抽象类

抽象方法是只有方法签名，没有方法实现的方法。

**抽象方法和抽象类必须使用 abstract 修饰符来定义，有抽象方法的类只能被定义成抽象类，抽象类里可以没有抽象方法。**
抽象方法和抽象类的规则如下：

* 抽象类必须使用abstract修饰符来修饰，抽象方法也必须使用abstract修饰符来修饰，抽象方法不能有方法体。

~~~java
public abstract class shape
{
    private String color; 
    //有抽象方法，所以必须定义为抽象类
    public abstract double Test();//无方法体
}

~~~



* 抽象类不能被实例化，无法使用new关键字来调用抽象类的构造器创建抽象类的实例。即使抽象类里不包含抽象方法，这个抽象类也不能创建实例。



* 抽象类可以包含成员变量、方法(普通方法和抽象方法都可以)、构造器、初始化块、内部类(接口、枚举) 5种成分。抽象类的构造器不能用于创建实例，主要是用于被其子类调用。

  

* 含有抽象方法的类(三种情况)只能被定义成抽象类。

   * 直接定义了一个抽象方法;

   * 继承了一个抽象父类，但没有完全实现父类包含的抽象方法;

     ~~~java
     public class Circle extends shape
     {
         private double radius;
         public Circle(String color, double radius)
         {
             super(color);
             this.radius = radius;
         }
         //Circle是普通类，所以必须实现父类的抽象方法
         public double Test()
         {
             return 2*radius
         }//有方法体
         
         public static void main(){
             shape s = new Circle("黄色", 2);
         }
     
     }
     ~~~

     

  	* 实现了一个接口，但没有完全实现接口包含的抽象方法



### 注意：

（private、static、final和abstract）

* abstract关键字修饰的方法必须被其子类重写才有意义，否则这个方法将永远不会有
  方法体，因此abstract方法不能定义为private 访问权限，**即private 和abstract不能同时修饰方法。**



* final 修饰的类不能被继承，final 修饰的方法不能被重写；**因此final和abstract永远不能同时使用。**





# 二、接口

由于接口定义的是一种规范，因此接口里不能包含构造器和初始化块定义。

**接口是从多个相似类中抽象出来的规范，接口不提供任何实现。接口体现的是规范和实现分离的设计哲学。**

接口定义了某一批类所需要遵守的规范，接口不关心这些类的内部状态数据，也不关心这些类里方法的实现细节，它只规定这批类里必须提供某些方法，提供这些方法的类就可满足实际需要。




接口里可以包含哪些成员：

* 成员变量 (只能是静态常量)；

  * 系统自动添加，默认使用 `public static final` 修饰；

    ~~~java
    //下面两行代码结果是一样的
    int test = 50;
    public static final int test = 50;
    ~~~
    
    

* 方法；

  * 普通方法：默认使用 `public abstract` 修饰

  * 类方法：默认使用 `static` 修饰

  * 默认方法：默认使用 `default` 修饰

  * 接口里的 **普通方法** 不能有方法体；但 **类方法、默认方法** 都必须有方法体；

    

* 内部类(包括内部接口、枚举)定义；

  * 接口里定义的内部类、内部接口、内部枚举默认都采用public static 两个修饰符，不管定义时是否指定这两个修饰符，系统都会自动使用public static对它们进行修饰。



~~~java
public interface Output
{
    //接口里的成员变量只能是常量
    int a = 50;
    
    //(1)普通方法没有方法体
    void out();
    
    //(2)默认方法，需要default修饰
    default void test(){
        //方法体
    }
    
    //(3)类方法，需要static修饰
    static String staticTest(){
        //方法体
    }
}
~~~



# 三、final 修饰符

final 修饰变量时，表示该变量一旦获得 了初始值就不可被改变，final 既可以修饰成员变量(包括类变量和实例变量)，也可以修饰局部变量、形参。

当创建对象时，系统会为该对象的实例变量分配内存，并分配默认值。也就是说：

1. **当执行静态初始化块时可以对类变量赋初始值；**

2. **当执行普通初始化块、构造器时可对实例变量赋初始值。**





~~~java
public class test()
{
    //合法
    final int a = 6;
    
    //合法，实例变量在构造器或初始化块中分配初始化值了
    final String str;
    final int c;
    
    //合法，实例变量静态初始化块中分配初始化值了
    final static double d;
    
    //非法,既没有默认值也没有在初始化块、构造器中指定初始值
    final char ch;
    
    //初始化块
    {
        str = "hello";
        
        //非法，实例变量a已经有了默认值
        a = 9;
    }
    
    //静态初始化块
    static
    {
        d = 3.14;
    }
    
    //构造器
    public test()
    {
        c = 5;
    }
}
~~~

**注意：** 与普通成员变量不同的是，final 成员变量是必须由程序员显
式初始化，系统不会对 final 成员进行隐式初始化。

还有需要注意的是，如果打算在构造器、初始化块中对 final 成员变量进行初始化，则不要在初始化之前就访问成员变量的值。例如下面程序将会引起错误

~~~java
final int age;

{
    //报错,因为age还没有初始化
    System.out.println(age);
    age = 6;
}
~~~



## 1. final 成员变量和局部变量：

* 对于final 修饰的成员变量而言，如果既没有在定义成员变量时指定初始值，也没有在初始化块、构造器中为成员变量指定初始值，那么这些成员变量的值将一直是系统默认分配的 0、\u0000'、 false 或 null；

* final 修饰的局部变量在定义时没有指定默认值，则可以在后面代码中对该 final 变量赋初始值，但只能一次；

~~~java
public class test()
{
    //不赋值则失去了存在意义
    final int a;
    
    public void test()
    {
        final int a;
        //可以对没指定默认值局部变量赋值一次
        a = 5;
    }
}
~~~





## 2. 可执行 “宏替换” 的 final 变量

对一个final 变量来说，不管它是类变量、实例变量，还是局部变量，只要该变量满足三个条件，这个 final 变量就不再是一个变量，而是相当于一个直接量。

* 使用 final 修饰符修饰；
* 在定义该 final 变量时指定了初始值；
* 该初始值可以在编译时就被确定下来；
  

~~~java
public class test
{
    public static void main()
    {
        final int a = 5;
        System.out.println(a);
    }
}
~~~

对于这个程序而言，变量 a 根本不存在，当程序执行 System.out.println(a) 时，实际转换为执行System.out.println(5)。



final 修饰符的一个重要用途就是定义“宏变量”。当定义 final 变量时就为该变量指定了初始值，而且该初始值可以在编译时就确定下来，那么这个final 变量本质上就是一个 **“宏变量”，编译器会把程序中所有用到该变量的地方直接替换成该变量的值。** 

**如下 “宏变量” 例子：** 

~~~java
public class test
{
    public static void main()
    {
        String s1 = "hello world";
        //s2变量引用的字符串可以在编译器就确定下来
        String s2 = "hello" + "world";
        System.out.println(s1==s2);
        
        String str1 = "hello";
        String str2 = "world";
        String s3 = str1 + str2;
        System.out.println(s1==s3);
    }
}
~~~


Java 会使用常量池来管理曾经用过的字符串直接量；所以 s2 直接指向引用常量池中已经有的 “hello world” 字符串，即 s1==s2；

对于 s3 而言，它的值由 str1 和 str2 进行连接运算后得到。 **由于 str1、str2 只是两个普通变量，编译器不会执行“宏替换”，因此编译器无法在编译时确定s3的值** ，也就无法让 s3 指向字符串池中缓存的 “hello world” 。即s1==s3 将输出 false。

**将 str1、str2 用 final 修饰使得其执行 ”宏替换“，那么替换后 s3 的值就可以在编译器确定下来，即 s1==s3；**

~~~java
String s1 = "hello world";

final String str1 = "hello";
final String str2 = "world";
String s3 = str1 + str2;

System.out.println(s1==s3);

运行结果:
true
~~~





## 3. final修饰的方法

final 修饰的方法不可被重写，如果出于某些原因，不希望子类重写父类的某个方法，则可以使用 final 修饰该方法。



## 4. 继承注意点：

为了保证父类有良好的封装性，不会被子类随意改变，设计父类通常应该遵循如下规则：

* 尽量隐藏父类的内部数据。尽量把父类的所有成员变量都设置成 private 访问类型，不要让子类直接访问父类的成员变量。

  

* 不要让子类可以随意访问、修改父类的方法。父类中那些仅为辅助其他的工具方法，应该使用 private 访问控制符修饰，让子类无法访问该方法；如果父类中的方法需要被外部类调用，则必须以 public 修饰，但又不希望子类重写该方法，可以使用 final 修饰符来修饰该方法；如果希望父类的某个方法被子类重写，但不希望被其他类自由访问，则可以使用 protected 来修饰该方法。

  

* 尽量不要在父类构造器中调用将要被子类重写的方法。



# 四、this 关键字

**- this 默认的两种情形：**

* 构造器中引用该构造器正在初始化的对象。
* 在方法中引用调用该方法的对象。

~~~java
public class Dog
{
    public void jump(){}
    
    public void run()
    {
        Dog d = new Dog();
        d.jump();
    }
}
~~~



这里产生了两个问题............................

**1. 第一个问题：在 run() 方法中调用 jump()  方法时是否一定需要一个 Dog对象？**

答案是肯定的，因为没有使用 static 修饰的成员变量和方法都必须使用对象来调用。

**2.第二个问题：是否一定需要重新创建一个 Dog 对象?** 

答案是否定的，因为当程序调用 run() 方法时，一定会提供 一个Dog对象，（比如main方法里就提供了一个 Dog对象）这样就可以直接使用这个已经存在的Dog对象，而无须重新创建新的Dog对象了。



~~~java
public void run()
{
    //使用this引用调用run()方法的对象
    this.jump();
}
~~~

采用上面方法定义的 Dog 类更符合实际意义。从前一种 Dog 类定义来看，在 Dog 对象的 run() 方法内重新创建了一个新的Dog对象，并调用它的 jump() 方法，这意味着一个 Dog 对象的 run() 方法需要依赖于另一个Dog 对象的 jump() 方法，这不符合逻辑。

上面的代码更符合实际情形：当一个 Dog 对象调用 run() 方法时， run() 方法需要依赖它自己的 jump() 方法。

在现实世界里，对象的一个方法依赖于另一个方法的情形如此常见：例如，吃饭方法依赖于拿筷子方法，写程序方法依赖于敲键盘方法，**这种依赖都是同一个对象两个方法之间的依赖。** 因此，Java 允许对象的一个成员直接调用另一个成员，可以省略this前缀。也就是说，将上面的run()方法改为如下形
式也宗全正确。

~~~java
public void run()
{
    jump();
}
~~~



**大部分时候，一个方法访问该类中定义的其他方法、成员变量时加不加 this 前缀的效果是完全一样的。**

