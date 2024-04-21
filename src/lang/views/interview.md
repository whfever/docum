# Java 全栈知识点问题汇总（上）

## [#](#_1-java-基础) 1 Java 基础

> Java基础部分，包括语法基础，泛型，注解，异常，反射和其它（如SPI机制等）。

### [#](#_1-1-语法基础) 1.1 语法基础

#### [#](#面向对象特性) 面向对象特性？

- **封装**

利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个不可分割的独立实体。数据被保护在抽象数据类型的内部，尽可能地隐藏内部的细节，只保留一些对外接口使之与外部发生联系。用户无需知道对象内部的细节，但可以通过对象对外提供的接口来访问该对象。

优点:

- 减少耦合: 可以独立地开发、测试、优化、使用、理解和修改
- 减轻维护的负担: 可以更容易被程序员理解，并且在调试的时候可以不影响其他模块
- 有效地调节性能: 可以通过剖析确定哪些模块影响了系统的性能
- 提高软件的可重用性
- 降低了构建大型系统的风险: 即使整个系统不可用，但是这些独立的模块却有可能是可用的

以下 Person 类封装 name、gender、age 等属性，外界只能通过 get() 方法获取一个 Person 对象的 name 属性和 gender 属性，而无法获取 age 属性，但是 age 属性可以供 work() 方法使用。

注意到 gender 属性使用 int 数据类型进行存储，封装使得用户注意不到这种实现细节。并且在需要修改 gender 属性使用的数据类型时，也可以在不影响客户端代码的情况下进行。

```java
public class Person {

    private String name;
    private int gender;
    private int age;

    public String getName() {
        return name;
    }

    public String getGender() {
        return gender == 0 ? "man" : "woman";
    }

    public void work() {
        if (18 <= age && age <= 50) {
            System.out.println(name + " is working very hard!");
        } else {
            System.out.println(name + " can't work any more!");
        }
    }
}
```

- **继承**

继承实现了 **IS-A** 关系，例如 Cat 和 Animal 就是一种 IS-A 关系，因此 Cat 可以继承自 Animal，从而获得 Animal 非 private 的属性和方法。

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

Cat 可以当做 Animal 来使用，也就是说可以使用 Animal 引用 Cat 对象。父类引用指向子类对象称为 **向上转型** 。

```java
Animal animal = new Cat();
```

- **多态**

多态分为编译时多态和运行时多态:

- 编译时多态主要指方法的重载
- 运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定

运行时多态有三个条件:

- 继承
- 覆盖(重写)
- 向上转型

下面的代码中，乐器类(Instrument)有两个子类: Wind 和 Percussion，它们都覆盖了父类的 play() 方法，并且在 main() 方法中使用父类 Instrument 来引用 Wind 和 Percussion 对象。在 Instrument 引用调用 play() 方法时，会执行实际引用对象所在类的 play() 方法，而不是 Instrument 类的方法。

```java
public class Instrument {
    public void play() {
        System.out.println("Instrument is playing...");
    }
}

public class Wind extends Instrument {
    public void play() {
        System.out.println("Wind is playing...");
    }
}

public class Percussion extends Instrument {
    public void play() {
        System.out.println("Percussion is playing...");
    }
}

public class Music {
    public static void main(String[] args) {
        List<Instrument> instruments = new ArrayList<>();
        instruments.add(new Wind());
        instruments.add(new Percussion());
        for(Instrument instrument : instruments) {
            instrument.play();
        }
    }
}
```

#### [#](#a-a-b-与-a-b-的区别) a = a + b 与 a += b 的区别

+= 隐式的将加操作的结果类型强制转换为持有结果的类型。如果两个整型相加，如 byte、short 或者 int，首先会将它们提升到 int 类型，然后在执行加法操作。

```java
byte a = 127;
byte b = 127;
b = a + b; // error : cannot convert from int to byte
b += a; // ok
```

(因为 a+b 操作会将 a、b 提升为 int 类型，所以将 int 类型赋值给 byte 就会编译出错)

#### [#](#_3-0-1-0-3-将会返回什么-true-还是-false) 3*0.1 == 0.3 将会返回什么? true 还是 false?

false，因为有些浮点数不能完全精确的表示出来。

#### [#](#能在-switch-中使用-string-吗) 能在 Switch 中使用 String 吗?

从 Java 7 开始，我们可以在 switch case 中使用字符串，但这仅仅是一个语法糖。内部实现在 switch 中使用字符串的 hash code。

#### [#](#对equals-和hashcode-的理解) 对equals()和hashCode()的理解?

- **为什么在重写 equals 方法的时候需要重写 hashCode 方法**?

因为有强制的规范指定需要同时重写 hashcode 与 equals 是方法，许多容器类，如 HashMap、HashSet 都依赖于 hashcode 与 equals 的规定。

- **有没有可能两个不相等的对象有相同的 hashcode**?

有可能，两个不相等的对象可能会有相同的 hashcode 值，这就是为什么在 hashmap 中会有冲突。相等 hashcode 值的规定只是说如果两个对象相等，必须有相同的hashcode 值，但是没有关于不相等对象的任何规定。

- **两个相同的对象会有不同的 hash code 吗**?

不能，根据 hash code 的规定，这是不可能的。

#### [#](#final、finalize-和-finally-的不同之处) final、finalize 和 finally 的不同之处?

- final 是一个修饰符，可以修饰变量、方法和类。如果 final 修饰变量，意味着该变量的值在初始化后不能被改变。
- Java 技术允许使用 finalize() 方法在垃圾收集器将对象从内存中清除出去之前做必要的清理工作。这个方法是由垃圾收集器在确定这个对象没有被引用时对这个对象调用的，但是什么时候调用 finalize 没有保证。
- finally 是一个关键字，与 try 和 catch 一起用于异常的处理。finally 块一定会被执行，无论在 try 块中是否有发生异常。

#### [#](#string、stringbuffer与stringbuilder的区别) String、StringBuffer与StringBuilder的区别？

第一点: 可变和适用范围。String对象是不可变的，而StringBuffer和StringBuilder是可变字符序列。每次对String的操作相当于生成一个新的String对象，而对StringBuffer和StringBuilder的操作是对对象本身的操作，而不会生成新的对象，所以对于频繁改变内容的字符串避免使用String，因为频繁的生成对象将会对系统性能产生影响。

第二点: 线程安全。String由于有final修饰，是immutable的，安全性是简单而纯粹的。StringBuilder和StringBuffer的区别在于StringBuilder不保证同步，也就是说如果需要线程安全需要使用StringBuffer，不需要同步的StringBuilder效率更高。

#### [#](#接口与抽象类的区别) 接口与抽象类的区别？

- 一个子类只能继承一个抽象类, 但能实现多个接口
- 抽象类可以有构造方法, 接口没有构造方法
- 抽象类可以有普通成员变量, 接口没有普通成员变量
- 抽象类和接口都可有静态成员变量, 抽象类中静态成员变量访问类型任意，接口只能public static final(默认)
- 抽象类可以没有抽象方法, 抽象类可以有普通方法；接口在JDK8之前都是抽象方法，在JDK8可以有default方法，在JDK9中允许有私有普通方法
- 抽象类可以有静态方法；接口在JDK8之前不能有静态方法，在JDK8中可以有静态方法，且只能被接口类直接调用（不能被实现类的对象调用）
- 抽象类中的方法可以是public、protected; 接口方法在JDK8之前只有public abstract，在JDK8可以有default方法，在JDK9中允许有private方法

#### [#](#this-super-在构造方法中的区别) this() & super()在构造方法中的区别？

- 调用super()必须写在子类构造方法的第一行, 否则编译不通过
- super从子类调用父类构造, this在同一类中调用其他构造均需要放在第一行
- 尽管可以用this调用一个构造器, 却不能调用2个
- this和super不能出现在同一个构造器中, 否则编译不通过
- this()、super()都指的对象,不可以在static环境中使用
- 本质this指向本对象的指针。super是一个关键字

#### [#](#java移位运算符) Java移位运算符？

java中有三种移位运算符

- `<<` :左移运算符,`x << 1`,相当于x乘以2(不溢出的情况下),低位补0
- `>>` :带符号右移,`x >> 1`,相当于x除以2,正数高位补0,负数高位补1
- `>>>` :无符号右移,忽略符号位,空位都以0补齐

### [#](#_1-2-泛型) 1.2 泛型

> 在编译时，Java编译器会将泛型类型擦除为其上界或Object类型，并在需要的地方插入强制类型转换。这种类型擦除的机制导致无法直接使用基本类型作为泛型参数

#### [#](#为什么需要泛型) 为什么需要泛型？

1. **适用于多种数据类型执行相同的代码**

```java
private static int add(int a, int b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static float add(float a, float b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}

private static double add(double a, double b) {
    System.out.println(a + "+" + b + "=" + (a + b));
    return a + b;
}
```

如果没有泛型，要实现不同类型的加法，每种类型都需要重载一个add方法；通过泛型，我们可以复用为一个方法：

```java
private static <T extends Number> double add(T a, T b) {
    System.out.println(a + "+" + b + "=" + (a.doubleValue() + b.doubleValue()));
    return a.doubleValue() + b.doubleValue();
}
```

- **泛型中的类型在使用时指定，不需要强制类型转换**（**类型安全**，编译器会**检查类型**）

看下这个例子：

```java
List list = new ArrayList();
list.add("xxString");
list.add(100d);
list.add(new Person());
```

我们在使用上述list中，list中的元素都是Object类型（无法约束其中的类型），所以在取出集合元素时需要人为的强制类型转化到具体的目标类型，且很容易出现`java.lang.ClassCastException`异常。

引入泛型，它将提供类型的约束，提供编译前的检查：

```java
List<String> list = new ArrayList<String>();

// list中只能放String, 不能放其它类型的元素
```

#### [#](#泛型类如何定义使用) 泛型类如何定义使用？

- 从一个简单的泛型类看起：

```java
class Point<T>{         // 此处可以随便写标识符号，T是type的简称  
    private T var ;     // var的类型由T指定，即：由外部指定  
    public T getVar(){  // 返回值的类型由外部决定  
        return var ;  
    }  
    public void setVar(T var){  // 设置的类型也由外部决定  
        this.var = var ;  
    }  
}  
public class GenericsDemo06{  
    public static void main(String args[]){  
        Point<String> p = new Point<String>() ;     // 里面的var类型为String类型  
        p.setVar("it") ;                            // 设置字符串  
        System.out.println(p.getVar().length()) ;   // 取得字符串的长度  
    }  
}
```

- 多元泛型

```java
class Notepad<K,V>{       // 此处指定了两个泛型类型  
    private K key ;     // 此变量的类型由外部决定  
    private V value ;   // 此变量的类型由外部决定  
    public K getKey(){  
        return this.key ;  
    }  
    public V getValue(){  
        return this.value ;  
    }  
    public void setKey(K key){  
        this.key = key ;  
    }  
    public void setValue(V value){  
        this.value = value ;  
    }  
} 
public class GenericsDemo09{  
    public static void main(String args[]){  
        Notepad<String,Integer> t = null ;        // 定义两个泛型类型的对象  
        t = new Notepad<String,Integer>() ;       // 里面的key为String，value为Integer  
        t.setKey("汤姆") ;        // 设置第一个内容  
        t.setValue(20) ;            // 设置第二个内容  
        System.out.print("姓名；" + t.getKey()) ;      // 取得信息  
        System.out.print("，年龄；" + t.getValue()) ;       // 取得信息  
  
    }  
}
```

#### [#](#泛型接口如何定义使用) 泛型接口如何定义使用？

- 简单的泛型接口

```java
interface Info<T>{        // 在接口上定义泛型  
    public T getVar() ; // 定义抽象方法，抽象方法的返回值就是泛型类型  
}  
class InfoImpl<T> implements Info<T>{   // 定义泛型接口的子类  
    private T var ;             // 定义属性  
    public InfoImpl(T var){     // 通过构造方法设置属性内容  
        this.setVar(var) ;    
    }  
    public void setVar(T var){  
        this.var = var ;  
    }  
    public T getVar(){  
        return this.var ;  
    }  
} 
public class GenericsDemo24{  
    public static void main(String arsg[]){  
        Info<String> i = null;        // 声明接口对象  
        i = new InfoImpl<String>("汤姆") ;  // 通过子类实例化对象  
        System.out.println("内容：" + i.getVar()) ;  
    }  
}  
```

#### [#](#泛型方法如何定义使用) 泛型方法如何定义使用？

泛型方法，是在调用方法的时候指明泛型的具体类型。

- 定义泛型方法语法格式

![img](/images/java/java-basic-generic-4.png)

- 调用泛型方法语法格式

![img](/images/java/java-basic-generic-5.png)

说明一下，定义泛型方法时，必须在返回值前边加一个`<T>`，来声明这是一个泛型方法，持有一个泛型`T`，然后才可以用泛型T作为方法的返回值。

`Class<T>`的作用就是指明泛型的具体类型，而`Class<T>`类型的变量c，可以用来创建泛型类的对象。

为什么要用变量c来创建对象呢？既然是泛型方法，就代表着我们不知道具体的类型是什么，也不知道构造方法如何，因此没有办法去new一个对象，但可以利用变量c的newInstance方法去创建对象，也就是利用反射创建对象。

泛型方法要求的参数是`Class<T>`类型，而`Class.forName()`方法的返回值也是`Class<T>`，因此可以用`Class.forName()`作为参数。其中，`forName()`方法中的参数是何种类型，返回的`Class<T>`就是何种类型。在本例中，`forName()`方法中传入的是User类的完整路径，因此返回的是`Class<User>`类型的对象，因此调用泛型方法时，变量c的类型就是`Class<User>`，因此泛型方法中的泛型T就被指明为User，因此变量obj的类型为User。

当然，泛型方法不是仅仅可以有一个参数`Class<T>`，可以根据需要添加其他参数。

**为什么要使用泛型方法呢**？因为泛型类要在实例化的时候就指明类型，如果想换一种类型，不得不重新new一次，可能不够灵活；而泛型方法可以在调用的时候指明类型，更加灵活。

#### [#](#泛型的上限和下限) 泛型的上限和下限？

在使用泛型的时候，我们可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

上限

```java
class Info<T extends Number>{    // 此处泛型只能是数字类型
    private T var ;        // 定义泛型变量
    public void setVar(T var){
        this.var = var ;
    }
    public T getVar(){
        return this.var ;
    }
    public String toString(){    // 直接打印
        return this.var.toString() ;
    }
}
public class demo1{
    public static void main(String args[]){
        Info<Integer> i1 = new Info<Integer>() ;        // 声明Integer的泛型对象
    }
}
```

下限

```java
class Info<T>{
    private T var ;        // 定义泛型变量
    public void setVar(T var){
        this.var = var ;
    }
    public T getVar(){
        return this.var ;
    }
    public String toString(){    // 直接打印
        return this.var.toString() ;
    }
}
public class GenericsDemo21{
    public static void main(String args[]){
        Info<String> i1 = new Info<String>() ;        // 声明String的泛型对象
        Info<Object> i2 = new Info<Object>() ;        // 声明Object的泛型对象
        i1.setVar("hello") ;
        i2.setVar(new Object()) ;
        fun(i1) ;
        fun(i2) ;
    }
    public static void fun(Info<? super String> temp){    // 只能接收String或Object类型的泛型，String类的父类只有Object类
        System.out.print(temp + ", ") ;
    }
}
```

#### [#](#如何理解java中的泛型是伪泛型) 如何理解Java中的泛型是伪泛型？

泛型中类型擦除 Java泛型这个特性是从JDK 1.5才开始加入的，因此为了兼容之前的版本，Java泛型的实现采取了“伪泛型”的策略，即Java在语法上支持泛型，但是在编译阶段会进行所谓的“类型擦除”（Type Erasure），将所有的泛型表示（尖括号中的内容）都替换为具体的类型（其对应的原生态类型），就像完全没有泛型一样。

### [#](#_1-3-注解) 1.3 注解

#### [#](#注解的作用) 注解的作用？

注解是JDK1.5版本开始引入的一个特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。它主要的作用有以下四方面：

- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。

#### [#](#注解的常见分类) 注解的常见分类？

- **Java自带的标准注解**，包括`@Override`、`@Deprecated`和`@SuppressWarnings`，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告，用这些注解标明后编译器就会进行检查。

- 元注解

  ，元注解是用于定义注解的注解，包括

  ```
  @Retention
  ```

  、

  ```
  @Target
  ```

  、

  ```
  @Inherited
  ```

  、

  ```
  @Documented
  ```

  - `@Retention`用于标明注解被保留的阶段
  - `@Target`用于标明注解使用的范围
  - `@Inherited`用于标明注解可继承
  - `@Documented`用于标明是否生成javadoc文档

- **自定义注解**，可以根据自己的需求定义注解，并可用元注解对自定义注解进行注解。

### [#](#_1-4-异常) 1.4 异常

#### [#](#java异常类层次结构) Java异常类层次结构?

- Throwable

   是 Java 语言中所有错误与异常的超类。 

  - **Error** 类及其子类：程序中无法处理的错误，表示运行应用程序中出现了严重的错误。
  - **Exception** 程序本身可以捕获并且可以处理的异常。Exception 这种异常又分为两类：运行时异常和编译时异常。

![img](/images/java/java-basic-exception-1.png)

- **运行时异常**

都是RuntimeException类及其子类异常，如NullPointerException(空指针异常)、IndexOutOfBoundsException(下标越界异常)等，这些异常是不检查异常，程序中可以选择捕获处理，也可以不处理。这些异常一般是由程序逻辑错误引起的，程序应该从逻辑角度尽可能避免这类异常的发生。

运行时异常的特点是Java编译器不会检查它，也就是说，当程序中可能出现这类异常，即使没有用try-catch语句捕获它，也没有用throws子句声明抛出它，也会编译通过。

- **非运行时异常** （编译异常）

是RuntimeException以外的异常，类型上都属于Exception类及其子类。从程序语法角度讲是必须进行处理的异常，如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。

#### [#](#可查的异常-checked-exceptions-和不可查的异常-unchecked-exceptions-区别) 可查的异常（checked exceptions）和不可查的异常（unchecked exceptions）区别？

- **可查异常**（编译器要求必须处置的异常）：

正确的程序在运行中，很容易出现的、情理可容的异常状况。可查异常虽然是异常状况，但在一定程度上它的发生是可以预计的，而且一旦发生这种异常状况，就必须采取某种方式进行处理。

除了RuntimeException及其子类以外，其他的Exception类及其子类都属于可查异常。这种异常的特点是Java编译器会检查它，也就是说，当程序中可能出现这类异常，要么用try-catch语句捕获它，要么用throws子句声明抛出它，否则编译不会通过。

- **不可查异常**(编译器不要求强制处置的异常)

包括运行时异常（RuntimeException与其子类）和错误（Error）。

#### [#](#throw和throws的区别) throw和throws的区别？

- **异常的申明(throws)**

在Java中，当前执行的语句必属于某个方法，Java解释器调用main方法执行开始执行程序。若方法中存在检查异常，如果不对其捕获，那必须在方法头中显式声明该异常，以便于告知方法调用者此方法有异常，需要进行处理。 在方法中声明一个异常，方法头中使用关键字throws，后面接上要声明的异常。若声明多个异常，则使用逗号分割。如下所示：

```java
public static void method() throws IOException, FileNotFoundException{
    //something statements
}
```

- **异常的抛出(throw)**

如果代码可能会引发某种错误，可以创建一个合适的异常类实例并抛出它，这就是抛出异常。如下所示：

```java
public static double method(int value) {
    if(value == 0) {
        throw new ArithmeticException("参数不能为0"); //抛出一个运行时异常
    }
    return 5.0 / value;
}
```

#### [#](#java-7-的-try-with-resource) Java 7 的 try-with-resource?

如果你的资源实现了 AutoCloseable 接口，你可以使用这个语法。大多数的 Java 标准资源都继承了这个接口。当你在 try 子句中打开资源，资源会在 try 代码块执行后或异常处理后自动关闭。

```java
public void automaticallyCloseResource() {
    File file = new File("./tmp.txt");
    try (FileInputStream inputStream = new FileInputStream(file);) {
        // use the inputStream to read a file
    } catch (FileNotFoundException e) {
        log.error(e);
    } catch (IOException e) {
        log.error(e);
    }
}
```

#### [#](#异常的底层) 异常的底层？

提到JVM处理异常的机制，就需要提及Exception Table，以下称为异常表。我们暂且不急于介绍异常表，先看一个简单的 Java 处理异常的小例子。

```java
public static void simpleTryCatch() {
   try {
       testNPE();
   } catch (Exception e) {
       e.printStackTrace();
   }
}
```

使用javap来分析这段代码（需要先使用javac编译）

```java
//javap -c Main
 public static void simpleTryCatch();
    Code:
       0: invokestatic  #3                  // Method testNPE:()V
       3: goto          11
       6: astore_0
       7: aload_0
       8: invokevirtual #5                  // Method java/lang/Exception.printStackTrace:()V
      11: return
    Exception table:
       from    to  target type
           0     3     6   Class java/lang/Exception
```

看到上面的代码，应该会有会心一笑，因为终于看到了Exception table，也就是我们要研究的异常表。

异常表中包含了一个或多个异常处理者(Exception Handler)的信息，这些信息包含如下

- **from** 可能发生异常的起始点
- **to** 可能发生异常的结束点
- **target** 上述from和to之前发生异常后的异常处理者的位置
- **type** 异常处理者处理的异常的类信息

### [#](#_1-5-反射) 1.5 反射

#### [#](#什么是反射) 什么是反射？

JAVA反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为java语言的反射机制。

![img](/images/java/java-basic-reflection-3.png)

#### [#](#反射的使用) 反射的使用？

在Java中，Class类与java.lang.reflect类库一起对反射技术进行了全力的支持。在反射包中，我们常用的类主要有Constructor类表示的是Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象、Field表示Class对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值(包含private)、Method表示Class对象所表示的类的成员方法，通过它可以动态调用对象的方法(包含private)

- Class类对象的获取

```java
    @Test
    public void classTest() throws Exception {
        // 获取Class对象的三种方式
        logger.info("根据类名:  \t" + User.class);
        logger.info("根据对象:  \t" + new User().getClass());
        logger.info("根据全限定类名:\t" + Class.forName("com.test.User"));
        // 常用的方法
        logger.info("获取全限定类名:\t" + userClass.getName());
        logger.info("获取类名:\t" + userClass.getSimpleName());
        logger.info("实例化:\t" + userClass.newInstance());
    }
```

- Constructor类及其用法
- Field类及其用法
- Method类及其用法

#### [#](#getname、getcanonicalname与getsimplename的区别) getName、getCanonicalName与getSimpleName的区别?

- getSimpleName：只获取类名
- getName：类的全限定名，jvm中Class的表示，可以用于动态加载Class对象，例如Class.forName。
- getCanonicalName：返回更容易理解的表示，主要用于输出（toString）或log打印，大多数情况下和getName一样，但是在内部类、数组等类型的表示形式就不同了。

### [#](#_1-6-spi机制) 1.6 SPI机制

#### [#](#什么是spi机制) 什么是SPI机制？

SPI（Service Provider Interface），是JDK内置的一种 服务提供发现机制，可以用来启用框架扩展和替换组件，主要是被框架的开发人员使用，比如java.sql.Driver接口，其他不同厂商可以针对同一接口做出不同的实现，MySQL和PostgreSQL都有不同的实现提供给用户，而Java的SPI机制可以为某个接口寻找服务实现。Java中SPI机制主要思想是将装配的控制权移到程序之外，在模块化设计中这个机制尤其重要，其核心思想就是 **解耦**。

SPI整体机制图如下：

![img](/images/java/java-advanced-spi-8.jpg)

当服务的提供者提供了一种接口的实现之后，需要在classpath下的`META-INF/services/`目录里创建一个以服务接口命名的文件，这个文件里的内容就是这个接口的具体的实现类。当其他的程序需要这个服务的时候，就可以通过查找这个jar包（一般都是以jar包做依赖）的`META-INF/services/`中的配置文件，配置文件中有接口的具体实现类名，可以根据这个类名进行加载实例化，就可以使用该服务了。JDK中查找服务的实现的工具类是：`java.util.ServiceLoader`。

#### [#](#spi机制的应用) SPI机制的应用？

- SPI机制 - JDBC DriverManager

在JDBC4.0之前，我们开发有连接数据库的时候，通常会用Class.forName("com.mysql.jdbc.Driver")这句先加载数据库相关的驱动，然后再进行获取连接等的操作。**而JDBC4.0之后不需要用Class.forName("com.mysql.jdbc.Driver")来加载驱动，直接获取连接就可以了，现在这种方式就是使用了Java的SPI扩展机制来实现**。

- JDBC接口定义

首先在java中定义了接口`java.sql.Driver`，并没有具体的实现，具体的实现都是由不同厂商来提供的。

- mysql实现

在mysql的jar包`mysql-connector-java-6.0.6.jar`中，可以找到`META-INF/services`目录，该目录下会有一个名字为`java.sql.Driver`的文件，文件内容是`com.mysql.cj.jdbc.Driver`，这里面的内容就是针对Java中定义的接口的实现。

- postgresql实现

同样在postgresql的jar包`postgresql-42.0.0.jar`中，也可以找到同样的配置文件，文件内容是`org.postgresql.Driver`，这是postgresql对Java的`java.sql.Driver`的实现。

- 使用方法

上面说了，现在使用SPI扩展来加载具体的驱动，我们在Java中写连接数据库的代码的时候，不需要再使用`Class.forName("com.mysql.jdbc.Driver")`来加载驱动了，而是直接使用如下代码：

```java
String url = "jdbc:xxxx://xxxx:xxxx/xxxx";
Connection conn = DriverManager.getConnection(url,username,password);
.....
```

#### [#](#spi机制的简单示例) SPI机制的简单示例？

我们现在需要使用一个内容搜索接口，搜索的实现可能是基于文件系统的搜索，也可能是基于数据库的搜索。

- 先定义好接口

```java
public interface Search {
    public List<String> searchDoc(String keyword);   
}
```

- 文件搜索实现

```java
public class FileSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("文件搜索 "+keyword);
        return null;
    }
}
```

- 数据库搜索实现

```java
public class DatabaseSearch implements Search{
    @Override
    public List<String> searchDoc(String keyword) {
        System.out.println("数据搜索 "+keyword);
        return null;
    }
}
```

- resources 接下来可以在resources下新建META-INF/services/目录，然后新建接口全限定名的文件：`com.cainiao.ys.spi.learn.Search`，里面加上我们需要用到的实现类

```xml
com.cainiao.ys.spi.learn.FileSearch
```

- 测试方法

```java
public class TestCase {
    public static void main(String[] args) {
        ServiceLoader<Search> s = ServiceLoader.load(Search.class);
        Iterator<Search> iterator = s.iterator();
        while (iterator.hasNext()) {
           Search search =  iterator.next();
           search.searchDoc("hello world");
        }
    }
}
```

可以看到输出结果：文件搜索 hello world

如果在`com.cainiao.ys.spi.learn.Search`文件里写上两个实现类，那最后的输出结果就是两行了。

这就是因为`ServiceLoader.load(Search.class)`在加载某接口时，会去`META-INF/services`下找接口的全限定名文件，再根据里面的内容加载相应的实现类。

这就是spi的思想，接口的实现由provider实现，provider只用在提交的jar包里的`META-INF/services`下根据平台定义的接口新建文件，并添加进相应的实现类内容就好。

## [#](#_2-java-集合) 2 Java 集合

> 容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对(两个对象)的映射表。

### [#](#_2-1-collection) 2.1 Collection

#### [#](#集合有哪些类) 集合有哪些类？

- Set 
  - TreeSet 基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
  - HashSet 基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
  - LinkedHashSet 具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。
- List 
  - ArrayList 基于动态数组实现，支持随机访问。
  - Vector 和 ArrayList 类似，但它是线程安全的。
  - LinkedList 基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。
- Queue 
  - LinkedList 可以用它来实现双向队列。
  - PriorityQueue 基于堆结构实现，可以用它来实现优先队列。

#### [#](#arraylist的底层) ArrayList的底层？

*ArrayList*实现了*List*接口，是顺序容器，即元素存放的数据与放进去的顺序相同，允许放入`null`元素，底层通过**数组实现**。除该类未实现同步外，其余跟*Vector*大致相同。每个*ArrayList*都有一个容量(capacity)，表示底层数组的实际大小，容器内存储元素的个数不能多于当前容量。当向容器中添加元素时，如果容量不足，容器会自动增大底层数组的大小。前面已经提过，Java泛型只是编译器提供的语法糖，所以这里的数组是一个Object数组，以便能够容纳任何类型的对象。

![ArrayList_base](/images/collection/ArrayList_base.png)

#### [#](#arraylist自动扩容) ArrayList自动扩容？

每当向数组中添加元素时，都要去检查添加后元素的个数是否会超出当前数组的长度，如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过ensureCapacity(int minCapacity)方法来实现。在实际添加大量元素前，我也可以使用ensureCapacity来手动增加ArrayList实例的容量，以减少递增式再分配的数量。

数组进行扩容时，会将老数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是其原容量的1.5倍。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素的多少时，要在构造ArrayList实例时，就指定其容量，以避免数组扩容的发生。或者根据实际需求，通过调用ensureCapacity方法来手动增加ArrayList实例的容量。

![ArrayList_add](/images/collection/ArrayList_add.png)

#### [#](#arraylist的fail-fast机制) ArrayList的Fail-Fast机制？

ArrayList也采用了快速失败的机制，通过记录modCount参数来实现。在面对并发的修改时，迭代器很快就会完全失败，而不是冒着在将来某个不确定时间发生任意不确定行为的风险。

### [#](#_2-2-map) 2.2 Map

#### [#](#map有哪些类) Map有哪些类？

- `TreeMap` 基于红黑树实现。
- `HashMap` 1.7基于哈希表实现，1.8基于数组+链表+红黑树。
- `HashTable` 和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高(1.7 ConcurrentHashMap 引入了分段锁, 1.8 引入了红黑树)。
- `LinkedHashMap` 使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用(LRU)顺序。

#### [#](#jdk7-hashmap如何实现) JDK7 HashMap如何实现？

哈希表有两种实现方式，一种开放地址方式(Open addressing)，另一种是冲突链表方式(Separate chaining with linked lists)。**Java7 \*HashMap\*采用的是冲突链表方式**。

![HashMap_base](/images/collection/HashMap_base.png)

从上图容易看出，如果选择合适的哈希函数，`put()`和`get()`方法可以在常数时间内完成。但在对*HashMap*进行迭代时，需要遍历整个table以及后面跟的冲突链表。因此对于迭代比较频繁的场景，不宜将*HashMap*的初始大小设的过大。

有两个参数可以影响*HashMap*的性能: 初始容量(inital capacity)和负载系数(load factor)。初始容量指定了初始`table`的大小，负载系数用来指定自动扩容的临界值。当`entry`的数量超过`capacity*load_factor`时，容器将自动扩容并重新哈希。对于插入元素较多的场景，将初始容量设大可以减少重新哈希的次数。

#### [#](#jdk8-hashmap如何实现) JDK8 HashMap如何实现？

根据 Java7 HashMap 的介绍，我们知道，查找的时候，根据 hash 值我们能够快速定位到数组的具体下标，但是之后的话，需要顺着链表一个个比较下去才能找到我们需要的，时间复杂度取决于链表的长度，为 O(n)。

为了降低这部分的开销，在 Java8 中，当链表中的元素达到了 8 个时，会将链表转换为红黑树，在这些位置进行查找的时候可以降低时间复杂度为 O(logN)。

![img](/images/java/java-collection-hashmap8.png)

#### [#](#hashset是如何实现的) HashSet是如何实现的？

*HashSet*是对*HashMap*的简单包装，对*HashSet*的函数调用都会转换成合适的*HashMap*方法

```Java
//HashSet是对HashMap的简单包装
public class HashSet<E>
{
	......
	private transient HashMap<E,Object> map;//HashSet里面有一个HashMap
    // Dummy value to associate with an Object in the backing Map
    private static final Object PRESENT = new Object();
    public HashSet() {
        map = new HashMap<>();
    }
    ......
    public boolean add(E e) {//简单的方法转换
        return map.put(e, PRESENT)==null;
    }
    ......
}
```

#### [#](#什么是weakhashmap) 什么是WeakHashMap?

我们都知道Java中内存是通过GC自动管理的，GC会在程序运行过程中自动判断哪些对象是可以被回收的，并在合适的时机进行内存释放。GC判断某个对象是否可被回收的依据是，**是否有有效的引用指向该对象**。如果没有有效引用指向该对象(基本意味着不存在访问该对象的方式)，那么该对象就是可回收的。这里的**有效引用** 并不包括**弱引用**。也就是说，**虽然弱引用可以用来访问对象，但进行垃圾回收时弱引用并不会被考虑在内，仅有弱引用指向的对象仍然会被GC回收**。

WeakHashMap 内部是通过弱引用来管理entry的，弱引用的特性对应到 WeakHashMap 上意味着什么呢？

*WeakHashMap* 里的`entry`可能会被GC自动删除，即使程序员没有调用`remove()`或者`clear()`方法。

***WeakHashMap\* 的这个特点特别适用于需要缓存的场景**。在缓存场景下，由于内存是有限的，不能缓存所有对象；对象缓存命中可以提高系统效率，但缓存MISS也不会造成错误，因为可以通过计算重新得到。

## [#](#_3-java-并发) 3 Java 并发

> 并发和多线程

### [#](#_3-1-并发基础) 3.1 并发基础

- [Java 并发 - 理论基础]()
- [Java 并发 - 线程基础]()

#### [#](#多线程的出现是要解决什么问题的-本质什么) 多线程的出现是要解决什么问题的? 本质什么?

CPU、内存、I/O 设备的速度是有极大差异的，为了合理利用 CPU 的高性能，平衡这三者的速度差异，计算机体系结构、操作系统、编译程序都做出了贡献，主要体现为:

- CPU 增加了缓存，以均衡与内存的速度差异；// 导致可见性问题
- 操作系统增加了进程、线程，以分时复用 CPU，进而均衡 CPU 与 I/O 设备的速度差异；// 导致原子性问题
- 编译程序优化指令执行次序，使得缓存能够得到更加合理地利用。// 导致有序性问题

#### [#](#java是怎么解决并发问题的) Java是怎么解决并发问题的?

Java 内存模型是个很复杂的规范，具体看[Java 内存模型详解]()。

**理解的第一个维度：核心知识点**

JMM本质上可以理解为，Java 内存模型规范了 JVM 如何提供按需禁用缓存和编译优化的方法。具体来说，这些方法包括：

- volatile、synchronized 和 final 三个关键字
- Happens-Before 规则

**理解的第二个维度：可见性，有序性，原子性**

- **原子性**

在Java中，对基本数据类型的变量的读取和赋值操作是原子性操作，即这些操作是不可被中断的，要么执行，要么不执行。 请分析以下哪些操作是原子性操作：

```java
x = 10;        //语句1: 直接将数值10赋值给x，也就是说线程执行这个语句的会直接将数值10写入到工作内存中
y = x;         //语句2: 包含2个操作，它先要去读取x的值，再将x的值写入工作内存，虽然读取x的值以及 将x的值写入工作内存 这2个操作都是原子性操作，但是合起来就不是原子性操作了。
x++;           //语句3： x++包括3个操作：读取x的值，进行加1操作，写入新的值。
x = x + 1;     //语句4： 同语句3
```

上面4个语句只有语句1的操作具备原子性。

也就是说，只有简单的读取、赋值（而且必须是将数字赋值给某个变量，变量之间的相互赋值不是原子操作）才是原子操作。

> 从上面可以看出，Java内存模型只保证了基本读取和赋值是原子性操作，如果要实现更大范围操作的原子性，可以通过synchronized和Lock来实现。由于synchronized和Lock能够保证任一时刻只有一个线程执行该代码块，那么自然就不存在原子性问题了，从而保证了原子性。

- **可见性**

Java提供了volatile关键字来保证可见性。

当一个共享变量被volatile修饰时，它会保证修改的值会立即被更新到主存，当有其他线程需要读取时，它会去内存中读取新值。

而普通的共享变量不能保证可见性，因为普通共享变量被修改之后，什么时候被写入主存是不确定的，当其他线程去读取时，此时内存中可能还是原来的旧值，因此无法保证可见性。

> 另外，通过synchronized和Lock也能够保证可见性，synchronized和Lock能保证同一时刻只有一个线程获取锁然后执行同步代码，并且在释放锁之前会将对变量的修改刷新到主存当中。因此可以保证可见性。

- **有序性**

在Java里面，可以通过volatile关键字来保证一定的“有序性”。另外可以通过synchronized和Lock来保证有序性，很显然，synchronized和Lock保证每个时刻是有一个线程执行同步代码，相当于是让线程顺序执行同步代码，自然就保证了有序性。当然JMM是通过Happens-Before 规则来保证有序性的。

#### [#](#线程安全有哪些实现思路) 线程安全有哪些实现思路?

1. **互斥同步**

synchronized 和 ReentrantLock。

1. **非阻塞同步**

互斥同步最主要的问题就是线程阻塞和唤醒所带来的性能问题，因此这种同步也称为阻塞同步。

互斥同步属于一种悲观的并发策略，总是认为只要不去做正确的同步措施，那就肯定会出现问题。无论共享数据是否真的会出现竞争，它都要进行加锁(这里讨论的是概念模型，实际上虚拟机会优化掉很大一部分不必要的加锁)、用户态核心态转换、维护锁计数器和检查是否有被阻塞的线程需要唤醒等操作。

- CAS

随着硬件指令集的发展，我们可以使用基于冲突检测的乐观并发策略: 先进行操作，如果没有其它线程争用共享数据，那操作就成功了，否则采取补偿措施(不断地重试，直到成功为止)。这种乐观的并发策略的许多实现都不需要将线程阻塞，因此这种同步操作称为非阻塞同步。

乐观锁需要操作和冲突检测这两个步骤具备原子性，这里就不能再使用互斥同步来保证了，只能靠硬件来完成。硬件支持的原子性操作最典型的是: 比较并交换(Compare-and-Swap，CAS)。CAS 指令需要有 3 个操作数，分别是内存地址 V、旧的预期值 A 和新值 B。当执行操作时，只有当 V 的值等于 A，才将 V 的值更新为 B。

- AtomicInteger

J.U.C 包里面的整数原子类 AtomicInteger，其中的 compareAndSet() 和 getAndIncrement() 等方法都使用了 Unsafe 类的 CAS 操作。

1. **无同步方案**

要保证线程安全，并不是一定就要进行同步。如果一个方法本来就不涉及共享数据，那它自然就无须任何同步措施去保证正确性。

- 栈封闭

多个线程访问同一个方法的局部变量时，不会出现线程安全问题，因为局部变量存储在虚拟机栈中，属于线程私有的。

- 线程本地存储(Thread Local Storage)

如果一段代码中所需要的数据必须与其他代码共享，那就看看这些共享数据的代码是否能保证在同一个线程中执行。如果能保证，我们就可以把共享数据的可见范围限制在同一个线程之内，这样，无须同步也能保证线程之间不出现数据争用的问题。

#### [#](#如何理解并发和并行的区别) 如何理解并发和并行的区别?

并发是指一个处理器同时处理多个任务。

![img](/images/java/java-bingfa-2.png)

并行是指多个处理器或者是多核的处理器同时处理多个不同的任务。

![img](/images/java/java-bingfa-1.png)

#### [#](#线程有哪几种状态-分别说明从一种状态到另一种状态转变有哪些方式) 线程有哪几种状态? 分别说明从一种状态到另一种状态转变有哪些方式?

![image](/images/pics/ace830df-9919-48ca-91b5-60b193f593d2.png)

- 新建(New)

创建后尚未启动。

- 可运行(Runnable)

可能正在运行，也可能正在等待 CPU 时间片。

包含了操作系统线程状态中的 Running 和 Ready。

- 阻塞(Blocking)

等待获取一个排它锁，如果其线程释放了锁就会结束此状态。

- 无限期等待(Waiting)

等待其它线程显式地唤醒，否则不会被分配 CPU 时间片。

| 进入方法                                   | 退出方法                             |
| ------------------------------------------ | ------------------------------------ |
| 没有设置 Timeout 参数的 Object.wait() 方法 | Object.notify() / Object.notifyAll() |
| 没有设置 Timeout 参数的 Thread.join() 方法 | 被调用的线程执行完毕                 |
| LockSupport.park() 方法                    | -                                    |

- 限期等待(Timed Waiting)

无需等待其它线程显式地唤醒，在一定时间之后会被系统自动唤醒。

调用 Thread.sleep() 方法使线程进入限期等待状态时，常常用“使一个线程睡眠”进行描述。

调用 Object.wait() 方法使线程进入限期等待或者无限期等待时，常常用“挂起一个线程”进行描述。

睡眠和挂起是用来描述行为，而阻塞和等待用来描述状态。

阻塞和等待的区别在于，阻塞是被动的，它是在等待获取一个排它锁。而等待是主动的，通过调用 Thread.sleep() 和 Object.wait() 等方法进入。

| 进入方法                                 | 退出方法                                        |
| ---------------------------------------- | ----------------------------------------------- |
| Thread.sleep() 方法                      | 时间结束                                        |
| 设置了 Timeout 参数的 Object.wait() 方法 | 时间结束 / Object.notify() / Object.notifyAll() |
| 设置了 Timeout 参数的 Thread.join() 方法 | 时间结束 / 被调用的线程执行完毕                 |
| LockSupport.parkNanos() 方法             | -                                               |
| LockSupport.parkUntil() 方法             | -                                               |

- 死亡(Terminated)

可以是线程结束任务之后自己结束，或者产生了异常而结束。

#### [#](#通常线程有哪几种使用方式) 通常线程有哪几种使用方式?

有三种使用线程的方法:

- 实现 Runnable 接口；
- 实现 Callable 接口；
- 继承 Thread 类。

实现 Runnable 和 Callable 接口的类只能当做一个可以在线程中运行的任务，不是真正意义上的线程，因此最后还需要通过 Thread 来调用。可以说任务是通过线程驱动从而执行的。

#### [#](#基础线程机制有哪些) 基础线程机制有哪些?

- **Executor**

Executor 管理多个异步任务的执行，而无需程序员显式地管理线程的生命周期。这里的异步是指多个任务的执行互不干扰，不需要进行同步操作。

主要有三种 Executor:

1. CachedThreadPool: 一个任务创建一个线程；
2. FixedThreadPool: 所有任务只能使用固定大小的线程；
3. SingleThreadExecutor: 相当于大小为 1 的 FixedThreadPool。

- **Daemon**

守护线程是程序运行时在后台提供服务的线程，不属于程序中不可或缺的部分。

当所有非守护线程结束时，程序也就终止，同时会杀死所有守护线程。

main() 属于非守护线程。使用 setDaemon() 方法将一个线程设置为守护线程。

- **sleep()**

Thread.sleep(millisec) 方法会休眠当前正在执行的线程，millisec 单位为毫秒。

sleep() 可能会抛出 InterruptedException，因为异常不能跨线程传播回 main() 中，因此必须在本地进行处理。线程中抛出的其它异常也同样需要在本地进行处理。

- **yield()**

对静态方法 Thread.yield() 的调用声明了当前线程已经完成了生命周期中最重要的部分，可以切换给其它线程来执行。该方法只是对线程调度器的一个建议，而且也只是建议具有相同优先级的其它线程可以运行。

#### [#](#线程的中断方式有哪些) 线程的中断方式有哪些?

一个线程执行完毕之后会自动结束，如果在运行过程中发生异常也会提前结束。

- **InterruptedException**

通过调用一个线程的 interrupt() 来中断该线程，如果该线程处于阻塞、限期等待或者无限期等待状态，那么就会抛出 InterruptedException，从而提前结束该线程。但是不能中断 I/O 阻塞和 synchronized 锁阻塞。

对于以下代码，在 main() 中启动一个线程之后再中断它，由于线程中调用了 Thread.sleep() 方法，因此会抛出一个 InterruptedException，从而提前结束线程，不执行之后的语句。

```java
public class InterruptExample {

    private static class MyThread1 extends Thread {
        @Override
        public void run() {
            try {
                Thread.sleep(2000);
                System.out.println("Thread run");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new MyThread1();
        thread1.start();
        thread1.interrupt();
        System.out.println("Main run");
    }
}
Main run
java.lang.InterruptedException: sleep interrupted
    at java.lang.Thread.sleep(Native Method)
    at InterruptExample.lambda$main$0(InterruptExample.java:5)
    at InterruptExample$$Lambda$1/713338599.run(Unknown Source)
    at java.lang.Thread.run(Thread.java:745)
```

- **interrupted()**

如果一个线程的 run() 方法执行一个无限循环，并且没有执行 sleep() 等会抛出 InterruptedException 的操作，那么调用线程的 interrupt() 方法就无法使线程提前结束。

但是调用 interrupt() 方法会设置线程的中断标记，此时调用 interrupted() 方法会返回 true。因此可以在循环体中使用 interrupted() 方法来判断线程是否处于中断状态，从而提前结束线程。

- **Executor 的中断操作**

调用 Executor 的 shutdown() 方法会等待线程都执行完毕之后再关闭，但是如果调用的是 shutdownNow() 方法，则相当于调用每个线程的 interrupt() 方法。

#### [#](#线程的互斥同步方式有哪些-如何比较和选择) 线程的互斥同步方式有哪些? 如何比较和选择?

Java 提供了两种锁机制来控制多个线程对共享资源的互斥访问，第一个是 JVM 实现的 synchronized，而另一个是 JDK 实现的 ReentrantLock。

**1. 锁的实现**

synchronized 是 JVM 实现的，而 ReentrantLock 是 JDK 实现的。

**2. 性能**

新版本 Java 对 synchronized 进行了很多优化，例如自旋锁等，synchronized 与 ReentrantLock 大致相同。

**3. 等待可中断**

当持有锁的线程长期不释放锁的时候，正在等待的线程可以选择放弃等待，改为处理其他事情。

ReentrantLock 可中断，而 synchronized 不行。

**4. 公平锁**

公平锁是指多个线程在等待同一个锁时，必须按照申请锁的时间顺序来依次获得锁。

synchronized 中的锁是非公平的，ReentrantLock 默认情况下也是非公平的，但是也可以是公平的。

**5. 锁绑定多个条件**

一个 ReentrantLock 可以同时绑定多个 Condition 对象。

#### [#](#线程之间有哪些协作方式) 线程之间有哪些协作方式?

当多个线程可以一起工作去解决某个问题时，如果某些部分必须在其它部分之前完成，那么就需要对线程进行协调。

- **join()**

在线程中调用另一个线程的 join() 方法，会将当前线程挂起，而不是忙等待，直到目标线程结束。

对于以下代码，虽然 b 线程先启动，但是因为在 b 线程中调用了 a 线程的 join() 方法，b 线程会等待 a 线程结束才继续执行，因此最后能够保证 a 线程的输出先于 b 线程的输出。

```java
public class JoinExample {

    private class A extends Thread {
        @Override
        public void run() {
            System.out.println("A");
        }
    }

    private class B extends Thread {

        private A a;

        B(A a) {
            this.a = a;
        }

        @Override
        public void run() {
            try {
                a.join();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("B");
        }
    }

    public void test() {
        A a = new A();
        B b = new B(a);
        b.start();
        a.start();
    }
}
public static void main(String[] args) {
    JoinExample example = new JoinExample();
    example.test();
}
A
B
```

- **wait() notify() notifyAll()**

调用 wait() 使得线程等待某个条件满足，线程在等待时会被挂起，当其他线程的运行使得这个条件满足时，其它线程会调用 notify() 或者 notifyAll() 来唤醒挂起的线程。

它们都属于 Object 的一部分，而不属于 Thread。

只能用在同步方法或者同步控制块中使用，否则会在运行时抛出 IllegalMonitorStateExeception。

使用 wait() 挂起期间，线程会释放锁。这是因为，如果没有释放锁，那么其它线程就无法进入对象的同步方法或者同步控制块中，那么就无法执行 notify() 或者 notifyAll() 来唤醒挂起的线程，造成死锁。

**wait() 和 sleep() 的区别**

- wait() 是 Object 的方法，而 sleep() 是 Thread 的静态方法；
- wait() 会释放锁，sleep() 不会。

- **await() signal() signalAll()**

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。

### [#](#_3-2-并发关键字) 3.2 并发关键字

- [关键字: synchronized详解]()
- [关键字: volatile详解]()
- [关键字: final详解]()

#### [#](#synchronized可以作用在哪里) Synchronized可以作用在哪里?

- 对象锁
- 方法锁
- 类锁

#### [#](#synchronized本质上是通过什么保证线程安全的) Synchronized本质上是通过什么保证线程安全的?

- **加锁和释放锁的原理**

深入JVM看字节码，创建如下的代码：

```java
public class SynchronizedDemo2 {
    Object object = new Object();
    public void method1() {
        synchronized (object) {

        }
    }
}
```

使用javac命令进行编译生成.class文件

```bash
>javac SynchronizedDemo2.java
```

使用javap命令反编译查看.class文件的信息

```bash
>javap -verbose SynchronizedDemo2.class
```

得到如下的信息：

![img](/images/thread/java-thread-x-key-schronized-1.png)

关注红色方框里的`monitorenter`和`monitorexit`即可。

`Monitorenter`和`Monitorexit`指令，会让对象在执行，使其锁计数器加1或者减1。每一个对象在同一时间只与一个monitor(锁)相关联，而一个monitor在同一时间只能被一个线程获得，一个对象在尝试获得与这个对象相关联的Monitor锁的所有权的时候，monitorenter指令会发生如下3中情况之一：

- monitor计数器为0，意味着目前还没有被获得，那这个线程就会立刻获得然后把锁计数器+1，一旦+1，别的线程再想获取，就需要等待
- 如果这个monitor已经拿到了这个锁的所有权，又重入了这把锁，那锁计数器就会累加，变成2，并且随着重入的次数，会一直累加
- 这把锁已经被别的线程获取了，等待锁释放

`monitorexit指令`：释放对于monitor的所有权，释放过程很简单，就是讲monitor的计数器减1，如果减完以后，计数器不是0，则代表刚才是重入进来的，当前线程还继续持有这把锁的所有权，如果计数器变成0，则代表当前线程不再拥有该monitor的所有权，即释放锁。

下图表现了对象，对象监视器，同步队列以及执行线程状态之间的关系：

![img](/images/thread/java-thread-x-key-schronized-2.png)

该图可以看出，任意线程对Object的访问，首先要获得Object的监视器，如果获取失败，该线程就进入同步状态，线程状态变为BLOCKED，当Object的监视器占有者释放后，在同步队列中得线程就会有机会重新获取该监视器。

- **可重入原理：加锁次数计数器**

看如下的例子：

```java
public class SynchronizedDemo {

    public static void main(String[] args) {
        synchronized (SynchronizedDemo.class) {

        }
        method2();
    }

    private synchronized static void method2() {

    }
}
```

对应的字节码

```bash
  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #2                  // class tech/pdai/test/synchronized/SynchronizedDemo
         2: dup
         3: astore_1
         4: monitorenter
         5: aload_1
         6: monitorexit
         7: goto          15
        10: astore_2
        11: aload_1
        12: monitorexit
        13: aload_2
        15: invokestatic  #3                  // Method method2:()V
      Exception table:
         from    to  target type
             5     7    10   any
            10    13    10   any
```

上面的SynchronizedDemo中在执行完同步代码块之后紧接着再会去执行一个静态同步方法，而这个方法锁的对象依然就这个类对象，那么这个正在执行的线程还需要获取该锁吗? 答案是不必的，从上图中就可以看出来，执行静态同步方法的时候就只有一条monitorexit指令，并没有monitorenter获取锁的指令。这就是锁的重入性，即在同一锁程中，线程不需要再次获取同一把锁。

Synchronized先天具有重入性。每个对象拥有一个计数器，当线程获取该对象锁后，计数器就会加一，释放锁后就会将计数器减一。

- **保证可见性的原理：内存模型和happens-before规则**

Synchronized的happens-before规则，即监视器锁规则：对同一个监视器的解锁，happens-before于对该监视器的加锁。继续来看代码：

```java
public class MonitorDemo {
    private int a = 0;

    public synchronized void writer() {     // 1
        a++;                                // 2
    }                                       // 3

    public synchronized void reader() {    // 4
        int i = a;                         // 5
    }                                      // 6
}
```

该代码的happens-before关系如图所示：

![img](/images/thread/java-thread-x-key-schronized-3.png)

在图中每一个箭头连接的两个节点就代表之间的happens-before关系，黑色的是通过程序顺序规则推导出来，红色的为监视器锁规则推导而出：线程A释放锁happens-before线程B加锁，蓝色的则是通过程序顺序规则和监视器锁规则推测出来happens-befor关系，通过传递性规则进一步推导的happens-before关系。现在我们来重点关注2 happens-before 5，通过这个关系我们可以得出什么?

根据happens-before的定义中的一条:如果A happens-before B，则A的执行结果对B可见，并且A的执行顺序先于B。线程A先对共享变量A进行加一，由2 happens-before 5关系可知线程A的执行结果对线程B可见即线程B所读取到的a的值为1。

#### [#](#synchronized使得同时只有一个线程可以执行-性能比较差-有什么提升的方法) Synchronized使得同时只有一个线程可以执行，性能比较差，有什么提升的方法?

简单来说在JVM中monitorenter和monitorexit字节码依赖于底层的操作系统的Mutex Lock来实现的，但是由于使用Mutex Lock需要将当前线程挂起并从用户态切换到内核态来执行，这种切换的代价是非常昂贵的；然而在现实中的大部分情况下，同步方法是运行在单线程环境(无锁竞争环境)如果每次都调用Mutex Lock那么将严重的影响程序的性能。**不过在jdk1.6中对锁的实现引入了大量的优化，如锁粗化(Lock Coarsening)、锁消除(Lock Elimination)、轻量级锁(Lightweight Locking)、偏向锁(Biased Locking)、适应性自旋(Adaptive Spinning)等技术来减少锁操作的开销**。

- **锁粗化(Lock Coarsening)**：也就是减少不必要的紧连在一起的unlock，lock操作，将多个连续的锁扩展成一个范围更大的锁。
- **锁消除(Lock Elimination)**：通过运行时JIT编译器的逃逸分析来消除一些没有在当前同步块以外被其他线程共享的数据的锁保护，通过逃逸分析也可以在线程本地Stack上进行对象空间的分配(同时还可以减少Heap上的垃圾收集开销)。
- **轻量级锁(Lightweight Locking)**：这种锁实现的背后基于这样一种假设，即在真实的情况下我们程序中的大部分同步代码一般都处于无锁竞争状态(即单线程执行环境)，在无锁竞争的情况下完全可以避免调用操作系统层面的重量级互斥锁，取而代之的是在monitorenter和monitorexit中只需要依靠一条CAS原子指令就可以完成锁的获取及释放。当存在锁竞争的情况下，执行CAS指令失败的线程将调用操作系统互斥锁进入到阻塞状态，当锁被释放的时候被唤醒。
- **偏向锁(Biased Locking)**：是为了在无锁竞争的情况下避免在锁获取过程中执行不必要的CAS原子指令，因为CAS原子指令虽然相对于重量级锁来说开销比较小但还是存在非常可观的本地延迟。
- **适应性自旋(Adaptive Spinning)**：当线程在获取轻量级锁的过程中执行CAS操作失败时，在进入与monitor相关联的操作系统重量级锁(mutex semaphore)前会进入忙等待(Spinning)然后再次尝试，当尝试一定的次数后如果仍然没有成功则调用与该monitor关联的semaphore(即互斥锁)进入到阻塞状态。

#### [#](#synchronized由什么样的缺陷-java-lock是怎么弥补这些缺陷的) Synchronized由什么样的缺陷? Java Lock是怎么弥补这些缺陷的?

- **synchronized的缺陷**

1. **效率低**：锁的释放情况少，只有代码执行完毕或者异常结束才会释放锁；试图获取锁的时候不能设定超时，不能中断一个正在使用锁的线程，相对而言，Lock可以中断和设置超时
2. **不够灵活**：加锁和释放的时机单一，每个锁仅有一个单一的条件(某个对象)，相对而言，读写锁更加灵活
3. **无法知道是否成功获得锁**，相对而言，Lock可以拿到状态

- **Lock解决相应问题**

Lock类这里不做过多解释，主要看里面的4个方法:

1. `lock()`: 加锁
2. `unlock()`: 解锁
3. `tryLock()`: 尝试获取锁，返回一个boolean值
4. `tryLock(long,TimeUtil)`: 尝试获取锁，可以设置超时

Synchronized只有锁只与一个条件(是否获取锁)相关联，不灵活，后来**Condition与Lock的结合**解决了这个问题。

多线程竞争一个锁时，其余未得到锁的线程只能不停的尝试获得锁，而不能中断。高并发的情况下会导致性能下降。ReentrantLock的lockInterruptibly()方法可以优先考虑响应中断。 一个线程等待时间过长，它可以中断自己，然后ReentrantLock响应这个中断，不再让这个线程继续等待。有了这个机制，使用ReentrantLock时就不会像synchronized那样产生死锁了。

#### [#](#synchronized和lock的对比-和选择) Synchronized和Lock的对比，和选择?

- **存在层次上**

synchronized: Java的关键字，在jvm层面上

Lock: 是一个接口

- **锁的释放**

synchronized: 1、以获取锁的线程执行完同步代码，释放锁 2、线程执行发生异常，jvm会让线程释放锁

Lock: 在finally中必须释放锁，不然容易造成线程死锁

- **锁的获取**

synchronized: 假设A线程获得锁，B线程等待。如果A线程阻塞，B线程会一直等待

Lock: 分情况而定，Lock有多个锁获取的方式，大致就是可以尝试获得锁，线程可以不用一直等待(可以通过tryLock判断有没有锁)

- **锁的释放（死锁产生）**

synchronized: 在发生异常时候会自动释放占有的锁，因此不会出现死锁

Lock: 发生异常时候，不会主动释放占有的锁，必须手动unlock来释放锁，可能引起死锁的发生

- **锁的状态**

synchronized: 无法判断

Lock: 可以判断

- **锁的类型**

synchronized: 可重入 不可中断 非公平

Lock: 可重入 可判断 可公平（两者皆可）

- **性能**

synchronized: 少量同步

Lock: 大量同步

Lock可以提高多个线程进行读操作的效率。（可以通过readwritelock实现读写分离） 在资源竞争不是很激烈的情况下，Synchronized的性能要优于ReetrantLock，但是在资源竞争很激烈的情况下，Synchronized的性能会下降几十倍，但是ReetrantLock的性能能维持常态；

ReentrantLock提供了多样化的同步，比如有时间限制的同步，可以被Interrupt的同步（synchronized的同步是不能Interrupt的）等。在资源竞争不激烈的情形下，性能稍微比synchronized差点点。但是当同步非常激烈的时候，synchronized的性能一下子能下降好几十倍。而ReentrantLock确还能维持常态。

- **调度**

synchronized: 使用Object对象本身的wait 、notify、notifyAll调度机制

Lock: 可以使用Condition进行线程之间的调度

- **用法**

synchronized: 在需要同步的对象中加入此控制，synchronized可以加在方法上，也可以加在特定代码块中，括号中表示需要锁的对象。

Lock: 一般使用ReentrantLock类做为锁。在加锁和解锁处需要通过lock()和unlock()显示指出。所以一般会在finally块中写unlock()以防死锁。

- **底层实现**

synchronized: 底层使用指令码方式来控制锁的，映射成字节码指令就是增加来两个指令：monitorenter和monitorexit。当线程执行遇到monitorenter指令时会尝试获取内置锁，如果获取锁则锁计数器+1，如果没有获取锁则阻塞；当遇到monitorexit指令时锁计数器-1，如果计数器为0则释放锁。

Lock: 底层是CAS乐观锁，依赖AbstractQueuedSynchronizer类，把所有的请求线程构成一个CLH队列。而对该队列的操作均通过Lock-Free（CAS）操作。

#### [#](#synchronized在使用时有何注意事项) Synchronized在使用时有何注意事项?

- 锁对象不能为空，因为锁的信息都保存在对象头里
- 作用域不宜过大，影响程序执行的速度，控制范围过大，编写代码也容易出错
- 避免死锁
- 在能选择的情况下，既不要用Lock也不要用synchronized关键字，用java.util.concurrent包中的各种各样的类，如果不用该包下的类，在满足业务的情况下，可以使用synchronized关键，因为代码量少，避免出错

#### [#](#synchronized修饰的方法在抛出异常时-会释放锁吗) Synchronized修饰的方法在抛出异常时,会释放锁吗?

会

#### [#](#多个线程等待同一个synchronized锁的时候-jvm如何选择下一个获取锁的线程) 多个线程等待同一个Synchronized锁的时候，JVM如何选择下一个获取锁的线程?

非公平锁，即抢占式。

#### [#](#synchronized是公平锁吗) synchronized是公平锁吗？

synchronized实际上是非公平的，新来的线程有可能立即获得监视器，而在等待区中等候已久的线程可能再次等待，这样有利于提高性能，但是也可能会导致饥饿现象。

#### [#](#volatile关键字的作用是什么) volatile关键字的作用是什么?

- **防重排序** 我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁(DCL)的方式来实现。其源码如下：

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

现在我们分析一下为什么要在变量singleton之间加上volatile关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤：

1. 分配内存空间。
2. 初始化对象。
3. 将内存空间的地址赋值给对应的引用。

但是由于操作系统可以**对指令进行重排序**，所以上面的过程也可能会变成如下过程：

1. 分配内存空间。
2. 将内存空间的地址赋值给对应的引用。
3. 初始化对象

如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。

- **实现可见性**

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。volatile关键字能有效的解决这个问题，我们看下下面的例子，就可以知道其作用：

```java
public class TestVolatile {
    private static boolean stop = false;

    public static void main(String[] args) {
        // Thread-A
        new Thread("Thread A") {
            @Override
            public void run() {
                while (!stop) {
                }
                System.out.println(Thread.currentThread() + " stopped");
            }
        }.start();

        // Thread-main
        try {
            TimeUnit.SECONDS.sleep(1);
            System.out.println(Thread.currentThread() + " after 1 seconds");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        stop = true;
    }
}
```

执行输出如下

```bash
Thread[main,5,main] after 1 seconds

// Thread A一直在loop, 因为Thread A 由于可见性原因看不到Thread Main 已经修改stop的值
```

可以看到 Thread-main 休眠1秒之后，设置 stop = ture，但是Thread A根本没停下来，这就是可见性问题。如果通过在stop变量前面加上volatile关键字则会真正stop:

```bash
Thread[main,5,main] after 1 seconds
Thread[Thread A,5,main] stopped

Process finished with exit code 0
```

- **保证原子性:单次读/写**

volatile不能保证完全的原子性，只能保证单次的读/写操作具有原子性。

#### [#](#volatile能保证原子性吗) volatile能保证原子性吗?

不能完全保证，只能保证单次的读/写操作具有原子性。

#### [#](#_32位机器上共享的long和double变量的为什么要用volatile) 32位机器上共享的long和double变量的为什么要用volatile?

因为long和double两种数据类型的操作可分为高32位和低32位两部分，因此普通的long或double类型读/写可能不是原子的。因此，鼓励大家将共享的long和double变量设置为volatile类型，这样能保证任何情况下对long和double的单次读/写操作都具有原子性。

如下是JLS中的解释：

> 17.7 Non-Atomic Treatment of double and long

- For the purposes of the Java programming language memory model, a single write to a non-volatile long or double value is treated as two separate writes: one to each 32-bit half. This can result in a situation where a thread sees the first 32 bits of a 64-bit value from one write, and the second 32 bits from another write.
- Writes and reads of volatile long and double values are always atomic.
- Writes to and reads of references are always atomic, regardless of whether they are implemented as 32-bit or 64-bit values.
- Some implementations may find it convenient to divide a single write action on a 64-bit long or double value into two write actions on adjacent 32-bit values. For efficiency’s sake, this behavior is implementation-specific; an implementation of the Java Virtual Machine is free to perform writes to long and double values atomically or in two parts.
- Implementations of the Java Virtual Machine are encouraged to avoid splitting 64-bit values where possible. Programmers are encouraged to declare shared 64-bit values as volatile or synchronize their programs correctly to avoid possible complications.

目前各种平台下的商用虚拟机都选择把 64 位数据的读写操作作为原子操作来对待，因此我们在编写代码时一般不把long 和 double 变量专门声明为 volatile多数情况下也是不会错的。

#### [#](#volatile是如何实现可见性的) volatile是如何实现可见性的?

内存屏障。

#### [#](#volatile是如何实现有序性的) volatile是如何实现有序性的?

happens-before等

#### [#](#说下volatile的应用场景) 说下volatile的应用场景?

使用 volatile 必须具备的条件

1. 对变量的写操作不依赖于当前值。
2. 该变量没有包含在具有其他变量的不变式中。
3. 只有在状态真正独立于程序内其他内容时才能使用 volatile。

- **例子 1： 单例模式**

单例模式的一种实现方式，但很多人会忽略 volatile 关键字，因为没有该关键字，程序也可以很好的运行，只不过代码的稳定性总不是 100%，说不定在未来的某个时刻，隐藏的 bug 就出来了。

```java
class Singleton {
    private volatile static Singleton instance;
    private Singleton() {
    }
    public static Singleton getInstance() {
        if (instance == null) {
            syschronized(Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    } 
}
```

- **例子2： volatile bean**

在 volatile bean 模式中，JavaBean 的所有数据成员都是 volatile 类型的，并且 getter 和 setter 方法必须非常普通 —— 除了获取或设置相应的属性外，不能包含任何逻辑。此外，对于对象引用的数据成员，引用的对象必须是有效不可变的。(这将禁止具有数组值的属性，因为当数组引用被声明为 volatile 时，只有引用而不是数组本身具有 volatile 语义)。对于任何 volatile 变量，不变式或约束都不能包含 JavaBean 属性。

```java
@ThreadSafe
public class Person {
    private volatile String firstName;
    private volatile String lastName;
    private volatile int age;
 
    public String getFirstName() { return firstName; }
    public String getLastName() { return lastName; }
    public int getAge() { return age; }
 
    public void setFirstName(String firstName) { 
        this.firstName = firstName;
    }
 
    public void setLastName(String lastName) { 
        this.lastName = lastName;
    }
 
    public void setAge(int age) { 
        this.age = age;
    }
}
```

#### [#](#所有的final修饰的字段都是编译期常量吗) 所有的final修饰的字段都是编译期常量吗?

不是

#### [#](#如何理解private所修饰的方法是隐式的final) 如何理解private所修饰的方法是隐式的final?

类中所有private方法都隐式地指定为final的，由于无法取用private方法，所以也就不能覆盖它。可以对private方法增添final关键字，但这样做并没有什么好处。看下下面的例子：

```java
public class Base {
    private void test() {
    }
}

public class Son extends Base{
    public void test() {
    }
    public static void main(String[] args) {
        Son son = new Son();
        Base father = son;
        //father.test();
    }
}
```

Base和Son都有方法test(),但是这并不是一种覆盖，因为private所修饰的方法是隐式的final，也就是无法被继承，所以更不用说是覆盖了，在Son中的test()方法不过是属于Son的新成员罢了，Son进行向上转型得到father，但是father.test()是不可执行的，因为Base中的test方法是private的，无法被访问到。

#### [#](#说说final类型的类如何拓展) 说说final类型的类如何拓展?

比如String是final类型，我们想写个MyString复用所有String中方法，同时增加一个新的toMyString()的方法，应该如何做?

外观模式：

```java
/**
* @pdai
*/
class MyString{

    private String innerString;

    // ...init & other methods

    // 支持老的方法
    public int length(){
        return innerString.length(); // 通过innerString调用老的方法
    }

    // 添加新方法
    public String toMyString(){
        //...
    }
}
```

#### [#](#final方法可以被重载吗) final方法可以被重载吗?

我们知道父类的final方法是不能够被子类重写的，那么final方法可以被重载吗? 答案是可以的，下面代码是正确的。

```java
public class FinalExampleParent {
    public final void test() {
    }

    public final void test(String str) {
    }
}
```

#### [#](#父类的final方法能不能够被子类重写) 父类的final方法能不能够被子类重写?

不可以

#### [#](#说说基本类型的final域重排序规则) 说说基本类型的final域重排序规则?

先看一段示例性的代码：

```java
public class FinalDemo {
    private int a;  //普通域
    private final int b; //final域
    private static FinalDemo finalDemo;

    public FinalDemo() {
        a = 1; // 1. 写普通域
        b = 2; // 2. 写final域
    }

    public static void writer() {
        finalDemo = new FinalDemo();
    }

    public static void reader() {
        FinalDemo demo = finalDemo; // 3.读对象引用
        int a = demo.a;    //4.读普通域
        int b = demo.b;    //5.读final域
    }
}
```

假设线程A在执行writer()方法，线程B执行reader()方法。

- **写final域重排序规则**

写final域的重排序规则禁止对final域的写重排序到构造函数之外，这个规则的实现主要包含了两个方面：

- JMM禁止编译器把final域的写重排序到构造函数之外；
- 编译器会在final域写之后，构造函数return之前，插入一个storestore屏障。这个屏障可以禁止处理器把final域的写重排序到构造函数之外。

我们再来分析writer方法，虽然只有一行代码，但实际上做了两件事情：

- 构造了一个FinalDemo对象；
- 把这个对象赋值给成员变量finalDemo。

我们来画下存在的一种可能执行时序图，如下：

![img](/images/thread/java-thread-x-key-final-1.png)

由于a,b之间没有数据依赖性，普通域(普通变量)a可能会被重排序到构造函数之外，线程B就有可能读到的是普通变量a初始化之前的值(零值)，这样就可能出现错误。而final域变量b，根据重排序规则，会禁止final修饰的变量b重排序到构造函数之外，从而b能够正确赋值，线程B就能够读到final变量初始化后的值。

因此，写final域的重排序规则可以确保：在对象引用为任意线程可见之前，对象的final域已经被正确初始化过了，而普通域就不具有这个保障。比如在上例，线程B有可能就是一个未正确初始化的对象finalDemo。

- **读final域重排序规则**

读final域重排序规则为：在一个线程中，初次读对象引用和初次读该对象包含的final域，JMM会禁止这两个操作的重排序。(注意，这个规则仅仅是针对处理器)，处理器会在读final域操作的前面插入一个LoadLoad屏障。实际上，读对象的引用和读该对象的final域存在间接依赖性，一般处理器不会重排序这两个操作。但是有一些处理器会重排序，因此，这条禁止重排序规则就是针对这些处理器而设定的。

read()方法主要包含了三个操作：

- 初次读引用变量finalDemo;
- 初次读引用变量finalDemo的普通域a;
- 初次读引用变量finalDemo的final域b;

假设线程A写过程没有重排序，那么线程A和线程B有一种的可能执行时序为下图：

![img](/images/thread/java-thread-x-key-final-2.png)

读对象的普通域被重排序到了读对象引用的前面就会出现线程B还未读到对象引用就在读取该对象的普通域变量，这显然是错误的操作。而final域的读操作就“限定”了在读final域变量前已经读到了该对象的引用，从而就可以避免这种情况。

读final域的重排序规则可以确保：在读一个对象的final域之前，一定会先读这个包含这个final域的对象的引用。

#### [#](#说说final的原理) 说说final的原理?

- 写final域会要求编译器在final域写之后，构造函数返回前插入一个StoreStore屏障。
- 读final域的重排序规则会要求编译器在读final域的操作前插入一个LoadLoad屏障。

PS:很有意思的是，如果以X86处理为例，X86不会对写-写重排序，所以StoreStore屏障可以省略。由于不会对有间接依赖性的操作重排序，所以在X86处理器中，读final域需要的LoadLoad屏障也会被省略掉。也就是说，以X86为例的话，对final域的读/写的内存屏障都会被省略！具体是否插入还是得看是什么处理器。

### [#](#_3-3-juc全局观) 3.3 JUC全局观

- [JUC - 类汇总和学习指南]()

#### [#](#juc框架包含几个部分) JUC框架包含几个部分?

五个部分：

![image](https://pdai.tech/images/thread/java-thread-x-juc-overview-1-u.png)

主要包含: (注意: 上图是网上找的图，无法表述一些继承关系，同时少了部分类；但是主体上可以看出其分类关系也够了)

- Lock框架和Tools类(把图中这两个放到一起理解)
- Collections: 并发集合
- Atomic: 原子类
- Executors: 线程池

#### [#](#lock框架和tools哪些核心的类) Lock框架和Tools哪些核心的类?

![image](https://pdai.tech/images/thread/java-thread-x-juc-overview-lock.png)

- **接口: Condition**， Condition为接口类型，它将 Object 监视器方法(wait、notify 和 notifyAll)分解成截然不同的对象，以便通过将这些对象与任意 Lock 实现组合使用，为每个对象提供多个等待 set (wait-set)。其中，Lock 替代了 synchronized 方法和语句的使用，Condition 替代了 Object 监视器方法的使用。可以通过await(),signal()来休眠/唤醒线程。
- **接口: Lock**，Lock为接口类型，Lock实现提供了比使用synchronized方法和语句可获得的更广泛的锁定操作。此实现允许更灵活的结构，可以具有差别很大的属性，可以支持多个相关的Condition对象。
- **接口ReadWriteLock** ReadWriteLock为接口类型， 维护了一对相关的锁，一个用于只读操作，另一个用于写入操作。只要没有 writer，读取锁可以由多个 reader 线程同时保持。写入锁是独占的。
- **抽象类: AbstractOwnableSynchonizer** AbstractOwnableSynchonizer为抽象类，可以由线程以独占方式拥有的同步器。此类为创建锁和相关同步器(伴随着所有权的概念)提供了基础。AbstractOwnableSynchronizer 类本身不管理或使用此信息。但是，子类和工具可以使用适当维护的值帮助控制和监视访问以及提供诊断。
- **抽象类(long): AbstractQueuedLongSynchronizer** AbstractQueuedLongSynchronizer为抽象类，以 long 形式维护同步状态的一个 AbstractQueuedSynchronizer 版本。此类具有的结构、属性和方法与 AbstractQueuedSynchronizer 完全相同，但所有与状态相关的参数和结果都定义为 long 而不是 int。当创建需要 64 位状态的多级别锁和屏障等同步器时，此类很有用。
- **核心抽象类(int): AbstractQueuedSynchronizer** AbstractQueuedSynchronizer为抽象类，其为实现依赖于先进先出 (FIFO) 等待队列的阻塞锁和相关同步器(信号量、事件，等等)提供一个框架。此类的设计目标是成为依靠单个原子 int 值来表示状态的大多数同步器的一个有用基础。
- **锁常用类: LockSupport** LockSupport为常用类，用来创建锁和其他同步类的基本线程阻塞原语。LockSupport的功能和"Thread中的 Thread.suspend()和Thread.resume()有点类似"，LockSupport中的park() 和 unpark() 的作用分别是阻塞线程和解除阻塞线程。但是park()和unpark()不会遇到“Thread.suspend 和 Thread.resume所可能引发的死锁”问题。
- **锁常用类: ReentrantLock** ReentrantLock为常用类，它是一个可重入的互斥锁 Lock，它具有与使用 synchronized 方法和语句所访问的隐式监视器锁相同的一些基本行为和语义，但功能更强大。
- **锁常用类: ReentrantReadWriteLock** ReentrantReadWriteLock是读写锁接口ReadWriteLock的实现类，它包括Lock子类ReadLock和WriteLock。ReadLock是共享锁，WriteLock是独占锁。
- **锁常用类: StampedLock** 它是java8在java.util.concurrent.locks新增的一个API。StampedLock控制锁有三种模式(写，读，乐观读)，一个StampedLock状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。
- **工具常用类: CountDownLatch** CountDownLatch为常用类，它是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
- **工具常用类: CyclicBarrier** CyclicBarrier为常用类，其是一个同步辅助类，它允许一组线程互相等待，直到到达某个公共屏障点 (common barrier point)。在涉及一组固定大小的线程的程序中，这些线程必须不时地互相等待，此时 CyclicBarrier 很有用。因为该 barrier 在释放等待线程后可以重用，所以称它为循环 的 barrier。
- **工具常用类: Phaser** Phaser是JDK 7新增的一个同步辅助类，它可以实现CyclicBarrier和CountDownLatch类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。
- **工具常用类: Semaphore** Semaphore为常用类，其是一个计数信号量，从概念上讲，信号量维护了一个许可集。如有必要，在许可可用前会阻塞每一个 acquire()，然后再获取该许可。每个 release() 添加一个许可，从而可能释放一个正在阻塞的获取者。但是，不使用实际的许可对象，Semaphore 只对可用许可的号码进行计数，并采取相应的行动。通常用于限制可以访问某些资源(物理或逻辑的)的线程数目。
- **工具常用类: Exchanger** Exchanger是用于线程协作的工具类, 主要用于两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange()方法交换数据，当一个线程先执行exchange()方法后，它会一直等待第二个线程也执行exchange()方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

#### [#](#juc并发集合哪些核心的类) JUC并发集合哪些核心的类?

![image](https://pdai.tech/images/thread/java-thread-x-juc-overview-2.png)

- **Queue: ArrayBlockingQueue** 一个由数组支持的有界阻塞队列。此队列按 FIFO(先进先出)原则对元素进行排序。队列的头部 是在队列中存在时间最长的元素。队列的尾部 是在队列中存在时间最短的元素。新元素插入到队列的尾部，队列获取操作则是从队列头部开始获得元素。
- **Queue: LinkedBlockingQueue** 一个基于已链接节点的、范围任意的 blocking queue。此队列按 FIFO(先进先出)排序元素。队列的头部 是在队列中时间最长的元素。队列的尾部 是在队列中时间最短的元素。新元素插入到队列的尾部，并且队列获取操作会获得位于队列头部的元素。链接队列的吞吐量通常要高于基于数组的队列，但是在大多数并发应用程序中，其可预知的性能要低。
- **Queue: LinkedBlockingDeque** 一个基于已链接节点的、任选范围的阻塞双端队列。
- **Queue: ConcurrentLinkedQueue** 一个基于链接节点的无界线程安全队列。此队列按照 FIFO(先进先出)原则对元素进行排序。队列的头部 是队列中时间最长的元素。队列的尾部 是队列中时间最短的元素。新的元素插入到队列的尾部，队列获取操作从队列头部获得元素。当多个线程共享访问一个公共 collection 时，ConcurrentLinkedQueue 是一个恰当的选择。此队列不允许使用 null 元素。
- **Queue: ConcurrentLinkedDeque** 是双向链表实现的无界队列，该队列同时支持FIFO和FILO两种操作方式。
- **Queue: DelayQueue** 延时无界阻塞队列，使用Lock机制实现并发访问。队列里只允许放可以“延期”的元素，队列中的head是最先“到期”的元素。如果队里中没有元素到“到期”，那么就算队列中有元素也不能获取到。
- **Queue: PriorityBlockingQueue** 无界优先级阻塞队列，使用Lock机制实现并发访问。priorityQueue的线程安全版，不允许存放null值，依赖于comparable的排序，不允许存放不可比较的对象类型。
- **Queue: SynchronousQueue** 没有容量的同步队列，通过CAS实现并发访问，支持FIFO和FILO。
- **Queue: LinkedTransferQueue** JDK 7新增，单向链表实现的无界阻塞队列，通过CAS实现并发访问，队列元素使用 FIFO(先进先出)方式。LinkedTransferQueue可以说是ConcurrentLinkedQueue、SynchronousQueue(公平模式)和LinkedBlockingQueue的超集, 它不仅仅综合了这几个类的功能，同时也提供了更高效的实现。
- **List: CopyOnWriteArrayList** ArrayList 的一个线程安全的变体，其中所有可变操作(add、set 等等)都是通过对底层数组进行一次新的复制来实现的。这一般需要很大的开销，但是当遍历操作的数量大大超过可变操作的数量时，这种方法可能比其他替代方法更 有效。在不能或不想进行同步遍历，但又需要从并发线程中排除冲突时，它也很有用。
- **Set: CopyOnWriteArraySet** 对其所有操作使用内部CopyOnWriteArrayList的Set。即将所有操作转发至CopyOnWriteArayList来进行操作，能够保证线程安全。在add时，会调用addIfAbsent，由于每次add时都要进行数组遍历，因此性能会略低于CopyOnWriteArrayList。
- **Set: ConcurrentSkipListSet** 一个基于ConcurrentSkipListMap 的可缩放并发 NavigableSet 实现。set 的元素可以根据它们的自然顺序进行排序，也可以根据创建 set 时所提供的 Comparator 进行排序，具体取决于使用的构造方法。
- **Map: ConcurrentHashMap** 是线程安全HashMap的。ConcurrentHashMap在JDK 7之前是通过Lock和segment(分段锁)实现，JDK 8 之后改为CAS+synchronized来保证并发安全。
- **Map: ConcurrentSkipListMap** 线程安全的有序的哈希表(相当于线程安全的TreeMap);映射可以根据键的自然顺序进行排序，也可以根据创建映射时所提供的 Comparator 进行排序，具体取决于使用的构造方法。

#### [#](#juc原子类哪些核心的类) JUC原子类哪些核心的类?

其基本的特性就是在多线程环境下，当有多个线程同时执行这些类的实例包含的方法时，具有排他性，即当某个线程进入方法，执行其中的指令时，不会被其他线程打断，而别的线程就像自旋锁一样，一直等到该方法执行完成，才由JVM从等待队列中选择一个另一个线程进入，这只是一种逻辑上的理解。实际上是借助硬件的相关指令来实现的，不会阻塞线程(或者说只是在硬件级别上阻塞了)。

- 原子更新基本类型
  - AtomicBoolean: 原子更新布尔类型。
  - AtomicInteger: 原子更新整型。
  - AtomicLong: 原子更新长整型。
- 原子更新数组
  - AtomicIntegerArray: 原子更新整型数组里的元素。
  - AtomicLongArray: 原子更新长整型数组里的元素。
  - AtomicReferenceArray: 原子更新引用类型数组里的元素。
- 原子更新引用类型
  - AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
  - AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
  - AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。
  - AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述
- 原子更新字段类
  - AtomicReference: 原子更新引用类型。
  - AtomicStampedReference: 原子更新引用类型, 内部使用Pair来存储元素值及其版本号。
  - AtomicMarkableReferce: 原子更新带有标记位的引用类型。

#### [#](#juc线程池哪些核心的类) JUC线程池哪些核心的类?

![img](/images/thread/java-thread-x-juc-executors-1.png)

- **接口: Executor** Executor接口提供一种将任务提交与每个任务将如何运行的机制(包括线程使用的细节、调度等)分离开来的方法。通常使用 Executor 而不是显式地创建线程。
- **ExecutorService** ExecutorService继承自Executor接口，ExecutorService提供了管理终止的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以关闭 ExecutorService，这将导致其停止接受新任务。关闭后，执行程序将最后终止，这时没有任务在执行，也没有任务在等待执行，并且无法提交新任务。
- **ScheduledExecutorService** ScheduledExecutorService继承自ExecutorService接口，可安排在给定的延迟后运行或定期执行的命令。
- **AbstractExecutorService** AbstractExecutorService继承自ExecutorService接口，其提供 ExecutorService 执行方法的默认实现。此类使用 newTaskFor 返回的 RunnableFuture 实现 submit、invokeAny 和 invokeAll 方法，默认情况下，RunnableFuture 是此包中提供的 FutureTask 类。
- **FutureTask** FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由CAS来保证。
- **核心: ThreadPoolExecutor** ThreadPoolExecutor实现了AbstractExecutorService接口，也是一个 ExecutorService，它使用可能的几个池线程之一执行每个提交的任务，通常使用 Executors 工厂方法配置。 线程池可以解决两个不同问题: 由于减少了每个任务调用的开销，它们通常可以在执行大量异步任务时提供增强的性能，并且还可以提供绑定和管理资源(包括执行任务集时使用的线程)的方法。每个 ThreadPoolExecutor 还维护着一些基本的统计数据，如完成的任务数。
- **核心: ScheduledThreadExecutor** ScheduledThreadPoolExecutor实现ScheduledExecutorService接口，可安排在给定的延迟后运行命令，或者定期执行命令。需要多个辅助线程时，或者要求 ThreadPoolExecutor 具有额外的灵活性或功能时，此类要优于 Timer。
- **核心: Fork/Join框架** ForkJoinPool 是JDK 7加入的一个线程池类。Fork/Join 技术是分治算法(Divide-and-Conquer)的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。
- **工具类: Executors** Executors是一个工具类，用其可以创建ExecutorService、ScheduledExecutorService、ThreadFactory、Callable等对象。它的使用融入到了ThreadPoolExecutor, ScheduledThreadExecutor和ForkJoinPool中。

### [#](#_3-4-juc原子类) 3.4 JUC原子类

> 需要加锁就在多线程并发场景下实现数据的一致性

- [JUC原子类: CAS, Unsafe和原子类详解]()

#### [#](#线程安全的实现方法有哪些) 线程安全的实现方法有哪些?

线程安全的实现方法包含:

- 互斥同步: synchronized 和 ReentrantLock
- 非阻塞同步: CAS, AtomicXXXX
- 无同步方案: 栈封闭，Thread Local，可重入代码

#### [#](#什么是cas) 什么是CAS?

CAS的全称为Compare-And-Swap，直译就是对比交换。是一条CPU的原子指令，其作用是让CPU先进行比较两个值是否相等，然后原子地更新某个位置的值，经过调查发现，其实现方式是基于硬件平台的汇编指令，就是说CAS是靠硬件实现的，JVM只是封装了汇编调用，那些AtomicInteger类便是使用了这些封装后的接口。   简单解释：CAS操作需要输入两个数值，一个旧值(期望操作前的值)和一个新值，在操作期间先比较下在旧值有没有发生变化，如果没有发生变化，才交换成新值，发生了变化则不交换。

CAS操作是原子性的，所以多线程并发使用CAS更新数据时，可以不使用锁。JDK中大量使用了CAS来更新数据而防止加锁(synchronized 重量级锁)来保持原子更新。

相信sql大家都熟悉，类似sql中的条件更新一样：update set id=3 from table where id=2。因为单条sql执行具有原子性，如果有多个线程同时执行此sql语句，只有一条能更新成功。

#### [#](#cas使用示例-结合atomicinteger给出示例) CAS使用示例，结合AtomicInteger给出示例?

如果不使用CAS，在高并发下，多线程同时修改一个变量的值我们需要synchronized加锁(可能有人说可以用Lock加锁，Lock底层的AQS也是基于CAS进行获取锁的)。

```java
public class Test {
    private int i=0;
    public synchronized int add(){
        return i++;
    }
}
```

java中为我们提供了AtomicInteger 原子类(底层基于CAS进行更新数据的)，不需要加锁就在多线程并发场景下实现数据的一致性。

```java
public class Test {
    private  AtomicInteger i = new AtomicInteger(0);
    public int add(){
        return i.addAndGet(1);
    }
}
```

#### [#](#cas会有哪些问题) CAS会有哪些问题?

CAS 方式为乐观锁，synchronized 为悲观锁。因此使用 CAS 解决并发问题通常情况下性能更优。

但使用 CAS 方式也会有几个问题：

- ABA问题

因为CAS需要在操作值的时候，检查值有没有发生变化，比如没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用CAS进行检查时则会发现它的值没有发生变化，但是实际上却变化了。

ABA问题的解决思路就是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加1，那么A->B->A就会变成1A->2B->3A。

从Java 1.5开始，JDK的Atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法的作用是首先检查当前引用是否等于预期引用，并且检查当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

- 循环时间长开销大

自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令，那么效率会有一定的提升。pause指令有两个作用：第一，它可以延迟流水线执行命令(de-pipeline)，使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零；第二，它可以避免在退出循环的时候因内存顺序冲突(Memory Order Violation)而引起CPU流水线被清空(CPU Pipeline Flush)，从而提高CPU的执行效率。

- 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁。

还有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如，有两个共享变量i = 2，j = a，合并一下ij = 2a，然后用CAS来操作ij。

从Java 1.5开始，JDK提供了AtomicReference类来保证引用对象之间的原子性，就可以把多个变量放在一个对象里来进行CAS操作。

#### [#](#atomicinteger底层实现) AtomicInteger底层实现?

- CAS+volatile
- volatile保证线程的可见性，多线程并发时，一个线程修改数据，可以保证其它线程立马看到修改后的值CAS 保证数据更新的原子性。

#### [#](#请阐述你对unsafe类的理解) 请阐述你对Unsafe类的理解?

UnSafe类总体功能：

![img](/images/thread/java-thread-x-atomicinteger-unsafe.png)

如上图所示，Unsafe提供的API大致可分为内存操作、CAS、Class相关、对象操作、线程调度、系统信息获取、内存屏障、数组操作等几类，下面将对其相关方法和应用场景进行详细介绍。

#### [#](#说说你对java原子类的理解) 说说你对Java原子类的理解?

包含13个，4组分类，说说作用和使用场景。

- 原子更新基本类型 
  - AtomicBoolean: 原子更新布尔类型。
  - AtomicInteger: 原子更新整型。
  - AtomicLong: 原子更新长整型。
- 原子更新数组 
  - AtomicIntegerArray: 原子更新整型数组里的元素。
  - AtomicLongArray: 原子更新长整型数组里的元素。
  - AtomicReferenceArray: 原子更新引用类型数组里的元素。
- 原子更新引用类型 
  - AtomicIntegerFieldUpdater: 原子更新整型的字段的更新器。
  - AtomicLongFieldUpdater: 原子更新长整型字段的更新器。
  - AtomicStampedFieldUpdater: 原子更新带有版本号的引用类型。
  - AtomicReferenceFieldUpdater: 上面已经说过此处不在赘述
- 原子更新字段类 
  - AtomicReference: 原子更新引用类型。
  - AtomicStampedReference: 原子更新引用类型, 内部使用Pair来存储元素值及其版本号。
  - AtomicMarkableReferce: 原子更新带有标记位的引用类型。

#### [#](#atomicstampedreference是怎么解决aba的) AtomicStampedReference是怎么解决ABA的?

AtomicStampedReference主要维护包含一个对象引用以及一个可以自动更新的整数"stamp"的pair对象来解决ABA问题。

### [#](#_3-5-juc锁) 3.5 JUC锁

- [JUC锁: LockSupport详解]()
- [JUC锁: 锁核心类AQS详解]()
- [JUC锁: ReentrantReadWriteLock详解]()

#### [#](#为什么locksupport也是核心基础类) 为什么LockSupport也是核心基础类?

AQS框架借助于两个类：Unsafe(提供CAS操作)和LockSupport(提供park/unpark操作)

#### [#](#通过wait-notify实现同步) 通过wait/notify实现同步?

```java
class MyThread extends Thread {
    
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();            
        synchronized (myThread) {
            try {        
                myThread.start();
                // 主线程睡眠3s
                Thread.sleep(3000);
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果

```html
before wait
before notify
after notify
after wait
```

说明: 具体的流程图如下

![img](/images/thread/java-thread-x-locksupport-1.png)

使用wait/notify实现同步时，必须先调用wait，后调用notify，如果先调用notify，再调用wait，将起不了作用。具体代码如下

```java
class MyThread extends Thread {
    public void run() {
        synchronized (this) {
            System.out.println("before notify");            
            notify();
            System.out.println("after notify");    
        }
    }
}

public class WaitAndNotifyDemo {
    public static void main(String[] args) throws InterruptedException {
        MyThread myThread = new MyThread();        
        myThread.start();
        // 主线程睡眠3s
        Thread.sleep(3000);
        synchronized (myThread) {
            try {        
                System.out.println("before wait");
                // 阻塞主线程
                myThread.wait();
                System.out.println("after wait");
            } catch (InterruptedException e) {
                e.printStackTrace();
            }            
        }        
    }
}
```

运行结果:

```html
before notify
after notify
before wait
```

说明: 由于先调用了notify，再调用的wait，此时主线程还是会一直阻塞。

#### [#](#通过locksupport的park-unpark实现同步) 通过LockSupport的park/unpark实现同步？

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
        // 释放许可
        LockSupport.unpark((Thread) object);
        // 休眠500ms，保证先执行park中的setBlocker(t, null);
        try {
            Thread.sleep(500);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        // 再次获取blocker
        System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

        System.out.println("after unpark");
    }
}

public class test {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

运行结果:

```html
before park
before unpark
Blocker info ParkAndUnparkDemo
after park
Blocker info null
after unpark
```

说明: 本程序先执行park，然后在执行unpark，进行同步，并且在unpark的前后都调用了getBlocker，可以看到两次的结果不一样，并且第二次调用的结果为null，这是因为在调用unpark之后，执行了Lock.park(Object blocker)函数中的setBlocker(t, null)函数，所以第二次调用getBlocker时为null。

上例是先调用park，然后调用unpark，现在修改程序，先调用unpark，然后调用park，看能不能正确同步。具体代码如下

```java
import java.util.concurrent.locks.LockSupport;

class MyThread extends Thread {
    private Object object;

    public MyThread(Object object) {
        this.object = object;
    }

    public void run() {
        System.out.println("before unpark");        
        // 释放许可
        LockSupport.unpark((Thread) object);
        System.out.println("after unpark");
    }
}

public class ParkAndUnparkDemo {
    public static void main(String[] args) {
        MyThread myThread = new MyThread(Thread.currentThread());
        myThread.start();
        try {
            // 主线程睡眠3s
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("before park");
        // 获取许可
        LockSupport.park("ParkAndUnparkDemo");
        System.out.println("after park");
    }
}
```

运行结果:

```html
before unpark
after unpark
before park
after park
```

说明: 可以看到，在先调用unpark，再调用park时，仍能够正确实现同步，不会造成由wait/notify调用顺序不当所引起的阻塞。因此park/unpark相比wait/notify更加的灵活。

#### [#](#thread-sleep-、object-wait-、condition-await-、locksupport-park-的区别-重点) Thread.sleep()、Object.wait()、Condition.await()、LockSupport.park()的区别? 重点

- **Thread.sleep()和Object.wait()的区别**

首先，我们先来看看Thread.sleep()和Object.wait()的区别，这是一个烂大街的题目了，大家应该都能说上来两点。

1. Thread.sleep()不会释放占有的锁，Object.wait()会释放占有的锁；
2. Thread.sleep()必须传入时间，Object.wait()可传可不传，不传表示一直阻塞下去；
3. Thread.sleep()到时间了会自动唤醒，然后继续执行；
4. Object.wait()不带时间的，需要另一个线程使用Object.notify()唤醒；
5. Object.wait()带时间的，假如没有被notify，到时间了会自动唤醒，这时又分好两种情况，一是立即获取到了锁，线程自然会继续执行；二是没有立即获取锁，线程进入同步队列等待获取锁；

其实，他们俩最大的区别就是Thread.sleep()不会释放锁资源，Object.wait()会释放锁资源。

- **Object.wait()和Condition.await()的区别**

Object.wait()和Condition.await()的原理是基本一致的，不同的是Condition.await()底层是调用LockSupport.park()来实现阻塞当前线程的。

实际上，它在阻塞当前线程之前还干了两件事，一是把当前线程添加到条件队列中，二是“完全”释放锁，也就是让state状态变量变为0，然后才是调用LockSupport.park()阻塞当前线程。

- **Thread.sleep()和LockSupport.park()的区别** LockSupport.park()还有几个兄弟方法——parkNanos()、parkUtil()等，我们这里说的park()方法统称这一类方法。

1. 从功能上来说，Thread.sleep()和LockSupport.park()方法类似，都是阻塞当前线程的执行，且都不会释放当前线程占有的锁资源；
2. Thread.sleep()没法从外部唤醒，只能自己醒过来；
3. LockSupport.park()方法可以被另一个线程调用LockSupport.unpark()方法唤醒；
4. Thread.sleep()方法声明上抛出了InterruptedException中断异常，所以调用者需要捕获这个异常或者再抛出；
5. LockSupport.park()方法不需要捕获中断异常；
6. Thread.sleep()本身就是一个native方法；
7. LockSupport.park()底层是调用的Unsafe的native方法；

- **Object.wait()和LockSupport.park()的区别**

二者都会阻塞当前线程的运行，他们有什么区别呢? 经过上面的分析相信你一定很清楚了，真的吗? 往下看！

1. Object.wait()方法需要在synchronized块中执行；
2. LockSupport.park()可以在任意地方执行；
3. Object.wait()方法声明抛出了中断异常，调用者需要捕获或者再抛出；
4. LockSupport.park()不需要捕获中断异常；
5. Object.wait()不带超时的，需要另一个线程执行notify()来唤醒，但不一定继续执行后续内容；
6. LockSupport.park()不带超时的，需要另一个线程执行unpark()来唤醒，一定会继续执行后续内容；

park()/unpark()底层的原理是“二元信号量”，你可以把它相像成只有一个许可证的Semaphore，只不过这个信号量在重复执行unpark()的时候也不会再增加许可证，最多只有一个许可证。

#### [#](#如果在wait-之前执行了notify-会怎样) 如果在wait()之前执行了notify()会怎样?

如果当前的线程不是此对象锁的所有者，却调用该对象的notify()或wait()方法时抛出IllegalMonitorStateException异常；

如果当前线程是此对象锁的所有者，wait()将一直阻塞，因为后续将没有其它notify()唤醒它。

#### [#](#如果在park-之前执行了unpark-会怎样) 如果在park()之前执行了unpark()会怎样?

线程不会被阻塞，直接跳过park()，继续执行后续内容

#### [#](#什么是aqs-为什么它是核心) 什么是AQS? 为什么它是核心?

AQS是一个用来构建锁和同步器的框架，使用AQS能简单且高效地构造出应用广泛的大量的同步器，比如我们提到的ReentrantLock，Semaphore，其他的诸如ReentrantReadWriteLock，SynchronousQueue，FutureTask等等皆是基于AQS的。

AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

AbstractQueuedSynchronizer类底层的数据结构是使用**CLH(Craig,Landin,and Hagersten)队列**是一个虚拟的双向队列(虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系)。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点(Node)来实现锁的分配。其中Sync queue，即同步队列，是双向链表，包括head结点和tail结点，head结点主要用作后续的调度。而Condition queue不是必须的，其是一个单向链表，只有当使用Condition时，才会存在此单向链表。并且可能会有多个Condition queue。

![image](/images/thread/java-thread-x-juc-aqs-1.png)

#### [#](#aqs的核心思想是什么) AQS的核心思想是什么?

底层数据结构: AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。

#### [#](#aqs有哪些核心的方法) AQS有哪些核心的方法?

```java
isHeldExclusively()//该线程是否正在独占资源。只有用到condition才需要去实现它。
tryAcquire(int)//独占方式。尝试获取资源，成功则返回true，失败则返回false。
tryRelease(int)//独占方式。尝试释放资源，成功则返回true，失败则返回false。
tryAcquireShared(int)//共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
tryReleaseShared(int)//共享方式。尝试释放资源，成功则返回true，失败则返回false。
```

#### [#](#aqs定义什么样的资源获取方式) AQS定义什么样的资源获取方式?

AQS定义了两种资源获取方式：

- **独占**(只有一个线程能访问执行，又根据是否按队列的顺序分为**公平锁**和**非公平锁**，如`ReentrantLock`)
- **共享**(多个线程可同时访问执行，如`Semaphore`、`CountDownLatch`、 `CyclicBarrier` )。`ReentrantReadWriteLock`可以看成是组合式，允许多个线程同时对某一资源进行读。

#### [#](#aqs底层使用了什么样的设计模式) AQS底层使用了什么样的设计模式?

模板， 共享锁和独占锁在一个接口类中。

- [JUC锁: ReentrantLock详解]()

#### [#](#什么是可重入-什么是可重入锁-它用来解决什么问题) 什么是可重入，什么是可重入锁? 它用来解决什么问题?

**可重入**：（来源于维基百科）若一个程序或子程序可以“在任意时刻被中断然后操作系统调度执行另外一段代码，这段代码又调用了该子程序不会出错”，则称其为可重入（reentrant或re-entrant）的。即当该子程序正在运行时，执行线程可以再次进入并执行它，仍然获得符合设计时预期的结果。与多线程并发执行的线程安全不同，可重入强调对单个线程执行时重新进入同一个子程序仍然是安全的。

**可重入锁**：又名递归锁，是指在同一个线程在外层方法获取锁的时候，再进入该线程的内层方法会自动获取锁（前提锁对象得是同一个对象或者class），不会因为之前已经获取过还没释放而阻塞。

#### [#](#reentrantlock的核心是aqs-那么它怎么来实现的-继承吗) ReentrantLock的核心是AQS，那么它怎么来实现的，继承吗?

ReentrantLock总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

![image](https://pdai.tech/images/thread/java-thread-x-juc-reentrantlock-1.png)

说明: ReentrantLock类内部总共存在Sync、NonfairSync、FairSync三个类，NonfairSync与FairSync类继承自Sync类，Sync类继承自AbstractQueuedSynchronizer抽象类。下面逐个进行分析。





```
ReentrantLock 的实现原理涉及到 Java 的底层机制，包括 AbstractQueuedSynchronizer（AQS）类和 CAS（Compare and Swap）操作。ReentrantLock 是基于 AQS 构建的，AQS 是一个提供了底层同步机制的框架，它是许多并发工具的基础。

以下是 ReentrantLock 的简要实现原理：

AQS 原理： AbstractQueuedSynchronizer 是一个提供了队列式同步机制的基类。它维护了一个双向队列，用于存储等待获取锁的线程。AQS 使用 CAS 操作来修改状态（state），通过状态来表示锁的获取状态和线程等待状态。

ReentrantLock 的状态： 在 ReentrantLock 中，每个锁对象都关联着一个 AQS 实例，通过状态值来表示锁的重入次数和持有线程等信息。当锁被一个线程获取时，状态会递增，当锁被释放时，状态递减。当状态为 0 时，表示锁没有被任何线程占用。

lock() 操作： 当一个线程调用 lock() 方法尝试获取锁时，如果锁没有被其他线程占用，它会成功获取锁并将状态置为 1。如果锁已经被当前线程占用，它会增加状态的值，并且记录重入的次数。

unlock() 操作： 当一个线程调用 unlock() 方法释放锁时，它会将状态递减。如果状态为 0，表示锁已完全释放；如果状态大于 0，表示锁被当前线程重入，不会真正释放。

线程等待和唤醒： 当一个线程无法获取锁时，它会被放入 AQS 的等待队列中。在释放锁时，ReentrantLock 会通知等待队列中的线程，让其中的某个线程继续获取锁。

总之，ReentrantLock 的实现依赖于 AbstractQueuedSynchronizer 的队列式同步机制，通过状态值来表示锁的状态和线程重入次数。它使用 CAS 操作来确保状态的原子性操作，并且通过等待队列来管理线程的等待和唤醒。这种基于 AQS 的实现使得 ReentrantLock 可以支持可重入性、公平性等高级特性。
```



#### [#](#reentrantlock是如何实现公平锁的) ReentrantLock是如何实现公平锁的?

FairSync

#### [#](#reentrantlock是如何实现非公平锁的) ReentrantLock是如何实现非公平锁的?

UnFairSync

#### [#](#reentrantlock默认实现的是公平还是非公平锁) ReentrantLock默认实现的是公平还是非公平锁?

非公平锁

#### [#](#为了有了reentrantlock还需要reentrantreadwritelock) 为了有了ReentrantLock还需要ReentrantReadWriteLock?

读锁和写锁分离：ReentrantReadWriteLock表示可重入读写锁，ReentrantReadWriteLock中包含了两种锁，读锁ReadLock和写锁WriteLock，可以通过这两种锁实现线程间的同步。

#### [#](#reentrantreadwritelock底层实现原理) ReentrantReadWriteLock底层实现原理?

ReentrantReadWriteLock有五个内部类，五个内部类之间也是相互关联的。内部类的关系如下图所示。

![img](https://pdai.tech/images/thread/java-thread-x-readwritelock-1.png)

说明: 如上图所示，Sync继承自AQS、NonfairSync继承自Sync类、FairSync继承自Sync类；ReadLock实现了Lock接口、WriteLock也实现了Lock接口。

#### [#](#reentrantreadwritelock底层读写状态如何设计的) ReentrantReadWriteLock底层读写状态如何设计的?

高16位为读锁，低16位为写锁

#### [#](#读锁和写锁的最大数量是多少) 读锁和写锁的最大数量是多少?

2的16次方-1

#### [#](#本地线程计数器threadlocalholdcounter是用来做什么的) 本地线程计数器ThreadLocalHoldCounter是用来做什么的?

本地线程计数器，与对象绑定（线程-》线程重入的次数）

#### [#](#写锁的获取与释放是怎么实现的) 写锁的获取与释放是怎么实现的?

tryAcquire/tryRelease

#### [#](#读锁的获取与释放是怎么实现的) 读锁的获取与释放是怎么实现的?

tryAcquireShared/tryReleaseShared

#### [#](#什么是锁的升降级) 什么是锁的升降级?

RentrantReadWriteLock为什么不支持锁升级? RentrantReadWriteLock不支持锁升级(把持读锁、获取写锁，最后释放读锁的过程)。目的也是保证数据可见性，如果读锁已被多个线程获取，其中任意线程成功获取了写锁并更新了数据，则其更新对其他获取到读锁的线程是不可见的。

### [#](#_3-6-juc集合类) 3.6 JUC集合类

- [JUC集合: ConcurrentHashMap详解]()
- [JUC集合: CopyOnWriteArrayList详解]()
- [JUC集合: ConcurrentLinkedQueue详解]()
- [JUC集合: BlockingQueue详解]()

#### [#](#为什么hashtable慢-它的并发度是什么-那么concurrenthashmap并发度是什么) 为什么HashTable慢? 它的并发度是什么? 那么ConcurrentHashMap并发度是什么?

Hashtable之所以效率低下主要是因为其实现使用了synchronized关键字对put等操作进行加锁，而synchronized关键字加锁是对整个对象进行加锁，也就是说在进行put等修改Hash表的操作时，锁住了整个Hash表，从而使得其表现的效率低下。

#### [#](#concurrenthashmap在jdk1-7和jdk1-8中实现有什么差别-jdk1-8解決了jdk1-7中什么问题) ConcurrentHashMap在JDK1.7和JDK1.8中实现有什么差别? JDK1.8解決了JDK1.7中什么问题

- `HashTable` : 使用了synchronized关键字对put等操作进行加锁;
- `ConcurrentHashMap JDK1.7`: 使用分段锁机制实现;
- `ConcurrentHashMap JDK1.8`: 则使用数组+链表+红黑树数据结构和CAS原子操作实现;

#### [#](#concurrenthashmap-jdk1-7实现的原理是什么) ConcurrentHashMap JDK1.7实现的原理是什么?

在JDK1.5~1.7版本，Java使用了分段锁机制实现ConcurrentHashMap.

简而言之，ConcurrentHashMap在对象中保存了一个Segment数组，即将整个Hash表划分为多个分段；而每个Segment元素，它通过继承 ReentrantLock 来进行加锁，所以每次需要加锁的操作锁住的是一个 segment，这样只要保证每个 Segment 是线程安全的，也就实现了全局的线程安全；这样，在执行put操作时首先根据hash算法定位到元素属于哪个Segment，然后对该Segment加锁即可。因此，ConcurrentHashMap在多线程并发编程中可是实现多线程put操作。

![img](https://pdai.tech/images/thread/java-thread-x-concurrent-hashmap-1.png)

`concurrencyLevel`: Segment 数（并行级别、并发数）。默认是 16，也就是说 ConcurrentHashMap 有 16 个 Segments，所以理论上，这个时候，最多可以同时支持 16 个线程并发写，只要它们的操作分别分布在不同的 Segment 上。这个值可以在初始化的时候设置为其他值，但是一旦初始化以后，它是不可以扩容的。

#### [#](#concurrenthashmap-jdk1-7中segment数-concurrencylevel-默认值是多少-为何一旦初始化就不可再扩容) ConcurrentHashMap JDK1.7中Segment数(concurrencyLevel)默认值是多少? 为何一旦初始化就不可再扩容?

默认是 16

#### [#](#concurrenthashmap-jdk1-7说说其put的机制) ConcurrentHashMap JDK1.7说说其put的机制?

整体流程还是比较简单的，由于有独占锁的保护，所以 segment 内部的操作并不复杂

1. 计算 key 的 hash 值
2. 根据 hash 值找到 Segment 数组中的位置 j； ensureSegment(j) 对 segment[j] 进行初始化（Segment 内部是由 **数组+链表** 组成的）
3. 插入新值到 槽 s 中

#### [#](#concurrenthashmap-jdk1-7是如何扩容的) ConcurrentHashMap JDK1.7是如何扩容的?

rehash(注：segment 数组不能扩容，扩容是 segment 数组某个位置内部的数组 HashEntry<K,V>[] 进行扩容)

#### [#](#concurrenthashmap-jdk1-8实现的原理是什么) ConcurrentHashMap JDK1.8实现的原理是什么?

在JDK1.7之前，ConcurrentHashMap是通过分段锁机制来实现的，所以其最大并发度受Segment的个数限制。因此，在JDK1.8中，ConcurrentHashMap的实现原理摒弃了这种设计，而是选择了与HashMap类似的数组+链表+红黑树的方式实现，而加锁则采用CAS和synchronized实现。

简而言之：数组+链表+红黑树，CAS

#### [#](#concurrenthashmap-jdk1-8是如何扩容的) ConcurrentHashMap JDK1.8是如何扩容的?

tryPresize, 扩容也是做翻倍扩容的，扩容后数组容量为原来的 2 倍

#### [#](#concurrenthashmap-jdk1-8链表转红黑树的时机是什么-临界值为什么是8) ConcurrentHashMap JDK1.8链表转红黑树的时机是什么? 临界值为什么是8?

size = 8, log(N)

#### [#](#concurrenthashmap-jdk1-8是如何进行数据迁移的) ConcurrentHashMap JDK1.8是如何进行数据迁移的?

transfer, 将原来的 tab 数组的元素迁移到新的 nextTab 数组中



#### 介绍一下CAS乐观锁

> CAS（Compare and Swap）是一种乐观锁技术，用于解决并发环境下多线程对共享数据的竞争问题。它是一种基于硬件原子性操作的原理，常用于实现无锁算法和乐观并发控制。



CAS 操作涉及三个参数：内存位置（通常是共享变量）、期望值和新值。它的操作逻辑如下：

1. 读取内存位置的当前值（旧值）。
2. 检查当前值是否与期望值相等。如果相等，则将内存位置的值更新为新值。
3. 如果当前值与期望值不相等，说明其他线程已经修改了内存位置的值，操作失败。

这个过程是原子性的，意味着在执行过程中不会被其他线程中断。

CAS 乐观锁的优点包括：

- 无锁：不需要像传统锁那样进行加锁和解锁操作，避免了线程等待和上下文切换的开销。
- 高并发：适用于高并发场景，多个线程可以同时执行 CAS 操作，减少了竞争锁资源的问题。

然而，CAS 乐观锁也有一些局限性：

- ABA 问题：如果在执行 CAS 时，内存位置的值从 A 变为 B，再变回 A，这时 CAS 操作可能会错误地成功，尽管实际上内存位置的值已经发生了变化。
- 自旋等待：如果 CAS 操作失败，线程可能会进入自旋等待状态，不断尝试 CAS 操作，这可能会占用 CPU 资源。

在 Java 中，`java.util.concurrent.atomic` 包提供了一系列原子类，如 `AtomicInteger`、`AtomicLong` 等，它们使用了 CAS 技术来实现线程安全的操作，避免了传统锁的开销。这些原子类可以在多线程环境中实现高效的并发操作。

#### [#](#先说说非并发集合中fail-fast机制) 先说说非并发集合中Fail-fast机制?

快速失败

#### [#](#copyonwritearraylist的实现原理) CopyOnWriteArrayList的实现原理?

COW基于拷贝

```java
  // 将toCopyIn转化为Object[]类型数组，然后设置当前数组
  setArray(Arrays.copyOf(toCopyIn, toCopyIn.length, Object[].class));
```

属性中有一个可重入锁，用来保证线程安全访问，还有一个Object类型的数组，用来存放具体的元素。当然，也使用到了反射机制和CAS来保证原子性的修改lock域。

```java
// 可重入锁
final transient ReentrantLock lock = new ReentrantLock();
// 对象数组，用于存放元素
private transient volatile Object[] array;
// 反射机制
private static final sun.misc.Unsafe UNSAFE;
// lock域的内存偏移量
private static final long lockOffset;
```

#### [#](#弱一致性的迭代器原理是怎么样的) 弱一致性的迭代器原理是怎么样的?

```
COWIterator<E>
```

COWIterator表示迭代器，其也有一个Object类型的数组作为CopyOnWriteArrayList数组的快照，这种快照风格的迭代器方法在创建迭代器时使用了对当时数组状态的引用。此数组在迭代器的生存期内不会更改，因此不可能发生冲突，并且迭代器保证不会抛出 ConcurrentModificationException。创建迭代器以后，迭代器就不会反映列表的添加、移除或者更改。在迭代器上进行的元素更改操作(remove、set 和 add)不受支持。这些方法将抛出 UnsupportedOperationException。

#### [#](#copyonwritearraylist为什么并发安全且性能比vector好) CopyOnWriteArrayList为什么并发安全且性能比Vector好?

Vector对单独的add，remove等方法都是在方法上加了synchronized; 并且如果一个线程A调用size时，另一个线程B 执行了remove，然后size的值就不是最新的，然后线程A调用remove就会越界(这时就需要再加一个Synchronized)。这样就导致有了双重锁，效率大大降低，何必呢。于是vector废弃了，要用就用CopyOnWriteArrayList 吧。

#### [#](#copyonwritearraylist有何缺陷-说说其应用场景) CopyOnWriteArrayList有何缺陷，说说其应用场景?

CopyOnWriteArrayList 有几个缺点：

- 由于写操作的时候，需要拷贝数组，会消耗内存，如果原数组的内容比较多的情况下，可能导致young gc或者full gc
- 不能用于实时读的场景，像拷贝数组、新增元素都需要时间，所以调用一个set操作后，读取到数据可能还是旧的,虽然CopyOnWriteArrayList 能做到最终一致性,但是还是没法满足实时性要求；

**CopyOnWriteArrayList 合适读多写少的场景，不过这类慎用**

因为谁也没法保证CopyOnWriteArrayList 到底要放置多少数据，万一数据稍微有点多，每次add/set都要重新复制数组，这个代价实在太高昂了。在高性能的互联网应用中，这种操作分分钟引起故障。

#### [#](#要想用线程安全的队列有哪些选择) 要想用线程安全的队列有哪些选择?

Vector，`Collections.synchronizedList( List<T> list)`, ConcurrentLinkedQueue等

#### [#](#concurrentlinkedqueue实现的数据结构) ConcurrentLinkedQueue实现的数据结构?

ConcurrentLinkedQueue的数据结构与LinkedBlockingQueue的数据结构相同，都是使用的链表结构。ConcurrentLinkedQueue的数据结构如下:

![img](/images/thread/java-thread-x-juc-concurrentlinkedqueue-1.png)

说明: ConcurrentLinkedQueue采用的链表结构，并且包含有一个头节点和一个尾结点。

#### [#](#concurrentlinkedqueue底层原理) ConcurrentLinkedQueue底层原理?

```java
// 反射机制
private static final sun.misc.Unsafe UNSAFE;
// head域的偏移量
private static final long headOffset;
// tail域的偏移量
private static final long tailOffset;
```

说明: 属性中包含了head域和tail域，表示链表的头节点和尾结点，同时，ConcurrentLinkedQueue也使用了反射机制和CAS机制来更新头节点和尾结点，保证原子性。

#### [#](#concurrentlinkedqueue的核心方法有哪些) ConcurrentLinkedQueue的核心方法有哪些?

offer()，poll()，peek()，isEmpty()等队列常用方法

#### [#](#说说concurrentlinkedqueue的hops-延迟更新的策略-的设计) 说说ConcurrentLinkedQueue的HOPS(延迟更新的策略)的设计?

通过上面对offer和poll方法的分析，我们发现tail和head是延迟更新的，两者更新触发时机为：

- **tail更新触发时机**：当tail指向的节点的下一个节点不为null的时候，会执行定位队列真正的队尾节点的操作，找到队尾节点后完成插入之后才会通过casTail进行tail更新；当tail指向的节点的下一个节点为null的时候，只插入节点不更新tail。
- **head更新触发时机**：当head指向的节点的item域为null的时候，会执行定位队列真正的队头节点的操作，找到队头节点后完成删除之后才会通过updateHead进行head更新；当head指向的节点的item域不为null的时候，只删除节点不更新head。

并且在更新操作时，源码中会有注释为：`hop two nodes at a time`。所以这种延迟更新的策略就被叫做HOPS的大概原因是这个(猜的 😃)，从上面更新时的状态图可以看出，head和tail的更新是“跳着的”即中间总是间隔了一个。那么这样设计的意图是什么呢?

如果让tail永远作为队列的队尾节点，实现的代码量会更少，而且逻辑更易懂。但是，这样做有一个缺点，如果大量的入队操作，每次都要执行CAS进行tail的更新，汇总起来对性能也会是大大的损耗。如果能减少CAS更新的操作，无疑可以大大提升入队的操作效率，所以doug lea大师每间隔1次(tail和队尾节点的距离为1)进行才利用CAS更新tail。对head的更新也是同样的道理，虽然，这样设计会多出在循环中定位队尾节点，但总体来说读的操作效率要远远高于写的性能，因此，多出来的在循环中定位尾节点的操作的性能损耗相对而言是很小的。

#### [#](#concurrentlinkedqueue适合什么样的使用场景) ConcurrentLinkedQueue适合什么样的使用场景?

ConcurrentLinkedQueue通过无锁来做到了更高的并发量，是个高性能的队列，但是使用场景相对不如阻塞队列常见，毕竟取数据也要不停的去循环，不如阻塞的逻辑好设计，但是在并发量特别大的情况下，是个不错的选择，性能上好很多，而且这个队列的设计也是特别费力，尤其的使用的改良算法和对哨兵的处理。整体的思路都是比较严谨的，这个也是使用了无锁造成的，我们自己使用无锁的条件的话，这个队列是个不错的参考。

#### [#](#什么是blockingdeque-适合用在什么样的场景) 什么是BlockingDeque? 适合用在什么样的场景?

BlockingQueue 通常用于一个线程生产对象，而另外一个线程消费这些对象的场景。下图是对这个原理的阐述:

![img](/images/thread/java-thread-x-blocking-queue-1.png)

一个线程往里边放，另外一个线程从里边取的一个 BlockingQueue。

一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。 负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列。

#### [#](#blockingqueue大家族有哪些) BlockingQueue大家族有哪些?

ArrayBlockingQueue, DelayQueue, LinkedBlockingQueue, SynchronousQueue...

#### [#](#blockingqueue常用的方法) BlockingQueue常用的方法?

BlockingQueue 具有 4 组不同的方法用于插入、移除以及对队列中的元素进行检查。如果请求的操作不能得到立即执行的话，每个方法的表现也不同。这些方法如下:

|      | 抛异常    | 特定值   | 阻塞   | 超时                        |
| ---- | --------- | -------- | ------ | --------------------------- |
| 插入 | add(o)    | offer(o) | put(o) | offer(o, timeout, timeunit) |
| 移除 | remove()  | poll()   | take() | poll(timeout, timeunit)     |
| 检查 | element() | peek()   |        |                             |

四组不同的行为方式解释:

- 抛异常: 如果试图的操作无法立即执行，抛一个异常。
- 特定值: 如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。
- 阻塞: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。
- 超时: 如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

#### [#](#blockingqueue-实现例子) BlockingQueue 实现例子?

这里是一个 Java 中使用 BlockingQueue 的示例。本示例使用的是 BlockingQueue 接口的 ArrayBlockingQueue 实现。 首先，BlockingQueueExample 类分别在两个独立的线程中启动了一个 Producer 和 一个 Consumer。Producer 向一个共享的 BlockingQueue 中注入字符串，而 Consumer 则会从中把它们拿出来。

```java
public class BlockingQueueExample {
 
    public static void main(String[] args) throws Exception {
 
        BlockingQueue queue = new ArrayBlockingQueue(1024);
 
        Producer producer = new Producer(queue);
        Consumer consumer = new Consumer(queue);
 
        new Thread(producer).start();
        new Thread(consumer).start();
 
        Thread.sleep(4000);
    }
}
```

以下是 Producer 类。注意它在每次 put() 调用时是如何休眠一秒钟的。这将导致 Consumer 在等待队列中对象的时候发生阻塞。

```java
public class Producer implements Runnable{
 
    protected BlockingQueue queue = null;
 
    public Producer(BlockingQueue queue) {
        this.queue = queue;
    }
 
    public void run() {
        try {
            queue.put("1");
            Thread.sleep(1000);
            queue.put("2");
            Thread.sleep(1000);
            queue.put("3");
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

以下是 Consumer 类。它只是把对象从队列中抽取出来，然后将它们打印到 System.out。

```java
public class Consumer implements Runnable{
 
    protected BlockingQueue queue = null;
 
    public Consumer(BlockingQueue queue) {
        this.queue = queue;
    }
 
    public void run() {
        try {
            System.out.println(queue.take());
            System.out.println(queue.take());
            System.out.println(queue.take());
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### [#](#什么是blockingdeque-适合用在什么样的场景-1) 什么是BlockingDeque? 适合用在什么样的场景?

java.util.concurrent 包里的 BlockingDeque 接口表示一个线程安放入和提取实例的双端队列。

BlockingDeque 类是一个双端队列，在不能够插入元素时，它将阻塞住试图插入元素的线程；在不能够抽取元素时，它将阻塞住试图抽取的线程。 deque(双端队列) 是 "Double Ended Queue" 的缩写。因此，双端队列是一个你可以从任意一端插入或者抽取元素的队列。

在线程既是一个队列的生产者又是这个队列的消费者的时候可以使用到 BlockingDeque。如果生产者线程需要在队列的两端都可以插入数据，消费者线程需要在队列的两端都可以移除数据，这个时候也可以使用 BlockingDeque。BlockingDeque 图解:

![img](/images/thread/java-thread-x-blocking-deque-1.png)

#### [#](#blockingdeque-与blockingqueue有何关系-请对比下它们的方法) BlockingDeque 与BlockingQueue有何关系，请对比下它们的方法?

BlockingDeque 接口继承自 BlockingQueue 接口。这就意味着你可以像使用一个 BlockingQueue 那样使用 BlockingDeque。如果你这么干的话，各种插入方法将会把新元素添加到双端队列的尾端，而移除方法将会把双端队列的首端的元素移除。正如 BlockingQueue 接口的插入和移除方法一样。

以下是 BlockingDeque 对 BlockingQueue 接口的方法的具体内部实现:

| BlockingQueue | BlockingDeque   |
| ------------- | --------------- |
| add()         | addLast()       |
| offer() x 2   | offerLast() x 2 |
| put()         | putLast()       |
| remove()      | removeFirst()   |
| poll() x 2    | pollFirst()     |
| take()        | takeFirst()     |
| element()     | getFirst()      |
| peek()        | peekFirst()     |

#### [#](#blockingdeque大家族有哪些) BlockingDeque大家族有哪些?

LinkedBlockingDeque 是一个双端队列，在它为空的时候，一个试图从中抽取数据的线程将会阻塞，无论该线程是试图从哪一端抽取数据。

#### [#](#blockingdeque-实现例子) BlockingDeque 实现例子?

既然 BlockingDeque 是一个接口，那么你想要使用它的话就得使用它的众多的实现类的其中一个。java.util.concurrent 包提供了以下 BlockingDeque 接口的实现类: LinkedBlockingDeque。

以下是如何使用 BlockingDeque 方法的一个简短代码示例:

```java
BlockingDeque<String> deque = new LinkedBlockingDeque<String>();
deque.addFirst("1");
deque.addLast("2");
 
String two = deque.takeLast();
String one = deque.takeFirst();
```

### [#](#_3-7-juc线程池) 3.7 JUC线程池

- [JUC线程池: FutureTask详解]()
- [JUC线程池: ThreadPoolExecutor详解]()
- [JUC线程池: ScheduledThreadPool详解]()
- [JUC线程池: Fork/Join框架详解]()

#### [#](#futuretask用来解决什么问题的-为什么会出现) FutureTask用来解决什么问题的? 为什么会出现?

FutureTask 为 Future 提供了基础实现，如获取任务执行结果(get)和取消任务(cancel)等。如果任务尚未完成，获取任务执行结果时将会阻塞。一旦执行结束，任务就不能被重启或取消(除非使用runAndReset执行计算)。FutureTask 常用来封装 Callable 和 Runnable，也可以作为一个任务提交到线程池中执行。除了作为一个独立的类之外，此类也提供了一些功能性函数供我们创建自定义 task 类使用。FutureTask 的线程安全由CAS来保证。

#### [#](#futuretask类结构关系怎么样的) FutureTask类结构关系怎么样的?

![img](https://pdai.tech/images/thread/java-thread-x-juc-futuretask-1.png)

可以看到,FutureTask实现了RunnableFuture接口，则RunnableFuture接口继承了Runnable接口和Future接口，所以FutureTask既能当做一个Runnable直接被Thread执行，也能作为Future用来得到Callable的计算结果。

#### [#](#futuretask的线程安全是由什么保证的) FutureTask的线程安全是由什么保证的?

FutureTask 的线程安全由CAS来保证。

#### [#](#futuretask通常会怎么用-举例说明。) FutureTask通常会怎么用? 举例说明。

```java
import java.util.concurrent.*;
 
public class CallDemo {
 
    public static void main(String[] args) throws ExecutionException, InterruptedException {
 
        /**
         * 第一种方式:Future + ExecutorService
         * Task task = new Task();
         * ExecutorService service = Executors.newCachedThreadPool();
         * Future<Integer> future = service.submit(task1);
         * service.shutdown();
         */
 
 
        /**
         * 第二种方式: FutureTask + ExecutorService
         * ExecutorService executor = Executors.newCachedThreadPool();
         * Task task = new Task();
         * FutureTask<Integer> futureTask = new FutureTask<Integer>(task);
         * executor.submit(futureTask);
         * executor.shutdown();
         */
 
        /**
         * 第三种方式:FutureTask + Thread
         */
 
        // 2. 新建FutureTask,需要一个实现了Callable接口的类的实例作为构造函数参数
        FutureTask<Integer> futureTask = new FutureTask<Integer>(new Task());
        // 3. 新建Thread对象并启动
        Thread thread = new Thread(futureTask);
        thread.setName("Task thread");
        thread.start();
 
        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
 
        System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
 
        // 4. 调用isDone()判断任务是否结束
        if(!futureTask.isDone()) {
            System.out.println("Task is not done");
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        int result = 0;
        try {
            // 5. 调用get()方法获取任务结果,如果任务没有执行完成则阻塞等待
            result = futureTask.get();
        } catch (Exception e) {
            e.printStackTrace();
        }
 
        System.out.println("result is " + result);
 
    }
 
    // 1. 继承Callable接口,实现call()方法,泛型参数为要返回的类型
    static class Task  implements Callable<Integer> {
 
        @Override
        public Integer call() throws Exception {
            System.out.println("Thread [" + Thread.currentThread().getName() + "] is running");
            int result = 0;
            for(int i = 0; i < 100;++i) {
                result += i;
            }
 
            Thread.sleep(3000);
            return result;
        }
    }
}
```

#### [#](#为什么要有线程池) 为什么要有线程池?

线程池能够对线程进行统一分配，调优和监控:

- 降低资源消耗(线程无限制地创建，然后使用完毕后销毁)
- 提高响应速度(无须创建线程)
- 提高线程的可管理性

#### [#](#java是实现和管理线程池有哪些方式-请简单举例如何使用。) Java是实现和管理线程池有哪些方式? 请简单举例如何使用。

从JDK 5开始，把工作单元与执行机制分离开来，工作单元包括Runnable和Callable，而执行机制由Executor框架提供。

- WorkerThread

```java
public class WorkerThread implements Runnable {
     
    private String command;
     
    public WorkerThread(String s){
        this.command=s;
    }
 
    @Override
    public void run() {
        System.out.println(Thread.currentThread().getName()+" Start. Command = "+command);
        processCommand();
        System.out.println(Thread.currentThread().getName()+" End.");
    }
 
    private void processCommand() {
        try {
            Thread.sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
 
    @Override
    public String toString(){
        return this.command;
    }
}
```

- SimpleThreadPool

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
 
public class SimpleThreadPool {
 
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(5);
        for (int i = 0; i < 10; i++) {
            Runnable worker = new WorkerThread("" + i);
            executor.execute(worker);
          }
        executor.shutdown(); // This will make the executor accept no new threads and finish all existing threads in the queue
        while (!executor.isTerminated()) { // Wait until all threads are finish,and also you can use "executor.awaitTermination();" to wait
        }
        System.out.println("Finished all threads");
    }

}
```

程序中我们创建了固定大小为五个工作线程的线程池。然后分配给线程池十个工作，因为线程池大小为五，它将启动五个工作线程先处理五个工作，其他的工作则处于等待状态，一旦有工作完成，空闲下来工作线程就会捡取等待队列里的其他工作进行执行。

这里是以上程序的输出。

```html
pool-1-thread-2 Start. Command = 1
pool-1-thread-4 Start. Command = 3
pool-1-thread-1 Start. Command = 0
pool-1-thread-3 Start. Command = 2
pool-1-thread-5 Start. Command = 4
pool-1-thread-4 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
pool-1-thread-3 End.
pool-1-thread-3 Start. Command = 8
pool-1-thread-2 End.
pool-1-thread-2 Start. Command = 9
pool-1-thread-1 Start. Command = 7
pool-1-thread-5 Start. Command = 6
pool-1-thread-4 Start. Command = 5
pool-1-thread-2 End.
pool-1-thread-4 End.
pool-1-thread-3 End.
pool-1-thread-5 End.
pool-1-thread-1 End.
Finished all threads
```

输出表明线程池中至始至终只有五个名为 "pool-1-thread-1" 到 "pool-1-thread-5" 的五个线程，这五个线程不随着工作的完成而消亡，会一直存在，并负责执行分配给线程池的任务，直到线程池消亡。

Executors 类提供了使用了 ThreadPoolExecutor 的简单的 ExecutorService 实现，但是 ThreadPoolExecutor 提供的功能远不止于此。我们可以在创建 ThreadPoolExecutor 实例时指定活动线程的数量，我们也可以限制线程池的大小并且创建我们自己的 RejectedExecutionHandler 实现来处理不能适应工作队列的工作。

这里是我们自定义的 RejectedExecutionHandler 接口的实现。

- RejectedExecutionHandlerImpl.java

```java
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadPoolExecutor;
 
public class RejectedExecutionHandlerImpl implements RejectedExecutionHandler {
 
    @Override
    public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
        System.out.println(r.toString() + " is rejected");
    }
 
}
```

ThreadPoolExecutor 提供了一些方法，我们可以使用这些方法来查询 executor 的当前状态，线程池大小，活动线程数量以及任务数量。因此我是用来一个监控线程在特定的时间间隔内打印 executor 信息。

- MyMonitorThread.java

```java
import java.util.concurrent.ThreadPoolExecutor;
 
public class MyMonitorThread implements Runnable
{
    private ThreadPoolExecutor executor;
     
    private int seconds;
     
    private boolean run=true;
 
    public MyMonitorThread(ThreadPoolExecutor executor, int delay)
    {
        this.executor = executor;
        this.seconds=delay;
    }
     
    public void shutdown(){
        this.run=false;
    }
 
    @Override
    public void run()
    {
        while(run){
                System.out.println(
                    String.format("[monitor] [%d/%d] Active: %d, Completed: %d, Task: %d, isShutdown: %s, isTerminated: %s",
                        this.executor.getPoolSize(),
                        this.executor.getCorePoolSize(),
                        this.executor.getActiveCount(),
                        this.executor.getCompletedTaskCount(),
                        this.executor.getTaskCount(),
                        this.executor.isShutdown(),
                        this.executor.isTerminated()));
                try {
                    Thread.sleep(seconds*1000);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
        }
             
    }
}
```

这里是使用 ThreadPoolExecutor 的线程池实现例子。

- WorkerPool.java

```java
import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
 
public class WorkerPool {
 
    public static void main(String args[]) throws InterruptedException{
        //RejectedExecutionHandler implementation
        RejectedExecutionHandlerImpl rejectionHandler = new RejectedExecutionHandlerImpl();
        //Get the ThreadFactory implementation to use
        ThreadFactory threadFactory = Executors.defaultThreadFactory();
        //creating the ThreadPoolExecutor
        ThreadPoolExecutor executorPool = new ThreadPoolExecutor(2, 4, 10, TimeUnit.SECONDS, new ArrayBlockingQueue<Runnable>(2), threadFactory, rejectionHandler);
        //start the monitoring thread
        MyMonitorThread monitor = new MyMonitorThread(executorPool, 3);
        Thread monitorThread = new Thread(monitor);
        monitorThread.start();
        //submit work to the thread pool
        for(int i=0; i<10; i++){
            executorPool.execute(new WorkerThread("cmd"+i));
        }
         
        Thread.sleep(30000);
        //shut down the pool
        executorPool.shutdown();
        //shut down the monitor thread
        Thread.sleep(5000);
        monitor.shutdown();
         
    }
}
```

注意在初始化 ThreadPoolExecutor 时，我们保持初始池大小为 2，最大池大小为 4 而工作队列大小为 2。因此如果已经有四个正在执行的任务而此时分配来更多任务的话，工作队列将仅仅保留他们(新任务)中的两个，其他的将会被 RejectedExecutionHandlerImpl 处理。

上面程序的输出可以证实以上观点。

```html
pool-1-thread-1 Start. Command = cmd0
pool-1-thread-4 Start. Command = cmd5
cmd6 is rejected
pool-1-thread-3 Start. Command = cmd4
pool-1-thread-2 Start. Command = cmd1
cmd7 is rejected
cmd8 is rejected
cmd9 is rejected
[monitor] [0/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 4, Completed: 0, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-4 End.
pool-1-thread-1 End.
pool-1-thread-2 End.
pool-1-thread-3 End.
pool-1-thread-1 Start. Command = cmd3
pool-1-thread-4 Start. Command = cmd2
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
[monitor] [4/2] Active: 2, Completed: 4, Task: 6, isShutdown: false, isTerminated: false
pool-1-thread-1 End.
pool-1-thread-4 End.
[monitor] [4/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [2/2] Active: 0, Completed: 6, Task: 6, isShutdown: false, isTerminated: false
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
[monitor] [0/2] Active: 0, Completed: 6, Task: 6, isShutdown: true, isTerminated: true
```

注意 executor 的活动任务、完成任务以及所有完成任务，这些数量上的变化。我们可以调用 shutdown() 方法来结束所有提交的任务并终止线程池。

#### [#](#threadpoolexecutor的原理) ThreadPoolExecutor的原理?

其实java线程池的实现原理很简单，说白了就是一个线程集合workerSet和一个阻塞队列workQueue。当用户向线程池提交一个任务(也就是线程)时，线程池会先将任务放入workQueue中。workerSet中的线程会不断的从workQueue中获取线程然后执行。当workQueue中没有任务的时候，worker就会阻塞，直到队列中有任务了就取出来继续执行。

![img](https://pdai.tech/images/thread/java-thread-x-executors-1.png)

当一个任务提交至线程池之后:

1. 线程池首先当前运行的线程数量是否少于corePoolSize。如果是，则创建一个新的工作线程来执行任务。如果都在执行任务，则进入2.
2. 判断BlockingQueue是否已经满了，倘若还没有满，则将线程放入BlockingQueue。否则进入3.
3. 如果创建一个新的工作线程将使当前运行的线程数量超过maximumPoolSize，则交给RejectedExecutionHandler来处理任务。

当ThreadPoolExecutor创建新线程时，通过CAS来更新线程池的状态ctl.

#### [#](#threadpoolexecutor有哪些核心的配置参数-请简要说明) ThreadPoolExecutor有哪些核心的配置参数? 请简要说明

```java
public ThreadPoolExecutor(int corePoolSize,
                              int maximumPoolSize,
                              long keepAliveTime,
                              TimeUnit unit,
                              BlockingQueue<Runnable> workQueue,
                              RejectedExecutionHandler handler)
```

- `corePoolSize` 线程池中的核心线程数，当提交一个任务时，线程池创建一个新线程执行任务，直到当前线程数等于corePoolSize, 即使有其他空闲线程能够执行新来的任务, 也会继续创建线程；如果当前线程数为corePoolSize，继续提交的任务被保存到阻塞队列中，等待被执行；如果执行了线程池的prestartAllCoreThreads()方法，线程池会提前创建并启动所有核心线程。
- `workQueue` 用来保存等待被执行的任务的阻塞队列. 在JDK中提供了如下阻塞队列: 具体可以参考[JUC 集合: BlockQueue详解]()
  - `ArrayBlockingQueue`: 基于数组结构的有界阻塞队列，按FIFO排序任务；
  - `LinkedBlockingQueue`: 基于链表结构的阻塞队列，按FIFO排序任务，吞吐量通常要高于ArrayBlockingQueue；
  - `SynchronousQueue`: 一个不存储元素的阻塞队列，每个插入操作必须等到另一个线程调用移除操作，否则插入操作一直处于阻塞状态，吞吐量通常要高于LinkedBlockingQueue；
  - `PriorityBlockingQueue`: 具有优先级的无界阻塞队列；

`LinkedBlockingQueue`比`ArrayBlockingQueue`在插入删除节点性能方面更优，但是二者在`put()`, `take()`任务的时均需要加锁，`SynchronousQueue`使用无锁算法，根据节点的状态判断执行，而不需要用到锁，其核心是`Transfer.transfer()`.

- `maximumPoolSize ` 线程池中允许的最大线程数。如果当前阻塞队列满了，且继续提交任务，则创建新的线程执行任务，前提是当前线程数小于maximumPoolSize；当阻塞队列是无界队列, 则maximumPoolSize则不起作用, 因为无法提交至核心线程池的线程会一直持续地放入workQueue.
- `keepAliveTime ` 线程空闲时的存活时间，即当线程没有任务执行时，该线程继续存活的时间；默认情况下，该参数只在线程数大于corePoolSize时才有用, 超过这个时间的空闲线程将被终止；
- `unit ` keepAliveTime的单位
- `threadFactory ` 创建线程的工厂，通过自定义的线程工厂可以给每个新建的线程设置一个具有识别度的线程名。默认为DefaultThreadFactory
- `handler ` 线程池的饱和策略，当阻塞队列满了，且没有空闲的工作线程，如果继续提交任务，必须采取一种策略处理该任务，线程池提供了4种策略:
  - `AbortPolicy`: 直接抛出异常，默认策略；
  - `CallerRunsPolicy`: 用调用者所在的线程来执行任务；
  - `DiscardOldestPolicy`: 丢弃阻塞队列中靠最前的任务，并执行当前任务；
  - `DiscardPolicy`: 直接丢弃任务；

当然也可以根据应用场景实现RejectedExecutionHandler接口，自定义饱和策略，如记录日志或持久化存储不能处理的任务。

#### [#](#threadpoolexecutor可以创建哪是哪三种线程池呢) ThreadPoolExecutor可以创建哪是哪三种线程池呢?

- newFixedThreadPool

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>());
}
```

线程池的线程数量达corePoolSize后，即使线程池没有可执行任务时，也不会释放线程。

FixedThreadPool的工作队列为无界队列LinkedBlockingQueue(队列容量为Integer.MAX_VALUE), 这会导致以下问题:

- 线程池里的线程数量不超过corePoolSize,这导致了maximumPoolSize和keepAliveTime将会是个无用参数
- 由于使用了无界队列, 所以FixedThreadPool永远不会拒绝, 即饱和策略失效

- newSingleThreadExecutor

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

初始化的线程池中只有一个线程，如果该线程异常结束，会重新创建一个新的线程继续执行任务，唯一的线程可以保证所提交任务的顺序执行.

由于使用了无界队列, 所以SingleThreadPool永远不会拒绝, 即饱和策略失效

- newCachedThreadPool

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                    60L, TimeUnit.SECONDS,
                                    new SynchronousQueue<Runnable>());
}
```

线程池的线程数可达到Integer.MAX_VALUE，即2147483647，内部使用SynchronousQueue作为阻塞队列； 和newFixedThreadPool创建的线程池不同，newCachedThreadPool在没有任务执行时，当线程的空闲时间超过keepAliveTime，会自动释放线程资源，当提交新任务时，如果没有空闲线程，则创建新线程执行任务，会导致一定的系统开销； 执行过程与前两种稍微不同:

- 主线程调用SynchronousQueue的offer()方法放入task, 倘若此时线程池中有空闲的线程尝试读取 SynchronousQueue的task, 即调用了SynchronousQueue的poll(), 那么主线程将该task交给空闲线程. 否则执行(2)
- 当线程池为空或者没有空闲的线程, 则创建新的线程执行任务.
- 执行完任务的线程倘若在60s内仍空闲, 则会被终止. 因此长时间空闲的CachedThreadPool不会持有任何线程资源.

#### [#](#当队列满了并且worker的数量达到maxsize的时候-会怎么样) 当队列满了并且worker的数量达到maxSize的时候，会怎么样?

当队列满了并且worker的数量达到maxSize的时候,执行具体的拒绝策略

```java
private volatile RejectedExecutionHandler handler;
```

#### [#](#说说threadpoolexecutor有哪些rejectedexecutionhandler策略-默认是什么策略) 说说ThreadPoolExecutor有哪些RejectedExecutionHandler策略? 默认是什么策略?

- AbortPolicy, 默认

该策略是线程池的默认策略。使用该策略时，如果线程池队列满了丢掉这个任务并且抛出RejectedExecutionException异常。 源码如下：

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
  //不做任何处理，直接抛出异常
  throw new RejectedExecutionException("xxx");
}
```

- DiscardPolicy

这个策略和AbortPolicy的slient版本，如果线程池队列满了，会直接丢掉这个任务并且不会有任何异常。 源码如下：

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    //就是一个空的方法
}
```

- DiscardOldestPolicy

这个策略从字面上也很好理解，丢弃最老的。也就是说如果队列满了，会将最早进入队列的任务删掉腾出空间，再尝试加入队列。 因为队列是队尾进，队头出，所以队头元素是最老的，因此每次都是移除对头元素后再尝试入队。 源码如下：

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
        //移除队头元素
        e.getQueue().poll();
        //再尝试入队
        e.execute(r);
    }
}
```

- CallerRunsPolicy

使用此策略，如果添加到线程池失败，那么主线程会自己去执行该任务，不会等待线程池中的线程去执行。就像是个急脾气的人，我等不到别人来做这件事就干脆自己干。 源码如下：

```java
public void rejectedExecution(Runnable r, ThreadPoolExecutor e) {
    if (!e.isShutdown()) {
        //直接执行run方法
        r.run();
    }
}
```

#### [#](#简要说下线程池的任务执行机制) 简要说下线程池的任务执行机制?

execute –> addWorker –>runworker (getTask)

1. 线程池的工作线程通过Woker类实现，在ReentrantLock锁的保证下，把Woker实例插入到HashSet后，并启动Woker中的线程。
2. 从Woker类的构造方法实现可以发现: 线程工厂在创建线程thread时，将Woker实例本身this作为参数传入，当执行start方法启动线程thread时，本质是执行了Worker的runWorker方法。
3. firstTask执行完成之后，通过getTask方法从阻塞队列中获取等待的任务，如果队列中没有任务，getTask方法会被阻塞并挂起，不会占用cpu资源；

#### [#](#线程池中任务是如何提交的) 线程池中任务是如何提交的?

![img](/images/thread/java-thread-x-executors-3.png)

1. submit任务，等待线程池execute
2. 执行FutureTask类的get方法时，会把主线程封装成WaitNode节点并保存在waiters链表中， 并阻塞等待运行结果；
3. FutureTask任务执行完成后，通过UNSAFE设置waiters相应的waitNode为null，并通过LockSupport类unpark方法唤醒主线程；

```java
public class Test{
    public static void main(String[] args) {

        ExecutorService es = Executors.newCachedThreadPool();
        Future<String> future = es.submit(new Callable<String>() {
            @Override
            public String call() throws Exception {
                try {
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                return "future result";
            }
        });
        try {
            String result = future.get();
            System.out.println(result);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```

在实际业务场景中，Future和Callable基本是成对出现的，Callable负责产生结果，Future负责获取结果。

1. Callable接口类似于Runnable，只是Runnable没有返回值。
2. Callable任务除了返回正常结果之外，如果发生异常，该异常也会被返回，即Future可以拿到异步执行任务各种结果；
3. Future.get方法会导致主线程阻塞，直到Callable任务执行完成；

#### [#](#线程池中任务是如何关闭的) 线程池中任务是如何关闭的?

- shutdown

将线程池里的线程状态设置成SHUTDOWN状态, 然后中断所有没有正在执行任务的线程.

- shutdownNow

将线程池里的线程状态设置成STOP状态, 然后停止所有正在执行或暂停任务的线程. 只要调用这两个关闭方法中的任意一个, isShutDown() 返回true. 当所有任务都成功关闭了, isTerminated()返回true.

#### [#](#在配置线程池的时候需要考虑哪些配置因素) 在配置线程池的时候需要考虑哪些配置因素?

从任务的优先级，任务的执行时间长短，任务的性质(CPU密集/ IO密集)，任务的依赖关系这四个角度来分析。并且近可能地使用有界的工作队列。

性质不同的任务可用使用不同规模的线程池分开处理:

- CPU密集型: 尽可能少的线程，Ncpu+1
- IO密集型: 尽可能多的线程, Ncpu*2，比如数据库连接池
- 混合型: CPU密集型的任务与IO密集型任务的执行时间差别较小，拆分为两个线程池；否则没有必要拆分。

#### [#](#如何监控线程池的状态) 如何监控线程池的状态?

可以使用ThreadPoolExecutor以下方法:

- `getTaskCount()` Returns the approximate total number of tasks that have ever been scheduled for execution.
- `getCompletedTaskCount()` Returns the approximate total number of tasks that have completed execution. 返回结果少于getTaskCount()。
- `getLargestPoolSize()` Returns the largest number of threads that have ever simultaneously been in the pool. 返回结果小于等于maximumPoolSize
- `getPoolSize()` Returns the current number of threads in the pool.
- `getActiveCount()` Returns the approximate number of threads that are actively executing tasks.

#### [#](#为什么很多公司不允许使用executors去创建线程池-那么推荐怎么使用呢) 为什么很多公司不允许使用Executors去创建线程池? 那么推荐怎么使用呢?

线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors各个方法的弊端：

- newFixedThreadPool和newSingleThreadExecutor:   主要问题是堆积的请求处理队列可能会耗费非常大的内存，甚至OOM。
- newCachedThreadPool和newScheduledThreadPool:   主要问题是线程数最大数是Integer.MAX_VALUE，可能会创建数量非常多的线程，甚至OOM。
- 推荐方式 1 首先引入：commons-lang3包

```java
ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
```



- 推荐方式 2 首先引入：com.google.guava包

```java
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder().setNameFormat("demo-pool-%d").build();

//Common Thread Pool
ExecutorService pool = new ThreadPoolExecutor(5, 200, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());

// excute
pool.execute(()-> System.out.println(Thread.currentThread().getName()));

 //gracefully shutdown
pool.shutdown();
```

- 推荐方式 3 spring配置线程池方式：自定义线程工厂bean需要实现ThreadFactory，可参考该接口的其它默认实现类，使用方式直接注入bean调用execute(Runnable task)方法即可

```xml
    <bean id="userThreadPool" class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
        <property name="corePoolSize" value="10" />
        <property name="maxPoolSize" value="100" />
        <property name="queueCapacity" value="2000" />

    <property name="threadFactory" value= threadFactory />
        <property name="rejectedExecutionHandler">
            <ref local="rejectedExecutionHandler" />
        </property>
    </bean>
    
    //in code
    userThreadPool.execute(thread);
```

#### [#](#scheduledthreadpoolexecutor要解决什么样的问题) ScheduledThreadPoolExecutor要解决什么样的问题?

在很多业务场景中，我们可能需要周期性的运行某项任务来获取结果，比如周期数据统计，定时发送数据等。在并发包出现之前，Java 早在1.3就提供了 Timer 类(只需要了解，目前已渐渐被 ScheduledThreadPoolExecutor 代替)来适应这些业务场景。随着业务量的不断增大，我们可能需要多个工作线程运行任务来尽可能的增加产品性能，或者是需要更高的灵活性来控制和监控这些周期业务。这些都是 ScheduledThreadPoolExecutor 诞生的必然性。

#### [#](#scheduledthreadpoolexecutor相比threadpoolexecutor有哪些特性) ScheduledThreadPoolExecutor相比ThreadPoolExecutor有哪些特性?

ScheduledThreadPoolExecutor继承自 ThreadPoolExecutor，为任务提供延迟或周期执行，属于线程池的一种。和 ThreadPoolExecutor 相比，它还具有以下几种特性:

- 使用专门的任务类型—ScheduledFutureTask 来执行周期任务，也可以接收不需要时间调度的任务(这些任务通过 ExecutorService 来执行)。
- 使用专门的存储队列—DelayedWorkQueue 来存储任务，DelayedWorkQueue 是无界延迟队列DelayQueue 的一种。相比ThreadPoolExecutor也简化了执行机制(delayedExecute方法，后面单独分析)。
- 支持可选的run-after-shutdown参数，在池被关闭(shutdown)之后支持可选的逻辑来决定是否继续运行周期或延迟任务。并且当任务(重新)提交操作与 shutdown 操作重叠时，复查逻辑也不相同。

#### [#](#scheduledthreadpoolexecutor有什么样的数据结构-核心内部类和抽象类) ScheduledThreadPoolExecutor有什么样的数据结构，核心内部类和抽象类?

![img](/images/thread/java-thread-x-stpe-1.png)

ScheduledThreadPoolExecutor继承自 `ThreadPoolExecutor`:

- 详情请参考: [JUC线程池: ThreadPoolExecutor详解]()

ScheduledThreadPoolExecutor 内部构造了两个内部类 `ScheduledFutureTask` 和 `DelayedWorkQueue`:

- `ScheduledFutureTask`: 继承了FutureTask，说明是一个异步运算任务；最上层分别实现了Runnable、Future、Delayed接口，说明它是一个可以延迟执行的异步运算任务。
- `DelayedWorkQueue`: 这是 ScheduledThreadPoolExecutor 为存储周期或延迟任务专门定义的一个延迟队列，继承了 AbstractQueue，为了契合 ThreadPoolExecutor 也实现了 BlockingQueue 接口。它内部只允许存储 RunnableScheduledFuture 类型的任务。与 DelayQueue 的不同之处就是它只允许存放 RunnableScheduledFuture 对象，并且自己实现了二叉堆(DelayQueue 是利用了 PriorityQueue 的二叉堆结构)。

#### [#](#scheduledthreadpoolexecutor有哪两个关闭策略-区别是什么) ScheduledThreadPoolExecutor有哪两个关闭策略? 区别是什么?

**shutdown**: 在shutdown方法中调用的关闭钩子onShutdown方法，它的主要作用是在关闭线程池后取消并清除由于关闭策略不应该运行的所有任务，这里主要是根据 run-after-shutdown 参数(continueExistingPeriodicTasksAfterShutdown和executeExistingDelayedTasksAfterShutdown)来决定线程池关闭后是否关闭已经存在的任务。

**showDownNow**: 立即关闭

#### [#](#scheduledthreadpoolexecutor中scheduleatfixedrate-和-schedulewithfixeddelay区别是什么) ScheduledThreadPoolExecutor中scheduleAtFixedRate 和 scheduleWithFixedDelay区别是什么?

**注意scheduleAtFixedRate和scheduleWithFixedDelay的区别**: 乍一看两个方法一模一样，其实，在unit.toNanos这一行代码中还是有区别的。没错，scheduleAtFixedRate传的是正值，而scheduleWithFixedDelay传的则是负值，这个值就是 ScheduledFutureTask 的period属性。

#### [#](#为什么threadpoolexecutor-的调整策略却不适用于-scheduledthreadpoolexecutor) 为什么ThreadPoolExecutor 的调整策略却不适用于 ScheduledThreadPoolExecutor?

例如: 由于 ScheduledThreadPoolExecutor 是一个固定核心线程数大小的线程池，并且使用了一个无界队列，所以调整maximumPoolSize对其没有任何影响(所以 ScheduledThreadPoolExecutor 没有提供可以调整最大线程数的构造函数，默认最大线程数固定为Integer.MAX_VALUE)。此外，设置corePoolSize为0或者设置核心线程空闲后清除(allowCoreThreadTimeOut)同样也不是一个好的策略，因为一旦周期任务到达某一次运行周期时，可能导致线程池内没有线程去处理这些任务。

#### [#](#executors-提供了几种方法来构造-scheduledthreadpoolexecutor) Executors 提供了几种方法来构造 ScheduledThreadPoolExecutor?

- newScheduledThreadPool: 可指定核心线程数的线程池。
- newSingleThreadScheduledExecutor: 只有一个工作线程的线程池。如果内部工作线程由于执行周期任务异常而被终止，则会新建一个线程替代它的位置。

#### [#](#fork-join主要用来解决什么样的问题) Fork/Join主要用来解决什么样的问题?

ForkJoinPool 是JDK 7加入的一个线程池类。Fork/Join 技术是分治算法(Divide-and-Conquer)的并行实现，它是一项可以获得良好的并行性能的简单且高效的设计技术。目的是为了帮助我们更好地利用多处理器带来的好处，使用所有可用的运算能力来提升应用的性能。

#### [#](#fork-join框架是在哪个jdk版本中引入的) Fork/Join框架是在哪个JDK版本中引入的?

JDK 7

#### [#](#fork-join框架主要包含哪三个模块-模块之间的关系是怎么样的) Fork/Join框架主要包含哪三个模块? 模块之间的关系是怎么样的?

Fork/Join框架主要包含三个模块:

- 任务对象: `ForkJoinTask` (包括`RecursiveTask`、`RecursiveAction` 和 `CountedCompleter`)
- 执行Fork/Join任务的线程: `ForkJoinWorkerThread`
- 线程池: `ForkJoinPool`

这三者的关系是: ForkJoinPool可以通过池中的ForkJoinWorkerThread来处理ForkJoinTask任务。

#### [#](#forkjoinpool类继承关系) ForkJoinPool类继承关系?

![img](/images/thread/java-thread-x-forkjoin-1.png)

内部类介绍:

- ForkJoinWorkerThreadFactory: 内部线程工厂接口，用于创建工作线程ForkJoinWorkerThread
- DefaultForkJoinWorkerThreadFactory: ForkJoinWorkerThreadFactory 的默认实现类
- InnocuousForkJoinWorkerThreadFactory: 实现了 ForkJoinWorkerThreadFactory，无许可线程工厂，当系统变量中有系统安全管理相关属性时，默认使用这个工厂创建工作线程。
- EmptyTask: 内部占位类，用于替换队列中 join 的任务。
- ManagedBlocker: 为 ForkJoinPool 中的任务提供扩展管理并行数的接口，一般用在可能会阻塞的任务(如在 Phaser 中用于等待 phase 到下一个generation)。
- WorkQueue: ForkJoinPool 的核心数据结构，本质上是work-stealing 模式的双端任务队列，内部存放 ForkJoinTask 对象任务，使用 @Contented 注解修饰防止伪共享。
  - 工作线程在运行中产生新的任务(通常是因为调用了 fork())时，此时可以把 WorkQueue 的数据结构视为一个栈，新的任务会放入栈顶(top 位)；工作线程在处理自己工作队列的任务时，按照 LIFO 的顺序。
  - 工作线程在处理自己的工作队列同时，会尝试窃取一个任务(可能是来自于刚刚提交到 pool 的任务，或是来自于其他工作线程的队列任务)，此时可以把 WorkQueue 的数据结构视为一个 FIFO 的队列，窃取的任务位于其他线程的工作队列的队首(base位)。
- 伪共享状态: 缓存系统中是以缓存行(cache line)为单位存储的。缓存行是2的整数幂个连续字节，一般为32-256个字节。最常见的缓存行大小是64个字节。当多线程修改互相独立的变量时，如果这些变量共享同一个缓存行，就会无意中影响彼此的性能，这就是伪共享。

#### [#](#forkjointask抽象类继承关系) ForkJoinTask抽象类继承关系?

![img](/images/thread/java-thread-x-forkjoin-4.png)

ForkJoinTask 实现了 Future 接口，说明它也是一个可取消的异步运算任务，实际上ForkJoinTask 是 Future 的轻量级实现，主要用在纯粹是计算的函数式任务或者操作完全独立的对象计算任务。fork 是主运行方法，用于异步执行；而 join 方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果。 其内部类都比较简单，ExceptionNode 是用于存储任务执行期间的异常信息的单向链表；其余四个类是为 Runnable/Callable 任务提供的适配器类，用于把 Runnable/Callable 转化为 ForkJoinTask 类型的任务(因为 ForkJoinPool 只可以运行 ForkJoinTask 类型的任务)。

#### [#](#整个fork-join-框架的执行流程-运行机制是怎么样的) 整个Fork/Join 框架的执行流程/运行机制是怎么样的?

- 首先介绍任务的提交流程 - 外部任务(external/submissions task)提交
- 然后介绍任务的提交流程 - 子任务(Worker task)提交
- 再分析任务的执行过程(ForkJoinWorkerThread.run()到ForkJoinTask.doExec()这一部分)；
- 最后介绍任务的结果获取(ForkJoinTask.join()和ForkJoinTask.invoke())

#### [#](#具体阐述fork-join的分治思想和work-stealing-实现方式) 具体阐述Fork/Join的分治思想和work-stealing 实现方式?

- 分治算法(Divide-and-Conquer)

分治算法(Divide-and-Conquer)把任务递归的拆分为各个子任务，这样可以更好的利用系统资源，尽可能的使用所有可用的计算能力来提升应用性能。首先看一下 Fork/Join 框架的任务运行机制:

![img](/images/thread/java-thread-x-forkjoin-2.png)

- work-stealing(工作窃取)算法

work-stealing(工作窃取)算法: 线程池内的所有工作线程都尝试找到并执行已经提交的任务，或者是被其他活动任务创建的子任务(如果不存在就阻塞等待)。这种特性使得 ForkJoinPool 在运行多个可以产生子任务的任务，或者是提交的许多小任务时效率更高。尤其是构建异步模型的 ForkJoinPool 时，对不需要合并(join)的事件类型任务也非常适用。

在 ForkJoinPool 中，线程池中每个工作线程(ForkJoinWorkerThread)都对应一个任务队列(WorkQueue)，工作线程优先处理来自自身队列的任务(LIFO或FIFO顺序，参数 mode 决定)，然后以FIFO的顺序随机窃取其他队列中的任务。

具体思路如下:

- 每个线程都有自己的一个WorkQueue，该工作队列是一个双端队列。
- 队列支持三个功能push、pop、poll
- push/pop只能被队列的所有者线程调用，而poll可以被其他线程调用。
- 划分的子任务调用fork时，都会被push到自己的队列中。
- 默认情况下，工作线程从自己的双端队列获出任务并执行。
- 当自己的队列为空时，线程随机从另一个线程的队列末尾调用poll方法窃取任务。

![img](/images/thread/java-thread-x-forkjoin-3.png)

#### [#](#有哪些jdk源码中使用了fork-join思想) 有哪些JDK源码中使用了Fork/Join思想?

我们常用的数组工具类 Arrays 在JDK 8之后新增的并行排序方法(parallelSort)就运用了 ForkJoinPool 的特性，还有 ConcurrentHashMap 在JDK 8之后添加的函数式方法(如forEach等)也有运用。

#### [#](#如何使用executors工具类创建forkjoinpool) 如何使用Executors工具类创建ForkJoinPool?

Java8在Executors工具类中新增了两个工厂方法:

```java
// parallelism定义并行级别
public static ExecutorService newWorkStealingPool(int parallelism);
// 默认并行级别为JVM可用的处理器个数
// Runtime.getRuntime().availableProcessors()
public static ExecutorService newWorkStealingPool();
```

#### [#](#写一个例子-用forkjoin方式实现1-2-3-100000) 写一个例子: 用ForkJoin方式实现1+2+3+...+100000?

```java
public class Test {
	static final class SumTask extends RecursiveTask<Integer> {
		private static final long serialVersionUID = 1L;
		
		final int start; //开始计算的数
		final int end; //最后计算的数
		
		SumTask(int start, int end) {
			this.start = start;
			this.end = end;
		}

		@Override
		protected Integer compute() {
			//如果计算量小于1000，那么分配一个线程执行if中的代码块，并返回执行结果
			if(end - start < 1000) {
				System.out.println(Thread.currentThread().getName() + " 开始执行: " + start + "-" + end);
				int sum = 0;
				for(int i = start; i <= end; i++)
					sum += i;
				return sum;
			}
			//如果计算量大于1000，那么拆分为两个任务
			SumTask task1 = new SumTask(start, (start + end) / 2);
			SumTask task2 = new SumTask((start + end) / 2 + 1, end);
			//执行任务
			task1.fork();
			task2.fork();
			//获取任务执行的结果
			return task1.join() + task2.join();
		}
	}
	
	public static void main(String[] args) throws InterruptedException, ExecutionException {
		ForkJoinPool pool = new ForkJoinPool();
		ForkJoinTask<Integer> task = new SumTask(1, 10000);
		pool.submit(task);
		System.out.println(task.get());
	}
}
```

- 执行结果

```java
ForkJoinPool-1-worker-1 开始执行: 1-625
ForkJoinPool-1-worker-7 开始执行: 6251-6875
ForkJoinPool-1-worker-6 开始执行: 5626-6250
ForkJoinPool-1-worker-10 开始执行: 3751-4375
ForkJoinPool-1-worker-13 开始执行: 2501-3125
ForkJoinPool-1-worker-8 开始执行: 626-1250
ForkJoinPool-1-worker-11 开始执行: 5001-5625
ForkJoinPool-1-worker-3 开始执行: 7501-8125
ForkJoinPool-1-worker-14 开始执行: 1251-1875
ForkJoinPool-1-worker-4 开始执行: 9376-10000
ForkJoinPool-1-worker-8 开始执行: 8126-8750
ForkJoinPool-1-worker-0 开始执行: 1876-2500
ForkJoinPool-1-worker-12 开始执行: 4376-5000
ForkJoinPool-1-worker-5 开始执行: 8751-9375
ForkJoinPool-1-worker-7 开始执行: 6876-7500
ForkJoinPool-1-worker-1 开始执行: 3126-3750
50005000
```

#### [#](#fork-join在使用时有哪些注意事项-结合jdk中的斐波那契数列实例具体说明。) Fork/Join在使用时有哪些注意事项? 结合JDK中的斐波那契数列实例具体说明。

斐波那契数列: 1、1、2、3、5、8、13、21、34、…… 公式 : F(1)=1，F(2)=1, F(n)=F(n-1)+F(n-2)(n>=3，n∈N*)

```java
public static void main(String[] args) {
    ForkJoinPool forkJoinPool = new ForkJoinPool(4); // 最大并发数4
    Fibonacci fibonacci = new Fibonacci(20);
    long startTime = System.currentTimeMillis();
    Integer result = forkJoinPool.invoke(fibonacci);
    long endTime = System.currentTimeMillis();
    System.out.println("Fork/join sum: " + result + " in " + (endTime - startTime) + " ms.");
}
//以下为官方API文档示例
static  class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) {
        this.n = n;
    }
    @Override
    protected Integer compute() {
        if (n <= 1) {
            return n;
        }
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork(); 
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join(); 
    }
}
```

当然你也可以两个任务都fork，要注意的是两个任务都fork的情况，必须按照f1.fork()，f2.fork()， f2.join()，f1.join()这样的顺序，不然有性能问题，详见上面注意事项中的说明。

官方API文档是这样写到的，所以平日用invokeAll就好了。invokeAll会把传入的任务的第一个交给当前线程来执行，其他的任务都fork加入工作队列，这样等于利用当前线程也执行任务了。

```java
{
    // ...
    Fibonacci f1 = new Fibonacci(n - 1);
    Fibonacci f2 = new Fibonacci(n - 2);
    invokeAll(f1,f2);
    return f2.join() + f1.join();
}

public static void invokeAll(ForkJoinTask<?>... tasks) {
    Throwable ex = null;
    int last = tasks.length - 1;
    for (int i = last; i >= 0; --i) {
        ForkJoinTask<?> t = tasks[i];
        if (t == null) {
            if (ex == null)
                ex = new NullPointerException();
        }
        else if (i != 0)   //除了第一个都fork
            t.fork();
        else if (t.doInvoke() < NORMAL && ex == null)  //留一个自己执行
            ex = t.getException();
    }
    for (int i = 1; i <= last; ++i) {
        ForkJoinTask<?> t = tasks[i];
        if (t != null) {
            if (ex != null)
                t.cancel(false);
            else if (t.doJoin() < NORMAL)
                ex = t.getException();
        }
    }
    if (ex != null)
        rethrow(ex);
}
```

### [#](#_3-8-juc工具类) 3.8 JUC工具类

- [JUC工具类: CountDownLatch详解]()
- [JUC工具类: CyclicBarrier详解]()
- [JUC工具类: Semaphore详解]()
- [JUC工具类: Phaser详解]()
- [JUC工具类: Exchanger详解]()
- [Java 并发 - ThreadLocal详解]()

#### [#](#什么是countdownlatch) 什么是CountDownLatch?

CountDownLatch底层也是由AQS，用来同步一个或多个任务的常用并发工具类，强制它们等待由其他任务执行的一组操作完成。

`CountDownLatch` 是 Java 中的一个同步辅助类，用于在多线程环境中控制线程的执行顺序和等待。它提供了一种简单的方式，使一个或多个线程等待其他线程的完成，然后再继续执行。

以下是关于 `CountDownLatch` 的一些关键特点和用法：

1. **初始化计数器：** `CountDownLatch` 初始化时需要指定一个计数器值。这个计数器值表示需要等待的线程数。
2. **等待操作：** 调用 `await()` 方法的线程会被阻塞，直到计数器值减为 0。当计数器值变为 0 时，等待的线程会被唤醒，继续执行。
3. **倒计数操作：** 其他线程可以通过调用 `countDown()` 方法来减小计数器的值。每次调用 `countDown()` 方法都会减小计数器的值，直到计数器减为 0。
4. **应用场景：** `CountDownLatch` 适用于一个或多个线程需要等待其他线程完成某些操作后再继续执行的场景。比如，主线程等待多个子线程完成初始化，多个线程等待一个共享资源就绪等。
5. **一次性使用：** `CountDownLatch` 是一次性的，一旦计数器减为 0，就无法再次使用。如果需要重复使用类似功能，可以考虑使用 `CyclicBarrier` 或者 `Semaphore`。
6. **线程安全：** `CountDownLatch` 是线程安全的，多个线程可以同时调用 `countDown()` 方法和 `await()` 方法，不需要额外的同步操作。

`CountDownLatch` 提供了一种简单的线程同步方式，可以在多线程编程中实现等待其他线程完成的需求。它可以帮助协调多个线程的执行顺序，以及实现一些并发控制的场景。

#### [#](#countdownlatch底层实现原理) CountDownLatch底层实现原理?

其底层是由AQS提供支持，所以其数据结构可以参考AQS的数据结构，而AQS的数据结构核心就是两个虚拟队列: 同步队列sync queue 和条件队列condition queue，不同的条件会有不同的条件队列。CountDownLatch典型的用法是将一个程序分为n个互相独立的可解决任务，并创建值为n的CountDownLatch。当每一个任务完成时，都会在这个锁存器上调用countDown，等待问题被解决的任务调用这个锁存器的await，将他们自己拦住，直至锁存器计数结束。

#### [#](#countdownlatch一次可以唤醒几个任务) CountDownLatch一次可以唤醒几个任务?

多个

#### [#](#countdownlatch有哪些主要方法) CountDownLatch有哪些主要方法?

await(), 此函数将会使当前线程在锁存器倒计数至零之前一直等待，除非线程被中断。

countDown(), 此函数将递减锁存器的计数，如果计数到达零，则释放所有等待的线程

#### [#](#写道题-实现一个容器-提供两个方法-add-size-写两个线程-线程1添加10个元素到容器中-线程2实现监控元素的个数-当个数到5个时-线程2给出提示并结束) 写道题：实现一个容器，提供两个方法，add，size 写两个线程，线程1添加10个元素到容器中，线程2实现监控元素的个数，当个数到5个时，线程2给出提示并结束?

说出使用CountDownLatch 代替wait notify 好处?

```java
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

/**
 * 使用CountDownLatch 代替wait notify 好处是通讯方式简单，不涉及锁定  Count 值为0时当前线程继续执行，
 */
public class T3 {

   volatile List list = new ArrayList();

    public void add(int i){
        list.add(i);
    }

    public int getSize(){
        return list.size();
    }


    public static void main(String[] args) {
        T3 t = new T3();
        CountDownLatch countDownLatch = new CountDownLatch(1);

        new Thread(() -> {
            System.out.println("t2 start");
           if(t.getSize() != 5){
               try {
                   countDownLatch.await();
                   System.out.println("t2 end");
               } catch (InterruptedException e) {
                   e.printStackTrace();
               }
           }
        },"t2").start();

        new Thread(()->{
            System.out.println("t1 start");
           for (int i = 0;i<9;i++){
               t.add(i);
               System.out.println("add"+ i);
               if(t.getSize() == 5){
                   System.out.println("countdown is open");
                   countDownLatch.countDown();
               }
           }
            System.out.println("t1 end");
        },"t1").start();
    }

}
```

#### [#](#什么是cyclicbarrier) 什么是CyclicBarrier?

- 对于CountDownLatch，其他线程为游戏玩家，比如英雄联盟，主线程为控制游戏开始的线程。在所有的玩家都准备好之前，主线程是处于等待状态的，也就是游戏不能开始。当所有的玩家准备好之后，下一步的动作实施者为主线程，即开始游戏。
- 对于CyclicBarrier，假设有一家公司要全体员工进行团建活动，活动内容为翻越三个障碍物，每一个人翻越障碍物所用的时间是不一样的。但是公司要求所有人在翻越当前障碍物之后再开始翻越下一个障碍物，也就是所有人翻越第一个障碍物之后，才开始翻越第二个，以此类推。类比地，每一个员工都是一个“其他线程”。当所有人都翻越的所有的障碍物之后，程序才结束。而主线程可能早就结束了，这里我们不用管主线程。

#### [#](#countdownlatch和cyclicbarrier对比) CountDownLatch和CyclicBarrier对比?

- CountDownLatch减计数，CyclicBarrier加计数。
- CountDownLatch是一次性的，CyclicBarrier可以重用。
- CountDownLatch和CyclicBarrier都有让多个线程等待同步然后再开始下一步动作的意思，但是CountDownLatch的下一步的动作实施者是主线程，具有不可重复性；而CyclicBarrier的下一步动作实施者还是“其他线程”本身，具有往复多次实施动作的特点。

#### [#](#什么是semaphore) 什么是Semaphore?

Semaphore底层是基于AbstractQueuedSynchronizer来实现的。Semaphore称为计数信号量，它允许n个任务同时访问某个资源，可以将信号量看做是在向外分发使用资源的许可证，只有成功获取许可证，才能使用资源

#### [#](#semaphore内部原理) Semaphore内部原理?

Semaphore总共有三个内部类，并且三个内部类是紧密相关的，下面先看三个类的关系。

![img](https://pdai.tech/images/thread/java-thread-x-semaphore-1.png)

说明: Semaphore与ReentrantLock的内部类的结构相同，类内部总共存在Sync、NonfairSync、FairSync三个类，NonfairSync与FairSync类继承自Sync类，Sync类继承自AbstractQueuedSynchronizer抽象类。下面逐个进行分析。

#### [#](#semaphore常用方法有哪些-如何实现线程同步和互斥的) Semaphore常用方法有哪些? 如何实现线程同步和互斥的?

#### [#](#单独使用semaphore是不会使用到aqs的条件队列) 单独使用Semaphore是不会使用到AQS的条件队列?

不同于CyclicBarrier和ReentrantLock，单独使用Semaphore是不会使用到AQS的条件队列的，其实，只有进行await操作才会进入条件队列，其他的都是在同步队列中，只是当前线程会被park。

#### [#](#semaphore初始化有10个令牌-11个线程同时各调用1次acquire方法-会发生什么) Semaphore初始化有10个令牌，11个线程同时各调用1次acquire方法，会发生什么?

拿不到令牌的线程阻塞，不会继续往下运行。

#### [#](#semaphore初始化有10个令牌-一个线程重复调用11次acquire方法-会发生什么) Semaphore初始化有10个令牌，一个线程重复调用11次acquire方法，会发生什么?

线程阻塞，不会继续往下运行。可能你会考虑类似于锁的重入的问题，很好，但是，令牌没有重入的概念。你只要调用一次acquire方法，就需要有一个令牌才能继续运行。

#### [#](#semaphore初始化有1个令牌-1个线程调用一次acquire方法-然后调用两次release方法-之后另外一个线程调用acquire-2-方法-此线程能够获取到足够的令牌并继续运行吗) Semaphore初始化有1个令牌，1个线程调用一次acquire方法，然后调用两次release方法，之后另外一个线程调用acquire(2)方法，此线程能够获取到足够的令牌并继续运行吗?

能，原因是release方法会添加令牌，并不会以初始化的大小为准。

#### [#](#semaphore初始化有2个令牌-一个线程调用1次release方法-然后一次性获取3个令牌-会获取到吗) Semaphore初始化有2个令牌，一个线程调用1次release方法，然后一次性获取3个令牌，会获取到吗?

能，原因是release会添加令牌，并不会以初始化的大小为准。Semaphore中release方法的调用并没有限制要在acquire后调用。

具体示例如下，如果不相信的话，可以运行一下下面的demo，在做实验之前，笔者也认为应该是不允许的。。

```java
public class TestSemaphore2 {
    public static void main(String[] args) {
        int permitsNum = 2;
        final Semaphore semaphore = new Semaphore(permitsNum);
        try {
            System.out.println("availablePermits:"+semaphore.availablePermits()+",semaphore.tryAcquire(3,1, TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
            semaphore.release();
            System.out.println("availablePermits:"+semaphore.availablePermits()+",semaphore.tryAcquire(3,1, TimeUnit.SECONDS):"+semaphore.tryAcquire(3,1, TimeUnit.SECONDS));
        }catch (Exception e) {

        }
    }
}
```

#### [#](#phaser主要用来解决什么问题) Phaser主要用来解决什么问题?

Phaser是JDK 7新增的一个同步辅助类，它可以实现CyclicBarrier和CountDownLatch类似的功能，而且它支持对任务的动态调整，并支持分层结构来达到更高的吞吐量。

#### [#](#phaser与cyclicbarrier和countdownlatch的区别是什么) Phaser与CyclicBarrier和CountDownLatch的区别是什么?

Phaser 和 CountDownLatch、CyclicBarrier 都有很相似的地方。

Phaser 顾名思义，就是可以分阶段的进行线程同步。

- CountDownLatch 只能在创建实例时，通过构造方法指定同步数量；
- Phaser 支持线程动态地向它注册。

利用这个动态注册的特性，可以达到分阶段同步控制的目的：

注册一批操作，等待它们执行结束；再注册一批操作，等它们结束...

#### [#](#phaser运行机制是什么样的) Phaser运行机制是什么样的?

![img](/images/thread/java-thread-x-juc-phaser-1.png)

- **Registration(注册)**

跟其他barrier不同，在phaser上注册的parties会随着时间的变化而变化。任务可以随时注册(使用方法register,bulkRegister注册，或者由构造器确定初始parties)，并且在任何抵达点可以随意地撤销注册(方法arriveAndDeregister)。就像大多数基本的同步结构一样，注册和撤销只影响内部count；不会创建更深的内部记录，所以任务不能查询他们是否已经注册。(不过，可以通过继承来实现类似的记录)

- **Synchronization(同步机制)**

和CyclicBarrier一样，Phaser也可以重复await。方法arriveAndAwaitAdvance的效果类似CyclicBarrier.await。phaser的每一代都有一个相关的phase number，初始值为0，当所有注册的任务都到达phaser时phase+1，到达最大值(Integer.MAX_VALUE)之后清零。使用phase number可以独立控制 到达phaser 和 等待其他线程 的动作，通过下面两种类型的方法:

> - **Arrival(到达机制)** arrive和arriveAndDeregister方法记录到达状态。这些方法不会阻塞，但是会返回一个相关的arrival phase number；也就是说，phase number用来确定到达状态。当所有任务都到达给定phase时，可以执行一个可选的函数，这个函数通过重写onAdvance方法实现，通常可以用来控制终止状态。重写此方法类似于为CyclicBarrier提供一个barrierAction，但比它更灵活。
> - **Waiting(等待机制)** awaitAdvance方法需要一个表示arrival phase number的参数，并且在phaser前进到与给定phase不同的phase时返回。和CyclicBarrier不同，即使等待线程已经被中断，awaitAdvance方法也会一直等待。中断状态和超时时间同样可用，但是当任务等待中断或超时后未改变phaser的状态时会遭遇异常。如果有必要，在方法forceTermination之后可以执行这些异常的相关的handler进行恢复操作，Phaser也可能被ForkJoinPool中的任务使用，这样在其他任务阻塞等待一个phase时可以保证足够的并行度来执行任务。

- **Termination(终止机制)** :

可以用isTerminated方法检查phaser的终止状态。在终止时，所有同步方法立刻返回一个负值。在终止时尝试注册也没有效果。当调用onAdvance返回true时Termination被触发。当deregistration操作使已注册的parties变为0时，onAdvance的默认实现就会返回true。也可以重写onAdvance方法来定义终止动作。forceTermination方法也可以释放等待线程并且允许它们终止。

- **Tiering(分层结构)** :

Phaser支持分层结构(树状构造)来减少竞争。注册了大量parties的Phaser可能会因为同步竞争消耗很高的成本， 因此可以设置一些子Phaser来共享一个通用的parent。这样的话即使每个操作消耗了更多的开销，但是会提高整体吞吐量。 在一个分层结构的phaser里，子节点phaser的注册和取消注册都通过父节点管理。子节点phaser通过构造或方法register、bulkRegister进行首次注册时，在其父节点上注册。子节点phaser通过调用arriveAndDeregister进行最后一次取消注册时，也在其父节点上取消注册。

- **Monitoring(状态监控)** :

由于同步方法可能只被已注册的parties调用，所以phaser的当前状态也可能被任何调用者监控。在任何时候，可以通过getRegisteredParties获取parties数，其中getArrivedParties方法返回已经到达当前phase的parties数。当剩余的parties(通过方法getUnarrivedParties获取)到达时，phase进入下一代。这些方法返回的值可能只表示短暂的状态，所以一般来说在同步结构里并没有啥卵用。

#### [#](#给一个phaser使用的示例) 给一个Phaser使用的示例?

模拟了100米赛跑，10名选手，只等裁判一声令下。当所有人都到达终点时，比赛结束。

```java
public class Match {

    // 模拟了100米赛跑，10名选手，只等裁判一声令下。当所有人都到达终点时，比赛结束。
    public static void main(String[] args) throws InterruptedException {

        final Phaser phaser=new Phaser(1) ;
        // 十名选手
        for (int index = 0; index < 10; index++) {
            phaser.register();
            new Thread(new player(phaser),"player"+index).start();
        }
        System.out.println("Game Start");
        //注销当前线程,比赛开始
        phaser.arriveAndDeregister();
        //是否非终止态一直等待
        while(!phaser.isTerminated()){
        }
        System.out.println("Game Over");
    }
}
class player implements Runnable{

    private  final Phaser phaser ;

    player(Phaser phaser){
        this.phaser=phaser;
    }
    @Override
    public void run() {
        try {
            // 第一阶段——等待创建好所有线程再开始
            phaser.arriveAndAwaitAdvance();

            // 第二阶段——等待所有选手准备好再开始
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println(Thread.currentThread().getName() + " ready");
            phaser.arriveAndAwaitAdvance();

            // 第三阶段——等待所有选手准备好到达，到达后，该线程从phaser中注销，不在进行下面的阶段。
            Thread.sleep((long) (Math.random() * 10000));
            System.out.println(Thread.currentThread().getName() + " arrived");
            phaser.arriveAndDeregister();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

#### [#](#exchanger主要解决什么问题) Exchanger主要解决什么问题?

Exchanger用于进行两个线程之间的数据交换。它提供一个同步点，在这个同步点，两个线程可以交换彼此的数据。这两个线程通过exchange()方法交换数据，当一个线程先执行exchange()方法后，它会一直等待第二个线程也执行exchange()方法，当这两个线程到达同步点时，这两个线程就可以交换数据了。

#### [#](#对比synchronousqueue-为什么说exchanger可被视为-synchronousqueue-的双向形式) 对比SynchronousQueue，为什么说Exchanger可被视为 SynchronousQueue 的双向形式?

Exchanger是一种线程间安全交换数据的机制。可以和之前分析过的SynchronousQueue对比一下：线程A通过SynchronousQueue将数据a交给线程B；线程A通过Exchanger和线程B交换数据，线程A把数据a交给线程B，同时线程B把数据b交给线程A。可见，SynchronousQueue是交给一个数据，Exchanger是交换两个数据。

#### [#](#exchanger在不同的jdk版本中实现有什么差别) Exchanger在不同的JDK版本中实现有什么差别?

- 在JDK5中Exchanger被设计成一个容量为1的容器，存放一个等待线程，直到有另外线程到来就会发生数据交换，然后清空容器，等到下一个到来的线程。
- 从JDK6开始，Exchanger用了类似ConcurrentMap的分段思想，提供了多个slot，增加了并发执行时的吞吐量。

#### [#](#exchanger实现举例) Exchanger实现举例

来一个非常经典的并发问题：你有相同的数据buffer，一个或多个数据生产者，和一个或多个数据消费者。只是Exchange类只能同步2个线程，所以你只能在你的生产者和消费者问题中只有一个生产者和一个消费者时使用这个类。

```java
public class Test {
    static class Producer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Producer(String name, Exchanger<Integer> exchanger) {
            super("Producer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            for (int i=1; i<5; i++) {
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = i;
                    System.out.println(getName()+" 交换前:" + data);
                    data = exchanger.exchange(data);
                    System.out.println(getName()+" 交换后:" + data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }
    }

    static class Consumer extends Thread {
        private Exchanger<Integer> exchanger;
        private static int data = 0;
        Consumer(String name, Exchanger<Integer> exchanger) {
            super("Consumer-" + name);
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            while (true) {
                data = 0;
                System.out.println(getName()+" 交换前:" + data);
                try {
                    TimeUnit.SECONDS.sleep(1);
                    data = exchanger.exchange(data);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                System.out.println(getName()+" 交换后:" + data);
            }
        }
    }

    public static void main(String[] args) throws InterruptedException {
        Exchanger<Integer> exchanger = new Exchanger<Integer>();
        new Producer("", exchanger).start();
        new Consumer("", exchanger).start();
        TimeUnit.SECONDS.sleep(7);
        System.exit(-1);
    }
}
```

可以看到，其结果可能如下：

```html
Consumer- 交换前:0
Producer- 交换前:1
Consumer- 交换后:1
Consumer- 交换前:0
Producer- 交换后:0
Producer- 交换前:2
Producer- 交换后:0
Consumer- 交换后:2
Consumer- 交换前:0
Producer- 交换前:3
Producer- 交换后:0
Consumer- 交换后:3
Consumer- 交换前:0
Producer- 交换前:4
Producer- 交换后:0
Consumer- 交换后:4
Consumer- 交换前:0
```

#### [#](#什么是threadlocal-用来解决什么问题的) 什么是ThreadLocal? 用来解决什么问题的?

我们在[Java 并发 - 并发理论基础]()总结过线程安全(是指广义上的共享资源访问安全性，因为线程隔离是通过副本保证本线程访问资源安全性，它不保证线程之间还存在共享关系的狭义上的安全性)的解决思路：

- 互斥同步: synchronized 和 ReentrantLock
- 非阻塞同步: CAS, AtomicXXXX
- 无同步方案: 栈封闭，本地存储(Thread Local)，可重入代码

ThreadLocal是通过线程隔离的方式防止任务在共享资源上产生冲突, 线程本地存储是一种自动化机制，可以为使用相同变量的每个不同线程都创建不同的存储。

ThreadLocal是一个将在多线程中为每一个线程创建单独的变量副本的类; 当使用ThreadLocal来维护变量时, ThreadLocal会为每个线程创建单独的变量副本, 避免因多线程操作共享变量而导致的数据不一致的情况。

#### [#](#说说你对threadlocal的理解) 说说你对ThreadLocal的理解

提到ThreadLocal被提到应用最多的是session管理和数据库链接管理，这里以数据访问为例帮助你理解ThreadLocal：

- 如下数据库管理类在单线程使用是没有任何问题的

```java
class ConnectionManager {
    private static Connection connect = null;

    public static Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public static void closeConnection() {
        if (connect != null)
            connect.close();
    }
}
```

很显然，在多线程中使用会存在线程安全问题：第一，这里面的2个方法都没有进行同步，很可能在openConnection方法中会多次创建connect；第二，由于connect是共享变量，那么必然在调用connect的地方需要使用到同步来保障线程安全，因为很可能一个线程在使用connect进行数据库操作，而另外一个线程调用closeConnection关闭链接。

- 为了解决上述线程安全的问题，第一考虑：互斥同步

你可能会说，将这段代码的两个方法进行同步处理，并且在调用connect的地方需要进行同步处理，比如用Synchronized或者ReentrantLock互斥锁。

- 这里再抛出一个问题：这地方到底需不需要将connect变量进行共享?

事实上，是不需要的。假如每个线程中都有一个connect变量，各个线程之间对connect变量的访问实际上是没有依赖关系的，即一个线程不需要关心其他线程是否对这个connect进行了修改的。即改后的代码可以这样：

```java
class ConnectionManager {
    private Connection connect = null;

    public Connection openConnection() {
        if (connect == null) {
            connect = DriverManager.getConnection();
        }
        return connect;
    }

    public void closeConnection() {
        if (connect != null)
            connect.close();
    }
}

class Dao {
    public void insert() {
        ConnectionManager connectionManager = new ConnectionManager();
        Connection connection = connectionManager.openConnection();

        // 使用connection进行操作

        connectionManager.closeConnection();
    }
}
```

这样处理确实也没有任何问题，由于每次都是在方法内部创建的连接，那么线程之间自然不存在线程安全问题。但是这样会有一个致命的影响：导致服务器压力非常大，并且严重影响程序执行性能。由于在方法中需要频繁地开启和关闭数据库连接，这样不仅严重影响程序执行效率，还可能导致服务器压力巨大。

- 这时候ThreadLocal登场了

那么这种情况下使用ThreadLocal是再适合不过的了，因为ThreadLocal在每个线程中对该变量会创建一个副本，即每个线程内部都会有一个该变量，且在线程内部任何地方都可以使用，线程之间互不影响，这样一来就不存在线程安全问题，也不会严重影响程序执行性能。下面就是网上出现最多的例子：

```java
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

public class ConnectionManager {

    private static final ThreadLocal<Connection> dbConnectionLocal = new ThreadLocal<Connection>() {
        @Override
        protected Connection initialValue() {
            try {
                return DriverManager.getConnection("", "", "");
            } catch (SQLException e) {
                e.printStackTrace();
            }
            return null;
        }
    };

    public Connection getConnection() {
        return dbConnectionLocal.get();
    }
}
```

#### [#](#threadlocal是如何实现线程隔离的) ThreadLocal是如何实现线程隔离的?

ThreadLocalMap

#### [#](#为什么threadlocal会造成内存泄露-如何解决) 为什么ThreadLocal会造成内存泄露? 如何解决

网上有这样一个例子：

```java
import java.util.concurrent.LinkedBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;

public class ThreadLocalDemo {
    static class LocalVariable {
        private Long[] a = new Long[1024 * 1024];
    }

    // (1)
    final static ThreadPoolExecutor poolExecutor = new ThreadPoolExecutor(5, 5, 1, TimeUnit.MINUTES,
            new LinkedBlockingQueue<>());
    // (2)
    final static ThreadLocal<LocalVariable> localVariable = new ThreadLocal<LocalVariable>();

    public static void main(String[] args) throws InterruptedException {
        // (3)
        Thread.sleep(5000 * 4);
        for (int i = 0; i < 50; ++i) {
            poolExecutor.execute(new Runnable() {
                public void run() {
                    // (4)
                    localVariable.set(new LocalVariable());
                    // (5)
                    System.out.println("use local varaible" + localVariable.get());
                    localVariable.remove();
                }
            });
        }
        // (6)
        System.out.println("pool execute over");
    }
}
```

如果用线程池来操作ThreadLocal 对象确实会造成内存泄露, 因为对于线程池里面不会销毁的线程, 里面总会存在着`<ThreadLocal, LocalVariable>`的强引用, 因为final static 修饰的 ThreadLocal 并不会释放, 而ThreadLocalMap 对于 Key 虽然是弱引用, 但是强引用不会释放, 弱引用当然也会一直有值, 同时创建的LocalVariable对象也不会释放, 就造成了内存泄露; 如果LocalVariable对象不是一个大对象的话, 其实泄露的并不严重, `泄露的内存 = 核心线程数 * LocalVariable`对象的大小;

所以, 为了避免出现内存泄露的情况, ThreadLocal提供了一个清除线程中对象的方法, 即 remove, 其实内部实现就是调用 ThreadLocalMap 的remove方法:

```java
private void remove(ThreadLocal<?> key) {
    Entry[] tab = table;
    int len = tab.length;
    int i = key.threadLocalHashCode & (len-1);
    for (Entry e = tab[i];
         e != null;
         e = tab[i = nextIndex(i, len)]) {
        if (e.get() == key) {
            e.clear();
            expungeStaleEntry(i);
            return;
        }
    }
}
```

找到Key对应的Entry, 并且清除Entry的Key(ThreadLocal)置空, 随后清除过期的Entry即可避免内存泄露。

#### [#](#还有哪些使用threadlocal的应用场景) 还有哪些使用ThreadLocal的应用场景?

- 每个线程维护了一个“序列号”

~~~java
public class SerialNum {
    // The next serial number to be assigned
    private static int nextSerialNum = 0;

    private static ThreadLocal serialNum = new ThreadLocal() {
        protected synchronized Object initialValue() {
            return new Integer(nextSerialNum++);
        }
    };

    public static int get() {
        return ((Integer) (serialNum.get())).intValue();
    }
}

+ 经典的另外一个例子：

```java
private static final ThreadLocal threadSession = new ThreadLocal();  
  
public static Session getSession() throws InfrastructureException {  
    Session s = (Session) threadSession.get();  
    try {  
        if (s == null) {  
            s = getSessionFactory().openSession();  
            threadSession.set(s);  
        }  
    } catch (HibernateException ex) {  
        throw new InfrastructureException(ex);  
    }  
    return s;  
}  
~~~

- 看看阿里巴巴 java 开发手册中推荐的 ThreadLocal 的用法:

```java
import java.text.DateFormat;
import java.text.SimpleDateFormat;
 
public class DateUtils {
    public static final ThreadLocal<DateFormat> df = new ThreadLocal<DateFormat>(){
        @Override
        protected DateFormat initialValue() {
            return new SimpleDateFormat("yyyy-MM-dd");
        }
    };
}
```

然后我们再要用到 DateFormat 对象的地方，这样调用：

```java
DateUtils.df.get().format(new Date());
```

## [#](#_4-java-io) 4 Java IO

> Java IO相关

### [#](#_4-1-基础io) 4.1 基础IO

#### [#](#如何从数据传输方式理解io流) 如何从数据传输方式理解IO流？

从数据传输方式或者说是运输方式角度看，可以将 IO 类分为:

1. **字节流**, 字节流读取单个字节，字符流读取单个字符(一个字符根据编码的不同，对应的字节也不同，如 UTF-8 编码中文汉字是 3 个字节，GBK编码中文汉字是 2 个字节。)
2. **字符流**, 字节流用来处理二进制文件(图片、MP3、视频文件)，字符流用来处理文本文件(可以看做是特殊的二进制文件，使用了某种编码，人可以阅读)。

**字节是给计算机看的，字符才是给人看的**

- **字节流**

![img](/images/io/java-io-category-1.png)

- **字符流**

![img](/images/io/java-io-category-2.png)

- **字节转字符**？

![img](/images/io/java-io-1.png)

#### [#](#如何从数据操作上理解io流) 如何从数据操作上理解IO流？

从数据来源或者说是操作对象角度看，IO 类可以分为:

![img](/images/io/java-io-category-3.png)

#### [#](#java-io设计上使用了什么设计模式) Java IO设计上使用了什么设计模式？

**装饰者模式**： 所谓装饰，就是把这个装饰者套在被装饰者之上，从而动态扩展被装饰者的功能。

- **装饰者举例**

设计不同种类的饮料，饮料可以添加配料，比如可以添加牛奶，并且支持动态添加新配料。每增加一种配料，该饮料的价格就会增加，要求计算一种饮料的价格。

下图表示在 DarkRoast 饮料上新增新添加 Mocha 配料，之后又添加了 Whip 配料。DarkRoast 被 Mocha 包裹，Mocha 又被 Whip 包裹。它们都继承自相同父类，都有 cost() 方法，外层类的 cost() 方法调用了内层类的 cost() 方法。

![img](/images/pics/c9cfd600-bc91-4f3a-9f99-b42f88a5bb24.jpg)

- 以 InputStream 为例
  - InputStream 是抽象组件；
  - FileInputStream 是 InputStream 的子类，属于具体组件，提供了字节流的输入操作；
  - FilterInputStream 属于抽象装饰者，装饰者用于装饰组件，为组件提供额外的功能。例如 BufferedInputStream 为 FileInputStream 提供缓存的功能。

![image](/images/pics/DP-Decorator-java.io.png)

实例化一个具有缓存功能的字节流对象时，只需要在 FileInputStream 对象上再套一层 BufferedInputStream 对象即可。

```java
FileInputStream fileInputStream = new FileInputStream(filePath);
BufferedInputStream bufferedInputStream = new BufferedInputStream(fileInputStream);
```

DataInputStream 装饰者提供了对更多数据类型进行输入的操作，比如 int、double 等基本类型。

### [#](#_4-2-5种io模型) 4.2 5种IO模型

#### [#](#什么是阻塞-什么是同步) 什么是阻塞？什么是同步？

- **阻塞IO 和 非阻塞IO**

这两个概念是**程序级别**的。主要描述的是程序请求操作系统IO操作后，如果IO资源没有准备好，那么程序该如何处理的问题: 前者等待；后者继续执行(并且使用线程一直轮询，直到有IO资源准备好了)

- **同步IO 和 非同步IO**

这两个概念是**操作系统级别**的。主要描述的是操作系统在收到程序请求IO操作后，如果IO资源没有准备好，该如何响应程序的问题: 前者不响应，直到IO资源准备好以后；后者返回一个标记(好让程序和自己知道以后的数据往哪里通知)，当IO资源准备好以后，再用事件机制返回给程序。

#### [#](#什么是linux的io模型) 什么是Linux的IO模型？

网络IO的本质是socket的读取，socket在linux系统被抽象为流，IO可以理解为对流的操作。刚才说了，对于一次IO访问（以read举例），**数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝到应用程序的地址空间**。所以说，当一个read操作发生时，它会经历两个阶段：

- 第一阶段：等待数据准备 (Waiting for the data to be ready)。
- 第二阶段：将数据从内核拷贝到进程中 (Copying the data from the kernel to the process)。

对于socket流而言，

- 第一步：通常涉及等待网络上的数据分组到达，然后被复制到内核的某个缓冲区。
- 第二步：把数据从内核缓冲区复制到应用进程缓冲区。

网络应用需要处理的无非就是两大类问题，网络IO，数据计算。相对于后者，网络IO的延迟，给应用带来的性能瓶颈大于后者。网络IO的模型大致有如下几种：

1. 同步阻塞IO（bloking IO）
2. 同步非阻塞IO（non-blocking IO）
3. 多路复用IO（multiplexing IO）
4. 信号驱动式IO（signal-driven IO）
5. 异步IO（asynchronous IO）

![img](/images/io/java-io-compare.png)

PS: 这块略复杂，在后面的提供了问答，所以用了最简单的举例结合Linux IO图例帮你快速理解。@pdai

#### [#](#什么是同步阻塞io) 什么是同步阻塞IO？

应用进程被阻塞，直到数据复制到应用进程缓冲区中才返回。

- **举例理解**

你早上去买有现炸油条，你点单，之后一直等店家做好，期间你啥其它事也做不了。（你就是应用级别，店家就是操作系统级别， 应用被阻塞了不能做其它事）

- **Linux 中IO图例**

![img](/images/io/java-io-model-0.png)

#### [#](#什么是同步非阻塞io) 什么是同步非阻塞IO？

应用进程执行系统调用之后，内核返回一个错误码。应用进程可以继续执行，但是需要不断的执行系统调用来获知 I/O 是否完成，这种方式称为轮询(polling)。

- **举例理解**

你早上去买现炸油条，你点单，点完后每隔一段时间询问店家有没有做好，期间你可以做点其它事情。（你就是应用级别，店家就是操作系统级别，应用可以做其它事情并通过轮询来看操作系统是否完成）

- **Linux 中IO图例**

![img](/images/io/java-io-model-1.png)

#### [#](#什么是多路复用io) 什么是多路复用IO？

系统调用可能是由多个任务组成的，所以可以拆成多个任务，这就是多路复用。

- **举例理解**

你早上去买现炸油条，点单收钱和炸油条原来都是由一个人完成的，现在他成了瓶颈，所以专门找了个收银员下单收钱，他则专注在炸油条。（本质上炸油条是耗时的瓶颈，将他职责分离出不是瓶颈的部分，比如下单收银，对应到系统级别也时一样的意思）

- **Linux 中IO图例**

使用 select 或者 poll 等待数据，并且可以等待多个套接字中的任何一个变为可读，这一过程会被阻塞，当某一个套接字可读时返回。之后再使用 recvfrom 把数据从内核复制到进程中。

它可以让单个进程具有处理多个 I/O 事件的能力。又被称为 Event Driven I/O，即事件驱动 I/O。

![img](/images/io/java-io-model-2.png)

#### [#](#有哪些多路复用io) 有哪些多路复用IO？

目前流程的多路复用IO实现主要包括四种: `select`、`poll`、`epoll`、`kqueue`。下表是他们的一些重要特性的比较:

| IO模型 | 相对性能 | 关键思路         | 操作系统      | JAVA支持情况                                                 |
| ------ | -------- | ---------------- | ------------- | ------------------------------------------------------------ |
| select | 较高     | Reactor          | windows/Linux | 支持,Reactor模式(反应器设计模式)。Linux操作系统的 kernels 2.4内核版本之前，默认使用select；而目前windows下对同步IO的支持，都是select模型 |
| poll   | 较高     | Reactor          | Linux         | Linux下的JAVA NIO框架，Linux kernels 2.6内核版本之前使用poll进行支持。也是使用的Reactor模式 |
| epoll  | 高       | Reactor/Proactor | Linux         | Linux kernels 2.6内核版本及以后使用epoll进行支持；Linux kernels 2.6内核版本之前使用poll进行支持；另外一定注意，由于Linux下没有Windows下的IOCP技术提供真正的 异步IO 支持，所以Linux下使用epoll模拟异步IO |
| kqueue | 高       | Proactor         | Linux         | 目前JAVA的版本不支持                                         |

多路复用IO技术最适用的是“高并发”场景，所谓高并发是指1毫秒内至少同时有上千个连接请求准备好。其他情况下多路复用IO技术发挥不出来它的优势。另一方面，使用JAVA NIO进行功能实现，相对于传统的Socket套接字实现要复杂一些，所以实际应用中，需要根据自己的业务需求进行技术选择。

#### [#](#什么是信号驱动io) 什么是信号驱动IO？

应用进程使用 sigaction 系统调用，内核立即返回，应用进程可以继续执行，也就是说等待数据阶段应用进程是非阻塞的。内核在数据到达时向应用进程发送 SIGIO 信号，应用进程收到之后在信号处理程序中调用 recvfrom 将数据从内核复制到应用进程中。

相比于非阻塞式 I/O 的轮询方式，信号驱动 I/O 的 CPU 利用率更高。

- **举例理解**

你早上去买现炸油条，门口排队的人多，现在引入了一个叫号系统，点完单后你就可以做自己的事情了，然后等叫号就去拿就可以了。（所以不用再去自己频繁跑去问有没有做好了）

- **Linux 中IO图例**

![img](/images/io/java-io-model-3.png)

#### [#](#什么是异步io) 什么是异步IO？

相对于同步IO，异步IO不是顺序执行。用户进程进行aio_read系统调用之后，无论内核数据是否准备好，都会直接返回给用户进程，然后用户态进程可以去做别的事情。等到socket数据准备好了，内核直接复制数据给进程，然后从内核向进程发送通知。IO两个阶段，进程都是非阻塞的。

- **举例理解**

你早上去买现炸油条， 不用去排队了，打开美团外卖下单，然后做其它事，一会外卖自己送上门。(你就是应用级别，店家就是操作系统级别, 应用无需阻塞，这就是非阻塞；系统还可能在处理中，但是立刻响应了应用，这就是异步)

- **Linux 中IO图例**

（Linux提供了AIO库函数实现异步，但是用的很少。目前有很多开源的异步IO库，例如libevent、libev、libuv）

![img](/images/io/java-io-model-4.png)

#### [#](#什么是reactor模型) 什么是Reactor模型？

大多数网络框架都是基于Reactor模型进行设计和开发，Reactor模型基于事件驱动，特别适合处理海量的I/O事件。

- **传统的IO模型**？

这种模式是传统设计，每一个请求到来时，大致都会按照：请求读取->请求解码->服务执行->编码响应->发送答复 这个流程去处理。

![img](/images/io/java-io-reactor-1.png)

服务器会分配一个线程去处理，如果请求暴涨起来，那么意味着需要更多的线程来处理该请求。若请求出现暴涨，线程池的工作线程数量满载那么其它请求就会出现等待或者被抛弃。若每个小任务都可以使用非阻塞的模式，然后基于异步回调模式。这样就大大提高系统的吞吐量，这便引入了Reactor模型。

- **Reactor模型中定义的三种角色**：

1. **Reactor**：负责监听和分配事件，将I/O事件分派给对应的Handler。新的事件包含连接建立就绪、读就绪、写就绪等。
2. **Acceptor**：处理客户端新连接，并分派请求到处理器链中。
3. **Handler**：将自身与事件绑定，执行非阻塞读/写任务，完成channel的读入，完成处理业务逻辑后，负责将结果写出channel。可用资源池来管理。

- **单Reactor单线程模型**

Reactor线程负责多路分离套接字，accept新连接，并分派请求到handler。Redis使用单Reactor单进程的模型。

![img](/images/io/java-io-reactor-2.png)

消息处理流程：

1. Reactor对象通过select监控连接事件，收到事件后通过dispatch进行转发。
2. 如果是连接建立的事件，则由acceptor接受连接，并创建handler处理后续事件。
3. 如果不是建立连接事件，则Reactor会分发调用Handler来响应。
4. handler会完成read->业务处理->send的完整业务流程。

- **单Reactor多线程模型**

将handler的处理池化。

![img](/images/io/java-io-reactor-3.png)

- **多Reactor多线程模型**

主从Reactor模型： 主Reactor用于响应连接请求，从Reactor用于处理IO操作请求，读写分离了。

![img](/images/io/java-io-reactor-4.png)

#### [#](#什么是java-nio) 什么是Java NIO？

NIO主要有三大核心部分：Channel(通道)，Buffer(缓冲区), Selector。**传统IO基于字节流和字符流进行操作**，而**NIO基于Channel和Buffer(缓冲区)进行操作**，数据总是从通道读取到缓冲区中，或者从缓冲区写入到通道中。Selector(选择区)用于监听多个通道的事件（比如：连接打开，数据到达）。因此，单个线程可以监听多个数据通道。

NIO和传统IO（一下简称IO）之间第一个最大的区别是，IO是面向流的，NIO是面向缓冲区的。

![img](/images/io/java-io-nio-x.png)

### [#](#_4-3-零拷贝) 4.3 零拷贝

#### [#](#传统的io存在什么问题-为什么引入零拷贝的) 传统的IO存在什么问题？为什么引入零拷贝的？

如果服务端要提供文件传输的功能，我们能想到的最简单的方式是：将磁盘上的文件读取出来，然后通过网络协议发送给客户端。

传统 I/O 的工作方式是，数据读取和写入是从用户空间到内核空间来回复制，而内核空间的数据是通过操作系统层面的 I/O 接口从磁盘读取或写入。

代码通常如下，一般会需要两个系统调用：

```c
read(file, tmp_buf, len);
write(socket, tmp_buf, len);
```

代码很简单，虽然就两行代码，但是这里面发生了不少的事情。

![img](/images/io/java-io-copy-3.png)

首先，**期间共发生了 4 次用户态与内核态的上下文切换**，因为发生了两次系统调用，一次是 read() ，一次是 write()，每次系统调用都得先从用户态切换到内核态，等内核完成任务后，再从内核态切换回用户态。

上下文切换到成本并不小，一次切换需要耗时几十纳秒到几微秒，虽然时间看上去很短，但是在高并发的场景下，这类时间容易被累积和放大，从而影响系统的性能。

其次，还发生了 **4 次数据拷贝**，其中**两次是 DMA 的拷贝**，另外**两次则是通过 CPU 拷贝**的，下面说一下这个过程：

- **第一次拷贝**，把磁盘上的数据拷贝到操作系统内核的缓冲区里，这个拷贝的过程是通过 DMA 搬运的。
- **第二次拷贝**，把内核缓冲区的数据拷贝到用户的缓冲区里，于是我们应用程序就可以使用这部分数据了，这个拷贝到过程是由 CPU 完成的。
- **第三次拷贝**，把刚才拷贝到用户的缓冲区里的数据，再拷贝到内核的 socket 的缓冲区里，这个过程依然还是由 CPU 搬运的。
- **第四次拷贝**，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程又是由 DMA 搬运的。

我们回过头看这个文件传输的过程，我们只是搬运一份数据，结果却搬运了 4 次，过多的数据拷贝无疑会消耗 CPU 资源，大大降低了系统性能。

这种简单又传统的文件传输方式，存在冗余的上文切换和数据拷贝，在高并发系统里是非常糟糕的，多了很多不必要的开销，会严重影响系统性能。

所以，**要想提高文件传输的性能，就需要减少「用户态与内核态的上下文切换」和「内存拷贝」的次数**。

#### [#](#mmap-write怎么实现的零拷贝) mmap + write怎么实现的零拷贝？

在前面我们知道，read() 系统调用的过程中会把内核缓冲区的数据拷贝到用户的缓冲区里，于是为了减少这一步开销，我们可以用 mmap() 替换 read() 系统调用函数。

```c
buf = mmap(file, len);
write(sockfd, buf, len);
```

mmap() 系统调用函数会直接把内核缓冲区里的数据「映射」到用户空间，这样，操作系统内核与用户空间就不需要再进行任何的数据拷贝操作。

![img](/images/io/java-io-copy-4.png)

具体过程如下：

- 应用进程调用了 mmap() 后，DMA 会把磁盘的数据拷贝到内核的缓冲区里。接着，应用进程跟操作系统内核「共享」这个缓冲区；
- 应用进程再调用 write()，操作系统直接将内核缓冲区的数据拷贝到 socket 缓冲区中，这一切都发生在内核态，由 CPU 来搬运数据；
- 最后，把内核的 socket 缓冲区里的数据，拷贝到网卡的缓冲区里，这个过程是由 DMA 搬运的。

我们可以得知，通过使用 mmap() 来代替 read()， 可以减少一次数据拷贝的过程。

但这还不是最理想的零拷贝，因为仍然需要通过 CPU 把内核缓冲区的数据拷贝到 socket 缓冲区里，而且仍然需要 4 次上下文切换，因为系统调用还是 2 次。

#### [#](#sendfile怎么实现的零拷贝) sendfile怎么实现的零拷贝？

在 Linux 内核版本 2.1 中，提供了一个专门发送文件的系统调用函数 sendfile()，函数形式如下：

```c
#include <sys/socket.h>
ssize_t sendfile(int out_fd, int in_fd, off_t *offset, size_t count);
```

它的前两个参数分别是目的端和源端的文件描述符，后面两个参数是源端的偏移量和复制数据的长度，返回值是实际复制数据的长度。

首先，它可以替代前面的 read() 和 write() 这两个系统调用，这样就可以减少一次系统调用，也就减少了 2 次上下文切换的开销。

其次，该系统调用，可以直接把内核缓冲区里的数据拷贝到 socket 缓冲区里，不再拷贝到用户态，这样就只有 2 次上下文切换，和 3 次数据拷贝。如下图：

![img](/images/io/java-io-copy-5.png)

但是这还不是真正的零拷贝技术，如果网卡支持 SG-DMA（**The Scatter-Gather Direct Memory Access**）技术（和普通的 DMA 有所不同），我们可以进一步减少通过 CPU 把内核缓冲区里的数据拷贝到 socket 缓冲区的过程。

你可以在你的 Linux 系统通过下面这个命令，查看网卡是否支持 scatter-gather 特性：

```c
$ ethtool -k eth0 | grep scatter-gather
scatter-gather: on
```

于是，从 Linux 内核 2.4 版本开始起，对于支持网卡支持 SG-DMA 技术的情况下， sendfile() 系统调用的过程发生了点变化，具体过程如下：

- 第一步，通过 DMA 将磁盘上的数据拷贝到内核缓冲区里；
- 第二步，缓冲区描述符和数据长度传到 socket 缓冲区，这样网卡的 SG-DMA 控制器就可以直接将内核缓存中的数据拷贝到网卡的缓冲区里，此过程不需要将数据从操作系统内核缓冲区拷贝到 socket 缓冲区中，这样就减少了一次数据拷贝；

所以，这个过程之中，只进行了 2 次数据拷贝，如下图：

![img](/images/io/java-io-copy-6.png)

这就是所谓的**零拷贝（Zero-copy）技术，因为我们没有在内存层面去拷贝数据，也就是说全程没有通过 CPU 来搬运数据，所有的数据都是通过 DMA 来进行传输的**。

零拷贝技术的文件传输方式相比传统文件传输的方式，**减少了 2 次上下文切换和数据拷贝次数，只需要 2 次上下文切换和数据拷贝次数，就可以完成文件的传输，而且 2 次的数据拷贝过程，都不需要通过 CPU，2 次都是由 DMA 来搬运**。

## [#](#_5-jvm和调优) 5 JVM和调优

> JVM虚拟机和调优相关。

### [#](#_5-1-类加载机制) 5.1 类加载机制

#### [#](#类加载的生命周期) 类加载的生命周期？

其中类加载的过程包括了**加载、验证、准备、解析、初始化**五个阶段。在这五个阶段中，加载、验证、准备和初始化这四个阶段发生的顺序是确定的，而解析阶段则不一定，它在某些情况下可以在初始化阶段之后开始，这是为了支持Java语言的运行时绑定(也成为动态绑定或晚期绑定)*。另外注意这里的几个阶段是按顺序开始，而不是按顺序进行或完成，因为这些阶段通常都是互相交叉地混合进行的，通常在一个阶段执行的过程中调用或激活另一个阶段。

![img](https://pdai.tech/images/jvm/java_jvm_classload_2.png)

- 类的加载: 查找并加载类的二进制数据
- 连接 
  - 验证: 确保被加载的类的正确性
  - 准备: 为类的静态变量分配内存，并将其初始化为默认值
  - 解析: 把类中的符号引用转换为直接引用
- 初始化：为类的静态变量赋予正确的初始值，JVM负责对类进行初始化，主要对类变量进行初始化。
- 使用： 类访问方法区内的数据结构的接口， 对象是Heap区的数据
- 卸载： 结束生命周期

#### [#](#类加载器的层次) 类加载器的层次?

![img](https://pdai.tech/images/jvm/java_jvm_classload_3.png)

- **启动类加载器**: Bootstrap ClassLoader，负责加载存放在JDK\jre\lib(JDK代表JDK的安装目录，下同)下，或被-Xbootclasspath参数指定的路径中的，并且能被虚拟机识别的类库(如rt.jar，所有的java.*开头的类均被Bootstrap ClassLoader加载)。启动类加载器是无法被Java程序直接引用的。
- **扩展类加载器**: Extension ClassLoader，该加载器由`sun.misc.Launcher$ExtClassLoader`实现，它负责加载JDK\jre\lib\ext目录中，或者由java.ext.dirs系统变量指定的路径中的所有类库(如javax.*开头的类)，开发者可以直接使用扩展类加载器。
- **应用程序类加载器**: Application ClassLoader，该类加载器由`sun.misc.Launcher$AppClassLoader`来实现，它负责加载用户类路径(ClassPath)所指定的类，开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中默认的类加载器。
- **自定义类加载器**: 因为JVM自带的ClassLoader只是懂得从本地文件系统加载标准的java class文件，因此如果编写了自己的ClassLoader，便可以做到如下几点:
  - 在执行非置信代码之前，自动验证数字签名。
  - 动态地创建符合用户特定需要的定制化构建类。
  - 从特定的场所取得java class，例如数据库中和网络中。

#### [#](#class-forname-和classloader-loadclass-区别) Class.forName()和ClassLoader.loadClass()区别?

- `Class.forName()`: 将类的.class文件加载到jvm中之外，还会对类进行解释，执行类中的static块；
- `ClassLoader.loadClass()`: 只干一件事情，就是将.class文件加载到jvm中，不会执行static中的内容,只有在newInstance才会去执行static块。
- `Class.forName(name, initialize, loader)`带参函数也可控制是否加载static块。并且只有调用了newInstance()方法采用调用构造函数，创建类的对象 。

#### [#](#jvm有哪些类加载机制) JVM有哪些类加载机制？

- **JVM类加载机制有哪些**？

1. **全盘负责**，当一个类加载器负责加载某个Class时，该Class所依赖的和引用的其他Class也将由该类加载器负责载入，除非显示使用另外一个类加载器来载入
2. **父类委托**，先让父类加载器试图加载该类，只有在父类加载器无法加载该类时才尝试从自己的类路径中加载该类
3. **缓存机制**，缓存机制将会保证所有加载过的Class都会被缓存，当程序中需要使用某个Class时，类加载器先从缓存区寻找该Class，只有缓存区不存在，系统才会读取该类对应的二进制数据，并将其转换成Class对象，存入缓存区。这就是为什么修改了Class后，必须重启JVM，程序的修改才会生效
4. **双亲委派机制**, 如果一个类加载器收到了类加载的请求，它首先不会自己去尝试加载这个类，而是把请求委托给父加载器去完成，依次向上，因此，所有的类加载请求最终都应该被传递到顶层的启动类加载器中，只有当父加载器在它的搜索范围中没有找到所需的类时，即无法完成该加载，子加载器才会尝试自己去加载该类。

- **双亲委派机制过程？**

1. 当AppClassLoader加载一个class时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器ExtClassLoader去完成。
2. 当ExtClassLoader加载一个class时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给BootStrapClassLoader去完成。
3. 如果BootStrapClassLoader加载失败(例如在$JAVA_HOME/jre/lib里未查找到该class)，会使用ExtClassLoader来尝试加载；
4. 若ExtClassLoader也加载失败，则会使用AppClassLoader来加载，如果AppClassLoader也加载失败，则会报出异常ClassNotFoundException。

### [#](#_5-2-内存结构) 5.2 内存结构

#### [#](#说说jvm内存整体的结构-线程私有还是共享的) 说说JVM内存整体的结构？线程私有还是共享的？

JVM 整体架构，中间部分就是 Java 虚拟机定义的各种运行时数据区域。

![jvm-framework](https://tva1.sinaimg.cn/large/0082zybply1gc6fz21n8kj30u00wpn5v.jpg)

Java 虚拟机定义了若干种程序运行期间会使用到的运行时数据区，其中有一些会随着虚拟机启动而创建，随着虚拟机退出而销毁。另外一些则是与线程一一对应的，这些与线程一一对应的数据区域会随着线程开始和结束而创建和销毁。

- **线程私有**：程序计数器、虚拟机栈、本地方法区
- **线程共享**：堆、方法区, 堆外内存（Java7的永久代或JDK8的元空间、代码缓存）

#### [#](#什么是程序计数器-线程私有) 什么是程序计数器（线程私有）？

PC 寄存器用来存储指向下一条指令的地址，即将要执行的指令代码。由执行引擎读取下一条指令。

- **PC寄存器为什么会被设定为线程私有的？**

多线程在一个特定的时间段内只会执行其中某一个线程方法，CPU会不停的做任务切换，这样必然会导致经常中断或恢复。为了能够准确的记录各个线程正在执行的当前字节码指令地址，所以为每个线程都分配了一个PC寄存器，每个线程都独立计算，不会互相影响。

#### [#](#什么是虚拟机栈-线程私有) 什么是虚拟机栈（线程私有）？

主管 Java 程序的运行，它保存方法的局部变量、部分结果，并参与方法的调用和返回。每个线程在创建的时候都会创建一个虚拟机栈，其内部保存一个个的栈帧(Stack Frame），对应着一次次 Java 方法调用，是线程私有的，生命周期和线程一致。

- **特点？**

1. 栈是一种快速有效的分配存储方式，访问速度仅次于程序计数器
2. JVM 直接对虚拟机栈的操作只有两个：每个方法执行，伴随着**入栈**（进栈/压栈），方法执行结束**出栈**
3. 栈不存在垃圾回收问题
4. 可以通过参数`-Xss`来设置线程的最大栈空间，栈的大小直接决定了函数调用的最大可达深度

- **该区域有哪些异常**？

1. 如果采用固定大小的 Java 虚拟机栈，那每个线程的 Java 虚拟机栈容量可以在线程创建的时候独立选定。如果线程请求分配的栈容量超过 Java 虚拟机栈允许的最大容量，Java 虚拟机将会抛出一个 **StackOverflowError** 异常
2. 如果 Java 虚拟机栈可以动态扩展，并且在尝试扩展的时候无法申请到足够的内存，或者在创建新的线程时没有足够的内存去创建对应的虚拟机栈，那 Java 虚拟机将会抛出一个**OutOfMemoryError**异常

- **栈帧的内部结构？**

1. 局部变量表（Local Variables）
2. 操作数栈（Operand Stack）(或称为表达式栈)
3. 动态链接（Dynamic Linking）：指向运行时常量池的方法引用
4. 方法返回地址（Return Address）：方法正常退出或异常退出的地址
5. 一些附加信息

![jvm-stack-frame](https://tva1.sinaimg.cn/large/0082zybply1gc8tjehg8bj318m0lbtbu.jpg)

#### [#](#java虚拟机栈如何进行方法计算的) Java虚拟机栈如何进行方法计算的？

以如下代码为例：

```java
private static int add(int a, int b) {
    int c = 0;
    c = a + b;
    return c;
}
```

可以通过jsclass 等工具查看bytecode

![img](/images/jvm/java-jvm-stack-2.png)

压栈的步骤如下：

```java
0:   iconst_0 // 0压栈
1:   istore_2 // 弹出int，存放于局部变量2
2:   iload_0  // 把局部变量0压栈
3:   iload_1  // 局部变量1压栈
4:   iadd     //弹出2个变量，求和，结果压栈
5:   istore_2 //弹出结果，放于局部变量2
6:   iload_2  //局部变量2压栈
7:   ireturn  //返回
```

如果计算100+98的值，那么操作数栈的变化如下图

![img](/images/jvm/java-jvm-stack-3.png)

#### [#](#什么是本地方法栈-线程私有) 什么是本地方法栈（线程私有）？

- **本地方法接口**

一个 Native Method 就是一个 Java 调用非 Java 代码的接口。我们知道的 Unsafe 类就有很多本地方法。

- **本地方法栈(Native Method Stack)**

Java 虚拟机栈用于管理 Java 方法的调用，而本地方法栈用于管理本地方法的调用

#### [#](#什么是方法区-线程共享) 什么是方法区（线程共享）？

方法区（method area）只是 **JVM 规范**中定义的一个概念，用于存储类信息、常量池、静态变量、JIT编译后的代码等数据，并没有规定如何去实现它，不同的厂商有不同的实现。而**永久代（PermGen）\**是 \*\*Hotspot\*\* 虚拟机特有的概念， Java8 的时候又被\**元空间**取代了，永久代和元空间都可以理解为方法区的落地实现。

JDK1.8之前调节方法区大小：

```bash
-XX:PermSize=N //方法区（永久代）初始大小
-XX:MaxPermSize=N //方法区（永久代）最大大小，超出这个值将会抛出OutOfMemoryError 
```

JDK1.8开始方法区（HotSpot的永久代）被彻底删除了，取而代之的是元空间，元空间直接使用的是本机内存。参数设置：

```bash
-XX:MetaspaceSize=N //设置Metaspace的初始（和最小大小）
-XX:MaxMetaspaceSize=N //设置Metaspace的最大大小
```

**栈、堆、方法区的交互关系**

![img](https://pdai.tech/images/jvm/java-jvm-stack-1.png)

#### [#](#永久代和元空间内存使用上的差异) 永久代和元空间内存使用上的差异?

Java虚拟机规范中只定义了方法区用于存储已被虚拟机加载的类信息、常量、静态变量和即时编译后的代码等数据

1. jdk1.7开始符号引用存储在native heap中，字符串常量和静态类型变量存储在普通的堆区中，但分离的并不彻底,此时永久代中还保存另一些与类的元数据无关的杂项
2. jdk8后HotSpot 原永久代中存储的类的**元数据将存储在metaspace**中，而**类的静态变量和字符串常量将放在Java堆中**，metaspace是方法区的一种实现，只不过它使用的不是虚拟机内的内存，而是本地内存。在元空间中保存的数据比永久代中纯粹很多，就只是类的元数据，这些信息只对编译期或JVM的运行时有用。
3. 永久代有一个JVM本身设置固定大小上线，无法进行调整，而**元空间使用的是直接内存，受本机可用内存的限制，并且永远不会得到java.lang.OutOfMemoryError**。
4. **符号引用没有存在元空间中，而是存在native heap中**，这是两个方式和位置，不过都可以算作是本地内存，在虚拟机之外进行划分，没有设置限制参数时只受物理内存大小限制，即只有占满了操作系统可用内存后才OOM。

#### [#](#堆区内存是怎么细分的) 堆区内存是怎么细分的？

对于大多数应用，Java 堆是 Java 虚拟机管理的内存中最大的一块，被所有线程共享。此内存区域的唯一目的就是存放对象实例，几乎所有的对象实例以及数据都在这里分配内存。

为了进行高效的垃圾回收，虚拟机把堆内存**逻辑上**划分成三块区域（分代的唯一理由就是优化 GC 性能）：

1. 新生带（年轻代）：新对象和没达到一定年龄的对象都在新生代
2. 老年代（养老区）：被长时间使用的对象，老年代的内存空间应该要比年轻代更大

![JDK7](https://tva1.sinaimg.cn/large/00831rSTly1gdbr7ek6pfj30ci0560t4.jpg)

Java 虚拟机规范规定，Java 堆可以是处于物理上不连续的内存空间中，只要逻辑上是连续的即可，像磁盘空间一样。实现时，既可以是固定大小，也可以是可扩展的，主流虚拟机都是可扩展的（通过 `-Xmx` 和 `-Xms` 控制），如果堆中没有完成实例分配，并且堆无法再扩展时，就会抛出 `OutOfMemoryError` 异常。

- **年轻代 (Young Generation)**

年轻代是所有新对象创建的地方。当填充年轻代时，执行垃圾收集。这种垃圾收集称为 **Minor GC**。年轻一代被分为三个部分——伊甸园（**Eden Memory**）和两个幸存区（**Survivor Memory**，被称为from/to或s0/s1），默认比例是`8:1:1`

1. 大多数新创建的对象都位于 Eden 内存空间中
2. 当 Eden 空间被对象填充时，执行**Minor GC**，并将所有幸存者对象移动到一个幸存者空间中
3. Minor GC 检查幸存者对象，并将它们移动到另一个幸存者空间。所以每次，一个幸存者空间总是空的
4. 经过多次 GC 循环后存活下来的对象被移动到老年代。通常，这是通过设置年轻一代对象的年龄阈值来实现的，然后他们才有资格提升到老一代

- **老年代(Old Generation)**

旧的一代内存包含那些经过许多轮小型 GC 后仍然存活的对象。通常，垃圾收集是在老年代内存满时执行的。老年代垃圾收集称为 主GC（Major GC），通常需要更长的时间。

大对象直接进入老年代（大对象是指需要大量连续内存空间的对象）。这样做的目的是避免在 Eden 区和两个Survivor 区之间发生大量的内存拷贝

![img](https://tva1.sinaimg.cn/large/007S8ZIlly1gg06065oa9j31kw0u0q69.jpg)

#### [#](#jvm中对象在堆中的生命周期) JVM中对象在堆中的生命周期?

1. 在 JVM 内存模型的堆中，堆被划分为新生代和老年代 
   - 新生代又被进一步划分为 **Eden区** 和 **Survivor区**，Survivor 区由 **From Survivor** 和 **To Survivor** 组成
2. 当创建一个对象时，对象会被优先分配到新生代的 Eden 区 
   - 此时 JVM 会给对象定义一个**对象年轻计数器**（`-XX:MaxTenuringThreshold`）
3. 当 Eden 空间不足时，JVM 将执行新生代的垃圾回收（Minor GC） 
   - JVM 会把存活的对象转移到 Survivor 中，并且对象年龄 +1
   - 对象在 Survivor 中同样也会经历 Minor GC，每经历一次 Minor GC，对象年龄都会+1
4. 如果分配的对象超过了`-XX:PetenureSizeThreshold`，对象会**直接被分配到老年代**

#### [#](#jvm中对象的分配过程) JVM中对象的分配过程?

为对象分配内存是一件非常严谨和复杂的任务，JVM 的设计者们不仅需要考虑内存如何分配、在哪里分配等问题，并且由于内存分配算法和内存回收算法密切相关，所以还需要考虑 GC 执行完内存回收后是否会在内存空间中产生内存碎片。

1. new 的对象先放在伊甸园区，此区有大小限制
2. 当伊甸园的空间填满时，程序又需要创建对象，JVM 的垃圾回收器将对伊甸园区进行垃圾回收（Minor GC），将伊甸园区中的不再被其他对象所引用的对象进行销毁。再加载新的对象放到伊甸园区
3. 然后将伊甸园中的剩余对象移动到幸存者 0 区
4. 如果再次触发垃圾回收，此时上次幸存下来的放到幸存者 0 区，如果没有回收，就会放到幸存者 1 区
5. 如果再次经历垃圾回收，此时会重新放回幸存者 0 区，接着再去幸存者 1 区
6. 什么时候才会去养老区呢？ 默认是 15 次回收标记
7. 在养老区，相对悠闲。当养老区内存不足时，再次触发 Major GC，进行养老区的内存清理
8. 若养老区执行了 Major GC 之后发现依然无法进行对象的保存，就会产生 OOM 异常

#### [#](#什么是-tlab-thread-local-allocation-buffer) 什么是 TLAB （Thread Local Allocation Buffer）?

- 从内存模型而不是垃圾回收的角度，对 Eden 区域继续进行划分，JVM 为每个线程分配了一个私有缓存区域，它包含在 Eden 空间内
- 多线程同时分配内存时，使用 TLAB 可以避免一系列的非线程安全问题，同时还能提升内存分配的吞吐量，因此我们可以将这种内存分配方式称为**快速分配策略**
- OpenJDK 衍生出来的 JVM 大都提供了 TLAB 设计

#### [#](#为什么要有-tlab) 为什么要有 TLAB ?

- 堆区是线程共享的，任何线程都可以访问到堆区中的共享数据
- 由于对象实例的创建在 JVM 中非常频繁，因此在并发环境下从堆区中划分内存空间是线程不安全的
- 为避免多个线程操作同一地址，需要使用加锁等机制，进而影响分配速度

尽管不是所有的对象实例都能够在 TLAB 中成功分配内存，但 JVM 确实是将 TLAB 作为内存分配的首选。

在程序中，可以通过 `-XX:UseTLAB` 设置是否开启 TLAB 空间。

默认情况下，TLAB 空间的内存非常小，仅占有整个 Eden 空间的 1%，我们可以通过 `-XX:TLABWasteTargetPercent` 设置 TLAB 空间所占用 Eden 空间的百分比大小。

一旦对象在 TLAB 空间分配内存失败时，JVM 就会尝试着通过使用加锁机制确保数据操作的原子性，从而直接在 Eden 空间中分配内存。

### [#](#_5-3-gc垃圾回收) 5.3 GC垃圾回收

#### [#](#如何判断一个对象是否可以回收) 如何判断一个对象是否可以回收？

- **引用计数算法**

给对象添加一个引用计数器，当对象增加一个引用时计数器加 1，引用失效时计数器减 1。引用计数为 0 的对象可被回收。

两个对象出现循环引用的情况下，此时引用计数器永远不为 0，导致无法对它们进行回收。

正因为循环引用的存在，因此 Java 虚拟机不使用引用计数算法。

- **可达性分析算法**

通过 GC Roots 作为起始点进行搜索，能够到达到的对象都是存活的，不可达的对象可被回收。

![image](/images/pics/0635cbe8.png)

Java 虚拟机使用该算法来判断对象是否可被回收，在 Java 中 GC Roots 一般包含以下内容:

- 虚拟机栈中引用的对象
- 本地方法栈中引用的对象
- 方法区中类静态属性引用的对象
- 方法区中的常量引用的对象

#### [#](#对象有哪些引用类型) 对象有哪些引用类型？

无论是通过引用计算算法判断对象的引用数量，还是通过可达性分析算法判断对象是否可达，判定对象是否可被回收都与引用有关。

Java 具有四种强度不同的引用类型。

- **强引用**

被强引用关联的对象不会被回收。

使用 new 一个新对象的方式来创建强引用。

```java
Object obj = new Object();
```

- **软引用**

被软引用关联的对象只有在内存不够的情况下才会被回收。

使用 SoftReference 类来创建软引用。

```java
Object obj = new Object();
SoftReference<Object> sf = new SoftReference<Object>(obj);
obj = null;  // 使对象只被软引用关联
```

- **弱引用**

被弱引用关联的对象一定会被回收，也就是说它只能存活到下一次垃圾回收发生之前。

使用 WeakReference 类来实现弱引用。

```java
Object obj = new Object();
WeakReference<Object> wf = new WeakReference<Object>(obj);
obj = null;
```

- **虚引用**

又称为幽灵引用或者幻影引用。一个对象是否有虚引用的存在，完全不会对其生存时间构成影响，也无法通过虚引用取得一个对象。

为一个对象设置虚引用关联的唯一目的就是能在这个对象被回收时收到一个系统通知。

使用 PhantomReference 来实现虚引用。

```java
Object obj = new Object();
PhantomReference<Object> pf = new PhantomReference<Object>(obj);
obj = null;
```

#### [#](#有哪些基本的垃圾回收算法) 有哪些基本的垃圾回收算法？

- **标记 - 清除**

![image](https://pdai.tech/images/pics/a4248c4b-6c1d-4fb8-a557-86da92d3a294.jpg)

将存活的对象进行标记，然后清理掉未被标记的对象。

不足:

- 标记和清除过程效率都不高；
- 会产生大量不连续的内存碎片，导致无法给大对象分配内存。

- **标记 - 整理**

![image](https://pdai.tech/images/pics/902b83ab-8054-4bd2-898f-9a4a0fe52830.jpg)

让所有存活的对象都向一端移动，然后直接清理掉端边界以外的内存。

- **复制**

![image](https://pdai.tech/images/pics/e6b733ad-606d-4028-b3e8-83c3a73a3797.jpg)

将内存划分为大小相等的两块，每次只使用其中一块，当这一块内存用完了就将还存活的对象复制到另一块上面，然后再把使用过的内存空间进行一次清理。

主要不足是只使用了内存的一半。

现在的商业虚拟机都采用这种收集算法来回收新生代，但是并不是将新生代划分为大小相等的两块，而是分为一块较大的 Eden 空间和两块较小的 Survivor 空间，每次使用 Eden 空间和其中一块 Survivor。在回收时，将 Eden 和 Survivor 中还存活着的对象一次性复制到另一块 Survivor 空间上，最后清理 Eden 和使用过的那一块 Survivor。

HotSpot 虚拟机的 Eden 和 Survivor 的大小比例默认为 8:1，保证了内存的利用率达到 90%。如果每次回收有多于 10% 的对象存活，那么一块 Survivor 空间就不够用了，此时需要依赖于老年代进行分配担保，也就是借用老年代的空间存储放不下的对象。

- **分代收集**

现在的商业虚拟机采用分代收集算法，它根据对象存活周期将内存划分为几块，不同块采用适当的收集算法。

一般将堆分为新生代和老年代。

- 新生代使用: 复制算法
- 老年代使用: 标记 - 清除 或者 标记 - 整理 算法

#### [#](#分代收集算法和分区收集算法区别) 分代收集算法和分区收集算法区别？

![img](/images/jvm/cmsgc/cms-gc-5.jpg)

- **分代收集算法**

当前主流 VM 垃圾收集都采用”分代收集”(Generational Collection)算法, 这种算法会根据 对象存活周期的不同将内存划分为几块, 如 JVM 中的 新生代、老年代、永久代，这样就可以根据 各年代特点分别采用最适当的 GC 算法

在新生代-复制算法：

每次垃圾收集都能发现大批对象已死, 只有少量存活. 因此选用复制算法, 只需要付出少量 存活对象的复制成本就可以完成收集

在老年代-标记整理算法：

因为对象存活率高、没有额外空间对它进行分配担保, 就必须采用“标记—清理”或“标 记—整理”算法来进行回收, 不必进行内存复制, 且直接腾出空闲内存.

1. **ParNew**： 一款多线程的收集器，采用复制算法，主要工作在 Young 区，可以通过 `-XX:ParallelGCThreads` 参数来控制收集的线程数，整个过程都是 STW 的，常与 CMS 组合使用。
2. **CMS**： 以获取最短回收停顿时间为目标，采用“标记-清除”算法，分 4 大步进行垃圾收集，其中初始标记和重新标记会 STW ，多数应用于互联网站或者 B/S 系统的服务器端上，JDK9 被标记弃用，JDK14 被删除。

- **分区收集算法**

分区算法则将整个堆空间划分为连续的不同小区间, 每个小区间独立使用, 独立回收. 这样做的 好处是可以控制一次回收多少个小区间 , 根据目标停顿时间, 每次合理地回收若干个小区间(而不是 整个堆), 从而减少一次 GC 所产生的停顿。

1. **G1**： 一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中，在实现高吞吐量的同时，尽可能地满足垃圾收集暂停时间的要求。
2. **ZGC**： JDK11 中推出的一款低延迟垃圾回收器，适用于大内存低延迟服务的内存管理和回收，SPECjbb 2015 基准测试，在 128G 的大堆下，最大停顿时间才 1.68 ms，停顿时间远胜于 G1 和 CMS。

#### [#](#什么是minor-gc、major-gc、full-gc) 什么是Minor GC、Major GC、Full GC?

JVM 在进行 GC 时，并非每次都对堆内存（新生代、老年代；方法区）区域一起回收的，大部分时候回收的都是指新生代。

针对 HotSpot VM 的实现，它里面的 GC 按照回收区域又分为两大类：部分收集（Partial GC），整堆收集（Full GC）

- 部分收集：不是完整收集整个 Java 堆的垃圾收集。其中又分为： 
  - 新生代收集（Minor GC/Young GC）：只是新生代的垃圾收集
  - 老年代收集（Major GC/Old GC）：只是老年代的垃圾收集 
    - 目前，只有 CMS GC 会有单独收集老年代的行为
    - 很多时候 Major GC 会和 Full GC 混合使用，需要具体分辨是老年代回收还是整堆回收
  - 混合收集（Mixed GC）：收集整个新生代以及部分老年代的垃圾收集 
    - 目前只有 G1 GC 会有这种行为
- 整堆收集（Full GC）：收集整个 Java 堆和方法区的垃圾

#### [#](#说说jvm内存分配策略) 说说JVM内存分配策略？

- **对象优先在 Eden 分配**

大多数情况下，对象在新生代 Eden 区分配，当 Eden 区空间不够时，发起 Minor GC。

- **大对象直接进入老年代**

大对象是指需要连续内存空间的对象，最典型的大对象是那种很长的字符串以及数组。

经常出现大对象会提前触发垃圾收集以获取足够的连续空间分配给大对象。

-XX:PretenureSizeThreshold，大于此值的对象直接在老年代分配，避免在 Eden 区和 Survivor 区之间的大量内存复制。

- **长期存活的对象进入老年代**

为对象定义年龄计数器，对象在 Eden 出生并经过 Minor GC 依然存活，将移动到 Survivor 中，年龄就增加 1 岁，增加到一定年龄则移动到老年代中。

-XX:MaxTenuringThreshold 用来定义年龄的阈值。

- **动态对象年龄判定**

虚拟机并不是永远地要求对象的年龄必须达到 MaxTenuringThreshold 才能晋升老年代，如果在 Survivor 中相同年龄所有对象大小的总和大于 Survivor 空间的一半，则年龄大于或等于该年龄的对象可以直接进入老年代，无需等到 MaxTenuringThreshold 中要求的年龄。

- **空间分配担保**

在发生 Minor GC 之前，虚拟机先检查老年代最大可用的连续空间是否大于新生代所有对象总空间，如果条件成立的话，那么 Minor GC 可以确认是安全的。

如果不成立的话虚拟机会查看 HandlePromotionFailure 设置值是否允许担保失败，如果允许那么就会继续检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小，如果大于，将尝试着进行一次 Minor GC；如果小于，或者 HandlePromotionFailure 设置不允许冒险，那么就要进行一次 Full GC。

#### [#](#什么情况下会触发full-gc) 什么情况下会触发Full GC？

对于 Minor GC，其触发条件非常简单，当 Eden 空间满时，就将触发一次 Minor GC。而 Full GC 则相对复杂，有以下条件:

- **调用 System.gc()**

只是建议虚拟机执行 Full GC，但是虚拟机不一定真正去执行。不建议使用这种方式，而是让虚拟机管理内存。

- **老年代空间不足**

老年代空间不足的常见场景为前文所讲的大对象直接进入老年代、长期存活的对象进入老年代等。

为了避免以上原因引起的 Full GC，应当尽量不要创建过大的对象以及数组。除此之外，可以通过 -Xmn 虚拟机参数调大新生代的大小，让对象尽量在新生代被回收掉，不进入老年代。还可以通过 -XX:MaxTenuringThreshold 调大对象进入老年代的年龄，让对象在新生代多存活一段时间。

- **空间分配担保失败**

使用复制算法的 Minor GC 需要老年代的内存空间作担保，如果担保失败会执行一次 Full GC。

- **JDK 1.7 及以前的永久代空间不足**

在 JDK 1.7 及以前，HotSpot 虚拟机中的方法区是用永久代实现的，永久代中存放的为一些 Class 的信息、常量、静态变量等数据。

当系统中要加载的类、反射的类和调用的方法较多时，永久代可能会被占满，在未配置为采用 CMS GC 的情况下也会执行 Full GC。如果经过 Full GC 仍然回收不了，那么虚拟机会抛出 java.lang.OutOfMemoryError。

为避免以上原因引起的 Full GC，可采用的方法为增大永久代空间或转为使用 CMS GC。

- **Concurrent Mode Failure**

执行 CMS GC 的过程中同时有对象要放入老年代，而此时老年代空间不足(可能是 GC 过程中浮动垃圾过多导致暂时性的空间不足)，便会报 Concurrent Mode Failure 错误，并触发 Full GC。

#### [#](#hotspot中有哪些垃圾回收器) Hotspot中有哪些垃圾回收器？

![image](https://pdai.tech/images/pics/c625baa0-dde6-449e-93df-c3a67f2f430f.jpg)

以上是 HotSpot 虚拟机中的 7 个垃圾收集器，连线表示垃圾收集器可以配合使用。

- 单线程与多线程: 单线程指的是垃圾收集器只使用一个线程进行收集，而多线程使用多个线程；
- 串行与并行: 串行指的是垃圾收集器与用户程序交替执行，这意味着在执行垃圾收集的时候需要停顿用户程序；并形指的是垃圾收集器和用户程序同时执行。除了 CMS 和 G1 之外，其它垃圾收集器都是以串行的方式执行。

1. **Serial 收集器**

![image](https://pdai.tech/images/pics/22fda4ae-4dd5-489d-ab10-9ebfdad22ae0.jpg)

Serial 翻译为串行，也就是说它以串行的方式执行。

它是单线程的收集器，只会使用一个线程进行垃圾收集工作。

它的优点是简单高效，对于单个 CPU 环境来说，由于没有线程交互的开销，因此拥有最高的单线程收集效率。

它是 Client 模式下的默认新生代收集器，因为在用户的桌面应用场景下，分配给虚拟机管理的内存一般来说不会很大。Serial 收集器收集几十兆甚至一两百兆的新生代停顿时间可以控制在一百多毫秒以内，只要不是太频繁，这点停顿是可以接受的。

1. **ParNew 收集器**

![image](https://pdai.tech/images/pics/81538cd5-1bcf-4e31-86e5-e198df1e013b.jpg)

它是 Serial 收集器的多线程版本。

是 Server 模式下的虚拟机首选新生代收集器，除了性能原因外，主要是因为除了 Serial 收集器，只有它能与 CMS 收集器配合工作。

默认开启的线程数量与 CPU 数量相同，可以使用 -XX:ParallelGCThreads 参数来设置线程数。

1. **Parallel Scavenge 收集器**

与 ParNew 一样是多线程收集器。

其它收集器关注点是尽可能缩短垃圾收集时用户线程的停顿时间，而它的目标是达到一个可控制的吞吐量，它被称为“吞吐量优先”收集器。这里的吞吐量指 CPU 用于运行用户代码的时间占总时间的比值。

停顿时间越短就越适合需要与用户交互的程序，良好的响应速度能提升用户体验。而高吞吐量则可以高效率地利用 CPU 时间，尽快完成程序的运算任务，主要适合在后台运算而不需要太多交互的任务。

缩短停顿时间是以牺牲吞吐量和新生代空间来换取的: 新生代空间变小，垃圾回收变得频繁，导致吞吐量下降。

可以通过一个开关参数打开 GC 自适应的调节策略(GC Ergonomics)，就不需要手动指定新生代的大小(-Xmn)、Eden 和 Survivor 区的比例、晋升老年代对象年龄等细节参数了。虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间或者最大的吞吐量。

1. **Serial Old 收集器**

![image](/images/pics/08f32fd3-f736-4a67-81ca-295b2a7972f2.jpg)

是 Serial 收集器的老年代版本，也是给 Client 模式下的虚拟机使用。如果用在 Server 模式下，它有两大用途:

- 在 JDK 1.5 以及之前版本(Parallel Old 诞生以前)中与 Parallel Scavenge 收集器搭配使用。
- 作为 CMS 收集器的后备预案，在并发收集发生 Concurrent Mode Failure 时使用。

1. **Parallel Old 收集器**

![image](/images/pics/278fe431-af88-4a95-a895-9c3b80117de3.jpg)

是 Parallel Scavenge 收集器的老年代版本。

在注重吞吐量以及 CPU 资源敏感的场合，都可以优先考虑 Parallel Scavenge 加 Parallel Old 收集器。

1. **CMS 收集器**

![image](https://pdai.tech/images/pics/62e77997-6957-4b68-8d12-bfd609bb2c68.jpg)

CMS(Concurrent Mark Sweep)，Mark Sweep 指的是标记 - 清除算法。

分为以下四个流程:

- 初始标记: 仅仅只是标记一下 GC Roots 能直接关联到的对象，速度很快，需要停顿。
- 并发标记: 进行 GC Roots Tracing 的过程，它在整个回收过程中耗时最长，不需要停顿。
- 重新标记: 为了修正并发标记期间因用户程序继续运作而导致标记产生变动的那一部分对象的标记记录，需要停顿。
- 并发清除: 不需要停顿。

在整个过程中耗时最长的并发标记和并发清除过程中，收集器线程都可以与用户线程一起工作，不需要进行停顿。

具有以下缺点:

- 吞吐量低: 低停顿时间是以牺牲吞吐量为代价的，导致 CPU 利用率不够高。
- 无法处理浮动垃圾，可能出现 Concurrent Mode Failure。浮动垃圾是指并发清除阶段由于用户线程继续运行而产生的垃圾，这部分垃圾只能到下一次 GC 时才能进行回收。由于浮动垃圾的存在，因此需要预留出一部分内存，意味着 CMS 收集不能像其它收集器那样等待老年代快满的时候再回收。如果预留的内存不够存放浮动垃圾，就会出现 Concurrent Mode Failure，这时虚拟机将临时启用 Serial Old 来替代 CMS。
- 标记 - 清除算法导致的空间碎片，往往出现老年代空间剩余，但无法找到足够大连续空间来分配当前对象，不得不提前触发一次 Full GC。

1. **G1 收集器**

G1(Garbage-First)，它是一款面向服务端应用的垃圾收集器，在多 CPU 和大内存的场景下有很好的性能。HotSpot 开发团队赋予它的使命是未来可以替换掉 CMS 收集器。

堆被分为新生代和老年代，其它收集器进行收集的范围都是整个新生代或者老年代，而 **G1 可以直接对新生代和老年代一起回收**。

![image](https://pdai.tech/images/pics/4cf711a8-7ab2-4152-b85c-d5c226733807.png)

G1 把堆划分成多个大小相等的独立区域(Region)，新生代和老年代不再物理隔离。

![image](/images/pics/9bbddeeb-e939-41f0-8e8e-2b1a0aa7e0a7.png)

**通过引入 Region 的概念，从而将原来的一整块内存空间划分成多个的小空间，使得每个小空间可以单独进行垃圾回收**。这种划分方法带来了很大的灵活性，使得可预测的停顿时间模型成为可能。通过记录每个 Region 垃圾回收时间以及回收所获得的空间(这两个值是通过过去回收的经验获得)，并维护一个优先列表，每次根据允许的收集时间，优先回收价值最大的 Region。

每个 Region 都有一个 Remembered Set，用来记录该 Region 对象的引用对象所在的 Region。通过使用 Remembered Set，在做可达性分析的时候就可以避免全堆扫描。

![image](/images/pics/f99ee771-c56f-47fb-9148-c0036695b5fe.jpg)

如果不计算维护 Remembered Set 的操作，G1 收集器的运作大致可划分为以下几个步骤:

- 初始标记
- 并发标记
- 最终标记: 为了修正在并发标记期间因用户程序继续运作而导致标记产生变动的那一部分标记记录，虚拟机将这段时间对象变化记录在线程的 Remembered Set Logs 里面，最终标记阶段需要把 Remembered Set Logs 的数据合并到 Remembered Set 中。这阶段需要停顿线程，但是可并行执行。
- 筛选回收: 首先对各个 Region 中的回收价值和成本进行排序，根据用户所期望的 GC 停顿时间来制定回收计划。此阶段其实也可以做到与用户程序一起并发执行，但是因为只回收一部分 Region，时间是用户可控制的，而且停顿用户线程将大幅度提高收集效率。

具备如下特点:

- 空间整合: 整体来看是基于“标记 - 整理”算法实现的收集器，从局部(两个 Region 之间)上来看是基于“复制”算法实现的，这意味着运行期间不会产生内存空间碎片。
- 可预测的停顿: 能让使用者明确指定在一个长度为 M 毫秒的时间片段内，消耗在 GC 上的时间不得超过 N 毫秒。

### [#](#_5-4-问题排查) 5.4 问题排查

#### [#](#常见的linux定位问题的工具) 常见的Linux定位问题的工具？

- 文本操作 
  - 文本查找 - grep
  - 文本分析 - awk
  - 文本处理 - sed
- 文件操作 
  - 文件监听 - tail
  - 文件查找 - find
- 网络和进程 
  - 网络接口 - ifconfig
  - 防火墙 - iptables -L
  - 路由表 - route -n
  - netstat
- 其它常用 
  - 进程 ps -ef | grep java
  - 分区大小 df -h
  - 内存 free -m
  - 硬盘大小 fdisk -l |grep Disk
  - top
  - 环境变量 env

#### [#](#jdk自带的定位问题的工具) JDK自带的定位问题的工具？

- **jps** jps是jdk提供的一个查看当前java进程的小工具， 可以看做是JavaVirtual Machine Process Status Tool的缩写。

```bash
jps –l # 输出输出完全的包名，应用主类名，jar的完全路径名 
```

- **jstack** jstack是jdk自带的线程堆栈分析工具，使用该命令可以查看或导出 Java 应用程序中线程堆栈信息。

```bash
# 基本
jstack 2815
jstack -m 2815 # java和native c/c++框架的所有栈信息
jstack -l 2815 # 额外的锁信息列表，查看是否死锁
```

- **jinfo** jinfo 是 JDK 自带的命令，可以用来查看正在运行的 java 应用程序的扩展参数，包括Java System属性和JVM命令行参数；也可以动态的修改正在运行的 JVM 一些参数。当系统崩溃时，jinfo可以从core文件里面知道崩溃的Java应用程序的配置信息

```bash
jinfo 2815 # 输出当前 jvm 进程的全部参数和系统属性
```

- **jmap** 命令jmap是一个多功能的命令。它可以生成 java 程序的 dump 文件， 也可以查看堆内对象示例的统计信息、查看 ClassLoader 的信息以及 finalizer 队列。

```bash
# 查看堆的情况
jmap -heap 2815

# dump
jmap -dump:live,format=b,file=/tmp/heap2.bin 2815
```

- **jstat** jstat参数众多，但是使用一个就够了

```bash
jstat -gcutil 2815 1000 
```

#### [#](#如何使用在线调试工具arthas) 如何使用在线调试工具Arthas？

举几个例子

- **查看最繁忙的线程，以及是否有阻塞情况发生**?

场景：我想看下查看最繁忙的线程，以及是否有阻塞情况发生? 常规查看线程，一般我们可以通过 top 等系统命令进行查看，但是那毕竟要很多个步骤，很麻烦。

```bash
thread -n 3 # 查看最繁忙的三个线程栈信息
thread  # 以直观的方式展现所有的线程情况
thread -b #找出当前阻塞其他线程的线程
```

- **确认某个类是否已被系统加载**?

场景：我新写了一个类或者一个方法，我想知道新写的代码是否被部署了?

```bash
# 即可以找到需要的类全路径，如果存在的话
sc *MyServlet

# 查看这个某个类所有的方法
sm pdai.tech.servlet.TestMyServlet *

# 查看某个方法的信息，如果存在的话
sm pdai.tech.servlet.TestMyServlet testMethod  
```

- **如何查看一个class类的源码信息**?

场景：我新修改的内容在方法内部，而上一个步骤只能看到方法，这时候可以反编译看下源码

```bash
# 直接反编译出java 源代码，包含一此额外信息的
jad pdai.tech.servlet.TestMyServlet
```

- **如何跟踪某个方法的返回值、入参**?

场景：我想看下我新加的方法在线运行的参数和返回值?

```bash
# 同时监控入参，返回值，及异常
watch pdai.tech.servlet.TestMyServlet testMethod "{params, returnObj, throwExp}" -e -x 2 
```

- **如何看方法调用栈的信息**?

场景：我想看下某个方法的调用栈的信息?

```bash
stack pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上

- **找到最耗时的方法调用**?

场景：testMethod这个方法入口响应很慢，如何找到最耗时的子调用?

```bash
# 执行的时候每个子调用的运行时长，可以找到最耗时的子调用。
stack pdai.tech.servlet.TestMyServlet testMethod
```

运行此命令之后需要即时触发方法才会有响应的信息打印在控制台上，然后一层一层看子调用。

- **如何临时更改代码运行**?

场景：我找到了问题所在，能否线上直接修改测试，而不需要在本地改了代码后，重新打包部署，然后重启观察效果?

```bash
# 先反编译出class源码
jad --source-only com.example.demo.arthas.user.UserController > /tmp/UserController.java  

# 然后使用外部工具编辑内容
mc /tmp/UserController.java -d /tmp  # 再编译成class

# 最后，重新载入定义的类，就可以实时验证你的猜测了
redefine /tmp/com/example/demo/arthas/user/UserController.class
```

如上，是直接更改线上代码的方式，但是一般好像是编译不成功的。所以，最好是本地ide编译成 class文件后，再上传替换为好！

总之，已经完全不用重启和发布了！这个功能真的很方便，比起重启带来的代价，真的是不可比的。比如，重启时可能导致负载重分配，选主等等问题，就不是你能控制的了。

- **我如何测试某个方法的性能问题**?

```bash
monitor -c 5 demo.MathGame primeFactors
```

#### [#](#如何使用idea的远程调试) 如何使用Idea的远程调试？

要让远程服务器运行的代码支持远程调试，则启动的时候必须加上特定的JVM参数，这些参数是：

```bash
-Xdebug -Xrunjdwp:transport=dt_socket,suspend=n,server=y,address=127.0.0.1:5555
```

#### [#](#复杂综合类型问题的定位思路) 复杂综合类型问题的定位思路？

![img](/images/jvm/java-jvm-debug.png)

## [#](#_6-java-新版本) 6 Java 新版本

> Java 8版本特性，及Java8+版本特性。

### [#](#_6-1-java-8-特性) 6.1 Java 8 特性

#### [#](#什么是函数式编程-lambda表达式) 什么是函数式编程？Lambda表达式？

- **函数式编程**

面向对象编程是对数据进行抽象；函数式编程是对行为进行抽象。

核心思想: 使用不可变值和函数，函数对一个值进行处理，映射成另一个值。

- **Lambda表达式**

lambda表达式仅能放入如下代码: 预定义使用了 `@Functional` 注释的函数式接口，自带一个抽象函数的方法，或者SAM(Single Abstract Method 单个抽象方法)类型。这些称为lambda表达式的目标类型，可以用作返回类型，或lambda目标代码的参数。例如，若一个方法接收Runnable、Comparable或者 Callable 接口，都有单个抽象方法，可以传入lambda表达式。类似的，如果一个方法接受声明于 java.util.function 包内的接口，例如 Predicate、Function、Consumer 或 Supplier，那么可以向其传lambda表达式

#### [#](#stream中常用方法) Stream中常用方法？

- `stream()`, `parallelStream()`
- `filter()`
- `findAny()` `findFirst()`
- `sort`
- `forEach` void
- `map(), reduce()`
- `flatMap()` - 将多个Stream连接成一个Stream
- `collect(Collectors.toList())`
- `distinct`, `limit`
- `count`
- `min`, `max`, `summaryStatistics`

#### [#](#什么是functionalinterface) 什么是FunctionalInterface？

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
public @interface FunctionalInterface{}
```

- interface做注解的注解类型，被定义成java语言规
- 一个被它注解的接口只能有一个抽象方法，有两种例外
- 第一是接口允许有实现的方法，这种实现的方法是用default关键字来标记的(java反射中java.lang.reflect.Method#isDefault()方法用来判断是否是default方法)
- 第二如果声明的方法和java.lang.Object中的某个方法一样，它可以不当做未实现的方法，不违背这个原则: 一个被它注解的接口只能有一个抽象方法, 比如: `java public interface Comparator<T> { int compare(T o1, T o2); boolean equals(Object obj); } `
- 如果一个类型被这个注解修饰，那么编译器会要求这个类型必须满足如下条件: 
  - 这个类型必须是一个interface，而不是其他的注解类型、枚举enum或者类class
  - 这个类型必须满足function interface的所有要求，如你个包含两个抽象方法的接口增加这个注解，会有编译错误。
- 编译器会自动把满足function interface要求的接口自动识别为function interface。

#### [#](#如何自定义函数接口) 如何自定义函数接口？

```java
@FunctionalInterface
public interface IMyInterface {
    void study();
}

public class TestIMyInterface {
    public static void main(String[] args) {
        IMyInterface iMyInterface = () -> System.out.println("I like study");
        iMyInterface.study();
    }
}
```

#### [#](#内置的四大函数接口及使用) 内置的四大函数接口及使用？

- **消费型接口: Consumer< T> void accept(T t)有参数，无返回值的抽象方法**；

```java
Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
greeter.accept(new Person("Luke", "Skywalker"));
```

- **供给型接口: Supplier < T> T get() 无参有返回值的抽象方法**；

以stream().collect(Collector<? super T, A, R> collector)为例:

比如:

```java
Supplier<Person> personSupplier = Person::new;
personSupplier.get();   // new Person
```

- **断定型接口: Predicate<T> boolean test(T t):有参，但是返回值类型是固定的boolean**

比如: steam().filter()中参数就是Predicate

```java
Predicate<String> predicate = (s) -> s.length() > 0;

predicate.test("foo");              // true
predicate.negate().test("foo");     // false

Predicate<Boolean> nonNull = Objects::nonNull;
Predicate<Boolean> isNull = Objects::isNull;

Predicate<String> isEmpty = String::isEmpty;
Predicate<String> isNotEmpty = isEmpty.negate();
```

- **函数型接口: Function<T,R> R apply(T t)有参有返回值的抽象方法**；

比如: steam().map() 中参数就是Function<? super T, ? extends R>；reduce()中参数BinaryOperator<T> (ps: BinaryOperator<T> extends BiFunction<T,T,T>)

```java
Function<String, Integer> toInteger = Integer::valueOf;
Function<String, String> backToString = toInteger.andThen(String::valueOf);

backToString.apply("123");     // "123"
```

#### [#](#optional要解决什么问题) Optional要解决什么问题？

在调用一个方法得到了返回值却不能直接将返回值作为参数去调用别的方法，我们首先要判断这个返回值是否为null，只有在非空的前提下才能将其作为其他方法的参数。Java 8引入了一个新的Optional类：这是一个可以为null的容器对象，如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。

#### [#](#如何使用optional来解决嵌套对象的判空问题) 如何使用Optional来解决嵌套对象的判空问题？

假设我们有一个像这样的类层次结构:

```java
class Outer {
    Nested nested;
    Nested getNested() {
        return nested;
    }
}
class Nested {
    Inner inner;
    Inner getInner() {
        return inner;
    }
}
class Inner {
    String foo;
    String getFoo() {
        return foo;
    }
}
```

解决这种结构的深层嵌套路径是有点麻烦的。我们必须编写一堆 null 检查来确保不会导致一个 NullPointerException:

```java
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```

我们可以通过利用 Java 8 的 Optional 类型来摆脱所有这些 null 检查。map 方法接收一个 Function 类型的 lambda 表达式，并自动将每个 function 的结果包装成一个 Optional 对象。这使我们能够在一行中进行多个 map 操作。Null 检查是在底层自动处理的。

```java
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo)
    .ifPresent(System.out::println);
```

还有一种实现相同作用的方式就是通过利用一个 supplier 函数来解决嵌套路径的问题:

```java
Outer obj = new Outer();
resolve(() -> obj.getNested().getInner().getFoo())
    .ifPresent(System.out::println);
```

#### [#](#什么是默认方法-为什么要有默认方法) 什么是默认方法，为什么要有默认方法？

就是接口可以有实现方法，而且不需要实现类去实现其方法。只需在方法名前面加个default关键字即可。

```java
public interface A {
    default void foo(){
       System.out.println("Calling A.foo()");
    }
}

public class Clazz implements A {
    public static void main(String[] args){
       Clazz clazz = new Clazz();
       clazz.foo();//调用A.foo()
    }
}
```

- **为什么出现默认方法**？

首先，之前的接口是个双刃剑，好处是面向抽象而不是面向具体编程，缺陷是，当需要修改接口时候，需要修改全部实现该接口的类，目前的java 8之前的集合框架没有foreach方法，通常能想到的解决办法是在JDK里给相关的接口添加新的方法及实现。然而，对于已经发布的版本，是没法在给接口添加新方法的同时不影响已有的实现。所以引进的默认方法。他们的目的是为了解决接口的修改与现有的实现不兼容的问题。

#### [#](#什么是类型注解) 什么是类型注解？

类型注解**被用来支持在Java的程序中做强类型检查。配合插件式的check framework，可以在编译的时候检测出runtime error，以提高代码质量**。这就是类型注解的作用了。

1. 在java 8之前，注解只能是在声明的地方所使用，比如类，方法，属性；
2. java 8里面，注解可以应用在任何地方，比如:

创建类实例

```java
new @Interned MyObject();
```

类型映射

```java
myString = (@NonNull String) str;
```

implements 语句中

```java
class UnmodifiableList<T> implements @Readonly List<@Readonly T> { … }
```

throw exception声明

```java
void monitorTemperature() throws @Critical TemperatureException { … }
```

需要注意的是，**类型注解只是语法而不是语义，并不会影响java的编译时间，加载时间，以及运行时间，也就是说，编译成class文件的时候并不包含类型注解**。

#### [#](#什么是重复注解) 什么是重复注解？

允许在同一申明类型(类，属性，或方法)的多次使用同一个注解

- **JDK8之前**

java 8之前也有重复使用注解的解决方案，但可读性不是很好，比如下面的代码:

```java
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseOldVersion {

    @Authorities({@Authority(role="Admin"),@Authority(role="Manager")})
    public void doSomeThing(){
    }
}
```

由另一个注解来存储重复注解，在使用时候，用存储注解Authorities来扩展重复注解。

- **Jdk8重复注解**

我们再来看看java 8里面的做法:

```java
@Repeatable(Authorities.class)
public @interface Authority {
     String role();
}

public @interface Authorities {
    Authority[] value();
}

public class RepeatAnnotationUseNewVersion {
    @Authority(role="Admin")
    @Authority(role="Manager")
    public void doSomeThing(){ }
}
```

不同的地方是，创建重复注解Authority时，加上@Repeatable,指向存储注解Authorities，在使用时候，直接可以重复使用Authority注解。从上面例子看出，java 8里面做法更适合常规的思维，可读性强一点

### [#](#_6-2-java-9-特性) 6.2 Java 9+ 特性

#### [#](#java-9后续版本发布是按照什么样的发布策略呢) Java 9后续版本发布是按照什么样的发布策略呢？

Java现在发布的版本很快，每年两个，但是真正会被大规模使用的是三年一个的TLS版本。@pdai

- 每3年发布一个TLS，长期维护版本。意味着Java 8 ，Java 11， Java 17 才可能被大规模使用。
- 每年发布两个正式版本，分别是3月份和9月份。

#### [#](#java-9后续新版本中你知道哪些) Java 9后续新版本中你知道哪些？

能够举几个即可：

- **Java10 - 并行全垃圾回收器 G1**

大家如果接触过 Java 性能调优工作，应该会知道，调优的最终目标是通过参数设置来达到快速、低延时的内存垃圾回收以提高应用吞吐量，尽可能的避免因内存回收不及时而触发的完整 GC（Full GC 会带来应用出现卡顿）。

G1 垃圾回收器是 Java 9 中 Hotspot 的默认垃圾回收器，是以一种低延时的垃圾回收器来设计的，旨在避免进行 Full GC，但是当并发收集无法快速回收内存时，会触发垃圾回收器回退进行 Full GC。之前 Java 版本中的 G1 垃圾回收器执行 GC 时采用的是基于单线程标记扫描压缩算法（mark-sweep-compact）。为了最大限度地减少 Full GC 造成的应用停顿的影响，Java 10 中将为 G1 引入多线程并行 GC，同时会使用与年轻代回收和混合回收相同的并行工作线程数量，从而减少了 Full GC 的发生，以带来更好的性能提升、更大的吞吐量。

Java 10 中将采用并行化 mark-sweep-compact 算法，并使用与年轻代回收和混合回收相同数量的线程。具体并行 GC 线程数量可以通过： `-XX：ParallelGCThreads` 参数来调节，但这也会影响用于年轻代和混合收集的工作线程数。

- **Java11 - ZGC：可伸缩低延迟垃圾收集器**

ZGC 即 Z Garbage Collector（垃圾收集器或垃圾回收器），这应该是 Java 11 中最为瞩目的特性，没有之一。ZGC 是一个可伸缩的、低延迟的垃圾收集器，主要为了满足如下目标进行设计：

1. GC 停顿时间不超过 10ms
2. 即能处理几百 MB 的小堆，也能处理几个 TB 的大堆
3. 应用吞吐能力不会下降超过 15%（与 G1 回收算法相比）
4. 方便在此基础上引入新的 GC 特性和利用 colord
5. 针以及 Load barriers 优化奠定基础
6. 当前只支持 Linux/x64 位平台

停顿时间在 10ms 以下，10ms 其实是一个很保守的数据，即便是 10ms 这个数据，也是 GC 调优几乎达不到的极值。根据 SPECjbb 2015 的基准测试，128G 的大堆下最大停顿时间才 1.68ms，远低于 10ms，和 G1 算法相比，改进非常明显。

![img](/images/java/java-11-1.png)

- **Java 14 - Switch 表达式**（正式版）

switch 表达式在之前的 Java 12 和 Java 13 中都是处于预览阶段，而在这次更新的 Java 14 中，终于成为稳定版本，能够正式可用。

switch 表达式带来的不仅仅是编码上的简洁、流畅，也精简了 switch 语句的使用方式，同时也兼容之前的 switch 语句的使用；之前使用 switch 语句时，在每个分支结束之前，往往都需要加上 break 关键字进行分支跳出，以防 switch 语句一直往后执行到整个 switch 语句结束，由此造成一些意想不到的问题。switch 语句一般使用冒号 ：来作为语句分支代码的开始，而 switch 表达式则提供了新的分支切换方式，即 -> 符号右则表达式方法体在执行完分支方法之后，自动结束 switch 分支，同时 -> 右则方法块中可以是表达式、代码块或者是手动抛出的异常。以往的 switch 语句写法如下：

```java
int dayOfWeek;
switch (day) {
    case MONDAY:
    case FRIDAY:
    case SUNDAY:
        dayOfWeek = 6;
        break;
    case TUESDAY:
        dayOfWeek = 7;
        break;
    case THURSDAY:
    case SATURDAY:
        dayOfWeek = 8;
        break;
    case WEDNESDAY:
        dayOfWeek = 9;
        break;
    default:
        dayOfWeek = 0;
        break;
}
```

而现在 Java 14 可以使用 switch 表达式正式版之后，上面语句可以转换为下列写法：

```java
int dayOfWeek = switch (day) {
    case MONDAY, FRIDAY, SUNDAY -> 6;
    case TUESDAY                -> 7;
    case THURSDAY, SATURDAY     -> 8;
case WEDNESDAY              -> 9;
    default              -> 0;

};
```

很明显，switch 表达式将之前 switch 语句从编码方式上简化了不少，但是还是需要注意下面几点：

1. 需要保持与之前 switch 语句同样的 case 分支情况。
2. 之前需要用变量来接收返回值，而现在直接使用 yield 关键字来返回 case 分支需要返回的结果。
3. 现在的 switch 表达式中不再需要显式地使用 return、break 或者 continue 来跳出当前分支。
4. 现在不需要像之前一样，在每个分支结束之前加上 break 关键字来结束当前分支，如果不加，则会默认往后执行，直到遇到 break 关键字或者整个 switch 语句结束，在 Java 14 表达式中，表达式默认执行完之后自动跳出，不会继续往后执行。
5. 对于多个相同的 case 方法块，可以将 case 条件并列，而不需要像之前一样，通过每个 case 后面故意不加 break 关键字来使用相同方法块。

使用 switch 表达式来替换之前的 switch 语句，确实精简了不少代码，提高了编码效率，同时也可以规避一些可能由于不太经意而出现的意想不到的情况，可见 Java 在提高使用者编码效率、编码体验和简化使用方面一直在不停的努力中，同时也期待未来有更多的类似 lambda、switch 表达式这样的新特性出来。

- **Java 14 - Records**

在 Java 14 中引入了 Record 类型，其效果有些类似 Lombok 的 @Data 注解、Kotlin 中的 data class，但是又不尽完全相同，它们的共同点都是类的部分或者全部可以直接在类头中定义、描述，并且这个类只用于存储数据而已。对于 Record 类型，具体可以用下面代码来说明：

```java
public record Person(String name, int age) {
    public static String address;

    public String getName() {
        return name;
    }
}
```

对上述代码进行编译，然后反编译之后可以看到如下结果：

```java
public final class Person extends java.lang.Record {
    private final java.lang.String name;
    private final java.lang.String age;

    public Person(java.lang.String name, java.lang.String age) { /* compiled code */ }

    public java.lang.String getName() { /* compiled code */ }

    public java.lang.String toString() { /* compiled code */ }

    public final int hashCode() { /* compiled code */ }

    public final boolean equals(java.lang.Object o) { /* compiled code */ }

    public java.lang.String name() { /* compiled code */ }

    public java.lang.String age() { /* compiled code */ }
}
```

## [#](#_7-数据结构和算法) 7 数据结构和算法

> 数据结构和算法

### [#](#_7-1-数据结构基础) 7.1 数据结构基础

#### [#](#如何理解基础的数据结构) 如何理解基础的数据结构？

避免孤立的学习知识点，要关联学习。比如实际应用当中，我们经常使用的是**查找，排序以及增删改**，这在我们的各种管理系统、数据库系统、操作系统等当中，十分常用，我们通过这个线索将知识点串联起来：

- **数组**的下标寻址十分迅速，但计算机的内存是有限的，故数组的长度也是有限的，实际应用当中的数据往往十分庞大；而且无序数组的查找最坏情况需要遍历整个数组；后来人们提出了二分查找，二分查找要求数组的构造一定有序，二分法查找解决了普通数组查找复杂度过高的问题。任何一种数组无法解决的问题就是插入、删除操作比较复杂，因此，在一个增删查改比较频繁的数据结构中，数组不会被优先考虑
- **普通链表**由于它的结构特点被证明根本不适合进行查找
- **哈希表**是数组和链表的折中，同时它的设计依赖散列函数的设计，数组不能无限长、链表也不适合查找，所以也不适合大规模的查找
- **二叉查找树**因为可能退化成链表，同样不适合进行查找
- **AVL树**是为了解决二叉查找树可能退化成链表问题。**AVL树是严格的平衡二叉树**，平衡条件必须满足（所有节点的左右子树高度差的绝对值不超过1）。不管我们是执行插入还是删除操作，只要不满足上面的条件，就要通过旋转来保持平衡，而旋转是非常耗时的，由此我们可以知道AVL树适合用于插入与删除次数比较少，但查找多的情况。
- **红黑树**是二叉查找树和AVL树的折中。它是一种弱平衡二叉树，但在每个节点增加一个存储位表示节点的颜色，可以是红或黑（非红即黑）。通过对任何一条从根到叶子的路径上各个节点着色的方式的限制，红黑树确保没有一条路径会比其它路径长出两倍，因此，**红黑树是一种弱平衡二叉树**（由于是弱平衡，可以看到，在相同的节点情况下，AVL树的高度低于红黑树），相对于要求严格的AVL树来说，它的旋转次数少，所以对于搜索，插入，删除操作较多的情况下，我们就用红黑树。
- **多路查找树** 是大规模数据存储中，实现索引查询这样一个实际背景下，树节点存储的元素数量是有限的(如果元素数量非常多的话，查找就退化成节点内部的线性查找了)，这样导致二叉查找树结构由于树的深度过大而造成磁盘I/O读写过于频繁，进而导致查询效率低下。
- **B树**与自平衡二叉查找树不同，B树适用于读写相对大的数据块的存储系统，例如磁盘。它的应用是文件系统及部分非关系型数据库索引。
- **B+树**在B树基础上，为叶子结点增加链表指针(B树+叶子有序链表)，所有关键字都在叶子结点 中出现，非叶子结点作为叶子结点的索引；B+树总是到叶子结点才命中。通常用于关系型数据库(如Mysql)和操作系统的文件系统中。
- **B\*树**是B+树的变体，在B+树的非根和非叶子结点再增加指向兄弟的指针, 在B+树基础上，为非叶子结点也增加链表指针，将结点的最低利用率从1/2提高到2/3。
- **R树**是用来做空间数据存储的树状数据结构。例如给地理位置，矩形和多边形这类多维数据建立索引。
- **Trie树**是自然语言处理中最常用的数据结构，很多字符串处理任务都会用到。Trie树本身是一种有限状态自动机，还有很多变体。什么模式匹配、正则表达式，都与这有关。

### [#](#_7-2-算法思想) 7.2 算法思想

#### [#](#有哪些常见的算法思想) 有哪些常见的算法思想？

- **分治算法** ：分治算法的基本思想是将一个规模为N的问题分解为K个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解
- **动态规划算法**： 通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可行解。每一个解都对应于一个值，我们希望找到具有最优值的解。动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，先求解子问题，然后从这些子问题的解得到原问题的解。和分治算法最大的差别：适用于动态规划算法求解的问题经过分解后得到的子问题往往不是相互独立的，而是下一个子阶段的求解是建立在上一个子阶段的解的基础上的。
- **贪心算法**: 保证每次操作都是局部最优的，并且最后得到的结果是全局最优的
- **二分法**： 比如重要的二分法，比如二分查找；二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。
- **搜索算法**： 主要包含BFS，DFS
- **Backtracking(回溯)**： 属于 DFS, 回溯算法实际上一个类似枚举的搜索尝试过程，主要是在搜索尝试过程中寻找问题的解，当发现已不满足求解条件时，就“回溯”返回，尝试别的路径。回溯法是一种选优搜索法，按选优条件向前搜索，以达到目标。但当探索到某一步时，发现原先选择并不优或达不到目标，就退回一步重新选择，这种走不通就退回再走的技术为回溯法

### [#](#_7-3-常见排序算法) 7.3 常见排序算法

#### [#](#有哪些常见的排序算法) 有哪些常见的排序算法？

在综合复杂度及稳定性情况下，通常**希尔, 快排和 归并**需要重点掌握

![img](/images/alg/alg-sort-overview-1.png)

- 冒泡排序

  (Bubble Sort) 

  - 它是一种较简单的排序算法。它会遍历若干次要排序的数列，每次遍历时，它都会从前往后依次的比较相邻两个数的大小；如果前者比后者大，则交换它们的位置。这样，一次遍历之后，最大的元素就在数列的末尾！ 采用相同的方法再次遍历时，第二大的元素就被排列在最大元素之前。重复此操作，直到整个数列都有序为止

- 快速排序

  (Quick Sort) 

  - 它的基本思想是: 选择一个基准数，通过一趟排序将要排序的数据分割成独立的两部分；其中一部分的所有数据都比另外一部分的所有数据都要小。然后，再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

- 插入排序

  (Insertion Sort) 

  - 直接插入排序(Straight Insertion Sort)的基本思想是: 把n个待排序的元素看成为一个有序表和一个无序表。开始时有序表中只包含1个元素，无序表中包含有n-1个元素，排序过程中每次从无序表中取出第一个元素，将它插入到有序表中的适当位置，使之成为新的有序表，重复n-1次可完成排序过程。

- Shell排序

  (Shell Sort) 

  - 希尔排序实质上是一种分组插入方法。它的基本思想是: 对于n个待排序的数列，取一个小于n的整数gap(gap被称为步长)将待排序元素分成若干个组子序列，所有距离为gap的倍数的记录放在同一个组中；然后，对各组内的元素进行直接插入排序。 这一趟排序完成之后，每一个组的元素都是有序的。然后减小gap的值，并重复执行上述的分组和排序。重复这样的操作，当gap=1时，整个数列就是有序的。

- 选择排序

  (Selection sort) 

  - 它的基本思想是: 首先在未排序的数列中找到最小(or最大)元素，然后将其存放到数列的起始位置；接着，再从剩余未排序的元素中继续寻找最小(or最大)元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

- 堆排序

  (Heap Sort) 

  - 堆排序是指利用堆这种数据结构所设计的一种排序算法。堆是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

- 归并排序

  (Merge Sort) 

  - 将两个的有序数列合并成一个有序数列，我们称之为"归并"。归并排序(Merge Sort)就是利用归并思想对数列进行排序。

- 桶排序

  (Bucket Sort) 

  - 桶排序(Bucket Sort)的原理很简单，将数组分到有限数量的桶子里。每个桶子再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）

- 基数排序

  (Radix Sort) 

  - 它的基本思想是: 将整数按位数切割成不同的数字，然后按每个位数分别比较。具体做法是: 将所有待比较数值统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后, 数列就变成一个有序序列

### [#](#_7-4-大数据处理算法) 7.4 大数据处理算法

#### [#](#何谓海量数据处理-解决的思路) 何谓海量数据处理? 解决的思路？

所谓海量数据处理，无非就是基于海量数据上的存储、处理、操作。何谓海量，就是数据量太大，所以导致要么是无法在较短时间内迅速解决，要么是数据太大，导致无法一次性装入内存。

那解决办法呢?

- **针对时间**: 我们可以采用巧妙的算法搭配合适的数据结构，如Bloom filter/Hash/bit-map/堆/数据库或倒排索引/trie树；
- **针对空间**: 无非就一个办法: 大而化小，分而治之(hash映射);
- **集群|分布式**: 通俗点来讲，单机就是处理装载数据的机器有限(只要考虑cpu，内存，硬盘的数据交互); 而集群适合分布式处理，并行计算(更多考虑节点和节点间的数据交互)。

#### [#](#大数据处理之分治思想) 大数据处理之分治思想？

分而治之/hash映射 + hash统计 + 堆/快速/归并排序，说白了，就是先映射，而后统计，最后排序:

- **分而治之/hash映射**: 针对数据太大，内存受限，只能是: 把大文件化成(取模映射)小文件，即16字方针: 大而化小，各个击破，缩小规模，逐个解决
- **hash_map统计**: 当大文件转化了小文件，那么我们便可以采用常规的hash_map(ip，value)来进行频率统计。
- **堆/快速排序**: 统计完了之后，便进行排序(可采取堆排序)，得到次数最多的IP。

#### [#](#海量日志数据-提取出某日访问百度次数最多的那个ip) 海量日志数据，提取出某日访问百度次数最多的那个IP?

分析: “首先是这一天，并且是访问百度的日志中的IP取出来，逐个写入到一个大文件中。注意到IP是32位的，最多有个2^32个IP。同样可以采用映射的方法，比如%1000，把整个大文件映射为1000个小文件，再找出每个小文中出现频率最大的IP(可以采用hash_map对那1000个文件中的所有IP进行频率统计，然后依次找出各个文件中频率最大的那个IP)及相应的频率。然后再在这1000个最大的IP中，找出那个频率最大的IP，即为所求。”

关于本题，还有几个问题，如下:

- Hash取模是一种等价映射，不会存在同一个元素分散到不同小文件中的情况，即这里采用的是mod1000算法，那么相同的IP在hash取模后，只可能落在同一个文件中，不可能被分散的。因为如果两个IP相等，那么经过Hash(IP)之后的哈希值是相同的，将此哈希值取模(如模1000)，必定仍然相等。
- 那到底什么是hash映射呢? 简单来说，就是为了便于计算机在有限的内存中处理big数据，从而通过一种映射散列的方式让数据均匀分布在对应的内存位置(如大数据通过取余的方式映射成小树存放在内存中，或大文件映射成多个小文件)，而这个映射散列方式便是我们通常所说的hash函数，设计的好的hash函数能让数据均匀分布而减少冲突。尽管数据映射到了另外一些不同的位置，但数据还是原来的数据，只是代替和表示这些原始数据的形式发生了变化而已。

#### [#](#寻找热门查询-300万个查询字符串中统计最热门的10个查询) 寻找热门查询，300万个查询字符串中统计最热门的10个查询?

原题: 搜索引擎会通过日志文件把用户每次检索使用的所有检索串都记录下来，每个查询串的长度为1-255字节。假设目前有一千万个记录(这些查询串的重复度比较高，虽然总数是1千万，但如果除去重复后，不超过3百万个。一个查询串的重复度越高，说明查询它的用户越多，也就是越热门)，请你统计最热门的10个查询串，要求使用的内存不能超过1G。

解答: 由上面第1题，我们知道，数据大则划为小的，如如一亿个Ip求Top 10，可先%1000将ip分到1000个小文件中去，并保证一种ip只出现在一个文件中，再对每个小文件中的ip进行hashmap计数统计并按数量排序，最后归并或者最小堆依次处理每个小文件的top10以得到最后的结。

但如果数据规模比较小，能一次性装入内存呢?比如这第2题，虽然有一千万个Query，但是由于重复度比较高，因此事实上只有300万的Query，每个Query255Byte，因此我们可以考虑把他们都放进内存中去(300万个字符串假设没有重复，都是最大长度，那么最多占用内存3M*1K/4=0.75G。所以可以将所有字符串都存放在内存中进行处理)，而现在只是需要一个合适的数据结构，在这里，HashTable绝对是我们优先的选择。

所以我们放弃分而治之/hash映射的步骤，直接上hash统计，然后排序。So，针对此类典型的TOP K问题，采取的对策往往是: hashmap + 堆。如下所示:

- **hash_map统计**: 先对这批海量数据预处理。具体方法是: 维护一个Key为Query字串，Value为该Query出现次数的HashTable，即hash_map(Query，Value)，每次读取一个Query，如果该字串不在Table中，那么加入该字串，并且将Value值设为1；如果该字串在Table中，那么将该字串的计数加一即可。最终我们在O(N)的时间复杂度内用Hash表完成了统计； 堆排序: 第二步、借助堆这个数据结构，找出Top K，时间复杂度为N‘logK。即借助堆结构，我们可以在log量级的时间内查找和调整/移动。因此，维护一个K(该题目中是10)大小的小根堆，然后遍历300万的Query，分别和根元素进行对比。所以，我们最终的时间复杂度是: O(N) + N' * O(logK)，(N为1000万，N’为300万)。

别忘了这篇文章中所述的堆排序思路: “维护k个元素的最小堆，即用容量为k的最小堆存储最先遍历到的k个数，并假设它们即是最大的k个数，建堆费时O(k)，并调整堆(费时O(logk))后，有k1>k2>...kmin(kmin设为小顶堆中最小元素)。继续遍历数列，每次遍历一个元素x，与堆顶元素比较，若x>kmin，则更新堆(x入堆，用时logk)，否则不更新堆。这样下来，总费时O(k*logk+(n-k)*logk)=O(n*logk)。此方法得益于在堆中，查找等各项操作时间复杂度均为logk。”--第三章续、Top K算法问题的实现。

当然，你也可以采用trie树，关键字域存该查询串出现的次数，没有出现为0。最后用10个元素的最小推来对出现频率进行排序。

#### [#](#有一个1g大小的一个文件-里面每一行是一个词-词的大小不超过16字节-内存限制大小是1m。返回频数最高的100个词) 有一个1G大小的一个文件，里面每一行是一个词，词的大小不超过16字节，内存限制大小是1M。返回频数最高的100个词?

- **分而治之/hash映射**: 顺序读文件中，对于每个词x，取hash(x)%5000，然后按照该值存到5000个小文件(记为x0,x1,...x4999)中。这样每个文件大概是200k左右。如果其中的有的文件超过了1M大小，还可以按照类似的方法继续往下分，直到分解得到的小文件的大小都不超过1M。
- **hash_map统计**: 对每个小文件，采用trie树/hash_map等统计每个文件中出现的词以及相应的频率。
- **堆/归并排序**: 取出出现频率最大的100个词(可以用含100个结点的最小堆)后，再把100个词及相应的频率存入文件，这样又得到了5000个文件。最后就是把这5000个文件进行归并(类似于归并排序)的过程了。

#### [#](#海量数据分布在100台电脑中-想个办法高效统计出这批数据的top10) 海量数据分布在100台电脑中，想个办法高效统计出这批数据的TOP10?

如果每个数据元素只出现一次，而且只出现在某一台机器中，那么可以采取以下步骤统计出现次数TOP10的数据元素:

- **堆排序**: 在每台电脑上求出TOP10，可以采用包含10个元素的堆完成(TOP10小，用最大堆，TOP10大，用最小堆，比如求TOP10大，我们首先取前10个元素调整成最小堆，如果发现，然后扫描后面的数据，并与堆顶元素比较，如果比堆顶元素大，那么用该元素替换堆顶，然后再调整为最小堆。最后堆中的元素就是TOP10大)。 求出每台电脑上的TOP10后，然后把这100台电脑上的TOP10组合起来，共1000个数据，再利用上面类似的方法求出TOP10就可以了。

但如果同一个元素重复出现在不同的电脑中呢，如下例子所述, 这个时候，你可以有两种方法:

- 遍历一遍所有数据，重新hash取摸，如此使得同一个元素只出现在单独的一台电脑中，然后采用上面所说的方法，统计每台电脑中各个元素的出现次数找出TOP10，继而组合100台电脑上的TOP10，找出最终的TOP10。
- 或者，暴力求解: 直接统计统计每台电脑中各个元素的出现次数，然后把同一个元素在不同机器中的出现次数相加，最终从所有数据中找出TOP10。

#### [#](#有10个文件-每个文件1g-每个文件的每一行存放的都是用户的query-每个文件的query都可能重复。要求你按照query的频度排序) 有10个文件，每个文件1G，每个文件的每一行存放的都是用户的query，每个文件的query都可能重复。要求你按照query的频度排序?

方案1:

- **hash映射**: 顺序读取10个文件，按照hash(query)%10的结果将query写入到另外10个文件(记为a0,a1,..a9)中。这样新生成的文件每个的大小大约也1G(假设hash函数是随机的)。
- **hash_map统计**: 找一台内存在2G左右的机器，依次对用hash_map(query, query_count)来统计每个query出现的次数。注: hash_map(query,query_count)是用来统计每个query的出现次数，不是存储他们的值，出现一次，则count+1。
- **堆/快速/归并排序**: 利用快速/堆/归并排序按照出现次数进行排序，将排序好的query和对应的query_cout输出到文件中，这样得到了10个排好序的文件(记为)。最后，对这10个文件进行归并排序(内排序与外排序相结合)。

方案2: 一般query的总量是有限的，只是重复的次数比较多而已，可能对于所有的query，一次性就可以加入到内存了。这样，我们就可以采用trie树/hash_map等直接来统计每个query出现的次数，然后按出现次数做快速/堆/归并排序就可以了。

方案3: 与方案1类似，但在做完hash，分成多个文件后，可以交给多个文件来处理，采用分布式的架构来处理(比如MapReduce)，最后再进行合并。

#### [#](#给定a、b两个文件-各存放50亿个url-每个url各占64字节-内存限制是4g-让你找出a、b文件共同的url) 给定a、b两个文件，各存放50亿个url，每个url各占64字节，内存限制是4G，让你找出a、b文件共同的url?

可以估计每个文件安的大小为5G×64=320G，远远大于内存限制的4G。所以不可能将其完全加载到内存中处理。考虑采取分而治之的方法。

- **分而治之/hash映射**: 遍历文件a，对每个url求取，然后根据所取得的值将url分别存储到1000个小文件(记为，这里漏写个了a1)中。这样每个小文件的大约为300M。遍历文件b，采取和a相同的方式将url分别存储到1000小文件中(记为)。这样处理后，所有可能相同的url都在对应的小文件()中，不对应的小文件不可能有相同的url。然后我们只要求出1000对小文件中相同的url即可。
- **hash_set统计**: 求每对小文件中相同的url时，可以把其中一个小文件的url存储到hash_set中。然后遍历另一个小文件的每个url，看其是否在刚才构建的hash_set中，如果是，那么就是共同的url，存到文件里面就可以了。

如果允许有一定的错误率，可以使用Bloom filter，4G内存大概可以表示340亿bit。将其中一个文件中的url使用Bloom filter映射为这340亿bit，然后挨个读取另外一个文件的url，检查是否与Bloom filter，如果是，那么该url应该是共同的url(注意会有一定的错误率)。”

#### [#](#怎么在海量数据中找出重复次数最多的一个) 怎么在海量数据中找出重复次数最多的一个?

方案: 先做hash，然后求模映射为小文件，求出每个小文件中重复次数最多的一个，并记录重复次数。然后找出上一步求出的数据中重复次数最多的一个就是所求(具体参考前面的题)。

#### [#](#上千万或上亿数据-有重复-统计其中出现次数最多的前n个数据) 上千万或上亿数据(有重复)，统计其中出现次数最多的前N个数据?

方案: 上千万或上亿的数据，现在的机器的内存应该能存下。所以考虑采用hash_map/搜索二叉树/红黑树等来进行统计次数。然后利用堆取出前N个出现次数最多的数据。

#### [#](#一个文本文件-大约有一万行-每行一个词-要求统计出其中最频繁出现的前10个词-请给出思想-给出时间复杂度分析) 一个文本文件，大约有一万行，每行一个词，要求统计出其中最频繁出现的前10个词，请给出思想，给出时间复杂度分析?

方案1: 如果文件比较大，无法一次性读入内存，可以采用hash取模的方法，将大文件分解为多个小文件，对于单个小文件利用hash_map统计出每个小文件中10个最常出现的词，然后再进行归并处理，找出最终的10个最常出现的词。

方案2: 通过hash取模将大文件分解为多个小文件后，除了可以用hash_map统计出每个小文件中10个最常出现的词，也可以用trie树统计每个词出现的次数，时间复杂度是O(n*le)(le表示单词的平准长度)，最终同样找出出现最频繁的前10个词(可用堆来实现)，时间复杂度是O(n*lg10)。

#### [#](#一个文本文件-找出前10个经常出现的词-但这次文件比较长-说是上亿行或十亿行-总之无法一次读入内存-问最优解) 一个文本文件，找出前10个经常出现的词，但这次文件比较长，说是上亿行或十亿行，总之无法一次读入内存，问最优解?

方案1: 首先根据用hash并求模，将文件分解为多个小文件，对于单个文件利用上题的方法求出每个文件件中10个最常出现的词。然后再进行归并处理，找出最终的10个最常出现的词。

#### [#](#_100w个数中找出最大的100个数) 100w个数中找出最大的100个数?

方案1: 采用局部淘汰法。选取前100个元素，并排序，记为序列L。然后一次扫描剩余的元素x，与排好序的100个元素中最小的元素比，如果比这个最小的要大，那么把这个最小的元素删除，并把x利用插入排序的思想，插入到序列L中。依次循环，知道扫描了所有的元素。复杂度为O(100w*100)。

方案2: 采用快速排序的思想，每次分割之后只考虑比轴大的一部分，知道比轴大的一部分在比100多的时候，采用传统排序算法排序，取前100个。复杂度为O(100w*100)。

方案3: 在前面的题中，我们已经提到了，用一个含100个元素的最小堆完成。复杂度为O(100w*lg100)。

#### [#](#_5亿个int找它们的中位数) 5亿个int找它们的中位数?

- **思路一**

这个例子比上面那个更明显。首先我们将int划分为2^16个区域，然后读取数据统计落到各个区域里的数的个数，之后我们根据统计结果就可以判断中位数落到那个区域，同时知道这个区域中的第几大数刚好是中位数。然后第二次扫描我们只统计落在这个区域中的那些数就可以了。

实际上，如果不是int是int64，我们可以经过3次这样的划分即可降低到可以接受的程度。即可以先将int64分成224个区域，然后确定区域的第几大数，在将该区域分成220个子区域，然后确定是子区域的第几大数，然后子区域里的数的个数只有2^20，就可以直接利用direct addr table进行统计了。

- **思路二**

同样需要做两遍统计，如果数据存在硬盘上，就需要读取2次。

方法同基数排序有些像，开一个大小为65536的Int数组，第一遍读取，统计Int32的高16位的情况，也就是0-65535，都算作0,65536 - 131071都算作1。就相当于用该数除以65536。Int32 除以 65536的结果不会超过65536种情况，因此开一个长度为65536的数组计数就可以。每读取一个数，数组中对应的计数+1，考虑有负数的情况，需要将结果加32768后，记录在相应的数组内。

第一遍统计之后，遍历数组，逐个累加统计，看中位数处于哪个区间，比如处于区间k，那么0- k-1的区间里数字的数量sum应该`<n/2`(2.5亿)。而k+1 - 65535的计数和也`<n/2`，第二遍统计同上面的方法类似，但这次只统计处于区间k的情况，也就是说(x / 65536) + 32768 = k。统计只统计低16位的情况。并且利用刚才统计的sum，比如sum = 2.49亿，那么现在就是要在低16位里面找100万个数(2.5亿-2.49亿)。这次计数之后，再统计一下，看中位数所处的区间，最后将高位和低位组合一下就是结果了。

#### [#](#在2-5亿个整数中找出不重复的整数-注-内存不足以容纳这2-5亿个整数。) 在2.5亿个整数中找出不重复的整数，注，内存不足以容纳这2.5亿个整数。

- 方案1: 采用2-Bitmap(每个数分配2bit，00表示不存在，01表示出现一次，10表示多次，11无意义)进行，共需内存2^32 * 2 bit=1 GB内存，还可以接受。然后扫描这2.5亿个整数，查看Bitmap中相对应位，如果是00变01，01变10，10保持不变。所描完事后，查看bitmap，把对应位是01的整数输出即可。
- 方案2: 也可采用分治，划分小文件的方法。然后在小文件中找出不重复的整数，并排序。然后再进行归并，注意去除重复的元素。

#### [#](#给40亿个不重复的unsigned-int的整数-没排过序的-然后再给一个数-如何快速判断这个数是否在那40亿个数当中) 给40亿个不重复的unsigned int的整数，没排过序的，然后再给一个数，如何快速判断这个数是否在那40亿个数当中?

用位图/Bitmap的方法，申请512M的内存，一个bit位代表一个unsigned int值。读入40亿个数，设置相应的bit位，读入要查询的数，查看相应bit位是否为1，为1表示存在，为0表示不存在。

### [#](#_7-5-加密算法) 7.5 加密算法

#### [#](#什么是摘要算法-有哪些) 什么是摘要算法？有哪些?

消息摘要算法的主要特征是加密过程不需要密钥，并且经过加密的数据无法被解密，目前可以解密逆向的只有CRC32算法，只有输入相同的明文数据经过相同的消息摘要算法才能得到相同的密文。消息摘要算法不存在密钥的管理与分发问题，适合于分布式网络上使用。消息摘要算法主要应用在“数字签名”领域，作为对明文的摘要算法。

- **何谓数字签名？**

数字签名主要用到了非对称密钥加密技术与数字摘要技术。数字签名技术是将摘要信息用发送者的私钥加密，与原文一起传送给接收者。接收者只有用发送者的公钥才能解密被加密的摘要信息，然后用HASH函数对收到的原文产生一个摘要信息，与解密的摘要信息对比。如果相同，则说明收到的信息是完整的，在传输过程中没有被修改，否则说明信息被修改过.

因此数字签名能够验证信息的完整性。

数字签名是个加密的过程，数字签名验证是个解密的过程。

- **有哪些摘要算法**？

著名的摘要算法有RSA公司的MD5算法和SHA-1算法及其大量的变体

#### [#](#什么是加密算法-有哪些) 什么是加密算法？有哪些？

数据加密的基本过程就是对原来为明文的文件或数据按某种算法进行处理，使其成为不可读的一段代码为“密文”，使其只能在输入相应的密钥之后才能显示出原容，通过这样的途径来达到保护数据不被非法人窃取、阅读的目的。 该过程的逆过程为解密，即将该编码信息转化为其原来数据的过程。

**加密算法分类**

密钥加密技术的密码体制分为**对称密钥**体制和**非对称密**钥体制两种。相应地，对数据加密的技术分为两类，即对称加密(私人密钥加密)和非对称加密(公开密钥加密)。

对称加密以数据加密标准(**DES**，Data Encryption Standard)算法为典型代表，非对称加密通常以**RSA**(Rivest Shamir Adleman)算法为代表。

对称加密的加密密钥和解密密钥相同。非对称加密的加密密钥和解密密钥不同，加密密钥可以公开而解密密钥需要保密

#### [#](#什么是国密算法-有哪些) 什么是国密算法？有哪些？

- SM1 **为对称加密**。其加密强度与AES相当。该算法不公开，调用该算法时，需要通过**加密芯片的接口进行调用**。
- SM2 **非对称加密**，基于ECC。该算法已公开。由于该算法基于ECC，故其签名速度与秘钥生成速度都快于RSA。ECC 256位（SM2采用的就是ECC 256位的一种）安全强度比RSA 2048位高，但运算速度快于RSA。
- SM3 **消息摘要**。可以用MD5作为对比理解。该算法已公开。校验结果为256位。
- SM4 无线局域网标准的**分组数据算法**。对称加密，密钥长度和分组长度均为128位。
- SM7 是一种分组密码算法，分组长度为128比特，密钥长度为128比特。SM7适用于非接触式IC卡，应用包括身份识别类应用(门禁卡、工作证、参赛证)，票务类应用(大型赛事门票、展会门票)，支付与通卡类应用（积分消费卡、校园一卡通、企业一卡通等）。
- SM9 不需要申请数字证书，适用于互联网应用的各种新兴应用的安全保障。如基于云技术的密码服务、电子邮件安全、智能终端保护、物联网安全、云存储安全等等。这些安全应用可采用手机号码或邮件地址作为公钥，实现数据加密、身份认证、通话加密、通道加密等安全应用，并具有使用方便，易于部署的特点，从而开启了普及密码算法的大门。

## [#](#_8-数据库) 8 数据库

> 数据库相关，包括MySQL，Redis，MongoDB， ElasticSearch等。

### [#](#_8-1-原理和sql) 8.1 原理和SQL

#### [#](#什么是事务-事务基本特性acid) 什么是事务？事务基本特性ACID？

事务指的是满足 ACID 特性的一组操作，可以通过 Commit 提交一个事务，也可以使用 Rollback 进行回滚。

![image](/images/pics/185b9c49-4c13-4241-a848-fbff85c03a64.png)

- 事务基本特性ACID

  ?： 

  - **A原子性(atomicity)** 指的是一个事务中的操作要么全部成功，要么全部失败。
  - **C一致性(consistency)** 指的是数据库总是从一个一致性的状态转换到另外一个一致性的状态。比如A转账给B100块钱，假设中间sql执行过程中系统崩溃A也不会损失100块，因为事务没有提交，修改也就不会保存到数据库。
  - **I隔离性(isolation)** 指的是一个事务的修改在最终提交前，对其他事务是不可见的。
  - **D持久性(durability)** 指的是一旦事务提交，所做的修改就会永久保存到数据库中。

#### [#](#数据库中并发一致性问题) 数据库中并发一致性问题？

在并发环境下，事务的隔离性很难保证，因此会出现很多并发一致性问题。

- **丢失修改**

T1 和 T2 两个事务都对一个数据进行修改，T1 先修改，T2 随后修改，T2 的修改覆盖了 T1 的修改。

![image](/images/pics/88ff46b3-028a-4dbb-a572-1f062b8b96d3.png)

- **读脏数据**

T1 修改一个数据，T2 随后读取这个数据。如果 T1 撤销了这次修改，那么 T2 读取的数据是脏数据。

![image](/images/pics/dd782132-d830-4c55-9884-cfac0a541b8e.png)

- **不可重复读**

T2 读取一个数据，T1 对该数据做了修改。如果 T2 再次读取这个数据，此时读取的结果和第一次读取的结果不同。

![image](/images/pics/c8d18ca9-0b09-441a-9a0c-fb063630d708.png)

- **幻影读**

T1 读取某个范围的数据，T2 在这个范围内插入新的数据，T1 再次读取这个范围的数据，此时读取的结果和和第一次读取的结果不同。

![image](/images/pics/72fe492e-f1cb-4cfc-92f8-412fb3ae6fec.png)

#### [#](#事务的隔离等级) 事务的隔离等级？

- **未提交读(READ UNCOMMITTED)** 事务中的修改，即使没有提交，对其它事务也是可见的。
- **提交读(READ COMMITTED)** 一个事务只能读取已经提交的事务所做的修改。换句话说，一个事务所做的修改在提交之前对其它事务是不可见的。
- **可重复读(REPEATABLE READ)** 保证在同一个事务中多次读取同样数据的结果是一样的。
- **可串行化(SERIALIZABLE)** 强制事务串行执行。

| 隔离级别 | 脏读 | 不可重复读 | 幻影读 |
| :------: | :--: | :--------: | :----: |
| 未提交读 |  √   |     √      |   √    |
|  提交读  |  ×   |     √      |   √    |
| 可重复读 |  ×   |     ×      |   √    |
| 可串行化 |  ×   |     ×      |   ×    |

#### [#](#acid靠什么保证的呢) ACID靠什么保证的呢？

- **A原子性(atomicity)** 由undo log日志保证，它记录了需要回滚的日志信息，事务回滚时撤销已经执行成功的sql
- **C一致性(consistency)** 一般由代码层面来保证
- **I隔离性(isolation)** 由MVCC来保证
- **D持久性(durability)** 由内存+redo log来保证，mysql修改数据同时在内存和redo log记录这次操作，事务提交的时候通过redo log刷盘，宕机的时候可以从redo log恢复

#### [#](#sql-优化的实践经验) SQL 优化的实践经验？

1.对查询进行优化，要尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。

2.应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：

```sql
select id from t where num is null
```

最好不要给数据库留NULL，尽可能的使用 NOT NULL填充数据库.

备注、描述、评论之类的可以设置为 NULL，其他的，最好不要使用NULL。

不要以为 NULL 不需要空间，比如：char(100) 型，在字段建立时，空间就固定了， 不管是否插入值（NULL也包含在内），都是占用 100个字符的空间的，如果是varchar这样的变长字段， null 不占用空间。

可以在num上设置默认值0，确保表中num列没有null值，然后这样查询：

```sql
select id from t where num = 0
```

3.应尽量避免在 where 子句中使用 != 或 <> 操作符，否则将引擎放弃使用索引而进行全表扫描。

4.应尽量避免在 where 子句中使用 or 来连接条件，如果一个字段有索引，一个字段没有索引，将导致引擎放弃使用索引而进行全表扫描，如：

```sql
select id from t where num=10 or Name = 'admin'
```

可以这样查询：

```sql
select id from t where num = 10
union all
select id from t where Name = 'admin'
```

5.in 和 not in 也要慎用，否则会导致全表扫描，如：

```sql
select id from t where num in(1,2,3)
```

对于连续的数值，能用 between 就不要用 in 了：

```sql
select id from t where num between 1 and 3
```

很多时候用 exists 代替 in 是一个好的选择：

```sql
select num from a where num in(select num from b)
```

用下面的语句替换：

```sql
select num from a where exists(select 1 from b where num=a.num)
```

6.下面的查询也将导致全表扫描：

```sql
select id from t where name like ‘%abc%’
```

若要提高效率，可以考虑全文检索。

7.如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：

```sql
select id from t where num = @num
```

可以改为强制查询使用索引：

```sql
select id from t with(index(索引名)) where num = @num
```

.应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```sql
select id from t where num/2 = 100
```

应改为:

```sql
select id from t where num = 100*2
```

9.应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：

```sql
select id from t where substring(name,1,3) = ’abc’       -–name以abc开头的id
select id from t where datediff(day,createdate,’2005-11-30′) = 0    -–‘2005-11-30’    --生成的id
```

应改为:

```sql
select id from t where name like 'abc%'
select id from t where createdate >= '2005-11-30' and createdate < '2005-12-1'
```

10.不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。

11.在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。

12.不要写一些没有意义的查询，如需要生成一个空表结构：

```sql
select col1,col2 into #t from t where 1=0
```

这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样：

```sql
create table #t(…)
```

13.Update 语句，如果只更改1、2个字段，不要Update全部字段，否则频繁调用会引起明显的性能消耗，同时带来大量日志。

14.对于多张大数据量（这里几百条就算大了）的表JOIN，要先分页再JOIN，否则逻辑读会很高，性能很差。

15.select count(*) from table；这样不带任何条件的count会引起全表扫描，并且没有任何业务意义，是一定要杜绝的。

16.索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。

17.应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

18.尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连 接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。

19.尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。

20.任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。

21.尽量使用表变量来代替临时表。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

1. 避免频繁创建和删除临时表，以减少系统表资源的消耗。临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件， 最好使用导出表。

23.在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。

24.如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。

25.尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。

26.使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。

27.与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时 间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。

28.在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

29.尽量避免大事务操作，提高系统并发能力。

30.尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。

#### [#](#buffer-pool、redo-log-buffer-和undo-log、redo-log、bin-log-概念以及关系) Buffer Pool、Redo Log Buffer 和undo log、redo log、bin log 概念以及关系？

- Buffer Pool 是 MySQL 的一个非常重要的组件，因为针对数据库的增删改操作都是在 Buffer Pool 中完成的
- Undo log 记录的是数据操作前的样子
- redo log 记录的是数据被操作后的样子（redo log 是 Innodb 存储引擎特有）
- bin log 记录的是整个操作记录（这个对于主从复制具有非常重要的意义）

#### [#](#从准备更新一条数据到事务的提交的流程描述) 从准备更新一条数据到事务的提交的流程描述？

![img](/images/db/mysql/db-mysql-sql-14.png)

- 首先执行器根据 MySQL 的执行计划来查询数据，先是从缓存池中查询数据，如果没有就会去数据库中查询，如果查询到了就将其放到缓存池中
- 在数据被缓存到缓存池的同时，会写入 undo log 日志文件
- 更新的动作是在 BufferPool 中完成的，同时会将更新后的数据添加到 redo log buffer 中
- 完成以后就可以提交事务，在提交的同时会做以下三件事 
  - 将redo log buffer中的数据刷入到 redo log 文件中
  - 将本次操作记录写入到 bin log文件中
  - 将 bin log 文件名字和更新内容在 bin log 中的位置记录到redo log中，同时在 redo log 最后添加 commit 标记

### [#](#_8-2-mysql) 8.2 MySQL

#### [#](#能说下myisam-和-innodb的区别吗) 能说下myisam 和 innodb的区别吗？

**myisam**引擎是5.1版本之前的默认引擎，支持全文检索、压缩、空间函数等，但是不支持**事务**和**行级锁**，所以一般用于有大量查询少量插入的场景来使用，而且myisam不支持**外键**，并且索引和数据是分开存储的。

**innodb**是基于B+Tree索引建立的，和myisam相反它支持事务、外键，并且通过MVCC来支持高并发，索引和数据存储在一起。

#### [#](#说下mysql的索引有哪些吧) 说下MySQL的索引有哪些吧？

索引在什么层面？

首先，索引是在**存储引擎层实现**的，而不是在服务器层实现的，所以不同存储引擎具有不同的索引类型和实现。

有哪些？

- **B+Tree 索引**
  - 是大多数 MySQL 存储引擎的默认索引类型。
- **哈希索引**
  - 哈希索引能以 O(1) 时间进行查找，但是失去了有序性；
  - InnoDB 存储引擎有一个特殊的功能叫“自适应哈希索引”，当某个索引值被使用的非常频繁时，会在 B+Tree 索引之上再创建一个哈希索引，这样就让 B+Tree 索引具有哈希索引的一些优点，比如快速的哈希查找。
- **全文索引**
  - MyISAM 存储引擎支持全文索引，用于查找文本中的关键词，而不是直接比较是否相等。查找条件使用 MATCH AGAINST，而不是普通的 WHERE。
  - 全文索引一般使用倒排索引实现，它记录着关键词到其所在文档的映射。
  - InnoDB 存储引擎在 MySQL 5.6.4 版本中也开始支持全文索引。
- **空间数据索引**
  - MyISAM 存储引擎支持空间数据索引(R-Tree)，可以用于地理数据存储。空间数据索引会从所有维度来索引数据，可以有效地使用任意维度来进行组合查询。

#### [#](#什么是b-树-为什么b-树成为主要的sql数据库的索引实现) 什么是B+树？为什么B+树成为主要的SQL数据库的索引实现？

- **什么是B+Tree?**

B+ Tree 是基于 B Tree 和叶子节点顺序访问指针进行实现，它具有 B Tree 的平衡性，并且通过顺序访问指针来提高区间查询的性能。在 B+ Tree 中，一个节点中的 key 从左到右非递减排列，如果某个指针的左右相邻 key 分别是 keyi 和 keyi+1，且不为 null，则该指针指向节点的所有 key 大于等于 keyi 且小于等于 keyi+1。

![img](/images/mysql/061c88c1-572f-424f-b580-9cbce903a3fe.png)

- **为什么是B+Tree**?
  - 为了减少磁盘读取次数，决定了树的高度不能高，所以必须是先B-Tree；
  - 以页为单位读取使得一次 I/O 就能完全载入一个节点，且相邻的节点也能够被预先载入；所以数据放在叶子节点，本质上是一个Page页；
  - 为了支持范围查询以及关联关系， 页中数据需要有序，且页的尾部节点指向下个页的头部；
- **B+树索引可分为聚簇索引和非聚簇索引**?

1. 主索引就是聚簇索引（也称聚集索引，clustered index）
2. 辅助索引（有时也称非聚簇索引或二级索引，secondary index，non-clustered index）。

![img](/images/db/mysql/db-mysql-index-1.png)

如上图，**主键索引的叶子节点保存的是真正的数据。而辅助索引叶子节点的数据区保存的是主键索引关键字的值**。

假如要查询name = C 的数据，其搜索过程如下：a) 先在辅助索引中通过C查询最后找到主键id = 9; b) 在主键索引中搜索id为9的数据，最终在主键索引的叶子节点中获取到真正的数据。所以通过辅助索引进行检索，需要检索两次索引。

之所以这样设计，一个原因就是：如果和MyISAM一样在主键索引和辅助索引的叶子节点中都存放数据行指针，一旦数据发生迁移，则需要去重新组织维护所有的索引。

#### [#](#那你知道什么是覆盖索引和回表吗) 那你知道什么是覆盖索引和回表吗？

覆盖索引指的是在一次查询中，如果一个索引包含或者说覆盖所有需要查询的字段的值，我们就称之为覆盖索引，而不再需要回表查询。

而要确定一个查询是否是覆盖索引，我们只需要explain sql语句看Extra的结果是否是“Using index”即可。

比如：

```sql
explain select * from user where age=1; // 查询的name无法从索引数据获取
explain select id,age from user where age=1; //可以直接从索引获取
```

#### [#](#什么是mvcc-说说mysql实现mvcc的原理) 什么是MVCC？ 说说MySQL实现MVCC的原理？

- **什么是MVCC？**

MVCC，全称Multi-Version Concurrency Control，即多版本并发控制。MVCC是一种并发控制的方法，一般在数据库管理系统中，实现对数据库的并发访问，在编程语言中实现事务内存。

在Mysql的InnoDB引擎中就是指在已提交读(READ COMMITTD)和可重复读(REPEATABLE READ)这两种隔离级别下的事务对于SELECT操作会访问版本链中的记录的过程。

这就使得别的事务可以修改这条记录，反正每次修改都会在版本链中记录。SELECT可以去版本链中拿记录，这就实现了读-写，写-读的并发执行，提升了系统的性能。

- **MySQL的InnoDB引擎实现MVCC的3个基础点**

1. **隐式字段**

![img](/images/db/mysql/db-mysql-mvcc-1.png)

如上图，DB_ROW_ID是数据库默认为该行记录生成的唯一隐式主键；DB_TRX_ID是当前操作该记录的事务ID； 而DB_ROLL_PTR是一个回滚指针，用于配合undo日志，指向上一个旧版本；delete flag没有展示出来。

1. **undo log**

![img](/images/db/mysql/db-mysql-mvcc-4.png)

从上面，我们就可以看出，不同事务或者相同事务的对同一记录的修改，会导致该记录的undo log成为一条记录版本线性表，既链表，undo log的链首就是最新的旧记录，链尾就是最早的旧记录

1. **ReadView**

已提交读和可重复读的区别就在于它们生成ReadView的策略不同。

ReadView中主要就是有个列表来存储我们系统中当前活跃着的读写事务，也就是begin了还未提交的事务。通过这个列表来判断记录的某个版本是否对当前事务可见。假设当前列表里的事务id为[80,100]。

a) 如果你要访问的记录版本的事务id为50，比当前列表最小的id80小，那说明这个事务在之前就提交了，所以对当前活动的事务来说是可访问的。

b) 如果你要访问的记录版本的事务id为90,发现此事务在列表id最大值和最小值之间，那就再判断一下是否在列表内，如果在那就说明此事务还未提交，所以版本不能被访问。如果不在那说明事务已经提交，所以版本可以被访问。

c) 如果你要访问的记录版本的事务id为110，那比事务列表最大id100都大，那说明这个版本是在ReadView生成之后才发生的，所以不能被访问。

这些记录都是去undo log 链里面找的，先找最近记录，如果最近这一条记录事务id不符合条件，不可见的话，再去找上一个版本再比较当前事务的id和这个版本事务id看能不能访问，以此类推直到返回可见的版本或者结束。

- **举个例子** ，在已提交读隔离级别下：

比如此时有一个事务id为100的事务，修改了name,使得的name等于小明2，但是事务还没提交。则此时的版本链是

![img](/images/db/mysql/db-mysql-mvcc-11.jpeg)

那此时另一个事务发起了select 语句要查询id为1的记录，那此时生成的ReadView 列表只有[100]。那就去版本链去找了，首先肯定找最近的一条，发现trx_id是100,也就是name为小明2的那条记录，发现在列表内，所以不能访问。

这时候就通过指针继续找下一条，name为小明1的记录，发现trx_id是60，小于列表中的最小id,所以可以访问，直接访问结果为小明1。

那这时候我们把事务id为100的事务提交了，并且新建了一个事务id为110也修改id为1的记录，并且不提交事务

![img](/images/db/mysql/db-mysql-mvcc-12.jpeg)

这时候版本链就是

![img](/images/db/mysql/db-mysql-mvcc-13.jpeg)

这时候之前那个select事务又执行了一次查询,要查询id为1的记录。

**已提交读隔离级别下的事务在每次查询的开始都会生成一个独立的ReadView,而可重复读隔离级别则在第一次读的时候生成一个ReadView，之后的读都复用之前的ReadView**。

1. 如果你是已提交读隔离级别，这时候你会重新一个ReadView，那你的活动事务列表中的值就变了，变成了[110]。按照上的说法，你去版本链通过trx_id对比查找到合适的结果就是小明2。
2. 如果你是可重复读隔离级别，这时候你的ReadView还是第一次select时候生成的ReadView,也就是列表的值还是[100]。所以select的结果是小明1。所以第二次select结果和第一次一样，所以叫可重复读！

这就是Mysql的MVCC,通过版本链，实现多版本，可并发读-写，写-读。通过ReadView生成策略的不同实现不同的隔离级别。

#### [#](#mysql-锁的类型有哪些呢) MySQL 锁的类型有哪些呢？

**说两个维度**：

- 共享锁(简称S锁)和排他锁(简称X锁) 

  - **读锁**是共享的，可以通过lock in share mode实现，这时候只能读不能写。
  - **写锁**是排他的，它会阻塞其他的写锁和读锁。从颗粒度来区分，可以分为表锁和行锁两种。

- 表锁和行锁 

  - **表锁**会锁定整张表并且阻塞其他用户对该表的所有读写操作，比如alter修改表结构的时候会锁表。

  - 行锁

    又可以分为乐观锁和悲观锁 

    - 悲观锁可以通过for update实现
    - 乐观锁则通过版本号实现。

**两个维度结合来看**：

- 共享锁(行锁):Shared Locks 
  - 读锁(s锁),多个事务对于同一数据可以共享访问,不能操作修改
  - 使用方法: 
    - 加锁:SELECT * FROM table WHERE id=1 LOCK IN SHARE MODE
    - 释锁:COMMIT/ROLLBACK
- 排他锁（行锁）：Exclusive Locks 
  - 写锁(X锁)，互斥锁/独占锁,事务获取了一个数据的X锁，其他事务就不能再获取该行的读锁和写锁（S锁、X锁），只有获取了该排他锁的事务是可以对数据行进行读取和修改
  - 使用方法: 
    - DELETE/ UPDATE/ INSERT -- 加锁
    - SELECT * FROM table WHERE ... FOR UPDATE -- 加锁
    - COMMIT/ROLLBACK -- 释锁
- 意向共享锁(IS) 
  - 一个数据行加共享锁前必须先取得该表的IS锁，意向共享锁之间是可以相互兼容的 意向排它锁(IX) 一个数据行加排他锁前必须先取得该表的IX锁，意向排它锁之间是可以相互兼容的 意向锁(IS、IX)是InnoDB引擎操作数据之前自动加的，不需要用户干预; 意义： 当事务操作需要锁表时，只需判断意向锁是否存在，存在时则可快速返回该表不能启用表锁
  - 意向共享锁(IS锁)（表锁）：Intention Shared Locks 
    - 表示事务准备给数据行加入共享锁，也就是说一个数据行加共享锁 前必须先取得该表的IS锁。
  - 意向排它锁(IX锁)（表锁）：Intention Exclusive Locks 
    - 表示事务准备给数据行加入排他锁，说明事务在一个数据行加排他 锁前必须先取得该表的IX锁。

#### [#](#你们数据量级多大-分库分表怎么做的) 你们数据量级多大？分库分表怎么做的？

首先分库分表分为垂直和水平两个方式，一般来说我们拆分的顺序是先垂直后水平。

- **垂直分库**

基于现在微服务拆分来说，都是已经做到了垂直分库了

- **垂直分表**

垂直切分是将一张表按列切分成多个表，通常是按照列的关系密集程度进行切分，也可以利用垂直切分将经常被使用的列和不经常被使用的列切分到不同的表中。

在数据库的层面使用垂直切分将按数据库中表的密集程度部署到不同的库中，例如将原来的电商数据库垂直切分成商品数据库、用户数据库等。

![img](/images/mysql/e130e5b8-b19a-4f1e-b860-223040525cf6.jpg)

- **水平分表**

首先根据业务场景来决定使用什么字段作为分表字段(sharding_key)，比如我们现在日订单1000万，我们大部分的场景来源于C端，我们可以用user_id作为sharding_key，数据查询支持到最近3个月的订单，超过3个月的做归档处理，那么3个月的数据量就是9亿，可以分1024张表，那么每张表的数据大概就在100万左右。

比如用户id为100，那我们都经过hash(100)，然后对1024取模，就可以落到对应的表上了。

![img](/images/mysql/63c2909f-0c5f-496f-9fe5-ee9176b31aba.jpg)

#### [#](#那分表后的id怎么保证唯一性的呢) 那分表后的ID怎么保证唯一性的呢？

因为我们主键默认都是自增的，那么分表之后的主键在不同表就肯定会有冲突了。有几个办法考虑：

- 设定步长，比如1-1024张表我们分别设定1-1024的基础步长，这样主键落到不同的表就不会冲突了。
- 分布式ID，自己实现一套分布式ID生成算法或者使用开源的比如雪花算法这种
- 分表后不使用主键作为查询依据，而是每张表单独新增一个字段作为唯一主键使用，比如订单表订单号是唯一的，不管最终落在哪张表都基于订单号作为查询依据，更新也一样。

#### [#](#分表后非sharding-key的查询怎么处理呢) 分表后非sharding_key的查询怎么处理呢？

- 可以做一个mapping表，比如这时候商家要查询订单列表怎么办呢？不带user_id查询的话你总不能扫全表吧？所以我们可以做一个映射关系表，保存商家和用户的关系，查询的时候先通过商家查询到用户列表，再通过user_id去查询。
- 大宽表，一般而言，商户端对数据实时性要求并不是很高，比如查询订单列表，可以把订单表同步到离线（实时）数仓，再基于数仓去做成一张宽表，再基于其他如es提供查询服务。
- 数据量不是很大的话，比如后台的一些查询之类的，也可以通过多线程扫表，然后再聚合结果的方式来做。或者异步的形式也是可以的。

```java
List<Callable<List<User>>> taskList = Lists.newArrayList();
for (int shardingIndex = 0; shardingIndex < 1024; shardingIndex++) {
    taskList.add(() -> (userMapper.getProcessingAccountList(shardingIndex)));
}
List<ThirdAccountInfo> list = null;
try {
    list = taskExecutor.executeTask(taskList);
} catch (Exception e) {
    //do something
}

public class TaskExecutor {
    public <T> List<T> executeTask(Collection<? extends Callable<T>> tasks) throws Exception {
        List<T> result = Lists.newArrayList();
        List<Future<T>> futures = ExecutorUtil.invokeAll(tasks);
        for (Future<T> future : futures) {
            result.add(future.get());
        }
        return result;
    }
}
```

#### [#](#mysql主从复制) MySQL主从复制？

主要涉及三个线程: binlog 线程、I/O 线程和 SQL 线程。

- **binlog 线程** : 负责将主服务器上的数据更改写入二进制日志中。
- **I/O 线程** : 负责从主服务器上读取二进制日志，并写入从服务器的中继日志中。
- **SQL 线程** : 负责读取中继日志并重放其中的 SQL 语句。

![img](/images/mysql/master-slave.png)

**全同步复制**

主库写入binlog后强制同步日志到从库，所有的从库都执行完成后才返回给客户端，但是很显然这个方式的话性能会受到严重影响。

**半同步复制**

和全同步不同的是，半同步复制的逻辑是这样，从库写入日志成功后返回ACK确认给主库，主库收到至少一个从库的确认就认为写操作完成。

#### [#](#mysql主从的延迟怎么解决呢) MySQL主从的延迟怎么解决呢？

这个问题貌似真的是个无解的问题，只能是说自己来判断了，需要走主库的强制走主库查询。

#### [#](#mysql读写分离方案) MySQL读写分离方案?

主服务器处理写操作以及实时性要求比较高的读操作，而从服务器处理读操作。

读写分离能提高性能的原因在于:

- 主从服务器负责各自的读和写，极大程度缓解了锁的争用；
- 从服务器可以使用 MyISAM，提升查询性能以及节约系统开销；
- 增加冗余，提高可用性。

读写分离常用代理方式来实现，代理服务器接收应用层传来的读写请求，然后决定转发到哪个服务器。

![img](/images/mysql/master-slave-proxy.png)

### [#](#_8-3-redis) 8.3 Redis

#### [#](#什么是redis-为什么用redis) 什么是Redis，为什么用Redis？

Redis是一种支持key-value等多种数据结构的存储系统。可用于缓存，事件发布或订阅，高速队列等场景。支持网络，提供字符串，哈希，列表，队列，集合结构直接存取，基于内存，可持久化。

- 读写性能优异
  - Redis能读的速度是110000次/s,写的速度是81000次/s （测试条件见下一节）。
- 数据类型丰富
  - Redis支持二进制案例的 Strings, Lists, Hashes, Sets 及 Ordered Sets 数据类型操作。
- 原子性
  - Redis的所有操作都是原子性的，同时Redis还支持对几个操作全并后的原子性执行。
- 丰富的特性
  - Redis支持 publish/subscribe, 通知, key 过期等特性。
- 持久化
  - Redis支持RDB, AOF等持久化方式
- 发布订阅
  - Redis支持发布/订阅模式
- 分布式
  - Redis Cluster

#### [#](#为什么redis-是单线程的以及为什么这么快) 为什么Redis 是单线程的以及为什么这么快？

- redis完全基于内存,绝大部分请求是纯粹的内存操作,非常快速.
- 数据结构简单,对数据操作也简单,redis中的数据结构是专门进行设计的
- 采用单线程模型, 避免了不必要的上下文切换和竞争条件, 也不存在多线程或者多线程切换而消耗CPU, 不用考虑各种锁的问题, 不存在加锁, 释放锁的操作, 没有因为可能出现死锁而导致性能消耗
- 使用了多路IO复用模型,非阻塞IO
- 使用底层模型不同,它们之间底层实现方式及与客户端之间的 通信的应用协议不一样,Redis直接构建了自己的VM机制,因为一般的系统调用系统函数的话,会浪费一定的时间去移动和请求

#### [#](#redis-一般有哪些使用场景) Redis 一般有哪些使用场景？

可以结合自己的项目讲讲，比如

- **热点数据的缓存**

缓存是Redis最常见的应用场景，之所有这么使用，主要是因为Redis读写性能优异。而且逐渐有取代memcached，成为首选服务端缓存的组件。而且，Redis内部是支持事务的，在使用时候能有效保证数据的一致性。

- **限时业务的运用**

redis中可以使用expire命令设置一个键的生存时间，到时间后redis会删除它。利用这一特性可以运用在限时的优惠活动信息、手机验证码等业务场景。

- **计数器相关问题**

redis由于incrby命令可以实现原子性的递增，所以可以运用于高并发的秒杀活动、分布式序列号的生成、具体业务还体现在比如限制一个手机号发多少条短信、一个接口一分钟限制多少请求、一个接口一天限制调用多少次等等。

- **分布式锁**

这个主要利用redis的setnx命令进行，setnx："set if not exists"就是如果不存在则成功设置缓存同时返回1，否则返回0 ，这个特性在俞你奔远方的后台中有所运用，因为我们服务器是集群的，定时任务可能在两台机器上都会运行，所以在定时任务中首先 通过setnx设置一个lock，如果成功设置则执行，如果没有成功设置，则表明该定时任务已执行。 当然结合具体业务，我们可以给这个lock加一个过期时间，比如说30分钟执行一次的定时任务，那么这个过期时间设置为小于30分钟的一个时间就可以，这个与定时任务的周期以及定时任务执行消耗时间相关。

在分布式锁的场景中，主要用在比如秒杀系统等。

#### [#](#redis-有哪些数据类型) Redis 有哪些数据类型？

- **5种基础数据类型**，分别是：String、List、Set、Zset、Hash。

![img](/images/db/redis/db-redis-ds-1.jpeg)

| 结构类型         | 结构存储的值                               | 结构的读写能力                                               |
| ---------------- | ------------------------------------------ | ------------------------------------------------------------ |
| **String字符串** | 可以是字符串、整数或浮点数                 | 对整个字符串或字符串的一部分进行操作；对整数或浮点数进行自增或自减操作； |
| **List列表**     | 一个链表，链表上的每个节点都包含一个字符串 | 对链表的两端进行push和pop操作，读取单个或多个元素；根据值查找或删除元素； |
| **Set集合**      | 包含字符串的无序集合                       | 字符串的集合，包含基础的方法有看是否存在添加、获取、删除；还包含计算交集、并集、差集等 |
| **Hash散列**     | 包含键值对的无序散列表                     | 包含方法有添加、获取、删除单个元素                           |
| **Zset有序集合** | 和散列一样，用于存储键值对                 | 字符串成员与浮点数分数之间的有序映射；元素的排列顺序由分数的大小决定；包含方法有添加、获取、删除单个元素以及根据分值范围或成员来获取元素 |

- **三种特殊的数据类型** 分别是 HyperLogLogs（基数统计）， Bitmaps (位图) 和 geospatial （地理位置)

#### [#](#谈谈redis-的对象机制-redisobject) 谈谈Redis 的对象机制（redisObject)？

比如说， 集合类型就可以由字典和整数集合两种不同的数据结构实现， 但是， 当用户执行 ZADD 命令时， 他/她应该不必关心集合使用的是什么编码， 只要 Redis 能按照 ZADD 命令的指示， 将新元素添加到集合就可以了。

这说明, **操作数据类型的命令除了要对键的类型进行检查之外, 还需要根据数据类型的不同编码进行多态处理**.

为了解决以上问题, **Redis 构建了自己的类型系统**, 这个系统的主要功能包括:

- redisObject 对象.
- 基于 redisObject 对象的类型检查.
- 基于 redisObject 对象的显式多态函数.
- 对 redisObject 进行分配、共享和销毁的机制.

```c
/*
 * Redis 对象
 */
typedef struct redisObject {

    // 类型
    unsigned type:4;

    // 编码方式
    unsigned encoding:4;

    // LRU - 24位, 记录最末一次访问时间（相对于lru_clock）; 或者 LFU（最少使用的数据：8位频率，16位访问时间）
    unsigned lru:LRU_BITS; // LRU_BITS: 24

    // 引用计数
    int refcount;

    // 指向底层数据结构实例
    void *ptr;

} robj;
```

下图对应上面的结构

![img](/images/db/redis/db-redis-object-1.png)

#### [#](#redis-数据类型有哪些底层数据结构) Redis 数据类型有哪些底层数据结构？

![img](/images/db/redis/db-redis-object-2-3.png)

- 简单动态字符串 - sds
- 压缩列表 - ZipList
- 快表 - QuickList
- 字典/哈希表 - Dict
- 整数集 - IntSet
- 跳表 - ZSkipList

#### [#](#为什么要设计sds) 为什么要设计sds？

- **常数复杂度获取字符串长度**

由于 len 属性的存在，我们获取 SDS 字符串的长度只需要读取 len 属性，时间复杂度为 O(1)。而对于 C 语言，获取字符串的长度通常是经过遍历计数来实现的，时间复杂度为 O(n)。通过 `strlen key` 命令可以获取 key 的字符串长度。

- **杜绝缓冲区溢出**

我们知道在 C 语言中使用 `strcat` 函数来进行两个字符串的拼接，一旦没有分配足够长度的内存空间，就会造成缓冲区溢出。而对于 SDS 数据类型，在进行字符修改的时候，**会首先根据记录的 len 属性检查内存空间是否满足需求**，如果不满足，会进行相应的空间扩展，然后在进行修改操作，所以不会出现缓冲区溢出。

- **减少修改字符串的内存重新分配次数**

C语言由于不记录字符串的长度，所以如果要修改字符串，必须要重新分配内存（先释放再申请），因为如果没有重新分配，字符串长度增大时会造成内存缓冲区溢出，字符串长度减小时会造成内存泄露。

而对于SDS，由于`len`属性和`alloc`属性的存在，对于修改字符串SDS实现了**空间预分配**和**惰性空间释放**两种策略：

1. **空间预分配**：对字符串进行空间扩展的时候，扩展的内存比实际需要的多，这样可以减少连续执行字符串增长操作所需的内存重分配次数。
2. **惰性空间释放**：对字符串进行缩短操作时，程序不立即使用内存重新分配来回收缩短后多余的字节，而是使用 `alloc` 属性将这些字节的数量记录下来，等待后续使用。（当然SDS也提供了相应的API，当我们有需要时，也可以手动释放这些未使用的空间。）

- **二进制安全**

因为C字符串以空字符作为字符串结束的标识，而对于一些二进制文件（如图片等），内容可能包括空字符串，因此C字符串无法正确存取；而所有 SDS 的API 都是以处理二进制的方式来处理 `buf` 里面的元素，并且 SDS 不是以空字符串来判断是否结束，而是以 len 属性表示的长度来判断字符串是否结束。

- **兼容部分 C 字符串函数**

虽然 SDS 是二进制安全的，但是一样遵从每个字符串都是以空字符串结尾的惯例，这样可以重用 C 语言库`<string.h>` 中的一部分函数。

#### [#](#redis-一个字符串类型的值能存储最大容量是多少) Redis 一个字符串类型的值能存储最大容量是多少？

512M

#### [#](#为什么会设计stream) 为什么会设计Stream？

用过Redis做消息队列的都了解，基于Reids的消息队列实现有很多种，例如：

- PUB/SUB，订阅/发布模式

  - 但是发布订阅模式是无法持久化的，如果出现网络断开、Redis 宕机等，消息就会被丢弃；

- 基于

  List LPUSH+BRPOP

   或者 

  基于Sorted-Set

  的实现 

  - 支持了持久化，但是不支持多播，分组消费等

**消费组消费图**

![img](/images/db/redis/db-redis-stream-3.png)

#### [#](#redis-stream用在什么样场景) Redis Stream用在什么样场景？

可用作时通信等，大数据分析，异地数据备份等

![img](/images/db/redis/db-redis-stream-4.png)

客户端可以平滑扩展，提高处理能力

![img](/images/db/redis/db-redis-stream-5.png)

#### [#](#redis-stream消息id的设计是否考虑了时间回拨的问题) Redis Stream消息ID的设计是否考虑了时间回拨的问题？

XADD生成的1553439850328-0，就是Redis生成的消息ID，由两部分组成:**时间戳-序号**。时间戳是毫秒级单位，是生成消息的Redis服务器时间，它是个64位整型（int64）。序号是在这个毫秒时间点内的消息序号，它也是个64位整型。

可以通过multi批处理，来验证序号的递增：

```bash
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> XADD memberMessage * msg one
QUEUED
127.0.0.1:6379> XADD memberMessage * msg two
QUEUED
127.0.0.1:6379> XADD memberMessage * msg three
QUEUED
127.0.0.1:6379> XADD memberMessage * msg four
QUEUED
127.0.0.1:6379> XADD memberMessage * msg five
QUEUED
127.0.0.1:6379> EXEC
1) "1553441006884-0"
2) "1553441006884-1"
3) "1553441006884-2"
4) "1553441006884-3"
5) "1553441006884-4"
```

由于一个redis命令的执行很快，所以可以看到在同一时间戳内，是通过序号递增来表示消息的。

为了保证消息是有序的，因此Redis生成的ID是单调递增有序的。由于ID中包含时间戳部分，为了避免服务器时间错误而带来的问题（例如服务器时间延后了），Redis的每个Stream类型数据都维护一个latest_generated_id属性，用于记录最后一个消息的ID。**若发现当前时间戳退后（小于latest_generated_id所记录的），则采用时间戳不变而序号递增的方案来作为新消息ID**（这也是序号为什么使用int64的原因，保证有足够多的的序号），从而保证ID的单调递增性质。

强烈建议使用Redis的方案生成消息ID，因为这种时间戳+序号的单调递增的ID方案，几乎可以满足你全部的需求。但同时，记住ID是支持自定义的，别忘了！

#### [#](#redis-stream消费者崩溃带来的会不会消息丢失问题) Redis Stream消费者崩溃带来的会不会消息丢失问题?

为了解决组内消息读取但处理期间消费者崩溃带来的消息丢失问题，STREAM 设计了 Pending 列表，用于记录读取但并未处理完毕的消息。命令XPENDIING 用来获消费组或消费内消费者的未处理完毕的消息。演示如下：

```bash
127.0.0.1:6379> XPENDING mq mqGroup # mpGroup的Pending情况
1) (integer) 5 # 5个已读取但未处理的消息
2) "1553585533795-0" # 起始ID
3) "1553585533795-4" # 结束ID
4) 1) 1) "consumerA" # 消费者A有3个
      2) "3"
   2) 1) "consumerB" # 消费者B有1个
      2) "1"
   3) 1) "consumerC" # 消费者C有1个
      2) "1"

127.0.0.1:6379> XPENDING mq mqGroup - + 10 # 使用 start end count 选项可以获取详细信息
1) 1) "1553585533795-0" # 消息ID
   2) "consumerA" # 消费者
   3) (integer) 1654355 # 从读取到现在经历了1654355ms，IDLE
   4) (integer) 5 # 消息被读取了5次，delivery counter
2) 1) "1553585533795-1"
   2) "consumerA"
   3) (integer) 1654355
   4) (integer) 4
# 共5个，余下3个省略 ...

127.0.0.1:6379> XPENDING mq mqGroup - + 10 consumerA # 在加上消费者参数，获取具体某个消费者的Pending列表
1) 1) "1553585533795-0"
   2) "consumerA"
   3) (integer) 1641083
   4) (integer) 5
# 共3个，余下2个省略 ...
```

每个Pending的消息有4个属性：

- 消息ID
- 所属消费者
- IDLE，已读取时长
- delivery counter，消息被读取次数

上面的结果我们可以看到，我们之前读取的消息，都被记录在Pending列表中，说明全部读到的消息都没有处理，仅仅是读取了。那如何表示消费者处理完毕了消息呢？使用命令 XACK 完成告知消息处理完成，演示如下：

```bash
127.0.0.1:6379> XACK mq mqGroup 1553585533795-0 # 通知消息处理结束，用消息ID标识
(integer) 1

127.0.0.1:6379> XPENDING mq mqGroup # 再次查看Pending列表
1) (integer) 4 # 已读取但未处理的消息已经变为4个
2) "1553585533795-1"
3) "1553585533795-4"
4) 1) 1) "consumerA" # 消费者A，还有2个消息处理
      2) "2"
   2) 1) "consumerB"
      2) "1"
   3) 1) "consumerC"
      2) "1"
127.0.0.1:6379>
```

有了这样一个Pending机制，就意味着在某个消费者读取消息但未处理后，消息是不会丢失的。等待消费者再次上线后，可以读取该Pending列表，就可以继续处理该消息了，保证消息的有序和不丢失。

#### [#](#redis-steam-坏消息问题-死信问题) Redis Steam 坏消息问题，死信问题?

正如上面所说，如果某个消息，不能被消费者处理，也就是不能被XACK，这是要长时间处于Pending列表中，即使被反复的转移给各个消费者也是如此。此时该消息的delivery counter就会累加（上一节的例子可以看到），当累加到某个我们预设的临界值时，我们就认为是坏消息（也叫死信，DeadLetter，无法投递的消息），由于有了判定条件，我们将坏消息处理掉即可，删除即可。删除一个消息，使用XDEL语法，演示如下：

```bash
# 删除队列中的消息
127.0.0.1:6379> XDEL mq 1553585533795-1
(integer) 1
# 查看队列中再无此消息
127.0.0.1:6379> XRANGE mq - +
1) 1) "1553585533795-0"
   2) 1) "msg"
      2) "1"
2) 1) "1553585533795-2"
   2) 1) "msg"
      2) "3"
```

注意本例中，并没有删除Pending中的消息因此你查看Pending，消息还会在。可以执行XACK标识其处理完毕！

#### [#](#redis-的持久化机制是什么-各自的优缺点-一般怎么用) Redis 的持久化机制是什么？各自的优缺点？一般怎么用？

1. RDB持久化是把当前进程数据生成快照保存到磁盘上的过程; 针对RDB不适合实时持久化的问题，Redis提供了AOF持久化方式来解决.
2. AOF是“写后”日志，Redis先执行命令，把数据写入内存，然后才记录日志。日志里记录的是Redis收到的每一条命令，这些命令是以文本形式保存。
3. Redis 4.0 中提出了一个**混合使用 AOF 日志和内存快照**的方法。简单来说，内存快照以一定的频率执行，在两次快照之间，使用 AOF 日志记录这期间的所有命令操作。

这样一来，快照不用很频繁地执行，这就避免了频繁 fork 对主线程的影响。而且，AOF 日志也只用记录两次快照间的操作，也就是说，不需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写开销。

如下图所示，T1 和 T2 时刻的修改，用 AOF 日志记录，等到第二次做全量快照时，就可以清空 AOF 日志，因为此时的修改都已经记录到快照中了，恢复时就不再用日志了。

![img](/images/db/redis/redis-x-rdb-4.jpg)

这个方法既能享受到 RDB 文件快速恢复的好处，又能享受到 AOF 只记录操作命令的简单优势, 实际环境中用的很多。

#### [#](#rdb-触发方式) RDB 触发方式?

触发rdb持久化的方式有2种，分别是**手动触发**和**自动触发**。

- 手动触发
  - **save命令**：阻塞当前Redis服务器，直到RDB过程完成为止，对于内存 比较大的实例会造成长时间**阻塞**，线上环境不建议使用
  - **bgsave命令**：Redis进程执行fork操作创建子进程，RDB持久化过程由子 进程负责，完成后自动结束。阻塞只发生在fork阶段，一般时间很短

bgsave流程图如下所示

![img](/images/db/redis/redis-x-rdb-1.png)

- 自动触发
  - redis.conf中配置`save m n`，即在m秒内有n次修改时，自动触发bgsave生成rdb文件；
  - 主从复制时，从节点要从主节点进行全量复制时也会触发bgsave操作，生成当时的快照发送到从节点；
  - 执行debug reload命令重新加载redis时也会触发bgsave操作；
  - 默认情况下执行shutdown命令时，如果没有开启aof持久化，那么也会触发bgsave操作；

#### [#](#rdb由于生产环境中我们为redis开辟的内存区域都比较大-例如6gb-那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间-而实际情况是这段时间redis服务一般都会收到数据写操作请求。那么如何保证数据一致性呢) RDB由于生产环境中我们为Redis开辟的内存区域都比较大（例如6GB），那么将内存中的数据同步到硬盘的过程可能就会持续比较长的时间，而实际情况是这段时间Redis服务一般都会收到数据写操作请求。那么如何保证数据一致性呢？

RDB中的核心思路是Copy-on-Write，来保证在进行快照操作的这段时间，需要压缩写入磁盘上的数据在内存中不会发生变化。在正常的快照操作中，一方面Redis主进程会fork一个新的快照进程专门来做这个事情，这样保证了Redis服务不会停止对客户端包括写请求在内的任何响应。另一方面这段时间发生的数据变化会以副本的方式存放在另一个新的内存区域，待快照操作结束后才会同步到原来的内存区域。

举个例子：如果主线程对这些数据也都是读操作（例如图中的键值对 A），那么，主线程和 bgsave 子进程相互不影响。但是，如果主线程要修改一块数据（例如图中的键值对 C），那么，这块数据就会被复制一份，生成该数据的副本。然后，bgsave 子进程会把这个副本数据写入 RDB 文件，而在这个过程中，主线程仍然可以直接修改原来的数据。

![img](/images/db/redis/redis-x-aof-42.jpg)

#### [#](#在进行rdb快照操作的这段时间-如果发生服务崩溃怎么办) 在进行RDB快照操作的这段时间，如果发生服务崩溃怎么办？

很简单，在没有将数据全部写入到磁盘前，这次快照操作都不算成功。如果出现了服务崩溃的情况，将以上一次完整的RDB快照文件作为恢复内存数据的参考。也就是说，在快照操作过程中不能影响上一次的备份数据。Redis服务会在磁盘上创建一个临时文件进行数据操作，待操作成功后才会用这个临时文件替换掉上一次的备份。

#### [#](#可以每秒做一次rdb快照吗) 可以每秒做一次RDB快照吗？

对于快照来说，所谓“连拍”就是指连续地做快照。这样一来，快照的间隔时间变得很短，即使某一时刻发生宕机了，因为上一时刻快照刚执行，丢失的数据也不会太多。但是，这其中的快照间隔时间就很关键了。

如下图所示，我们先在 T0 时刻做了一次快照，然后又在 T0+t 时刻做了一次快照，在这期间，数据块 5 和 9 被修改了。如果在 t 这段时间内，机器宕机了，那么，只能按照 T0 时刻的快照进行恢复。此时，数据块 5 和 9 的修改值因为没有快照记录，就无法恢复了。 　　 ![img](/images/db/redis/redis-x-rdb-2.jpg)

所以，要想尽可能恢复数据，t 值就要尽可能小，t 越小，就越像“连拍”。那么，t 值可以小到什么程度呢，比如说是不是可以每秒做一次快照？毕竟，每次快照都是由 bgsave 子进程在后台执行，也不会阻塞主线程。

这种想法其实是错误的。虽然 bgsave 执行时不阻塞主线程，但是，**如果频繁地执行全量快照，也会带来两方面的开销**：

- 一方面，频繁将全量数据写入磁盘，会给磁盘带来很大压力，多个快照竞争有限的磁盘带宽，前一个快照还没有做完，后一个又开始做了，容易造成恶性循环。
- 另一方面，bgsave 子进程需要通过 fork 操作从主线程创建出来。虽然，子进程在创建后不会再阻塞主线程，但是，fork 这个创建过程本身会阻塞主线程，而且主线程的内存越大，阻塞时间越长。如果频繁 fork 出 bgsave 子进程，这就会频繁**阻塞主线程**了。

那么，有什么其他好方法吗？此时，我们可以做增量快照，就是指做了一次全量快照后，后续的快照只对修改的数据进行快照记录，这样可以避免每次全量快照的开销。这个比较好理解。

但是它需要我们使用额外的元数据信息去记录哪些数据被修改了，这会带来额外的**空间开销问题**。那么，还有什么方法既能利用 RDB 的快速恢复，又能以较小的开销做到尽量少丢数据呢？RDB和AOF的混合方式。

#### [#](#aof是写前日志还是写后日志) AOF是写前日志还是写后日志？

AOF日志采用写后日志，即**先写内存，后写日志**。

![img](/images/db/redis/redis-x-aof-41.jpg)

**为什么采用写后日志**？

Redis要求高性能，采用写日志有两方面好处：

- **避免额外的检查开销**：Redis 在向 AOF 里面记录日志的时候，并不会先去对这些命令进行语法检查。所以，如果先记日志再执行命令的话，日志中就有可能记录了错误的命令，Redis 在使用日志恢复数据时，就可能会出错。
- 不会阻塞当前的写操作

但这种方式存在潜在风险：

- 如果命令执行完成，写日志之前宕机了，会丢失数据。
- 主线程写磁盘压力大，导致写盘慢，阻塞后续操作。

#### [#](#如何实现aof的) 如何实现AOF的？

AOF日志记录Redis的每个写命令，步骤分为：命令追加（append）、文件写入（write）和文件同步（sync）。

- **命令追加** 当AOF持久化功能打开了，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器的 aof_buf 缓冲区。
- **文件写入和同步** 关于何时将 aof_buf 缓冲区的内容写入AOF文件中，Redis提供了三种写回策略：

![img](/images/db/redis/redis-x-aof-4.jpg)

`Always`，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；

`Everysec`，每秒写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；

`No`，操作系统控制的写回：每个写命令执行完，只是先把日志写到AOF文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。

- **三种写回策略的优缺点**

上面的三种写回策略体现了一个重要原则：**trade-off**，取舍，指在性能和可靠性保证之间做取舍。

关于AOF的同步策略是涉及到操作系统的 write 函数和 fsync 函数的，在《Redis设计与实现》中是这样说明的：

```bash
为了提高文件写入效率，在现代操作系统中，当用户调用write函数，将一些数据写入文件时，操作系统通常会将数据暂存到一个内存缓冲区里，当缓冲区的空间被填满或超过了指定时限后，才真正将缓冲区的数据写入到磁盘里。

这样的操作虽然提高了效率，但也为数据写入带来了安全问题：如果计算机停机，内存缓冲区中的数据会丢失。为此，系统提供了fsync、fdatasync同步函数，可以强制操作系统立刻将缓冲区中的数据写入到硬盘里，从而确保写入数据的安全性。
```

#### [#](#什么是aof重写) 什么是AOF重写？

Redis通过创建一个新的AOF文件来替换现有的AOF，新旧两个AOF文件保存的数据相同，但新AOF文件没有了冗余命令。

![img](/images/db/redis/redis-x-aof-1.jpg)

#### [#](#aof重写会阻塞吗) AOF重写会阻塞吗？

AOF重写过程是由后台进程bgrewriteaof来完成的。主线程fork出后台的bgrewriteaof子进程，fork会把主线程的内存拷贝一份给bgrewriteaof子进程，这里面就包含了数据库的最新数据。然后，bgrewriteaof子进程就可以在不影响主线程的情况下，逐一把拷贝的数据写成操作，记入重写日志。

所以aof在重写时，在fork进程时是会阻塞住主线程的。

#### [#](#aof日志何时会重写) AOF日志何时会重写？

有两个配置项控制AOF重写的触发：

`auto-aof-rewrite-min-size`:表示运行AOF重写时文件的最小大小，默认为64MB。

`auto-aof-rewrite-percentage`:这个值的计算方式是，当前aof文件大小和上一次重写后aof文件大小的差值，再除以上一次重写后aof文件大小。也就是当前aof文件比上一次重写后aof文件的增量大小，和上一次重写后aof文件大小的比值。

#### [#](#aof重写日志时-有新数据写入咋整) AOF重写日志时，有新数据写入咋整？

重写过程总结为：“一个拷贝，两处日志”。在fork出子进程时的拷贝，以及在重写时，如果有新数据写入，主线程就会将命令记录到两个aof日志内存缓冲区中。如果AOF写回策略配置的是always，则直接将命令写回旧的日志文件，并且保存一份命令至AOF重写缓冲区，这些操作对新的日志文件是不存在影响的。（旧的日志文件：主线程使用的日志文件，新的日志文件：bgrewriteaof进程使用的日志文件）

而在bgrewriteaof子进程完成会日志文件的重写操作后，会提示主线程已经完成重写操作，主线程会将AOF重写缓冲中的命令追加到新的日志文件后面。这时候在高并发的情况下，AOF重写缓冲区积累可能会很大，这样就会造成阻塞，Redis后来通过Linux管道技术让aof重写期间就能同时进行回放，这样aof重写结束后只需回放少量剩余的数据即可。

最后通过修改文件名的方式，保证文件切换的原子性。

在AOF重写日志期间发生宕机的话，因为日志文件还没切换，所以恢复数据时，用的还是旧的日志文件。

#### [#](#主线程fork出子进程的是如何复制内存数据的) 主线程fork出子进程的是如何复制内存数据的？

fork采用操作系统提供的写时复制（copy on write）机制，就是为了避免一次性拷贝大量内存数据给子进程造成阻塞。fork子进程时，子进程时会拷贝父进程的页表，即虚实映射关系（虚拟内存和物理内存的映射索引表），而不会拷贝物理内存。这个拷贝会消耗大量cpu资源，并且拷贝完成前会阻塞主线程，阻塞时间取决于内存中的数据量，数据量越大，则内存页表越大。拷贝完成后，父子进程使用相同的内存地址空间。

但主进程是可以有数据写入的，这时候就会拷贝物理内存中的数据。如下图（进程1看做是主进程，进程2看做是子进程）：

![img](/images/db/redis/redis-x-aof-3.png)

在主进程有数据写入时，而这个数据刚好在页c中，操作系统会创建这个页面的副本（页c的副本），即拷贝当前页的物理数据，将其映射到主进程中，而子进程还是使用原来的的页c。

#### [#](#在重写日志整个过程时-主线程有哪些地方会被阻塞) 在重写日志整个过程时，主线程有哪些地方会被阻塞？

1. fork子进程时，需要拷贝虚拟页表，会对主线程阻塞。
2. 主进程有bigkey写入时，操作系统会创建页面的副本，并拷贝原有的数据，会对主线程阻塞。
3. 子进程重写日志完成后，主进程追加aof重写缓冲区时可能会对主线程阻塞。

#### [#](#为什么aof重写不复用原aof日志) 为什么AOF重写不复用原AOF日志？

两方面原因：

1. 父子进程写同一个文件会产生竞争问题，影响父进程的性能。
2. 如果AOF重写过程中失败了，相当于污染了原本的AOF文件，无法做恢复数据使用。

#### [#](#redis-过期键的删除策略有哪些) Redis 过期键的删除策略有哪些?

在单机版Redis中，存在两种删除策略：

- **惰性删除**：服务器不会主动删除数据，只有当客户端查询某个数据时，服务器判断该数据是否过期，如果过期则删除。
- **定期删除**：服务器执行定时任务删除过期数据，但是考虑到内存和CPU的折中（删除会释放内存，但是频繁的删除操作对CPU不友好），该删除的频率和执行时间都受到了限制。

在主从复制场景下，为了主从节点的数据一致性，从节点不会主动删除数据，而是由主节点控制从节点中过期数据的删除。由于主节点的惰性删除和定期删除策略，都不能保证主节点及时对过期数据执行删除操作，因此，当客户端通过Redis从节点读取数据时，很容易读取到已经过期的数据。

Redis 3.2中，从节点在读取数据时，增加了对数据是否过期的判断：如果该数据已过期，则不返回给客户端；将Redis升级到3.2可以解决数据过期问题。

#### [#](#redis-内存淘汰算法有哪些) Redis 内存淘汰算法有哪些?

Redis共支持八种淘汰策略，分别是noeviction、volatile-random、volatile-ttl、volatile-lru、volatile-lfu、allkeys-lru、allkeys-random 和 allkeys-lfu 策略。

**怎么理解呢**？主要看分三类看：

- 不淘汰 
  - noeviction （v4.0后默认的）
- 对设置了过期时间的数据中进行淘汰 
  - 随机：volatile-random
  - ttl：volatile-ttl
  - lru：volatile-lru
  - lfu：volatile-lfu
- 全部数据进行淘汰 
  - 随机：allkeys-random
  - lru：allkeys-lru
  - lfu：allkeys-lfu

**LRU算法**：LRU 算法的全称是 Least Recently Used，按照最近最少使用的原则来筛选数据。这种模式下会使用 LRU 算法筛选设置了过期时间的键值对。

Redis优化的**LRU算法实现**：

Redis会记录每个数据的最近一次被访问的时间戳。在Redis在决定淘汰的数据时，第一次会随机选出 N 个数据，把它们作为一个候选集合。接下来，Redis 会比较这 N 个数据的 lru 字段，把 lru 字段值最小的数据从缓存中淘汰出去。通过随机读取待删除集合，可以让Redis不用维护一个巨大的链表，也不用操作链表，进而提升性能。

**LFU 算法**：LFU 缓存策略是在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。当使用 LFU 策略筛选淘汰数据时，首先会根据数据的访问次数进行筛选，把访问次数最低的数据淘汰出缓存。如果两个数据的访问次数相同，LFU 策略再比较这两个数据的访问时效性，把距离上一次访问时间更久的数据淘汰出缓存。 Redis的LFU算法实现:

当 LFU 策略筛选数据时，Redis 会在候选集合中，根据数据 lru 字段的后 8bit 选择访问次数最少的数据进行淘汰。当访问次数相同时，再根据 lru 字段的前 16bit 值大小，选择访问时间最久远的数据进行淘汰。

Redis 只使用了 8bit 记录数据的访问次数，而 8bit 记录的最大值是 255，这样在访问快速的情况下，如果每次被访问就将访问次数加一，很快某条数据就达到最大值255，可能很多数据都是255，那么退化成LRU算法了。所以Redis为了解决这个问题，实现了一个更优的计数规则，并可以通过配置项，来控制计数器增加的速度。

#### [#](#redis的内存用完了会发生什么) Redis的内存用完了会发生什么？

如果达到设置的上限，Redis的写命令会返回错误信息（但是读命令还可以正常返回。）或者你可以配置内存淘汰机制，当Redis达到内存上限时会冲刷掉旧的内容。

#### [#](#redis如何做内存优化) Redis如何做内存优化？

1. 缩减键值对象: 缩减键（key）和值（value）的长度，

key长度：如在设计键时，在完整描述业务情况下，键值越短越好。

value长度：值对象缩减比较复杂，常见需求是把业务对象序列化成二进制数组放入Redis。首先应该在业务上精简业务对象，去掉不必要的属性避免存储无效数据。其次在序列化工具选择上，应该选择更高效的序列化工具来降低字节数组大小。以JAVA为例，内置的序列化方式无论从速度还是压缩比都不尽如人意，这时可以选择更高效的序列化工具，如: protostuff，kryo等，下图是JAVA常见序列化工具空间压缩对比。

1. 共享对象池

对象共享池指Redis内部维护[0-9999]的整数对象池。创建大量的整数类型redisObject存在内存开销，每个redisObject内部结构至少占16字节，甚至超过了整数自身空间消耗。所以Redis内存维护一个[0-9999]的整数对象池，用于节约内存。 除了整数值对象，其他类型如list,hash,set,zset内部元素也可以使用整数对象池。因此开发中在满足需求的前提下，尽量使用整数对象以节省内存。

1. 字符串优化
2. 编码优化
3. 控制key的数量

#### [#](#redis-key-的过期时间和永久有效分别怎么设置) Redis key 的过期时间和永久有效分别怎么设置？

EXPIRE 和 PERSIST 命令

#### [#](#redis-中的管道有什么用) Redis 中的管道有什么用？

一次请求/响应服务器能实现处理新的请求即使旧的请求还未被响应，这样就可以将多个命令发送到服务器，而不用等待回复，最后在一个步骤中读取该答复。

这就是管道（pipelining），是一种几十年来广泛使用的技术。例如许多 POP3 协议已经实现支持这个功能，大大加快了从服务器下载新邮件的过程。

#### [#](#什么是redis事务) 什么是redis事务？

Redis 事务的本质是一组命令的集合。事务支持一次执行多个命令，一个事务中所有命令都会被序列化。在事务执行过程，会按照顺序串行化执行队列中的命令，其他客户端提交的命令请求不会插入到事务执行命令序列中。

总结说：redis事务就是一次性、顺序性、排他性的执行一个队列中的一系列命令。

#### [#](#redis事务相关命令) Redis事务相关命令？

MULTI 、 EXEC 、 DISCARD 和 WATCH 是 Redis 事务相关的命令。

- MULTI ：开启事务，redis会将后续的命令逐个放入队列中，然后使用EXEC命令来原子化执行这个命令系列。
- EXEC：执行事务中的所有操作命令。
- DISCARD：取消事务，放弃执行事务块中的所有命令。
- WATCH：监视一个或多个key,如果事务在执行前，这个key(或多个key)被其他命令修改，则事务被中断，不会执行事务中的任何命令。
- UNWATCH：取消WATCH对所有key的监视。

#### [#](#redis事务的三个阶段) Redis事务的三个阶段？

Redis事务执行是三个阶段：

- **开启**：以MULTI开始一个事务
- **入队**：将多个命令入队到事务中，接到这些命令并不会立即执行，而是放到等待执行的事务队列里面
- **执行**：由EXEC命令触发事务

当一个客户端切换到事务状态之后， 服务器会根据这个客户端发来的不同命令执行不同的操作：

- 如果客户端发送的命令为 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令的其中一个， 那么服务器立即执行这个命令。
- 与此相反， 如果客户端发送的命令是 EXEC 、 DISCARD 、 WATCH 、 MULTI 四个命令以外的其他命令， 那么服务器并不立即执行这个命令， 而是将这个命令放入一个事务队列里面， 然后向客户端返回 QUEUED 回复。

![img](/images/db/redis/db-redis-trans-1.png)

#### [#](#redis事务其它实现) Redis事务其它实现？

- 基于Lua脚本，Redis可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完
- 基于中间标记变量，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐

#### [#](#redis事务中出现错误的处理) Redis事务中出现错误的处理？

- **语法错误（编译器错误）**

在开启事务后，修改k1值为11，k2值为22，但k2语法错误，**最终导致事务提交失败，k1、k2保留原值**。

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k2 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> sets k2 22
(error) ERR unknown command `sets`, with args beginning with: `k2`, `22`, 
127.0.0.1:6379> exec
(error) EXECABORT Transaction discarded because of previous errors.
127.0.0.1:6379> get k1
"v1"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

- **Redis类型错误（运行时错误）**

在开启事务后，修改k1值为11，k2值为22，但将k2的类型作为List，**在运行时检测类型错误，最终导致事务提交失败，此时事务并没有回滚，而是跳过错误命令继续执行**， 结果k1值改变、k2保留原值

```bash
127.0.0.1:6379> set k1 v1
OK
127.0.0.1:6379> set k1 v2
OK
127.0.0.1:6379> MULTI
OK
127.0.0.1:6379> set k1 11
QUEUED
127.0.0.1:6379> lpush k2 22
QUEUED
127.0.0.1:6379> EXEC
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
127.0.0.1:6379> get k1
"11"
127.0.0.1:6379> get k2
"v2"
127.0.0.1:6379>
```

#### [#](#redis事务中watch是如何监视实现的呢) Redis事务中watch是如何监视实现的呢？

Redis使用WATCH命令来决定事务是继续执行还是回滚，那就需要在MULTI之前使用WATCH来监控某些键值对，然后使用MULTI命令来开启事务，执行对数据结构操作的各种命令，此时这些命令入队列。

当使用EXEC执行事务时，首先会比对WATCH所监控的键值对，如果没发生改变，它会执行事务队列中的命令，提交事务；如果发生变化，将不会执行事务中的任何命令，同时事务回滚。当然无论是否回滚，Redis都会取消执行事务前的WATCH命令。

![img](/images/db/redis/db-redis-trans-2.png)

#### [#](#为什么-redis-不支持回滚) 为什么 Redis 不支持回滚？

以下是这种做法的优点：

- Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
- 因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。

有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， **回滚并不能解决编程错误带来的问题**。 举个例子， 如果你本来想通过 INCR 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 INCR ， 回滚是没有办法处理这些情况的。

#### [#](#redis-对-acid的支持性理解) Redis 对 ACID的支持性理解？

- **原子性atomicity**

首先通过上文知道 运行期的错误是不会回滚的，很多文章由此说Redis事务违背原子性的；而官方文档认为是遵从原子性的。

Redis官方文档给的理解是，**Redis的事务是原子性的：所有的命令，要么全部执行，要么全部不执行**。而不是完全成功。

- **一致性consistency**

redis事务可以保证命令失败的情况下得以回滚，数据能恢复到没有执行之前的样子，是保证一致性的，除非redis进程意外终结。

- **隔离性Isolation**

redis事务是严格遵守隔离性的，原因是redis是单进程单线程模式(v6.0之前），可以保证命令执行过程中不会被其他客户端命令打断。

但是，Redis不像其它结构化数据库有隔离级别这种设计。

- **持久性Durability**

**redis事务是不保证持久性的**，这是因为redis持久化策略中不管是RDB还是AOF都是异步执行的，不保证持久性是出于对性能的考虑。

#### [#](#redis事务其他实现) Redis事务其他实现？

基于Lua脚本，Redis可以保证脚本内的命令一次性、按顺序地执行，其同时也不提供事务运行错误的回滚，执行过程中如果部分命令运行错误，剩下的命令还是会继续运行完

基于中间标记变量，通过另外的标记变量来标识事务是否执行完成，读取数据时先读取该标记变量判断是否事务执行完成。但这样会需要额外写代码实现，比较繁琐

#### [#](#redis集群的主从复制模型是怎样的) Redis集群的主从复制模型是怎样的？

主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点(master)，后者称为从节点(slave)；数据的复制是单向的，只能由主节点到从节点。

**主从复制的作用**主要包括：

- **数据冗余**：主从复制实现了数据的热备份，是持久化之外的一种数据冗余方式。
- **故障恢复**：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。
- **负载均衡**：在主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点），分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。
- **高可用基石**：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制是Redis高可用的基础。

主从库之间采用的是**读写分离**的方式。

- 读操作：主库、从库都可以接收；
- 写操作：首先到主库执行，然后，主库将写操作同步给从库。

![img](/images/db/redis/db-redis-copy-1.png)

注意：在2.8版本之前只有全量复制，而2.8版本后有全量和增量复制：

- **全量（同步）复制**：比如第一次同步时
- **增量（同步）复制**：只会把主从库网络断连期间主库收到的命令，同步给从库

#### [#](#redis-全量复制的三个阶段) Redis 全量复制的三个阶段？

![img](/images/db/redis/db-redis-copy-2.jpg)

**第一阶段是主从库间建立连接、协商同步的过程**，主要是为全量复制做准备。在这一步，从库和主库建立起连接，并告诉主库即将进行同步，主库确认回复后，主从库间就可以开始同步了。

具体来说，从库给主库发送 psync 命令，表示要进行数据同步，主库根据这个命令的参数来启动复制。psync 命令包含了主库的 runID 和复制进度 offset 两个参数。runID，是每个 Redis 实例启动时都会自动生成的一个随机 ID，用来唯一标记这个实例。当从库和主库第一次复制时，因为不知道主库的 runID，所以将 runID 设为“？”。offset，此时设为 -1，表示第一次复制。主库收到 psync 命令后，会用 FULLRESYNC 响应命令带上两个参数：主库 runID 和主库目前的复制进度 offset，返回给从库。从库收到响应后，会记录下这两个参数。这里有个地方需要注意，FULLRESYNC 响应表示第一次复制采用的全量复制，也就是说，主库会把当前所有的数据都复制给从库。

**第二阶段，主库将所有数据同步给从库**。从库收到数据后，在本地完成数据加载。这个过程依赖于内存快照生成的 RDB 文件。

具体来说，主库执行 bgsave 命令，生成 RDB 文件，接着将文件发给从库。从库接收到 RDB 文件后，会先清空当前数据库，然后加载 RDB 文件。这是因为从库在通过 replicaof 命令开始和主库同步前，可能保存了其他数据。为了避免之前数据的影响，从库需要先把当前数据库清空。在主库将数据同步给从库的过程中，主库不会被阻塞，仍然可以正常接收请求。否则，Redis 的服务就被中断了。但是，这些请求中的写操作并没有记录到刚刚生成的 RDB 文件中。为了保证主从库的数据一致性，主库会在内存中用专门的 replication buffer，记录 RDB 文件生成后收到的所有写操作。

**第三个阶段，主库会把第二阶段执行过程中新收到的写命令，再发送给从库**。具体的操作是，当主库完成 RDB 文件发送后，就会把此时 replication buffer 中的修改操作发给从库，从库再重新执行这些操作。这样一来，主从库就实现同步了。

#### [#](#redis-为什么会设计增量复制) Redis 为什么会设计增量复制？

如果主从库在命令传播时出现了网络闪断，那么，从库就会和主库重新进行一次全量复制，开销非常大。从 Redis 2.8 开始，网络断了之后，主从库会采用增量复制的方式继续同步。

#### [#](#redis-增量复制的流程) Redis 增量复制的流程？

![img](/images/db/redis/db-redis-copy-3.jpg)

先看两个概念： `replication buffer` 和 `repl_backlog_buffer`

`repl_backlog_buffer`：它是为了从库断开之后，如何找到主从差异数据而设计的环形缓冲区，从而避免全量复制带来的性能开销。如果从库断开时间太久，repl_backlog_buffer环形缓冲区被主库的写命令覆盖了，那么从库连上主库后只能乖乖地进行一次全量复制，所以**repl_backlog_buffer配置尽量大一些，可以降低主从断开后全量复制的概率**。而在repl_backlog_buffer中找主从差异的数据后，如何发给从库呢？这就用到了replication buffer。

`replication buffer`：Redis和客户端通信也好，和从库通信也好，Redis都需要给分配一个 内存buffer进行数据交互，客户端是一个client，从库也是一个client，我们每个client连上Redis后，Redis都会分配一个client buffer，所有数据交互都是通过这个buffer进行的：Redis先把数据写到这个buffer中，然后再把buffer中的数据发到client socket中再通过网络发送出去，这样就完成了数据交互。所以主从在增量同步时，从库作为一个client，也会分配一个buffer，只不过这个buffer专门用来传播用户的写命令到从库，保证主从数据一致，我们通常把它叫做replication buffer。

#### [#](#增量复制如果在网络断开期间-repl-backlog-size环形缓冲区写满之后-从库是会丢失掉那部分被覆盖掉的数据-还是直接进行全量复制呢) 增量复制如果在网络断开期间，repl_backlog_size环形缓冲区写满之后，从库是会丢失掉那部分被覆盖掉的数据，还是直接进行全量复制呢？

对于这个问题来说，有两个关键点：

1. 一个从库如果和主库断连时间过长，造成它在主库repl_backlog_buffer的slave_repl_offset位置上的数据已经被覆盖掉了，此时从库和主库间将进行全量复制。
2. 每个从库会记录自己的slave_repl_offset，每个从库的复制进度也不一定相同。在和主库重连进行恢复时，从库会通过psync命令把自己记录的slave_repl_offset发给主库，主库会根据从库各自的复制进度，来决定这个从库可以进行增量复制，还是全量复制。

#### [#](#redis-为什么不持久化的主服务器自动重启非常危险呢) Redis 为什么不持久化的主服务器自动重启非常危险呢?

- 我们设置节点A为主服务器，关闭持久化，节点B和C从节点A复制数据。
- 这时出现了一个崩溃，但Redis具有自动重启系统，重启了进程，因为关闭了持久化，节点重启后只有一个空的数据集。
- 节点B和C从节点A进行复制，现在节点A是空的，所以节点B和C上的复制数据也会被删除。
- 当在高可用系统中使用Redis Sentinel，关闭了主服务器的持久化，并且允许自动重启，这种情况是很危险的。比如主服务器可能在很短的时间就完成了重启，以至于Sentinel都无法检测到这次失败，那么上面说的这种失败的情况就发生了。

如果数据比较重要，并且在使用主从复制时关闭了主服务器持久化功能的场景中，都应该禁止实例自动重启。

#### [#](#redis-为什么主从全量复制使用rdb而不使用aof) Redis 为什么主从全量复制使用RDB而不使用AOF？

1、RDB文件内容是经过压缩的二进制数据（不同数据类型数据做了针对性优化），文件很小。而AOF文件记录的是每一次写操作的命令，写操作越多文件会变得很大，其中还包括很多对同一个key的多次冗余操作。在主从全量数据同步时，传输RDB文件可以尽量降低对主库机器网络带宽的消耗，从库在加载RDB文件时，一是文件小，读取整个文件的速度会很快，二是因为RDB文件存储的都是二进制数据，从库直接按照RDB协议解析还原数据即可，速度会非常快，而AOF需要依次重放每个写命令，这个过程会经历冗长的处理逻辑，恢复速度相比RDB会慢得多，所以使用RDB进行主从全量复制的成本最低。

2、假设要使用AOF做全量复制，意味着必须打开AOF功能，打开AOF就要选择文件刷盘的策略，选择不当会严重影响Redis性能。而RDB只有在需要定时备份和主从全量复制数据时才会触发生成一次快照。而在很多丢失数据不敏感的业务场景，其实是不需要开启AOF的。

#### [#](#redis-为什么还有无磁盘复制模式) Redis 为什么还有无磁盘复制模式？

Redis 默认是磁盘复制，但是**如果使用比较低速的磁盘，这种操作会给主服务器带来较大的压力**。Redis从2.8.18版本开始尝试支持无磁盘的复制。使用这种设置时，子进程直接将RDB通过网络发送给从服务器，不使用磁盘作为中间存储。

**无磁盘复制模式**：master创建一个新进程直接dump RDB到slave的socket，不经过主进程，不经过硬盘。适用于disk较慢，并且网络较快的时候。

使用`repl-diskless-sync`配置参数来启动无磁盘复制。

使用`repl-diskless-sync-delay` 参数来配置传输开始的延迟时间；master等待一个`repl-diskless-sync-delay`的秒数，如果没slave来的话，就直接传，后来的得排队等了; 否则就可以一起传。

#### [#](#redis-为什么还会有从库的从库的设计) Redis 为什么还会有从库的从库的设计？

通过分析主从库间第一次数据同步的过程，你可以看到，一次全量复制中，对于主库来说，需要完成两个耗时的操作：**生成 RDB 文件和传输 RDB 文件**。

如果从库数量很多，而且都要和主库进行全量复制的话，就会导致主库忙于 fork 子进程生成 RDB 文件，进行数据全量复制。fork 这个操作会阻塞主线程处理正常请求，从而导致主库响应应用程序的请求速度变慢。此外，传输 RDB 文件也会占用主库的网络带宽，同样会给主库的资源使用带来压力。那么，有没有好的解决方法可以分担主库压力呢？

其实是有的，这就是“主 - 从 - 从”模式。

在刚才介绍的主从库模式中，所有的从库都是和主库连接，所有的全量复制也都是和主库进行的。现在，我们可以通过“主 - 从 - 从”模式**将主库生成 RDB 和传输 RDB 的压力，以级联的方式分散到从库上**。

简单来说，我们在部署主从集群的时候，可以手动选择一个从库（比如选择内存资源配置较高的从库），用于级联其他的从库。然后，我们可以再选择一些从库（例如三分之一的从库），在这些从库上执行如下命令，让它们和刚才所选的从库，建立起主从关系。

```bash
replicaof 所选从库的IP 6379
```

这样一来，这些从库就会知道，在进行同步时，不用再和主库进行交互了，只要和级联的从库进行写操作同步就行了，这就可以减轻主库上的压力，如下图所示：

![img](/images/db/redis/db-redis-copy-4.jpg)

级联的“主-从-从”模式好了，到这里，我们了解了主从库间通过全量复制实现数据同步的过程，以及通过“主 - 从 - 从”模式分担主库压力的方式。那么，一旦主从库完成了全量复制，它们之间就会一直维护一个网络连接，主库会通过这个连接将后续陆续收到的命令操作再同步给从库，这个过程也称为基于长连接的命令传播，可以避免频繁建立连接的开销。

#### [#](#redis哨兵机制-哨兵实现了什么功能呢) Redis哨兵机制？哨兵实现了什么功能呢?

哨兵的核心功能是主节点的自动故障转移。

下图是一个典型的哨兵集群监控的逻辑图：

![img](/images/db/redis/db-redis-sen-1.png)

哨兵实现了什么功能呢？下面是Redis官方文档的描述：

- **监控（Monitoring）**：哨兵会不断地检查主节点和从节点是否运作正常。
- **自动故障转移（Automatic failover）**：当主节点不能正常工作时，哨兵会开始自动故障转移操作，它会将失效主节点的其中一个从节点升级为新的主节点，并让其他从节点改为复制新的主节点。
- **配置提供者（Configuration provider）**：客户端在初始化时，通过连接哨兵来获得当前Redis服务的主节点地址。
- **通知（Notification）**：哨兵可以将故障转移的结果发送给客户端。

其中，监控和自动故障转移功能，使得哨兵可以及时发现主节点故障并完成转移；而配置提供者和通知功能，则需要在与客户端的交互中才能体现。

#### [#](#redis-哨兵集群是通过什么方式组建的) Redis 哨兵集群是通过什么方式组建的？

哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制，也就是发布 / 订阅机制。

在主从集群中，主库上有一个名为`__sentinel__:hello`的频道，不同哨兵就是通过它来相互发现，实现互相通信的。在下图中，哨兵 1 把自己的 IP（172.16.19.3）和端口（26579）发布到`__sentinel__:hello`频道上，哨兵 2 和 3 订阅了该频道。那么此时，哨兵 2 和 3 就可以从这个频道直接获取哨兵 1 的 IP 地址和端口号。然后，哨兵 2、3 可以和哨兵 1 建立网络连接。

![img](/images/db/redis/db-redis-sen-6.jpg)

通过这个方式，哨兵 2 和 3 也可以建立网络连接，这样一来，哨兵集群就形成了。它们相互间可以通过网络连接进行通信，比如说对主库有没有下线这件事儿进行判断和协商。

#### [#](#redis-哨兵是如何监控redis集群的) Redis 哨兵是如何监控Redis集群的？

这是由哨兵向主库发送 INFO 命令来完成的。就像下图所示，哨兵 2 给主库发送 INFO 命令，主库接受到这个命令后，就会把从库列表返回给哨兵。接着，哨兵就可以根据从库列表中的连接信息，和每个从库建立连接，并在这个连接上持续地对从库进行监控。哨兵 1 和 3 可以通过相同的方法和从库建立连接。

![img](/images/db/redis/db-redis-sen-7.jpg)

#### [#](#redis-哨兵如何判断主库已经下线了呢) Redis 哨兵如何判断主库已经下线了呢？

首先要理解两个概念：**主观下线**和**客观下线**

- **主观下线**：任何一个哨兵都是可以监控探测，并作出Redis节点下线的判断；
- **客观下线**：有哨兵集群共同决定Redis节点是否下线；

当某个哨兵（如下图中的哨兵2）判断主库“主观下线”后，就会给其他哨兵发送 `is-master-down-by-addr` 命令。接着，其他哨兵会根据自己和主库的连接情况，做出 Y 或 N 的响应，Y 相当于赞成票，N 相当于反对票。

![img](/images/db/redis/db-redis-sen-2.jpg)

如果赞成票数（这里是2）是大于等于哨兵配置文件中的 `quorum` 配置项（比如这里如果是quorum=2）, 则可以判定**主库客观下线**了。

#### [#](#redis-哨兵的选举机制是什么样的) Redis 哨兵的选举机制是什么样的？

- **为什么必然会出现选举/共识机制**？

为了避免哨兵的单点情况发生，所以需要一个哨兵的分布式集群。作为分布式集群，必然涉及共识问题（即选举问题）；同时故障的转移和通知都只需要一个主的哨兵节点就可以了。

- **哨兵的选举机制是什么样的**？

哨兵的选举机制其实很简单，就是一个Raft选举算法： **选举的票数大于等于num(sentinels)/2+1时，将成为领导者，如果没有超过，继续选举**

Raft算法你可以参看这篇文章[分布式算法 - Raft算法]()

- 任何一个想成为 Leader 的哨兵，要满足两个条件

  ： 

  - 第一，拿到半数以上的赞成票；
  - 第二，拿到的票数同时还需要大于等于哨兵配置文件中的 quorum 值。

以 3 个哨兵为例，假设此时的 quorum 设置为 2，那么，任何一个想成为 Leader 的哨兵只要拿到 2 张赞成票，就可以了。

#### [#](#redis-1主4从-5个哨兵-哨兵配置quorum为2-如果3个哨兵故障-当主库宕机时-哨兵能否判断主库-客观下线-能否自动切换) Redis 1主4从，5个哨兵，哨兵配置quorum为2，如果3个哨兵故障，当主库宕机时，哨兵能否判断主库“客观下线”？能否自动切换？

经过实际测试：

1、哨兵集群可以判定主库“主观下线”。由于quorum=2，所以当一个哨兵判断主库“主观下线”后，询问另外一个哨兵后也会得到同样的结果，2个哨兵都判定“主观下线”，达到了quorum的值，因此，**哨兵集群可以判定主库为“客观下线”**。

2、**但哨兵不能完成主从切换**。哨兵标记主库“客观下线后”，在选举“哨兵领导者”时，一个哨兵必须拿到超过多数的选票(5/2+1=3票)。但目前只有2个哨兵活着，无论怎么投票，一个哨兵最多只能拿到2票，永远无法达到`N/2+1`选票的结果。

#### [#](#主库判定客观下线了-那么如何从剩余的从库中选择一个新的主库呢) 主库判定客观下线了，那么如何从剩余的从库中选择一个新的主库呢？

- 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点
- 选择`salve-priority`从节点优先级最高（redis.conf）的
- 选择复制偏移量最大，只复制最完整的从节点

![img](/images/db/redis/db-redis-sen-3.jpg)

#### [#](#新的主库选择出来后-如何进行故障的转移) 新的主库选择出来后，如何进行故障的转移？

假设根据我们一开始的图：（我们假设：判断主库客观下线了，同时选出`sentinel 3`是哨兵leader）

![img](/images/db/redis/db-redis-sen-1.png)

**故障转移流程如下**：

![img](/images/db/redis/db-redis-sen-4.png)

- 将slave-1脱离原从节点（PS: 5.0 中应该是`replicaof no one`)，升级主节点，
- 将从节点slave-2指向新的主节点
- 通知客户端主节点已更换
- 将原主节点（oldMaster）变成从节点，指向新的主节点

**转移之后**

![img](/images/db/redis/db-redis-sen-5.png)

#### [#](#redis事件机制) Redis事件机制?

Redis中的事件驱动库只关注网络IO，以及定时器。该事件库处理下面两类事件：

- **文件事件**(file event)：用于处理 Redis 服务器和客户端之间的网络IO。
- **时间事件**(time eveat)：Redis 服务器中的一些操作（比如serverCron函数）需要在给定的时间点执行，而时间事件就是处理这类定时操作的。

事件驱动库的代码主要是在`src/ae.c`中实现的，其示意图如下所示。

![img](/images/db/redis/db-redis-event-1.png)

`aeEventLoop`是整个事件驱动的核心，它管理着文件事件表和时间事件列表，不断地循环处理着就绪的文件事件和到期的时间事件。

#### [#](#redis文件事件的模型) Redis文件事件的模型？

Redis基于**Reactor模式**开发了自己的网络事件处理器，也就是文件事件处理器。文件事件处理器使用**IO多路复用技术**，同时监听多个套接字，并为套接字关联不同的事件处理函数。当套接字的可读或者可写事件触发时，就会调用相应的事件处理函数。

- **1. 为什么单线程的 Redis 能那么快**？

Redis的瓶颈主要在IO而不是CPU，所以为了省开发量，在6.0版本前是单线程模型；其次，Redis 是单线程主要是指 **Redis 的网络 IO 和键值对读写是由一个线程来完成的**，这也是 Redis 对外提供键值存储服务的主要流程。（但 Redis 的其他功能，比如持久化、异步删除、集群数据同步等，其实是由额外的线程执行的）。

Redis 采用了多路复用机制使其在网络 IO 操作中能并发处理大量的客户端请求，实现高吞吐率。

- **2. Redis事件响应框架ae_event及文件事件处理器**

Redis并没有使用 libevent 或者 libev 这样的成熟开源方案，而是自己实现一个非常简洁的事件驱动库 ae_event。@pdai

Redis 使用的IO多路复用技术主要有：`select`、`epoll`、`evport`和`kqueue`等。每个IO多路复用函数库在 Redis 源码中都对应一个单独的文件，比如`ae_select.c`，`ae_epoll.c`， `ae_kqueue.c`等。Redis 会根据不同的操作系统，按照不同的优先级选择多路复用技术。事件响应框架一般都采用该架构，比如 netty 和 libevent。

![img](/images/db/redis/db-redis-event-2.png)

如下图所示，文件事件处理器有四个组成部分，它们分别是套接字、I/O多路复用程序、文件事件分派器以及事件处理器。

![img](/images/db/redis/db-redis-event-3.png)

文件事件是对套接字操作的抽象，每当一个套接字准备好执行 `accept`、`read`、`write`和 `close` 等操作时，就会产生一个文件事件。因为 Redis 通常会连接多个套接字，所以多个文件事件有可能并发的出现。

I/O多路复用程序负责监听多个套接字，并向文件事件派发器传递那些产生了事件的套接字。

尽管多个文件事件可能会并发地出现，但I/O多路复用程序总是会将所有产生的套接字都放到同一个队列(也就是后文中描述的aeEventLoop的fired就绪事件表)里边，然后文件事件处理器会以有序、同步、单个套接字的方式处理该队列中的套接字，也就是处理就绪的文件事件。

![img](/images/db/redis/db-redis-event-4.png)

#### [#](#什么是redis发布订阅) 什么是Redis发布订阅？

Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

Redis 的 SUBSCRIBE 命令可以让客户端订阅任意数量的频道， 每当有新信息发送到被订阅的频道时， 信息就会被发送给所有订阅指定频道的客户端。

作为例子， 下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![img](/images/db/redis/db-redis-sub-1.svg)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![img](/images/db/redis/db-redis-sub-2.svg)

#### [#](#redis发布订阅有哪两种方式) Redis发布订阅有哪两种方式？

- **基于频道(Channel)的发布/订阅**

"发布/订阅"模式包含两种角色，分别是发布者和订阅者。发布者可以向指定的频道(channel)发送消息; 订阅者可以订阅一个或者多个频道(channel),所有订阅此频道的订阅者都会收到此消息。

![img](/images/db/redis/db-redis-sub-8.png)

- **基于模式(pattern)的发布/订阅**

下图展示了一个带有频道和模式的例子， 其中 tweet.shop.* 模式匹配了 tweet.shop.kindle 频道和 tweet.shop.ipad 频道， 并且有不同的客户端分别订阅它们三个：

![img](/images/db/redis/db-redis-sub-5.svg)

当有信息发送到 tweet.shop.kindle 频道时， 信息除了发送给 clientX 和 clientY 之外， 还会发送给订阅 tweet.shop.* 模式的 client123 和 client256 ：

![img](/images/db/redis/db-redis-sub-6.svg)

另一方面， 如果接收到信息的是频道 tweet.shop.ipad ， 那么 client123 和 client256 同样会收到信息：

![img](/images/db/redis/db-redis-sub-7.svg)

#### [#](#什么是redis-cluster) 什么是Redis Cluster？

Redis-cluster是一种服务器Sharding技术，Redis3.0以后版本正式提供支持。

#### [#](#说说redis哈希槽的概念-为什么是16384个) 说说Redis哈希槽的概念？为什么是16384个？

Redis-cluster没有使用一致性hash，而是引入了**哈希槽**的概念。Redis-cluster中有16384(即2的14次方）个哈希槽，每个key通过CRC16校验后对16383取模来决定放置哪个槽。Cluster中的每个节点负责一部分hash槽（hash slot）。

比如集群中存在三个节点，则可能存在的一种分配如下：

1. 节点A包含0到5500号哈希槽；
2. 节点B包含5501到11000号哈希槽；
3. 节点C包含11001 到 16384号哈希槽。

- **为什么是16384个**

在redis节点发送心跳包时需要把所有的槽放到这个心跳包里，以便让节点知道当前集群信息，16384=16k，在发送心跳包时使用char进行bitmap压缩后是2k（2 * 8 (8 bit) * 1024(1k) = 16K），也就是说使用2k的空间创建了16k的槽数。

虽然使用CRC16算法最多可以分配65535（2^16-1）个槽位，65535=65k，压缩后就是8k（8 * 8 (8 bit) * 1024(1k) =65K），也就是说需要需要8k的心跳包，作者认为这样做不太值得；并且一般情况下一个redis集群不会有超过1000个master节点，所以16k的槽位是个比较合适的选择。

#### [#](#redis集群会有写操作丢失吗-为什么) Redis集群会有写操作丢失吗？为什么？

Redis并不能保证数据的强一致性，这意味这在实际中集群在特定的条件下可能会丢失写操作。

#### [#](#redis-客户端有哪些) Redis 客户端有哪些？

Redisson、Jedis、lettuce等等，官方推荐使用Redisson。

Redisson是一个高级的分布式协调Redis客服端，能帮助用户在分布式环境中轻松实现一些Java的对象 (Bloom filter, BitSet, Set, SetMultimap, ScoredSortedSet, SortedSet, Map, ConcurrentMap, List, ListMultimap, Queue, BlockingQueue, Deque, BlockingDeque, Semaphore, Lock, ReadWriteLock, AtomicLong, CountDownLatch, Publish / Subscribe, HyperLogLog)。

#### [#](#redis如何做大量数据插入) Redis如何做大量数据插入？

Redis2.6开始redis-cli支持一种新的被称之为pipe mode的新模式用于执行大量数据插入工作。

#### [#](#redis实现分布式锁实现-什么是-redlock) Redis实现分布式锁实现? 什么是 RedLock?

- **常规**

加锁： SET NX PX + 校验唯一随机值

解锁： Lua脚本

- **RedLock**

搞多个Redis master部署，以保证它们不会同时宕掉。并且这些master节点是完全相互独立的，相互之间不存在数据同步。同时，需要确保在这多个master实例上，是与在Redis单实例，使用相同方法来获取和释放锁。

- **Redisson框架**

Redisson watchdog或者它实现了RedLock方式

具体可以看后文分布式锁中实现方式。

#### [#](#redis缓存有哪些问题-如何解决) Redis缓存有哪些问题，如何解决？

当缓存库出现时，必须要考虑如下问题：

- **缓存穿透**
  - **问题来源**: 缓存穿透是指**缓存和数据库中都没有的数据**，而用户不断发起请求。由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。
  - 解决方案
    - 接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
    - 从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击
    - 布隆过滤器。bloomfilter就类似于一个hash set，用于快速判某个元素是否存在于集合中，其典型的应用场景就是快速判断一个key是否存在于某容器，不存在就直接返回。布隆过滤器的关键就在于hash算法和容器大小
- **缓存穿击**
  - **问题来源**: 缓存击穿是指**缓存中没有但数据库中有的数据**（一般是缓存时间到期），这时由于并发用户特别多，同时读缓存没读到数据，又同时去数据库去取数据，引起数据库压力瞬间增大，造成过大压力。
  - 解决方案
    - 设置热点数据永远不过期。
    - 接口限流与熔断，降级。重要的接口一定要做好限流策略，防止用户恶意刷接口，同时要降级准备，当接口中的某些 服务 不可用时候，进行熔断，失败快速返回机制。
    - 加互斥锁
- **缓存雪崩**
  - **问题来源**: 缓存雪崩是指缓存中**数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机**。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
  - 解决方案
    - 缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
    - 如果缓存数据库是分布式部署，将热点数据均匀分布在不同的缓存数据库中。
    - 设置热点数据永远不过期。
- **缓存污染**（或者满了）

缓存污染问题说的是缓存中一些只会被访问一次或者几次的的数据，被访问完后，再也不会被访问到，但这部分数据依然留存在缓存中，消耗缓存空间。

缓存污染会随着数据的持续增加而逐渐显露，随着服务的不断运行，缓存中会存在大量的永远不会再次被访问的数据。缓存空间是有限的，如果缓存空间满了，再往缓存里写数据时就会有额外开销，影响Redis性能。这部分额外开销主要是指写的时候判断淘汰策略，根据淘汰策略去选择要淘汰的数据，然后进行删除操作。

#### [#](#redis性能问题有哪些-如何分析定位解决) Redis性能问题有哪些，如何分析定位解决?

举几个例子

- **看延迟** 60 秒内的最大响应延迟：

```bash
$ redis-cli -h 127.0.0.1 -p 6379 --intrinsic-latency 60
Max latency so far: 1 microseconds.
Max latency so far: 15 microseconds.
Max latency so far: 17 microseconds.
Max latency so far: 18 microseconds.
Max latency so far: 31 microseconds.
Max latency so far: 32 microseconds.
Max latency so far: 59 microseconds.
Max latency so far: 72 microseconds.
 
1428669267 total runs (avg latency: 0.0420 microseconds / 42.00 nanoseconds per run).
Worst run took 1429x longer than the average latency.
```

- **慢日志**（slowlog）

慢查询，就会导致后面的请求发生排队，对于客户端来说，响应延迟也会变长。

![img](/images/db/redis/redis-performance-2.jpeg)

- **bigkey**

大对象

- **集中过期**

一般有两种方案来规避这个问题：

1. 集中过期 key 增加一个随机过期时间，把集中过期的时间打散，降低 Redis 清理过期 key 的压力
2. 如果你使用的 Redis 是 4.0 以上版本，可以开启 lazy-free 机制，当删除过期 key 时，把释放内存的操作放到后台线程中执行，避免阻塞主线程

- **fork耗时严重**

主进程创建子进程，会调用操作系统提供的 fork 函数

- **使用Swap**

当内存中的数据被换到磁盘上后，Redis 再访问这些数据时，就需要从磁盘上读取，访问磁盘的速度要比访问内存慢几百倍！

- **内存碎片**

Redis 4.0 版本，它正好提供了自动碎片整理的功能，可以通过配置开启碎片自动整理

#### [#](#redis单线程模型-在6-0之前如何提高多核cpu的利用率) Redis单线程模型？ 在6.0之前如何提高多核CPU的利用率？

可以在同一个服务器部署多个Redis的实例，并把他们当作不同的服务器来使用，在某些时候，无论如何一个服务器是不够的， 所以，如果你想使用多个CPU，你可以考虑一下分片（shard）。

#### [#](#redis6-0之前的版本真的是单线程的吗) Redis6.0之前的版本真的是单线程的吗?

Redis在处理客户端请求时,包括获取(socket读)、解析、执行、内容返回(socket写)等都是由一个顺序串行的主线程执行的,这就是所谓的 单线程.单如果严格讲,从Redis4.0之后并不是单线程,除了主线程之外,它也有后台线程在处理一些较为缓慢的操作,例如 清理脏数据, 无用链接的释放, 大key的删除, 数据持久化bgsave,bgrewriteaof等,都是在主线程之外的子线程单独执行的.

#### [#](#redis6-0之前为什么一致不用多线程) Redis6.0之前为什么一致不用多线程?

官方曾做过类似问题的回复：使用Redis时，几乎不存在CPU成为瓶颈的情况， Redis主要受限于内存和网络。例如在一个普通的Linux系统上，Redis通过使用pipelining每秒可以处理100万个请求，所以如果应用程序主要使用O(N)或O(log(N))的命令，它几乎不会占用太多CPU。

使用了单线程后，可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。Redis通过AE事件模型以及IO多路复用等技术，处理性能非常高，因此没有必要使用多线程。单线程机制使得 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等 “线程不安全” 的命令都可以无锁进行。

#### [#](#redis6-0为什么要引入多线程呢) Redis6.0为什么要引入多线程呢？

Redis将所有数据放在内存中，内存的响应时长大约为100纳秒，对于小数据包，Redis服务器可以处理80,000到100,000 QPS，这也是Redis处理的极限了，对于80%的公司来说，单线程的Redis已经足够使用了。

但随着越来越复杂的业务场景，有些公司动不动就上亿的交易量，因此需要更大的QPS。常见的解决方案是在分布式架构中对数据进行分区并采用多个服务器，但该方案有非常大的缺点，例如要管理的Redis服务器太多，维护代价大；某些适用于单个Redis服务器的命令不适用于数据分区；数据分区无法解决热点读/写问题；数据偏斜，重新分配和放大/缩小变得更加复杂等等。

从Redis自身角度来说，因为读写网络的read/write系统调用占用了Redis执行期间大部分CPU时间，瓶颈主要在于网络的 IO 消耗, 优化主要有两个方向:

- 提高网络 IO 性能，典型的实现比如使用 DPDK 来替代内核网络栈的方式
- 使用多线程充分利用多核，典型的实现比如 Memcached

协议栈优化的这种方式跟 Redis 关系不大，支持多线程是一种最有效最便捷的操作方式。所以总结起来，redis支持多线程主要就是两个原因：

- 可以充分利用服务器 CPU 资源，目前主线程只能利用一个核
- 多线程任务可以分摊 Redis 同步 IO 读写负荷

#### [#](#redis6-0默认是否开启了多线程) Redis6.0默认是否开启了多线程？

Redis6.0的多线程默认是禁用的，只使用主线程。如需开启需要修改redis.conf配置文件：io-threads-do-reads yes

#### [#](#redis6-0多线程开启时-线程数如何设置) Redis6.0多线程开启时，线程数如何设置？

开启多线程后，还需要设置线程数，否则是不生效的。同样修改redis.conf配置文件 io-threads4

关于线程数的设置，官方有一个建议：4核的机器建议设置为2或3个线程，8核的建议设置为6个线程，线程数一定要小于机器核数。还需要注意的是，线程数并不是越大越好，官方认为超过了8个基本就没什么意义了。

#### [#](#redis6-0多线程的实现机制) Redis6.0多线程的实现机制？

核心思路是，将主线程的IO读写任务拆分出来给一组独立的线程执行，使得多个 socket 的读写可以并行化

- 主线程负责接收建立连接的请求,获取socket放到全局等待处理队列
- 主线程处理完读事件之后,通过Round Robin将这些连接分配给IO线程(并不会等待队列满)
- 主线程阻塞等待IO线程读取socket完毕
- 主线程通过单线程的方式执行请求命令，请求数据读取并解析完成，但并不执行
- 主线程阻塞等待IO线程将数据回写socket完毕
- 解除绑定,清空等待队列

该线程有如下特点:

- IO线程要么同时在读socket，要么同时在写，不会同时读或写
- IO线程只负责读写socket解析命令，不负责命令处理（主线程串行执行命令）

#### [#](#开启多线程后-是否会存在线程并发安全问题) 开启多线程后，是否会存在线程并发安全问题？

Redis的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行,因此不存在线程的并发安全问题

### [#](#_8-4-mongodb) 8.4 MongoDB

#### [#](#什么是mongodb-为什么使用mongodb) 什么是MongoDB？为什么使用MongoDB？

MongoDB是面向文档的NoSQL数据库，用于大量数据存储。MongoDB是一个在2000年代中期问世的数据库。属于NoSQL数据库的类别。以下是一些为什么应该开始使用MongoDB的原因

- **面向文档的**–由于MongoDB是NoSQL类型的数据库，它不是以关系类型的格式存储数据，而是将数据存储在文档中。这使得MongoDB非常灵活，可以适应实际的业务环境和需求。
- **临时查询**-MongoDB支持按字段，范围查询和正则表达式搜索。可以查询返回文档中的特定字段。
- **索引**-可以创建索引以提高MongoDB中的搜索性能。MongoDB文档中的任何字段都可以建立索引。
- **复制**-MongoDB可以提供副本集的高可用性。副本集由两个或多个mongo数据库实例组成。每个副本集成员可以随时充当主副本或辅助副本的角色。主副本是与客户端交互并执行所有读/写操作的主服务器。辅助副本使用内置复制维护主数据的副本。当主副本发生故障时，副本集将自动切换到辅助副本，然后它将成为主服务器。
- **负载平衡**-MongoDB使用分片的概念，通过在多个MongoDB实例之间拆分数据来水平扩展。MongoDB可以在多台服务器上运行，以平衡负载或复制数据，以便在硬件出现故障时保持系统正常运行。

#### [#](#mongodb与rdbms区别-有哪些术语) MongoDB与RDBMS区别？有哪些术语？

下表将帮助您更容易理解Mongo中的一些概念：

| SQL术语/概念 | MongoDB术语/概念 | 解释/说明                           |
| ------------ | ---------------- | ----------------------------------- |
| database     | database         | 数据库                              |
| table        | collection       | 数据库表/集合                       |
| row          | document         | 数据记录行/文档                     |
| column       | field            | 数据字段/域                         |
| index        | index            | 索引                                |
| table joins  |                  | 表连接,MongoDB不支持                |
| primary key  | primary key      | 主键,MongoDB自动将_id字段设置为主键 |

![img](/images/db/mongo/mongo-y-arch-2.png)

#### [#](#mongodb聚合的管道方式) MongoDB聚合的管道方式？

Aggregation Pipline： 类似于将SQL中的group by + order by + left join ... 等操作管道化。MongoDB的聚合管道（Pipline）将MongoDB文档在一个阶段（Stage）处理完毕后将结果传递给下一个阶段（Stage）处理。**阶段（Stage）操作是可以重复的**。

![img](/images/db/mongo/mongo-x-usage-11.png)

#### [#](#mongodb聚合的map-reduce方式) MongoDB聚合的Map Reduce方式？

![img](/images/db/mongo/mongo-x-usage-12.png)

#### [#](#spring-data-和mongodb集成) Spring Data 和MongoDB集成？

![img](/images/db/mongo/mongo-x-usage-spring-5.png)

- 引入`mongodb-driver`, 使用最原生的方式通过Java调用mongodb提供的Java driver;

- 引入

  ```
  spring-data-mongo
  ```

  , 自行配置使用spring data 提供的对MongoDB的封装 

  - 使用`MongoTemplate` 的方式
  - 使用`MongoRespository` 的方式

- 引入`spring-data-mongo-starter`, 采用spring autoconfig机制自动装配，然后再使用`MongoTemplate`或者`MongoRespository`方式。

#### [#](#mongodb-有哪几种存储引擎) MongoDB 有哪几种存储引擎？

MongoDB一共提供了三种存储引擎：WiredTiger，MMAPV1和In Memory；在MongoDB3.2之前采用的是MMAPV1; 后续版本v3.2开始默认采用WiredTiger； 且在v4.2版本中移除了MMAPV1的引擎。

#### [#](#谈谈你对mongodb-wt存储引擎的理解) 谈谈你对MongoDB WT存储引擎的理解？

从几个方面回答，比如：

- **插件式存储引擎架构**

实现了Server层和存储引擎层的解耦，可以支持多种存储引擎，如MySQL既可以支持B-Tree结构的InnoDB存储引擎，还可以支持LSM结构的RocksDB存储引擎。

- **B-Tree + Page**

![img](/images/db/mongo/mongo-y-ds-3.jpg)

上图是WiredTiger在内存里面的大概布局图，通过它我们可梳理清楚存储引擎是如何将数据加载到内存，然后如何通过相应数据结构来支持查询、插入、修改操作的。

内存里面B-Tree包含三种类型的page，即rootpage、internal page和leaf page，前两者包含指向其子页的page index指针，不包含集合中的真正数据，leaf page包含集合中的真正数据即keys/values和指向父页的home指针；

- **为什么是Page**？

数据以page为单位加载到cache、cache里面又会生成各种不同类型的page及为不同类型的page分配不同大小的内存、eviction触发机制和reconcile动作都发生在page上、page大小持续增加时会被分割成多个小page，所有这些操作都是围绕一个page来完成的。

Page的典型生命周期如下图所示：

![img](/images/db/mongo/mongo-y-page-1.png)

- **什么是CheckPoint**?

本质上来说，Checkpoint相当于一个日志，记录了上次Checkpoint后相关数据文件的变化。作用： 一是将内存里面发生修改的数据写到数据文件进行持久化保存，确保数据一致性； 二是实现数据库在某个时刻意外发生故障，再次启动时，缩短数据库的恢复时间，WiredTiger存储引擎中的Checkpoint模块就是来实现这个功能的。

一个Checkpoint包含关键信息如下图所示：

![img](/images/db/mongo/mongo-x-checkpoint-1.png)

每个checkpoint包含一个root page、三个指向磁盘具体位置上pages的列表以及磁盘上文件的大小。

- **如何理解WT事务机制**？

要了解实现先要知道它的事务的构造和使用相关的技术，WT在实现事务的时使用主要是使用了三个技术：**snapshot(事务快照)**、**MVCC (多版本并发控制)\**和\**redo log(重做日志)**，为了实现这三个技术，它还定义了一个基于这三个技术的事务对象和**全局事务管理器**。

- **如何理解WT缓存淘汰**？

eviction cache是一个LRU cache，即页面置换算法缓冲区，它对数据页采用的是**分段局部扫描和淘汰**，而不是对内存中所有的数据页做全局管理。基本思路是一个线程阶段性的去扫描各个btree，并把btree可以进行淘汰的数据页添加到一个lru queue中，当queue填满了后记录下这个过程当前的btree对象和btree的位置（这个位置是为了作为下次阶段性扫描位置）,然后对queue中的数据页按照访问热度排序，最后各个淘汰线程按照淘汰优先级淘汰queue中的数据页，整个过程是周期性重复。WT的这个evict过程涉及到多个eviction thread和hazard pointer技术。

**WT的evict过程都是以page为单位做淘汰，而不是以K/V。这一点和memcache、redis等常用的缓存LRU不太一样，因为在磁盘上数据的最小描述单位是page block，而不是记录**。

#### [#](#mongodb为什么要引入复制集-有哪些成员) MongoDB为什么要引入复制集？有哪些成员？

- **什么是复制集**？

保证数据在生产部署时的**冗余和可靠性**，通过在不同的机器上保存副本来保证数据的不会因为单点损坏而丢失。能够随时应对数据丢失、机器损坏带来的风险。换一句话来说，还能提高读取能力，用户的读取服务器和写入服务器在不同的地方，而且，由不同的服务器为不同的用户提供服务，提高整个系统的负载。

在**MongoDB中就是复制集（replica set)**： 一组复制集就是一组mongod实例掌管同一个数据集，实例可以在不同的机器上面。实例中包含一个主导，接受客户端所有的写入操作，其他都是副本实例，从主服务器上获得数据并保持同步。

![img](/images/db/mongo/mongo-z-rep-1.png)

- 基本的成员

  ? 

  - 主节点（Primary）

     包含了所有的写操作的日志。但是副本服务器集群包含有所有的主服务器数据，因此当主服务器挂掉了，就会在副本服务器上重新选取一个成为主服务器。MongoDB还细化将从节点（Primary）进行了细化 

    - **Priority0** Priority0节点的选举优先级为0，不会被选举为Primary

    - Hidden

       隐藏节点将不会收到来自应用程序的请求, 可使用Hidden节点做一些数据备份、离线计算的任务，不会影响复制集的服务 

      - **Delayed** Delayed节点必须是Hidden节点，并且其数据落后与Primary一段时间（可配置，比如1个小时）；当错误或者无效的数据写入Primary时，可通过Delayed节点的数据来恢复到之前的时间点。

  - **从节点（Seconary）** 正常情况下，复制集的Seconary会参与Primary选举（自身也可能会被选为Primary），并从Primary同步最新写入的数据，以保证与Primary存储相同的数据；增加Secondary节点可以提供复制集的读服务能力，同时提升复制集的可用性。

  - **仲裁节点（Arbiter）** Arbiter节点只参与投票，不能被选为Primary，并且不从Primary同步数据。比如你部署了一个2个节点的复制集，1个Primary，1个Secondary，任意节点宕机，复制集将不能提供服务了（无法选出Primary），这时可以给复制集添加一个Arbiter节点，即使有节点宕机，仍能选出Primary。

#### [#](#mongodb复制集常见部署架构) MongoDB复制集常见部署架构？

分别从三节点的单数据中心，和五节点的多数据中心来说：

三节点的单数据中心

- 三节点：一主两从方式
  - 一个主节点；
  - 两个从节点组成，主节点宕机时，这两个从节点都可以被选为主节点。

![img](/images/db/mongo/mongo-z-rep-5.png)

当主节点宕机后,两个从节点都会进行竞选，其中一个变为主节点，当原主节点恢复后，作为从节点加入当前的复制集群即可。

![img](/images/db/mongo/mongo-z-rep-6.png)

- 一主一从一仲裁方式
  - 一个主节点
  - 一个从节点，可以在选举中成为主节点
  - 一个仲裁节点，在选举中，只进行投票，不能成为主节点

![img](/images/db/mongo/mongo-z-rep-7.png)

当主节点宕机时，将会选择从节点成为主，主节点修复后，将其加入到现有的复制集群中即可。

![img](/images/db/mongo/mongo-z-rep-8.png)

对于具有5个成员的复制集，成员的一些可能的分布包括（相关注意事项和三个节点一致，这里仅展示分布方案）：

- 两个数据中心

  ：数据中心1的三个成员和数据中心2的两个成员。 

  - 如果数据中心1发生故障，则复制集将变为只读。
  - 如果数据中心2发生故障，则复制集将保持可写状态，因为数据中心1中的成员可以创建多数。

- 三个数据中心

  ：两个成员是数据中心1，两个成员是数据中心2，一个成员是站点数据中心3。 

  - 如果任何数据中心发生故障，复制集将保持可写状态，因为其余成员可以举行选举。

例如，以下5个成员复制集将其成员分布在三个数据中心中。

![img](/images/db/mongo/mongo-z-rep-9.png)

#### [#](#mongodb复制集是如何保证数据高可用的) MongoDB复制集是如何保证数据高可用的？

通过两方面阐述：一个是选举机制，另一个是故障转移期间的回滚。

- **如何选出Primary主节点的?**

假设复制集内**能够投票的成员**数量为N，则大多数为 N/2 + 1，当复制集内存活成员数量不足大多数时，整个复制集将**无法选举出Primary，复制集将无法提供写服务，处于只读状态**。

举例：3投票节点需要2个节点的赞成票，容忍选举失败次数为1；5投票节点需要3个节点的赞成票，容忍选举失败次数为2；通常投票节点为奇数，这样可以减少选举失败的概率。

在以下的情况将触发选举机制：

1. 往复制集中新加入节点
2. 初始化复制集时
3. 对复制集进行维护时，比如`rs.stepDown()`或者`rs.reconfig()`操作时
4. 从节点失联时，比如超时（默认是10秒）

- **故障转移期间的回滚**

当成员在故障转移后重新加入其复制集时，回滚将还原以前的主在数据库上的写操作。 **本质上就是保证数据的一致性**。

仅当主服务器接受了在主服务器降级之前辅助服务器未成功复制的写操作时，才需要回滚。 当主数据库作为辅助数据库重新加入集合时，它会还原或“回滚”其写入操作，以保持数据库与其他成员的一致性。

#### [#](#mongodb复制集如何同步数据) MongoDB复制集如何同步数据？

复制集中的数据同步是为了维护共享数据集的最新副本，包括复制集的辅助成员同步或复制其他成员的数据。 MongoDB使用两种形式的数据同步：

- 初始同步(Initial Sync)

   以使用完整的数据集填充新成员, 即

  全量同步

  - 新节点加入，无任何oplog，此时需先进性initial sync
  - initial sync开始时，会主动将_initialSyncFlag字段设置为true，正常结束后再设置为false；如果节点重启时，发现_initialSyncFlag为true，说明上次全量同步中途失败了，此时应该重新进行initial sync
  - 当用户发送resync命令时，initialSyncRequested会设置为true，此时会重新开始一次initial sync

- 复制(Replication)

   以将正在进行的更改应用于整个数据集, 即

  增量同步

  - initial sync结束后，接下来Secondary就会『不断拉取主上新产生的optlog并重放』

#### [#](#mongodb为什么要引入分片) MongoDB为什么要引入分片？

高数据量和吞吐量的数据库应用会对单机的性能造成较大压力, 大的查询量会将单机的CPU耗尽, 大的数据量对单机的存储压力较大, 最终会耗尽系统的内存而将压力转移到磁盘IO上。

为了解决这些问题, 有两个基本的方法: 垂直扩展和水平扩展。

- 垂直扩展：增加更多的CPU和存储资源来扩展容量。
- 水平扩展：将数据集分布在多个服务器上。**MongoDB的分片就是水平扩展的体现**。

**分片设计思想**

分片为应对高吞吐量与大数据量提供了方法。使用分片减少了每个分片需要处理的请求数，因此，通过水平扩展，集群可以提高自己的存储容量和吞吐量。举例来说，当插入一条数据时，应用只需要访问存储这条数据的分片.

**分片目的**

- 读/写能力提升
- 存储容量扩容
- 高可用性

#### [#](#mongodb分片集群的结构) MongoDB分片集群的结构？

一个MongoDB的分片集群包含如下组件：

- `shard`: 即分片，真正的数据存储位置，以chunk为单位存数据；每个分片可以部署为一个复制集。
- `mongos`: 查询的路由, 提供客户端和分片集群之间的接口。
- `config servers`: 存储元数据和配置数据。

![img](/images/db/mongo/mongo-z-shard-1.png)

这里要注意mongos提供的是客户端application与MongoDB分片集群的路由功能，这里分片集群包含了分片的collection和非分片的collection。如下展示了通过路由访问分片的collection和非分片的collection:

![img](/images/db/mongo/mongo-z-shard-2.png)

#### [#](#mongodb分片的内部是如何管理数据的呢) MongoDB分片的内部是如何管理数据的呢？

在一个shard server内部，MongoDB还是会**把数据分为chunks**，每个chunk代表这个shard server内部一部分数据。chunk的产生，会有以下两个用途：

- `Splitting`：当一个chunk的大小超过配置中的chunk size时，MongoDB的后台进程会把这个chunk切分成更小的chunk，从而避免chunk过大的情况
- `Balancing`：在MongoDB中，balancer是一个后台进程，负责chunk的迁移，从而均衡各个shard server的负载，系统初始1个chunk，chunk size默认值64M,生产库上选择适合业务的chunk size是最好的。MongoDB会自动拆分和迁移chunks。

分片集群的数据分布（shard节点）

- 使用chunk来存储数据
- 进群搭建完成之后，默认开启一个chunk，大小是64M，
- 存储需求超过64M，chunk会进行分裂，如果单位时间存储需求很大，设置更大的chunk
- chunk会被自动均衡迁移。

**chunk分裂及迁移**

随着数据的增长，其中的数据大小超过了配置的chunk size，默认是64M，则这个chunk就会分裂成两个。数据的增长会让chunk分裂得越来越多。

![img](/images/db/mongo/mongo-z-shard-4.png)

这时候，各个shard 上的chunk数量就会不平衡。这时候，mongos中的一个组件balancer 就会执行自动平衡。把chunk从chunk数量最多的shard节点挪动到数量最少的节点。

![img](/images/db/mongo/mongo-z-shard-5.png)

#### [#](#mongodb-中collection的数据是根据什么进行分片的呢) MongoDB 中Collection的数据是根据什么进行分片的呢？

MongoDB 中Collection的数据是根据什么进行分片的呢？这就是我们要介绍的**分片键（Shard key）**；那么又是采用过了什么算法进行分片的呢？这就是紧接着要介绍的**范围分片（range sharding）\**和\**哈希分片（Hash Sharding)**。

- **哈希分片（Hash Sharding)**

分片过程中利用哈希索引作为分片，基于哈希片键最大的好处就是保证数据在各个节点分布基本均匀。

对于基于哈希的分片，MongoDB计算一个字段的哈希值，并用这个哈希值来创建数据块。在使用基于哈希分片的系统中，拥有**相近分片键**的文档很可能不会存储在同一个数据块中，因此数据的分离性更好一些。

![img](/images/db/mongo/mongo-z-shard-6.png)

- **范围分片（range sharding）**

将单个Collection的数据分散存储在多个shard上，用户可以指定根据集合内文档的某个字段即shard key来进行范围分片（range sharding）。

对于基于范围的分片，MongoDB按照片键的范围把数据分成不同部分:

![img](/images/db/mongo/mongo-z-shard-7.png)

在使用片键做范围划分的系统中，拥有**相近分片键**的文档很可能存储在同一个数据块中，因此也会存储在同一个分片中。

- **哈希和范围的结合**

如下是基于X索引字段进行范围分片，但是随着X的增长，大于20的数据全部进入了Chunk C, 这导致了数据的不均衡。 ![img](/images/db/mongo/mongo-z-shard-111.png)

这时对X索引字段建哈希索引：

![img](/images/db/mongo/mongo-z-shard-112.png)

#### [#](#mongodb如何做备份恢复) MongoDB如何做备份恢复？

- JSON格式：mongoexport/mongoimport

JSON可读性强但体积较大，JSON虽然具有较好的跨版本通用性，但其只保留了数据部分，不保留索引，账户等其他基础信息。

- BSON格式：mongoexport/mongoimport

BSON则是二进制文件，体积小但对人类几乎没有可读性。

#### [#](#mongodb如何设计文档模型) MongoDB如何设计文档模型？

举几个例子：

- **多态模式**

当集合中的所有文档都具有**相似但不相同的结构**时，我们将其称为多态模式； 比如运动员的运动记录：

![img](/images/db/mongo/mongo-y-doc-2.png)

![img](/images/db/mongo/mongo-y-doc-2.gif)

- **属性模式**

出于性能原因考虑，为了优化搜索我们可能需要许多索引以照顾到所有子集。创建所有这些索引可能会降低性能。属性模式为这种情况提供了一个很好的解决方案。

假设现在有一个关于电影的集合。其中所有文档中可能都有类似的字段：标题、导演、制片人、演员等等。假如我们希望在上映日期这个字段进行搜索，这时面临的挑战是“哪个上映日期”？在不同的国家，电影通常在不同的日期上映。

```json
{
    title: "Star Wars",
    director: "George Lucas",
    ...
    release_US: ISODate("1977-05-20T01:00:00+01:00"),
    release_France: ISODate("1977-10-19T01:00:00+01:00"),
    release_Italy: ISODate("1977-10-20T01:00:00+01:00"),
    release_UK: ISODate("1977-12-27T01:00:00+01:00"),
    ...
}
```

使用属性模式，我们可以将此信息移至数组中并减少对索引需求。我们将这些信息转换成一个包含键值对的数组：

```json
{
    title: "Star Wars",
    director: "George Lucas",
    …
    releases: [
        {
        location: "USA",
        date: ISODate("1977-05-20T01:00:00+01:00")
        },
        {
        location: "France",
        date: ISODate("1977-10-19T01:00:00+01:00")
        },
        {
        location: "Italy",
        date: ISODate("1977-10-20T01:00:00+01:00")
        },
        {
        location: "UK",
        date: ISODate("1977-12-27T01:00:00+01:00")
        },
        … 
    ],
    … 
}
```

- **桶模式**

这种模式在处理物联网（IOT）、实时分析或通用时间序列数据时特别有效。通过将数据放在一起，我们可以更容易地将数据组织成特定的组，提高发现历史趋势或提供未来预测的能力，同时还能对存储进行优化。

随着数据在一段时间内持续流入（时间序列数据），我们可能倾向于将每个测量值存储在自己的文档中。然而，这种倾向是一种非常偏向于关系型数据处理的方式。如果我们有一个传感器每分钟测量温度并将其保存到数据库中，我们的数据流可能看起来像这样：

```json
{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:00:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:01:00.000Z"),
   temperature: 40
}

{
   sensor_id: 12345,
   timestamp: ISODate("2019-01-31T10:02:00.000Z"),
   temperature: 41
}
```

通过将桶模式应用于数据模型，我们可以在节省索引大小、简化潜在的查询以及在文档中使用预聚合数据的能力等方面获得一些收益。获取上面的数据流并对其应用桶模式，我们可以得到：

```json
{
    sensor_id: 12345,
    start_date: ISODate("2019-01-31T10:00:00.000Z"),
    end_date: ISODate("2019-01-31T10:59:59.000Z"),
    measurements: [
       {
       timestamp: ISODate("2019-01-31T10:00:00.000Z"),
       temperature: 40
       },
       {
       timestamp: ISODate("2019-01-31T10:01:00.000Z"),
       temperature: 40
       },
       … 
       {
       timestamp: ISODate("2019-01-31T10:42:00.000Z"),
       temperature: 42
       }
    ],
   transaction_count: 42,
   sum_temperature: 2413
}
```

#### [#](#mongodb如何进行性能优化) MongoDB如何进行性能优化？

- **慢查询**

为了定位查询，需要查看当前mongo profile的级别, profile的级别有0|1|2，分别代表意思: 0代表关闭，1代表记录慢命令，2代表全部

```json
db.getProfilingLevel()
```

显示为0， 表示默认下是没有记录的。

设置profile级别，设置为记录慢查询模式, 所有超过1000ms的查询语句都会被记录下来

```json
db.setProfilingLevel(1, 1000)
```

### [#](#_8-5-elasticsearch) 8.5 ElasticSearch

#### [#](#elasticsearch是什么-基于lucene的-那么为什么不是直接使用lucene呢) ElasticSearch是什么？基于Lucene的，那么为什么不是直接使用Lucene呢？

Lucene 可以说是当下最先进、高性能、全功能的搜索引擎库。Elasticsearch 也是使用 Java 编写的，它的内部使用 Lucene 做索引与搜索，但是它的目的是使全文检索变得简单，**通过隐藏 Lucene 的复杂性，取而代之的提供一套简单一致的 RESTful API**。

然而，Elasticsearch 不仅仅是 Lucene，并且也不仅仅只是一个全文搜索引擎：

- 一个分布式的实时文档存储，每个字段 可以被索引与搜索
- 一个分布式实时分析搜索引擎
- 能胜任上百个服务节点的扩展，并支持 PB 级别的结构化或者非结构化数据

一个ES和数据库的对比

![img](/images/db/es/es-introduce-1-3.png)

#### [#](#elk-技术栈的常见应用场景) ELK 技术栈的常见应用场景？

- 日志系统

![img](/images/db/es/es-introduce-2-5.png)

增加数据源，和使用MQ

![img](/images/db/es/es-introduce-2-6.png)

- Metric收集和APM性能监控

![img](/images/db/es/es-introduce-2-7.png)

#### [#](#es中索引模板是什么) ES中索引模板是什么？

索引模板是一种告诉Elasticsearch在创建索引时如何配置索引的方法。

- **使用方式**

在创建索引之前可以先配置模板，这样在创建索引（手动创建索引或通过对文档建立索引）时，模板设置将用作创建索引的基础。

- **模板类型**

1. **组件模板**是可重用的构建块，用于配置映射，设置和别名；它们不会直接应用于一组索引。
2. **索引模板**可以包含组件模板的集合，也可以直接指定设置，映射和别名。

#### [#](#es中索引的生命周期管理) ES中索引的生命周期管理？

- **为什么会引入**？

随着时间的增长索引的数量也会持续增长，然而这些场景基本上只有最近一段时间的数据有使用价值或者会被经常使用（热数据），而历史数据几乎没有作用或者很少会被使用（冷数据），这个时候就需要对索引进行一定策略的维护管理甚至是删除清理，否则随着数据量越来越多除了浪费磁盘与内存空间之外，还会严重影响 Elasticsearch 的性能。

- **哪个版本引入的**？

在 Elastic Stack 6.6 版本后推出了新功能 Index Lifecycle Management(索引生命周期管理)，支持针对索引的全生命周期托管管理，并且在 Kibana 上也提供了一套UI界面来配置策略。

- 索引生命周期常见的阶段

  ？ 

  - hot: 索引还存在着大量的读写操作。
  - warm:索引不存在写操作，还有被查询的需要。
  - cold:数据不存在写操作，读操作也不多。
  - delete:索引不再需要，可以被安全删除。

#### [#](#es查询和聚合的有哪些方式) ES查询和聚合的有哪些方式？

- DSL

  - 基于文本 - match, query string
  - 基于词项 - term
  - 复合查询 - 5种

- EQL

   Elastic Query Language 

  - bucket
  - metric
  - pipline

- **SQL**

#### [#](#es查询中query和filter的区别) ES查询中query和filter的区别？

query 是查询+score, 而filter仅包含查询， 比如在复合查询中constant_score查询无需计算score，所以对应查询是filter而不是query。

比如

```bash
GET /test-dsl-constant/_search
{
  "query": {
    "constant_score": {
      "filter": {
        "term": { "content": "apple" }
      },
      "boost": 1.2
    }
  }
}
```

#### [#](#es查询中match和term的区别) ES查询中match和term的区别？

term是基于索引的词项，而match基于文本。

比如：

```bash
GET /test-dsl-match/_search
{
    "query": {
        "match": {
            "title": "BROWN DOG"
        }
    }
}
```

等同于

```bash
GET /test-dsl-match/_search
{
  "query": {
    "match": {
      "title": {
        "query": "BROWN DOG",
        "operator": "or"
      }
    }
  }
}
```

也等同于

```bash
GET /test-dsl-match/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "term": {
            "title": "brown"
          }
        },
        {
          "term": {
            "title": "dog"
          }
        }
      ]
    }
  }
}
```

#### [#](#es查询中should和must的区别) ES查询中should和must的区别？

should是任意匹配，must是同时匹配。

比如，接上面的例子是should(任意匹配）， 而如下查询是must(同时匹配)：

```bash
GET /test-dsl-match/_search
{
  "query": {
    "match": {
      "title": {
        "query": "BROWN DOG",
        "operator": "and"
      }
    }
  }
}
```

等同于

```bash
GET /test-dsl-match/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "title": "brown"
          }
        },
        {
          "term": {
            "title": "dog"
          }
        }
      ]
    }
  }
}
```

#### [#](#es查询中match-match-phrase和match-phrase-prefix有什么区别) ES查询中match，match_phrase和match_phrase_prefix有什么区别？

match本质上是对term组合，match_phrase本质是连续的term的查询（and关系），match_phrase_prefix在match_phrase基础上提供了一种可以查最后一个词项是前缀的方法

比如：

某个字段内容是“quick brown fox”, 如果我们需要查询包含“quick brown f”，就需要使用match_phrase_prefix，因为f不是完整的term分词，不能用match_phrase。

#### [#](#es查询中什么是复合查询-有哪些复合查询方式) ES查询中什么是复合查询？有哪些复合查询方式？

在查询中会有多种条件组合的查询，在ElasticSearch中叫复合查询。它提供了5种复合查询方式：

- bool query(布尔查询)
  - 通过布尔逻辑将较小的查询组合成较大的查询。
- boosting query(提高查询)
  - 不同于bool查询，bool查询中只要一个子查询条件不匹配那么搜索的数据就不会出现。而boosting query则是降低显示的权重/优先级（即score)。
- constant_score（固定分数查询）
  - 查询某个条件时，固定的返回指定的score；显然当不需要计算score时，只需要filter条件即可，因为filter context忽略score。
- dis_max(最佳匹配查询）
  - 分离最大化查询（Disjunction Max Query）指的是： 将任何与任一查询匹配的文档作为结果返回，但只将最佳匹配的评分作为查询的评分结果返回 。
- function_score(函数查询）
  - 简而言之就是用自定义function的方式来计算_score。

#### [#](#es聚合中的bucket聚合有哪些-如何理解) ES聚合中的Bucket聚合有哪些？如何理解？

设计上大概分为三类（当然有些是第二和第三类的融合）

![img](/images/db/es/es-agg-bucket-1.png)

#### [#](#es聚合中的metric聚合有哪些-如何理解) ES聚合中的Metric聚合有哪些？如何理解？

如何理解？

1. **从分类看**：Metric聚合分析分为**单值分析**和**多值分析**两类
2. **从功能看**：根据具体的应用场景设计了一些分析api, 比如地理位置，百分数等等

- 单值分析

  : 只输出一个分析结果 

  - 标准stat型 
    - `avg` 平均值
    - `max` 最大值
    - `min` 最小值
    - `sum` 和
    - `value_count` 数量
  - 其它类型 
    - `cardinality` 基数（distinct去重）
    - `weighted_avg` 带权重的avg
    - `median_absolute_deviation` 中位值

- 多值分析

  : 单值之外的 

  - stats型 
    - `stats` 包含avg,max,min,sum和count
    - `matrix_stats` 针对矩阵模型
    - `extended_stats`
    - `string_stats` 针对字符串
  - 百分数型 
    - `percentiles` 百分数范围
    - `percentile_ranks` 百分数排行
  - 地理位置型 
    - `geo_bounds` Geo bounds
    - `geo_centroid` Geo-centroid
    - `geo_line` Geo-Line
  - Top型 
    - `top_hits` 分桶后的top hits
    - `top_metrics`

#### [#](#es聚合中的管道聚合有哪些-如何理解) ES聚合中的管道聚合有哪些？如何理解？

简单而言：让上一步的聚合结果成为下一个聚合的输入，这就是管道。

如何理解？

**第一个维度**：管道聚合有很多不同**类型**，每种类型都与其他聚合计算不同的信息，但是可以将这些类型分为两类：

- **父级** 父级聚合的输出提供了一组管道聚合，它可以计算新的存储桶或新的聚合以添加到现有存储桶中。
- **兄弟** 同级聚合的输出提供的管道聚合，并且能够计算与该同级聚合处于同一级别的新聚合。

**第二个维度**：根据**功能设计**的意图

比如前置聚合可能是Bucket聚合，后置的可能是基于Metric聚合，那么它就可以成为一类管道

进而引出了：`xxx bucket`

- Bucket聚合 -> Metric聚合

  ： bucket聚合的结果，成为下一步metric聚合的输入 

  - Average bucket
  - Min bucket
  - Max bucket
  - Sum bucket
  - Stats bucket
  - Extended stats bucket

...

#### [#](#如何理解es的结构和底层实现) 如何理解ES的结构和底层实现？

- ES的整体结构

  ? 

  - 一个 ES Index 在集群模式下，有多个 Node （节点）组成。每个节点就是 ES 的Instance (实例)。
  - 每个节点上会有多个 shard （分片）， P1 P2 是主分片, R1 R2 是副本分片
  - 每个分片上对应着就是一个 Lucene Index（底层索引文件）
  - Lucene Index 是一个统称 
    - 由多个 Segment （段文件，就是倒排索引）组成。每个段文件存储着就是 Doc 文档。
    - commit point记录了所有 segments 的信息

![img](/images/db/es/es-th-2-3.png)

- 底层和数据文件

  ？ 

  - 倒排索引（词典+倒排表）
  - doc values - 列式存储
  - 正向文件 - 行式存储

![img](/images/db/es/es-th-2-2.png)

文件的关系如下：

![img](/images/db/es/es-th-3-2.jpeg)

#### [#](#es内部读取文档是怎样的-如何实现的) ES内部读取文档是怎样的？如何实现的？

- **主分片或者副本分片检索文档的步骤顺序**：

![img](/images/db/es/es-th-2-21.png)

1. 客户端向 Node 1 发送获取请求。
2. 节点使用文档的 _id 来确定文档属于分片 0 。分片 0 的副本分片存在于所有的三个节点上。 在这种情况下，它将请求转发到 Node 2 。
3. Node 2 将文档返回给 Node 1 ，然后将文档返回给客户端。

在处理读取请求时，协调结点在每次请求的时候都会通过轮询所有的副本分片来达到负载均衡。

- **读取文档的两阶段查询**？

所有的搜索系统一般都是两阶段查询，第一阶段查询到匹配的DocID，第二阶段再查询DocID对应的完整文档，这种在Elasticsearch中称为query_then_fetch。（这里主要介绍最常用的2阶段查询）。

![img](/images/db/es/es-th-2-32.jpeg)

1. 在初始查询阶段时，查询会广播到索引中每一个分片拷贝（主分片或者副本分片）。 每个分片在本地执行搜索并构建一个匹配文档的大小为 from + size 的优先队列。PS：在2. 搜索的时候是会查询Filesystem Cache的，但是有部分数据还在Memory Buffer，所以搜索是近实时的。
2. 每个分片返回各自优先队列中 所有文档的 ID 和排序值 给协调节点，它合并这些值到自己的优先队列中来产生一个全局排序后的结果列表。
3. 接下来就是 取回阶段，协调节点辨别出哪些文档需要被取回并向相关的分片提交多个 GET 请求。每个分片加载并丰富文档，如果有需要的话，接着返回文档给协调节点。一旦所有的文档都被取回了，协调节点返回结果给客户端。

#### [#](#es内部索引文档是怎样的-如何实现的) ES内部索引文档是怎样的？如何实现的？

- **新建单个文档所需要的步骤顺序**：

![img](/images/db/es/es-th-2-4.png)

1. 客户端向 Node 1 发送新建、索引或者删除请求。
2. 节点使用文档的 _id 确定文档属于分片 0 。请求会被转发到 Node 3，因为分片 0 的主分片目前被分配在 Node 3 上。
3. Node 3 在主分片上面执行请求。如果成功了，它将请求并行转发到 Node 1 和 Node 2 的副本分片上。一旦所有的副本分片都报告成功, Node 3 将向协调节点报告成功，协调节点向客户端报告成功。

- **看下整体的索引流程**

![img](/images/db/es/es-th-2-5.jpeg)

1. 协调节点默认使用文档ID参与计算（也支持通过routing），以便为路由提供合适的分片。

```bash
shard = hash(document_id) % (num_of_primary_shards)
```

1. 当分片所在的节点接收到来自协调节点的请求后，会将请求写入到Memory Buffer，然后定时（默认是每隔1秒）写入到Filesystem Cache，这个从Momery Buffer到Filesystem Cache的过程就叫做refresh；
2. 当然在某些情况下，存在Momery Buffer和Filesystem Cache的数据可能会丢失，ES是通过translog的机制来保证数据的可靠性的。其实现机制是接收到请求后，同时也会写入到translog中，当Filesystem cache中的数据写入到磁盘中时，才会清除掉，这个过程叫做flush。
3. 在flush过程中，内存中的缓冲将被清除，内容被写入一个新段，段的fsync将创建一个新的提交点，并将内容刷新到磁盘，旧的translog将被删除并开始一个新的translog。 flush触发的时机是定时触发（默认30分钟）或者translog变得太大（默认为512M）时。

#### [#](#es底层数据持久化的过程) ES底层数据持久化的过程？

**通过分步骤看数据持久化过程**：**write -> refresh -> flush -> merge**

- **write 过程**

![img](/images/db/es/es-th-2-6.png)

一个新文档过来，会存储在 in-memory buffer 内存缓存区中，顺便会记录 Translog（Elasticsearch 增加了一个 translog ，或者叫事务日志，在每一次对 Elasticsearch 进行操作时均进行了日志记录）。

这时候数据还没到 segment ，是搜不到这个新文档的。数据只有被 refresh 后，才可以被搜索到。

- **refresh 过程**

![img](/images/db/es/es-th-2-7.png)

refresh 默认 1 秒钟，执行一次上图流程。ES 是支持修改这个值的，通过 index.refresh_interval 设置 refresh （冲刷）间隔时间。refresh 流程大致如下：

1. in-memory buffer 中的文档写入到新的 segment 中，但 segment 是存储在文件系统的缓存中。此时文档可以被搜索到
2. 最后清空 in-memory buffer。注意: Translog 没有被清空，为了将 segment 数据写到磁盘
3. 文档经过 refresh 后， segment 暂时写到文件系统缓存，这样避免了性能 IO 操作，又可以使文档搜索到。refresh 默认 1 秒执行一次，性能损耗太大。一般建议稍微延长这个 refresh 时间间隔，比如 5 s。因此，ES 其实就是准实时，达不到真正的实时。

- **flush 过程**

每隔一段时间—例如 translog 变得越来越大—索引被刷新（flush）；一个新的 translog 被创建，并且一个全量提交被执行

![img](/images/db/es/es-th-2-9.png)

上个过程中 segment 在文件系统缓存中，会有意外故障文档丢失。那么，为了保证文档不会丢失，需要将文档写入磁盘。那么文档从文件缓存写入磁盘的过程就是 flush。写入磁盘后，清空 translog。具体过程如下：

1. 所有在内存缓冲区的文档都被写入一个新的段。
2. 缓冲区被清空。
3. 一个Commit Point被写入硬盘。
4. 文件系统缓存通过 fsync 被刷新（flush）。
5. 老的 translog 被删除。

- **merge 过程**

由于自动刷新流程每秒会创建一个新的段 ，这样会导致短时间内的段数量暴增。而段数目太多会带来较大的麻烦。 每一个段都会消耗文件句柄、内存和cpu运行周期。更重要的是，每个搜索请求都必须轮流检查每个段；所以段越多，搜索也就越慢。

Elasticsearch通过在后台进行Merge Segment来解决这个问题。小的段被合并到大的段，然后这些大的段再被合并到更大的段。

当索引的时候，刷新（refresh）操作会创建新的段并将段打开以供搜索使用。合并进程选择一小部分大小相似的段，并且在后台将它们合并到更大的段中。这并不会中断索引和搜索。

![img](/images/db/es/es-th-2-10.png)

一旦合并结束，老的段被删除：

1. 新的段被刷新（flush）到了磁盘。 ** 写入一个包含新段且排除旧的和较小的段的新提交点。
2. 新的段被打开用来搜索。
3. 老的段被删除。

![img](/images/db/es/es-th-2-11.png)

合并大的段需要消耗大量的I/O和CPU资源，如果任其发展会影响搜索性能。Elasticsearch在默认情况下会对合并流程进行资源限制，所以搜索仍然 有足够的资源很好地执行。

#### [#](#es遇到什么性能问题-如何优化的) ES遇到什么性能问题，如何优化的？

分几个方向说几个点：

- **硬件配置优化** 包括三个因素：CPU、内存和 IO。

  - **CPU**： 大多数 Elasticsearch 部署往往对 CPU 要求不高； CPUs 和更多的核数之间选择，选择更多的核数更好。多个内核提供的额外并发远胜过稍微快一点点的时钟频率。

  - 内存

    ： 

    - **配置**： 由于 ES 构建基于 lucene，而 lucene 设计强大之处在于 lucene 能够很好的利用操作系统内存来缓存索引数据，以提供快速的查询性能。lucene 的索引文件 segements 是存储在单文件中的，并且不可变，对于 OS 来说，能够很友好地将索引文件保持在 cache 中，以便快速访问；因此，我们很有必要将一半的物理内存留给 lucene；另**一半的物理内存留给 ES**（JVM heap）。
    - **禁止 swap** 禁止 swap，一旦允许内存与磁盘的交换，会引起致命的性能问题。可以通过在 elasticsearch.yml 中 bootstrap.memory_lock: true，以保持 JVM 锁定内存，保证 ES 的性能。
    - **垃圾回收器**： 已知JDK 8附带的HotSpot JVM的早期版本存在一些问题，当启用G1GC收集器时，这些问题可能导致索引损坏。受影响的版本早于JDK 8u40随附的HotSpot版本。如果你使用的JDK8较高版本，或者JDK9+，我推荐你使用G1 GC； 因为我们目前的项目使用的就是G1 GC，运行效果良好，对Heap大对象优化尤为明显。

  - **磁盘** 在经济压力能承受的范围下，尽量使用固态硬盘（SSD）

- **索引方面优化**

  - **批量提交** 当有大量数据提交的时候，建议采用批量提交（Bulk 操作）；此外使用 bulk 请求时，每个请求不超过几十M，因为太大会导致内存使用过大。
  - **增加 Refresh 时间间隔** 为了提高索引性能，Elasticsearch 在写入数据的时候，采用延迟写入的策略，即数据先写到内存中，当超过默认1秒（index.refresh_interval）会进行一次写入操作，就是将内存中 segment 数据刷新到磁盘中，此时我们才能将数据搜索出来，所以这就是为什么 Elasticsearch 提供的是近实时搜索功能，而不是实时搜索功能。如果我们的系统对数据延迟要求不高的话，我们可以**通过延长 refresh 时间间隔，可以有效地减少 segment 合并压力，提高索引速度**。比如在做全链路跟踪的过程中，我们就将 index.refresh_interval 设置为30s，减少 refresh 次数。再如，在进行全量索引时，可以将 refresh 次数临时关闭，即 index.refresh_interval 设置为-1，数据导入成功后再打开到正常模式，比如30s。
  - **索引缓冲的设置可以控制多少内存分配** indices.memory.index_buffer_size 接受一个百分比或者一个表示字节大小的值。默认是10%
  - **translog 相关的设置** 控制数据从内存到硬盘的操作频率，以减少硬盘 IO。可将 sync_interval 的时间设置大一些。默认为5s。也可以控制 tranlog 数据块的大小，达到 threshold 大小时，才会 flush 到 lucene 索引文件。默认为512m。
  - **_id 字段的使用** _id 字段的使用，应尽可能避免自定义 _id，以避免针对 ID 的版本管理；建议使用 ES 的默认 ID 生成策略或使用数字类型 ID 做为主键。
  - **_all 字段及 _source 字段的使用** _all 字段及 _source 字段的使用，应该注意场景和需要，_all 字段包含了所有的索引字段，方便做全文检索，如果无此需求，可以禁用；_source 存储了原始的 document 内容，如果没有获取原始文档数据的需求，可通过设置 includes、excludes 属性来定义放入 _source 的字段。
  - **合理的配置使用 index 属性** 合理的配置使用 index 属性，analyzed 和 not_analyzed，根据业务需求来控制字段是否分词或不分词。只有 groupby 需求的字段，配置时就设置成 not_analyzed，以提高查询或聚类的效率。

- **查询方面优化**

  - **Filter VS Query**
  - **深度翻页** 使用 Elasticsearch scroll 和 scroll-scan 高效滚动的方式来解决这样的问题。也可以结合实际业务特点，文档 id 大小如果和文档创建时间是一致有序的，可以以文档 id 作为分页的偏移量，并将其作为分页查询的一个条件。
  - **避免层级过深的聚合查询**， 层级过深的aggregation , 会导致内存、CPU消耗，建议在服务层通过程序来组装业务，也可以通过pipeline的方式来优化。
  - **通过开启慢查询配置定位慢查询**

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/interview/x-interview.html



# Java 全栈知识点问题汇总（下）

提示

Java 全栈知识点问题汇总（下）, [Java 全栈知识点问题汇总（上）]()。@pdai

- Java 全栈知识点问题汇总（下）
  - 9 开发基础
    - 9.1 常用类库
      - [平时常用的开发工具库有哪些？](#平时常用的开发工具库有哪些)
      - [Java常用的JSON库有哪些？有啥注意点？](#java常用的json库有哪些有啥注意点)
      - [Lombok工具库用来解决什么问题？](#lombok工具库用来解决什么问题)
      - [为什么很多公司禁止使用lombok？](#为什么很多公司禁止使用lombok)
      - [MapStruct工具库用来解决什么问题？](#mapstruct工具库用来解决什么问题)
      - [Lombok和MapStruct工具库的原理？](#lombok和mapstruct工具库的原理)
    - 9.2 网络协议和工具
      - [什么是754层网络模型？](#什么是754层网络模型)
      - [TCP建立连接过程的三次握手？](#tcp建立连接过程的三次握手)
      - [SYN洪泛攻击(SYN Flood，半开放攻击)，怎么解决？](#syn洪泛攻击syn-flood半开放攻击怎么解决)
      - [TCP断开连接过程的四次挥手？](#tcp断开连接过程的四次挥手)
      - [DNS 解析流程？](#dns-解析流程)
      - [为什么DNS通常基于UDP？](#为什么dns通常基于udp)
      - [什么是DNS劫持？](#什么是dns劫持)
      - [什么是DNS污染？](#什么是dns污染)
      - [为什么要DNS流量监控？](#为什么要dns流量监控)
      - [输入URL 到页面加载过程？](#输入url-到页面加载过程)
      - [如何使用netstat查看服务及监听端口？](#如何使用netstat查看服务及监听端口)
      - [如何使用TCPDump抓包？](#如何使用tcpdump抓包)
      - [如何使用Wireshark抓包分析？](#如何使用wireshark抓包分析)
    - 9.3 开发安全
      - [开发中有哪些常见的Web安全漏洞？](#开发中有哪些常见的web安全漏洞)
      - [什么是注入攻击？举例说明？](#什么是注入攻击举例说明)
      - [什么是CSRF？举例说明并给出开发中解决方案？](#什么是csrf举例说明并给出开发中解决方案)
      - [什么是XSS？举例说明？](#什么是xss举例说明)
      - [一般的渗透测试流程？](#一般的渗透测试流程)
    - 9.4 单元测试
      - [谈谈你对单元测试的理解？](#谈谈你对单元测试的理解)
      - [JUnit 5整体架构？](#junit-5整体架构)
      - [JUnit 5与Junit4的差别在哪里？](#junit-5与junit4的差别在哪里)
      - [你在开发中使用什么框架来做单元测试？](#你在开发中使用什么框架来做单元测试)
    - 9.5 代码质量
      - [你们项目中是如何保证代码质量的？](#你们项目中是如何保证代码质量的)
      - [你们项目中是如何做code review的？](#你们项目中是如何做code-review的)
    - 9.6 代码重构
      - [如何去除多余的if else？](#如何去除多余的if-else)
      - [如何去除不必要的!=判空？](#如何去除不必要的判空)
  - 10 开发框架和中间件
    - 10.1 Spring
      - [什么是Spring框架？](#什么是spring框架)
      - [列举一些重要的Spring模块？](#列举一些重要的spring模块)
      - [什么是IOC? 如何实现的？](#什么是ioc-如何实现的)
      - [什么是AOP? 有哪些AOP的概念？](#什么是aop-有哪些aop的概念)
      - [AOP 有哪些应用场景？](#aop-有哪些应用场景)
      - [有哪些AOP Advice通知的类型？](#有哪些aop-advice通知的类型)
      - [AOP 有哪些实现方式？](#aop-有哪些实现方式)
      - [谈谈你对CGLib的理解？](#谈谈你对cglib的理解)
      - [Spring AOP和AspectJ AOP有什么区别？](#spring-aop和aspectj-aop有什么区别)
      - [Spring中的bean的作用域有哪些？](#spring中的bean的作用域有哪些)
      - [Spring中的单例bean的线程安全问题了解吗？](#spring中的单例bean的线程安全问题了解吗)
      - [Spring中的bean生命周期？](#spring中的bean生命周期)
      - [说说自己对于Spring MVC的了解？](#说说自己对于spring-mvc的了解)
      - [Spring MVC的工作原理了解嘛？](#spring-mvc的工作原理了解嘛)
      - [Spring框架中用到了哪些设计模式？](#spring框架中用到了哪些设计模式)
      - [@Component和@Bean的区别是什么？](#component和bean的区别是什么)
      - [将一个类声明为Spring的bean的注解有哪些？](#将一个类声明为spring的bean的注解有哪些)
      - [Spring事务管理的方式有几种？](#spring事务管理的方式有几种)
      - [Spring事务中的隔离级别有哪几种？](#spring事务中的隔离级别有哪几种)
      - [Spring事务中有哪几种事务传播行为？](#spring事务中有哪几种事务传播行为)
      - [Bean Factory和ApplicationContext有什么区别？](#bean-factory和applicationcontext有什么区别)
      - [如何定义bean的范围？](#如何定义bean的范围)
      - [可以通过多少种方式完成依赖注入？](#可以通过多少种方式完成依赖注入)
    - 10.2 Spring Boot
      - [什么是SpringBoot？](#什么是springboot)
      - [为什么使用SpringBoot？](#为什么使用springboot)
      - [Spring、Spring MVC和SpringBoot有什么区别？](#springspring-mvc和springboot有什么区别)
      - [SpringBoot自动配置的原理?](#springboot自动配置的原理)
      - [Spring Boot的核心注解是哪些？他主由哪几个注解组成的？](#spring-boot的核心注解是哪些他主由哪几个注解组成的)
      - [SpringBoot的核心配置文件有哪几个？他们的区别是什么？](#springboot的核心配置文件有哪几个他们的区别是什么)
      - [什么是Spring Boot Starter？有哪些常用的？](#什么是spring-boot-starter有哪些常用的)
      - [spring-boot-starter-parent有什么作用？](#spring-boot-starter-parent有什么作用)
      - [如何自定义Spring Boot Starter？](#如何自定义spring-boot-starter)
      - [为什么需要spring-boot-maven-plugin？](#为什么需要spring-boot-maven-plugin)
      - [SpringBoot 打成jar和普通的jar有什么区别？](#springboot-打成jar和普通的jar有什么区别)
      - [如何使用Spring Boot实现异常处理？](#如何使用spring-boot实现异常处理)
      - [SpringBoot 实现热部署有哪几种方式？](#springboot-实现热部署有哪几种方式)
      - [Spring Boot中的监视器是什么？](#spring-boot中的监视器是什么)
      - [Spring Boot 可以兼容老 Spring 项目吗？](#spring-boot-可以兼容老-spring-项目吗)
    - 10.3 Spring Security
      - [什么是Spring Security？核心功能？](#什么是spring-security核心功能)
      - [Spring Security的原理?](#spring-security的原理)
      - [Spring Security基于用户名和密码的认证模式流程？](#spring-security基于用户名和密码的认证模式流程)
    - [10.4 MyBatis](#104-mybatis)
    - [10.5 JPA](#105-jpa)
    - 10.6 日志框架
      - [什么是日志系统和日志门面？分别有哪些框架？](#什么是日志系统和日志门面分别有哪些框架)
      - [日志库中使用桥接模式解决什么问题？](#日志库中使用桥接模式解决什么问题)
      - [在日志配置时会考虑哪些点？](#在日志配置时会考虑哪些点)
      - [对Java日志组件选型的建议？](#对java日志组件选型的建议)
      - [对日志架构使用比较好的实践？](#对日志架构使用比较好的实践)
      - [对现有系统日志架构的改造建议？](#对现有系统日志架构的改造建议)
    - 10.7 Tomcat
      - [Tomcat 整体架构的设计？](#tomcat-整体架构的设计)
      - [Tomcat 一个请求的处理流程？](#tomcat-一个请求的处理流程)
      - [Tomcat 中类加载机制？](#tomcat-中类加载机制)
      - [Tomcat Container设计？](#tomcat-container设计)
      - [Tomcat LifeCycle机制？](#tomcat-lifecycle机制)
      - [Tomcat 中Executor?](#tomcat-中executor)
      - [Tomcat 中的设计模式？](#tomcat-中的设计模式)
      - [Tomcat JMX拓展机制？](#tomcat-jmx拓展机制)
  - 11 开发工具
    - 11.1 Git
      - [Git中5个区，和具体操作？](#git中5个区和具体操作)
      - [平时是怎么提交代码的？](#平时是怎么提交代码的)
    - 11.2 Maven
      - [Maven中包的依赖原则？如何解决冲突？](#maven中包的依赖原则如何解决冲突)
      - [Maven 项目生命周期与构建原理？](#maven-项目生命周期与构建原理)
  - 12 架构
    - 12.1 架构基础
      - [如何理解架构的演进？](#如何理解架构的演进)
      - [如何理解架构的服务化趋势？](#如何理解架构的服务化趋势)
      - [架构中有哪些技术点？](#架构中有哪些技术点)
    - 12.2 缓存
      - [谈谈架构中的缓存应用？](#谈谈架构中的缓存应用)
      - [在开发中缓存具体如何实现？](#在开发中缓存具体如何实现)
      - [缓存会有哪些问题？如何解决？](#缓存会有哪些问题如何解决)
      - [使用缓存的经验？](#使用缓存的经验)
    - 12.3 限流
      - [什么是限流？三种限流的算法？](#什么是限流三种限流的算法)
      - [限流令牌桶和漏桶对比？](#限流令牌桶和漏桶对比)
      - [在单机情况下如何实现限流？](#在单机情况下如何实现限流)
      - [在分布式环境下如何实现限流？](#在分布式环境下如何实现限流)
    - 12.4 降级和熔断
      - [为什么会有容错？一般有哪些方式解决容错相关问题？](#为什么会有容错一般有哪些方式解决容错相关问题)
      - [谈谈你对服务降级的理解？](#谈谈你对服务降级的理解)
      - [什么是服务熔断？和服务降级有什么区别？](#什么是服务熔断和服务降级有什么区别)
      - [如何设计服务的熔断？](#如何设计服务的熔断)
      - [服务熔断有哪些实现方案？](#服务熔断有哪些实现方案)
    - 12.5 负载均衡
      - [什么是负载均衡？原理是什么？](#什么是负载均衡原理是什么)
      - [负载均衡有哪些分类？](#负载均衡有哪些分类)
      - [常见的负载均衡服务器有哪些？](#常见的负载均衡服务器有哪些)
      - [常见的负载均衡的算法？](#常见的负载均衡的算法)
    - 12.6 灾备和故障转移
      - [什么是容灾？一般基于什么实现？](#什么是容灾一般基于什么实现)
      - [一般怎么实现灾备？](#一般怎么实现灾备)
  - 13 分布式
    - 13.1 一致性算法
      - [什么是分布式系统的副本一致性？有哪些？](#什么是分布式系统的副本一致性有哪些)
      - [在分布式系统中有哪些常见的一致性算法？](#在分布式系统中有哪些常见的一致性算法)
      - [谈谈你对一致性hash算法的理解？](#谈谈你对一致性hash算法的理解)
      - [什么是Paxos算法？ 如何实现的？](#什么是paxos算法-如何实现的)
      - [什么是Raft算法？](#什么是raft算法)
    - 13.2 全局唯一ID
      - [全局唯一ID有哪些实现方案？](#全局唯一id有哪些实现方案)
      - [数据库方式实现方案？有什么缺陷？](#数据库方式实现方案有什么缺陷)
      - [雪花算法如何实现的？](#雪花算法如何实现的)
      - [雪花算法有什么问题？有哪些解决思路？](#雪花算法有什么问题有哪些解决思路)
    - 13.3 分布式锁
      - [有哪些方案实现分布式锁？](#有哪些方案实现分布式锁)
      - [基于数据库如何实现分布式锁？有什么缺陷？](#基于数据库如何实现分布式锁有什么缺陷)
      - [基于redis如何实现分布式锁？有什么缺陷？](#基于redis如何实现分布式锁有什么缺陷)
      - [基于zookeeper如何实现分布式锁？](#基于zookeeper如何实现分布式锁)
    - 13.4 分布式事务
      - [什么是ACID？](#什么是acid)
      - [分布式事务有哪些解决方案？](#分布式事务有哪些解决方案)
      - [什么是分布式的XA协议？](#什么是分布式的xa协议)
      - [什么是2PC？](#什么是2pc)
      - [什么是3PC?](#什么是3pc)
      - [什么是TCC？](#什么是tcc)
      - [什么是SAGA方案？](#什么是saga方案)
    - 13.5 分布式缓存
      - [分布式系统中常用的缓存方案有哪些？](#分布式系统中常用的缓存方案有哪些)
      - [分布式系统缓存的更新模式？](#分布式系统缓存的更新模式)
      - [分布式系统缓存淘汰策略](#分布式系统缓存淘汰策略)
    - 13.6 分布式任务
      - [Java中定时任务是有些？如何演化的？](#java中定时任务是有些如何演化的)
      - [常见的JOB实现方案？](#常见的job实现方案)
    - 13.7 分布式会话
      - [Cookie和Session有什么区别？](#cookie和session有什么区别)
      - [谈谈会话技术的发展？](#谈谈会话技术的发展)
      - [分布式会话有哪些解决方案？](#分布式会话有哪些解决方案)
      - [什么是Session Stick?](#什么是session-stick)
      - [什么是Session Replication？](#什么是session-replication)
      - [什么是Session 数据集中存储？](#什么是session-数据集中存储)
      - [什么是Cookie Based Session?](#什么是cookie-based-session)
      - [什么是JWT？使用JWT的流程？对比传统的会话有啥区别？](#什么是jwt使用jwt的流程对比传统的会话有啥区别)
    - 13.8 常见系统设计
      - [如何设计一个秒杀系统？](#如何设计一个秒杀系统)
      - [接口设计要考虑哪些方面？](#接口设计要考虑哪些方面)
      - [什么是接口幂等？如何保证接口的幂等性？](#什么是接口幂等如何保证接口的幂等性)
  - 14 微服务
    - 14.1 Spring Cloud
      - [什么是微服务？谈谈你对微服务的理解？](#什么是微服务谈谈你对微服务的理解)
      - [什么是Spring Cloud？](#什么是spring-cloud)
      - [springcloud中的组件有那些？](#springcloud中的组件有那些)
      - [具体说说SpringCloud主要项目?](#具体说说springcloud主要项目)
      - [Spring Cloud项目部署架构？](#spring-cloud项目部署架构)
      - [Spring Cloud 和dubbo区别?](#spring-cloud-和dubbo区别)
      - [服务注册和发现是什么意思？Spring Cloud 如何实现？](#服务注册和发现是什么意思spring-cloud-如何实现)
      - [什么是Eureka？](#什么是eureka)
      - [Eureka怎么实现高可用？](#eureka怎么实现高可用)
      - [什么是Eureka的自我保护模式？](#什么是eureka的自我保护模式)
      - [DiscoveryClient的作用？](#discoveryclient的作用)
      - [Eureka和ZooKeeper都可以提供服务注册与发现的功能,请说说两个的区别？](#eureka和zookeeper都可以提供服务注册与发现的功能请说说两个的区别)
      - [什么是网关?](#什么是网关)
      - [网关的作用是什么？](#网关的作用是什么)
      - [什么是Spring Cloud Zuul（服务网关）？](#什么是spring-cloud-zuul服务网关)
      - [网关与过滤器有什么区别？](#网关与过滤器有什么区别)
      - [常用网关框架有那些？](#常用网关框架有那些)
      - [Zuul与Nginx有什么区别？](#zuul与nginx有什么区别)
      - [既然Nginx可以实现网关？为什么还需要使用Zuul框架?](#既然nginx可以实现网关为什么还需要使用zuul框架)
      - [ZuulFilter常用有那些方法?](#zuulfilter常用有那些方法)
      - [如何实现动态Zuul网关路由转发?](#如何实现动态zuul网关路由转发)
      - [Zuul网关如何搭建集群?](#zuul网关如何搭建集群)
      - [Ribbon是什么？](#ribbon是什么)
      - [Nginx与Ribbon的区别？](#nginx与ribbon的区别)
      - [Ribbon底层实现原理？](#ribbon底层实现原理)
      - [@LoadBalanced注解的作用？](#loadbalanced注解的作用)
      - [什么是断路器](#什么是断路器)
      - [什么是 Hystrix？](#什么是-hystrix)
      - [什么是Feign？](#什么是feign)
      - [SpringCloud有几种调用接口方式？](#springcloud有几种调用接口方式)
      - [Ribbon和Feign调用服务的区别？](#ribbon和feign调用服务的区别)
      - [什么是 Spring Cloud Bus？](#什么是-spring-cloud-bus)
      - [什么是Spring Cloud Config?](#什么是spring-cloud-config)
      - [分布式配置中心有那些框架？](#分布式配置中心有那些框架)
      - [分布式配置中心的作用？](#分布式配置中心的作用)
      - [SpringCloud Config 可以实现实时刷新吗？](#springcloud-config-可以实现实时刷新吗)
      - [什么是Spring Cloud Gateway?](#什么是spring-cloud-gateway)
    - 14.2 Kubernetes
      - [什么是Kubernetes? Kubernetes与Docker有什么关系？](#什么是kubernetes-kubernetes与docker有什么关系)
      - [Kubernetes的整体架构？](#kubernetes的整体架构)
      - [Kubernetes中有哪些核心概念？](#kubernetes中有哪些核心概念)
      - [什么是Heapster？](#什么是heapster)
      - [什么是Minikube？](#什么是minikube)
      - [什么是Kubectl？](#什么是kubectl)
      - [kube-apiserver和kube-scheduler的作用是什么？](#kube-apiserver和kube-scheduler的作用是什么)
      - [请你说一下kubenetes针对pod资源对象的健康监测机制？](#请你说一下kubenetes针对pod资源对象的健康监测机制)
      - [K8s中镜像的下载策略是什么？](#k8s中镜像的下载策略是什么)
      - [image的状态有哪些？](#image的状态有哪些)
      - [如何控制滚动更新过程？](#如何控制滚动更新过程)
      - [DaemonSet资源对象的特性？](#daemonset资源对象的特性)
      - [说说你对Job这种资源对象的了解？](#说说你对job这种资源对象的了解)
      - [pod的重启策略是什么？](#pod的重启策略是什么)
      - [描述一下pod的生命周期有哪些状态？](#描述一下pod的生命周期有哪些状态)
      - [创建一个pod的流程是什么？](#创建一个pod的流程是什么)
      - [删除一个Pod会发生什么事情？](#删除一个pod会发生什么事情)
      - [K8s的Service是什么？](#k8s的service是什么)
      - [k8s是怎么进行服务注册的？](#k8s是怎么进行服务注册的)
      - [k8s集群外流量怎么访问Pod？](#k8s集群外流量怎么访问pod)
      - [k8s数据持久化的方式有哪些？](#k8s数据持久化的方式有哪些)
      - [Replica Set 和 Replication Controller 之间有什么区别？](#replica-set-和-replication-controller-之间有什么区别)
      - [其它](#其它)
    - 14.3 Service Mesh
      - [什么是Service Mesh（服务网格）？](#什么是service-mesh服务网格)
      - [什么是Istio?](#什么是istio)
      - [Istio的架构？](#istio的架构)
  - 15 DevOps
    - 15.1 Linux
      - [什么是Linux？](#什么是linux)
      - [UNIX和LINUX有什么区别？](#unix和linux有什么区别)
      - [什么是BASH？](#什么是bash)
      - [什么是Linux内核？](#什么是linux内核)
      - [什么是LILO？](#什么是lilo)
      - [什么是交换空间？](#什么是交换空间)
      - [Linux的基本组件是什么？](#linux的基本组件是什么)
      - [Linux系统安装多个桌面环境有帮助吗？](#linux系统安装多个桌面环境有帮助吗)
      - [BASH和DOS之间的基本区别是什么？](#bash和dos之间的基本区别是什么)
      - [GNU项目的重要性是什么？](#gnu项目的重要性是什么)
      - [描述root帐户？](#描述root帐户)
      - [如何在发出命令时打开命令提示符？](#如何在发出命令时打开命令提示符)
      - [如何知道Linux使用了多少内存？](#如何知道linux使用了多少内存)
      - [Linux系统下交换分区的典型大小是多少？](#linux系统下交换分区的典型大小是多少)
      - [什么是符号链接？](#什么是符号链接)
      - [Ctrl + Alt + Del组合键是否适用于Linux？](#ctrl--alt--del组合键是否适用于linux)
      - [如何引用连接打印机等设备的并行端口？](#如何引用连接打印机等设备的并行端口)
      - [硬盘驱动器和软盘驱动器等驱动器是否用驱动器号表示？](#硬盘驱动器和软盘驱动器等驱动器是否用驱动器号表示)
      - [如何在Linux下更改权限？](#如何在linux下更改权限)
      - [在Linux中，为不同的串口分配了哪些名称？](#在linux中为不同的串口分配了哪些名称)
      - [如何在Linux下访问分区？](#如何在linux下访问分区)
      - [什么是硬链接？](#什么是硬链接)
      - [Linux下文件名的最大长度是多少？](#linux下文件名的最大长度是多少)
      - [什么是以点开头的文件名？](#什么是以点开头的文件名)
      - [解释虚拟桌面?](#解释虚拟桌面)
      - [如何在Linux下跨不同的虚拟桌面共享程序？](#如何在linux下跨不同的虚拟桌面共享程序)
      - [无名（空）目录代表什么？](#无名空目录代表什么)
      - [什么是pwd命令？](#什么是pwd命令)
      - [什么是守护进程？](#什么是守护进程)
      - [如何从一个桌面环境切换到另一个桌面环境，例如从KDE切换到Gnome？](#如何从一个桌面环境切换到另一个桌面环境例如从kde切换到gnome)
      - [Linux下的权限有哪些？](#linux下的权限有哪些)
      - [区分大小写如何影响命令的使用方式？](#区分大小写如何影响命令的使用方式)
      - [是否可以使用快捷方式获取长路径名？](#是否可以使用快捷方式获取长路径名)
      - [什么是重定向？](#什么是重定向)
      - [什么是grep命令？](#什么是grep命令)
      - [当发出的命令与上次使用时产生的结果不同时，会出现什么问题？](#当发出的命令与上次使用时产生的结果不同时会出现什么问题)
      - [/ usr / local的内容是什么？](#-usr--local的内容是什么)
      - [你如何终止正在进行的流程？](#你如何终止正在进行的流程)
      - [如何在命令行提示符中插入注释？](#如何在命令行提示符中插入注释)
      - [什么是命令分组以及它是如何工作的？](#什么是命令分组以及它是如何工作的)
      - [如何从单个命令行条目执行多个命令或程序？](#如何从单个命令行条目执行多个命令或程序)
      - [编写一个命令，查找扩展名为“c”的文件，并在其中出现字符串“apple”?](#编写一个命令查找扩展名为c的文件并在其中出现字符串apple)
      - [编写一个显示所有.txt文件的命令，包括其个人权限。](#编写一个显示所有txt文件的命令包括其个人权限)
      - [解释如何为Git控制台着色？](#解释如何为git控制台着色)
      - [如何在Linux中将一个文件附加到另一个文件？](#如何在linux中将一个文件附加到另一个文件)
      - [解释如何使用终端找到文件？](#解释如何使用终端找到文件)
      - [解释如何使用终端创建文件夹？](#解释如何使用终端创建文件夹)
      - [解释如何使用终端查看文本文件？](#解释如何使用终端查看文本文件)
      - [解释如何在Ubuntu LAMP堆栈上启用curl？](#解释如何在ubuntu-lamp堆栈上启用curl)
      - [解释如何在Ubuntu中启用root日志记录？](#解释如何在ubuntu中启用root日志记录)
      - [如何在启动Linux服务器的同时在后台运行Linux程序？](#如何在启动linux服务器的同时在后台运行linux程序)
      - [解释如何在Linux中卸载库？](#解释如何在linux中卸载库)
    - 15.2 Docker
      - [什么是虚拟化技术？](#什么是虚拟化技术)
      - [什么是Docker?](#什么是docker)
      - [Docker和虚拟机的区别？](#docker和虚拟机的区别)
      - [Docker的架构？](#docker的架构)
      - [Docker镜像相关操作有哪些？](#docker镜像相关操作有哪些)
      - [Docker容器相关操作有哪些？](#docker容器相关操作有哪些)
      - [如何查看Docker容器的日志？](#如何查看docker容器的日志)
      - [如何启动Docker容器？参数含义？](#如何启动docker容器参数含义)
      - [如何进入Docker后台模式？有什么区别？](#如何进入docker后台模式有什么区别)
    - 15.3 CI/CD
      - [什么是CI？](#什么是ci)
      - [什么是CD？](#什么是cd)
      - [什么是CI/CD的管道？](#什么是cicd的管道)
      - [如何理解DevOPS?](#如何理解devops)
      - [在完全部署到所有用户之前，有哪些方法可以测试部署？](#在完全部署到所有用户之前有哪些方法可以测试部署)
      - [什么是持续测试？](#什么是持续测试)
      - [如何做版本管理？](#如何做版本管理)
    - 15.4 监控体系
      - [为什么要有监控系统？ 谈谈你对监控的理解？](#为什么要有监控系统-谈谈你对监控的理解)
      - [监控体系监控哪些内容？](#监控体系监控哪些内容)
      - [监控一般采用什么样的流程？](#监控一般采用什么样的流程)
  - 16 其它
    - [16.1 设计模式](#161-设计模式)
    - 16.2 开源协议
      - [说说常见的开源协议？](#说说常见的开源协议)
      - [GPL协议、LGPL协议与BSD协议的法律区别？](#gpl协议lgpl协议与bsd协议的法律区别)
      - [MongoDB修改开源协议？](#mongodb修改开源协议)
    - 16.3 软件理论
      - [什么是CAP理论？](#什么是cap理论)
      - [什么是BASE理论？](#什么是base理论)
      - [什么是SOLID原则？](#什么是solid原则)
      - [什么是合成/聚合复用原则？](#什么是合成聚合复用原则)
      - [什么是迪米特法则？](#什么是迪米特法则)
      - [什么是康威定律？](#什么是康威定律)
    - 16.4 软件成熟度模型
      - [什么是CMM？](#什么是cmm)
      - [什么是CMMI5 呢？](#什么是cmmi5-呢)
      - [CMMI与CMM的区别呢？](#cmmi与cmm的区别呢)
      - [CMM与ISO9000的主要区别？](#cmm与iso9000的主要区别)
    - 16.5 等级保护
      - [为什么是做等级保护？](#为什么是做等级保护)
      - [等级保护分为哪些等级？](#等级保护分为哪些等级)
      - [怎么做等级保护？](#怎么做等级保护)
      - [等保三的基本要求?](#等保三的基本要求)
    - 16.6 ISO27001
      - [什么是ISO27001？](#什么是iso27001)
      - [ISO27001认证流程?](#iso27001认证流程)

## [#](#_9-开发基础) 9 开发基础

> 开发基础相关。

### [#](#_9-1-常用类库) 9.1 常用类库

#### [#](#平时常用的开发工具库有哪些) 平时常用的开发工具库有哪些？

- Apache Common
  - Apache Commons是对JDK的拓展，包含了很多开源的工具，用于解决平时编程经常会遇到的问题，减少重复劳动。
- Google Guava
  - Guava工程包含了若干被Google的 Java项目广泛依赖 的核心库，例如：集合 [collections] 、缓存 [caching] 、原生类型支持 [primitives support] 、并发库 [concurrency libraries] 、通用注解 [common annotations] 、字符串处理 [string processing] 、I/O 等等。 所有这些工具每天都在被Google的工程师应用在产品服务中。
- Hutool
  - 国产后起之秀，Hutool是一个小而全的Java工具类库，通过静态方法封装，降低相关API的学习成本，提高工作效率
- Spring常用工具类
  - Spring作为常用的开发框架，在Spring框架应用中，排在ApacheCommon，Guava, Huool等通用库后，第二优先级可以考虑使用Spring-core-xxx.jar中的util包

#### [#](#java常用的json库有哪些-有啥注意点) Java常用的JSON库有哪些？有啥注意点？

- FastJSON（不推荐，漏洞太多）
- Jackson
- Gson 
  - 序列化
  - 反序列化
  - 自定义序列化和反序列化

#### [#](#lombok工具库用来解决什么问题) Lombok工具库用来解决什么问题？

我们通常需要编写大量代码才能使类变得有用。如以下内容：

- `toString()`方法
- `hashCode()` and `equals()`方法
- `Getter` and `Setter` 方法
- 构造函数

对于这种简单的类，这些方法通常是无聊的、重复的，而且是可以很容易地机械地生成的那种东西(ide通常提供这种功能)。

- `@Getter/@Setter`示例

```java
@Setter(AccessLevel.PUBLIC)
@Getter(AccessLevel.PROTECTED)
private int id;
private String shap;
```

- `@ToString`示例

```java
@ToString(exclude = "id", callSuper = true, includeFieldNames = true)
public class LombokDemo {
    private int id;
    private String name;
    private int age;
    public static void main(String[] args) {
        //输出LombokDemo(super=LombokDemo@48524010, name=null, age=0)
        System.out.println(new LombokDemo());
    }
}
```

- `@EqualsAndHashCode`示例

```java
@EqualsAndHashCode(exclude = {"id", "shape"}, callSuper = false)
public class LombokDemo {
    private int id;
    private String shap;
}
```

#### [#](#为什么很多公司禁止使用lombok) 为什么很多公司禁止使用lombok？

可以使用而且有着广泛的使用，但是需要理解部分注解的底层和潜在问题，否则会有坑：

- `@Data`： 如果只使用了`@Data`，而不使用`@EqualsAndHashCode(callSuper=true)`的话，会默认是`@EqualsAndHashCode(callSuper=false)`,这时候生成的`equals()`方法只会比较子类的属性，不会考虑从父类继承的属性，无论父类属性访问权限是否开放。
- **代码可读性，可调试性低** 在代码中使用了Lombok，确实可以帮忙减少很多代码，因为Lombok会帮忙自动生成很多代码。但是**这些代码是要在编译阶段才会生成的**，所以在开发的过程中，其实很多代码其实是缺失的。
- **Lombok有很强的侵入性**
  - 强J队友，如果项目组中有一个人使用了Lombok，那么其他人就必须也要安装IDE插件。
  - 如果我们需要升级到某个新版本的JDK的时候，若其中的特性在Lombok中不支持的话就会受到影响
- **Lombok破坏了封装性**

举个简单的例子，我们定义一个购物车类：

```java
@Data
public class ShoppingCart { 

    //商品数目
    private int itemsCount; 

    //总价格
    private double totalPrice; 

    //商品明细
    private List items = new ArrayList<>();

}

//例子来源于《极客时间-设计模式之美》
```

我们知道，购物车中商品数目、商品明细以及总价格三者之前其实是有关联关系的，如果需要修改的话是要一起修改的。

但是，我们使用了Lombok的`@Data`注解，对于itemsCount 和 totalPrice这两个属性。虽然我们将它们定义成 `private` 类型，但是提供了 `public` 的 `getter`、`setter` 方法。

外部可以通过 `setter` 方法随意地修改这两个属性的值。我们可以随意调用 `setter` 方法，来重新设置 itemsCount、totalPrice 属性的值，这也会导致其跟 items 属性的值不一致。

而面向对象封装的定义是：通过访问权限控制，隐藏内部数据，外部仅能通过类提供的有限的接口访问、修改内部数据。所以，暴露不应该暴露的 setter 方法，明显违反了面向对象的封装特性。

好的做法应该是不提供`getter/setter`，而是只提供一个public的addItem方法，同时去修改itemsCount、totalPrice以及items三个属性。（所以不能一股脑使用@Data注解）

- 此外，**Java14 提供的record语法糖**，来解决类似问题

```java
public record Range(int min, int max) {}
```

#### [#](#mapstruct工具库用来解决什么问题) MapStruct工具库用来解决什么问题？

MapStruct是一款非常实用Java工具，主要用于解决对象之间的拷贝问题，比如PO/DTO/VO/QueryParam之间的转换问题。区别于BeanUtils这种通过反射，它通过编译器编译生成常规方法，将可以很大程度上提升效率。

举例：

```java
@Mapper
public interface UserConverter {
    UserConverter INSTANCE = Mappers.getMapper(UserConverter.class);

    @Mapping(target = "gender", source = "sex")
    @Mapping(target = "createTime", dateFormat = "yyyy-MM-dd HH:mm:ss")
    UserVo do2vo(User var1);

    @Mapping(target = "sex", source = "gender")
    @Mapping(target = "password", ignore = true)
    @Mapping(target = "createTime", dateFormat = "yyyy-MM-dd HH:mm:ss")
    User vo2Do(UserVo var1);

    List<UserVo> do2voList(List<User> userList);

    default List<UserVo.UserConfig> strConfigToListUserConfig(String config) {
        return JSON.parseArray(config, UserVo.UserConfig.class);
    }

    default String listUserConfigToStrConfig(List<UserVo.UserConfig> list) {
        return JSON.toJSONString(list);
    }
}
```

#### [#](#lombok和mapstruct工具库的原理) Lombok和MapStruct工具库的原理？

会发现在Lombok使用的过程中，只需要添加相应的注解，无需再为此写任何代码。自动生成的代码到底是如何产生的呢？

核心之处就是对于注解的解析上。JDK5引入了注解的同时，也提供了两种解析方式。

- **运行时解析**

运行时能够解析的注解，必须将@Retention设置为RUNTIME, 比如`@Retention(RetentionPolicy.RUNTIME)`，这样就可以通过反射拿到该注解。java.lang,reflect反射包中提供了一个接口AnnotatedElement，该接口定义了获取注解信息的几个方法，Class、Constructor、Field、Method、Package等都实现了该接口，对反射熟悉的朋友应该都会很熟悉这种解析方式。

- **编译时解析**

编译时解析有两种机制，分别简单描述下：

1）Annotation Processing Tool

apt自JDK5产生，JDK7已标记为过期，不推荐使用，JDK8中已彻底删除，自JDK6开始，可以使用Pluggable Annotation Processing API来替换它，apt被替换主要有2点原因：

- api都在com.sun.mirror非标准包下
- 没有集成到javac中，需要额外运行

2）Pluggable Annotation Processing API

[JSR 269: Pluggable Annotation Processing API在新窗口打开](https://www.jcp.org/en/jsr/proposalDetails?id=269)自JDK6加入，作为apt的替代方案，它解决了apt的两个问题，javac在执行的时候会调用实现了该API的程序，这样我们就可以对编译器做一些增强，这时javac执行的过程如下：

![img](/images/develop/package/dev-package-lombok-2.png)

Lombok本质上就是一个实现了“JSR 269 API”的程序。在使用javac的过程中，它产生作用的具体流程如下：

- javac对源代码进行分析，生成了一棵抽象语法树（AST）
- 运行过程中调用实现了“JSR 269 API”的Lombok程序
- 此时Lombok就对第一步骤得到的AST进行处理，找到@Data注解所在类对应的语法树（AST），然后修改该语法树（AST），增加getter和setter方法定义的相应树节点
- javac使用修改后的抽象语法树（AST）生成字节码文件，即给class增加新的节点（代码块）

![img](/images/develop/package/dev-package-lombok-3.png)

从上面的Lombok执行的流程图中可以看出，在Javac 解析成AST抽象语法树之后, Lombok 根据自己编写的注解处理器，动态地修改 AST，增加新的节点（即Lombok自定义注解所需要生成的代码），最终通过分析生成JVM可执行的字节码Class文件。使用Annotation Processing自定义注解是在编译阶段进行修改，而JDK的反射技术是在运行时动态修改，两者相比，反射虽然更加灵活一些但是带来的性能损耗更加大。

### [#](#_9-2-网络协议和工具) 9.2 网络协议和工具

#### [#](#什么是754层网络模型) 什么是754层网络模型？

全局上理解 `7层协议，4层，5层`的对应关系。

![img](/images/develop/network/dev-network-protocol-1.png)

OSI依层次结构来划分：应用层（Application）、表示层（Presentation）、会话层（Session）、传输层（Transport）、网络层（Network）、数据链路层（Data Link）、物理层（Physical）

#### [#](#tcp建立连接过程的三次握手) TCP建立连接过程的三次握手？

TCP有6种标识：SYN(建立联机) ACK(确认) PSH(传送) FIN(结束) RST(重置) URG(紧急)； 然后我们来看三次握手

- **什么是三次握手**？

![img](/images/develop/network/dev-network-protocol-x1.png)

为了保证数据能到达目标，TCP采用三次握手策略：

1. 发送端首先发送一个带**SYN**（synchronize）标志的数据包给接收方【第一次的seq序列号是随机产生的，这样是为了网络安全，如果不是随机产生初始序列号，黑客将会以很容易的方式获取到你与其他主机之间的初始化序列号，并且伪造序列号进行攻击】
2. 接收端收到后，回传一个带有**SYN/ACK**（acknowledgement）标志的数据包以示传达确认信息【SYN 是为了告诉发送端，发送方到接收方的通道没问题；ACK 用来验证接收方到发送方的通道没问题】
3. 最后，发送端再回传一个带ACK标志的数据包，代表握手结束若在握手某个过程中某个阶段莫名中断，TCP协议会再次以相同的顺序发送相同的数据包

- **为什么要三次握手**？

三次握手的目的是建立可靠的通信信道，说到通讯，简单来说就是数据的发送与接收，而三次握手最主要的目的就是双方确认自己与对方的发送与接收是正常的

1. 第一次握手，发送端：什么都确认不了；接收端：对方发送正常，自己接受正常
2. 第二次握手，发送端：对方发送，接受正常，自己发送，接受正常 ；接收端：对方发送正常，自己接受正常
3. 第三次握手，发送端：对方发送，接受正常，自己发送，接受正常；接收端：对方发送，接受正常，自己发送，接受正常

- **两次握手不行吗？为什么TCP客户端最后还要发送一次确认呢**？

主要防止已经失效的连接请求报文突然又传送到了服务器，从而产生错误。经典场景：客户端发送了第一个请求连接并且没有丢失，只是因为在网络结点中滞留的时间太长了。

1. 由于TCP的客户端迟迟没有收到确认报文，以为服务器没有收到，此时重新向服务器发送这条报文，此后客户端和服务器经过两次握手完成连接，传输数据，然后关闭连接。
2. 此时此前滞留的那一次请求连接，网络通畅了到达服务器，这个报文本该是失效的，但是，两次握手的机制将会让客户端和服务器再次建立连接，这将导致不必要的错误和资源的浪费。
3. 如果采用的是三次握手，就算是那一次失效的报文传送过来了，服务端接受到了那条失效报文并且回复了确认报文，但是客户端不会再次发出确认。由于服务器收不到确认，就知道客户端并没有请求连接。

- **为什么三次握手，返回时，ack 值是 seq 加 1（ack = x+1）**

1. 假设对方接收到数据，比如sequence number = 1000，TCP Payload = 1000，数据第一个字节编号为1000，最后一个为1999，回应一个确认报文，确认号为2000，意味着编号2000前的字节接收完成，准备接收编号为2000及更多的数据
2. 确认收到的序列，并且告诉发送端下一次发送的序列号从哪里开始（便于接收方对数据排序，便于选择重传）

- **TCP三次握手中，最后一次回复丢失，会发生什么**？

1. 如果最后一次ACK在网络中丢失，那么Server端（服务端）该TCP连接的状态仍为SYN_RECV，并且根据 TCP的超时重传机制依次等待3秒、6秒、12秒后重新发送 SYN+ACK 包，以便 Client（客户端）重新发送ACK包
2. 如果重发指定次数后，仍然未收到ACK应答，那么一段时间后，Server（服务端）自动关闭这个连接
3. 但是Client（客户端）认为这个连接已经建立，如果Client（客户端）端向Server（服务端）发送数据，Server端（服务端）将以RST包（Reset，标示复位，用于异常的关闭连接）响应，此时，客户端知道第三次握手失败

#### [#](#syn洪泛攻击-syn-flood-半开放攻击-怎么解决) SYN洪泛攻击(SYN Flood，半开放攻击)，怎么解决？

- **什么是SYN洪范泛攻击**？

SYN Flood利用TCP协议缺陷，发送大量伪造的TCP连接请求，常用假冒的IP或IP号段发来海量的请求连接的第一个握手包（SYN包），被攻击服务器回应第二个握手包（SYN+ACK包），因为对方是假冒IP，对方永远收不到包且不会回应第三个握手包。导致被攻击服务器保持大量SYN_RECV状态的“半连接”，并且会重试默认5次回应第二个握手包，大量随机的恶意syn占满了未完成连接队列，导致正常合法的syn排不上队列，让正常的业务请求连接不进来。【服务器端的资源分配是在二次握手时分配的，而客户端的资源是在完成三次握手时分配的，所以服务器容易受到SYN洪泛攻击】

- **如何检测 SYN 攻击？**

当你在服务器上看到大量的半连接状态时，特别是源IP地址是随机的，基本上可以断定这是一次SYN攻击【在 Linux/Unix 上可以使用系统自带的 netstats 命令来检测 SYN 攻击】

- **怎么解决**？ SYN攻击不能完全被阻止，除非将TCP协议重新设计。我们所做的是尽可能的减轻SYN攻击的危害，

1. 缩短超时（SYN Timeout）时间
2. 增加最大半连接数
3. 过滤网关防护
4. SYN cookies技术： 
   1. 当服务器接受到 SYN 报文段时，不直接为该 TCP 分配资源，而只是打开一个半开的套接字。接着会使用 SYN 报文段的源 Id，目的 Id，端口号以及只有服务器自己知道的一个秘密函数生成一个 cookie，并把 cookie 作为序列号响应给客户端。
   2. 如果客户端是正常建立连接，将会返回一个确认字段为 cookie + 1 的报文段。接下来服务器会根据确认报文的源 Id，目的 Id，端口号以及秘密函数计算出一个结果，如果结果的值 + 1 等于确认字段的值，则证明是刚刚请求连接的客户端，这时候才为该 TCP 分配资源

#### [#](#tcp断开连接过程的四次挥手) TCP断开连接过程的四次挥手？

- **什么是四次挥手**？

![img](/images/develop/network/dev-network-protocol-x2.png)

1. 主动断开方（客户端/服务端）-发送一个 FIN，用来关闭主动断开方（客户端/服务端）到被动断开方（客户端/服务端）的数据传送
2. 被动断开方（客户端/服务端）-收到这个 FIN，它发回一 个 ACK，确认序号为收到的序号加1 。和 SYN 一样，一个 FIN 将占用一个序号
3. 被动断开方（客户端/服务端）-关闭与主动断开方（客户端/服务端）的连接，发送一个FIN给主动断开方（客户端/服务端）
4. 主动断开方（客户端/服务端）-发回 ACK 报文确认，并将确认序号设置为收到序号加1

- **为什么连接的时候是三次握手，关闭的时候却是四次握手**？

1. 建立连接的时候， 服务器在LISTEN状态下，收到建立连接请求的SYN报文后，把ACK和SYN放在一个报文里发送给客户端。
2. 关闭连接时，服务器收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，而自己也未必全部数据都发送给对方了,所以服务器可以立即关闭，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接。因此，服务器ACK和FIN一般都会分开发送，从而导致多了一次。

- **为什么TCP挥手每两次中间有一个 FIN-WAIT2等待时间**？

主动关闭的一端调用完close以后（即发FIN给被动关闭的一端， 并且收到其对FIN的确认ACK）则进入FIN_WAIT_2状态。如果这个时候因为网络突然断掉、被动关闭的一段宕机等原因，导致主动关闭的一端不能收到被动关闭的一端发来的FIN（防止对端不发送关闭连接的FIN包给本端），这个时候就需要FIN_WAIT_2定时器， 如果在该定时器超时的时候，还是没收到被动关闭一端发来的FIN，那么直接释放这个链接，进入CLOSE状态

- **为什么客户端最后还要等待2MSL？为什么还有个TIME-WAIT的时间等待**？

1. 保证客户端发送的最后一个ACK报文能够到达服务器，因为这个ACK报文可能丢失，服务器已经发送了FIN+ACK报文，请求断开，客户端却没有回应，于是服务器又会重新发送一次，而客户端就能在这个2MSL时间段内收到这个重传的报文，接着给出回应报文，并且会重启2MSL计时器。
2. 防止类似与“三次握手”中提到了的“已经失效的连接请求报文段”出现在本连接中。客户端发送完最后一个确认报文后，在这个2MSL时间中，就可以使本连接持续的时间内所产生的所有报文段都从网络中消失，这样新的连接中不会出现旧连接的请求报文。
3. 2MSL，最大报文生存时间，一个MSL 30 秒，2MSL = 60s

- **客户端 TIME-WAIT 状态过多会产生什么后果？怎样处理**？

1. 作为服务器，短时间内关闭了大量的Client连接，就会造成服务器上出现大量的TIME_WAIT连接，占据大量的tuple /tApl/ ，严重消耗着服务器的资源，此时部分客户端就会显示连接不上
2. 作为客户端，短时间内大量的短连接，会大量消耗的Client机器的端口，毕竟端口只有65535个，端口被耗尽了，后续就无法在发起新的连接了
3. 在高并发短连接的TCP服务器上，当服务器处理完请求后立刻主动正常关闭连接。这个场景下会出现大量socket处于TIME_WAIT状态。如果客户端的并发量持续很高，此时部分客户端就会显示连接不上
   1. 高并发可以让服务器在短时间范围内同时占用大量端口，而端口有个0~65535的范围，并不是很多，刨除系统和其他服务要用的，剩下的就更少了
   2. 短连接表示“业务处理+传输数据的时间 远远小于 TIMEWAIT超时的时间”的连接
4. 解决方法：
   1. 用负载均衡来抗这些高并发的短请求；
   2. 服务器可以设置 SO_REUSEADDR 套接字选项来避免 TIME_WAIT状态，TIME_WAIT 状态可以通过优化服务器参数得到解决，因为发生TIME_WAIT的情况是服务器自己可控的，要么就是对方连接的异常，要么就是自己没有迅速回收资源，总之不是由于自己程序错误导致的
   3. 强制关闭，发送 RST 包越过TIMEWAIT状态，直接进入CLOSED状态

- **服务器出现了大量 CLOSE_WAIT 状态如何解决**？

大量 CLOSE_WAIT 表示程序出现了问题，对方的 socket 已经关闭连接，而我方忙于读或写没有及时关闭连接，需要检查代码，特别是释放资源的代码，或者是处理请求的线程配置。

- **服务端会有一个TIME_WAIT状态吗？如果是服务端主动断开连接呢**？

1. 发起链接的主动方基本都是客户端，但是断开连接的主动方服务器和客户端都可以充当，也就是说，只要是主动断开连接的，就会有 TIME_WAIT状态
2. 四次挥手是指断开一个TCP连接时，需要客户端和服务端总共发送4个包以确认连接的断开。在socket编程中，这一过程由客户端或服务端任一方执行close来触发
3. 由于TCP连接时全双工的，因此，每个方向的数据传输通道都必须要单独进行关闭。

#### [#](#dns-解析流程) DNS 解析流程？

.com.fi国际金融域名DNS解析的步骤一共分为9步，如果每次解析都要走完9个步骤，大家浏览网站的速度也不会那么快，现在之所以能保持这么快的访问速度，其实一般的解析都是跑完第4步就可以了。除非一个地区完全是第一次访问（在都没有缓存的情况下）才会走完9个步骤，这个情况很少。

- 1、本地客户机提出域名解析请求，查找本地HOST文件后将该请求发送给本地的域名服务器。
- 2、将请求发送给本地的域名服务器。
- 3、当本地的域名服务器收到请求后，就先查询本地的缓存。
- 4、如果有该纪录项，则本地的域名服务器就直接把查询的结果返回浏览器。
- 5、如果本地DNS缓存中没有该纪录，则本地域名服务器就直接把请求发给根域名服务器。
- 6、然后根域名服务器再返回给本地域名服务器一个所查询域（根的子域）的主域名服务器的地址。
- 7、本地服务器再向上一步返回的域名服务器发送请求，然后接受请求的服务器查询自己的缓存，如果没有该纪录，则返回相关的下级的域名服务器的地址。
- 8、重复第7步，直到找到正确的纪录。
- 9、本地域名服务器把返回的结果保存到缓存，以备下一次使用，同时还将结果返回给客户机。

![img](/images/develop/network/dev-network-dns-12.png)

注意事项：

**递归查询**：在该模式下DNS服务器接收到客户机请求，必须使用一个准确的查询结果回复客户机。如果DNS服务器本地没有存储查询DNS信息，那么该服务器会询问其他服务器，并将返回的查询结果提交给客户机。

**迭代查询**：DNS所在服务器若没有可以响应的结果，会向客户机提供其他能够解析查询请求的DNS服务器地址，当客户机发送查询请求时，DNS服务器并不直接回复查询结果，而是告诉客户机另一台DNS服务器地址，客户机再向这台DNS服务器提交请求，依次循环直到返回查询的结果为止。

#### [#](#为什么dns通常基于udp) 为什么DNS通常基于UDP？

DNS通常是基于UDP的，但当数据长度大于512字节的时候，为了保证传输质量，就会使用基于TCP的实现方式

- **从数据包的数量以及占有网络资源的层面**

使用基于UDP的DNS协议只要一个请求、一个应答就好了; 而使用基于TCP的DNS协议要三次握手、发送数据以及应答、四次挥手; 明显基于TCP协议的DNS更浪费网络资源！

- **从数据一致性层面**

DNS数据包不是那种大数据包，所以使用UDP不需要考虑分包，如果丢包那么就是全部丢包，如果收到了数据，那就是收到了全部数据！所以只需要考虑丢包的情况，那就算是丢包了，重新请求一次就好了。而且DNS的报文允许填入序号字段，对于请求报文和其对应的应答报文，这个字段是相同的，通过它可以区分DNS应答是对应的哪个请求

#### [#](#什么是dns劫持) 什么是DNS劫持？

DNS劫持就是通过劫持了DNS服务器，通过某些手段取得某域名的解析记录控制权，进而修改此域名的解析结果，导致对该域名的访问由原IP地址转入到修改后的指定IP，其结果就是对特定的网址不能访问或访问的是假网址，从而实现窃取资料或者破坏原有正常服务的目的。DNS劫持通过篡改DNS服务器上的数据返回给用户一个错误的查询结果来实现的。

- **DNS劫持症状**

在某些地区的用户在成功连接宽带后，首次打开任何页面都指向ISP提供的“电信互联星空”、“网通黄页广告”等内容页面。还有就是曾经出现过用户访问Google域名的时候出现了百度的网站。这些都属于DNS劫持。

#### [#](#什么是dns污染) 什么是DNS污染？

DNS污染是一种让一般用户由于得到虚假目标主机IP而不能与其通信的方法，是一种DNS缓存投毒攻击（DNS cache poisoning）。其工作方式是：由于通常的DNS查询没有任何认证机制，而且DNS查询通常基于的UDP是无连接不可靠的协议，因此DNS的查询非常容易被篡改，通过对UDP端口53上的DNS查询进行入侵检测，一经发现与关键词相匹配的请求则立即伪装成目标域名的解析服务器（NS，Name Server）给查询者返回虚假结果。

而DNS污染则是发生在用户请求的第一步上，直接从协议上对用户的DNS请求进行干扰。

**DNS污染症状**：

目前一些被禁止访问的网站很多就是通过DNS污染来实现的，例如YouTube、Facebook等网站。

**解决方法**:

1. 对于DNS劫持，可以采用使用国外免费公用的DNS服务器解决。例如OpenDNS（208.67.222.222）或GoogleDNS（8.8.8.8）。
2. 对于DNS污染，可以说，个人用户很难单单靠设置解决，通常可以使用VPN或者域名远程解析的方法解决，但这大多需要购买付费的VPN或SSH等，也可以通过修改Hosts的方法，手动设置域名正确的IP地址。

#### [#](#为什么要dns流量监控) 为什么要DNS流量监控？

预示网络中正出现可疑或恶意代码的 DNS 组合查询或流量特征。例如：

- 1.来自伪造源地址的 DNS 查询、或未授权使用且无出口过滤地址的 DNS 查询，若同时观察到异常大的 DNS 查询量或使用 TCP 而非 UDP 进行 DNS 查询，这可能表明网络内存在被感染的主机，受到了 DDoS 攻击。
- 2.异常 DNS 查询可能是针对域名服务器或解析器（根据目标 IP 地址确定）的漏洞攻击的标志。与此同时，这些查询也可能表明网络中有不正常运行的设备。原因可能是恶意软件或未能成功清除恶意软件。
- 3.在很多情况下，DNS 查询要求解析的域名如果是已知的恶意域名，或具有域名生成算法( DGA )（与非法僵尸网络有关）常见特征的域名，或者向未授权使用的解析器发送的查询，都是证明网络中存在被感染主机的有力证据。
- 4.DNS 响应也能显露可疑或恶意数据在网络主机间传播的迹象。例如，DNS 响应的长度或组合特征可以暴露恶意或非法行为。例如，响应消息异常巨大（放大攻击），或响应消息的 Answer Section 或 Additional Section 非常可疑（缓存污染，隐蔽通道）。
- 5.针对自身域名组合的 DNS 响应，如果解析至不同于你发布在授权区域中的 IP 地址，或来自未授权区域主机的域名服务器的响应，或解析为名称错误( NXDOMAIN )的对区域主机名的肯定响应，均表明域名或注册账号可能被劫持或 DNS 响应被篡改。
- 6.来自可疑 IP 地址的 DNS 响应，例如来自分配给宽带接入网络 IP 段的地址、非标准端口上出现的 DNS 流量，异常大量的解析至短生存时间( TTL )域名的响应消息，或异常大量的包含“ name error ”( NXDOMAIN )的响应消息，往往是主机被僵尸网络控制、运行恶意软件或被感染的表现。

#### [#](#输入url-到页面加载过程) 输入URL 到页面加载过程？

1. 地址栏输入URL
2. DNS 域名解析IP
3. 请求和响应数据 
   1. 建立TCP连接（3次握手）
   2. 发送HTTP请求
   3. 服务器处理请求
   4. 返回HTTP响应结果
   5. 关闭TCP连接（4次挥手）
4. 浏览器加载，解析和渲染

下图是在数据传输过程中的工作方式，在发送端是应用层-->链路层这个方向的封包过程，每经过一层都会增加该层的头部。而接收端则是从链路层-->应用层解包的过程，每经过一层则会去掉相应的首部。

![img](/images/develop/network/dev-network-protocol-10.png)

#### [#](#如何使用netstat查看服务及监听端口) 如何使用netstat查看服务及监听端口？

`netstat -t/-u/-l/-r/-n`【显示网络相关信息,-t:TCP协议,-u:UDP协议,-l:监听,-r:路由,-n:显示IP地址和端口号】

- 查看本机监听的端口

```bash
[root@pdai-centos ~]# netstat -tlun
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State      
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN     
tcp        0      0 0.0.0.0:443             0.0.0.0:*               LISTEN                          
udp        0      0 172.21.0.14:123         0.0.0.0:*                          
udp        0      0 127.0.0.1:123           0.0.0.0:*                          
udp6       0      0 fe80::5054:ff:fe2b::123 :::*                               
udp6       0      0 ::1:123                 :::* 
```

#### [#](#如何使用tcpdump抓包) 如何使用TCPDump抓包？

tcpdump 是一款强大的网络抓包工具，它使用 libpcap 库来抓取网络数据包，这个库在几乎在所有的 Linux/Unix 中都有。

**tcpdump 的常用参数**如下：

```bash
$ tcpdump -i eth0 -nn -s0 -v port 80
```

- -i : 选择要捕获的接口，通常是以太网卡或无线网卡，也可以是 vlan 或其他特殊接口。如果该系统上只有一个网络接口，则无需指定。
- -nn : 单个 n 表示不解析域名，直接显示 IP；两个 n 表示不解析域名和端口。这样不仅方便查看 IP 和端口号，而且在抓取大量数据时非常高效，因为域名解析会降低抓取速度。
- -s0 : tcpdump 默认只会截取前 96 字节的内容，要想截取所有的报文内容，可以使用 `-s number`， number 就是你要截取的报文字节数，如果是 0 的话，表示截取报文全部内容。
- -v : 使用 `-v`，`-vv` 和 `-vvv` 来显示更多的详细信息，通常会显示更多与特定协议相关的信息。
- `port 80` : 这是一个常见的端口过滤器，表示仅抓取 80 端口上的流量，通常是 HTTP。

#### [#](#如何使用wireshark抓包分析) 如何使用Wireshark抓包分析？

Wireshark（前称Ethereal）是一个网络封包分析软件。网络封包分析软件的功能是撷取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换。

首先看下TCP报文首部，和wireshark捕获到的TCP包中的每个字段如下图所示：

![img](/images/develop/network/dev-network-wireshark-1.jpeg)

### [#](#_9-3-开发安全) 9.3 开发安全

#### [#](#开发中有哪些常见的web安全漏洞) 开发中有哪些常见的Web安全漏洞？

通过OWASP Top 10来回答

![img](/images/security/dev-security-overview-1.png)

2013版至2017版，应用程序的基础技术和结构发生了重大变化：

- 使用node.js和Spring Boot构建的微服务正在取代传统的单任务应用，微服务本身具有自己的安全挑战，包括微服务间互信、容器 工具、保密管理等等。原来没人期望代码要实现基于互联网的房屋，而现在这些代码就在API或RESTful服务的后面，提供给移动 应用或单页应用（SPA）的大量使用。代码构建时的假设，如受信任的调用等等，再也不存在了。
- 使用JavaScript框架（如：Angular和React）编写的单页应用程序，允许创建高度模块化的前端用户体验；原来交付服务器端处理 的功能现在变为由客户端处理，但也带来了安全挑战。
- JavaScript成为网页上最基本的语言。Node.js运行在服务器端，采用现代网页框架的Bootstrap、Electron、Angular和React则运 行在客户端。

#### [#](#什么是注入攻击-举例说明) 什么是注入攻击？举例说明？

- **什么是注入攻击？从具体的SQL注入说**？

重点看这条SQL，密码输入: ' OR '1'='1时，等同于不需要密码

```java
String sql = "SELECT * FROM t_user WHERE username='"+userName+"' AND pwd='"+password+"'"; 
```

- **如何解决注入攻击，比如SQL注入**？

1. **使用预编译处理输入参数**：要防御 SQL 注入，用户的输入就不能直接嵌套在 SQL 语句当中。使用参数化的语句，用户的输入就被限制于一个参数当中， 比如用prepareStatement
2. **输入验证**：检查用户输入的合法性，以确保输入的内容为正常的数据。数据检查应当在客户端和服务器端都执行，之所以要执行服务器端验证，是因为客户端的校验往往只是减轻服务器的压力和提高对用户的友好度，攻击者完全有可能通过抓包修改参数或者是获得网页的源代码后，修改验证合法性的脚本（或者直接删除脚本），然后将非法内容通过修改后的表单提交给服务器等等手段绕过客户端的校验。因此，要保证验证操作确实已经执行，唯一的办法就是在服务器端也执行验证。但是这些方法很容易出现由于过滤不严导致恶意攻击者可能绕过这些过滤的现象，需要慎重使用。
3. **错误消息处理**：防范 SQL 注入，还要避免出现一些详细的错误消息，恶意攻击者往往会利用这些报错信息来判断后台 SQL 的拼接形式，甚至是直接利用这些报错注入将数据库中的数据通过报错信息显示出来。
4. **加密处理**：将用户登录名称、密码等数据加密保存。加密用户输入的数据，然后再将它与数据库中保存的数据比较，这相当于对用户输入的数据进行了“消毒”处理，用户输入的数据不再对数据库有任何特殊的意义，从而也就防止了攻击者注入 SQL 命令。

- **还有哪些注入**？

1. xPath注入，XPath 注入是指利用 XPath 解析器的松散输入和容错特性，能够在 URL、表单或其它信息上附带恶意的 XPath 查询代码，以获得权限信息的访问权并更改这些信息
2. 命令注入，Java中`System.Runtime.getRuntime().exec(cmd);`可以在目标机器上执行命令，而构建参数的过程中可能会引发注入攻击
3. LDAP注入
4. CLRF注入
5. email注入
6. Host注入

#### [#](#什么是csrf-举例说明并给出开发中解决方案) 什么是CSRF？举例说明并给出开发中解决方案？

你这可以这么理解CSRF攻击：攻击者盗用了你的身份，以你的名义发送恶意请求。

![img](/images/security/dev-security-csrf-x1.jpeg)

- **黑客能拿到Cookie吗**?

CSRF 攻击是黑客借助受害者的 cookie 骗取服务器的信任，但是黑客并不能拿到 cookie，也看不到 cookie 的内容。

对于服务器返回的结果，由于浏览器同源策略的限制，黑客也无法进行解析。因此，黑客无法从返回的结果中得到任何东西，他所能做的就是给服务器发送请求，以执行请求中所描述的命令，在服务器端直接改变数据的值，而非窃取服务器中的数据。

- **什么样的请求是要CSRF保护**?

为什么有些框架（比如Spring Security)里防护CSRF的filter限定的Method是POST/PUT/DELETE等，而没有限定GET Method?

我们要保护的对象是那些可以直接产生数据改变的服务，而对于读取数据的服务，则不需要进行 CSRF 的保护。通常而言GET请作为请求数据，不作为修改数据，所以这些框架没有拦截Get等方式请求。比如银行系统中转账的请求会直接改变账户的金额，会遭到 CSRF 攻击，需要保护。而查询余额是对金额的读取操作，不会改变数据，CSRF 攻击无法解析服务器返回的结果，无需保护。

- **为什么对请求做了CSRF拦截，但还是会报CRSF漏洞**?

为什么我在前端已经采用POST+CSRF Token请求，后端也对POST请求做了CSRF Filter，但是渗透测试中还有CSRF漏洞?

直接看下面代码。

```java
// 这里没有限制POST Method，导致用户可以不通过POST请求提交数据。
@RequestMapping("/url")
public ReponseData saveSomething(XXParam param){
    // 数据保存操作...
}
```

PS：这一点是很容易被忽视的，在笔者经历过的几个项目的渗透测试中，多次出现。@pdai

- **有哪些CSRF 防御常规思路**？

1. **验证 HTTP Referer 字段**， 根据 HTTP 协议，在 HTTP 头中有一个字段叫 Referer，它记录了该 HTTP 请求的来源地址。只需要验证referer
2. **在请求地址中添加 token 并验证**，可以在 HTTP 请求中以参数的形式加入一个随机产生的 token，并在服务器端建立一个拦截器来验证这个 token，如果请求中没有 token 或者 token 内容不正确，则认为可能是 CSRF 攻击而拒绝该请求。 这种方法要比检查 Referer 要安全一些，token 可以在用户登陆后产生并放于 session 之中，然后在每次请求时把 token 从 session 中拿出，与请求中的 token 进行比对，但这种方法的难点在于如何把 token 以参数的形式加入请求。
3. **在 HTTP 头中自定义属性并验证**

- **开发中如何防御CSRF**？

可以通过自定义xxxCsrfFilter去拦截实现， 这里建议你参考 Spring Security - org.springframework.security.web.csrf.CsrfFilter.java。

#### [#](#什么是xss-举例说明) 什么是XSS？举例说明？

通常XSS攻击分为：`反射型xss攻击`, `存储型xss攻击` 和 `DOM型xss攻击`。同时注意以下例子只是简单的向你解释这三种类型的攻击方式而已，实际情况比这个复杂，具体可以再结合最后一节深入理解。

- **反射型xss攻击？**

反射型的攻击需要用户主动的去访问带攻击的链接，攻击者可以通过邮件或者短信的形式，诱导受害者点开链接。如果攻击者配合短链接URL，攻击成功的概率会更高。

在一个反射型XSS攻击中，恶意文本属于受害者发送给网站的请求中的一部分。随后网站又把恶意文本包含进用于响应用户的返回页面中，发还给用户。

![img](/images/security/dev-security-xss-1.png)

- **存储型xss攻击**？

这种攻击方式恶意代码会被存储在数据库中，其他用户在正常访问的情况下，也有会被攻击，影响的范围比较大。

![img](/images/security/dev-security-xss-3.png)

- **DOM型xss攻击**？

基于DOM的XSS攻击是反射型攻击的变种。服务器返回的页面是正常的，只是我们在页面执行js的过程中，会把攻击代码植入到页面中。

![img](/images/security/dev-security-xss-2.png)

- **XSS 攻击的防御**？

XSS攻击其实就是代码的注入。用户的输入被编译成恶意的程序代码。所以，为了防范这一类代码的注入，需要确保用户输入的安全性。对于攻击验证，我们可以采用以下两种措施：

1. **编码，就是转义用户的输入，把用户的输入解读为数据而不是代码**
2. **校验，对用户的输入及请求都进行过滤检查，如对特殊字符进行过滤，设置输入域的匹配规则等**。

具体比如：

1. **对于验证输入**，我们既可以在`服务端验证`，也可以在`客户端验证`
2. **对于持久性和反射型攻击**，`服务端验证`是必须的，服务端支持的任何语言都能够做到
3. **对于基于DOM的XSS攻击**，验证输入在客户端必须执行，因为从服务端来说，所有发出的页面内容是正常的，只是在客户端js代码执行的过程中才发生可攻击
4. 但是对于各种攻击方式，**我们最好做到客户端和服务端都进行处理**。

其它还有一些辅助措施，比如：

1. **入参长度限制**： 通过以上的案例我们不难发现xss攻击要能达成往往需要较长的字符串，因此对于一些可以预期的输入可以通过限制长度强制截断来进行防御。
2. 设置cookie httponly为true（具体请看下文的解释）

#### [#](#一般的渗透测试流程) 一般的渗透测试流程？

渗透测试就是利用我们所掌握的渗透知识，对网站进行一步一步的渗透，发现其中存在的漏洞和隐藏的风险，然后撰写一篇测试报告，提供给我们的客户。客户根据我们撰写的测试报告，对网站进行漏洞修补，以防止黑客的入侵！

- **渗透测试流程举例**？

我们现在就模拟黑客对一个网站进行渗透测试，这属于黑盒测试，我们只知道该网站的URL，其他什么的信息都不知道。

![img](/images/security/dev-security-flow-1.png)

- 确定目标 
  - 确定范围：测试目标的范围、ip、域名、内外网、测试账户。
  - 确定规则：能渗透到什么程度，所需要的时间、能否修改上传、能否提权、等等。
  - 确定需求：web应用的漏洞、业务逻辑漏洞、人员权限管理漏洞、等等。
- 信息收集 
  - 方式：主动扫描，开放搜索等。
  - 开放搜索：利用搜索引擎获得：后台、未授权页面、敏感url、等等。
  - 基础信息：IP、网段、域名、端口。
  - 应用信息：各端口的应用。例如web应用、邮件应用、等等。
  - 系统信息：操作系统版本
  - 版本信息：所有这些探测到的东西的版本。
  - 服务信息：中间件的各类信息，插件信息。
  - 人员信息：域名注册人员信息，web应用中发帖人的id，管理员姓名等。
  - 防护信息：试着看能否探测到防护设备。
- 漏洞探测
- 漏洞验证
- 内网转发
- 内网横向渗透
- 权限维持
- 痕迹清除
- 撰写渗透测试保告

![img](/images/security/dev-security-flow-2.png)

### [#](#_9-4-单元测试) 9.4 单元测试

#### [#](#谈谈你对单元测试的理解) 谈谈你对单元测试的理解？

- **什么是单元测试**？

单元测试（unit testing），是指对软件中的最小可测试单元进行检查和验证。

- **为什么要写单元测试**？

使用单元测试可以有效地降低程序出错的机率，提供准确的文档，并帮助我们改进设计方案等等。

- **什么时候写单元测试**？

比较推荐单元测试与具体实现代码同步进行这个方案的。只有对需求有一定的理解后才能知道什么是代码的正确性，才能写出有效的单元测试来验证正确性，而能写出一些功能代码则说明对需求有一定理解了。

- **单元测试要写多细**？

单元测试不是越多越好，而是越有效越好！进一步解读就是哪些代码需要有单元测试覆盖：

1. 逻辑复杂的
2. 容易出错的
3. 不易理解的，即使是自己过段时间也会遗忘的，看不懂自己的代码，单元测试代码有助于理解代码的功能和需求
4. 公共代码。比如自定义的所有http请求都会经过的拦截器；工具类等。
5. 核心业务代码。一个产品里最核心最有业务价值的代码应该要有较高的单元测试覆盖率。

#### [#](#junit-5整体架构) JUnit 5整体架构？

与以前版本的JUnit不同，JUnit 5由三个不同子项目中的几个不同模块组成。JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

- **JUnit Platform**是基于JVM的运行测试的基础框架在，它定义了开发运行在这个测试框架上的TestEngine API。此外该平台提供了一个控制台启动器，可以从命令行启动平台，可以为Gradle和 Maven构建插件，同时提供基于JUnit 4的Runner。
- **JUnit Jupiter**是在JUnit 5中编写测试和扩展的新编程模型和扩展模型的组合.Jupiter子项目提供了一个TestEngine在平台上运行基于Jupiter的测试。
- **JUnit Vintage**提供了一个TestEngine在平台上运行基于JUnit 3和JUnit 4的测试。

架构图如下:

![img](/images/develop/ut/dev-ut-1.png)

#### [#](#junit-5与junit4的差别在哪里) JUnit 5与Junit4的差别在哪里？

对比下Junit5和Junit4注解:

| Junit4       | Junit5          | 注释                                                         |
| ------------ | --------------- | ------------------------------------------------------------ |
| @Test        | @Test           | 表示该方法是一个测试方法                                     |
| @BeforeClass | **@BeforeAll**  | 表示使用了该注解的方法应该在当前类中所有测试方法之前执行（只执行一次），并且它必须是 static方法（除非@TestInstance指定生命周期为Lifecycle.PER_CLASS） |
| @AfterClass  | **@AfterAll**   | 表示使用了该注解的方法应该在当前类中所有测试方法之后执行（只执行一次），并且它必须是 static方法（除非@TestInstance指定生命周期为Lifecycle.PER_CLASS） |
| @Before      | **@BeforeEach** | 表示使用了该注解的方法应该在当前类中每一个测试方法之前执行   |
| @After       | **@AfterEach**  | 表示使用了该注解的方法应该在当前类中每一个测试方法之后执行   |
| @Ignore      | @Disabled       | 用于禁用（或者说忽略）一个测试类或测试方法                   |
| @Category    | @Tag            | 用于声明过滤测试的tag标签，该注解可以用在方法或类上          |

#### [#](#你在开发中使用什么框架来做单元测试) 你在开发中使用什么框架来做单元测试？

- JUnit4/5
- Mockito, mock测试
- Powermock, 静态util的测试

### [#](#_9-5-代码质量) 9.5 代码质量

#### [#](#你们项目中是如何保证代码质量的) 你们项目中是如何保证代码质量的？

- **checkstyle**, 静态样式检查
- **sonarlint** Sonar是一个用于代码质量管理的开源平台，用于管理源代码的质量 通过插件形式，可以支持包括java,C#,C/C++,PL/SQL,Cobol,JavaScrip,Groovy等等二十几种编程语言的代码质量管理与检测
- **spotbugs**, SpotBugs是Findbugs的继任者（Findbugs已经于2016年后不再维护），用于对代码进行静态分析，查找相关的漏洞; 它是一款自由软件，按照GNU Lesser General Public License 的条款发布

#### [#](#你们项目中是如何做code-review的) 你们项目中是如何做code review的？

Gerrit + 定期线下review

### [#](#_9-6-代码重构) 9.6 代码重构

#### [#](#如何去除多余的if-else) 如何去除多余的if else？

- 出现if/else和switch/case的场景

通常业务代码会包含这样的逻辑：每种条件下会有不同的处理逻辑。比如两个数a和b之间可以通过不同的操作符（+，-，*，/）进行计算，初学者通常会这么写：

```java
public int calculate(int a, int b, String operator) {
    int result = Integer.MIN_VALUE;
 
    if ("add".equals(operator)) {
        result = a + b;
    } else if ("multiply".equals(operator)) {
        result = a * b;
    } else if ("divide".equals(operator)) {
        result = a / b;
    } else if ("subtract".equals(operator)) {
        result = a - b;
    }
    return result;
}
```

这种最基础的代码如何重构呢？

- **工厂类**

```java
public class OperatorFactory {
    static Map<String, Operation> operationMap = new HashMap<>();
    static {
        operationMap.put("add", new Addition());
        operationMap.put("divide", new Division());
        // more operators
    }
 
    public static Optional<Operation> getOperation(String operator) {
        return Optional.ofNullable(operationMap.get(operator));
    }
}
```

- **枚举**

```java
public enum Operator {
    ADD {
        @Override
        public int apply(int a, int b) {
            return a + b;
        }
    },
    // other operators
    
    public abstract int apply(int a, int b);

}
```

- **Command模式**

```java
public class AddCommand implements Command {
    // Instance variables
 
    public AddCommand(int a, int b) {
        this.a = a;
        this.b = b;
    }
 
    @Override
    public Integer execute() {
        return a + b;
    }
}
```

- **规则引擎**

1. 定义规则

```java
public interface Rule {
    boolean evaluate(Expression expression);
    Result getResult();
}
```

1. Add 规则

```java
public class AddRule implements Rule {
    @Override
    public boolean evaluate(Expression expression) {
        boolean evalResult = false;
        if (expression.getOperator() == Operator.ADD) {
            this.result = expression.getX() + expression.getY();
            evalResult = true;
        }
        return evalResult;
    }    
}
```

1. 表达式

```java
public class Expression {
    private Integer x;
    private Integer y;
    private Operator operator;        
}
```

1. 规则引擎

```java
public class RuleEngine {
    private static List<Rule> rules = new ArrayList<>();
 
    static {
        rules.add(new AddRule());
    }
 
    public Result process(Expression expression) {
        Rule rule = rules
          .stream()
          .filter(r -> r.evaluate(expression))
          .findFirst()
          .orElseThrow(() -> new IllegalArgumentException("Expression does not matches any Rule"));
        return rule.getResult();
    }
}
```

- **策略模式**

1. 操作

```java
public interface Opt {
    int apply(int a, int b);
}

@Component(value = "addOpt")
public class AddOpt implements Opt {
    @Autowired
    xxxAddResource resource; // 这里通过Spring框架注入了资源

    @Override
    public int apply(int a, int b) {
       return resource.process(a, b);
    }
}

@Component(value = "devideOpt")
public class devideOpt implements Opt {
    @Autowired
    xxxDivResource resource; // 这里通过Spring框架注入了资源

    @Override
    public int apply(int a, int b) {
       return resource.process(a, b);
    }
}
```

1. 策略

```java
@Component
public class OptStrategyContext{
 

    private Map<String, Opt> strategyMap = new ConcurrentHashMap<>();
 
    @Autowired
    public OptStrategyContext(Map<String, TalkService> strategyMap) {
        this.strategyMap.clear();
        this.strategyMap.putAll(strategyMap);
    }
 
    public int apply(Sting opt, int a, int b) {
        return strategyMap.get(opt).apply(a, b);
    }
}
```

#### [#](#如何去除不必要的-判空) 如何去除不必要的!=判空？

- **空对象模式**

```java
public class MyParser implements Parser {
  private static Action NO_ACTION = new Action() {
    public void doSomething() { /* do nothing */ }
  };

  public Action findAction(String userInput) {
    // ...
    if ( /* we can't find any actions */ ) {
      return NO_ACTION;
    }
  }
}
```

然后便可以始终可以这么调用

```java
ParserFactory.getParser().findAction(someInput).doSomething();
```

- **Java8中使用Optional**

```java
Outer outer = new Outer();
if (outer != null && outer.nested != null && outer.nested.inner != null) {
    System.out.println(outer.nested.inner.foo);
}
```

我们可以通过利用 Java 8 的 Optional 类型来摆脱所有这些 null 检查。map 方法接收一个 Function 类型的 lambda 表达式，并自动将每个 function 的结果包装成一个 Optional 对象。这使我们能够在一行中进行多个 map 操作。Null 检查是在底层自动处理的。

```java
Optional.of(new Outer())
    .map(Outer::getNested)
    .map(Nested::getInner)
    .map(Inner::getFoo)
    .ifPresent(System.out::println);
```

还有一种实现相同作用的方式就是通过利用一个 supplier 函数来解决嵌套路径的问题:

```java
Outer obj = new Outer();
resolve(() -> obj.getNested().getInner().getFoo())
    .ifPresent(System.out::println);
```

## [#](#_10-开发框架和中间件) 10 开发框架和中间件

> 开发框架相关

### [#](#_10-1-spring) 10.1 Spring

#### [#](#什么是spring框架) 什么是Spring框架？

Spring是一种轻量级框架，旨在提高开发人员的开发效率以及系统的可维护性。

我们一般说的Spring框架就是Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是核心容器、数据访问/集成、Web、AOP（面向切面编程）、工具、消息和测试模块。比如Core Container中的Core组件是Spring所有组件的核心，Beans组件和Context组件是实现IOC和DI的基础，AOP组件用来实现面向切面编程。

Spring官网列出的Spring的6个特征：

- 核心技术：依赖注入（DI），AOP，事件（Events），资源，i18n，验证，数据绑定，类型转换，SpEL。
- 测试：模拟对象，TestContext框架，Spring MVC测试，WebTestClient。
- 数据访问：事务，DAO支持，JDBC，ORM，编组XML。
- Web支持：Spring MVC和Spring WebFlux Web框架。
- 集成：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
- 语言：Kotlin，Groovy，动态语言。

#### [#](#列举一些重要的spring模块) 列举一些重要的Spring模块？

下图对应的是Spring 4.x的版本，目前最新的5.x版本中Web模块的Portlet组件已经被废弃掉，同时增加了用于异步响应式处理的WebFlux组件。

- Spring Core：基础，可以说Spring其他所有的功能都依赖于该类库。主要提供IOC和DI功能。
- Spring Aspects：该模块为与AspectJ的集成提供支持。
- Spring AOP：提供面向切面的编程实现。
- Spring JDBC：Java数据库连接。
- Spring JMS：Java消息服务。
- Spring ORM：用于支持Hibernate等ORM工具。
- Spring Web：为创建Web应用程序提供支持。
- Spring Test：提供了对JUnit和TestNG测试的支持。

![img](/images/spring/spring-interview-1.png)

#### [#](#什么是ioc-如何实现的) 什么是IOC? 如何实现的？

IOC（Inversion Of Controll，控制反转）是一种设计思想，就是将原本在程序中手动创建对象的控制权，交给IOC容器来管理，并由IOC容器完成对象的注入。这样可以很大程度上简化应用的开发，把应用从复杂的依赖关系中解放出来。IOC容器就像是一个工厂一样，当我们需要创建一个对象的时候，只需要配置好配置文件/注解即可，完全不用考虑对象是如何被创建出来的。

Spring 中的 IoC 的实现原理就是工厂模式加反射机制。

示例：

```java
interface Fruit {
     public abstract void eat();
}
class Apple implements Fruit {
    public void eat(){
        System.out.println("Apple");
    }
}
class Orange implements Fruit {
    public void eat(){
        System.out.println("Orange");
    }
}
class Factory {
    public static Fruit getInstance(String ClassName) {
        Fruit f=null;
        try {
            f=(Fruit)Class.forName(ClassName).newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return f;
    }
}
class Client {
    public static void main(String[] a) {
        Fruit f=Factory.getInstance("io.github.dunwu.spring.Apple");
        if(f!=null){
            f.eat();
        }
    }
}
```

#### [#](#什么是aop-有哪些aop的概念) 什么是AOP? 有哪些AOP的概念？

AOP（Aspect-Oriented Programming，面向切面编程）能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可扩展性和可维护性。

Spring AOP是基于动态代理的，如果要代理的对象实现了某个接口，那么Spring AOP就会使用JDK动态代理去创建代理对象；而对于没有实现接口的对象，就无法使用JDK动态代理，转而使用CGlib动态代理生成一个被代理对象的子类来作为代理。

![img](/images/spring/spring-interview-3.png)

当然也可以使用AspectJ，Spring AOP中已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。使用AOP之后我们可以把一些通用功能抽象出来，在需要用到的地方直接使用即可，这样可以大大简化代码量。我们需要增加新功能也方便，提高了系统的扩展性。日志功能、事务管理和权限管理等场景都用到了AOP。

**AOP包含的几个概念**

1. Jointpoint（连接点）：具体的切面点点抽象概念，可以是在字段、方法上，Spring中具体表现形式是PointCut（切入点），仅作用在方法上。
2. Advice（通知）: 在连接点进行的具体操作，如何进行增强处理的，分为前置、后置、异常、最终、环绕五种情况。
3. 目标对象：被AOP框架进行增强处理的对象，也被称为被增强的对象。
4. AOP代理：AOP框架创建的对象，简单的说，代理就是对目标对象的加强。Spring中的AOP代理可以是JDK动态代理，也可以是CGLIB代理。
5. Weaving（织入）：将增强处理添加到目标对象中，创建一个被增强的对象的过程

总结为一句话就是：在目标对象（target object）的某些方法（jointpoint）添加不同种类的操作（通知、增强操处理），最后通过某些方法（weaving、织入操作）实现一个新的代理目标对象。

#### [#](#aop-有哪些应用场景) AOP 有哪些应用场景？

举几个例子：

- 记录日志(调用方法后记录日志)
- 监控性能(统计方法运行时间)
- 权限控制(调用方法前校验是否有权限)
- 事务管理(调用方法前开启事务，调用方法后提交关闭事务 )
- 缓存优化(第一次调用查询数据库，将查询结果放入内存对象， 第二次调用，直接从内存对象返回，不需要查询数据库 )

#### [#](#有哪些aop-advice通知的类型) 有哪些AOP Advice通知的类型？

特定 JoinPoint 处的 Aspect 所采取的动作称为 Advice。Spring AOP 使用一个 Advice 作为拦截器，在 JoinPoint “周围”维护一系列的拦截器。

- **前置通知**（Before advice） ： 这些类型的 Advice 在 joinpoint 方法之前执行，并使用 @Before 注解标记进行配置。
- **后置通知**（After advice） ：这些类型的 Advice 在连接点方法之后执行，无论方法退出是正常还是异常返回，并使用 @After 注解标记进行配置。
- **返回后通知**（After return advice） ：这些类型的 Advice 在连接点方法正常执行后执行，并使用@AfterReturning 注解标记进行配置。
- **环绕通知**（Around advice） ：这些类型的 Advice 在连接点之前和之后执行，并使用 @Around 注解标记进行配置。
- **抛出异常后通知**（After throwing advice） ：仅在 joinpoint 方法通过抛出异常退出并使用 @AfterThrowing 注解标记配置时执行。

#### [#](#aop-有哪些实现方式) AOP 有哪些实现方式？

实现 AOP 的技术，主要分为两大类：

- 静态代理

   \- 指使用 AOP 框架提供的命令进行编译，从而在编译阶段就可生成 AOP 代理类，因此也称为编译时增强； 

  - 编译时编织（特殊编译器实现）
  - 类加载时编织（特殊的类加载器实现）。

- 动态代理

   \- 在运行时在内存中“临时”生成 AOP 动态代理类，因此也被称为运行时增强。 

  - JDK 动态代理 
    - JDK Proxy 是 Java 语言自带的功能，无需通过加载第三方类实现；
    - Java 对 JDK Proxy 提供了稳定的支持，并且会持续的升级和更新，Java 8 版本中的 JDK Proxy 性能相比于之前版本提升了很多；
    - JDK Proxy 是通过拦截器加反射的方式实现的；
    - JDK Proxy 只能代理实现接口的类；
    - JDK Proxy 实现和调用起来比较简单；
  - CGLIB 
    - CGLib 是第三方提供的工具，基于 ASM 实现的，性能比较高；
    - CGLib 无需通过接口来实现，它是针对类实现代理，主要是对指定的类生成一个子类，它是通过实现子类的方式来完成调用的。

#### [#](#谈谈你对cglib的理解) 谈谈你对CGLib的理解？

JDK 动态代理机制只能代理实现接口的类，一般没有实现接口的类不能进行代理。使用 CGLib 实现动态代理，完全不受代理类必须实现接口的限制。

CGLib 的原理是对指定目标类生成一个子类，并覆盖其中方法实现增强，但因为采用的是继承，所以不能对 final 修饰的类进行代理。

举例：

```java
public class CGLibDemo {

    // 需要动态代理的实际对象
    static class Sister  {
        public void sing() {
            System.out.println("I am Jinsha, a little sister.");
        }
    }

    static class CGLibProxy implements MethodInterceptor {

        private Object target;

        public Object getInstance(Object target){
            this.target = target;
            Enhancer enhancer = new Enhancer();
            // 设置父类为实例类
            enhancer.setSuperclass(this.target.getClass());
            // 回调方法
            enhancer.setCallback(this);
            // 创建代理对象
            return enhancer.create();
        }

        @Override
        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
            System.out.println("introduce yourself...");
            Object result = methodProxy.invokeSuper(o,objects);
            System.out.println("score...");
            return result;
        }
    }

    public static void main(String[] args) {
        CGLibProxy cgLibProxy = new CGLibProxy();
        //获取动态代理类实例
        Sister proxySister = (Sister) cgLibProxy.getInstance(new Sister());
        System.out.println("CGLib Dynamic object name: " + proxySister.getClass().getName());
        proxySister.sing();
    }
}
```

CGLib 的调用流程就是通过调用拦截器的 intercept 方法来实现对被代理类的调用。而拦截逻辑可以写在 intercept 方法的 invokeSuper(o, objects);的前后实现拦截。

#### [#](#spring-aop和aspectj-aop有什么区别) Spring AOP和AspectJ AOP有什么区别？

Spring AOP是属于运行时增强，而AspectJ是编译时增强。Spring AOP基于代理（Proxying），而AspectJ基于字节码操作（Bytecode Manipulation）。

Spring AOP已经集成了AspectJ，AspectJ应该算得上是Java生态系统中最完整的AOP框架了。AspectJ相比于Spring AOP功能更加强大，但是Spring AOP相对来说更简单。

如果我们的切面比较少，那么两者性能差异不大。但是，当切面太多的话，最好选择AspectJ，它比SpringAOP快很多。

#### [#](#spring中的bean的作用域有哪些) Spring中的bean的作用域有哪些？

1. singleton：唯一bean实例，Spring中的bean默认都是单例的。
2. prototype：每次请求都会创建一个新的bean实例。
3. request：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP request内有效。
4. session：每一次HTTP请求都会产生一个新的bean，该bean仅在当前HTTP session内有效。
5. global-session：全局session作用域，仅仅在基于Portlet的Web应用中才有意义，Spring5中已经没有了。Portlet是能够生成语义代码（例如HTML）片段的小型Java Web插件。它们基于Portlet容器，可以像Servlet一样处理HTTP请求。但是与Servlet不同，每个Portlet都有不同的会话。

#### [#](#spring中的单例bean的线程安全问题了解吗) Spring中的单例bean的线程安全问题了解吗？

大部分时候我们并没有在系统中使用多线程，所以很少有人会关注这个问题。单例bean存在线程问题，主要是因为当多个线程操作同一个对象的时候，对这个对象的非静态成员变量的写操作会存在线程安全问题。

有两种常见的解决方案：

1.在bean对象中尽量避免定义可变的成员变量（不太现实）。

2.在类中定义一个ThreadLocal成员变量，将需要的可变成员变量保存在ThreadLocal中（推荐的一种方式）。

#### [#](#spring中的bean生命周期) Spring中的bean生命周期？

**Bean的完整生命周期经历了各种方法调用，这些方法可以划分为以下几类**：

- **Bean自身的方法**： 这个包括了Bean本身调用的方法和通过配置文件中`<bean>`的init-method和destroy-method指定的方法
- **Bean级生命周期接口方法**： 这个包括了BeanNameAware、BeanFactoryAware、ApplicationContextAware；当然也包括InitializingBean和DiposableBean这些接口的方法（可以被@PostConstruct和@PreDestroy注解替代)
- **容器级生命周期接口方法**： 这个包括了InstantiationAwareBeanPostProcessor 和 BeanPostProcessor 这两个接口实现，一般称它们的实现类为“后处理器”。
- **工厂后处理器接口方法**： 这个包括了AspectJWeavingEnabler, ConfigurationClassPostProcessor, CustomAutowireConfigurer等等非常有用的工厂后处理器接口的方法。工厂后处理器也是容器级的。在应用上下文装配配置文件之后立即调用。

![img](/images/spring/springframework/spring-framework-ioc-source-102.png)

**具体而言，流程如下**

- 如果 BeanFactoryPostProcessor 和 Bean 关联, 则调用postProcessBeanFactory方法.(即首**先尝试从Bean工厂中获取Bean**)

- 如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用postProcessBeforeInstantiation方法

- 根据配置情况调用 Bean 构造方法**实例化 Bean**。

- 利用依赖注入完成 Bean 中所有**属性值的配置注入**。

- 如果 InstantiationAwareBeanPostProcessor 和 Bean 关联，则调用postProcessAfterInstantiation方法和postProcessProperties

- 调用xxxAware接口

   (上图只是给了几个例子) 

  - 第一类Aware接口
    - 如果 Bean 实现了 BeanNameAware 接口，则 Spring 调用 Bean 的 setBeanName() 方法传入当前 Bean 的 id 值。
    - 如果 Bean 实现了 BeanClassLoaderAware 接口，则 Spring 调用 setBeanClassLoader() 方法传入classLoader的引用。
    - 如果 Bean 实现了 BeanFactoryAware 接口，则 Spring 调用 setBeanFactory() 方法传入当前工厂实例的引用。
  - 第二类Aware接口
    - 如果 Bean 实现了 EnvironmentAware 接口，则 Spring 调用 setEnvironment() 方法传入当前 Environment 实例的引用。
    - 如果 Bean 实现了 EmbeddedValueResolverAware 接口，则 Spring 调用 setEmbeddedValueResolver() 方法传入当前 StringValueResolver 实例的引用。
    - 如果 Bean 实现了 ApplicationContextAware 接口，则 Spring 调用 setApplicationContext() 方法传入当前 ApplicationContext 实例的引用。
    - ...

- 如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的预初始化方法 postProcessBeforeInitialzation() 对 Bean 进行加工操作，此处非常重要，Spring 的 AOP 就是利用它实现的。

- 如果 Bean 实现了 InitializingBean 接口，则 Spring 将调用 afterPropertiesSet() 方法。(或者有执行@PostConstruct注解的方法)

- 如果在配置文件中通过 **init-method** 属性指定了初始化方法，则调用该初始化方法。

- 如果 BeanPostProcessor 和 Bean 关联，则 Spring 将调用该接口的初始化方法 postProcessAfterInitialization()。此时，Bean 已经可以被应用系统使用了。

- 如果在 `<bean>` 中指定了该 Bean 的作用范围为 scope="singleton"，则将该 Bean 放入 Spring IoC 的缓存池中，将触发 Spring 对该 Bean 的生命周期管理；如果在 `<bean>` 中指定了该 Bean 的作用范围为 scope="prototype"，则将该 Bean 交给调用者，调用者管理该 Bean 的生命周期，Spring 不再管理该 Bean。

- 如果 Bean 实现了 DisposableBean 接口，则 Spring 会调用 destory() 方法将 Spring 中的 Bean 销毁；(或者有执行@PreDestroy注解的方法)

- 如果在配置文件中通过 **destory-method** 属性指定了 Bean 的销毁方法，则 Spring 将调用该方法对 Bean 进行销毁。

#### [#](#说说自己对于spring-mvc的了解) 说说自己对于Spring MVC的了解？

MVC是一种设计模式，Spring MVC是一款很优秀的MVC框架。Spring MVC可以帮助我们进行更简洁的Web层的开发，并且它天生与Spring框架集成。Spring MVC下我们一般把后端项目分为Service层（处理业务）、Dao层（数据库操作）、Entity层（实体类）、Controller层（控制层，返回数据给前台页面）。

Spring MVC的简单原理图如下：

![img](/images/spring/spring-interview-6.png)

#### [#](#spring-mvc的工作原理了解嘛) Spring MVC的工作原理了解嘛？

![img](/images/project/project-b-5.png)

流程说明：

1.客户端（浏览器）发送请求，直接请求到DispatcherServlet。

2.DispatcherServlet根据请求信息调用HandlerMapping，解析请求对应的Handler。

3.解析到对应的Handler（也就是我们平常说的Controller控制器）。

4.HandlerAdapter会根据Handler来调用真正的处理器来处理请求和执行相对应的业务逻辑。

5.处理器处理完业务后，会返回一个ModelAndView对象，Model是返回的数据对象，View是逻辑上的View。

6.ViewResolver会根据逻辑View去查找实际的View。

7.DispatcherServlet把返回的Model传给View（视图渲染）。

8.把View返回给请求者（浏览器）。

#### [#](#spring框架中用到了哪些设计模式) Spring框架中用到了哪些设计模式？

举几个例子

1.工厂设计模式：Spring使用工厂模式通过BeanFactory和ApplicationContext创建bean对象。

2.代理设计模式：Spring AOP功能的实现。

3.单例设计模式：Spring中的bean默认都是单例的。

4.模板方法模式：Spring中的jdbcTemplate、hibernateTemplate等以Template结尾的对数据库操作的类，它们就使用到了模板模式。

5.包装器设计模式：我们的项目需要连接多个数据库，而且不同的客户在每次访问中根据需要会去访问不同的数据库。这种模式让我们可以根据客户的需求能够动态切换不同的数据源。

6.观察者模式：Spring事件驱动模型就是观察者模式很经典的一个应用。

7.适配器模式：Spring AOP的增强或通知（Advice）使用到了适配器模式、Spring MVC中也是用到了适配器模式适配Controller。

#### [#](#component和-bean的区别是什么) @Component和@Bean的区别是什么？

1.作用对象不同。@Component注解作用于类，而@Bean注解作用于方法。

2.@Component注解通常是通过类路径扫描来自动侦测以及自动装配到Spring容器中（我们可以使用@ComponentScan注解定义要扫描的路径）。@Bean注解通常是在标有该注解的方法中定义产生这个bean，告诉Spring这是某个类的实例，当我需要用它的时候还给我。

3.@Bean注解比@Component注解的自定义性更强，而且很多地方只能通过@Bean注解来注册bean。比如当引用第三方库的类需要装配到Spring容器的时候，就只能通过@Bean注解来实现。

@Bean注解的使用示例：

```java
@Configuration
public class AppConfig {
    @Bean
    public TransferService transferService() {
        return new TransferServiceImpl();
    }
}
```

上面的代码相当于下面的XML配置：

```xml
<beans>
    <bean id="transferService" class="com.yanggb.TransferServiceImpl"/>
</beans>
```

下面这个例子是无法通过@Component注解实现的：

```java
@Bean
public OneService getService(status) {
    case (status)  {
        when 1:
                return new serviceImpl1();
        when 2:
                return new serviceImpl2();
        when 3:
                return new serviceImpl3();
    }
}
```

#### [#](#将一个类声明为spring的bean的注解有哪些) 将一个类声明为Spring的bean的注解有哪些？

我们一般使用@Autowired注解去自动装配bean。而想要把一个类标识为可以用@Autowired注解自动装配的bean，可以采用以下的注解实现：

1.@Component注解。通用的注解，可标注任意类为Spring组件。如果一个Bean不知道属于哪一个层，可以使用@Component注解标注。

2.@Repository注解。对应持久层，即Dao层，主要用于数据库相关操作。

3.@Service注解。对应服务层，即Service层，主要涉及一些复杂的逻辑，需要用到Dao层（注入）。

4.@Controller注解。对应Spring MVC的控制层，即Controller层，主要用于接受用户请求并调用Service层的方法返回数据给前端页面。

#### [#](#spring事务管理的方式有几种) Spring事务管理的方式有几种？

1.编程式事务：在代码中硬编码（不推荐使用）。

2.声明式事务：在配置文件中配置（推荐使用），分为基于XML的声明式事务和基于注解的声明式事务。

#### [#](#spring事务中的隔离级别有哪几种) Spring事务中的隔离级别有哪几种？

在TransactionDefinition接口中定义了五个表示隔离级别的常量：

ISOLATION_DEFAULT：使用后端数据库默认的隔离级别，Mysql默认采用的REPEATABLE_READ隔离级别；Oracle默认采用的READ_COMMITTED隔离级别。

ISOLATION_READ_UNCOMMITTED：最低的隔离级别，允许读取尚未提交的数据变更，可能会导致脏读、幻读或不可重复读。

ISOLATION_READ_COMMITTED：允许读取并发事务已经提交的数据，可以阻止脏读，但是幻读或不可重复读仍有可能发生

ISOLATION_REPEATABLE_READ：对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，可以阻止脏读和不可重复读，但幻读仍有可能发生。

ISOLATION_SERIALIZABLE：最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，该级别可以防止脏读、不可重复读以及幻读。但是这将严重影响程序的性能。通常情况下也不会用到该级别。  

#### [#](#spring事务中有哪几种事务传播行为) Spring事务中有哪几种事务传播行为？

在TransactionDefinition接口中定义了7个表示事务传播行为的常量。

支持当前事务的情况：

PROPAGATION_REQUIRED：如果当前存在事务，则加入该事务；如果当前没有事务，则创建一个新的事务。

PROPAGATION_SUPPORTS： 如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行。

PROPAGATION_MANDATORY： 如果当前存在事务，则加入该事务；如果当前没有事务，则抛出异常。（mandatory：强制性）。

不支持当前事务的情况：

PROPAGATION_REQUIRES_NEW： 创建一个新的事务，如果当前存在事务，则把当前事务挂起。

PROPAGATION_NOT_SUPPORTED： 以非事务方式运行，如果当前存在事务，则把当前事务挂起。

PROPAGATION_NEVER： 以非事务方式运行，如果当前存在事务，则抛出异常。

其他情况：

PROPAGATION_NESTED： 如果当前存在事务，则创建一个事务作为当前事务的嵌套事务来运行；如果当前没有事务，则该取值等价于PROPAGATION_REQUIRED。

#### [#](#bean-factory和applicationcontext有什么区别) Bean Factory和ApplicationContext有什么区别？

ApplicationContex提供了一种解析文本消息的方法，一种加载文件资源（如图像）的通用方法，它们可以将事件发布到注册为侦听器的bean。此外，可以在应用程序上下文中以声明方式处理容器中的容器或容器上的操作，这些操作必须以编程方式与Bean Factory一起处理。ApplicationContext实现MessageSource，一个用于获取本地化消息的接口，实际的实现是可插入的。

#### [#](#如何定义bean的范围) 如何定义bean的范围？

在Spring中定义一个时，我们也可以为bean声明一个范围。它可以通过bean定义中的scope属性定义。例如，当Spring每次需要生成一个新的bean实例时，bean'sscope属性就是原型。另一方面，当每次需要Spring都必须返回相同的bean实例时，bean scope属性必须设置为singleton。

#### [#](#可以通过多少种方式完成依赖注入) 可以通过多少种方式完成依赖注入？

通常，依赖注入可以通过三种方式完成，即：

- 构造函数注入
- setter 注入
- 接口注入

### [#](#_10-2-spring-boot) 10.2 Spring Boot

#### [#](#什么是springboot) 什么是SpringBoot？

Spring Boot 是 Spring 开源组织下的子项目，是 Spring 组件一站式解决方案，主要是简化了使用 Spring 的难度，简省了繁重的配置，提供了各种启动器，开发者能快速上手。

- 用来简化Spring应用的初始搭建以及开发过程，使用特定的方式来进行配置
- 创建独立的Spring引用程序main方法运行
- 嵌入的tomcat无需部署war文件
- 简化maven配置
- 自动配置Spring添加对应的功能starter自动化配置
- SpringBoot来简化Spring应用开发，约定大于配置，去繁化简

#### [#](#为什么使用springboot) 为什么使用SpringBoot？

- 独立运行

Spring Boot 而且内嵌了各种 servlet 容器，Tomcat、Jetty 等，现在不再需要打成war 包部署到容器中，Spring Boot 只要打成一个可执行的 jar 包就能独立运行，所有的依赖包都在一个 jar 包内。

- 简化配置

spring-boot-starter-web 启动器自动依赖其他组件，简少了 maven 的配置。

- 自动配置

Spring Boot 能根据当前类路径下的类、jar 包来自动配置 bean，如添加一个 spring

boot-starter-web 启动器就能拥有 web 的功能，无需其他配置。

- 无代码生成和XML配置

Spring Boot 配置过程中无代码生成，也无需 XML 配置文件就能完成所有配置工作，这一切都是借助于条件注解完成的，这也是 Spring4.x 的核心功能之一。

- 应用监控

Spring Boot 提供一系列端点可以监控服务及应用，做健康检测。

#### [#](#spring、spring-mvc和springboot有什么区别) Spring、Spring MVC和SpringBoot有什么区别？

- Spring

Spring最重要的特征是依赖注入。所有Spring Modules不是依赖注入就是IOC控制反转。

当我们恰当的使用DI或者是IOC的时候，可以开发松耦合应用。

- Spring MVC

Spring MVC提供了一种分离式的方法来开发Web应用。通过运用像DispatcherServelet，ModelAndView 和 ViewResolver 等一些简单的概念，开发 Web 应用将会变的非常简单。

- SpringBoot

Spring和Spring MVC的问题在于需要配置大量的参数。

SpringBoot通过一个自动配置和启动的项来解决这个问题。

#### [#](#springboot自动配置的原理) SpringBoot自动配置的原理?

在Spring程序main方法中，添加@SpringBootApplication或者@EnableAutoConfiguration会自动去maven中读取每个starter中的spring.factories文件，该文件里配置了所有需要被创建的Spring容器中的bean

#### [#](#spring-boot的核心注解是哪些-他主由哪几个注解组成的) Spring Boot的核心注解是哪些？他主由哪几个注解组成的？

启动类上面的注解是@SpringBootApplication，他也是SpringBoot的核心注解，主要组合包含了以下3个注解：

- @SpringBootConfiguration：组合了@Configuration注解，实现配置文件的功能；
- @EnableAutoConfiguration：打开自动配置的功能，也可以关闭某个自动配置的选项，如关闭数据源自动配置的功能：
- @SpringBootApplication(exclude={DataSourceAutoConfiguration.class})；
- @ComponentScan：Spring组件扫描。

#### [#](#springboot的核心配置文件有哪几个-他们的区别是什么) SpringBoot的核心配置文件有哪几个？他们的区别是什么？

SpringBoot的核心配置文件是application和bootstrap配置文件。

application配置文件这个容易理解，主要用于Spring Boot项目的自动化配置。

bootstrap配置文件有以下几个应用场景：

- 使用Spring Cloud Config配置中心时，这时需要在bootstrap配置文件中添加连接到配置中心的配置属性来加载外部配置中心的配置信息；
- 一些固定的不能被覆盖的属性；
- 一些加密/解密的场景；

#### [#](#什么是spring-boot-starter-有哪些常用的) 什么是Spring Boot Starter？有哪些常用的？

和自动配置一样，Spring Boot Starter的目的也是简化配置，而Spring Boot Starter解决的是依赖管理配置复杂的问题，有了它，当我需要构建一个Web应用程序时，不必再遍历所有的依赖包，一个一个地添加到项目的依赖管理中，而是只需要一个配置spring-boot-starter-web, 同理，如果想引入持久化功能，可以配置spring-boot-starter-data-jpa：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

Spring Boot 也提供了其它的启动器项目包括，包括用于开发特定类型应用程序的典型依赖项。

spring-boot-starter-web-services - SOAP Web Services

spring-boot-starter-web - Web 和 RESTful 应用程序

spring-boot-starter-test - 单元测试和集成测试

spring-boot-starter-jdbc - 传统的 JDBC

spring-boot-starter-hateoas - 为服务添加 HATEOAS 功能

spring-boot-starter-security - 使用 SpringSecurity 进行身份验证和授权

spring-boot-starter-data-jpa - 带有 Hibernate 的 Spring Data JPA

spring-boot-starter-data-rest - 使用 Spring Data REST 公布简单的 REST 服务

#### [#](#spring-boot-starter-parent有什么作用) spring-boot-starter-parent有什么作用？

我们知道，新建一个SpringBoot项目，默认都是有parent的，这个parent就是spring-boot-starter-parent，spring-boot-starter-parent主要有如下作用：

- 定义了Java编译版本
- 使用UTF-8格式编码
- 继承自spring-boor-dependencies，这里面定义了依赖的版本，也正是因为继承了这个依赖，所以我们在写依赖时才不需要写版本号
- 执行打包操作的配置
- 自动化的资源过滤
- 自动化的插件配置

#### [#](#如何自定义spring-boot-starter) 如何自定义Spring Boot Starter？

- 实现功能
- 添加Properties

```java
@Data
@ConfigurationProperties(prefix = "com.pdai")
public class DemoProperties {
    private String version;
    private String name;
}
```

- 添加AutoConfiguration

```java
@Configuration
@EnableConfigurationProperties(DemoProperties.class)
public class DemoAutoConfiguration {

    @Bean
    public com.pdai.demo.module.DemoModule demoModule(DemoProperties properties){
        com.pdai.demo.module.DemoModule demoModule = new com.pdai.demo.module.DemoModule();
        demoModule.setName(properties.getName());
        demoModule.setVersion(properties.getVersion());
        return demoModule;

    }
}
```

- 添加spring.factory

在META-INF下创建spring.factory文件

```bash
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.pdai.demospringbootstarter.DemoAutoConfiguration
```

- install

![img](/images/spring/springboot-starter-demo-2.png)

#### [#](#为什么需要spring-boot-maven-plugin) 为什么需要spring-boot-maven-plugin？

spring-boot-maven-plugin提供了一些像jar一样打包或者运行应用程序的命令。

1. spring-boot:run 运行SpringBoot应用程序；
2. spring-boot:repackage 重新打包你的jar包或者是war包使其可执行
3. spring-boot:start和spring-boot:stop管理Spring Boot应用程序的生命周期
4. spring-boot:build-info生成执行器可以使用的构造信息

#### [#](#springboot-打成jar和普通的jar有什么区别) SpringBoot 打成jar和普通的jar有什么区别？

Spring Boot 项目最终打包成的 jar 是可执行 jar ，这种 jar 可以直接通过java -jar xxx.jar命令来运行，这种 jar 不可以作为普通的 jar 被其他项目依赖，即使依赖了也无法使用其中的类。

Spring Boot 的 jar 无法被其他项目依赖，主要还是他和普通 jar 的结构不同。普通的 jar 包，解压后直接就是包名，包里就是我们的代码，而 Spring Boot 打包成的可执行 jar 解压后，在 \BOOT-INF\classes目录下才是我们的代码，因此无法被直接引用。如果非要引用，可以在 pom.xml 文件中增加配置，将 Spring Boot 项目打包成两个 jar ，一个可执行，一个可引用。

#### [#](#如何使用spring-boot实现异常处理) 如何使用Spring Boot实现异常处理？

Spring提供了一种使用ControllerAdvice处理异常的非常有用的方法。通过实现一个ControlerAdvice类，来处理控制类抛出的所有异常。

#### [#](#springboot-实现热部署有哪几种方式) SpringBoot 实现热部署有哪几种方式？

主要有两种方式：

- Spring Loaded
- Spring-boot-devtools

#### [#](#spring-boot中的监视器是什么) Spring Boot中的监视器是什么？

Spring boot actuator是spring启动框架中的重要功能之一。Spring boot监视器可帮助您访问生产环境中正在运行的应用程序的当前状态。

有几个指标必须在生产环境中进行检查和监控。即使一些外部应用程序可能正在使用这些服务来向相关人员触发警报消息。监视器模块公开了一组可直接作为HTTP URL访问的REST端点来检查状态。

#### [#](#spring-boot-可以兼容老-spring-项目吗) Spring Boot 可以兼容老 Spring 项目吗？

可以兼容，使用 @ImportResource 注解导入老 Spring 项目配置文件。

### [#](#_10-3-spring-security) 10.3 Spring Security

#### [#](#什么是spring-security-核心功能) 什么是Spring Security？核心功能？

Spring Security是基于Spring的安全框架.它提供全面的安全性解决方案,同时在Web请求级别和调用级别确认和授权.在Spring Framework基础上,Spring Security充分利用了依赖注入(DI)和面向切面编程(AOP)功能,为应用系统提供声明式的安全访问控制功能,建晒了为企业安全控制编写大量重复代码的工作,是一个轻量级的安全框架,并且很好集成Spring MVC

spring security 的核心功能主要包括：

- 认证(Authentication)：指的是验证某个用户是否为系统中的合法主体，也就是说用户能否访问该系统。
- 授权(Authorization)：指的是验证某个用户是否有权限执行某个操作
- 攻击防护：指的是防止伪造身份

#### [#](#spring-security的原理) Spring Security的原理?

简单谈谈其中的要点

- **基于Filter技术实现?**

首先SpringSecurity是基于Filter技术实现的。Spring通过DelegatingFilterProxy建立Web容器和Spring ApplicationContext的联系，而SpringSecurity使用FilterChainProxy 注册SecurityFilterChain。

![img](/images/spring/security/spring-security-3.png)

- **认证模块的实现**？

**SecurityContextHolder**(用于存储授权信息)

![img](/images/spring/security/spring-security-4.png)

手动授权的例子（SecurityContextHolder.getContext().setAuthentication(authentication)这种授权方式多线程不安全）：

```java
SecurityContext context = SecurityContextHolder.createEmptyContext(); 
Authentication authentication =
    new TestingAuthenticationToken("username", "password", "ROLE_USER"); 
context.setAuthentication(authentication);
SecurityContextHolder.setContext(context); 
```

除了手动授权外，**SpringSecurity通过AuthenticationManager和ProviderManager进行授权**。其中AuthenticationProvider代表不同的认证机制（最常用的账号/密码）。

**ProviderManager**

![img](/images/spring/security/spring-security-5.png)

**AuthenticationManager**

![img](/images/spring/security/spring-security-6.png)

- **授权模块的实现**？

认证完成之后，SpringSecurity通过AccessDecisionManager 完成授权操作。除了全局的授权配置之外，也可以通过@PreAuthorize, @PreFilter, @PostAuthorize , @PostFilter注解实现方法级别的权限控制。

![img](/images/spring/security/spring-security-7.png)

#### [#](#spring-security基于用户名和密码的认证模式流程) Spring Security基于用户名和密码的认证模式流程？

请求的用户名密码可以通过表单登录，基础认证，数字认证三种方式从HttpServletRequest中获得，用于认证的数据源策略有内存，数据库，ldap,自定义等。

拦截未授权的请求，重定向到登录页面

![img](/images/spring/security/spring-security-1.png)

表单登录的过程，进行账号密码认证

![img](/images/spring/security/spring-security-2.png)

### [#](#_10-4-mybatis) 10.4 MyBatis

### [#](#_10-5-jpa) 10.5 JPA

### [#](#_10-6-日志框架) 10.6 日志框架

#### [#](#什么是日志系统和日志门面-分别有哪些框架) 什么是日志系统和日志门面？分别有哪些框架？

日志系统是具体的日志框架，日志门面是不提供日志的具体实现，而是在运行时动态的绑定日志实现组件来工作，是一种外观模式。

- **日志系统**
  - java.util.logging (**JUL**)，JDK1.4 开始，通过 java.util.logging 提供日志功能。虽然是官方自带的log lib，JUL的使用确不广泛。
  - **Log4j**，Log4j 是 apache 的一个开源项目，创始人 Ceki Gulcu。Log4j 应该说是 Java 领域资格最老，应用最广的日志工具。Log4j 是高度可配置的，并可通过在运行时的外部文件配置。它根据记录的优先级别，并提供机制，以指示记录信息到许多的目的地，诸如：数据库，文件，控制台，UNIX 系统日志等。Log4j 的短板在于性能，在Logback 和 Log4j2 出来之后，Log4j的使用也减少了。
  - **Logback**，Logback 是由 log4j 创始人 Ceki Gulcu 设计的又一个开源日志组件，是作为 Log4j 的继承者来开发的，提供了性能更好的实现，异步 logger，Filter等更多的特性。
  - **Log4j2**，维护 Log4j 的人为了性能又搞出了 Log4j2。Log4j2 和 Log4j1.x 并不兼容，设计上很大程度上模仿了 SLF4J/Logback，性能上也获得了很大的提升。Log4j2 也做了 Facade/Implementation 分离的设计，分成了 log4j-api 和 log4j-core。
- **日志门面**
  - **common-logging**，common-logging 是 apache 的一个开源项目。也称Jakarta Commons Logging，缩写 JCL。
  - **slf4j**, 全称为 Simple Logging Facade for Java，即 java 简单日志门面。作者又是 Ceki Gulcu！这位大神写了 Log4j、Logback 和 slf4j。类似于 Common-Logging，slf4j 是对不同日志框架提供的一个 API 封装，可以在部署的时候不修改任何配置即可接入一种日志实现方案。但是，slf4j 在编译时静态绑定真正的 Log 库。使用 SLF4J 时，如果你需要使用某一种日志实现，那么你必须选择正确的 SLF4J 的 jar 包的集合（各种桥接包）。

#### [#](#日志库中使用桥接模式解决什么问题) 日志库中使用桥接模式解决什么问题？

- 什么是桥接呢？

假如你正在开发应用程序所调用的组件当中已经使用了 common-logging，这时你需要 jcl-over-slf4j.jar 把日志信息输出重定向到 slf4j-api，slf4j-api 再去调用 slf4j 实际依赖的日志组件。这个过程称为桥接。下图是官方的 slf4j 桥接策略图：

![img](/images/develop/package/dev-package-log-5.png)

从图中应该可以看出，无论你的老项目中使用的是 common-logging 或是直接使用 log4j、java.util.logging，都可以使用对应的桥接 jar 包来解决兼容问题。

- slf4j 兼容 common-logging

```xml
<dependency>
  <groupId>org.slf4j</groupId>
  <artifactId>jcl-over-slf4j</artifactId>
  <version>1.7.12</version>
</dependency>
```

- slf4j 兼容 log4j

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>log4j-over-slf4j</artifactId>
    <version>1.7.12</version>
</dependency>
```

#### [#](#在日志配置时会考虑哪些点) 在日志配置时会考虑哪些点？

- 支持日志路径，日志level等配置
- 日志控制配置通过application.yml下发
- 按天生成日志，当天的日志>50MB回滚
- 最多保存10天日志
- 生成的日志中Pattern自定义
- Pattern中添加用户自定义的MDC字段，比如用户信息(当前日志是由哪个用户的请求产生)，request信息。此种方式可以通过AOP切面控制，在MDC中添加requestID，在spring-logback.xml中配置Pattern。
- 根据不同的运行环境设置Profile - dev，test，product
- 对控制台，Err和全量日志分别配置
- 对第三方包路径日志控制

#### [#](#对java日志组件选型的建议) 对Java日志组件选型的建议？

slf4j已经成为了Java日志组件的明星选手，可以完美替代JCL，使用JCL桥接库也能完美兼容一切使用JCL作为日志门面的类库，现在的新系统已经没有不使用slf4j作为日志API的理由了。

日志记录服务方面，log4j在功能上输于logback和log4j2，在性能方面log4j2则全面超越log4j和logback。所以新系统应该在logback和log4j2中做出选择，对于性能有很高要求的系统，应优先考虑log4j2。

#### [#](#对日志架构使用比较好的实践) 对日志架构使用比较好的实践？

说几个点：

- 总是使用Log Facade，而不是具体Log Implementation
- 只添加一个 Log Implementation依赖
- 具体的日志实现依赖应该设置为optional和使用runtime scope
- 如果有必要, 排除依赖的第三方库中的Log Impementation依赖
- 避免为不会输出的log付出代价
- 日志格式中最好不要使用行号，函数名等字段

#### [#](#对现有系统日志架构的改造建议) 对现有系统日志架构的改造建议？

如果现有系统使用JCL作为日志门面，又确实面临着JCL的ClassLoader机制带来的问题，完全可以引入slf4j并通过桥接库将JCL api输出的日志桥接至slf4j，再通过适配库适配至现有的日志输出服务（如log4j），如下图：

![img](/images/develop/package/dev-package-log-4.png)

这样做不需要任何代码级的改造，就可以解决JCL的ClassLoader带来的问题，但没有办法享受日志模板等slf4j的api带来的优点。不过之后在现系统上开发的新功能就可以使用slf4j的api了，老代码也可以分批进行改造。

如果现有系统使用JCL作为日志门面，又头疼JCL不支持logback和log4j2等新的日志服务，也可以通过桥接库以slf4j替代JCL，但同样无法直接享受slf4j api的优点。

如果想要使用slf4j的api，那么就不得不进行代码改造了，当然改造也可以参考1中提到的方式逐步进行。

如果现系统面临着log4j的性能问题，可以使用Apache Logging提供的log4j到log4j2的桥接库log4j-1.2-api，把通过log4j api输出的日志桥接至log4j2。这样可以最快地使用上log4j2的先进性能，但组件中缺失了slf4j，对后续进行日志架构改造的灵活性有影响。另一种办法是先把log4j桥接至slf4j，再使用slf4j到log4j2的适配库。这样做稍微麻烦了一点，但可以逐步将系统中的日志输出标准化为使用slf4j的api，为后面的工作打好基础。

### [#](#_10-7-tomcat) 10.7 Tomcat

#### [#](#tomcat-整体架构的设计) Tomcat 整体架构的设计？

从组件的角度看

![img](/images/tomcat/tomcat-x-design-2-1.jpeg)

- **Server**: 表示服务器，它提供了一种优雅的方式来启动和停止整个系统，不必单独启停连接器和容器；它是Tomcat构成的顶级构成元素，所有一切均包含在Server中；
- **Service**: 表示服务，Server可以运行多个服务。比如一个Tomcat里面可运行订单服务、支付服务、用户服务等等；Server的实现类StandardServer可以包含一个到多个Services, Service的实现类为StandardService调用了容器(Container)接口，其实是调用了Servlet Engine(引擎)，而且StandardService类中也指明了该Service归属的Server;
- **Container**: 表示容器，可以看做Servlet容器；引擎(Engine)、主机(Host)、上下文(Context)和Wraper均继承自Container接口，所以它们都是容器。
  - Engine -- 引擎
  - Host -- 主机
  - Context -- 上下文
  - Wrapper -- 包装器
- **Connector**: 表示连接器, **它将Service和Container连接起来**，首先它需要注册到一个Service，它的作用就是把来自客户端的请求转发到Container(容器)，这就是它为什么称作连接器, 它支持的协议如下：
  - 支持AJP协议
  - 支持Http协议
  - 支持Https协议
- **Service内部**还有各种支撑组件，下面简单罗列一下这些组件
  - Manager -- 管理器，用于管理会话Session
  - Logger -- 日志器，用于管理日志
  - Loader -- 加载器，和类加载有关，只会开放给Context所使用
  - Pipeline -- 管道组件，配合Valve实现过滤器功能
  - Valve -- 阀门组件，配合Pipeline实现过滤器功能
  - Realm -- 认证授权组件

#### [#](#tomcat-一个请求的处理流程) Tomcat 一个请求的处理流程？

假设来自客户的请求为：http://localhost:8080/test/index.jsp 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector,然后

- Connector把该请求交给它所在的Service的Engine来处理，并等待Engine的回应
- Engine获得请求localhost:8080/test/index.jsp，匹配它所有虚拟主机Host
- Engine匹配到名为localhost的Host(即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机)
- localhost Host获得请求/test/index.jsp，匹配它所拥有的所有Context
- Host匹配到路径为/test的Context(如果匹配不到就把该请求交给路径名为""的Context去处理)
- path="/test"的Context获得请求/index.jsp，在它的mapping table中寻找对应的servlet
- Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类，构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法
- Context把执行完了之后的HttpServletResponse对象返回给Host
- Host把HttpServletResponse对象返回给Engine
- Engine把HttpServletResponse对象返回给Connector
- Connector把HttpServletResponse对象返回给客户browser

#### [#](#tomcat-中类加载机制) Tomcat 中类加载机制？

在Bootstrap中我们可以看到有如下三个classloader

```java
ClassLoader commonLoader = null;
ClassLoader catalinaLoader = null;
ClassLoader sharedLoader = null;
```

- **为什么要设计多个类加载器**？

> 如果所有的类都使用一个类加载器来加载，会出现什么问题呢？

假如我们自己编写一个类`java.util.Object`，它的实现可能有一定的危险性或者隐藏的bug。而我们知道Java自带的核心类里面也有`java.util.Object`，如果JVM启动的时候先行加载的是我们自己编写的`java.util.Object`，那么就有可能出现安全问题！

所以，Sun（后被Oracle收购）采用了另外一种方式来保证最基本的、也是最核心的功能不会被破坏。你猜的没错，那就是双亲委派模式！

- **什么是双亲委派模型**？

> 双亲委派模型解决了类错乱加载的问题，也设计得非常精妙。

双亲委派模式对类加载器定义了层级，每个类加载器都有一个父类加载器。在一个类需要加载的时候，首先委派给父类加载器来加载，而父类加载器又委派给祖父类加载器来加载，以此类推。如果父类及上面的类加载器都加载不了，那么由当前类加载器来加载，并将被加载的类缓存起来。

![img](/images/jvm/java_jvm_classload_3.png)

所以上述类是这么加载的

- Java自带的核心类 -- 由启动类加载器加载
- Java支持的可扩展类 -- 由扩展类加载器加载
- 我们自己编写的类 -- 默认由应用程序类加载器或其子类加载
- **为什么Tomcat的类加载器也不是双亲委派模型**？

Java默认的类加载机制是通过双亲委派模型来实现的，而Tomcat实现的方式又和双亲委派模型有所区别。

**原因在于一个Tomcat容器允许同时运行多个Web程序，每个Web程序依赖的类又必须是相互隔离的**。因此，如果Tomcat使用双亲委派模式来加载类的话，将导致Web程序依赖的类变为共享的。

举个例子，假如我们有两个Web程序，一个依赖A库的1.0版本，另一个依赖A库的2.0版本，他们都使用了类xxx.xx.Clazz，其实现的逻辑因类库版本的不同而结构完全不同。那么这两个Web程序的其中一个必然因为加载的Clazz不是所使用的Clazz而出现问题！而这对于开发来说是非常致命的！

#### [#](#tomcat-container设计) Tomcat Container设计？

我们看下几个Container之间的关系：

![img](/images/tomcat/tomcat-x-container-1.jpg)

从上图上，我们也可以看出Container顶层也是基于Lifecycle的组件设计的。

- **在设计Container组件层次组件时，上述4个组件分别做什么的呢？为什么要四种组件呢？**

**Engine** - 表示整个catalina的servlet引擎，多数情况下包含**一个或多个**子容器，这些子容器要么是Host，要么是Context实现，或者是其他自定义组。

**Host** - 表示包含多个Context的虚拟主机的。

**Context** — 表示一个ServletContext，表示一个webapp，它通常包含一个或多个wrapper。

**Wrapper** - 表示一个servlet定义的（如果servlet本身实现了SingleThreadModel，则可能支持多个servlet实例）。

- **结合整体的框架图中上述组件部分，我们看下包含了什么**？

![img](/images/tomcat/tomcat-x-container-3.png)

很明显，除了四个组件的嵌套关系，Container中还包含了Realm，Cluster，Listeners, Pipleline等支持组件。

这一点，还可以通过相关注释可以看出：

```html
**Loader** - Class loader to use for integrating new Java classes for this Container into the JVM in which Catalina is running.

**Logger** - Implementation of the log() method signatures of the ServletContext interface.

**Manager** - Manager for the pool of Sessions associated with this Container.

**Realm** - Read-only interface to a security domain, for authenticating user identities and their corresponding roles.

**Resources** - JNDI directory context enabling access to static resources, enabling custom linkages to existing server components when Catalina is embedded in a larger server.
```

#### [#](#tomcat-lifecycle机制) Tomcat LifeCycle机制？

- Server及其它组件

![img](/images/tomcat/tomcat-x-lifecycle-1.png)

- Server后续组件生命周期及初始化

![img](/images/tomcat/tomcat-x-lifecycle-2.png)

- Server的依赖结构

![img](/images/tomcat/tomcat-x-lifecycle-3.png)

```java
public interface Lifecycle {
    /** 第1类：针对监听器 **/
    // 添加监听器
    public void addLifecycleListener(LifecycleListener listener);
    // 获取所以监听器
    public LifecycleListener[] findLifecycleListeners();
    // 移除某个监听器
    public void removeLifecycleListener(LifecycleListener listener);
    
    /** 第2类：针对控制流程 **/
    // 初始化方法
    public void init() throws LifecycleException;
    // 启动方法
    public void start() throws LifecycleException;
    // 停止方法，和start对应
    public void stop() throws LifecycleException;
    // 销毁方法，和init对应
    public void destroy() throws LifecycleException;
    
    /** 第3类：针对状态 **/
    // 获取生命周期状态
    public LifecycleState getState();
    // 获取字符串类型的生命周期状态
    public String getStateName();
}
```

#### [#](#tomcat-中executor) Tomcat 中Executor?

- 1.Tomcat希望将Executor也纳入Lifecycle**生命周期管理**，所以让它实现了Lifecycle接口
- 2.**引入超时机制**：也就是说当work queue满时，会等待指定的时间，如果超时将抛出RejectedExecutionException，所以这里增加了一个`void execute(Runnable command, long timeout, TimeUnit unit)`方法; 其实本质上，它构造了JUC中ThreadPoolExecutor，通过它调用ThreadPoolExecutor的`void execute(Runnable command, long timeout, TimeUnit unit)`方法。

#### [#](#tomcat-中的设计模式) Tomcat 中的设计模式？

- **责任链模式：管道机制**

在软件开发的常接触的责任链模式是FilterChain，它体现在很多软件设计中：

1. **比如Spring Security框架中**

![img](/images/tomcat/tomcat-x-pipline-6.jpg)

1. **比如HttpServletRequest处理的过滤器中**

当一个request过来的时候，需要对这个request做一系列的加工，使用责任链模式可以使每个加工组件化，减少耦合。也可以使用在当一个request过来的时候，需要找到合适的加工方式。当一个加工方式不适合这个request的时候，传递到下一个加工方法，该加工方式再尝试对request加工。

网上找了图，这里我们后文将通过Tomcat请求处理向你阐述。

![img](/images/tomcat/tomcat-x-pipline-5.jpg)

- **外观模式：request请求**
- **观察者模式：事件监听**

java中的事件机制的参与者有**3种角色**

1. `Event Eource`：事件源，发起事件的主体。
2. `Event Object`：事件状态对象，传递的信息载体，就好比Watcher的update方法的参数，可以是事件源本身，一般作为参数存在于listerner 的方法之中。
3. `Event Listener`：事件监听器，当它监听到event object产生的时候，它就调用相应的方法，进行处理。

其实还有个东西比较重要：事件环境，在这个环境中，可以添加事件监听器，可以产生事件，可以触发事件监听器。

![img](/images/tomcat/tomcat-x-listener-3.png)

- **模板方式： Lifecycle**

LifecycleBase是使用了**状态机**+**模板模式**来实现的。模板方法有下面这几个：

```java
// 初始化方法
protected abstract void initInternal() throws LifecycleException;
// 启动方法
protected abstract void startInternal() throws LifecycleException;
// 停止方法
protected abstract void stopInternal() throws LifecycleException;
// 销毁方法
protected abstract void destroyInternal() throws LifecycleException;
```

#### [#](#tomcat-jmx拓展机制) Tomcat JMX拓展机制？

## [#](#_11-开发工具) 11 开发工具

> 开发工具问题汇总。

### [#](#_11-1-git) 11.1 Git

#### [#](#git中5个区-和具体操作) Git中5个区，和具体操作？

- 代码提交和同步代码

![img](/images/git-four-areas.png)

- 代码撤销和撤销同步

![img](/images/git-five-states.png)

#### [#](#平时是怎么提交代码的) 平时是怎么提交代码的？

- 第零步: 工作区与仓库保持一致
- 第一步: 文件增删改，变为已修改状态
- 第二步: git add ，变为已暂存状态

```bash
$ git status
$ git add --all # 当前项目下的所有更改
$ git add .  # 当前目录下的所有更改
$ git add xx/xx.py xx/xx2.py  # 添加某几个文件
```

- 第三步: git commit，变为已提交状态

```bash
$ git commit -m "<这里写commit的描述>"
```

- 第四步: git push，变为已推送状态

```bash
$ git push -u origin master # 第一次需要关联上
$ git push # 之后再推送就不用指明应该推送的远程分支了
$ git branch # 可以查看本地仓库的分支
$ git branch -a # 可以查看本地仓库和本地远程仓库(远程仓库的本地镜像)的所有分支
```

在某个分支下，我最常用的操作如下

```bash
$ git status
$ git add -a
$ git status
$ git commit -m 'xxx'
$ git pull --rebase
$ git push origin xxbranch
```

### [#](#_11-2-maven) 11.2 Maven

#### [#](#maven中包的依赖原则-如何解决冲突) Maven中包的依赖原则？如何解决冲突？

- **依赖原则**？

1. 依赖路径最短优先原则

```html
A -> B -> C -> X(1.0)
A -> D -> X(2.0)
```

由于 X(2.0) 路径最短，所以使用 X(2.0)。

1. 声明顺序优先原则

```html
A -> B -> X(1.0)
A -> C -> X(2.0)
```

在 POM 中最先声明的优先，上面的两个依赖如果先声明 B，那么最后使用 X(1.0)。

1. 覆写优先原则

子 POM 内声明的依赖优先于父 POM 中声明的依赖。

- **如何解决冲突**？

1. 找到 Maven 加载的 Jar 包版本，使用 `mvn dependency:tree` 查看依赖树，根据依赖原则来调整依赖在 POM 文件的声明顺序。
2. 发现了冲突的包之后，剩下的就是选择一个合适版本的包留下，如果是传递依赖的包正确，那么把显示依赖的包exclude掉。如果是某一个传递依赖的包有问题，那么我们需要手动把这个传递依赖execlude掉

#### [#](#maven-项目生命周期与构建原理) Maven 项目生命周期与构建原理？

Maven从项目的三个不同的角度，定义了单套生命周期，三套生命周期是相互独立的，它们之间不会相互影响。

- 默认构建生命周期(Default Lifeclyle): 该生命周期表示这项目的构建过程，定义了一个项目的构建要经过的不同的阶段。
- 清理生命周期(Clean Lifecycle): 该生命周期负责清理项目中的多余信息，保持项目资源和代码的整洁性。一般拿来清空directory(即一般的target)目录下的文件。
- 站点管理生命周期(Site Lifecycle) :向我们创建一个项目时，我们有时候需要提供一个站点，来介绍这个项目的信息，如项目介绍，项目进度状态、项目组成成员，版本控制信息，项目javadoc索引信息等等。站点管理生命周期定义了站点管理过程的各个阶段。

![img](/images/maven_1.jpg)

## [#](#_12-架构) 12 架构

> 架构相关。

### [#](#_12-1-架构基础) 12.1 架构基础

#### [#](#如何理解架构的演进) 如何理解架构的演进？

- 初始阶段的网站架构
- 应用服务和数据服务分离
- 使用缓存改善网站性能
- 使用应用服务器集群改善网站的并发处理能力
- 数据库读写分离
- 使用反向代理和CDN加上网站相应
- 使用分布式文件系统和分布式数据库系统
- 使用NoSQL和搜索引擎

![img](/images/arch/arch-x-ev-g-8.png)

- 业务拆分 : 拆成A, B服务，以及MQ服务

![img](/images/arch/arch-x-ev-g-9.png)

- 分布式服务

![img](/images/arch/arch-x-ev-g-10.png)

#### [#](#如何理解架构的服务化趋势) 如何理解架构的服务化趋势？

- 方向一: 

  架构服务化

  - 单体分层架构
  - 面向服务架构 -SOA
  - 微服务架构 - Microservices
  - 云原生架构 - Cloud Native

- 方向二: 

  部署容器编排化

  - 虚拟机
  - 容器
  - Kubernetes 与编排

#### [#](#架构中有哪些技术点) 架构中有哪些技术点？

所谓网站架构模式即为了解决大型网站面临的高并发访问、海量数据、高可靠运行灯一系列问题与挑战。为此，在实践中提出了许多解决方案，以实现网站高性能、高可靠性、易伸缩、可扩展、安全等各种技术架构目标。

- **分层**

分层是企业应用系统中最常见的一种架构模式，将系统在横向维度上切分成几个部分，每个部分负责一部分相对简单并比较单一的职责，然后通过上层对下层的依赖和调度组成一个完整的系统。

在网站的分层架构中，常见的为3层，即`应用层`、`服务层`、`数据层`:

1. 应用层具体负责业务和视图的展示；
2. 服务层为应用层提供服务支持；
3. 数据库提供数据存储访问服务，如数据库、缓存、文件、搜索引擎等。

分层架构是逻辑上的，在物理部署上，三层架构可以部署在同一个物理机器上，但是随着网站业务的发展，必然需要对已经分层的模块分离部署，即三层结构分别部署在不同的服务器上，是网站拥有更多的计算资源以应对越来越多的用户访问。

所以虽然分层架构模式最初的目的是规划软件清晰的逻辑结构以便于开发维护，但在网站的发展过程中，分层结构对网站支持高并发向分布式方向的发展至关重要。

- **分隔**

如果说分层是将软件在横向方面进行切分，那么分隔就是在纵向方面对软件进行切分。

网站越大，功能越复杂，服务和数据处理的种类也越多，将这些不同的功能和服务分隔开来，包装成高内聚低耦合的模块单元，不仅有助于软件的开发维护也便于不同模块的分布式部署，提高网站的并发处理能力和功能扩展能力。

大型网站分隔的粒度可能会很小。比如在应用层，将不同业务进行分隔，例如将购物、论坛、搜索、广告分隔成不同的应用，有对立的团队负责，部署在不同的服务器上。

- **分布式**

对于大型网站，分层和分隔的一个主要目的是为了切分后的模块便于分布式部署，即将不同模块部署在不同的服务器上，通过远程调用协同工作。分布式意味着可以使用更多的计算机完同样的工作，计算机越多，CPU、内存、存储资源就越多，能过处理的并发访问和数据量就越大，进而能够为更多的用户提供服务。

在网站应用中，常用的分布式方案有一下几种.

1. `分布式应用和服务`：将分层和分隔后的应用和服务模块分布式部署，可以改善网站性能和并发性、加快开发和发布速度、减少数据库连接资源消耗。
2. `分布式静态资源`：网站的静态资源如JS、CSS、Logo图片等资源对立分布式部署，并采用独立的域名，即人们常说的动静分离。静态资源分布式部署可以减轻应用服务器的负载压力；通过使用独立域名加快浏览器并发加载的速度。
3. `分布式数据和存储`：大型网站需要处理以P为单位的海量数据，单台计算机无法提供如此大的存储空间，这些数据库需要分布式存储。
4. `分布式计算`：目前网站普遍使用Hadoop和MapReduce分布式计算框架进行此类批处理计算，其特点是移动计算而不是移动数据，将计算程序分发到数据所在的位置以加速计算和分布式计算。

- **集群**

对于用户访问集中的模块需要将独立部署的服务器集群化，即多台服务器部署相同的应用构成一个集群，通过负载均衡设备共同对外提供服务。

服务器集群能够为相同的服务提供更多的并发支持，因此当有更多的用户访问时，只需要向集群中加入新的机器即可；另外可以实现当其中的某台服务器发生故障时，可以通过负载均衡的失效转移机制将请求转移至集群中其他的服务器上，因此可以提高系统的可用性。

- **缓存**

缓存目的就是减轻服务器的计算，使数据直接返回给用户。在现在的软件设计中，缓存已经无处不在。具体实现有CDN、反向代理、本地缓存、分布式缓存等。

使用缓存有两个条件：访问数据热点不均衡，即某些频繁访问的数据需要放在缓存中；数据在某个时间段内有效，不过很快过期，否则会因为数据过期而脏读，影响数据的正确性。

- **异步**

使用异步，业务之间的消息传递不是同步调用，而是将一个业务操作分成多个阶段，每个阶段之间通过共享数据的方法异步执行进行协作。

具体实现则在单一服务器内部可用通过多线程共享内存对了的方式处理；在分布式系统中可用通过分布式消息队列来实现异步。

异步架构的典型就是生产者消费者方式，两者不存在直接调用。

- **冗余**

网站需要7×24小时连续运行，那么就得有相应的冗余机制，以防某台机器宕掉时无法访问，而冗余则可以通过部署至少两台服务器构成一个集群实现服务高可用。数据库除了定期备份还需要实现冷热备份。甚至可以在全球范围内部署灾备数据中心。

- **自动化**

具体有自动化发布过程，自动化代码管理、自动化测试、自动化安全检测、自动化部署、自动化监控、自动化报警、自动化失效转移、自动化失效恢复等。

- **安全**

网站在安全架构方面有许多模式：通过密码和手机校验码进行身份认证；登录、交易需要对网络通信进行加密；为了防止机器人程序滥用资源，需要使用验证码进行识别；对常见的XSS攻击、SQL注入需要编码转换；垃圾信息需要过滤等。

- **敏捷性**

积极接受需求变更，快速响应业务发展需求。

### [#](#_12-2-缓存) 12.2 缓存

#### [#](#谈谈架构中的缓存应用) 谈谈架构中的缓存应用？

缓存有各类特征，而且有不同介质的区别，那么实际工程中我们怎么去对缓存分类呢? 在目前的应用服务框架中，比较常见的，时根据缓存雨应用的藕合度，分为local cache（本地缓存）和remote cache（分布式缓存）：

- **本地缓存**：指的是在应用中的缓存组件，其最大的优点是应用和cache是在同一个进程内部，请求缓存非常快速，没有过多的网络开销等，在单应用不需要集群支持或者集群情况下各节点无需互相通知的场景下使用本地缓存较合适；同时，它的缺点也是应为缓存跟应用程序耦合，多个应用程序无法直接的共享缓存，各应用或集群的各节点都需要维护自己的单独缓存，对内存是一种浪费。
- **分布式缓存**：指的是与应用分离的缓存组件或服务，其最大的优点是自身就是一个独立的应用，与本地应用隔离，多个应用可直接的共享缓存。

目前各种类型的缓存都活跃在成千上万的应用服务中，还没有一种缓存方案可以解决一切的业务场景或数据类型，我们需要根据自身的特殊场景和背景，选择最适合的缓存方案。缓存的使用是程序员、架构师的必备技能，好的程序员能根据数据类型、业务场景来准确判断使用何种类型的缓存，如何使用这种缓存，以最小的成本最快的效率达到最优的目的。

#### [#](#在开发中缓存具体如何实现) 在开发中缓存具体如何实现？

- 本地缓存
  - 成员变量或局部变量实现， 比如map
  - 静态变量实现
  - Ehcache
  - Guava Cache
- 分布式缓存
  - Redis集群+ Spring Cache注解方式

#### [#](#缓存会有哪些问题-如何解决) 缓存会有哪些问题？如何解决？

参见redis缓存问题

#### [#](#使用缓存的经验) 使用缓存的经验？

不合理使用缓存非但不能提高系统的性能，还会成为系统的累赘，甚至风险。

- **频繁修改的数据**

如果缓存中保存的是频繁修改的数据，就会出现数据写入缓存后，应用还来不及读取缓存，数据就已经失效，徒增系统负担。一般来说，数据的读写比在2：1（写入一次缓存，在数据更新前至少读取两次）以上，缓存才有意义。

- **没有热点的访问**

如果应用系统访问数据没有热点，不遵循二八定律，那么缓存就没有意义。

- **数据不一致与脏读**

一般会对缓存的数据设置失效时间，一旦超过失效时间，就要从数据库中重新加载。因此要容忍一定时间的数据不一致，如卖家已经编辑了商品属性，但是需要过一段时间才能被买家看到。还有一种策略是数据更新立即更新缓存，不过这也会带来更多系统开销和事务一致性问题。

- **缓存可用性**

缓存会承担大部分数据库访问压力，数据库已经习惯了有缓存的日子，所以当缓存服务崩溃时，数据库会因为完全不能承受如此大压力而宕机，导致网站不可用。这种情况被称作缓存雪崩，发生这种故障，甚至不能简单地重启缓存服务器和数据库服务器来恢复。

实践中，有的网站通过缓存热备份等手段提高缓存可用性：当某台缓存服务器宕机时，将缓存访问切换到热备服务器上。但这种设计有违缓存的初衷，缓存根本就不应该当做一个可靠的数据源来使用。

通过分布式缓存服务器集群，将缓存数据分布到集群多台服务器上可在一定程度上改善缓存的可用性。当一台缓存服务器宕机时，只有部分缓存数据丢失，重新从数据库加载这部分数据不会产生很大的影响。

- **缓存预热warm up**

缓存中存放的是热点数据，热点数据又是缓存系统利用LRU（最近最久未用算法）对不断访问的数据筛选淘汰出来，这个过程需要花费较长的时间。新系统的缓存系统如果没有任何数据，在重建缓存数据的过程中，系统的性能和数据库负载都不太好，那么最好在缓存系统启动时就把热点数据加载好，这个缓存预加载手段叫缓存预热。对于一些元数据如城市地名列表、类目信息，可以在启动时加载数据库中全部数据到缓存进行预热。

- **避免缓存穿透**

如果因为不恰当的业务、或者恶意攻击持续高并发地请求某个不存在的数据，由于缓存没有保存该数据，所有的请求都会落到数据库上，会对数据库造成压力，甚至崩溃。一个简单的对策是将不存在的数据也缓存起来(其value为null)。

### [#](#_12-3-限流) 12.3 限流

#### [#](#什么是限流-三种限流的算法) 什么是限流？三种限流的算法？

每个系统都有服务的上线，所以当流量超过服务极限能力时，系统可能会出现卡死、崩溃的情况，所以就有了降级和限流。限流其实就是：当高并发或者瞬时高并发时，为了保证系统的稳定性、可用性，系统以牺牲部分请求为代价或者延迟处理请求为代价，保证系统整体服务可用。

令牌桶(Token Bucket)、漏桶(leaky bucket)和计数器算法是最常用的三种限流的算法:

- 令牌桶方式(Token Bucket)
  - Guava RateLimiter

令牌桶算法是网络流量整形（Traffic Shaping）和速率限制（Rate Limiting）中最常使用的一种算法。先有一个木桶，系统按照固定速度，往桶里加入Token，如果桶已经满了就不再添加。当有请求到来时，会各自拿走一个Token，取到Token 才能继续进行请求处理，没有Token 就拒绝服务。

![img](/images/arch/arch-xianliu-1.png)

这里如果一段时间没有请求时，桶内就会积累一些Token，下次一旦有突发流量，只要Token足够，也能一次处理，所以令牌桶算法的特点是*允许突发流量*。

- **漏桶**

水(请求)先进入到漏桶里,漏桶以一定的速度出水(接口有响应速率),当水流入速度过大会直接溢出（访问频率超过接口响应速率),然后就拒绝请求,可以看出漏桶算法能强行限制数据的传输速率。

![img](/images/arch/arch-xianliu-2.png)

可见这里有两个变量,一个是桶的大小,支持流量突发增多时可以存多少的水(burst),另一个是水桶漏洞的大小(rate)。

因为漏桶的漏出速率是固定的参数,所以,即使网络中不存在资源冲突(没有发生拥塞),漏桶算法也不能使流突发(burst)到端口速率.因此,漏桶算法对于存在突发特性的流量来说缺乏效率.

- 计数器

   计数器限流算法也是比较常用的，主要用来限制总并发数，比如数据库连接池大小、线程池大小、程序访问并发数等都是使用计数器算法。也是最简单粗暴的算法。 

  - 采用AtomicInteger 
    - 使用AomicInteger来进行统计当前正在并发执行的次数，如果超过域值就简单粗暴的直接响应给用户，说明系统繁忙，请稍后再试或其它跟业务相关的信息。
    - 弊端：使用 AomicInteger 简单粗暴超过域值就拒绝请求，可能只是瞬时的请求量高，也会拒绝请求。
  - 采用令牌Semaphore: 
    - 使用Semaphore信号量来控制并发执行的次数，如果超过域值信号量，则进入阻塞队列中排队等待获取信号量进行执行。如果阻塞队列中排队的请求过多超出系统处理能力，则可以在拒绝请求。
    - 相对Atomic优点：如果是瞬时的高并发，可以使请求在阻塞队列中排队，而不是马上拒绝请求，从而达到一个流量削峰的目的。
  - 采用ThreadPoolExecutor java线程池: 
    - 固定线程池大小,超出固定先线程池和最大的线程数,拒绝线程请求;

#### [#](#限流令牌桶和漏桶对比) 限流令牌桶和漏桶对比？

- 令牌桶是按照固定速率往桶中添加令牌，请求是否被处理需要看桶中令牌是否足够，当令牌数减为零时则拒绝新的请求；
- 漏桶则是按照常量固定速率流出请求，流入请求速率任意，当流入的请求数累积到漏桶容量时，则新流入的请求被拒绝；
- 令牌桶限制的是平均流入速率（允许突发请求，只要有令牌就可以处理，支持一次拿3个令牌，4个令牌），并允许一定程度突发流量；
- 漏桶限制的是常量流出速率（即流出速率是一个固定常量值，比如都是1的速率流出，而不能一次是1，下次又是2），从而平滑突发流入速率；
- 令牌桶允许一定程度的突发，而漏桶主要目的是平滑流入速率；
- 两个算法实现可以一样，但是方向是相反的，对于相同的参数得到的限流效果是一样的。

#### [#](#在单机情况下如何实现限流) 在单机情况下如何实现限流？

应用级限流方式只是单应用内的请求限流，不能进行全局限流。

1. 限流总资源数
2. 限流总并发/连接/请求数
3. 限流某个接口的总并发/请求数
4. 限流某个接口的时间窗请求数
5. 平滑限流某个接口的请求数
6. Guava RateLimiter

#### [#](#在分布式环境下如何实现限流) 在分布式环境下如何实现限流？

我们需要分布式限流和接入层限流来进行全局限流。

1. redis+lua实现中的lua脚本
2. 使用Nginx+Lua实现的Lua脚本

### [#](#_12-4-降级和熔断) 12.4 降级和熔断

#### [#](#为什么会有容错-一般有哪些方式解决容错相关问题) 为什么会有容错？一般有哪些方式解决容错相关问题？

服务之间的依赖关系，如果有被依赖的服务挂了以后，造成其它服务也会出现请求堆积、资源占用，慢慢扩散到所有服务，引发雪崩效应。

而容错就是要解决这类问题，常见的方式：

- **主动超时**：Http请求主动设置一个超时时间，超时就直接返回，不会造成服务堆积
- **限流**：限制最大并发数
- **熔断**：当错误数超过阈值时快速失败，不调用后端服务，同时隔一定时间放几个请求去重试后端服务是否能正常调用，如果成功则关闭熔断状态，失败则继续快速失败，直接返回。（此处有个重试，重试就是弹性恢复的能力）
- **隔离**：把每个依赖或调用的服务都隔离开来，防止级联失败引起整体服务不可用
- **降级**：服务失败或异常后，返回指定的默认信息

![img](/images/arch/arch-x-reduce-2.png)

#### [#](#谈谈你对服务降级的理解) 谈谈你对服务降级的理解？

由于爆炸性的流量冲击，对一些服务进行有策略的放弃，以此缓解系统压力，保证目前主要业务的正常运行。它主要是针对非正常情况下的应急服务措施：当此时一些业务服务无法执行时，给出一个统一的返回结果。

- **降级服务的特征**
  - 原因：整体负荷超出整体负载承受能力。
  - 目的：保证重要或基本服务正常运行，非重要服务延迟使用或暂停使用
  - 大小：降低服务粒度，要考虑整体模块粒度的大小，将粒度控制在合适的范围内
  - 可控性：在服务粒度大小的基础上增加服务的可控性，后台服务开关的功能是一项必要配置（单机可配置文件，其他可领用数据库和缓存），可分为手动控制和自动控制。
  - 次序：一般从外围延伸服务开始降级，需要有一定的配置项，重要性低的优先降级，比如可以分组设置等级1-10，当服务需要降级到某一个级别时，进行相关配置
- **降级方式**
  - 延迟服务：比如发表了评论，重要服务，比如在文章中显示正常，但是延迟给用户增加积分，只是放到一个缓存中，等服务平稳之后再执行。
  - 在粒度范围内关闭服务（片段降级或服务功能降级）：比如关闭相关文章的推荐，直接关闭推荐区
  - 页面异步请求降级：比如商品详情页上有推荐信息/配送至等异步加载的请求，如果这些信息响应慢或者后端服务有问题，可以进行降级；
  - 页面跳转（页面降级）：比如可以有相关文章推荐，但是更多的页面则直接跳转到某一个地址
  - 写降级：比如秒杀抢购，我们可以只进行Cache的更新，然后异步同步扣减库存到DB，保证最终一致性即可，此时可以将DB降级为Cache。
  - 读降级：比如多级缓存模式，如果后端服务有问题，可以降级为只读缓存，这种方式适用于对读一致性要求不高的场景。
- **降级预案** 在进行降级之前要对系统进行梳理，看看系统是不是可以丢卒保帅；从而梳理出哪些必须誓死保护，哪些可降级；比如可以参考日志级别设置预案：
  - 一般：比如有些服务偶尔因为网络抖动或者服务正在上线而超时，可以自动降级；
  - 警告：有些服务在一段时间内成功率有波动（如在95~100%之间），可以自动降级或人工降级，并发送告警；
  - 错误：比如可用率低于90%，或者数据库连接池被打爆了，或者访问量突然猛增到系统能承受的最大阀值，此时可以根据情况自动降级或者人工降级；
  - 严重错误：比如因为特殊原因数据错误了，此时需要紧急人工降级。
- **服务降级分类**
  - 降级按照是否自动化可分为：自动开关降级（超时、失败次数、故障、限流）和人工开关降级（秒杀、电商大促等）。
  - 降级按照功能可分为：读服务降级、写服务降级。
  - 降级按照处于的系统层次可分为：多级降级。
- **自动降级分类**
  - 超时降级：主要配置好超时时间和超时重试次数和机制，并使用异步机制探测回复情况
  - 失败次数降级：主要是一些不稳定的api，当失败调用次数达到一定阀值自动降级，同样要使用异步机制探测回复情况
  - 故障降级：比如要调用的远程服务挂掉了（网络故障、DNS故障、http服务返回错误的状态码、rpc服务抛出异常），则可以直接降级。降级后的处理方案有：默认值（比如库存服务挂了，返回默认现货）、兜底数据（比如广告挂了，返回提前准备好的一些静态页面）、缓存（之前暂存的一些缓存数据）
  - 限流降级: 当我们去秒杀或者抢购一些限购商品时，此时可能会因为访问量太大而导致系统崩溃，此时开发者会使用限流来进行限制访问量，当达到限流阀值，后续请求会被降级；降级后的处理方案可以是：排队页面（将用户导流到排队页面等一会重试）、无货（直接告知用户没货了）、错误页（如活动太火爆了，稍后重试）

#### [#](#什么是服务熔断-和服务降级有什么区别) 什么是服务熔断？和服务降级有什么区别？

熔断机制是应对雪崩效应的一种微服务链路保护机制，当扇出链路的某个微服务不可用或者响应时间太长时，会进行服务的降级，进而熔断该节点微服务的调用，快速返回”错误”的响应信息。

**和服务降级有什么区别**？

服务熔断对服务提供了proxy，防止服务不可能时，出现串联故障（cascading failure），导致雪崩效应。

服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑。

- 共性： 
  - 目的 -> 都是从可用性、可靠性出发，提高系统的容错能力。
  - 最终表现->使某一些应用不可达或不可用，来保证整体系统稳定。
  - 粒度 -> 一般都是服务级别，但也有细粒度的层面：如做到数据持久层、只许查询不许增删改等。
  - 自治 -> 对其自治性要求很高。都要求具有较高的自动处理机制。
- 区别： 
  - 触发原因 -> 服务熔断通常是下级服务故障引起；服务降级通常为整体系统而考虑。
  - 管理目标 -> 熔断是每个微服务都需要的，是一个框架级的处理；而服务降级一般是关注业务，对业务进行考虑，抓住业务的层级，从而决定在哪一层上进行处理：比如在IO层，业务逻辑层，还是在外围进行处理。
  - 实现方式 -> 代码实现中的差异。

#### [#](#如何设计服务的熔断) 如何设计服务的熔断？

- **异常处理**：调用受熔断器保护的服务的时候，我们必须要处理当服务不可用时的异常情况。这些异常处理通常需要视具体的业务情况而定。比如，如果应用程序只是暂时的功能降级，可能需要切换到其它的可替换的服务上来执行相同的任务或者获取相同的数据，或者给用户报告错误然后提示他们稍后重试。
- **异常的类型**：请求失败的原因可能有很多种。一些原因可能会比其它原因更严重。比如，请求会失败可能是由于远程的服务崩溃，这可能需要花费数分钟来恢复；也可能是由于服务器暂时负载过重导致超时。熔断器应该能够检查错误的类型，从而根据具体的错误情况来调整策略。比如，可能需要很多次超时异常才可以断定需要切换到断开状态，而只需要几次错误提示就可以判断服务不可用而快速切换到断开状态。
- **日志**：熔断器应该能够记录所有失败的请求，以及一些可能会尝试成功的请求，使得的管理员能够监控使用熔断器保护的服务的执行情况。 测试服务是否可用：在断开状态下，熔断器可以采用定期的ping远程的服务或者资源，来判断是否服务是否恢复，而不是使用计时器来自动切换到半断开状态。这种ping操作可以模拟之前那些失败的请求，或者可以使用通过调用远程服务提供的检查服务是否可用的方法来判断。
- **手动重置**：在系统中对于失败操作的恢复时间是很难确定的，提供一个手动重置功能能够使得管理员可以手动的强制将熔断器切换到闭合状态。同样的，如果受熔断器保护的服务暂时不可用的话，管理员能够强制的将熔断器设置为断开状态。 并发问题：相同的熔断器有可能被大量并发请求同时访问。熔断器的实现不应该阻塞并发的请求或者增加每次请求调用的负担。 资源的差异性：使用单个熔断器时，一个资源如果有分布在多个地方就需要小心。比如，一个数据可能存储在多个磁盘分区上(shard)，某个分区可以正常访问，而另一个可能存在暂时性的问题。在这种情况下，不同的错误响应如果混为一谈，那么应用程序访问的这些存在问题的分区的失败的可能性就会高，而那些被认为是正常的分区，就有可能被阻塞。
- **加快熔断器的熔断操作**:有时候，服务返回的错误信息足够让熔断器立即执行熔断操作并且保持一段时间。比如，如果从一个分布式资源返回的响应提示负载超重，那么应该等待几分钟后再重试。（HTTP协议定义了”HTTP 503 Service Unavailable”来表示请求的服务当前不可用，他可以包含其他信息比如，超时等）
- **重复失败请求**：当熔断器在断开状态的时候，熔断器可以记录每一次请求的细节，而不是仅仅返回失败信息，这样当远程服务恢复的时候，可以将这些失败的请求再重新请求一次。

#### [#](#服务熔断有哪些实现方案) 服务熔断有哪些实现方案？

- **Hystrix**

Spring Cloud Netflix Hystrix就是隔离措施的一种实现,可以设置在某种超时或者失败情形下断开依赖调用或者返回指定逻辑,从而提高分布式系统的稳定性. 流程图如下：

![img](/images/arch/arch-x-reduce-4.png)

- **Sentinel**

Sentinel 以流量为切入点，从流量控制、熔断降级、系统负载保护等多个维度保护服务的稳定性。分为两个部分:

1. 核心库（Java 客户端）不依赖任何框架/库，能够运行于所有 Java 运行时环境，同时对 Dubbo / Spring Cloud 等框架也有较好的支持。
2. 控制台（Dashboard）基于 Spring Boot 开发，打包后可以直接运行，不需要额外的 Tomcat 等应用容器。

主要特性：

![img](/images/arch/arch-x-reduce-6.png)

### [#](#_12-5-负载均衡) 12.5 负载均衡

#### [#](#什么是负载均衡-原理是什么) 什么是负载均衡？原理是什么？

负载均衡（Load Balance），意思是将负载（工作任务，访问请求）进行平衡、分摊到多个操作单元（服务器，组件）上进行执行。是解决高性能，单点故障（高可用），扩展性（水平伸缩）的终极解决方案。

- **负载均衡原理**

采用横向扩展的方式，通过添加机器来满足大型网站服务的处理能力。比如：一台机器不能满足，则增加两台或者多台机器，共同承担访问压力。这就是典型的集群和负载均衡架构：如下图：

![img](/images/arch/arch-x-lb-1.png)

1. 应用集群：将同一应用部署到多台机器上，组成处理集群，接收负载均衡设备分发的请求，进行处理，并返回相应数据。
2. 负载均衡设备：将用户访问的请求，根据负载均衡算法，分发到集群中的一台处理服务器。（一种把网络请求分散到一个服务器集群中的可用服务器上去的设备）

- **负载均衡的作用**（解决的问题）：

1.解决并发压力，提高应用处理性能（增加吞吐量，加强网络处理能力）；

2.提供故障转移，实现高可用；

3.通过添加或减少服务器数量，提供网站伸缩性（扩展性）；

4.安全防护；（负载均衡设备上做一些过滤，黑白名单等处理）

#### [#](#负载均衡有哪些分类) 负载均衡有哪些分类？

根据实现技术不同，可分为DNS负载均衡，HTTP负载均衡，IP负载均衡，链路层负载均衡等。

- **DNS负载均衡**

最早的负载均衡技术，利用域名解析实现负载均衡，在DNS服务器，配置多个A记录，这些A记录对应的服务器构成集群。大型网站总是部分使用DNS解析，作为第一级负载均衡。如下图：

![img](/images/arch/arch-x-lb-2.png)

**实践建议**

将DNS作为第一级负载均衡，A记录对应着内部负载均衡的IP地址，通过内部负载均衡将请求分发到真实的Web服务器上。一般用于互联网公司，复杂的业务系统不合适使用。如下图：

![img](/images/arch/arch-x-lb-3.png)

- **IP负载均衡**

在网络层通过修改请求目标地址进行负载均衡。

用户请求数据包，到达负载均衡服务器后，负载均衡服务器在操作系统内核进程获取网络数据包，根据负载均衡算法得到一台真实服务器地址，然后将请求目的地址修改为，获得的真实ip地址，不需要经过用户进程处理。

真实服务器处理完成后，响应数据包回到负载均衡服务器，负载均衡服务器，再将数据包源地址修改为自身的ip地址，发送给用户浏览器。如下图：

![img](/images/arch/arch-x-lb-4.png)

IP负载均衡，真实物理服务器返回给负载均衡服务器，存在两种方式：（1）负载均衡服务器在修改目的ip地址的同时修改源地址。将数据包源地址设为自身盘，即源地址转换（snat）。（2）将负载均衡服务器同时作为真实物理服务器集群的网关服务器。

- **链路层负载均衡**

在通信协议的数据链路层修改mac地址，进行负载均衡。

数据分发时，不修改ip地址，指修改目标mac地址，配置真实物理服务器集群所有机器虚拟ip和负载均衡服务器ip地址一致，达到不修改数据包的源地址和目标地址，进行数据分发的目的。

实际处理服务器ip和数据请求目的ip一致，不需要经过负载均衡服务器进行地址转换，可将响应数据包直接返回给用户浏览器，避免负载均衡服务器网卡带宽成为瓶颈。也称为直接路由模式（DR模式）。如下图：

![img](/images/arch/arch-x-lb-5.png)

实践建议：DR模式是目前使用最广泛的一种负载均衡方式。

- **混合型负载均衡**

由于多个服务器群内硬件设备、各自的规模、提供的服务等的差异，可以考虑给每个服务器群采用最合适的负载均衡方式，然后又在这多个服务器群间再一次负载均衡或群集起来以一个整体向外界提供服务（即把这多个服务器群当做一个新的服务器群），从而达到最佳的性能。将这种方式称之为混合型负载均衡。

此种方式有时也用于单台均衡设备的性能不能满足大量连接请求的情况下。是目前大型互联网公司，普遍使用的方式。

方式一，如下图：

![img](/images/arch/arch-x-lb-6.png)

以上模式适合有动静分离的场景，反向代理服务器（集群）可以起到缓存和动态请求分发的作用，当时静态资源缓存在代理服务器时，则直接返回到浏览器。如果动态页面则请求后面的应用负载均衡（应用集群）。

方式二，如下图：

![img](/images/arch/arch-x-lb-7.png)

以上模式，适合动态请求场景。

因混合模式，可以根据具体场景，灵活搭配各种方式，以上两种方式仅供参考。

#### [#](#常见的负载均衡服务器有哪些) 常见的负载均衡服务器有哪些？

平时我们常用的有四层负载均衡和七层负载均衡，四层的负载均衡是基于IP和端口实现的，七层的负载均衡是在四层的基础上，基于URL等信息实现。

- **四层负载均衡**

LVS：重量级软件，本身不支持正则表达式，部署起来比较麻烦，但是性能高，应用范围广，一般的大型互联网公司都有用到。

HAProxy：轻量级软件，支持的负载均衡策略非常多，较灵活。

Nginx：轻量级软件，支持的协议少（HTTP、HTTPS和Email协议），对于Session支持不友好。

- **七层负载均衡**

HAProxy：全面支持七层代理，灵活性高，支持Session会话保持。

Nginx：可以针对HTTP应用进行分流，正则规则灵活，支持高并发，部署简单。

Apache：性能较差，一般不考虑。

MySQL Proxy：官方的数据库中间件，可以实现读写分离，负载均衡等功能，但是对分表分库支持不完善（可选替代品：Atlas，Cobar，TDDL）。

#### [#](#常见的负载均衡的算法) 常见的负载均衡的算法？

常见的负载均衡算法包含:

**第一类，轮询法**

- 轮询法(Round Robin) 
  - 将请求按顺序轮流地分配到后端服务器上，它均衡地对待后端的每一台服务器，而不关心服务器实际的连接数和当前的系统负载。
- 加权轮询法(Weight Round Robin) 
  - 不同的后端服务器可能机器的配置和当前系统的负载并不相同，因此它们的抗压能力也不相同。给配置高、负载低的机器配置更高的权重，让其处理更多的请；而配置低、负载高的机器，给其分配较低的权重，降低其系统负载，加权轮询能很好地处理这一问题，并将请求顺序且按照权重分配到后端。
- 平滑加权轮询法(Smooth Weight Round Robin)

**第二类，随机法**

- 随机法(Random) 
  - 通过系统的随机算法，根据后端服务器的列表大小值来随机选取其中的一台服务器进行访问。由概率统计理论可以得知，随着客户端调用服务端的次数增多， 其实际效果越来越接近于平均分配调用量到后端的每一台服务器，也就是轮询的结果。
- 加权随机法(Weight Random) 
  - 与加权轮询法一样，加权随机法也根据后端机器的配置，系统的负载分配不同的权重。不同的是，它是按照权重随机请求后端服务器，而非顺序。

**第三类，哈希**

- 源地址哈希法(Hash) 
  - 源地址哈希的思想是根据获取客户端的IP地址，通过哈希函数计算得到的一个数值，用该数值对服务器列表的大小进行取模运算，得到的结果便是客服端要访问服务器的序号。采用源地址哈希法进行负载均衡，同一IP地址的客户端，当后端服务器列表不变时，它每次都会映射到同一台后端服务器进行访问。

**第四类，连接数法**

- 最小连接数法(Least Connections) 
  - 最小连接数算法比较灵活和智能，由于后端服务器的配置不尽相同，对于请求的处理有快有慢，它是根据后端服务器当前的连接情况，动态地选取其中当前积压连接数最少的一台服务器来处理当前的请求，尽可能地提高后端服务的利用效率，将负责合理地分流到每一台服务器。

### [#](#_12-6-灾备和故障转移) 12.6 灾备和故障转移

#### [#](#什么是容灾-一般基于什么实现) 什么是容灾？一般基于什么实现？

容灾是指为了保证关键业务和应用在经历各种灾难后，仍然能够最大限度的提供正常服务的所进行的一系列系统计划及建设和管理行为。

容灾能力**基于数据复制**和**故障转移**。

#### [#](#一般怎么实现灾备) 一般怎么实现灾备？

备份是对数据进行保护，容灾是在备份的基础上，保障企业的业务连续性，从这个层面，一般将容灾划分为数据容灾和应用容灾。

- **数据容灾**是指建立一个异地的数据系统，该系统是本地关键应用数据的一个实时复制。
- **应用容灾**是指在数据容灾的基础上，在异地建立一套完整的与本地生产系统相当的备份应用系统，在灾难发生时，备端系统迅速接管业务继续运行。

## [#](#_13-分布式) 13 分布式

> 分布式相关。

### [#](#_13-1-一致性算法) 13.1 一致性算法

#### [#](#什么是分布式系统的副本一致性-有哪些) 什么是分布式系统的副本一致性？有哪些？

分布式系统通过副本控制协议，使得从系统外部读取系统内部各个副本的数据在一定的约束条件下相同，称之为副本一致性(consistency)。副本一致性是针对分布式系统而言的，不是针对某一个副本而言。

**强一致性(strong consistency)**：任何时刻任何用户或节点都可以读到最近一次成功更新的副本数据。强一致性是程度最高的一致性要求，也是实践中最难以实现的一致性。

**单调一致性(monotonic consistency)**：任何时刻，任何用户一旦读到某个数据在某次更新后的值，这个用户不会再读到比这个值更旧的值。单调一致性是弱于强一致性却非常实用的一种一致性级别。因为通常来说，用户只关心从己方视角观察到的一致性，而不会关注其他用户的一致性情况。

**会话一致性(session consistency)**：任何用户在某一次会话内一旦读到某个数据在某次更新后的值，这个用户在这次会话过程中不会再读到比这个值更旧的值。会话一致性通过引入会话的概念，在单调一致性的基础上进一步放松约束，会话一致性只保证单个用户单次会话内数据的单调修改，对于不同用户间的一致性和同一用户不同会话间的一致性没有保障。实践中有许多机制正好对应会话的概念，例如php 中的session 概念。

**最终一致性(eventual consistency)**：最终一致性要求一旦更新成功，各个副本上的数据最终将达 到完全一致的状态，但达到完全一致状态所需要的时间不能保障。对于最终一致性系统而言，一个用户只要始终读取某一个副本的数据，则可以实现类似单调一致性的效果，但一旦用户更换读取的副本，则无法保障任何一致性。

**弱一致性(week consistency)**：一旦某个更新成功，用户无法在一个确定时间内读到这次更新的值，且即使在某个副本上读到了新的值，也不能保证在其他副本上可以读到新的值。弱一致性系统一般很难在实际中使用，使用弱一致性系统需要应用方做更多的工作从而使得系统可用。

#### [#](#在分布式系统中有哪些常见的一致性算法) 在分布式系统中有哪些常见的一致性算法？

- 分布式算法 - 一致性Hash算法 
  - 一致性Hash算法是个经典算法，Hash环的引入是为解决`单调性(Monotonicity)`的问题；虚拟节点的引入是为了解决`平衡性(Balance)`问题
- 分布式算法 - Paxos算法 
  - Paxos算法是Lamport宗师提出的一种基于消息传递的分布式一致性算法，使其获得2013年图灵奖。自Paxos问世以来就持续垄断了分布式一致性算法，Paxos这个名词几乎等同于分布式一致性, 很多分布式一致性算法都由Paxos演变而来
- 分布式算法 - Raft算法 
  - Paxos是出了名的难懂，而Raft正是为了探索一种更易于理解的一致性算法而产生的。它的首要设计目的就是易于理解，所以在选主的冲突处理等方式上它都选择了非常简单明了的解决方案
- 分布式算法 - ZAB算法 
  - ZAB 协议全称：Zookeeper Atomic Broadcast（Zookeeper 原子广播协议）, 它应该是所有一致性协议中生产环境中应用最多的了。为什么呢？因为他是为 Zookeeper 设计的分布式一致性协议！

#### [#](#谈谈你对一致性hash算法的理解) 谈谈你对一致性hash算法的理解？

判定哈希算法好坏的四个定义:

- `平衡性(Balance)`: 平衡性是指哈希的结果能够尽可能分布到所有的缓冲中去，这样可以使得所有的缓冲空间都得到利用。很多哈希算法都能够满足这一条件。
- `单调性(Monotonicity)`: 单调性是指如果已经有一些内容通过哈希分派到了相应的缓冲中，又有新的缓冲加入到系统中。哈希的结果应能够保证原有已分配的内容可以被映射到原有的或者新的缓冲中去，而不会被映射到旧的缓冲集合中的其他缓冲区。
- `分散性(Spread)`: 在分布式环境中，终端有可能看不到所有的缓冲，而是只能看到其中的一部分。当终端希望通过哈希过程将内容映射到缓冲上时，由于不同终端所见的缓冲范围有可能不同，从而导致哈希的结果不一致，最终的结果是相同的内容被不同的终端映射到不同的缓冲区中。这种情况显然是应该避免的，因为它导致相同内容被存储到不同缓冲中去，降低了系统存储的效率。分散性的定义就是上述情况发生的严重程度。好的哈希算法应能够尽量避免不一致的情况发生，也就是尽量降低分散性。
- `负载(Load)`: 负载问题实际上是从另一个角度看待分散性问题。既然不同的终端可能将相同的内容映射到不同的缓冲区中，那么对于一个特定的缓冲区而言，也可能被不同的用户映射为不同 的内容。与分散性一样，这种情况也是应当避免的，因此好的哈希算法应能够尽量降低缓冲的负荷。

![img](/images/alg/alg-dist-hash-8.jpg)

#### [#](#什么是paxos算法-如何实现的) 什么是Paxos算法？ 如何实现的？

Paxos算法是Lamport宗师提出的一种基于消息传递的分布式一致性算法，使其获得2013年图灵奖。

- **三个角色**？ 可以理解为人大代表(Proposer)在人大向其它代表(Acceptors)提案，通过后让老百姓(Learner)落实

Paxos将系统中的角色分为`提议者 (Proposer)`，`决策者 (Acceptor)`，和`最终决策学习者 (Learner)`:

1. `Proposer`: 提出提案 (Proposal)。Proposal信息包括提案编号 (Proposal ID) 和提议的值 (Value)。
2. `Acceptor`: 参与决策，回应Proposers的提案。收到Proposal后可以接受提案，若Proposal获得多数Acceptors的接受，则称该Proposal被批准。
3. `Learner`: 不参与决策，从Proposers/Acceptors学习最新达成一致的提案(Value)。

在多副本状态机中，每个副本同时具有Proposer、Acceptor、Learner三种角色。

![img](/images/alg/alg-dst-paxos-1.jpg)

- **基于消息传递的3个阶段**

![img](/images/alg/alg-dst-paxos-2.jpg)

1. 第一阶段: Prepare阶段

   ；Proposer向Acceptors发出Prepare请求，Acceptors针对收到的Prepare请求进行Promise承诺。 

   1. `Prepare`: Proposer生成全局唯一且递增的Proposal ID (可使用时间戳加Server ID)，向所有Acceptors发送Prepare请求，这里无需携带提案内容，只携带Proposal ID即可。

   2. ```
      Promise
      ```

      : Acceptors收到Prepare请求后，做出“两个承诺，一个应答”。 

      1. 承诺1: 不再接受Proposal ID小于等于(注意: 这里是<= )当前请求的Prepare请求;
      2. 承诺2: 不再接受Proposal ID小于(注意: 这里是< )当前请求的Propose请求;
      3. 应答: 不违背以前作出的承诺下，回复已经Accept过的提案中Proposal ID最大的那个提案的Value和Proposal ID，没有则返回空值。

2. 第二阶段: Accept阶段

   ; Proposer收到多数Acceptors承诺的Promise后，向Acceptors发出Propose请求，Acceptors针对收到的Propose请求进行Accept处理。 

   1. `Propose`: Proposer 收到多数Acceptors的Promise应答后，从应答中选择Proposal ID最大的提案的Value，作为本次要发起的提案。如果所有应答的提案Value均为空值，则可以自己随意决定提案Value。然后携带当前Proposal ID，向所有Acceptors发送Propose请求。
   2. `Accept`: Acceptor收到Propose请求后，在不违背自己之前作出的承诺下，接受并持久化当前Proposal ID和提案Value。

3. **第三阶段: Learn阶段**; Proposer在收到多数Acceptors的Accept之后，标志着本次Accept成功，决议形成，将形成的决议发送给所有Learners。

#### [#](#什么是raft算法) 什么是Raft算法？

**不同于Paxos算法直接从分布式一致性问题出发推导出来，Raft算法则是从多副本状态机的角度提出**。Raft实现了和Paxos相同的功能，它将一致性分解为多个子问题: Leader选举(Leader election)、日志同步(Log replication)、安全性(Safety)、日志压缩(Log compaction)、成员变更(Membership change)等。同时，**Raft算法使用了更强的假设来减少了需要考虑的状态，使之变的易于理解和实现**。

- **三个角色**

Raft将系统中的角色分为`领导者(Leader)`、`跟从者(Follower)`和`候选人(Candidate)`:

1. `Leader`: 接受客户端请求，并向Follower同步请求日志，当日志同步到大多数节点上后告诉Follower提交日志。
2. `Follower`: 接受并持久化Leader同步的日志，在Leader告之日志可以提交之后，提交日志。
3. `Candidate`: Leader选举过程中的临时角色。

![img](/images/alg/alg-dst-raft-1.jpg)

Raft要求系统在任意时刻最多只有一个Leader，正常工作期间只有Leader和Followers。

- **以子问题Leader选举为例**?

Raft 使用心跳(heartbeat)触发Leader选举。当服务器启动时，初始化为Follower。Leader向所有Followers周期性发送heartbeat。如果Follower在选举超时时间内没有收到Leader的heartbeat，就会等待一段随机的时间后发起一次Leader选举。

Follower将其当前term加一然后转换为Candidate。它首先给自己投票并且给集群中的其他服务器发送 RequestVote RPC (RPC细节参见八、Raft算法总结)。结果有以下三种情况:

- 赢得了多数的选票，成功选举为Leader；
- 收到了Leader的消息，表示有其它服务器已经抢先当选了Leader；
- 没有服务器赢得多数的选票，Leader选举失败，等待选举时间超时后发起下一次选举。

![img](/images/alg/alg-dst-raft-4.jpg)

选举出Leader后，Leader通过定期向所有Followers发送心跳信息维持其统治。若Follower一段时间未收到Leader的心跳则认为Leader可能已经挂了，再次发起Leader选举过程。

### [#](#_13-2-全局唯一id) 13.2 全局唯一ID

#### [#](#全局唯一id有哪些实现方案) 全局唯一ID有哪些实现方案？

常见的分布式ID生成方式，大致分类的话可以分为两类：

1. **一种是类DB型的**，根据设置不同起始值和步长来实现趋势递增，需要考虑服务的容错性和可用性;
2. **另一种是类snowflake型**，这种就是将64位划分为不同的段，每段代表不同的涵义，基本就是时间戳、机器ID和序列数。这种方案就是需要考虑时钟回拨的问题以及做一些 buffer的缓冲设计提高性能。

#### [#](#数据库方式实现方案-有什么缺陷) 数据库方式实现方案？有什么缺陷？

- **MySQL为例**

我们将分布式系统中数据库的同一个业务表的自增ID设计成不一样的起始值，然后设置固定的步长，步长的值即为分库的数量或分表的数量。

以MySQL举例，利用给字段设置`auto_increment_increment`和`auto_increment_offset`来保证ID自增。

1. `auto_increment_offset`：表示自增长字段从那个数开始，他的取值范围是1 .. 65535。
2. `auto_increment_increment`：表示自增长字段每次递增的量，其默认值是1，取值范围是1 .. 65535。

缺点也很明显，首先它**强依赖DB**，当DB异常时整个系统不可用。虽然配置主从复制可以尽可能的增加可用性，但是**数据一致性在特殊情况下难以保证**。主从切换时的不一致可能会导致重复发号。还有就是**ID发号性能瓶颈限制在单台MySQL的读写性能**。

- **使用redis实现**

Redis实现分布式唯一ID主要是通过提供像 `INCR` 和 `INCRBY` 这样的自增原子命令，由于Redis自身的单线程的特点所以能保证生成的 ID 肯定是唯一有序的。

但是单机存在性能瓶颈，无法满足高并发的业务需求，所以可以采用集群的方式来实现。集群的方式又会涉及到和数据库集群同样的问题，所以也需要设置分段和步长来实现。

为了避免长期自增后数字过大可以通过与当前时间戳组合起来使用，另外为了保证并发和业务多线程的问题可以采用 Redis + Lua的方式进行编码，保证安全。

Redis 实现分布式全局唯一ID，它的性能比较高，生成的数据是有序的，对排序业务有利，但是同样它依赖于redis，**需要系统引进redis组件，增加了系统的配置复杂性**。

当然现在Redis的使用性很普遍，所以如果其他业务已经引进了Redis集群，则可以资源利用考虑使用Redis来实现。

#### [#](#雪花算法如何实现的) 雪花算法如何实现的？

Snowflake，雪花算法是由Twitter开源的分布式ID生成算法，以划分命名空间的方式将 64-bit位分割成多个部分，每个部分代表不同的含义。而 Java中64bit的整数是Long类型，所以在 Java 中 SnowFlake 算法生成的 ID 就是 long 来存储的。

- **第1位**占用1bit，其值始终是0，可看做是符号位不使用。
- **第2位**开始的41位是时间戳，41-bit位可表示2^41个数，每个数代表毫秒，那么雪花算法可用的时间年限是`(1L<<41)/(1000L360024*365)`=69 年的时间。
- **中间的10-bit位**可表示机器数，即2^10 = 1024台机器，但是一般情况下我们不会部署这么台机器。如果我们对IDC（互联网数据中心）有需求，还可以将 10-bit 分 5-bit 给 IDC，分5-bit给工作机器。这样就可以表示32个IDC，每个IDC下可以有32台机器，具体的划分可以根据自身需求定义。
- **最后12-bit位**是自增序列，可表示2^12 = 4096个数。

这样的划分之后相当于**在一毫秒一个数据中心的一台机器上可产生4096个有序的不重复的ID**。但是我们 IDC 和机器数肯定不止一个，所以毫秒内能生成的有序ID数是翻倍的。

![img](/images/arch/arch-z-id-3.png)

#### [#](#雪花算法有什么问题-有哪些解决思路) 雪花算法有什么问题？有哪些解决思路？

- **有哪些问题**？

1. 时钟回拨问题；
2. 趋势递增，而不是绝对递增；
3. 不能在一台服务器上部署多个分布式ID服务；

- **如何解决时钟回拨**？

以百度的UidGenerator为例，CachedUidGenerator方式主要通过采取如下一些措施和方案规避了时钟回拨问题和增强唯一性：

1. **自增列**：UidGenerator的workerId在实例每次重启时初始化，且就是数据库的自增ID，从而完美的实现每个实例获取到的workerId不会有任何冲突。
2. **RingBuffer**：UidGenerator不再在每次取ID时都实时计算分布式ID，而是利用RingBuffer数据结构预先生成若干个分布式ID并保存。
3. **时间递增**：传统的雪花算法实现都是通过System.currentTimeMillis()来获取时间并与上一次时间进行比较，这样的实现严重依赖服务器的时间。而UidGenerator的时间类型是AtomicLong，且通过incrementAndGet()方法获取下一次的时间，从而脱离了对服务器时间的依赖，也就不会有时钟回拨的问题

（这种做法也有一个小问题，即分布式ID中的时间信息可能并不是这个ID真正产生的时间点，例如：获取的某分布式ID的值为3200169789968523265，它的反解析结果为{"timestamp":"2019-05-02 23:26:39","workerId":"21","sequence":"1"}，但是这个ID可能并不是在"2019-05-02 23:26:39"这个时间产生的）。

### [#](#_13-3-分布式锁) 13.3 分布式锁

#### [#](#有哪些方案实现分布式锁) 有哪些方案实现分布式锁？

综合讲讲方案：

- 使用场景 
  - 需要保证一个方法在同一时间内只能被同一个线程执行
- 实现方式: 
  - 加锁和解锁
- 方案,考虑因素(性能,稳定,实现难度,死锁) 
  - 基于数据库做分布式锁--乐观锁(基于版本号)和悲观锁(基于排它锁)
  - 基于 redis 做分布式锁:setnx(key,当前时间+过期时间)和Redlock机制
  - 基于 zookeeper 做分布式锁:临时有序节点来实现的分布式锁,Curator
  - 基于 Consul 做分布式锁

#### [#](#基于数据库如何实现分布式锁-有什么缺陷) 基于数据库如何实现分布式锁？有什么缺陷？

- **基于数据库表**（锁表，很少使用）

最简单的方式可能就是直接创建一张锁表，然后通过操作该表中的数据来实现了。当我们想要获得锁的时候，就可以在该表中增加一条记录，想要释放锁的时候就删除这条记录。

为了更好的演示，我们先创建一张数据库表，参考如下：

```sql
CREATE TABLE database_lock (
	`id` BIGINT NOT NULL AUTO_INCREMENT,
	`resource` int NOT NULL COMMENT '锁定的资源',
	`description` varchar(1024) NOT NULL DEFAULT "" COMMENT '描述',
	PRIMARY KEY (id),
	UNIQUE KEY uiq_idx_resource (resource)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='数据库分布式锁表';
```

当我们想要获得锁时，可以插入一条数据：

```sql
INSERT INTO database_lock(resource, description) VALUES (1, 'lock');
```

当需要释放锁的时，可以删除这条数据：

```sql
DELETE FROM database_lock WHERE resource=1;
```

- **基于悲观锁**

**悲观锁实现思路**？

1. 在对任意记录进行修改前，先尝试为该记录加上排他锁（exclusive locking）。
2. 如果加锁失败，说明该记录正在被修改，那么当前查询可能要等待或者抛出异常。 具体响应方式由开发者根据实际需要决定。
3. 如果成功加锁，那么就可以对记录做修改，事务完成后就会解锁了。
4. 其间如果有其他对该记录做修改或加排他锁的操作，都会等待我们解锁或直接抛出异常。

**以MySQL InnoDB中使用悲观锁为例**？

要使用悲观锁，我们必须关闭mysql数据库的自动提交属性，因为MySQL默认使用autocommit模式，也就是说，当你执行一个更新操作后，MySQL会立刻将结果进行提交。set autocommit=0;

```sql
//0.开始事务
begin;/begin work;/start transaction; (三者选一就可以)
//1.查询出商品信息
select status from t_goods where id=1 for update;
//2.根据商品信息生成订单
insert into t_orders (id,goods_id) values (null,1);
//3.修改商品status为2
update t_goods set status=2;
//4.提交事务
commit;/commit work;
```

上面的查询语句中，我们使用了`select…for update`的方式，这样就通过开启排他锁的方式实现了悲观锁。此时在t_goods表中，id为1的 那条数据就被我们锁定了，其它的事务必须等本次事务提交之后才能执行。这样我们可以保证当前的数据不会被其它事务修改。

上面我们提到，使用`select…for update`会把数据给锁住，不过我们需要注意一些锁的级别，MySQL InnoDB默认行级锁。行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁把整张表锁住，这点需要注意。

- **基于乐观锁**

乐观并发控制（又名“乐观锁”，Optimistic Concurrency Control，缩写“OCC”）是一种并发控制的方法。它假设多用户并发的事务在处理时不会彼此互相影响，各事务能够在不产生锁的情况下处理各自影响的那部分数据。在提交数据更新之前，每个事务会先检查在该事务读取数据后，有没有其他事务又修改了该数据。如果其他事务有更新的话，正在提交的事务会进行回滚。

**以使用版本号实现乐观锁为例？**

使用版本号时，可以在数据初始化时指定一个版本号，每次对数据的更新操作都对版本号执行+1操作。并判断当前版本号是不是该数据的最新的版本号。

```sql
1.查询出商品信息
select (status,status,version) from t_goods where id=#{id}
2.根据商品信息生成订单
3.修改商品status为2
update t_goods 
set status=2,version=version+1
where id=#{id} and version=#{version};
```

需要注意的是，乐观锁机制往往基于系统中数据存储逻辑，因此也具备一定的局限性。由于乐观锁机制是在我们的系统中实现的，对于来自外部系统的用户数据更新操作不受我们系统的控制，因此可能会造成脏数据被更新到数据库中。在系统设计阶段，我们应该充分考虑到这些情况，并进行相应的调整（如将乐观锁策略在数据库存储过程中实现，对外只开放基于此存储过程的数据更新途径，而不是将数据库表直接对外公开）。

- **缺陷**

对数据库依赖，开销问题，行锁变表锁问题，无法解决数据库单点和可重入的问题。

#### [#](#基于redis如何实现分布式锁-有什么缺陷) 基于redis如何实现分布式锁？有什么缺陷？

- **最基本的Jedis方案**

**加锁**： set NX PX + 重试 + 重试间隔

向Redis发起如下命令: SET productId:lock 0xx9p03001 NX PX 30000 其中，"productId"由自己定义，可以是与本次业务有关的id，"0xx9p03001"是一串随机值，必须保证全局唯一(原因在后文中会提到)，“NX"指的是当且仅当key(也就是案例中的"productId:lock”)在Redis中不存在时，返回执行成功，否则执行失败。"PX 30000"指的是在30秒后，key将被自动删除。执行命令后返回成功，表明服务成功的获得了锁。

```java
@Override
public boolean lock(String key, long expire, int retryTimes, long retryDuration) {
    // use JedisCommands instead of setIfAbsense
    boolean result = setRedis(key, expire);

    // retry if needed
    while ((!result) && retryTimes-- > 0) {
        try {
            log.debug("lock failed, retrying..." + retryTimes);
            Thread.sleep(retryDuration);
        } catch (Exception e) {
            return false;
        }

        // use JedisCommands instead of setIfAbsense
        result = setRedis(key, expire);
    }
    return result;
}

private boolean setRedis(String key, long expire) {
    try {
        RedisCallback<String> redisCallback = connection -> {
            JedisCommands commands = (JedisCommands) connection.getNativeConnection();
            String uuid = SnowIDUtil.uniqueStr();
            lockFlag.set(uuid);
            return commands.set(key, uuid, NX, PX, expire); // 看这里
        };
        String result = redisTemplate.execute(redisCallback);
        return !StringUtil.isEmpty(result);
    } catch (Exception e) {
        log.error("set redis occurred an exception", e);
    }
    return false;
}
```

**解锁**： 采用lua脚本： 在删除key之前，一定要判断服务A持有的value与Redis内存储的value是否一致。如果贸然使用服务A持有的key来删除锁，则会误将服务B的锁释放掉。

```lua
if redis.call("get", KEYS[1])==ARGV[1] then
	return redis.call("del", KEYS[1])
else
	return 0
end
```

- **基于RedLock实现分布式锁**

假设有两个服务A、B都希望获得锁，有一个包含了5个redis master的Redis Cluster，执行过程大致如下:

1. 客户端获取当前时间戳，单位: 毫秒
2. 服务A轮寻每个master节点，尝试创建锁。(这里锁的过期时间比较短，一般就几十毫秒) RedLock算法会尝试在大多数节点上分别创建锁，假如节点总数为n，那么大多数节点指的是n/2+1。
3. 客户端计算成功建立完锁的时间，如果建锁时间小于超时时间，就可以判定锁创建成功。如果锁创建失败，则依次(遍历master节点)删除锁。
4. 只要有其它服务创建过分布式锁，那么当前服务就必须轮寻尝试获取锁。

- **基于Redisson实现分布式锁**？

**过程**？

1. 线程去获取锁，获取成功: 执行lua脚本，保存数据到redis数据库。
2. 线程去获取锁，获取失败: 订阅了解锁消息，然后再尝试获取锁，获取成功后，执行lua脚本，保存数据到redis数据库。

**互斥**？

如果这个时候客户端B来尝试加锁，执行了同样的一段lua脚本。第一个if判断会执行“exists myLock”，发现myLock这个锁key已经存在。接着第二个if判断，判断myLock锁key的hash数据结构中，是否包含客户端B的ID，但明显没有，那么客户端B会获取到pttl myLock返回的一个数字，代表myLock这个锁key的剩余生存时间。此时客户端B会进入一个while循环，不听的尝试加锁。

**watch dog自动延时机制**？

客户端A加锁的锁key默认生存时间只有30秒，如果超过了30秒，客户端A还想一直持有这把锁，怎么办？其实只要客户端A一旦加锁成功，就会启动一个watch dog看门狗，它是一个后台线程，会每隔10秒检查一下，如果客户端A还持有锁key，那么就会不断的延长锁key的生存时间。

**可重入**？

每次lock会调用incrby，每次unlock会减一。

- **方案比较**

1. 借助Redis实现分布式锁时，有一个共同的缺陷: 当获取锁被决绝后，需要不断的循环，重新发送获取锁(创建key)的请求，直到请求成功。这就造成空转，浪费宝贵的CPU资源。
2. RedLock算法本身有争议，并不能保证健壮性。
3. Redisson实现分布式锁时，除了将key新增到某个指定的master节点外，还需要由master自动异步的将key和value等数据同步至绑定的slave节点上。那么问题来了，如果master没来得及同步数据，突然发生宕机，那么通过故障转移和主备切换，slave节点被迅速升级为master节点，新的客户端加锁成功，旧的客户端的watch dog发现key存在，误以为旧客户端仍然持有这把锁，这就导致同时存在多个客户端持有同名锁的问题了。

#### [#](#基于zookeeper如何实现分布式锁) 基于zookeeper如何实现分布式锁？

说几个核心点：

- **顺序节点**

创建一个用于发号的节点“/test/lock”，然后以它为父亲节点的前缀为“/test/lock/seq-”依次发号：

![img](/images/zk/zk-1.png)

- **获得最小号得锁**

由于序号的递增性，可以规定排号最小的那个获得锁。所以，每个线程在尝试占用锁之前，首先判断自己是排号是不是当前最小，如果是，则获取锁。

- **节点监听机制**

每个线程抢占锁之前，先抢号创建自己的ZNode。同样，释放锁的时候，就需要删除抢号的Znode。抢号成功后，如果不是排号最小的节点，就处于等待通知的状态。等谁的通知呢？不需要其他人，只需要等前一个Znode 的通知就可以了。当前一个Znode 删除的时候，就是轮到了自己占有锁的时候。第一个通知第二个、第二个通知第三个，击鼓传花似的依次向后。

### [#](#_13-4-分布式事务) 13.4 分布式事务

#### [#](#什么是acid) 什么是ACID？

一个事务有四个基本特性，也就是我们常说的（ACID）：

1. **Atomicity（原子性）**：事务是一个不可分割的整体，事务内所有操作要么全做成功，要么全失败。
2. **Consistency（一致性）**：事务执行前后，数据从一个状态到另一个状态必须是一致的（A向B转账，不能出现A扣了钱，B却没收到）。
3. **Isolation（隔离性）**： 多个并发事务之间相互隔离，不能互相干扰。
4. **Durability（持久性）**：事务完成后，对数据库的更改是永久保存的，不能回滚。

#### [#](#分布式事务有哪些解决方案) 分布式事务有哪些解决方案？

#### [#](#什么是分布式的xa协议) 什么是分布式的XA协议？

XA协议是一个基于**数据库**的**分布式事务协议**，其分为两部分：**事务管理器**和**本地资源管理器**。事务管理器作为一个全局的调度者，负责对各个本地资源管理器统一号令提交或者回滚。`二阶提交协议（2PC）`和`三阶提交协议（3PC）`就是根据此协议衍生出来而来。主流的诸如Oracle、MySQL等数据库均已实现了XA接口。

XA接口是双向的系统接口，在事务管理器（Transaction Manager）以及一个或多个资源管理器（Resource Manager）之间形成通信桥梁。也就是说，在基于XA的一个事务中，我们可以针对多个资源进行事务管理，例如一个系统访问多个数据库，或即访问数据库、又访问像消息中间件这样的资源。这样我们就能够实现在多个数据库和消息中间件直接实现全部提交、或全部取消的事务。**XA规范不是java的规范，而是一种通用的规范**。

#### [#](#什么是2pc) 什么是2PC？

两段提交顾名思义就是要进行两个阶段的提交：

- 第一阶段，准备阶段（投票阶段）；
- 第二阶段，提交阶段（执行阶段）。

![img](/images/arch/arch-z-trans-3.png)

下面还拿下单扣库存举例子，简单描述一下两段提交（2PC）的原理：

之前说过业务服务化（SOA）以后，一个下单流程就会用到多个服务，各个服务都无法保证调用的其他服务的成功与否，这个时候就需要一个全局的角色（**协调者**）对各个服务（**参与者**）进行协调。

![img](/images/arch/arch-z-trans-4.png)

一个下单请求过来通过协调者，给每一个参与者发送Prepare消息，执行本地数据脚本但不提交事务。

如果协调者收到了参与者的失败消息或者超时，直接给每个参与者发送回滚(Rollback)消息；否则，发送提交（Commit）消息；参与者根据协调者的指令执行提交或者回滚操作，释放所有事务处理过程中被占用的资源，显然2PC做到了所有操作要么全部成功、要么全部失败。

**两段提交（2PC）的缺点**：

二阶段提交看似能够提供原子性的操作，但它存在着严重的缺陷：

- **网络抖动导致的数据不一致**：第二阶段中协调者向参与者发送commit命令之后，一旦此时发生网络抖动，导致一部分参与者接收到了commit请求并执行，可其他未接到commit请求的参与者无法执行事务提交。进而导致整个分布式系统出现了数据不一致。
- **超时导致的同步阻塞问题**：2PC中的所有的参与者节点都为事务阻塞型，当某一个参与者节点出现通信超时，其余参与者都会被动阻塞占用资源不能释放。
- **单点故障的风险**：由于严重的依赖协调者，一旦协调者发生故障，而此时参与者还都处于锁定资源的状态，无法完成事务commit操作。虽然协调者出现故障后，会重新选举一个协调者，可无法解决因前一个协调者宕机导致的参与者处于阻塞状态的问题。

#### [#](#什么是3pc) 什么是3PC?

三段提交（3PC）是对两段提交（2PC）的一种升级优化，**3PC在2PC的第一阶段和第二阶段中插入一个准备阶段**。保证了在最后提交阶段之前，各参与者节点的状态都一致。同时在协调者和参与者中都引入超时机制，当参与者各种原因未收到协调者的commit请求后，会对本地事务进行commit，不会一直阻塞等待，解决了2PC的单点故障问题，但3PC还是没能从根本上解决数据一致性的问题。

![img](/images/arch/arch-z-trans-5.png)

**3PC的三个阶段分别是CanCommit、PreCommit、DoCommit**：

- **CanCommit**：协调者向所有参与者发送CanCommit命令，询问是否可以执行事务提交操作。如果全部响应YES则进入下一个阶段。
- **PreCommit**：协调者向所有参与者发送PreCommit命令，询问是否可以进行事务的预提交操作，参与者接收到PreCommit请求后，如参与者成功的执行了事务操作，则返回Yes响应，进入最终commit阶段。一旦参与者中有向协调者发送了No响应，或因网络造成超时，协调者没有接到参与者的响应，协调者向所有参与者发送abort请求，参与者接受abort命令执行事务的中断。
- **DoCommit**：在前两个阶段中所有参与者的响应反馈均是YES后，协调者向参与者发送DoCommit命令正式提交事务，如协调者没有接收到参与者发送的ACK响应，会向所有参与者发送abort请求命令，执行事务的中断。

#### [#](#什么是tcc) 什么是TCC？

TCC（Try-Confirm-Cancel）又被称补偿事务，TCC与2PC的思想很相似，事务处理流程也很相似，但**2PC是应用于在DB层面，TCC则可以理解为在应用层面的2PC，是需要我们编写业务逻辑来实现**。

TCC它的核心思想是："针对每个操作都要注册一个与其对应的确认（Try）和补偿（Cancel）"。

还拿下单扣库存解释下它的三个操作：

- **Try阶段**：下单时通过Try操作去扣除库存预留资源。
- **Confirm阶段**：确认执行业务操作，在只预留的资源基础上，发起购买请求。
- **Cancel阶段**：只要涉及到的相关业务中，有一个业务方预留资源未成功，则取消所有业务资源的预留请求。

![img](/images/arch/arch-z-trans-6.png)

**TCC的缺点**：

- 应用侵入性强：TCC由于基于在业务层面，至使每个操作都需要有try、confirm、cancel三个接口。
- 开发难度大：代码开发量很大，要保证数据一致性confirm和cancel接口还必须实现幂等性。

#### [#](#什么是saga方案) 什么是SAGA方案？

### [#](#_13-5-分布式缓存) 13.5 分布式缓存

#### [#](#分布式系统中常用的缓存方案有哪些) 分布式系统中常用的缓存方案有哪些？

- 客户端缓存：页面和浏览器缓存，APP缓存，H5缓存，localStorage和sessionStorage
- CDN缓存： 
  - 内存存储：数据的缓存
  - 内容分发：负载均衡
- nginx缓存：本地缓存，外部缓存
- 数据库缓存：持久层缓存（mybatis，hibernate多级缓存），Mysql查询缓存
- 操作系统缓存：Page Cache，Buffer Cache

#### [#](#分布式系统缓存的更新模式) 分布式系统缓存的更新模式？

- **Cache Aside模式**

1. 读取失效：cache数据没有命中，查询DB，成功后把数据写入缓存
2. 读取命中：读取cache数据
3. 更新：把数据更新到DB，失效缓存

![img](/images/db/redis/cache-1.png)

```java
// Read
data = cache.get(id);
if (data == null) {
    data = db.get(id);
    cache.put(id, data);
}

// Write
db.save(data);
cache.invalid(data.id);
```

- **Read/Write Through模式**

缓存代理了DB读取、写入的逻辑，可以把缓存看成唯一的存储。

![img](/images/db/redis/cache-2.png)

- Write Back模式

这种模式下所有的操作都走缓存，缓存里的数据再通过**异步的方式同步**到数据库里面。所以系统的写性能能够大大提升了。

![img](/images/db/redis/cache-3.png)

#### [#](#分布式系统缓存淘汰策略) 分布式系统缓存淘汰策略

缓存淘汰，又称为缓存逐出(cache replacement algorithms或者cache replacement policies)，是指在存储空间不足的情况下，缓存系统主动释放一些缓存对象获取更多的存储空间。一般LRU用的比较多，可以重点了解一下。

- **FIFO** 先进先出（First In First Out）是一种简单的淘汰策略，缓存对象以队列的形式存在，如果空间不足，就释放队列头部的（先缓存）对象。一般用链表实现。
- **LRU** 最近最久未使用（Least Recently Used），这种策略是根据访问的时间先后来进行淘汰的，如果空间不足，会释放最久没有访问的对象（上次访问时间最早的对象）。比较常见的是通过优先队列来实现。
- **LFU** 最近最少使用（Least Frequently Used），这种策略根据最近访问的频率来进行淘汰，如果空间不足，会释放最近访问频率最低的对象。这个算法也是用优先队列实现的比较常见。

更进一步的谈谈Redis缓存淘汰的8个模式，可以参考上文Redis问答部分。

### [#](#_13-6-分布式任务) 13.6 分布式任务

#### [#](#java中定时任务是有些-如何演化的) Java中定时任务是有些？如何演化的？

这里主要讲讲Java的定时任务是如何一步步发展而来的：

- **Timer**

```java
new Timer("testTimer").schedule(new TimerTask() {
    @Override
    public void run() {
        System.out.println("TimerTask");
    }
}, 1000,2000);
```

解释：1000ms是延迟启动时间，2000ms是定时任务周期，每2s执行一次

- **ScheduledExecutorService**

```java
ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(10);
scheduledExecutorService.scheduleAtFixedRate(new Runnable() {
    @Override
    public void run() {
        System.out.println("ScheduledTask");
    }
}, 1, 1, TimeUnit.SECONDS);
```

解释：延迟1s启动，每隔1s执行一次，是前一个任务开始时就开始计算时间间隔，但是会等上一个任务结束在开始下一个

- **SpringTask**

```java
@Service
public class SpringTask {
    private static final Logger log = LoggerFactory.getLogger(SpringTask.class);

    @Scheduled(cron = "1/5 * * * * *")
    public void task1(){
        log.info("springtask 定时任务！");
    }
	
	@Scheduled(initialDelay = 1000,fixedRate = 1*1000)
    public void task2(){
        log.info("springtask 定时任务！");
    }
}
```

解释：

1. task1是每隔5s执行一次，{秒} {分} {时} {日期}
2. task2是延迟1s,每隔1S执行一次

- **Quartz**

quartz 是一个开源的分布式调度库，它基于java实现。

![img](/images/job/job-quartz-x1.png)

1. Job 表示一个任务，要执行的具体内容。
2. JobDetail 表示一个具体的可执行的调度程序，Job 是这个可执行程调度程序所要执行的内容，另外 JobDetail 还包含了这个任务调度的方案和策略。
3. Trigger 代表一个调度参数的配置，什么时候去调。
4. Scheduler 代表一个调度容器，一个调度容器中可以注册多个 JobDetail 和 Trigger。当 Trigger 与 JobDetail 组合，就可以被 Scheduler 容器调度了。

```java
//创建调度器Schedule
SchedulerFactory schedulerFactory = new StdSchedulerFactory();
Scheduler scheduler = schedulerFactory.getScheduler();
//创建JobDetail实例，并与HelloWordlJob类绑定
JobDetail jobDetail = JobBuilder.newJob(HelloWorldJob.class).withIdentity("job1", "jobGroup1")
        .build();
//创建触发器Trigger实例(立即执行，每隔1S执行一次)
Trigger trigger = TriggerBuilder.newTrigger()
        .withIdentity("trigger1", "triggerGroup1")
        .startNow()
        .withSchedule(SimpleScheduleBuilder.simpleSchedule().withIntervalInSeconds(1).repeatForever())
        .build();
//开始执行
scheduler.scheduleJob(jobDetail, trigger);
scheduler.start();
```

#### [#](#常见的job实现方案) 常见的JOB实现方案？

基于上面Java任务演化出分布式Job方案：

- **quartz**

JDBCJobStore 支持集群所有触发器和job都存储在数据库中无论服务器停止和重启都可以恢复任务同时支持事务处理。

![img](/images/job/job-quartz-x2.png)

- **elastic-job**

elastic-job 是由当当网基于quartz 二次开发之后的分布式调度解决方案 ， 由两个相对独立的子项目Elastic-Job-Lite和Elastic-Job-Cloud组成 。

Elastic-Job-Lite定位为轻量级无中心化解决方案，使用jar包的形式提供分布式任务的协调服务。

Elastic-Job-Cloud使用Mesos + Docker(TBD)的解决方案，额外提供资源治理、应用分发以及进程隔离等服务

![img](/images/job/job-x4.jpg)

亮点：

1. 基于quartz 定时任务框架为基础的，因此具备quartz的大部分功能
2. 使用zookeeper做协调，调度中心，更加轻量级
3. 支持任务的分片
4. 支持弹性扩容 ， 可以水平扩展 ， 当任务再次运行时，会检查当前的服务器数量，重新分片，分片结束之后才会继续执行任务
5. 失效转移，容错处理，当一台调度服务器宕机或者跟zookeeper断开连接之后，会立即停止作业，然后再去寻找其他空闲的调度服务器，来运行剩余的任务
6. 提供运维界面，可以管理作业和注册中心。

- **xxl-job**

个轻量级分布式任务调度框架 ，主要分为 调度中心和执行器两部分 ， 调度中心在启动初始化的时候，会默认生成执行器的RPC代理

对象(http协议调用)， 执行器项目启动之后， 调度中心在触发定时器之后通过jobHandle 来调用执行器项目里面的代码，核心功能和elastic-job差不多

![img](/images/job/job-x3.png)

### [#](#_13-7-分布式会话) 13.7 分布式会话

#### [#](#cookie和session有什么区别) Cookie和Session有什么区别？

cookie和session的方案虽然分别属于客户端和服务端，但是服务端的session的实现对客户端的cookie有依赖关系的，服务端执行session机制时候会生成session的id值，这个id值会发送给客户端，客户端每次请求都会把这个id值放到http请求的头部发送给服务端，而这个id值在客户端会保存下来，保存的容器就是cookie，因此当我们完全禁掉浏览器的cookie的时候，服务端的session也会不能正常使用。

#### [#](#谈谈会话技术的发展) 谈谈会话技术的发展？

- 单机 - Session + Cookie
- 多机器 
  - 在负载均衡侧 - Session 粘滞
  - Session数据同步
- 多机器，集群 - session集中管理，比如redis；目前方案上用的最多的是SpringSession，早前也有用tomcat集成方式的。
- 无状态token，比如JWT

#### [#](#分布式会话有哪些解决方案) 分布式会话有哪些解决方案？

- Session Stick
- Session Replication
- Session 数据集中存储
- Cookie Based
- JWT

#### [#](#什么是session-stick) 什么是Session Stick?

方案即将客户端的每次请求都转发至同一台服务器，这就需要负载均衡器能够根据每次请求的会话标识（SessionId）来进行请求转发，如下图所示。

![img](/images/arch/arch-z-session-1.png)

这种方案实现比较简单，对于Web服务器来说和单机的情况一样。但是可能会带来如下问题：

- 如果有一台服务器宕机或者重启，那么这台机器上的会话数据会全部丢失。
- 会话标识是应用层信息，那么负载均衡要将同一个会话的请求都保存到同一个Web服务器上的话，就需要进行应用层（第7层）的解析，这个开销比第4层大。
- 负载均衡器将变成一个有状态的节点，要将会话保存到具体Web服务器的映射。和无状态节点相比，内存消耗更大，容灾方面也会更麻烦。

PS：为什么这种方案到目前还有很多项目使用呢？因为不需要在项目代码侧改动，而是只需要在负载均衡侧改动。

#### [#](#什么是session-replication) 什么是Session Replication？

Session Replication 的方案则不对负载均衡器做更改，而是在Web服务器之间增加了会话数据同步的功能，各个服务器之间通过同步保证不同Web服务器之间的Session数据的一致性，如下图所示。

![img](/images/arch/arch-z-session-2.png)

Session Replication 方案对负载均衡器不再有要求，但是同样会带来以下问题：

- 同步Session数据会造成额外的网络带宽的开销，只要Session数据有变化，就需要将新产生的Session数据同步到其他服务器上，服务器数量越多，同步带来的网络带宽开销也就越大。
- 每台Web服务器都需要保存全部的Session数据，如果整个集群的Session数量太多的话，则对于每台机器用于保存Session数据的占用会很严重。

#### [#](#什么是session-数据集中存储) 什么是Session 数据集中存储？

Session 数据集中存储方案则是将集群中的所有Session集中存储起来，Web服务器本身则并不存储Session数据，不同的Web服务器从同样的地方来获取Session，如下图所示。

![img](/images/arch/arch-z-session-3.png)

相对于Session Replication方案，此方案的Session数据将不保存在本机，并且Web服务器之间也没有了Session数据的复制，但是该方案存在的问题在于：

- 读写Session数据引入了网络操作，这相对于本机的数据读取来说，问题就在于存在时延和不稳定性，但是通信发生在内网，则问题不大。
- 如果集中存储Session的机器或集群出现问题，则会影响应用。

#### [#](#什么是cookie-based-session) 什么是Cookie Based Session?

Cookie Based 方案是将**Session数据放在Cookie里**，访问Web服务器的时候，再由Web服务器生成对应的Session数据，如下图所示。

![img](/images/arch/arch-z-session-4.png)

但是Cookie Based 方案依然存在不足：

- Cookie长度的限制。这会导致Session长度的限制。
- 安全性。Seesion数据本来是服务端数据，却被保存在了客户端，即使可以加密，但是依然存在不安全性。
- 带宽消耗。这里不是指内部Web服务器之间的宽带消耗，而是数据中心的整体外部带宽的消耗。
- 性能影响。每次HTTP请求和响应都带有Seesion数据，对Web服务器来说，在同样的处理情况下，响应的结果输出越少，支持的并发就会越高。

#### [#](#什么是jwt-使用jwt的流程-对比传统的会话有啥区别) 什么是JWT？使用JWT的流程？对比传统的会话有啥区别？

JSON Web Token，一般用它来替换掉Session实现数据共享。

使用基于 Token 的身份验证方法，在服务端不需要存储用户的登录记录。大概的流程是这样的：

- 1、客户端通过用户名和密码登录服务器；
- 2、服务端对客户端身份进行验证；
- 3、服务端对该用户生成Token，返回给客户端；
- 4、客户端将Token保存到本地浏览器，一般保存到cookie中；
- 5、客户端发起请求，需要携带该Token；
- 6、服务端收到请求后，首先验证Token，之后返回数据。

![img](/images/arch/arch-z-session-5.png)

如上图为Token实现方式，浏览器第一次访问服务器，根据传过来的唯一标识userId，服务端会通过一些算法，如常用的HMAC-SHA256算法，然后加一个密钥，生成一个token，然后通过BASE64编码一下之后将这个token发送给客户端；客户端将token保存起来，下次请求时，带着token，服务器收到请求后，然后会用相同的算法和密钥去验证token，如果通过，执行业务操作，不通过，返回不通过信息。

可以对比下图session实现方式，流程大致一致。

![img](/images/arch/arch-z-session-6.png)

**优点**：

- 无状态、可扩展 ：在客户端存储的Token是无状态的，并且能够被扩展。基于这种无状态和不存储Session信息，负载均衡器能够将用户信息从一个服务传到其他服务器上。
- 安全：请求中发送token而不再是发送cookie能够防止CSRF(跨站请求伪造)。
- 可提供接口给第三方服务：使用token时，可以提供可选的权限给第三方应用程序。
- 多平台跨域

对应用程序和服务进行扩展的时候，需要介入各种各种的设备和应用程序。 假如我们的后端api服务器a.com只提供数据，而静态资源则存放在cdn 服务器b.com上。当我们从a.com请求b.com下面的资源时，由于触发浏览器的同源策略限制而被阻止。

**我们通过CORS（跨域资源共享）标准和token来解决资源共享和安全问题**。

举个例子，我们可以设置b.com的响应首部字段为：

```bash
// 第一行指定了允许访问该资源的外域 URI。
Access-Control-Allow-Origin: http://a.com

// 第二行指明了实际请求中允许携带的首部字段，这里加入了Authorization，用来存放token。
Access-Control-Allow-Headers: Authorization, X-Requested-With, Content-Type, Accept

// 第三行用于预检请求的响应。其指明了实际请求所允许使用的 HTTP 方法。
Access-Control-Allow-Methods: GET, POST, PUT,DELETE

// 然后用户从a.com携带有一个通过了验证的token访问B域名，数据和资源就能够在任何域上被请求到。
```



### [#](#_13-8-常见系统设计) 13.8 常见系统设计

#### [#](#如何设计一个秒杀系统) 如何设计一个秒杀系统？

- **秒杀特点及思路？**

短时间内，大量用户涌入，集中读和写有限的库存。

1. 尽量将请求拦截在系统上游（越上游越好）；
2. 读多写少的多使用缓存（缓存抗读压力）；

- **从分层角度理解？**

层层拦截，将请求尽量拦截在系统上游，避免将锁冲落到数据库上。

- 第一层：客户端优化

产品层面，用户点击“查询”或者“购票”后，按钮置灰，禁止用户重复提交请求； JS层面，限制用户在x秒之内只能提交一次请求，比如微信摇一摇抢红包。 基本可以拦截80%的请求。

- 第二层：站点层面的请求拦截（nginx层，写流控模块）

怎么防止程序员写for循环调用，有去重依据么? IP? cookie-id? …想复杂了，这类业务都需要登录，用uid即可。在站点层面，对uid进行请求计数和去重，甚至不需要统一存储计数，直接站点层内存存储（这样计数会不准，但最简单，比如guava本地缓存）。一个uid，5秒只准透过1个请求，这样又能拦住99%的for循环请求。 对于5s内的无效请求，统一返回错误提示或错误页面。

这个方式拦住了写for循环发HTTP请求的程序员，有些高端程序员（黑客）控制了10w个肉鸡，手里有10w个uid，同时发请求（先不考虑实名制的问题，小米抢手机不需要实名制），这下怎么办，站点层按照uid限流拦不住了。

- 第三层：服务层拦截

方案一：写请求放到队列中，每次只透有限的写请求到数据层，如果成功了再放下一批，直到库存不够，队列里的写请求全部返回“已售完”。

方案二：或采用漏斗机制，只放一倍的流量进来，多余的返回“已售完”，把写压力转换成读压力。 读请求，用cache，redis单机可以抗10W QPS,用异步线程定时更新缓存里的库存值。

还有提示“模糊化”，比如火车余票查询，票剩了58张，还是26张，你真的关注么，其实我们只关心有票和无票。

- 第四层：数据库层

浏览器拦截了80%，站点层拦截了99.9%并做了页面缓存，服务层又做了写请求队列与数据缓存，每次透到数据库层的请求都是可控的。 db基本就没什么压力了，通过自身锁机制来控制，避免出现超卖。

- **从架构角度理解？**

1. 高性能

   1. 动静分离

       秒杀过程中你是不需要刷新整个页面的，只有时间在不停跳动。这是因为一般都会对大流量的秒杀系统做系统的静态化改造，即数据意义上的动静分离。动静分离三步走： 

      1. 数据拆分；
      2. 静态缓存；
      3. 数据整合。

   2. **热点优化** 数据的热点优化与动静分离是不一样的，热点优化是基于二八原则对数据进行了纵向拆分，以便进行针对性地处理。热点识别和隔离不仅对“秒杀”这个场景有意义，对其他的高性能分布式系统也非常有参考价值。

   3. 系统优化

      1. 减少序列化：减少 Java 中的序列化操作可以很好的提升系统性能。序列化大部分是在 RPC 阶段发生，因此应该尽量减少 RPC 调用，一种可行的方案是将多个关联性较强的应用进行 “合并部署”，从而减少不同应用之间的 RPC 调用（微服务设计规范）
      2. 直接输出流数据：只要涉及字符串的I/O操作，无论是磁盘 I/O 还是网络 I/O，都比较耗费 CPU 资源，因为字符需要转换成字节，而这个转换又必须查表编码。所以对于常用数据，比如静态字符串，推荐提前编码成字节并缓存，具体到代码层面就是通过 OutputStream() 类函数从而减少数据的编码转换；另外，热点方法toString()不要直接调用ReflectionToString实现，推荐直接硬编码，并且只打印DO的基础要素和核心要素
      3. 裁剪日志异常堆栈：无论是外部系统异常还是应用本身异常，都会有堆栈打出，超大流量下，频繁的输出完整堆栈，只会加剧系统当前负载。可以通过日志配置文件控制异常堆栈输出的深度
      4. 去组件框架：极致优化要求下，可以去掉一些组件框架，比如去掉传统的 MVC 框架，直接使用 Servlet 处理请求。这样可以绕过一大堆复杂且用处不大的处理逻辑，节省毫秒级的时间，当然，需要合理评估你对框架的依赖程度

2. 高可用

   1. 流量削峰
      1. 答题：答题目前已经使用的非常普遍了，本质是通过在入口层削减流量，从而让系统更好地支撑瞬时峰值。
      2. MQ： 最为常见的削峰方案是使用消息队列，通过把同步的直接调用转换成异步的间接推送缓冲瞬时流量。
      3. 过滤
   2. **Plan B**： 为了保证系统的高可用，必须设计一个 Plan B 方案来进行兜底。

#### [#](#接口设计要考虑哪些方面) 接口设计要考虑哪些方面？

讲讲几个要点：

- 接口版本化
- 命名规范
- 请求参数的规范性及处理的统一性
- 返回数据类型、返回码及信息提示的规范性
- 接口安全验证及权限的控制
- 请求接口日志的记录
- 良好的接口说明文档和测试程序

#### [#](#什么是接口幂等-如何保证接口的幂等性) 什么是接口幂等？如何保证接口的幂等性？

接口的幂等性实际上就是接口可重复调用，在调用方多次调用的情况下，接口最终得到的结果是一致的。有些接口可以天然的实现幂等性，比如查询接口，对于查询来说，你查询一次和两次，对于系统来说，没有任何影响，查出的结果也是一样。

除了查询功能具有天然的幂等性之外，增加、更新、删除都要保证幂等性。那么如何来保证幂等性呢？

- **全局唯一ID**

如果使用全局唯一ID，就是根据业务的操作和内容生成一个全局ID，在执行操作前先根据这个全局唯一ID是否存在，来判断这个操作是否已经执行。如果不存在则把全局ID，存储到存储系统中，比如数据库、redis等。如果存在则表示该方法已经执行。

从工程的角度来说，使用全局ID做幂等可以作为一个业务的基础的微服务存在，在很多的微服务中都会用到这样的服务，在每个微服务中都完成这样的功能，会存在工作量重复。另外打造一个高可靠的幂等服务还需要考虑很多问题，比如一台机器虽然把全局ID先写入了存储，但是在写入之后挂了，这就需要引入全局ID的超时机制。

使用全局唯一ID是一个通用方案，可以支持插入、更新、删除业务操作。但是这个方案看起来很美但是实现起来比较麻烦，下面的方案适用于特定的场景，但是实现起来比较简单。

- **去重表**

这种方法适用于在业务中有唯一标的插入场景中，比如在以上的支付场景中，如果一个订单只会支付一次，所以订单ID可以作为唯一标识。这时，我们就可以建一张去重表，并且把唯一标识作为唯一索引，在我们实现时，把创建支付单据和写入去去重表，放在一个事务中，如果重复创建，数据库会抛出唯一约束异常，操作就会回滚。

- **插入或更新**

这种方法插入并且有唯一索引的情况，比如我们要关联商品品类，其中商品的ID和品类的ID可以构成唯一索引，并且在数据表中也增加了唯一索引。这时就可以使用InsertOrUpdate操作。在mysql数据库中如下：

```sql
insert into goods_category (goods_id,category_id,create_time,update_time) 
       values(#{goodsId},#{categoryId},now(),now()) 
       on DUPLICATE KEY UPDATE
       update_time=now()
```

- **多版本控制**

这种方法适合在更新的场景中，比如我们要更新商品的名字，这时我们就可以在更新的接口中增加一个版本号，来做幂等

```java
boolean updateGoodsName(int id,String newName,int version);
```

在实现时可以如下

```sql
update goods set name=#{newName},version=#{version} where id=#{id} and version<${version}
```

- **状态机控制**

这种方法适合在有状态机流转的情况下，比如就会订单的创建和付款，订单的付款肯定是在之前，这时我们可以通过在设计状态字段时，使用int类型，并且通过值类型的大小来做幂等，比如订单的创建为0，付款成功为100。付款失败为99

在做状态机更新时，我们就这可以这样控制

```sql
update `order` set status=#{status} where id=#{id} and status<#{status}
```

## [#](#_14-微服务) 14 微服务

### [#](#_14-1-spring-cloud) 14.1 Spring Cloud

#### [#](#什么是微服务-谈谈你对微服务的理解) 什么是微服务？谈谈你对微服务的理解？

- **微服务**

以前所有的代码都放在同一个工程中、部署在同一个服务器、同一项目的不同模块不同功能互相抢占资源，微服务就是将工程根据不同的业务规则拆分成微服务，部署在不同的服务器上，服务之间相互调用，java中有的微服务有dubbo(只能用来做微服务)、springcloud( 提供了服务的发现、断路器等)。

- **微服务的特点**：

1. 按业务划分为一个独立运行的程序，即服务单元
2. 服务之间通过HTTP协议相互通信
3. 自动化部署
4. 可以用不同的编程语言
5. 可以用不同的存储技术
6. 服务集中化管理
7. 微服务是一个分布式系统

- **微服务的优势**

1. 将一个复杂的业务拆分为若干小的业务，将复杂的业务简单化，新人只需要了解他所接管的服务的代码，减少了新人的学习成本。
2. 由于微服务是分布式服务，服务于服务之间没有任何耦合。微服务系统的微服务单元具有很强的横向拓展能力。
3. 服务于服务之间采用HTTP网络通信协议来通信，单个服务内部高度耦合，服务与服务之间完全独立，无耦合。这使得微服务可以采用任何的开发语言和技术来实现，提高开发效率、降低开发成本。
4. 微服务是按照业务进行拆分的，并有坚实的服务边界，若要重写某一业务代码，不需了解所有业务，重写简单。
5. 微服务的每个服务单元是独立部署的，即独立运行在某个进程中，微服务的修改和部署对其他服务没有影响。
6. 微服务在CAP理论中采用的AP架构，具有高可用分区容错特点。高可用主要体现在系统7x24不间断服务，他要求系统有大量的服务器集群，从而提高系统的负载能力。分区容错也使得系统更加健壮。

- **微服务的不足**

1. 微服务的复杂度：构建一个微服务比较复杂，服务与服务之间通过HTTP协议或其他消息传递机制通信，开发者要选出最佳的通信机制，并解决网络服务差时带来的风险。 2.分布式事物：将事物分成多阶段提交，如果一阶段某一节点失败仍会导致数据不正确。如果事物涉及的节点很多，某一节点的网络出现异常会导致整个事务处于阻塞状态，大大降低数据库的性能。
2. 服务划分：将一个完整的系统拆分成很多个服务，是一件非常困难的事，因为这涉及了具体的业务场景
3. 服务部署：最佳部署容器Docker

- **微服务和SOA的关系**

微服务相对于和ESB联系在一起的SOA轻便敏捷的多，微服务将复杂的业务组件化，也是一种面向服务思想的体现。对于微服务来说，它是SOA的一种体现，但是它比ESB实现的SOA更加轻便、敏捷和简单。

#### [#](#什么是spring-cloud) 什么是Spring Cloud？

Spring Cloud是一系列框架的有序集合。它利用Spring Boot的开发便利性巧妙地简化了分布式系统基础设施的开发，如服务发现注册、配置中心、智能路由、消息总线、负载均衡、断路器、数据监控等，都可以用Spring Boot的开发风格做到一键启动和部署。

Spring Cloud并没有重复制造轮子，它只是将各家公司开发的比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，最终给开发者留出了一套简单易懂、易部署和易维护的分布式系统开发工具包。

- **SpringCloud的优点**

1. 耦合度比较低。不会影响其他模块的开发。
2. 减轻团队的成本，可以并行开发，不用关注其他人怎么开发，先关注自己的开发。
3. 配置比较简单，基本用注解就能实现，不用使用过多的配置文件。
4. 微服务跨平台的，可以用任何一种语言开发。
5. 每个微服务可以有自己的独立的数据库也有用公共的数据库。
6. 直接写后端的代码，不用关注前端怎么开发，直接写自己的后端代码即可，然后暴露接口，通过组件进行服务通信。

- **SpringCloud的缺点**

1. 部署比较麻烦，给运维工程师带来一定的麻烦。
2. 针对数据的管理比麻烦，因为微服务可以每个微服务使用一个数据库。
3. 系统集成测试比较麻烦
4. 性能的监控比较麻烦。

#### [#](#springcloud中的组件有那些) springcloud中的组件有那些？

说出主要的组件：

- Spring Cloud Eureka,服务注册中心,特性有失效剔除、服务保护
- Spring Cloud Zuul,API服务网关,功能有路由分发和过滤
- Spring Cloud Config,分布式配置中心，支持本地仓库、SVN、Git、Jar包内配置等模式
- Spring Cloud Ribbon,客户端负载均衡,特性有区域亲和,重试机制
- Spring Cloud Hystrix,客户端容错保护,特性有服务降级、服务熔断、请求缓存、请求合并、依赖隔离
- Spring Cloud Feign,声明式服务调用本质上就是Ribbon+Hystrix
- Spring Cloud Stream,消息驱动,有Sink、Source、Processor三种通道,特性有订阅发布、消费组、消息分区
- Spring Cloud Bus,消息总线,配合Config仓库修改的一种Stream实现，
- Spring Cloud Sleuth,分布式服务追踪,需要搞清楚TraceID和SpanID以及抽样,如何与ELK整合

#### [#](#具体说说springcloud主要项目) 具体说说SpringCloud主要项目?

Spring Cloud的子项目，大致可分成两类，一类是对现有成熟框架"Spring Boot化"的封装和抽象，也是数量最多的项目；第二类是开发了一部分分布式系统的基础设施的实现，如Spring Cloud Stream扮演的就是kafka, ActiveMQ这样的角色。

- **Spring Cloud Config** Config能够管理所有微服务的配置文件

集中配置管理工具，分布式系统中统一的外部配置管理，默认使用Git来存储配置，可以支持客户端配置的刷新及加密、解密操作。

- **Spring Cloud Netflix**

Netflix OSS 开源组件集成，包括Eureka、Hystrix、Ribbon、Feign、Zuul等核心组件。

1. Eureka：服务治理组件，包括服务端的注册中心和客户端的服务发现机制；
2. Ribbon：负载均衡的服务调用组件，具有多种负载均衡调用策略；
3. Hystrix：服务容错组件，实现了断路器模式，为依赖服务的出错和延迟提供了容错能力；
4. Feign：基于Ribbon和Hystrix的声明式服务调用组件；
5. Zuul：API网关组件，对请求提供路由及过滤功能。

- **Spring Cloud Bus**

1. 用于传播集群状态变化的消息总线，使用轻量级消息代理链接分布式系统中的节点，可以用来动态刷新集群中的服务配置信息。
2. 简单来说就是修改了配置文件，发送一次请求，所有客户端便会重新读取配置文件（需要利用中间插件MQ）。

- **Spring Cloud Consul**

Consul 是 HashiCorp 公司推出的开源工具，用于实现分布式系统的服务发现与配置。与其它分布式服务注册与发现的方案，Consul 的方案更“一站式”，内置了服务注册与发现框架、分布一致性协议实现、健康检查、Key/Value 存储、多数据中心方案，不再需要依赖其它工具（比如 ZooKeeper 等）。使用起来也较为简单。Consul 使用 Go 语言编写，因此具有天然可移植性(支持Linux、windows和Mac OS X)；安装包仅包含一个可执行文件，方便部署，与 Docker 等轻量级容器可无缝配合。

- **Spring Cloud Security**

Spring Cloud Security提供了一组原语，用于构建安全的应用程序和服务，而且操作简便。可以在外部（或集中）进行大量配置的声明性模型有助于实现大型协作的远程组件系统，通常具有中央身份管理服务。它也非常易于在Cloud Foundry等服务平台中使用。在Spring Boot和Spring Security OAuth2的基础上，可以快速创建实现常见模式的系统，如单点登录，令牌中继和令牌交换。

- **Spring Cloud Sleuth**

在微服务中，通常根据业务模块分服务，项目中前端发起一个请求，后端可能跨几个服务调用才能完成这个请求（如下图）。如果系统越来越庞大，服务之间的调用与被调用关系就会变得很复杂，假如一个请求中需要跨几个服务调用，其中一个服务由于网络延迟等原因挂掉了，那么这时候我们需要分析具体哪一个服务出问题了就会显得很困难。Spring Cloud Sleuth服务链路跟踪功能就可以帮助我们快速的发现错误根源以及监控分析每条请求链路上的性能等等。

- **Spring Cloud Stream**

轻量级事件驱动微服务框架，可以使用简单的声明式模型来发送及接收消息，主要实现为Apache Kafka及RabbitMQ。

- **Spring Cloud Task**

Spring Cloud Task的目标是为Spring Boot应用程序提供创建短运行期微服务的功能。在Spring Cloud Task中，我们可以灵活地动态运行任何任务，按需分配资源并在任务完成后检索结果。Tasks是Spring Cloud Data Flow中的一个基础项目，允许用户将几乎任何Spring Boot应用程序作为一个短期任务执行。

- **Spring Cloud Zookeeper**

1. SpringCloud支持三种注册方式Eureka， Consul(go语言编写)，zookeeper
2. Spring Cloud Zookeeper是基于Apache Zookeeper的服务治理组件。

- **Spring Cloud Gateway**

Spring cloud gateway是spring官方基于Spring 5.0、Spring Boot2.0和Project Reactor等技术开发的网关，Spring Cloud Gateway旨在为微服务架构提供简单、有效和统一的API路由管理方式，Spring Cloud Gateway作为Spring Cloud生态系统中的网关，目标是替代Netflix Zuul，其不仅提供统一的路由方式，并且还基于Filer链的方式提供了网关基本的功能，例如：安全、监控/埋点、限流等。

- **Spring Cloud OpenFeign**

Feign是一个声明性的Web服务客户端。它使编写Web服务客户端变得更容易。要使用Feign，我们可以将调用的服务方法定义成抽象方法保存在本地添加一点点注解就可以了，不需要自己构建Http请求了，直接调用接口就行了，不过要注意，调用方法要和本地抽象方法的签名完全一致。

#### [#](#spring-cloud项目部署架构) Spring Cloud项目部署架构？

![img](/images/spring/spring-cloud-1.jpeg)

#### [#](#spring-cloud-和dubbo区别) Spring Cloud 和dubbo区别?

- 服务调用方式：dubbo是RPC springcloud Rest Api
- 注册中心：dubbo 是zookeeper springcloud是eureka，也可以是zookeeper
- 服务网关，dubbo本身没有实现，只能通过其他第三方技术整合，springcloud有Zuul路由网关，作为路由服务器，进行消费者的请求分发,springcloud支持断路器，与git完美集成配置文件支持版本控制，事物总线实现配置文件的更新与服务自动装配等等一系列的微服务架构要素。

#### [#](#服务注册和发现是什么意思-spring-cloud-如何实现) 服务注册和发现是什么意思？Spring Cloud 如何实现？

当我们开始一个项目时，我们通常在属性文件中进行所有的配置。随着越来越多的服务开发和部署，添加和修改这些属性变得更加复杂。有些服务可能会下降，而某些位置可能会发生变化。手动更改属性可能会产生问题。 Eureka 服务注册和发现可以在这种情况下提供帮助。由于所有服务都在 Eureka 服务器上注册并通过调用 Eureka 服务器完成查找，因此无需处理服务地点的任何更改和处理。

#### [#](#什么是eureka) 什么是Eureka？

Eureka作为SpringCloud的服务注册功能服务器，他是服务注册中心，系统中的其他服务使用Eureka的客户端将其连接到Eureka Service中，并且保持心跳，这样工作人员可以通过Eureka Service来监控各个微服务是否运行正常。

#### [#](#eureka怎么实现高可用) Eureka怎么实现高可用？

集群吧，注册多台Eureka，然后把SpringCloud服务互相注册，客户端从Eureka获取信息时，按照Eureka的顺序来访问。

#### [#](#什么是eureka的自我保护模式) 什么是Eureka的自我保护模式？

默认情况下，如果Eureka Service在一定时间内没有接收到某个微服务的心跳，Eureka Service会进入自我保护模式，在该模式下Eureka Service会保护服务注册表中的信息，不再删除注册表中的数据，当网络故障恢复后，Eureka Servic 节点会自动退出自我保护模式

#### [#](#discoveryclient的作用) DiscoveryClient的作用？

可以从注册中心中根据服务别名获取注册的服务器信息。

#### [#](#eureka和zookeeper都可以提供服务注册与发现的功能-请说说两个的区别) Eureka和ZooKeeper都可以提供服务注册与发现的功能,请说说两个的区别？

1. ZooKeeper中的节点服务挂了就要选举，在选举期间注册服务瘫痪,虽然服务最终会恢复,但是选举期间不可用的，选举就是改微服务做了集群，必须有一台主其他的都是从
2. Eureka各个节点是平等关系,服务器挂了没关系，只要有一台Eureka就可以保证服务可用，数据都是最新的。如果查询到的数据并不是最新的，就是因为Eureka的自我保护模式导致的
3. Eureka本质上是一个工程,而ZooKeeper只是一个进程
4. Eureka可以很好的应对因网络故障导致部分节点失去联系的情况,而不会像ZooKeeper 一样使得整个注册系统瘫痪
5. ZooKeeper保证的是CP，Eureka保证的是AP

#### [#](#什么是网关) 什么是网关?

网关相当于一个网络服务架构的入口，所有网络请求必须通过网关转发到具体的服务。

#### [#](#网关的作用是什么) 网关的作用是什么？

统一管理微服务请求，权限控制、负载均衡、路由转发、监控、安全控制黑名单和白名单等

#### [#](#什么是spring-cloud-zuul-服务网关) 什么是Spring Cloud Zuul（服务网关）？

Zuul是对SpringCloud提供的成熟对的路由方案，他会根据请求的路径不同，网关会定位到指定的微服务，并代理请求到不同的微服务接口，他对外隐蔽了微服务的真正接口地址。

- 三个重要概念：动态路由表，路由定位，反向代理： 
  - 动态路由表：Zuul支持Eureka路由，手动配置路由，这俩种都支持自动更新
  - 路由定位：根据请求路径，Zuul有自己的一套定位服务规则以及路由表达式匹配
  - 反向代理：客户端请求到路由网关，网关受理之后，在对目标发送请求，拿到响应之后在 给客户端
- 它可以和Eureka,Ribbon,Hystrix等组件配合使用，
- Zuul的应用场景： 
  - 对外暴露，权限校验，服务聚合，日志审计等

#### [#](#网关与过滤器有什么区别) 网关与过滤器有什么区别？

网关是对所有服务的请求进行分析过滤，过滤器是对单个服务而言。

#### [#](#常用网关框架有那些) 常用网关框架有那些？

Nginx、Zuul、Gateway

#### [#](#zuul与nginx有什么区别) Zuul与Nginx有什么区别？

Zuul是java语言实现的，主要为java服务提供网关服务，尤其在微服务架构中可以更加灵活的对网关进行操作。Nginx是使用C语言实现，性能高于Zuul，但是实现自定义操作需要熟悉lua语言，对程序员要求较高，可以使用Nginx做Zuul集群。

#### [#](#既然nginx可以实现网关-为什么还需要使用zuul框架) 既然Nginx可以实现网关？为什么还需要使用Zuul框架?

Zuul是SpringCloud集成的网关，使用Java语言编写，可以对SpringCloud架构提供更灵活的服务。

#### [#](#zuulfilter常用有那些方法) ZuulFilter常用有那些方法?

- Run()：过滤器的具体业务逻辑
- shouldFilter()：判断过滤器是否有效
- filterOrder()：过滤器执行顺序
- filterType()：过滤器拦截位置

#### [#](#如何实现动态zuul网关路由转发) 如何实现动态Zuul网关路由转发?

通过path配置拦截请求，通过ServiceId到配置中心获取转发的服务列表，Zuul内部使用Ribbon实现本地负载均衡和转发。

#### [#](#zuul网关如何搭建集群) Zuul网关如何搭建集群?

使用Nginx的upstream设置Zuul服务集群，通过location拦截请求并转发到upstream，默认使用轮询机制对Zuul集群发送请求。

#### [#](#ribbon是什么) Ribbon是什么？

Ribbon是Netflix发布的开源项目，主要功能是提供客户端的软件负载均衡算法

Ribbon客户端组件提供一系列完善的配置项，如连接超时，重试等。简单的说，就是在配置文件中列出后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随机连接等）去连接这些机器。我们也很容易使用Ribbon实现自定义的负载均衡算法。（有点类似Nginx）

#### [#](#nginx与ribbon的区别) Nginx与Ribbon的区别？

Nginx是反向代理同时可以实现负载均衡，nginx拦截客户端请求采用负载均衡策略根据upstream配置进行转发，相当于请求通过nginx服务器进行转发。Ribbon是客户端负载均衡，从注册中心读取目标服务器信息，然后客户端采用轮询策略对服务直接访问，全程在客户端操作。

#### [#](#ribbon底层实现原理) Ribbon底层实现原理？

Ribbon使用discoveryClient从注册中心读取目标服务信息，对同一接口请求进行计数，使用%取余算法获取目标服务集群索引，返回获取到的目标服务信息。

#### [#](#loadbalanced注解的作用) @LoadBalanced注解的作用？

 开启客户端负载均衡。

#### [#](#什么是断路器) 什么是断路器

当一个服务调用另一个服务由于网络原因或自身原因出现问题，调用者就会等待被调用者的响应 当更多的服务请求到这些资源导致更多的请求等待，发生连锁效应（雪崩效应）

断路器有三种状态

- 打开状态：一段时间内 达到一定的次数无法调用 并且多次监测没有恢复的迹象 断路器完全打开 那么下次请求就不会请求到该服务
- 半开状态：短时间内 有恢复迹象 断路器会将部分请求发给该服务，正常调用时 断路器关闭
- 关闭状态：当服务一直处于正常状态 能正常调用

#### [#](#什么是-hystrix) 什么是 Hystrix？

在分布式系统，我们一定会依赖各种服务，那么这些个服务一定会出现失败的情况，就会导致雪崩，Hystrix就是这样的一个工具，防雪崩利器，它具有服务降级，服务熔断，服务隔离，监控等一些防止雪崩的技术。

Hystrix有四种防雪崩方式:

- 服务降级：接口调用失败就调用本地的方法返回一个空
- 服务熔断：接口调用失败就会进入调用接口提前定义好的一个熔断的方法，返回错误信息
- 服务隔离：隔离服务之间相互影响
- 服务监控：在服务发生调用时,会将每秒请求数、成功请求数等运行指标记录下来。

#### [#](#什么是feign) 什么是Feign？

Feign 是一个声明web服务客户端，这使得编写web服务客户端更容易

他将我们需要调用的服务方法定义成抽象方法保存在本地就可以了，不需要自己构建Http请求了，直接调用接口就行了，不过要注意，调用方法要和本地抽象方法的签名完全一致。

#### [#](#springcloud有几种调用接口方式) SpringCloud有几种调用接口方式？

- Feign
- RestTemplate

#### [#](#ribbon和feign调用服务的区别) Ribbon和Feign调用服务的区别？

调用方式同：Ribbon需要我们自己构建Http请求，模拟Http请求然后通过RestTemplate发给其他服务，步骤相当繁琐

而Feign则是在Ribbon的基础上进行了一次改进，采用接口的形式，将我们需要调用的服务方法定义成抽象方法保存在本地就可以了，不需要自己构建Http请求了，直接调用接口就行了，不过要注意，调用方法要和本地抽象方法的签名完全一致。

#### [#](#什么是-spring-cloud-bus) 什么是 Spring Cloud Bus？

- Spring Cloud Bus就像一个分布式执行器，用于扩展的Spring Boot应用程序的配置文件，但也可以用作应用程序之间的通信通道。
- Spring Cloud Bus 不能单独完成通信，需要配合MQ支持
- Spring Cloud Bus一般是配合Spring Cloud Config做配置中心的
- Springcloud config实时刷新也必须采用SpringCloud Bus消息总线

#### [#](#什么是spring-cloud-config) 什么是Spring Cloud Config?

Spring Cloud Config为分布式系统中的外部配置提供服务器和客户端支持，可以方便的对微服务各个环境下的配置进行集中式管理。Spring Cloud Config分为Config Server和Config Client两部分。Config Server负责读取配置文件，并且暴露Http API接口，Config Client通过调用Config Server的接口来读取配置文件。

#### [#](#分布式配置中心有那些框架) 分布式配置中心有那些框架？

Apollo、zookeeper、springcloud config。

#### [#](#分布式配置中心的作用) 分布式配置中心的作用？

动态变更项目配置信息而不必重新部署项目。

#### [#](#springcloud-config-可以实现实时刷新吗) SpringCloud Config 可以实现实时刷新吗？

springcloud config实时刷新采用SpringCloud Bus消息总线。

#### [#](#什么是spring-cloud-gateway) 什么是Spring Cloud Gateway?

Spring Cloud Gateway是Spring Cloud官方推出的第二代网关框架，取代Zuul网关。网关作为流量的，在微服务系统中有着非常作用，网关常见的功能有路由转发、权限校验、限流控制等作用。

使用了一个RouteLocatorBuilder的bean去创建路由，除了创建路由RouteLocatorBuilder可以让你添加各种predicates和filters，predicates断言的意思，顾名思义就是根据具体的请求的规则，由具体的route去处理，filters是各种过滤器，用来对请求做各种判断和修改。

### [#](#_14-2-kubernetes) 14.2 Kubernetes

#### [#](#什么是kubernetes-kubernetes与docker有什么关系) 什么是Kubernetes? Kubernetes与Docker有什么关系？

- **是什么**？

Kubernetes是一个开源容器管理工具，负责容器部署，容器扩缩容以及负载平衡。作为Google的创意之作，它提供了出色的社区，并与所有云提供商合作。因此，我们可以说Kubernetes不是一个容器化平台，而是一个多容器管理解决方案。

众所周知，Docker提供容器的生命周期管理，Docker镜像构建运行时容器。但是，由于这些单独的容器必须通信，因此使用Kubernetes。因此，我们说Docker构建容器，这些容器通过Kubernetes相互通信。因此，可以使用Kubernetes手动关联和编排在多个主机上运行的容器。

- **有哪些特性**？

1. **自我修复**: 在节点故障时可以删除失效容器，重新创建新的容器，替换和重新部署，保证预期的副本数量，kill掉健康检查失败的容器，并且在容器未准备好之前不会处理客户端情况，确保线上服务不会中断
2. **弹性伸缩**: 使用命令、UI或者k8s基于cpu使用情况自动快速扩容和缩容应用程序实例，保证应用业务高峰并发时的高可用性，业务低峰时回收资源，以最小成本运行服务
3. **自动部署和回滚**: k8s采用滚动更新策略更新应用，一次更新一个pod，而不是同时删除所有pod，如果更新过程中出现问题，将回滚恢复，确保升级不影响业务
4. **服务发现和负载均衡**: k8s为多个容器提供一个统一访问入口(内部IP地址和一个dns名称)并且负载均衡关联的所有容器，使得用户无需考虑容器IP问题
5. **机密和配置管理**: 管理机密数据和应用程序配置，而不需要把敏感数据暴露在径向力，提高敏感数据安全性，并可以将一些常用的配置存储在k8s中，方便应用程序调用
6. **存储编排**: 挂载外部存储系统，无论时来自本地存储、公有云(aws)、还是网络存储（nfs、GFS、ceph），都作为集群资源的一部分使用，极大提高存储使用灵活性
7. **批处理**: 提供一次性任务，定时任务：满足批量数据处理和分析的场景

#### [#](#kubernetes的整体架构) Kubernetes的整体架构？

![img](/images/k8s/k8s-arch-1.png)

Kubernetes主要由以下几个核心组件组成：

1. **etcd**：提供数据库服务保存了整个集群的状态
2. **kube-apiserver**：提供了资源操作的唯一入口，并提供认证、授权、访问控制、API注册和发现等机制
3. **kube-controller-manager**：负责维护集群的状态，比如故障检测、自动扩展、滚动更新等
4. **cloud-controller-manager**：是与底层云计算服务商交互的控制器
5. **kub-scheduler**：负责资源的调度，按照预定的调度策略将Pod调度到相应的机器上
6. **kubelet**：负责维护容器的生命周期，同时也负责Volume（CVI）和网络（CNI）的管理；
7. **kube-proxy**：负责为Service提供内部的服务发现和负载均衡，并维护网络规则
8. **container-runtime**：是负责管理运行容器的软件，比如docker

除了核心组件，还有一些推荐的Add-ons：

1. kube-dns负责为整个集群提供DNS服务
2. Ingress Controller为服务提供外网入口
3. Heapster提供资源监控
4. Dashboard提供GUI
5. Federation提供跨可用区的集群
6. Fluentd-elasticsearch提供集群日志采集、存储与查询

#### [#](#kubernetes中有哪些核心概念) Kubernetes中有哪些核心概念？

- **Cluster、Master、Node**

1. Cluster
   1. Cluster（集群） 是计算、存储和网络资源的集合，Kubernetes 利用这些资源运行各种基于容器的应用。最简单的 Cluster 可以只有一台主机（它既是 Mater 也是 Node）
2. Master
   1. Master 是 Cluster 的大脑，它的主要职责是调度，即决定将应用放在哪里运行。
   2. Master 运行 Linux 操作系统，可以是物理机或者虚拟机。
   3. 为了实现高可用，可以运行多个 Master。
3. Node
   1. Node 的职责是运行容器应用。
   2. Node 由 Master 管理，Node 负责监控并汇报容器的状态，并根据 Master 的要求管理容器的生命周期。
   3. Node 运行在 Linux 操作系统，可以是物理机或者是虚拟机。

- **Pod**

1. 基本概念
   1. Pod 是 Kubernetes 的最小工作单元。
   2. 每个 Pod 包含一个或多个容器。Pod 中的容器会作为一个整体被 Master 调度到一个 Node 上运行。
2. 引入 Pod 的目的
   1. `可管理性`: 有些容器天生就是需要紧密联系，一起工作。Pod 提供了比容器更高层次的抽象，将它们封装到一个部署单元中。Kubernetes 以 Pod 为最小单位进行调度、扩展、共享资源、管理生命周期。
   2. `通信和资源共享`: Pod 中的所有容器使用同一个网络 namespace，即相同的 IP 地址和 Port 空间。它们可以直接用 localhost 通信。同样的，这些容器可以共享存储，当 Kubernetes 挂载 volume 到 Pod，本质上是将 volume 挂载到 Pod 中的每一个容器。
3. Pod 的使用方式
   1. `运行单一容器`: one-container-per-Pod 是 Kubernetes 最常见的模型，这种情况下，只是将单个容器简单封装成 Pod。即便是只有一个容器，Kubernetes 管理的也是 Pod 而不是直接管理容器。
   2. `运行多个容器`: 对于那些联系非常紧密，而且需要直接共享资源的容器，应该放在一个 Pod 中。比如下面这个 Pod 包含两个容器：一个 File Puller，一个是 Web Server。File Puller 会定期从外部的 Content Manager 中拉取最新的文件，将其存放在共享的 volume 中。Web Server 从 volume 读取文件，响应 Consumer 的请求。这两个容器是紧密协作的，它们一起为 Consumer 提供最新的数据；同时它们也通过 volume 共享数据。所以放到一个 Pod 是合适的。

- **Controller**

1. **基本概念**
   1. Kubernetes 通常不会直接创建 Pod，而是通过 Controller 来管理 Pod 的。Controller 中定义了 Pod 的部署特性，比如有几个副本，在什么样的 Node 上运行等。为了满足不同的业务场景，Kubernetes 提供了多种 Controller，包括 Deployment、ReplicaSet、DaemonSet、StatefuleSet、Job 等。
2. **各个 Controller**
   1. `Deployment`： Deployment 是最常用的 Controller，比如我们可以通过创建 Deployment 来部署应用的。Deployment 可以管理 Pod 的多个副本，并确保 Pod 按照期望的状态运行。
   2. `ReplicaSet`： ReplicaSet 实现了 Pod 的多副本管理。使用 Deployment 时会自动创建 ReplicaSet，也就是说 Deployment 是通过 ReplicaSet 来管理 Pod 的多个副本，我们通常不需要直接使用 ReplicaSet。
   3. `DaemonSet`： DaemonSet 用于每个 Node 最多只运行一个 Pod 副本的场景。正如其名称所揭示的，DaemonSet 通常用于运行 daemon。
   4. `StatefuleSet`： StatefuleSet 能够保证 Pod 的每个副本在整个生命周期中名称是不变的。而其他 Controller 不提供这个功能，当某个 Pod 发生故障需要删除并重新启动时，Pod 的名称会发生变化。同时 StatefuleSet 会保证副本按照固定的顺序启动、更新或者删除。
   5. `Job`： Job 用于运行结束就删除的应用。而其他 Controller 中的 Pod 通常是长期持续运行。

- **Service、Namespace**

1. **Service**
   1. Deployment 可以部署多个副本，每个 Pod 都有自己的 IP。而 Pod 很可能会被频繁地销毁和重启，它们的 IP 会发生变化，用 IP 来访问 Deployment 副本不太现实。
   2. Service 定义了外界访问一组特定 Pod 的方式。Service 有自己的 IP 和端口，Service 为 Pod 提供了负载均衡。
2. **Namespace**
   1. Namespace 可以将一个物理的 Cluster 逻辑上划分成多个虚拟 Cluster，每个 Cluster 就是一个 Namespace。不同 Namespace 里的资源是完全隔离的。
   2. Kubernetes 默认创建了两个 Namespace： 
      1. default：创建资源时如果不指定，将被放到这个 Namespace 中。
      2. kube-system：Kubernetes 自己创建的系统资源将放到这个 Namespace 中。

#### [#](#什么是heapster) 什么是Heapster？

Heapster是由每个节点上运行的Kubelet提供的集群范围的数据聚合器。此容器管理工具在Kubernetes集群上本机支持，并作为pod运行，就像集群中的任何其他pod一样。因此，它基本上发现集群中的所有节点，并通过机上Kubernetes代理查询集群中Kubernetes节点的使用信息。

#### [#](#什么是minikube) 什么是Minikube？

Minikube是一种工具，可以在本地轻松运行Kubernetes。这将在虚拟机中运行单节点Kubernetes群集。

#### [#](#什么是kubectl) 什么是Kubectl？

Kubectl是一个平台，您可以使用该平台将命令传递给集群。因此，它基本上为CLI提供了针对Kubernetes集群运行命令的方法，以及创建和管理Kubernetes组件的各种方法。

#### [#](#kube-apiserver和kube-scheduler的作用是什么) kube-apiserver和kube-scheduler的作用是什么？

kube -apiserver遵循横向扩展架构，是主节点控制面板的前端。这将公开Kubernetes主节点组件的所有API，并负责在Kubernetes节点和Kubernetes主组件之间建立通信。

kube-scheduler负责工作节点上工作负载的分配和管理。因此，它根据资源需求选择最合适的节点来运行未调度的pod，并跟踪资源利用率。它确保不在已满的节点上调度工作负载。

#### [#](#请你说一下kubenetes针对pod资源对象的健康监测机制) 请你说一下kubenetes针对pod资源对象的健康监测机制？

K8s中对于pod资源对象的健康状态检测，提供了三类probe（探针）来执行对pod的健康监测：

- livenessProbe探针

可以根据用户自定义规则来判定pod是否健康，如果livenessProbe探针探测到容器不健康，则kubelet会根据其重启策略来决定是否重启，如果一个容器不包含livenessProbe探针，则kubelet会认为容器的livenessProbe探针的返回值永远成功。

- ReadinessProbe探针 同样是可以根据用户自定义规则来判断pod是否健康，如果探测失败，控制器会将此pod从对应service的endpoint列表中移除，从此不再将任何请求调度到此Pod上，直到下次探测成功。
- startupProbe探针 启动检查机制，应用一些启动缓慢的业务，避免业务长时间启动而被上面两类探针kill掉，这个问题也可以换另一种方式解决，就是定义上面两类探针机制时，初始化时间定义的长一些即可。

#### [#](#k8s中镜像的下载策略是什么) K8s中镜像的下载策略是什么？

可通过命令`kubectl explain pod.spec.containers`来查看imagePullPolicy这行的解释。

K8s的镜像下载策略有三种：Always、Never、IFNotPresent；

- Always：镜像标签为latest时，总是从指定的仓库中获取镜像；
- Never：禁止从仓库中下载镜像，也就是说只能使用本地镜像；
- IfNotPresent：仅当本地没有对应镜像时，才从目标仓库中下载。

默认的镜像下载策略是：当镜像标签是latest时，默认策略是Always；当镜像标签是自定义时（也就是标签不是latest），那么默认策略是IfNotPresent。

#### [#](#image的状态有哪些) image的状态有哪些？

- Running：Pod所需的容器已经被成功调度到某个节点，且已经成功运行，
- Pending：APIserver创建了pod资源对象，并且已经存入etcd中，但它尚未被调度完成或者仍然处于仓库中下载镜像的过程
- Unknown：APIserver无法正常获取到pod对象的状态，通常是其无法与所在工作节点的kubelet通信所致。

#### [#](#如何控制滚动更新过程) 如何控制滚动更新过程？

可以通过下面的命令查看到更新时可以控制的参数：

```bash
[root@master yaml]# kubectl explain deploy.spec.strategy.rollingUpdate
```

- **maxSurge** ：此参数控制滚动更新过程，副本总数超过预期pod数量的上限。可以是百分比，也可以是具体的值。默认为1。 （上述参数的作用就是在更新过程中，值若为3，那么不管三七二一，先运行三个pod，用于替换旧的pod，以此类推）
- **maxUnavailable**：此参数控制滚动更新过程中，不可用的Pod的数量。 （这个值和上面的值没有任何关系，举个例子：我有十个pod，但是在更新的过程中，我允许这十个pod中最多有三个不可用，那么就将这个参数的值设置为3，在更新的过程中，只要不可用的pod数量小于或等于3，那么更新过程就不会停止）。

#### [#](#daemonset资源对象的特性) DaemonSet资源对象的特性？

DaemonSet这种资源对象会在每个k8s集群中的节点上运行，并且每个节点只能运行一个pod，这是它和deployment资源对象的最大也是唯一的区别。所以，在其yaml文件中，不支持定义replicas，除此之外，与Deployment、RS等资源对象的写法相同。

它的一般使用场景如下：

1. 在去做每个节点的日志收集工作；
2. 监控每个节点的的运行状态；

#### [#](#说说你对job这种资源对象的了解) 说说你对Job这种资源对象的了解？

Job与其他服务类容器不同，Job是一种工作类容器（一般用于做一次性任务）。使用常见不多，可以忽略这个问题。

```yaml
#提高Job执行效率的方法：
spec:
  parallelism: 2           #一次运行2个
  completions: 8           #最多运行8个
  template:
metadata:
```

#### [#](#pod的重启策略是什么) pod的重启策略是什么？

可以通过命令`kubectl explain pod.spec`查看pod的重启策略。（restartPolicy字段）

- Always：但凡pod对象终止就重启，此为默认策略。
- OnFailure：仅在pod对象出现错误时才重启

#### [#](#描述一下pod的生命周期有哪些状态) 描述一下pod的生命周期有哪些状态？

- Pending：表示pod已经被同意创建，正在等待kube-scheduler选择合适的节点创建，一般是在准备镜像；
- Running：表示pod中所有的容器已经被创建，并且至少有一个容器正在运行或者是正在启动或者是正在重启；
- Succeeded：表示所有容器已经成功终止，并且不会再启动；
- Failed：表示pod中所有容器都是非0（不正常）状态退出；
- Unknown：表示无法读取Pod状态，通常是kube-controller-manager无法与Pod通信。

#### [#](#创建一个pod的流程是什么) 创建一个pod的流程是什么？

1） 客户端提交Pod的配置信息（可以是yaml文件定义好的信息）到kube-apiserver； 2） Apiserver收到指令后，通知给controller-manager创建一个资源对象； 3） Controller-manager通过api-server将pod的配置信息存储到ETCD数据中心中； 4） Kube-scheduler检测到pod信息会开始调度预选，会先过滤掉不符合Pod资源配置要求的节点，然后开始调度调优，主要是挑选出更适合运行pod的节点，然后将pod的资源配置单发送到node节点上的kubelet组件上。 5） Kubelet根据scheduler发来的资源配置单运行pod，运行成功后，将pod的运行信息返回给scheduler，scheduler将返回的pod运行状况的信息存储到etcd数据中心。

#### [#](#删除一个pod会发生什么事情) 删除一个Pod会发生什么事情？

Kube-apiserver会接受到用户的删除指令，默认有30秒时间等待优雅退出，超过30秒会被标记为死亡状态，此时Pod的状态Terminating，kubelet看到pod标记为Terminating就开始了关闭Pod的工作；

关闭流程如下：

1. pod从service的endpoint列表中被移除；
2. 如果该pod定义了一个停止前的钩子，其会在pod内部被调用，停止钩子一般定义了如何优雅的结束进程；
3. 进程被发送TERM信号（kill -14）
4. 当超过优雅退出的时间后，Pod中的所有进程都会被发送SIGKILL信号（kill -9）。

#### [#](#k8s的service是什么) K8s的Service是什么？

Pod每次重启或者重新部署，其IP地址都会产生变化，这使得pod间通信和pod与外部通信变得困难，这时候，就需要Service为pod提供一个固定的入口。

Service的Endpoint列表通常绑定了一组相同配置的pod，通过负载均衡的方式把外界请求分配到多个pod上

#### [#](#k8s是怎么进行服务注册的) k8s是怎么进行服务注册的？

Pod启动后会加载当前环境所有Service信息，以便不同Pod根据Service名进行通信。

#### [#](#k8s集群外流量怎么访问pod) k8s集群外流量怎么访问Pod？

可以通过Service的NodePort方式访问，会在所有节点监听同一个端口，比如：30000，访问节点的流量会被重定向到对应的Service上面。

#### [#](#k8s数据持久化的方式有哪些) k8s数据持久化的方式有哪些？

- **EmptyDir(空目录)**：

没有指定要挂载宿主机上的某个目录，直接由Pod内保部映射到宿主机上。类似于docker中的manager volume。

主要使用场景：

1. 只需要临时将数据保存在磁盘上，比如在合并/排序算法中；
2. 作为两个容器的共享存储，使得第一个内容管理的容器可以将生成的数据存入其中，同时由同一个webserver容器对外提供这些页面。

emptyDir的特性： 同个pod里面的不同容器，共享同一个持久化目录，当pod节点删除时，volume的数据也会被删除。如果仅仅是容器被销毁，pod还在，则不会影响volume中的数据。 总结来说：emptyDir的数据持久化的生命周期和使用的pod一致。一般是作为临时存储使用。

- **Hostpath**：

将宿主机上已存在的目录或文件挂载到容器内部。类似于docker中的bind mount挂载方式。

这种数据持久化方式，运用场景不多，因为它增加了pod与节点之间的耦合。

一般对于k8s集群本身的数据持久化和docker本身的数据持久化会使用这种方式，可以自行参考apiService的yaml文件，位于：/etc/kubernetes/main…目录下。

- **PersistentVolume**（简称PV）：

基于NFS服务的PV，也可以基于GFS的PV。它的作用是统一数据持久化目录，方便管理。

在一个PV的yaml文件中，可以对其配置PV的大小，

指定PV的访问模式：

1. ReadWriteOnce：只能以读写的方式挂载到单个节点；
2. ReadOnlyMany：能以只读的方式挂载到多个节点；
3. ReadWriteMany：能以读写的方式挂载到多个节点。，

以及指定pv的回收策略(这里的回收策略指的是在PV被删除后，在这个PV下所存储的源文件是否删除)：

1. recycle：清除PV的数据，然后自动回收；
2. Retain：需要手动回收；
3. delete：删除云存储资源，云存储专用；

若需使用PV，那么还有一个重要的概念：PVC，PVC是向PV申请应用所需的容量大小，K8s集群中可能会有多个PV，PVC和PV若要关联，其定义的访问模式必须一致。定义的storageClassName也必须一致，若群集中存在相同的（名字、访问模式都一致）两个PV，那么PVC会选择向它所需容量接近的PV去申请，或者随机申请。

#### [#](#replica-set-和-replication-controller-之间有什么区别) Replica Set 和 Replication Controller 之间有什么区别？

Replica Set 和 Replication Controller 几乎完全相同。它们都确保在任何给定时间运行指定数量的 Pod 副本。不同之处在于复制 Pod 使用的选择器。Replica Set 使用基于集合的选择器，而 Replication Controller 使用基于权限的选择器。

Equity-Based 选择器：这种类型的选择器允许按标签键和值进行过滤。因此，在外行术语中，基于 Equity 的选择器将仅查找与标签具有完全相同短语的 Pod。示例：假设您的标签键表示 app = nginx，那么使用此选择器，您只能查找标签应用程序等于 nginx 的那些 Pod。

Selector-Based 选择器：此类型的选择器允许根据一组值过滤键。因此，换句话说，基于 Selector 的选择器将查找已在集合中提及其标签的 Pod。示例：假设您的标签键在（nginx、NPS、Apache）中显示应用程序。然后，使用此选择器，如果您的应用程序等于任何 nginx、NPS或 Apache，则选择器将其视为真实结果。

#### [#](#其它) 其它

基础篇 基础篇主要面向的初级、中级开发工程师职位，主要考察对k8s本身的理解。

kubernetes包含几个组件。各个组件的功能是什么。组件之间是如何交互的。 k8s的pause容器有什么用。是否可以去掉。 k8s中的pod内几个容器之间的关系是什么。 一个经典pod的完整生命周期。 k8s的service和ep是如何关联和相互影响的。 详述kube-proxy原理，一个请求是如何经过层层转发落到某个pod上的整个过程。请求可能来自pod也可能来自外部。 rc/rs功能是怎么实现的。详述从API接收到一个创建rc/rs的请求，到最终在节点上创建pod的全过程，尽可能详细。另外，当一个pod失效时，kubernetes是如何发现并重启另一个pod的？ deployment/rs有什么区别。其使用方式、使用条件和原理是什么。 cgroup中的cpu有哪几种限制方式。k8s是如何使用实现request和limit的。

拓展实践篇 拓展实践篇主要面向的高级开发工程师、架构师职位，主要考察实践经验和技术视野。

设想一个一千台物理机，上万规模的容器的kubernetes集群，请详述使用kubernetes时需要注意哪些问题？应该怎样解决？（提示可以从高可用，高性能等方向，覆盖到从镜像中心到kubernetes各个组件等） 设想kubernetes集群管理从一千台节点到五千台节点，可能会遇到什么样的瓶颈。应该如何解决。 kubernetes的运营中有哪些注意的要点。 集群发生雪崩的条件，以及预防手段。 设计一种可以替代kube-proxy的实现 sidecar的设计模式如何在k8s中进行应用。有什么意义。 灰度发布是什么。如何使用k8s现有的资源实现灰度发布。 介绍k8s实践中踩过的比较大的一个坑和解决方式。

### [#](#_14-3-service-mesh) 14.3 Service Mesh

#### [#](#什么是service-mesh-服务网格) 什么是Service Mesh（服务网格）？

Service Mesh是专用的基础设施层，轻量级高性能网络代理。提供安全的、快速的、可靠地服务间通讯，与实际应用部署一起，但对应用透明。

为了帮助理解， 下图展示了服务网格的典型边车部署方式：

https://www.jianshu.com/p/cc5b54ad8d5f

#### [#](#什么是istio) 什么是Istio?

#### [#](#istio的架构) Istio的架构？

## [#](#_15-devops) 15 DevOps

### [#](#_15-1-linux) 15.1 Linux

#### [#](#什么是linux) 什么是Linux？

Linux是一种基于UNIX的操作系统，最初是由Linus Torvalds引入的。它基于Linux内核，可以运行在由Intel，MIPS，HP，IBM，SPARC和Motorola制造的不同硬件平台上。Linux中另一个受欢迎的元素是它的吉祥物，一个名叫Tux的企鹅形象。

#### [#](#unix和linux有什么区别) UNIX和LINUX有什么区别？

Unix最初是作为Bell Laboratories的专有操作系统开始的，后来产生了不同的商业版本。另一方面，Linux是免费的，开源的，旨在为大众提供非适当的操作系统。

#### [#](#什么是bash) 什么是BASH？

BASH是Bourne Again SHell的缩写。它由Steve Bourne编写，作为原始Bourne Shell（由/ bin / sh表示）的替代品。它结合了原始版本的Bourne Shell的所有功能，以及其他功能，使其更容易使用。从那以后，它已被改编为运行Linux的大多数系统的默认shell。

#### [#](#什么是linux内核) 什么是Linux内核？

Linux内核是一种低级系统软件，其主要作用是为用户管理硬件资源。它还用于为用户级交互提供界面。

#### [#](#什么是lilo) 什么是LILO？

LILO是Linux的引导加载程序。它主要用于将Linux操作系统加载到主内存中，以便它可以开始运行。

#### [#](#什么是交换空间) 什么是交换空间？

交换空间是Linux使用的一定空间，用于临时保存一些并发运行的程序。当RAM没有足够的内存来容纳正在执行的所有程序时，就会发生这种情况。

#### [#](#linux的基本组件是什么) Linux的基本组件是什么？

就像任何其他典型的操作系统一样，Linux拥有所有这些组件：内核，shell和GUI，系统实用程序和应用程序。Linux比其他操作系统更具优势的是每个方面都附带其他功能，所有代码都可以免费下载。

#### [#](#linux系统安装多个桌面环境有帮助吗) Linux系统安装多个桌面环境有帮助吗？

通常，一个桌面环境，如KDE或Gnome，足以在没有问题的情况下运行。尽管系统允许从一个环境切换到另一个环境，但这对用户来说都是优先考虑的问题。有些程序在一个环境中工作而在另一个环境中无法工作，因此它也可以被视为选择使用哪个环境的一个因素。

#### [#](#bash和dos之间的基本区别是什么) BASH和DOS之间的基本区别是什么？

BASH和DOS控制台之间的主要区别在于3个方面：

BASH命令区分大小写，而DOS命令则不区分; 在BASH下，/ character是目录分隔符，\作为转义字符。在DOS下，/用作命令参数分隔符，\是目录分隔符 DOS遵循命名文件中的约定，即8个字符的文件名后跟一个点，扩展名为3个字符。BASH没有遵循这样的惯例。

#### [#](#gnu项目的重要性是什么) GNU项目的重要性是什么？

这种所谓的自由软件运动具有多种优势，例如可以自由地运行程序以及根据你的需要自由学习和修改程序。它还允许你将软件副本重新分发给其他人，以及自由改进软件并将其发布给公众。

#### [#](#描述root帐户) 描述root帐户？

root帐户就像一个系统管理员帐户，允许你完全控制系统。你可以在此处创建和维护用户帐户，为每个帐户分配不同的权限。每次安装Linux时都是默认帐户。

#### [#](#如何在发出命令时打开命令提示符) 如何在发出命令时打开命令提示符？

要打开默认shell（可以找到命令提示符的位置），请按Ctrl-Alt-F1。这将提供命令行界面（CLI），你可以根据需要从中运行命令。

#### [#](#如何知道linux使用了多少内存) 如何知道Linux使用了多少内存？

在命令shell中，使用“concatenate”命令：cat / proc / meminfo获取内存使用信息。你应该看到一行开始像Mem：64655360等。这是Linux认为它可以使用的总内存。

你也可以使用命令

```bash
free - m
vmstat
top
htop
```

找到当前的内存使用情况

#### [#](#linux系统下交换分区的典型大小是多少) Linux系统下交换分区的典型大小是多少？

交换分区的首选大小是系统上可用物理内存量的两倍。如果无法做到这一点，则最小大小应与安装的内存量相同。

#### [#](#什么是符号链接) 什么是符号链接？

符号链接的行为类似于Windows中的快捷方式。这些链接指向程序，文件或目录。它还允许你即时访问它，而无需直接转到整个路径名。

#### [#](#ctrl-alt-del组合键是否适用于linux) Ctrl + Alt + Del组合键是否适用于Linux？

是的，它确实。就像Windows一样，你可以使用此组合键来执行系统重启。一个区别是你不会收到任何确认消息，因此，立即重启。

#### [#](#如何引用连接打印机等设备的并行端口) 如何引用连接打印机等设备的并行端口？

在Windows下，你将并行端口称为LPT端口，而在Linux下，你将其称为/ dev / lp。因此，LPT1，LPT2和LPT3在Linux下称为/ dev / lp0，/ dev / lp1或/ dev / lp2。

#### [#](#硬盘驱动器和软盘驱动器等驱动器是否用驱动器号表示) 硬盘驱动器和软盘驱动器等驱动器是否用驱动器号表示？

在Linux中，每个驱动器和设备都有不同的名称。例如，软盘驱动器称为/ dev / fd0和/ dev / fd1。IDE / EIDE硬盘驱动器称为/ dev / hda，/ dev / hdb，/ dev / hdc等。

#### [#](#如何在linux下更改权限) 如何在Linux下更改权限？

假设你是系统管理员或文件或目录的所有者，则可以使用chmod命令授予权限。使用+符号添加权限或 - 符号拒绝权限，以及以下任何字母：u（用户），g（组），o（其他），a（所有），r（读取），w（写入）和x（执行）。例如，命令chmod go + rw FILE1.TXT授予对文件FILE1.TXT的读写访问权限，该文件分配给组和其他组。

#### [#](#在linux中-为不同的串口分配了哪些名称) 在Linux中，为不同的串口分配了哪些名称？

串行端口标识为/ dev / ttyS0到/ dev / ttyS7。这些是Windows中COM1到COM8的等效名称。

#### [#](#如何在linux下访问分区) 如何在Linux下访问分区？

Linux在驱动器标识符的末尾分配数字。例如，如果第一个IDE硬盘驱动器有三个主分区，则它们将命名/编号，/ dev / hda1，/ dev / hda2和/ dev / hda3。

#### [#](#什么是硬链接) 什么是硬链接？

硬链接直接指向磁盘上的物理文件，而不指向路径名。这意味着如果重命名或移动原始文件，链接将不会中断，因为链接是针对文件本身的，而不是文件所在的路径。

#### [#](#linux下文件名的最大长度是多少) Linux下文件名的最大长度是多少？

任何文件名最多可包含255个字符。此限制不包括路径名，因此整个路径名和文件名可能会超过255个字符。

#### [#](#什么是以点开头的文件名) 什么是以点开头的文件名？

通常，以点开头的文件名是隐藏文件。这些文件可以是包含重要数据或设置信息的配置文件。将这些文件设置为隐藏会使其不太可能被意外删除。

#### [#](#解释虚拟桌面) 解释虚拟桌面?

这可以作为最小化和最大化当前桌面上不同窗口的替代方案。当你可以打开一个或多个程序时，使用虚拟桌面可以清除桌面。你可以简单地在虚拟桌面之间进行随机播放，而不是在每个程序中保持完整的程序，而不是最小化/恢复所有这些程序。

#### [#](#如何在linux下跨不同的虚拟桌面共享程序) 如何在Linux下跨不同的虚拟桌面共享程序？

要在不同的虚拟桌面之间共享程序，请在程序窗口的左上角查找看起来像图钉的图标。按此按钮将“固定”该应用程序到位，使其显示在所有虚拟桌面上，位于屏幕上的相同位置。

#### [#](#无名-空-目录代表什么) 无名（空）目录代表什么？

此空目录名称用作Linux文件系统的无名基础。这用作所有其他目录，文件，驱动器和设备的附件。

#### [#](#什么是pwd命令) 什么是pwd命令？

pwd命令是print working directory命令的缩写。

```bash
PWD
/home/guru99/myDir
```

#### [#](#什么是守护进程) 什么是守护进程？

守护进程是提供基本操作系统下可能无法使用的多种功能的服务。其主要任务是监听服务请求，同时对这些请求采取行动。服务完成后，它将断开连接并等待进一步的请求。

#### [#](#如何从一个桌面环境切换到另一个桌面环境-例如从kde切换到gnome) 如何从一个桌面环境切换到另一个桌面环境，例如从KDE切换到Gnome？

假设你已安装这两个环境，只需从图形界面注销即可。然后在登录屏幕上，键入你的登录ID和密码，并选择要加载的会话类型。在你将其更改为其他选项之前，此选项将保持默认状态。

#### [#](#linux下的权限有哪些) Linux下的权限有哪些？

Linux下有3种权限：

- 读取：用户可以读取文件或列出目录
- 写入：用户可以写入新文件到目录的文件
- 执行：用户可以运行文件或查找特定文件一个目录

#### [#](#区分大小写如何影响命令的使用方式) 区分大小写如何影响命令的使用方式？

当我们讨论区分大小写时，只有当每个字符按原样编码时，命令才被认为是相同的，包括小写和大写字母。这意味着CD，CD和Cd是三个不同的命令。使用大写字母输入命令，它应该是小写的，将产生不同的输出。

#### [#](#是否可以使用快捷方式获取长路径名) 是否可以使用快捷方式获取长路径名？

就在这里。称为文件名扩展的功能允许你使用TAB键执行此操作。例如，如果你有一个名为/ home / iceman / assignments目录的路径，则键入如下：/ ho [tab] / ice [tab] / assi [tab]。但是，这假设路径是唯一的，并且你正在使用的shell支持此功能。

#### [#](#什么是重定向) 什么是重定向？

重定向是将数据从一个输出定向到另一个输出的过程。它还可以用于将输出作为输入定向到另一个进程。

#### [#](#什么是grep命令) 什么是grep命令？

grep使用基于模式的搜索的搜索命令。它使用与命令行一起指定的选项和参数，并在搜索所需的文件输出时应用此模式。

#### [#](#当发出的命令与上次使用时产生的结果不同时-会出现什么问题) 当发出的命令与上次使用时产生的结果不同时，会出现什么问题？

从看似相同的命令获得不同结果的一个非常可能的原因与区分大小写问题有关。由于Linux区分大小写，因此先前使用的命令可能以与当前格式不同的格式输入。例如，要列出目录中的所有文件，应键入命令ls，而不是LS。如果没有存在该确切名称的程序，则键入LS将导致错误消息，或者如果存在名为LS的程序执行另一个功能，则可能产生不同的输出。

#### [#](#usr-local的内容是什么) / usr / local的内容是什么？

它包含本地安装的文件。此目录在文件存储在网络上的环境中很重要。具体来说，本地安装的文件将转至/ usr / local / bin，/ usr / local / lib等。此目录的另一个应用是它用于从源安装的软件包，或未正式随分发一起提供的软件。

#### [#](#你如何终止正在进行的流程) 你如何终止正在进行的流程？

系统中的每个进程都由唯一的进程ID或pid标识。使用kill命令后跟pid来终止该进程。

要立即终止所有进程，请使用kill 0。

#### [#](#如何在命令行提示符中插入注释) 如何在命令行提示符中插入注释？

通过在实际注释文本之前键入＃符号来创建注释。这告诉shell完全忽略后面的内容。例如“＃这只是shell将忽略的注释。”

#### [#](#什么是命令分组以及它是如何工作的) 什么是命令分组以及它是如何工作的？

你可以使用括号对命令进行分组。例如，如果要将当前日期和时间以及名为OUTPUT的文件的内容发送到名为MYDATES的第二个文件，可以按如下方式应用命令分组：（date cat OUTPUT）> MYDATES

#### [#](#如何从单个命令行条目执行多个命令或程序) 如何从单个命令行条目执行多个命令或程序？

你可以通过使用分号符号分隔每个命令或程序来组合多个命令。例如，你可以在单个条目中发出这样一系列命令：

```bash
ls –l cd .. ls –a MYWORK which is equivalent to 3 commands: ls -l cd.. ls -a MYWORK
**请注意，这将按指定的顺序依次执行。
```

#### [#](#编写一个命令-查找扩展名为-c-的文件-并在其中出现字符串-apple) 编写一个命令，查找扩展名为“c”的文件，并在其中出现字符串“apple”?

```bash
find ./ -name "*.c" | xargs grep –i "apple"
```

#### [#](#编写一个显示所有-txt文件的命令-包括其个人权限。) 编写一个显示所有.txt文件的命令，包括其个人权限。

```bash
ls -al * .txt
```

#### [#](#解释如何为git控制台着色) 解释如何为Git控制台着色？

要为Git控制台着色，可以使用命令git config-global color.ui auto。在命令中，color.ui变量设置变量的默认值，例如color.diff和color.grep。

#### [#](#如何在linux中将一个文件附加到另一个文件) 如何在Linux中将一个文件附加到另一个文件？

要在Linux中将一个文件附加到另一个文件，你可以使用命令cat file2 >> file 1. operator >>附加指定文件的输出或创建文件（如果未创建）。而另一个命令cat文件1文件2>文件3将两个或多个文件附加到一个文件。

#### [#](#解释如何使用终端找到文件) 解释如何使用终端找到文件？

要查找文件，你必须使用命令，查找。-name“process.txt”。它将查找名为process.txt的文件的当前目录。

#### [#](#解释如何使用终端创建文件夹) 解释如何使用终端创建文件夹？

要创建文件夹，你必须使用命令mkdir。它将是这样的：〜$ mkdir Guru99

#### [#](#解释如何使用终端查看文本文件) 解释如何使用终端查看文本文件？

要查看文本文件，请使用命令cd转到文本文件所在的特定文件夹，然后键入less filename.txt。

#### [#](#解释如何在ubuntu-lamp堆栈上启用curl) 解释如何在Ubuntu LAMP堆栈上启用curl？

要在Ubuntu上启用curl，首先安装libcurl，完成后使用以下命令sudo /etc/init .d / apache2 restart或sudo service apache2 restart。

#### [#](#解释如何在ubuntu中启用root日志记录) 解释如何在Ubuntu中启用root日志记录？

启用root日志记录的命令是

```bash
#sudo sh-c'echo“greater-show-manual-login = true”>> / etc / lightdm / lightdm.conf'
```

#### [#](#如何在启动linux服务器的同时在后台运行linux程序) 如何在启动Linux服务器的同时在后台运行Linux程序？

通过使用nohup。它将停止接收NOHUP信号的进程，从而终止它，你注销了调用的程序。并在后台运行该过程。

#### [#](#解释如何在linux中卸载库) 解释如何在Linux中卸载库？

要在Linux中卸载库，可以使用命令

```bash
sudo apt-get remove library_name
```

### [#](#_15-2-docker) 15.2 Docker

#### [#](#什么是虚拟化技术) 什么是虚拟化技术？

在计算机技术中，虚拟化（Virtualization）是一种资源管理技术。它是将计算机的各种实体资源，如：服务器、网络、内存及存储等，予以抽象、转换后呈现出来，打破实体结构间的不可切割的障碍，使用户可以用更好的方式来利用这些资源。

虚拟化的目的是为了在同一个主机上运行多个系统或应用，从而提高系统资源的利用率，并带来降低成本、方便管理和容错容灾等好处。

- **硬件虚拟化**

硬件虚拟化就是硬件物理平台本身提供了对特殊指令的截获和重定向的支持。支持虚拟化的硬件，也是一些基于硬件实现软件虚拟化技术的关键。在基于硬件实现软件虚拟化的技术中，在硬件是实现虚拟化的基础，硬件(主要是CPU)会为虚拟化软件提供支持，从而实现硬件资源的虚拟化。

- **软件虚拟化**

软件虚拟化就是利用软件技术，在现有的物理平台上实现对物理平台访问的截获和模拟。在软件虚拟化技术中，有些技术不需要硬件支持，如：QEMU；而有些软件虚拟化技术，则依赖硬件支持，如：VMware、KVM。

#### [#](#什么是docker) 什么是Docker?

Docker是一个开源的应用容器引擎，它让开发者可以打包他们的应用以及依赖包到一个可移植的容器中，然后发布到安装了任何 Linux 发行版本的机器上。Docker基于LXC来实现类似VM的功能，可以在更有限的硬件资源上提供给用户更多的计算资源。与同VM等虚拟化的方式不同，LXC不属于全虚拟化、部分虚拟化或半虚拟化中的任何一个分类，而是一个操作系统级虚拟化。

Docker是直接运行在宿主操作系统之上的一个容器，使用沙箱机制完全虚拟出一个完整的操作，容器之间不会有任何接口，从而让容器与宿主机之间、容器与容器之间隔离的更加彻底。每个容器会有自己的权限管理，独立的网络与存储栈，及自己的资源管理能，使同一台宿主机上可以友好的共存多个容器。

Docker借助Linux的内核特性，如：控制组（Control Group）、命名空间（Namespace）等，并直接调用操作系统的系统调用接口。从而降低每个容器的系统开销，并实现降低容器复杂度、启动快、资源占用小等特征。

#### [#](#docker和虚拟机的区别) Docker和虚拟机的区别？

虚拟机Virtual Machine与容器化技术（代表Docker）都是虚拟化技术，两者的区别在于虚拟化的程度不同。

- **举个例子**

1. **服务器**：比作一个大型的仓管基地，包含场地与零散的货物——相当于各种服务器资源。
2. **虚拟机技术**：比作仓库，拥有独立的空间堆放各种货物或集装箱，仓库之间完全独立——仓库相当于各种系统，独立的应用系统和操作系统。
3. **Docker**：比作集装箱，操作各种货物的打包——将各种应用程序和他们所依赖的运行环境打包成标准的容器，容器之间隔离。

- **基于一个图解释**

![img](/images/devops/docker/docker-y-0.jpg)

1. 虚拟机管理系统（Hypervisor）。利用Hypervisor，可以在主操作系统之上运行多个不同的从操作系统。类型1的Hypervisor有支持MacOS的HyperKit，支持Windows的Hyper-V以及支持Linux的KVM。类型2的Hypervisor有VirtualBox和VMWare。
2. Docker守护进程（Docker Daemon）。Docker守护进程取代了Hypervisor，它是运行在操作系统之上的后台进程，负责管理Docker容器。
3. vm多了一层guest OS，虚拟机的Hypervisor会对硬件资源也进行虚拟化，而容器Docker会直接使用宿主机的硬件资源

- **基于虚拟化角度**

1. **隔离性** 由于vm对操作系统也进行了虚拟化，隔离的更加彻底。而Docker共享宿主机的操作系统，隔离性较差。
2. **运行效率** 由于vm的隔离操作，导致生成虚拟机的速率大大低于容器Docker生成的速度，因为Docker直接利用宿主机的系统内核。因为虚拟机增加了一层虚拟硬件层，运行在虚拟机上的应用程序在进行数值计算时是运行在Hypervisor虚拟的CPU上的；另外一方面是由于计算程序本身的特性导致的差异。虚拟机虚拟的cpu架构不同于实际cpu架构，数值计算程序一般针对特定的cpu架构有一定的优化措施，虚拟化使这些措施作废，甚至起到反效果。
3. **资源利用率** 在资源利用率上虚拟机由于隔离更彻底，因此利用率也会相对较低。

#### [#](#docker的架构) Docker的架构？

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。

- **Docker 客户端(Client)** : Docker 客户端通过命令行或者其他工具使用 Docker SDK (https://docs.docker.com/develop/sdk/) 与 Docker 的守护进程通信。
- **Docker 主机(Host)** ：一个物理或者虚拟的机器用于执行 Docker 守护进程和容器。

Docker 包括三个基本概念:

- **镜像（Image）**：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- **容器（Container）**：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- **仓库（Repository）**：仓库可看着一个代码控制中心，用来保存镜像。

![img](/images/devops/docker/docker-x-1.png)

#### [#](#docker镜像相关操作有哪些) Docker镜像相关操作有哪些？

```bash
# 查找镜像
docker search mysql

# 拉取镜像
docker pull mysql

# 删除镜像
docker rmi hello-world

# 更新镜像
docker commit -m="update test" -a="pdai" 0a1556ca3c27  pdai/ubuntu:v1.0.1

# 生成镜像
docker build -t pdai/ubuntu:v2.0.1 .

# 镜像标签
docker tag a733d5a264b5 pdai/ubuntu:v3.0.1

# 镜像导出
docker save > pdai-ubuntu-v2.0.2.tar 57544a04cd1a

# 镜像导入
docker load < pdai-ubuntu-v2.0.2.tar
```

#### [#](#docker容器相关操作有哪些) Docker容器相关操作有哪些？

```bash
# 容器查看
docker ps -a

# 容器启动
docker run -it pdai/ubuntu:v2.0.1 /bin/bash

# 容器停止
docker stop f5332ebce695

# 容器再启动
docker start f5332ebce695

# 容器重启
docker restart f5332ebce695

# 容器导出
docker export f5332ebce695 > ubuntu-pdai-v2.tar

# 容器导入
docker import ubuntu-pdai-v2.tar pdai/ubuntu:v2.0.2

# 容器强制停止并删除
docker rm -f f5332ebce695

# 容器清理
docker container prune

# 容器别名操作
docker run -itd --name pdai-ubuntu-202 pdai/ubuntu:v2.0.2 /bin/bash
```

#### [#](#如何查看docker容器的日志) 如何查看Docker容器的日志？

```bash
#例：实时查看docker容器名为user-uat的最后10行日志
docker logs -f -t --tail 10 user-uat

#例：查看指定时间后的日志，只显示最后100行：
docker logs -f -t --since="2018-02-08" --tail=100 user-uat

#例：查看最近30分钟的日志:
docker logs --since 30m user-uat

#例：查看某时间之后的日志：
docker logs -t --since="2018-02-08T13:23:37" user-uat

#例：查看某时间段日志：
docker logs -t --since="2018-02-08T13:23:37" --until "2018-02-09T12:23:37" user-uat

#例：将错误日志写入文件：
docker logs -f -t --since="2018-02-18" user-uat | grep error >> logs_error.txt
```

#### [#](#如何启动docker容器-参数含义) 如何启动Docker容器？参数含义？

```bash
[root@pdai docker-test]# docker run -itd pdai/ubuntu:v2.0.1 /bin/bash
```

- `-it` 可以连写的，表示 `-i -t`
- `-t`: 在新容器内指定一个伪终端或终端。
- `-i`: 允许你对容器内的标准输入 (STDIN) 进行交互
- `-d`: 后台模式

#### [#](#如何进入docker后台模式-有什么区别) 如何进入Docker后台模式？有什么区别？

- 第一种：`docker attach`

```bash
[root@pdai ~]# docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
f5332ebce695        pdai/ubuntu:v2.0.1   "/bin/bash"         38 minutes ago      Up 2 seconds        22/tcp, 80/tcp      jolly_kepler
[root@pdai ~]# docker attach f5332ebce695
root@f5332ebce695:/# echo 'pdai'
pdai
root@f5332ebce695:/# exit
exit
[root@pdai ~]# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```

看到没，使用`docker attach`进入后，exit便容器也停止了。

- 第二种：`docker exec`

```bash
[root@pdai ~]# docker exec -it f5332ebce695 /bin/bash
Error response from daemon: Container f5332ebce69520fba353f035ccddd4bd42055fbd1e595f916ba7233e26476464 is not running
[root@pdai ~]# docker restart f5332ebce695
f5332ebce695
[root@pdai ~]# docker exec -it f5332ebce695 /bin/bash
root@f5332ebce695:/# exit
exit
[root@pdai ~]# docker ps
CONTAINER ID        IMAGE                COMMAND             CREATED             STATUS              PORTS               NAMES
f5332ebce695        pdai/ubuntu:v2.0.1   "/bin/bash"         42 minutes ago      Up 8 seconds        22/tcp, 80/tcp      jolly_kepler
```

注意：

- 我特意在容器停止状态下执行了`docker exec`，是让你看到`docker exec`是在容器启动状态下用的，且注意下错误信息；
- 推荐大家使用 `docker exec` 命令，因为此退出容器终端，不会导致容器的停止。

### [#](#_15-3-ci-cd) 15.3 CI/CD

#### [#](#什么是ci) 什么是CI？

CI的英文名称是Continuous Integration，中文翻译为：持续集成。

![img](/images/devops/cicd/cicd-2.png)

CI中，开发人员将会频繁地向主干提交代码，这些新提交的代码在最终合并到主干前，需要经过编译和自动化测试流进行验证。 持续集成（CI）是在源代码变更后自动检测、拉取、构建和（在大多数情况下）进行单元测试的过程。持续集成的目标是快速确保开发人员新提交的变更是好的，并且适合在代码库中进一步使用。CI的流程执行和理论实践让我们可以确定新代码和原有代码能否正确地集成在一起。

通俗点讲就是：通过持续集成， 开发人员能够在任何时候多次向仓库提交作品，而不是独立地开发每个功能模块并在开发周期结束时一一提交。这里的一个重要思想就是让开发人员更快更、频繁地做到这一点，从而降低集成的开销。 实际情况中，开发人员在集成时经常会发现新代码和已有代码存在冲突。 如果集成较早并更加频繁，那么冲突将更容易解决且执行成本更低。当然，这里也有一些权衡，这个流程不提供额外的质量保障。 事实上，许多组织发现这样的集成方式开销更大，因为它们依赖人工确保新代码不会引起新的 bug 或者破坏现有代码。 为了减少集成期间的摩擦，持续集成依赖于测试套件和自动化测试。 然而，要认识到自动化测试和持续测试是完全不同的这一点很重要。

CI 的目标是将集成简化成一个简单、易于重复的日常开发任务， 这样有助于降低总体的构建成本并在开发周期的早期发现缺陷。 要想有效地使用 CI 必须转变开发团队的习惯，要鼓励频繁迭代构建， 并且在发现 bug 的早期积极解决。

#### [#](#什么是cd) 什么是CD？

这里的CD可对应多个英文名称，**持续交付Continuous Delivery**和**持续部署Continuous Deployment**。下面我们分别来看看上面是持续交付和持续部署。

- **持续交付**

持续交付（CD）实际上是 CI 的扩展，其中软件交付流程进一步自动化，以便随时轻松地部署到生成环境中。 成熟的持续交付方案也展示了一个始终可部署的代码库。使用 CD 后，软件发布将成为一个没有任何紧张感的例行事件。 开发团队可以在日常开发的任何时间进行产品级的发布，而不需要详细的发布方案或者特殊的后期测试。

完成 CI 中构建及单元测试和集成测试的自动化流程后，持续交付可自动将已验证的代码发布到存储库。为了实现高效的持续交付流程，务必要确保 CI 已内置于开发管道。持续交付的目标是拥有一个可随时部署到生产环境的代码库。

![img](/images/devops/cicd/cicd-3.png)

在持续交付中，每个阶段（从代码更改的合并，到生产就绪型构建版本的交付）都涉及测试自动化和代码发布自动化。在流程结束时，运维团队可以快速、轻松地将应用部署到生产环境中或发布给最终使用的用户。

CD 集中依赖于部署流水线，团队通过流水线自动化测试和部署过程。此流水线是一个自动化系统， 可以针对构建执行一组渐进的测试套件。CD 具有高度的自动化，并且在一些云计算环境中也易于配置。在流水线的每个阶段，如果构建无法通过关键测试会向团队发出警报。否则，将继续进入下一个测试， 并在连续通过测试后自动进入下一个阶段。流水线的最后一个部分会将构建部署到和生产环境等效的环境中。 这是一个整体的过程，因为构建、部署和环境都是一起执行和测试的，它能让构建在实际的生产环境可部署和可验证。

- **持续部署**

持续部署扩展了持续交付，以便软件构建在通过所有测试时自动部署。在这样的流程中， 不需要人为决定何时及如何投入生产环境。CI/CD 系统的最后一步将在构建后的组件/包退出流水线时自动部署。 此类自动部署可以配置为快速向客户分发组件、功能模块或修复补丁，并准确说明当前提供的内容。采用持续部署的组织可以将新功能快速传递给用户，得到用户对于新版本的快速反馈，并且可以迅速处理任何明显的缺陷。 用户对无用或者误解需求的功能的快速反馈有助于团队规划投入，避免将精力集中于不容易产生回报的地方。

随着 DevOps 的发展，新的用来实现 CI/CD 流水线的自动化工具也在不断涌现。这些工具通常能与各种开发工具配合， 包括像 GitHub 这样的代码仓库和 Jira 这样的 bug 跟踪工具。此外，随着 SaaS 这种交付方式变得更受欢迎， 许多工具都可以在现代开发人员运行应用程序的云环境中运行，例如 GCP 和 AWS。但是对于一个成熟的CI/CD管道（Pipeline）来说，最后的阶段是持续部署。作为持续交付——自动将生产就绪型构建版本发布到代码存储库——的延伸，持续部署可以自动将应用发布到生产环境。

![img](/images/devops/cicd/cicd-4.png)

#### [#](#什么是ci-cd的管道) 什么是CI/CD的管道？

CI / CD管道是与自动化工具和改进的工作流程集成的部署管道。 如果执行得当，它将最大程度地减少人为错误，并增强整个SDLC的反馈循环，使团队可以在更短的时间内交付较小的发行版。

![img](/images/devops/cicd/cicd-1.png)

典型的CI / CD管道必须包括以下阶段：

- 构建阶段
- 测试阶段
- 部署阶段
- 自动化测试阶段
- 部署到生产

#### [#](#如何理解devops) 如何理解DevOPS?

DevOps是Development和Operations的组合，是一种方法论，是一组过程、方法与系统的统称，用于促进应用开发、应用运维和质量保障（QA）部门之间的沟通、协作与整合。以期打破传统开发和运营之间的壁垒和鸿沟。

CI、CD和DevOps之间的关系 ：

![img](/images/devops/cicd/cicd-5.png)

#### [#](#在完全部署到所有用户之前-有哪些方法可以测试部署) 在完全部署到所有用户之前，有哪些方法可以测试部署？

由于必须回滚/撤消对所有用户的部署可能是一种代价高昂的情况（无论是技术上还是用户的感知），已经有许多技术允许“尝试”部署新功能并在发现问题时轻松“撤消”它们。这些包括：

- **蓝/绿测试/部署**

在这种部署软件的方法中，维护了两个相同的主机环境 —— 一个“蓝色” 和一个“绿色”。（颜色并不重要，仅作为标识。）对应来说，其中一个是“生产环境”，另一个是“预发布环境”。

在这些实例的前面是调度系统，它们充当产品或应用程序的客户“网关”。通过将调度系统指向蓝色或绿色实例，可以将客户流量引流到期望的部署环境。通过这种方式，切换指向哪个部署实例（蓝色或绿色）对用户来说是快速，简单和透明的。

当新版本准备好进行测试时，可以将其部署到非生产环境中。在经过测试和批准后，可以更改调度系统设置以将传入的线上流量指向它（因此它将成为新的生产站点）。现在，曾作为生产环境实例可供下一次候选发布使用。

同理，如果在最新部署中发现问题并且之前的生产实例仍然可用，则简单的更改可以将客户流量引流回到之前的生产实例 —— 有效地将问题实例“下线”并且回滚到以前的版本。然后有问题的新实例可以在其它区域中修复。

- **金丝雀测试/部署**

在某些情况下，通过蓝/绿发布切换整个部署可能不可行或不是期望的那样。另一种方法是为金丝雀测试/部署。在这种模型中，一部分客户流量被重新引流到新的版本部署中。例如，新版本的搜索服务可以与当前服务的生产版本一起部署。然后，可以将 10％ 的搜索查询引流到新版本，以在生产环境中对其进行测试。

如果服务那些流量的新版本没问题，那么可能会有更多的流量会被逐渐引流过去。如果仍然没有问题出现，那么随着时间的推移，可以对新版本增量部署，直到 100％ 的流量都调度到新版本。这有效地“更替”了以前版本的服务，并让新版本对所有客户生效。

- **功能开关**

对于可能需要轻松关掉的新功能（如果发现问题），开发人员可以添加功能开关。这是代码中的 if-then 软件功能开关，仅在设置数据值时才激活新代码。此数据值可以是全局可访问的位置，部署的应用程序将检查该位置是否应执行新代码。如果设置了数据值，则执行代码；如果没有，则不执行。

这为开发人员提供了一个远程“终止开关”，以便在部署到生产环境后发现问题时关闭新功能。

- **暗箱发布**

在暗箱发布中，代码被逐步测试/部署到生产环境中，但是用户不会看到更改（因此名称中有暗箱一词）。例如，在生产版本中，网页查询的某些部分可能会重定向到查询新数据源的服务。开发人员可收集此信息进行分析，而不会将有关接口，事务或结果的任何信息暴露给用户。

这个想法是想获取候选版本在生产环境负载下如何执行的真实信息，而不会影响用户或改变他们的经验。随着时间的推移，可以调度更多负载，直到遇到问题或认为新功能已准备好供所有人使用。实际上功能开关标志可用于这种暗箱发布机制。

#### [#](#什么是持续测试) 什么是持续测试？

持续测试是一个过程，它将自动化测试作为软件交付通道中内嵌的一部分，以尽快获得软件发布后业务风险的反馈。

**持续测试与自动化测试的侧重点**？

- 自动化测试旨在生成一组与用户故事或应用程序要求相关的通过/失败的数据点。
- 持续测试侧重于业务风险，并提供有关软件是否可以发布的判断。要实现这一转变，我们需要停止询问“我们是否已完成测试？”而是集中精力在“发布版本是否具有可接受的业务风险级别？”

**为什么我们需要持续测试**？

今天，整个行业的变化要求测试更多，同时使自动化测试更难实现（至少使用传统工具和方法）：

- 应用程序体系结构越来越分散和复杂，包含云，API，微服务等，并在单个业务事务中创建几乎无限的不同协议和技术组合。
- 由于Agile，DevOps和持续交付，许多应用程序现在每两周发布一次，每天发布数千次。因此，可用于测试设计，维护和特别是执行的时间大大减少。

既然软件是业务的主要接口，那么应用程序故障就是业务失败， 如果它影响用户体验，即使是看似微不足道的小故障也会产生严重后果。因此，与应用相关的风险已成为即使是非技术性商业领袖的主要关注点。

#### [#](#如何做版本管理) 如何做版本管理？

![img](/images/git/git-gitflow-1.png)

- **Master 分支** 主分支，这个分支最近发布到生产环境的代码，最近发布的Release， 这个分支只能从其他分支合并，不能在这个分支直接修改
- **Develop 分支** 这个分支是我们是我们的主开发分支，包含所有要发布到下一个Release的代码，这个主要合并与其他分支，比如Feature分支
- **Feature 分支** 这个分支主要是用来开发一个新的功能，一旦开发完成，我们合并回Develop分支进入下一个Release
- **Release 分支** 当你需要发布一个新Release的时候，我们基于Develop分支创建一个Release分支，完成Release后，我们合并到Master和Develop分支
- **Hotfix 分支** 当我们在Production发现新的Bug时候，我们需要创建一个Hotfix, 完成Hotfix后，我们合并回Master和Develop分支，所以Hotfix的改动会进入下一个Release

### [#](#_15-4-监控体系) 15.4 监控体系

#### [#](#为什么要有监控系统-谈谈你对监控的理解) 为什么要有监控系统？ 谈谈你对监控的理解？

**监控的目标**？

- 发现问题：当系统发生故障报警，我们会收到故障报警的信息。
- 定位问题：故障邮件一般都会写某某主机故障、具体故障的内容，我们需要对报警内容进行分析。比如一台服务器连不上，我们就需要考虑是网络问题、还是负载太高导致长时间无法连接，又或者某开发触发了防火墙禁止的相关策略等，我们就需要去分析故障具体原因。
- 解决问题：当然我们了解到故障的原因后，就需要通过故障解决的优先级去解决该故障。
- 总结问题：当我们解决完重大故障后，需要对故障原因以及防范进行总结归纳，避免以后重复出现。

**具体而言**？

- 对系统不间断的实时监控：实际上是对系统不间断的实时监控(这就是监控)；
- 实时反馈系统当前状态：我们监控某个硬件、或者某个系统，都是需要能实时看到当前系统的状态，是正常、异常、或者故障。
- 保证服务可靠性安全性：我们监控的目的就是要保证系统、服务、业务正常运行
- 保证业务持续稳定运行：如果我们的监控做得很完善，即使出现故障，能第一时间接收到故障报警，在第一时间处理解决，从而保证业务持续性的稳定运行。

#### [#](#监控体系监控哪些内容) 监控体系监控哪些内容？

1、**硬件监控** 通过SNMP来进行路由器交换机的监控(这些可以跟一些厂商沟通来了解如何做)、服务器的温度以及其它，可以通过IPMI来实现。当然如果没有硬件全都是云，直接跳过这一步骤。

2、**系统监控** 如CPU的负载，上下文切换、内存使用率、磁盘读写、磁盘使用率、磁盘inode使用率。当然这些都是需要配置触发器，因为默认太低会频繁报警。

3、**服务监控** 比如公司用的LNMP架构，Nginx自带Status模块、PHP也有相关的Status、MySQL的话可以通过Percona官方工具来进行监控。Redis这些通过自身的info获取信息进行过滤等。方法都类似。要么服务自带。要么通过脚本来实现想监控的内容，以及报警和图形功能。

4、**网络监控** 如果是云主机又不是跨机房，那么可以选择不监控网络。当然你说我们是跨机房以及如何如何，推荐使用smokeping来做网络相关的监控，或者直接交给你们的网络工程师来做，因为术业有专攻。

5、**安全监控** 如果是云主机可以考虑使用自带的安全防护。当然也可以使用iptables。如果是硬件，那么推荐使用硬件防火墙。使用云可以购买防DDOS，避免出现故障导致down机一天。如果是系统，那么权限、密码、备份、恢复等基础方案要做好。Web同时也可以使用Nginx+Lua来实现一个Web层面的防火墙。当然也可以使用集成好的OpenResty。

6、**Web监控** Web监控的话题其实还是很多。比如可以使用自带的Web监控来监控页面相关的延迟、js响应时间、下载时间、等等。这里我推荐使用专业的商业软件监控宝或听云来实现。毕竟人家全国各地都有机房（如果本身是多机房那就另说了）。

7、**日志监控** 如果是Web的话可以使用监控Nginx的50x、40x的错误日志，PHP的ERROR日志。其实这些需求无非是，收集、存储、查询、展示，我们其实可以使用开源的ELKStack来实现。Logstash（收集）、Elasticsearch（存储+搜索）、Kibana（展示）。

8、**业务监控** 上面做了那么多，其实最终还是保证业务的运行。这样我们做的监控才有意义。所以业务层面这块的监控需要和开发以及总监开会讨论，监控比较重要的业务指标，（需要开会确认）然后通过简单的脚本就可以实现，最后设置触发器即可 。

9、**流量分析** 平时我们分析日志都是拿awk sed xxx一堆工具来实现。这样对我们统计IP、PV、UV不是很方便。那么可以使用百度统计、Google统计、商业，让开发嵌入代码即可。为了避免隐私也可以使用Piwik来做相关的流量分析。

10、**可视化** 通过Screen以及引入一些第三方的库来美化界面，同时我们也需要知道，订单量突然增加、突然减少。或者说突然来了一大波流量，这流量从哪儿来，是不是推广了，还是被攻击了。可以结合监控平来梳理各个系统之间的业务关系。

11、**自动化监控** 如上我们做了那么多的工作，当然不能是一台一台的来加key实现。可以通过Zabbix的主动模式以及被动模式来实现。当然最好还是通过API来实现。

#### [#](#监控一般采用什么样的流程) 监控一般采用什么样的流程？

- **采集** 通过SNMP、Agent、ICMP、SSH、IPMI等对系统进行数据采集
- **存储** 各类数据库服务,MySQL、PostgreSQL, 时序库等
- **分析** 提供图形及时间线情况信息，方便我们定位故障所在
- **展示** 指标信息、指标趋势展示
- **报警** 电话、邮件、微信、短信、报警升级机制
- **处理** 故障级别判定，找响应人员进行快速处理

## [#](#_16-其它) 16 其它

### [#](#_16-1-设计模式) 16.1 设计模式

### [#](#_16-2-开源协议) 16.2 开源协议

#### [#](#说说常见的开源协议) 说说常见的开源协议？

最流行的六种：MIT、Apache、BSD、GPL和LGPL、Mozilla。

![img](/images/dev_opensource_1.png)

#### [#](#gpl协议、lgpl协议与bsd协议的法律区别) GPL协议、LGPL协议与BSD协议的法律区别？

简而言之，GPL协议就是一个开放源代码协议，软件的初始开发者使用了GPL协议并公开软件的源程序后，后续使用该软件源程序开发软件者亦应当根据GPL协议把自己编写的源程序进行公开。GPL协议要求的关键在于开放源程序，但并不排斥软件作者向用户收费。虽然如此，很多大公司对GPL协议还是又爱又恨，爱的是这个协议项下的软件历经众多程序员千锤百炼的修改，已经非常成熟完善，恨的是必须开放自己后续的源程序，导致竞争对手也可以根据自己修改的源程序开发竞争产品。

正因大公司对GPL协议在商业上存在顾虑，因此，另两种协议被采用的更多，第一种是LGPL(亦称GPL V2)协议，可以翻译为更宽松的GPL协议。与GPL协议的区别为，后者如果只是对LGPL软件的程序库的程序进行调用而不是包含其源代码时，相关的源程序无需开源。调用和包含的区别类似在互联网网网页上对他人网页内容的引用: 如果把他人的内容全部或部分复制到自己的网页上，就类似包含，如果只是贴一个他人网页的网址链接而不引用内容，就类似调用。有了这个协议，很多大公司就可以把很多自己后续开发内容的源程序隐藏起来。

第二种是BSD协议(类似的还有MIT协议)。BSD协议鼓励软件的作者公开自己后续开发的源代码，但不强求。在BSD协议项下开发的软件，原始的源程序是开放源代码的，但使用者修改以后，可以自行选择发布源程序或者二进制程序(即目标程序)，当然，使用者有义务把自己原来使用的源程序与BSD协议在软件对外发布时一并发布。因为比较灵活，所以BSD深受大公司的欢迎。

#### [#](#mongodb修改开源协议) MongoDB修改开源协议？

2018年10月，MongoDB宣布其开源许可证将从GNU AGPLv3，切换到SSPL，新许可证将适用于新版本的MongoDB Community Server以及打过补丁的旧版本。

根据 MongoDB 之前的 GNU AGPLv3 协议，想要将 MongoDB 作为公共服务运行的公司必须将他们的软件开源，或需要从 MongoDB 获得商业许可，”该公司解释说，“然而，MongoDB 的普及使一些组织在违反 GNU AGPLv3 协议的边缘疯狂试探，甚至直接违反了协议。”

尽管 SSPL 与 GNU AGPLv3 没有什么不同，但 SSPL 会明确要求托管 MongoDB 实例的云计算公司要么从 MongoDB 获取商业许可证，要么向社区开源其服务代码。

随后Red Hat宣布，将不会在Red Hat Enterprise Linux或Fedora中使用MongoDB。事实上，MongoDB修改开源协议之后，Red Hat并不是首家弃用的Linux社区。2018年12月5日，Linux发行版Debian在邮件列表中讨论并决定不使用SSPL协议下的软件。2019年1月，Fedora Legal也对SSPL v1协议做出了相关决定，Fedora已确定服务器端公共许可证v1（SSPL）不是自由软件许可证。

### [#](#_16-3-软件理论) 16.3 软件理论

#### [#](#什么是cap理论) 什么是CAP理论？

CAP原理指的是，在分布式系统中这三个要素最多只能同时实现两点，不可能三者兼顾。因此在进行分布式架构设计时，必须做出取舍。而对于分布式数据系统，分区容忍性是基本要求，否则就失去了价值。因此设计分布式数据系统，就是在一致性和可用性之间取一个平衡。对于大多数Web应用，其实并不需要强一致性，因此牺牲一致性而换取高可用性，是目前多数分布式数据库产品的方向。

1. 一致性（Consistency）：数据在多个副本之间是否能够保持一致的特性。（当一个系统在一致状态下更新后，应保持系统中所有数据仍处于一致的状态）
2. 可用性（Availability）：系统提供的服务必须一直处于可用状态，对每一个操作的请求必须在有限时间内返回结果。
3. 分区容错性（Tolerance of network Partition）：分布式系统在遇到网络分区故障时，仍然需要保证对外提供一致性和可用性的服务，除非整个网络都发生故障。

**为什么只能同时满足两个**？

例如，服务器中原本存储的value=0，当客户端A修改value=1时，为了保证数据的一致性，要写到3个服务器中，当服务器C故障时，数据无法写入服务器C，则导致了此时服务器A、B和C的value是不一致的。这时候要保证分区容错性，即当服务器C故障时，仍然能保持良好的一致性和可用性服务，则Consistency和Availability不能同时满足。为什么呢？

如果满足了一致性，则客户端A的写操作value=1不能成功，这时服务器中所有value=0。 如果满足可用性，即所有客户端都可以提交操作并得到返回的结果，则此时允许客户端A写入服务器A和B，客户端C将得到未修改之前的value=0结果。

#### [#](#什么是base理论) 什么是BASE理论？

1. **Basically Available**（基本可用）分布式系统在出现不可预知故障的时候，允许损失部分可用性
2. **Soft state**（软状态）软状态也称为弱状态，和硬状态相对，是指允许系统中的数据存在中间状态，并认为该中间状态的存在不会影响系统的整体可用性，即允许系统在不同节点的数据副本之间进行数据同步的过程存在延时。
3. **Eventually consistent**（最终一致性）最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能够达到一个一致的状态。因此，最终一致性的本质是需要系统保证最终数据能够达到一致，而不需要实时保证系统数据的强一致性

**CAP 与 BASE 关系**？

BASE是对CAP中一致性和可用性权衡的结果，其来源于对大规模互联网系统分布式实践的结论，是基于CAP定理逐步演化而来的，其核心思想是即使无法做到强一致性（Strong consistency），更具体地说，是对 CAP 中 AP 方案的一个补充。其基本思路就是：通过业务，牺牲强一致性而获得可用性，并允许数据在一段时间内是不一致的，但是最终达到一致性状态。

**CAP 与 ACID 关系**？

ACID 是传统数据库常用的设计理念，追求强一致性模型。BASE 支持的是大型分布式系统，提出通过牺牲强一致性获得高可用性。

ACID 和 BASE 代表了两种截然相反的设计哲学，在分布式系统设计的场景中，系统组件对一致性要求是不同的，因此 ACID 和 BASE 又会结合使用。

#### [#](#什么是solid原则) 什么是SOLID原则？

- **S单一职责SRP Single-Responsibility Principle**

一个类,最好只做一件事,只有一个引起它的变化。单一职责原则可以看做是低耦合,高内聚在面向对象原则的引申,将职责定义为引起变化的原因,以提高内聚性减少引起变化的原因。

比如： SpringMVC 中Entity,DAO,Service,Controller, Util等的分离。

- **O开放封闭原则OCP Open - Closed Principle**

对扩展开放，对修改关闭(设计模式的核心原则)

比如： 设计模式中模板方法模式和观察者模式都是开闭原则的极好体现

- **L里氏替换原则LSP Liskov Substitution Principle**

任何基类可以出现的地方,子类也可以出现；这一思想表现为对继承机制的约束规范,只有子类能够替换其基类时,才能够保证系统在运行期内识别子类,这是保证继承复用的基础。

比如：正方形是长方形是理解里氏代换原则的经典例子。（讲的是基类和子类的关系，只有这种关系存在时，里氏代换原则才存在）

- **I接口隔离法则ISL Interface Segregation Principle**

客户端不应该依赖那些它不需要的接口。(接口隔离原则是指使用多个专门的接口，而不使用单一的总接口; 这个法则与迪米特法则是相通的)

- **D依赖倒置原则DIP Dependency-Inversion Principle**

要依赖抽象,而不要依赖具体的实现, 具体而言就是高层模块不依赖于底层模块,二者共同依赖于抽象。抽象不依赖于具体, 具体依赖于抽象。

#### [#](#什么是合成-聚合复用原则) 什么是合成/聚合复用原则？

Composite/Aggregate ReusePrinciple ，CARP: 要尽量使用对象组合,而不是继承关系达到软件复用的目的。

组合/聚合可以使系统更加灵活，类与类之间的耦合度降低，一个类的变化对其他类造成的影响相对较少，因此一般首选使用组合/聚合来实现复用；其次才考虑继承，在使用继承时，需要严格遵循里氏代换原则，有效使用继承会有助于对问题的理解，降低复杂度，而滥用继承反而会增加系统构建和维护的难度以及系统的复杂度，因此需要慎重使用继承复用。

此原则和里氏代换原则氏相辅相成的,两者都是具体实现"开-闭"原则的规范。违反这一原则，就无法实现"开-闭"原则。

#### [#](#什么是迪米特法则) 什么是迪米特法则？

Law of Demeter，LoD: 系统中的类,尽量不要与其他类互相作用,减少类之间的耦合度.

又叫最少知识原则(Least Knowledge Principle或简写为LKP).

- 不要和“陌生人”说话。英文定义为: Don't talk to strangers.
- 只与你的直接朋友通信。英文定义为: Talk only to your immediate friends.

比如：外观模式Facade(结构型)

#### [#](#什么是康威定律) 什么是康威定律？

康威在一篇文章中描述： 设计系统的组织，其产生的设计等同于组织之内、组织之间的沟通结构。

- 定律一: 组织沟通方式会通过系统设计表达出来，就是说架构的布局和组织结构会有相似。
- 定律二: 时间再多一件事情也不可能做的完美，但总有时间做完一件事情。一口气吃不成胖子，先搞定能搞定的。
- 定律三: 线型系统和线型组织架构间有潜在的异质同态特性。种瓜得瓜，做独立自治的子系统减少沟通成本。
- 定律四: 大的系统组织总是比小系统更倾向于分解。合久必分，分而治之。

### [#](#_16-4-软件成熟度模型) 16.4 软件成熟度模型

#### [#](#什么是cmm) 什么是CMM？

由美国卡内基梅隆大学的软件工程研究所(SEI)创立的CMM(Capability Maturity Model 软件能力成熟度模型)认证评估，在过去的十几年中，对全球的软件产业产生了非常深远的影响。CMM共有五个等级，分别标志着软件企业能力成熟度的五个层次。从低到高，软件开发生产计划精度逐级升高，单位工程生产周期逐级缩短，单位工程成本逐级降低。据SEI统计，通过评估的软件公司对项目的估计与控制能力约提升40%到50%；生产率提高10%到20%，软件产品出错率下降超过1/3。

对一个软件企业来说，达到CMM2就基本上进入了规模开发，基本具备了一个现代化软件企业的基本架构和方法，具备了承接外包项目的能力。CMM3评估则需要对大软件集成的把握，包括整体架构的整合。一般来说，通过CMM认证的级别越高，其越容易获得用户的信任，在国内、国际市场上的竞争力也就越强。因此，是否能够通过CMM认证也成为国际上衡量软件企业工程开发能力的一个重要标志。

CMM是目前世界公认的软件产品进入国际市场的通行证，它不仅仅是对产品质量的认证，更是一种软件过程改善的途径。参与CMM评估的博科负责人表示，通过CMM的评估认证不是目标，它只是推动软件企业在产品的研发、生产、服务和管理上不断成熟和进步的手段，是一种持续提升和完善企业自身能力的过程。此次由美国PIA咨询公司负责评估并最终通过CMM3认证，标志着博科在质量管理的能力已经上升到一个新的高度。

#### [#](#什么是cmmi5-呢) 什么是CMMI5 呢？

CMMI全称是Capability Maturity Model Integration, 即软件能力成熟度模型集成模型，是由美国国防部与卡内基-梅隆大学和美国国防工业协会共同开发和研制的。CMMI是一套融合多学科的、可扩充的产品集合， 其研制的初步动机是为了利用两个或多个单一学科的模型实现一个组织的集成化过程改进

CMMI分为五个等级，二十五个过程区域（PA）。

1． **初始级** 软件过程是**无序的**，有时甚至是混乱的，对过程几乎没有定义，成功取决于个人努力。管理是反应式的。

2． 已管理级 建立了**基本的项目管理**过程来跟踪费用、进度和功能特性。制定了必要的过程纪律，能重复早先类似应用项目取得的成功经验。

3． 已定义级 已将软件管理和工程两方面的过程**文档化、标准化**，并综合成该组织的标准软件过程。所有项目均使用经批准、剪裁的标准软件过程来开发和维护软件，软件产品的生产在整个软件过程是可见的。

4． 量化管理级 分析对软件过程和产品质量的详细度量数据，对软件过程和产品都有定量的理解与控制。管理有一个作出结论的客观依据，管理能够在定量的范围内**预测**性能。

5． 优化管理级 过程的**量化反馈**和先进的新思想、新技术促使过程持续不断改进。

每个等级都被分解为过程域，特殊目标和特殊实践，通用目标、通用实践和共同特性：

每个等级都有几个过程区域组成，这几个过程域共同形成一种软件过程能力。每个过程域，都有一些特殊目标和通用目标，通过相应的特殊实践和通用实践来实现这些目标。当一个过程域的所有特殊实践和通用实践都按要求得到实施，就能实现该过程域的目标。

#### [#](#cmmi与cmm的区别呢) CMMI与CMM的区别呢？

CMM是指“能力成熟度模型”，其英文全称为Capability Maturity Model for Software；

CMMI 是指“能力成熟度模型集成”，全称为：Capability Maturity Model Integration；

CMMI是系统工程和软件工程的集成成熟度模型，CMMI更适合于信息系统集成企业。CMMI是在CMM基础上发展起来的，它继承并发扬了CMM的优良特性，借鉴了其他模型的优点，融入了新的理论和实际研究成果。它不仅能够应用在软件工程领域，而且可以用于系统工程及其他工程领域。

#### [#](#cmm与iso9000的主要区别) CMM与ISO9000的主要区别？

1.CMM是专门针对软件产品开发和服务的，而ISO9000涉及的范围则相当宽。

2.CMM强调软件开发过程的成熟度，即过程的不断改进和提高。而ISO9000则强调可接收的质量体系的最低标准。

### [#](#_16-5-等级保护) 16.5 等级保护

#### [#](#为什么是做等级保护) 为什么是做等级保护？

1. **法律法规要求**

《网络安全法》明确规定信息系统运营、使用单位应当按照网络安全等级保护制度要求，履行安全保护义务，如果拒不履行，将会受到相应处罚。

第二十一条：国家实行网络安全等级保护制度。网络运营者应当按照网络安全等级保护制度的要求，履行下列安全保护义务，保障网络免受干扰、破坏或者未经授权的访问，防止网络数据泄露或者被窃取、篡改。

1. **行业要求**

在金融、电力、广电、医疗、教育等行业，主管单位明确要求从业机构的信息系统（APP）要开展等级保护工作。

1. **企业系统安全的需求**

信息系统运营、使用单位通过开展等级保护工作可以发现系统内部的安全隐患与不足之处，可通过安全整改提升系统的安全防护能力，降低被攻击的风险。

简单来说，《网络安全法》一直对网站、信息系统、APP有等级保护要求，中小型企业通常是行业要求才意识到问题。

#### [#](#等级保护分为哪些等级) 等级保护分为哪些等级？

- **第一级 自主保护级**：

（无需备案，对测评周期无要求）此类信息系统受到破坏后，会对公民、法人和其他组织的合法权益造成一般损害，不损害国家安全、社会秩序和公共利益。

- **第二级 指导保护级**:

（公安部门备案，建议两年测评一次）此类信息系统受到破坏后，会对公民、法人和其他组织的合法权益造成严重损害。会对社会秩序、公共利益造成一般损害，不损害国家安全。

- **第三级 监督保护级**：

（公安部门备案，要求每年测评一次）此类信息系统受到破坏后，会对国家安全、社会秩序造成损害，对公共利益造成严重损害，对公民、法人和其他组织的合法权益造成特别严重的损害。

- **第四级 强制保护级**：

（公安部门备案，要求半年一次）此类信息系统受到破坏后，会对国家安全造成严重损害，对社会秩序、公共利益造成特别严重损害。

- **第五级 专控保护级**：

（公安部门备案，依据特殊安全需求进行）此类信息系统受到破坏后会对国家安全造成特别严重损害。

#### [#](#怎么做等级保护) 怎么做等级保护？

- **等级保护通常需要5个步骤**：

1. 定级（企业自主定级-专家评审-主管部门审核-公安机关审核）
2. 备案（企业提交备案材料-公安机关审核-发放备案证明）
3. 测评（等级测评-三级每年测评一次）
4. 建设整改（安全建设-安全整改）
5. 监督检查（公安机关每年监督检查）

- **企业自己如何做等级保护**？

1. 在定级备案的步骤，一级不需要备案仅需企业自主定级。二级、三级是大部分普通企业的信息系统定级。四级、五级普通企业不会涉及，通常是与国家相关（如等保四级-涉及民生的，如铁路、能源、电力等）的重要系统。根据地区不同备案文件修改递交通常需要1个月左右的时间。
2. 定级备案后，寻找本地区测评机构进行等级测评。
3. 根据测评评分(GBT22239-2019信息安全技术网络安全等级保护基本要求。具体分数需要测评后才能给出)对信息系统（APP）进行安全整改，如果企业没有专业的安全团队，需要寻找安全公司进行不同项目的整改。等级保护2.0三级有211项内容，通常企业需要根据自身情况采购安全产品完成整改。
4. 进行安全建设整改后，通过测评。当地公安机关会进行监督检查包含定级备案测评、测评后抽查。

整个流程企业自行做等级保护，顺利的话3-4个月完成，如果不熟悉需要半年甚至更久。

#### [#](#等保三的基本要求) 等保三的基本要求?

说说等级保护三级的技术要求，主要包含五个部门

- **物理安全**

保证物理的安全，比如物理位置，机房的访问安全；涉及访问控制，防火防盗防雷防电磁，保备用电等

- **网络安全**

保证网络层面安全，比如访问控制，安全审计，入侵防范，恶意代码防范，设备防范等。

- **主机安全**

比如，身份鉴别，访问控制，安全审计，剩余信息保护（比如退出时清理信息），安全审计，入侵防范等。

- **应用安全**

比如，数据完整性，数据保密性（加密），数据备份和回复。

### [#](#_16-6-iso27001) 16.6 ISO27001

#### [#](#什么是iso27001) 什么是ISO27001？

信息安全管理体系标准（ISO27001)可有效保护信息资源,保护信息化进程健康、有序、可持续发展。ISO27001是信息安全领域的管理体系标准，类似于质量管理体系认证的 ISO9000标准。当您的组织通过了ISO27001的认证,就相当于通过ISO9000的质量认证一般，表示您的组织信息安全管理已建立了一套科学有效的管理体系作为保障。

#### [#](#iso27001认证流程) ISO27001认证流程?

第一阶段：现状调研

从日常运维、管理机制、系统配置等方面对贵公司信息安全管理安全现状进行调研，通过培训使贵公司相关人员全面了解信息安全管理的基本知识。

第二阶段：风险评估

对贵公司信息资产进行资产价值、威胁因素、脆弱性分析，从而评估贵公司信息安全风险，选择适当的措施、方法实现管理风险的目的。

第三阶段：管理策划

根据贵公司对信息安全风险的策略，制定相应信息安全整体规划、管理规划、技术规划等，形成完整的信息安全管理系统。

第四阶段：体系实施

ISMS建立起来（体系文件正式发布实施）之后，要通过一定时间的试运行来检验其有效性和稳定性。

第五阶段：认证审核

经过一定时间运行，ISMS达到一个稳定的状态，各项文档和记录已经建立完备，此时，可以提请进行认证。

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/interview/x-interview-2.html