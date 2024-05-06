# Design

## 什么是设计模式

- 设计模式，是一套被反复使用、多数人知晓的、经过分类编目的、代码设计经验的总结。使用设计模式是为了可重用代码、让代码更容易被他人理解、保证代码可靠性、程序的重用性。

### 为什么要学习设计模式

- 看懂源代码：如果你不懂设计模式去看Jdk、Spring、SpringMVC、IO等等等等的源码，你会很迷茫，你会寸步难行
- 看看前辈的代码：你去个公司难道都是新项目让你接手？很有可能是接盘的，前辈的开发难道不用设计模式？
- 编写自己的理想中的好代码：我个人反正是这样的，对于我自己开发的项目我会很认真，我对他比对我女朋友还好，把项目当成自己的儿子一样

### 设计模式分类



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/17172acfe20b2d2d~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



- 创建型模式，共五种：**工厂方法模式、抽象工厂模式**、**单例模式**、建造者模式、**原型模式。**
- 结构型模式，共七种：适配器模式、装饰器模式、代理模式、外观模式、桥接模式、组合模式、享元模式。
- 行为型模式，共十一种：策略模式、模板方法模式、观察者模式、迭代子模式、责任链模式、命令模式、备忘录模式、状态模式、访问者模式、中介者模式、解释器模式。

### 设计模式的六大原则



![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/17172acff10be430~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)



#### 开放封闭原则（Open Close Principle）

- 原则思想：尽量通过扩展软件实体来解决需求变化，而不是通过修改已有的代码来完成变化
- 描述：一个软件产品在生命周期内，都会发生变化，既然变化是一个既定的事实，我们就应该在设计的时候尽量适应这些变化，以提高项目的稳定性和灵活性。
- 优点：单一原则告诉我们，每个类都有自己负责的职责，里氏替换原则不能破坏继承关系的体系。

#### 里氏代换原则（Liskov Substitution Principle）

- 原则思想：使用的基类可以在任何地方使用继承的子类，完美的替换基类。
- 大概意思是：子类可以扩展父类的功能，但不能改变父类原有的功能。子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法，子类中可以增加自己特有的方法。
- 优点：增加程序的健壮性，即使增加了子类，原有的子类还可以继续运行，互不影响。

#### 依赖倒转原则（Dependence Inversion Principle）

- 依赖倒置原则的核心思想是面向接口编程.
- 依赖倒转原则要求我们在程序代码中传递参数时或在关联关系中，尽量引用层次高的抽象层类，
- 这个是开放封闭原则的基础，具体内容是：对接口编程，依赖于抽象而不依赖于具体。

#### 接口隔离原则（Interface Segregation Principle）

- 这个原则的意思是：使用多个隔离的接口，比使用单个接口要好。还是一个降低类之间的耦合度的意思，从这儿我们看出，其实设计模式就是一个软件的设计思想，从大型软件架构出发，为了升级和维护方便。所以上文中多次出现：降低依赖，降低耦合。
- 例如：支付类的接口和订单类的接口，需要把这俩个类别的接口变成俩个隔离的接口

#### 迪米特法则（最少知道原则）（Demeter Principle）

- 原则思想：一个对象应当对其他对象有尽可能少地了解，简称类间解耦
- 大概意思就是一个类尽量减少自己对其他对象的依赖，原则是低耦合，高内聚，只有使各个模块之间的耦合尽量的低，才能提高代码的复用率。
- 优点：低耦合，高内聚。

#### 单一职责原则（Principle of single responsibility）

- 原则思想：一个方法只负责一件事情。
- 描述：单一职责原则很简单，一个方法 一个类只负责一个职责，各个职责的程序改动，不影响其它程序。 这是常识，几乎所有程序员都会遵循这个原则。
- 优点：降低类和类的耦合，提高可读性，增加可维护性和可拓展性，降低可变性的风险。

## 单例模式

### 1.什么是单例

- 保证一个类只有一个实例，并且提供一个访问该全局访问点

### 2.那些地方用到了单例模式

1. 网站的计数器，一般也是采用单例模式实现，否则难以同步。
2. 应用程序的日志应用，一般都是单例模式实现，只有一个实例去操作才好，否则内容不好追加显示。
3. 多线程的线程池的设计一般也是采用单例模式，因为线程池要方便对池中的线程进行控制
4. Windows的（任务管理器）就是很典型的单例模式，他不能打开俩个
5. windows的（回收站）也是典型的单例应用。在整个系统运行过程中，回收站只维护一个实例。

### 3.单例优缺点

**优点：**

1. 在单例模式中，活动的单例只有一个实例，对单例类的所有实例化得到的都是相同的一个实例。这样就防止其它对象对自己的实例化，确保所有的对象都访问一个实例
2. 单例模式具有一定的伸缩性，类自己来控制实例化进程，类就在改变实例化进程上有相应的伸缩性。
3. 提供了对唯一实例的受控访问。
4. 由于在系统内存中只存在一个对象，因此可以节约系统资源，当需要频繁创建和销毁的对象时单例模式无疑可以提高系统的性能。
5. 允许可变数目的实例。
6. 避免对共享资源的多重占用。

**缺点：**

1. 不适用于变化的对象，如果同一类型的对象总是要在不同的用例场景发生变化，单例就会引起数据的错误，不能保存彼此的状态。
2. 由于单利模式中没有抽象层，因此单例类的扩展有很大的困难。
3. 单例类的职责过重，在一定程度上违背了“单一职责原则”。
4. 滥用单例将带来一些负面问题，如为了节省资源将数据库连接池对象设计为的单例类，可能会导致共享连接池对象的程序过多而出现连接池溢出；如果实例化的对象长时间不被利用，系统会认为是垃圾而被回收，这将导致对象状态的丢失。

### 4.单例模式使用注意事项：

1. 使用时不能用反射模式创建单例，否则会实例化一个新的对象
2. 使用懒单例模式时注意线程安全问题
3. 饿单例模式和懒单例模式构造方法都是私有的，因而是不能被继承的，有些单例模式可以被继承（如登记式模式）

### 5.单例防止反射漏洞攻击

```arduino
复制代码private static boolean flag = false;

private Singleton() {

	if (flag == false) {
		flag = !flag;
	} else {
		throw new RuntimeException("单例模式被侵犯！");
	}
}

public static void main(String[] args) {

}
```

### 6.如何选择单例创建方式

- 如果不需要延迟加载单例，可以使用枚举或者饿汉式，相对来说枚举性好于饿汉式。 如果需要延迟加载，可以使用静态内部类或者懒汉式，相对来说静态内部类好于懒韩式。 最好使用饿汉式

### 7.单例创建方式

**（主要使用懒汉和懒汉式）**

1. 饿汉式:类初始化时,会立即加载该对象，线程天生安全,调用效率高。
2. 懒汉式: 类初始化时,不会初始化该对象,真正需要使用的时候才会创建该对象,具备懒加载功能。
3. 静态内部方式:结合了懒汉式和饿汉式各自的优点，真正需要对象的时候才会加载，加载类是线程安全的。
4. 枚举单例: 使用枚举实现单例模式 优点:实现简单、调用效率高，枚举本身就是单例，由jvm从根本上提供保障!避免通过反射和反序列化的漏洞， 缺点没有延迟加载。
5. 双重检测锁方式 (因为JVM本质重排序的原因，可能会初始化多次，不推荐使用)

#### 1.饿汉式

1. 饿汉式:类初始化时,会立即加载该对象，线程天生安全,调用效率高。

```csharp
复制代码package com.lijie;

//饿汉式
public class Demo1 {

    // 类初始化时,会立即加载该对象，线程安全,调用效率高
    private static Demo1 demo1 = new Demo1();

    private Demo1() {
        System.out.println("私有Demo1构造参数初始化");
    }

    public static Demo1 getInstance() {
        return demo1;
    }

    public static void main(String[] args) {
        Demo1 s1 = Demo1.getInstance();
        Demo1 s2 = Demo1.getInstance();
        System.out.println(s1 == s2);
    }
}
```

#### 2.懒汉式

1. 懒汉式: 类初始化时,不会初始化该对象,真正需要使用的时候才会创建该对象,具备懒加载功能。

```csharp
复制代码package com.lijie;

//懒汉式
public class Demo2 {

    //类初始化时，不会初始化该对象，真正需要使用的时候才会创建该对象。
    private static Demo2 demo2;

    private Demo2() {
        System.out.println("私有Demo2构造参数初始化");
    }

    public synchronized static Demo2 getInstance() {
        if (demo2 == null) {
            demo2 = new Demo2();
        }
        return demo2;
    }

    public static void main(String[] args) {
        Demo2 s1 = Demo2.getInstance();
        Demo2 s2 = Demo2.getInstance();
        System.out.println(s1 == s2);
    }
}
```

#### 3.静态内部类

1. 静态内部方式:结合了懒汉式和饿汉式各自的优点，真正需要对象的时候才会加载，加载类是线程安全的。

```java
复制代码package com.lijie;

// 静态内部类方式
public class Demo3 {

    private Demo3() {
        System.out.println("私有Demo3构造参数初始化");
    }

    public static class SingletonClassInstance {
        private static final Demo3 DEMO_3 = new Demo3();
    }

    // 方法没有同步
    public static Demo3 getInstance() {
        return SingletonClassInstance.DEMO_3;
    }

    public static void main(String[] args) {
        Demo3 s1 = Demo3.getInstance();
        Demo3 s2 = Demo3.getInstance();
        System.out.println(s1 == s2);
    }
}
```

#### 4.枚举单例式

1. 枚举单例: 使用枚举实现单例模式 优点:实现简单、调用效率高，枚举本身就是单例，由jvm从根本上提供保障!避免通过反射和反序列化的漏洞， 缺点没有延迟加载。

```csharp
复制代码package com.lijie;

//使用枚举实现单例模式 优点:实现简单、枚举本身就是单例，由jvm从根本上提供保障!避免通过反射和反序列化的漏洞 缺点没有延迟加载
public class Demo4 {

    public static Demo4 getInstance() {
        return Demo.INSTANCE.getInstance();
    }

    public static void main(String[] args) {
        Demo4 s1 = Demo4.getInstance();
        Demo4 s2 = Demo4.getInstance();
        System.out.println(s1 == s2);
    }

    //定义枚举
	private static enum Demo {
		INSTANCE;
		// 枚举元素为单例
		private Demo4 demo4;

		private Demo() {
			System.out.println("枚举Demo私有构造参数");
			demo4 = new Demo4();
		}

		public Demo4 getInstance() {
			return demo4;
		}
	}
}
```

#### 5.双重检测锁方式

1. 双重检测锁方式 (因为JVM本质重排序的原因，可能会初始化多次，不推荐使用)

```csharp
复制代码package com.lijie;

//双重检测锁方式
public class Demo5 {

	private static Demo5 demo5;

	private Demo5() {
		System.out.println("私有Demo4构造参数初始化");
	}

	public static Demo5 getInstance() {
		if (demo5 == null) {
			synchronized (Demo5.class) {
				if (demo5 == null) {
					demo5 = new Demo5();
				}
			}
		}
		return demo5;
	}

	public static void main(String[] args) {
		Demo5 s1 = Demo5.getInstance();
		Demo5 s2 = Demo5.getInstance();
		System.out.println(s1 == s2);
	}
}
```

## 工厂模式

### 1.什么是工厂模式

- 它提供了一种创建对象的最佳方式。在工厂模式中，我们在创建对象时不会对客户端暴露创建逻辑，并且是通过使用一个共同的接口来指向新创建的对象。实现了创建者和调用者分离，工厂模式分为简单工厂、工厂方法、抽象工厂模式

### 2.工厂模式好处

- 工厂模式是我们最常用的实例化对象模式了，是用工厂方法代替new操作的一种模式。
- 利用工厂模式可以降低程序的耦合性，为后期的维护修改提供了很大的便利。
- 将选择实现类、创建对象统一管理和控制。从而将调用者跟我们的实现类解耦。

### 3.为什么要学习工厂设计模式

- 不知道你们面试题问到过源码没有，你知道Spring的源码吗，MyBatis的源码吗，等等等 如果你想学习很多框架的源码，或者你想自己开发自己的框架，就必须先掌握设计模式（工厂设计模式用的是非常非常广泛的）

### 4.Spring开发中的工厂设计模式

**1.Spring IOC**

- 看过Spring源码就知道，在Spring IOC容器创建bean的过程是使用了工厂设计模式
- Spring中无论是通过xml配置还是通过配置类还是注解进行创建bean，大部分都是通过简单工厂来进行创建的。
- 当容器拿到了beanName和class类型后，动态的通过反射创建具体的某个对象，最后将创建的对象放到Map中。

**2.为什么Spring IOC要使用工厂设计模式创建Bean呢**

- 在实际开发中，如果我们A对象调用B，B调用C，C调用D的话我们程序的耦合性就会变高。（耦合大致分为类与类之间的依赖，方法与方法之间的依赖。）
- 在很久以前的三层架构编程时，都是控制层调用业务层，业务层调用数据访问层时，都是是直接new对象，耦合性大大提升，代码重复量很高，对象满天飞
- 为了避免这种情况，Spring使用工厂模式编程，写一个工厂，由工厂创建Bean，以后我们如果要对象就直接管工厂要就可以，剩下的事情不归我们管了。Spring IOC容器的工厂中有个静态的Map集合，是为了让工厂符合单例设计模式，即每个对象只生产一次，生产出对象后就存入到Map集合中，保证了实例不会重复影响程序效率。

### 5.工厂模式分类

- 工厂模式分为简单工厂、工厂方法、抽象工厂模式

```
复制代码简单工厂 ：用来生产同一等级结构中的任意产品。（不支持拓展增加产品）
工厂方法 ：用来生产同一等级结构中的固定产品。（支持拓展增加产品）   
抽象工厂 ：用来生产不同产品族的全部产品。（不支持拓展增加产品；支持增加产品族）
我下面来使用代码演示一下：
```

#### 5.1 简单工厂模式

**什么是简单工厂模式**

- 简单工厂模式相当于是一个工厂中有各种产品，创建在一个类中，客户无需知道具体产品的名称，只需要知道产品类所对应的参数即可。但是工厂的职责过重，而且当类型过多时不利于系统的扩展维护。

**代码演示：**

1. 创建工厂

```aspectj
复制代码package com.lijie;

public interface Car {
	public void run();
}
```

1. 创建工厂的产品（宝马）

```angelscript
复制代码package com.lijie;

public class Bmw implements Car {
	public void run() {
		System.out.println("我是宝马汽车...");
	}
}
```

1. 创建工另外一种产品（奥迪）

```angelscript
复制代码package com.lijie;

public class AoDi implements Car {
	public void run() {
		System.out.println("我是奥迪汽车..");
	}
}
```

1. 创建核心工厂类，由他决定具体调用哪产品

```haxe
复制代码package com.lijie;

public class CarFactory {

	 public static Car createCar(String name) {
		if ("".equals(name)) {
             return null;
		}
		if(name.equals("奥迪")){
			return new AoDi();
		}
		if(name.equals("宝马")){
			return new Bmw();
		}
		return null;
	}
}
```

1. 演示创建工厂的具体实例

```java
复制代码package com.lijie;

public class Client01 {

	public static void main(String[] args) {
		Car aodi  =CarFactory.createCar("奥迪");
		Car bmw  =CarFactory.createCar("宝马");
		aodi.run();
		bmw.run();
	}
}
```

**单工厂的优点/缺点**

- 优点：简单工厂模式能够根据外界给定的信息，决定究竟应该创建哪个具体类的对象。明确区分了各自的职责和权力，有利于整个软件体系结构的优化。
- 缺点：很明显工厂类集中了所有实例的创建逻辑，容易违反GRASPR的高内聚的责任分配原则

#### 5.2 工厂方法模式

**什么是工厂方法模式**

- 工厂方法模式Factory Method，又称多态性工厂模式。在工厂方法模式中，核心的工厂类不再负责所有的产品的创建，而是将具体创建的工作交给子类去做。该核心类成为一个抽象工厂角色，仅负责给出具体工厂子类必须实现的接口，而不接触哪一个产品类应当被实例化这种细节

**代码演示：**

1. 创建工厂

```aspectj
复制代码package com.lijie;

public interface Car {
	public void run();
}
```

1. 创建工厂方法调用接口（所有的产品需要new出来必须继承他来实现方法）

```aspectj
复制代码package com.lijie;

public interface CarFactory {

	Car createCar();

}
```

1. 创建工厂的产品（奥迪）

```angelscript
复制代码package com.lijie;

public class AoDi implements Car {
	public void run() {
		System.out.println("我是奥迪汽车..");
	}
}
```

1. 创建工厂另外一种产品（宝马）

```angelscript
复制代码package com.lijie;

public class Bmw implements Car {
	public void run() {
		System.out.println("我是宝马汽车...");
	}
}
```

1. 创建工厂方法调用接口的实例（奥迪）

```haxe
复制代码package com.lijie;

public class AoDiFactory implements CarFactory {

	public Car createCar() {
	
		return new AoDi();
	}
}
```

1. 创建工厂方法调用接口的实例（宝马）

```haxe
复制代码package com.lijie;

public class BmwFactory implements CarFactory {

	public Car createCar() {

		return new Bmw();
	}

}
```

1. 演示创建工厂的具体实例

```haxe
复制代码package com.lijie;

public class Client {

	public static void main(String[] args) {
		Car aodi = new AoDiFactory().createCar();
		Car jili = new BmwFactory().createCar();
		aodi.run();
		jili.run();
	}
}
```

#### 5.3 抽象工厂模式

**什么是抽象工厂模式**

- 抽象工厂简单地说是工厂的工厂，抽象工厂可以创建具体工厂，由具体工厂来产生具体产品。

  ![在这里插入图片描述](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2020/4/13/17172acff9e664a6~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

  代码演示：

1. 创建第一个子工厂，及实现类

```csharp
复制代码package com.lijie;

//汽车
public interface Car {
	   void run();
}

 class CarA implements Car{

	public void run() {
		System.out.println("宝马");
	}
	
}
 class CarB implements Car{

	public void run() {
		System.out.println("摩拜");
	}
	
}
```

1. 创建第二个子工厂，及实现类

```csharp
复制代码package com.lijie;

//发动机
public interface Engine {

    void run();

}

class EngineA implements Engine {

    public void run() {
        System.out.println("转的快!");
    }

}

class EngineB implements Engine {

    public void run() {
        System.out.println("转的慢!");
    }

}
```

1. 创建一个总工厂，及实现类（由总工厂的实现类决定调用那个工厂的那个实例）

```csharp
复制代码package com.lijie;

public interface TotalFactory {
	// 创建汽车
	Car createChair();
	// 创建发动机
	Engine createEngine();
}

//总工厂实现类，由他决定调用哪个工厂的那个实例
class TotalFactoryReally implements TotalFactory {

	public Engine createEngine() {

		return new EngineA();
	}

	public Car createChair() {

		return new CarA();
	}
}
```

1. 运行测试

```java
复制代码package com.lijie;

public class Test {

    public static void main(String[] args) {
        TotalFactory totalFactory2 = new TotalFactoryReally();
        Car car = totalFactory2.createChair();
        car.run();

        TotalFactory totalFactory = new TotalFactoryReally();
        Engine engine = totalFactory.createEngine();
        engine.run();
    }
}
```

## 代理模式

### 1.什么是代理模式

- 通过代理控制对象的访问，可以在这个对象调用方法之前、调用方法之后去处理/添加新的功能。(也就是AO的P微实现)
- 代理在原有代码乃至原业务流程都不修改的情况下，直接在业务流程中切入新代码，增加新功能，这也和Spring的（面向切面编程）很相似

### 2.代理模式应用场景

- Spring AOP、日志打印、异常处理、事务控制、权限控制等

### 3.代理的分类

- 静态代理(静态定义代理类)
- 动态代理(动态生成代理类，也称为Jdk自带动态代理)
- Cglib 、javaassist（字节码操作库）

### 4.三种代理的区别

1. 静态代理：简单代理模式，是动态代理的理论基础。常见使用在代理模式
2. jdk动态代理：使用反射完成代理。需要有顶层接口才能使用，常见是mybatis的mapper文件是代理。
3. cglib动态代理：也是使用反射完成代理，可以直接代理类（jdk动态代理不行），使用字节码技术，不能对 final类进行继承。（需要导入jar包）

### 5.用代码演示三种代理

#### 5.1.静态代理

**什么是静态代理**

- 由程序员创建或工具生成代理类的源码，再编译代理类。所谓静态也就是在程序运行前就已经存在代理类的字节码文件，代理类和委托类的关系在运行前就确定了。

**代码演示：**

- 我有一段这样的代码：（如何能在不修改UserDao接口类的情况下开事务和关闭事务呢）

```csharp
复制代码package com.lijie;

//接口类
public class UserDao{
	public void save() {
		System.out.println("保存数据方法");
	}
}
复制代码package com.lijie;

//运行测试类
public  class Test{
	public static void main(String[] args) {
		UserDao userDao = new UserDao();
		userDao.save();
	}
}
```

**修改代码，添加代理类**

```scala
复制代码package com.lijie;

//代理类
public class UserDaoProxy extends UserDao {
	private UserDao userDao;

	public UserDaoProxy(UserDao userDao) {
		this.userDao = userDao;
	}

	public void save() {
		System.out.println("开启事物...");
		userDao.save();
		System.out.println("关闭事物...");
	}

}
复制代码//添加完静态代理的测试类
public class Test{
	public static void main(String[] args) {
		UserDao userDao = new UserDao();
		UserDaoProxy userDaoProxy = new UserDaoProxy(userDao);
		userDaoProxy.save();
	}
}
```

- 缺点：每个需要代理的对象都需要自己重复编写代理，很不舒服，
- 优点：但是可以面相实际对象或者是接口的方式实现代理

#### 2.2.动态代理

**什么是动态代理**

- 动态代理也叫做，JDK代理、接口代理。
- 动态代理的对象，是利用JDK的API，动态的在内存中构建代理对象（是根据被代理的接口来动态生成代理类的class文件，并加载运行的过程），这就叫动态代理

```aspectj
复制代码package com.lijie;

//接口
public interface UserDao {
    void save();
}
复制代码package com.lijie;

//接口实现类
public class UserDaoImpl implements UserDao {
	public void save() {
		System.out.println("保存数据方法");
	}
}
```

- //下面是代理类，可重复使用，不像静态代理那样要自己重复编写代理

```aspectj
复制代码package com.lijie;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;

// 每次生成动态代理类对象时,实现了InvocationHandler接口的调用处理器对象
public class InvocationHandlerImpl implements InvocationHandler {

	// 这其实业务实现类对象，用来调用具体的业务方法
    private Object target;

    // 通过构造函数传入目标对象
    public InvocationHandlerImpl(Object target) {
        this.target = target;
    }

    //动态代理实际运行的代理方法
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("调用开始处理");
        //下面invoke()方法是以反射的方式来创建对象，第一个参数是要创建的对象，第二个是构成方法的参数，由第二个参数来决定创建对象使用哪个构造方法
		Object result = method.invoke(target, args);
        System.out.println("调用结束处理");
        return result;
    }
}
```

- //利用动态代理使用代理方法

```haxe
复制代码package com.lijie;

import java.lang.reflect.Proxy;

public class Test {
    public static void main(String[] args) {
        // 被代理对象
        UserDao userDaoImpl = new UserDaoImpl();
        InvocationHandlerImpl invocationHandlerImpl = new InvocationHandlerImpl(userDaoImpl);

        //类加载器
        ClassLoader loader = userDaoImpl.getClass().getClassLoader();
        Class<?>[] interfaces = userDaoImpl.getClass().getInterfaces();

        // 主要装载器、一组接口及调用处理动态代理实例
        UserDao newProxyInstance = (UserDao) Proxy.newProxyInstance(loader, interfaces, invocationHandlerImpl);
        newProxyInstance.save();
    }
}
```

- 缺点：必须是面向接口，目标业务类必须实现接口
- 优点：不用关心代理类，只需要在运行阶段才指定代理哪一个对象

#### 5.3.CGLIB动态代理

**CGLIB动态代理原理：**

- 利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。

**什么是CGLIB动态代理**

- CGLIB动态代理和jdk代理一样，使用反射完成代理，不同的是他可以直接代理类（jdk动态代理不行，他必须目标业务类必须实现接口），CGLIB动态代理底层使用字节码技术，CGLIB动态代理不能对 final类进行继承。（CGLIB动态代理需要导入jar包）

**代码演示：**

```aspectj
复制代码package com.lijie;

//接口
public interface UserDao {
    void save();
}
复制代码package com.lijie;

//接口实现类
public class UserDaoImpl implements UserDao {
	public void save() {
		System.out.println("保存数据方法");
	}
}
复制代码package com.lijie;

import org.springframework.cglib.proxy.Enhancer;
import org.springframework.cglib.proxy.MethodInterceptor;
import org.springframework.cglib.proxy.MethodProxy;
import java.lang.reflect.Method;

//代理主要类
public class CglibProxy implements MethodInterceptor {
	private Object targetObject;
	// 这里的目标类型为Object，则可以接受任意一种参数作为被代理类，实现了动态代理
	public Object getInstance(Object target) {
		// 设置需要创建子类的类
		this.targetObject = target;
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(target.getClass());
		enhancer.setCallback(this);
		return enhancer.create();
	}

	//代理实际方法
	public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
		System.out.println("开启事物");
		Object result = proxy.invoke(targetObject, args);
		System.out.println("关闭事物");
		// 返回代理对象
		return result;
	}
}
复制代码package com.lijie;

//测试CGLIB动态代理
public class Test {
    public static void main(String[] args) {
        CglibProxy cglibProxy = new CglibProxy();
        UserDao userDao = (UserDao) cglibProxy.getInstance(new UserDaoImpl());
        userDao.save();
    }
}
```

## 建造者模式

### 1.什么是建造者模式

- 建造者模式：是将一个复杂的对象的构建与它的表示分离，使得同样的构建过程可以创建不同的方式进行创建。
- 工厂类模式是提供的是创建单个类的产品
- 而建造者模式则是将各种产品集中起来进行管理，用来具有不同的属性的产品

**建造者模式通常包括下面几个角色：**

1. uilder：给出一个抽象接口，以规范产品对象的各个组成成分的建造。这个接口规定要实现复杂对象的哪些部分的创建，并不涉及具体的对象部件的创建。
2. ConcreteBuilder：实现Builder接口，针对不同的商业逻辑，具体化复杂对象的各部分的创建。 在建造过程完成后，提供产品的实例。
3. Director：调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建。
4. Product：要创建的复杂对象。

### 2.建造者模式的使用场景

**使用场景：**

1. 需要生成的对象具有复杂的内部结构。
2. 需要生成的对象内部属性本身相互依赖。

- 与工厂模式的区别是：建造者模式更加关注与零件装配的顺序。
- JAVA 中的 StringBuilder就是建造者模式创建的，他把一个单个字符的char数组组合起来
- Spring不是建造者模式，它提供的操作应该是对于字符串本身的一些操作，而不是创建或改变一个字符串。

### 3.代码案例

1. 建立一个装备对象Arms

```haxe
复制代码package com.lijie;

//装备类
public class Arms {
	//头盔
	private String helmet;
	//铠甲
	private String armor;
	//武器
	private String weapon;
	
	//省略Git和Set方法...........
}
```

1. 创建Builder接口（给出一个抽象接口，以规范产品对象的各个组成成分的建造，这个接口只是规范）

```csharp
复制代码package com.lijie;

public interface PersonBuilder {

	void builderHelmetMurder();

	void builderArmorMurder();

	void builderWeaponMurder();

	void builderHelmetYanLong();

	void builderArmorYanLong();

	void builderWeaponYanLong();

	Arms BuilderArms(); //组装
}
```

1. 创建Builder实现类（这个类主要实现复杂对象创建的哪些部分需要什么属性）

```csharp
复制代码package com.lijie;

public class ArmsBuilder implements PersonBuilder {
    private Arms arms;

    //创建一个Arms实例,用于调用set方法
    public ArmsBuilder() {
        arms = new Arms();
    }

    public void builderHelmetMurder() {
        arms.setHelmet("夺命头盔");
    }

    public void builderArmorMurder() {
        arms.setArmor("夺命铠甲");
    }

    public void builderWeaponMurder() {
        arms.setWeapon("夺命宝刀");
    }

    public void builderHelmetYanLong() {
        arms.setHelmet("炎龙头盔");
    }

    public void builderArmorYanLong() {
        arms.setArmor("炎龙铠甲");
    }

    public void builderWeaponYanLong() {
        arms.setWeapon("炎龙宝刀");
    }

    public Arms BuilderArms() {
        return arms;
    }
}
```

1. Director（调用具体建造者来创建复杂对象的各个部分，在指导者中不涉及具体产品的信息，只负责保证对象各部分完整创建或按某种顺序创建）

```reasonml
复制代码package com.lijie;

public class PersonDirector {
	
	//组装
	public Arms constructPerson(PersonBuilder pb) {
		pb.builderHelmetYanLong();
		pb.builderArmorMurder();
		pb.builderWeaponMurder();
		return pb.BuilderArms();
	}

	//这里进行测试
	public static void main(String[] args) {
		PersonDirector pb = new PersonDirector();
		Arms arms = pb.constructPerson(new ArmsBuilder());
		System.out.println(arms.getHelmet());
		System.out.println(arms.getArmor());
		System.out.println(arms.getWeapon());
	}
}
```

## 模板方法模式

### 1.什么是模板方法

- 模板方法模式：定义一个操作中的算法骨架（父类），而将一些步骤延迟到子类中。 模板方法使得子类可以不改变一个算法的结构来重定义该算法的

### 2.什么时候使用模板方法

- 实现一些操作时，整体步骤很固定，但是呢。就是其中一小部分需要改变，这时候可以使用模板方法模式，将容易变的部分抽象出来，供子类实现。

### 3.实际开发中应用场景哪里用到了模板方法

- 其实很多框架中都有用到了模板方法模式
- 例如：数据库访问的封装、Junit单元测试、servlet中关于doGet/doPost方法的调用等等

### 4.现实生活中的模板方法

**例如：**

1. 去餐厅吃饭，餐厅给我们提供了一个模板就是：看菜单，点菜，吃饭，付款，走人 （这里 “**点菜和付款**” 是不确定的由子类来完成的，其他的则是一个模板。）

### 5.代码实现模板方法模式

1. 先定义一个模板。把模板中的点菜和付款，让子类来实现。

```csharp
复制代码package com.lijie;

//模板方法
public abstract class RestaurantTemplate {

	// 1.看菜单
	public void menu() {
		System.out.println("看菜单");
	}

	// 2.点菜业务
	abstract void spotMenu();

	// 3.吃饭业务
	public void havingDinner(){ System.out.println("吃饭"); }

	// 3.付款业务
	abstract void payment();

	// 3.走人
	public void GoR() { System.out.println("走人"); }

	//模板通用结构
	public void process(){
		menu();
		spotMenu();
		havingDinner();
		payment();
		GoR();
	}
}
```

1. 具体的模板方法子类 1

```scala
复制代码package com.lijie;

public class RestaurantGinsengImpl extends RestaurantTemplate {

    void spotMenu() {
        System.out.println("人参");
    }

    void payment() {
        System.out.println("5快");
    }
}
```

1. 具体的模板方法子类 2

```scala
复制代码package com.lijie;

public class RestaurantLobsterImpl  extends RestaurantTemplate  {

    void spotMenu() {
        System.out.println("龙虾");
    }

    void payment() {
        System.out.println("50块");
    }
}
```

1. 客户端测试

```haxe
复制代码package com.lijie;

public class Client {

    public static void main(String[] args) {
        //调用第一个模板实例
        RestaurantTemplate restaurantTemplate = new RestaurantGinsengImpl();
        restaurantTemplate.process();
    }
}
```

## 外观模式

### 1.什么是外观模式

- 外观模式：也叫门面模式，隐藏系统的复杂性，并向客户端提供了一个客户端可以访问系统的接口。
- 它向现有的系统添加一个接口，用这一个接口来隐藏实际的系统的复杂性。
- 使用外观模式，他外部看起来就是一个接口，其实他的内部有很多复杂的接口已经被实现

### 2.外观模式例子

- 用户注册完之后，需要调用阿里短信接口、邮件接口、微信推送接口。

1. 创建阿里短信接口

```aspectj
复制代码package com.lijie;

//阿里短信消息
public interface AliSmsService {
	void sendSms();
}
复制代码package com.lijie;

public class AliSmsServiceImpl implements AliSmsService {

    public void sendSms() {
        System.out.println("阿里短信消息");
    }

}
```

1. 创建邮件接口

```aspectj
复制代码package com.lijie;

//发送邮件消息
public interface EamilSmsService {
	void sendSms();
}
复制代码package com.lijie;

public class EamilSmsServiceImpl implements   EamilSmsService{
	public void sendSms() {
		System.out.println("发送邮件消息");
	}
}
```

1. 创建微信推送接口

```aspectj
复制代码package com.lijie;

//微信消息推送
public interface WeiXinSmsService {
   void sendSms();
}
复制代码package com.lijie;

public class WeiXinSmsServiceImpl implements  WeiXinSmsService {
    public void sendSms() {
        System.out.println("发送微信消息推送");
    }
}
```

1. 创建门面（门面看起来很简单使用，复杂的东西以及被门面给封装好了）

```abnf
复制代码package com.lijie;

public class Computer {
	AliSmsService aliSmsService;
	EamilSmsService eamilSmsService;
	WeiXinSmsService weiXinSmsService;

	public Computer() {
		aliSmsService = new AliSmsServiceImpl();
		eamilSmsService = new EamilSmsServiceImpl();
		weiXinSmsService = new WeiXinSmsServiceImpl();
	}

	//只需要调用它
	public void sendMsg() {
		aliSmsService.sendSms();
		eamilSmsService.sendSms();
		weiXinSmsService.sendSms();
	}
}
```

1. 启动测试

```haxe
复制代码package com.lijie;

public class Client {

    public static void main(String[] args) {
        //普通模式需要这样
        AliSmsService aliSmsService = new AliSmsServiceImpl();
        EamilSmsService eamilSmsService = new EamilSmsServiceImpl();
        WeiXinSmsService weiXinSmsService = new WeiXinSmsServiceImpl();
        aliSmsService.sendSms();
        eamilSmsService.sendSms();
        weiXinSmsService.sendSms();

        //利用外观模式简化方法
        new Computer().sendMsg();
    }
}
```

## 原型模式

### 1.什么是原型模式

- 原型设计模式简单来说就是克隆
- 原型表明了有一个样板实例，这个原型是可定制的。原型模式多用于创建复杂的或者构造耗时的实例，因为这种情况下，复制一个已经存在的实例可使程序运行更高效。

### 2.原型模式的应用场景

1. 类初始化需要消化非常多的资源，这个资源包括数据、硬件资源等。这时我们就可以通过原型拷贝避免这些消耗。
2. 通过new产生的一个对象需要非常繁琐的数据准备或者权限，这时可以使用原型模式。
3. 一个对象需要提供给其他对象访问，而且各个调用者可能都需要修改其值时，可以考虑使用原型模式拷贝多个对象供调用者使用，即保护性拷贝。

```
我们Spring框架中的多例就是使用原型。
```

### 3.原型模式的使用方式

1. 实现Cloneable接口。在java语言有一个Cloneable接口，它的作用只有一个，就是在运行时通知虚拟机可以安全地在实现了此接口的类上使用clone方法。在java虚拟机中，只有实现了这个接口的类才可以被拷贝，否则在运行时会抛出CloneNotSupportedException异常。
2. 重写Object类中的clone方法。Java中，所有类的父类都是Object类，Object类中有一个clone方法，作用是返回对象的一个拷贝，但是其作用域protected类型的，一般的类无法调用，因此Prototype类需要将clone方法的作用域修改为public类型。

#### 3.1原型模式分为浅复制和深复制

1. （浅复制）只是拷贝了基本类型的数据，而引用类型数据，只是拷贝了一份引用地址。
2. （深复制）在计算机中开辟了一块新的内存地址用于存放复制的对象。

### 4.代码演示

1. 创建User类

```haxe
复制代码package com.lijie;

import java.util.ArrayList;

public class User implements Cloneable {
    private String name;
    private String password;
    private ArrayList<String> phones;

    protected User clone() {
        try {
            User user = (User) super.clone();
            //重点，如果要连带引用类型一起复制，需要添加底下一条代码，如果不加就对于是复制了引用地址
            user.phones = (ArrayList<String>) this.phones.clone();//设置深复制
            return user;
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return null;
    }
	
	//省略所有属性Git Set方法......
}
```

1. 测试复制

```pgsql
复制代码package com.lijie;

import java.util.ArrayList;

public class Client {
    public static void main(String[] args) {
        //创建User原型对象
        User user = new User();
        user.setName("李三");
        user.setPassword("123456");
        ArrayList<String> phones = new ArrayList<>();
        phones.add("17674553302");
        user.setPhones(phones);

        //copy一个user对象,并且对象的属性
        User user2 = user.clone();
        user2.setPassword("654321");

        //查看俩个对象是否是一个
        System.out.println(user == user2);

        //查看属性内容
        System.out.println(user.getName() + " | " + user2.getName());
        System.out.println(user.getPassword() + " | " + user2.getPassword());
        //查看对于引用类型拷贝
        System.out.println(user.getPhones() == user2.getPhones());
    }
}
```

1. 如果不需要深复制，需要删除User 中的

```processing
复制代码//默认引用类型为浅复制，这是设置了深复制
user.phones = (ArrayList<String>) this.phones.clone();
```

## 策略模式

### 1.什么是策略模式

- 定义了一系列的算法 或 逻辑 或 相同意义的操作，并将每一个算法、逻辑、操作封装起来，而且使它们还可以相互替换。（其实策略模式Java中用的非常非常广泛）
- 我觉得主要是为了 简化 if...else 所带来的复杂和难以维护。

### 2.策略模式应用场景

- 策略模式的用意是针对一组算法或逻辑，将每一个算法或逻辑封装到具有共同接口的独立的类中，从而使得它们之间可以相互替换。

1. 例如：我要做一个不同会员打折力度不同的三种策略，初级会员，中级会员，高级会员（三种不同的计算）。
2. 例如：我要一个支付模块，我要有微信支付、支付宝支付、银联支付等

### 3.策略模式的优点和缺点

- 优点： 1、算法可以自由切换。 2、避免使用多重条件判断。 3、扩展性非常良好。
- 缺点： 1、策略类会增多。 2、所有策略类都需要对外暴露。

### 4.代码演示

- 模拟支付模块有微信支付、支付宝支付、银联支付

1. 定义抽象的公共方法

```aspectj
复制代码package com.lijie;

//策略模式 定义抽象方法 所有支持公共接口
abstract class PayStrategy {

	// 支付逻辑方法
	abstract void algorithmInterface();

}
```

1. 定义实现微信支付

```scala
复制代码package com.lijie;

class PayStrategyA extends PayStrategy {

	void algorithmInterface() {
		System.out.println("微信支付");
	}
}
```

1. 定义实现支付宝支付

```scala
复制代码package com.lijie;

class PayStrategyB extends PayStrategy {

	void algorithmInterface() {
		System.out.println("支付宝支付");
	}
}
```

1. 定义实现银联支付

```scala
复制代码package com.lijie;

class PayStrategyC extends PayStrategy {

	void algorithmInterface() {
		System.out.println("银联支付");
	}
}
```

1. 定义下文维护算法策略

```aspectj
复制代码package com.lijie;// 使用上下文维护算法策略

class Context {

	PayStrategy strategy;

	public Context(PayStrategy strategy) {
		this.strategy = strategy;
	}

	public void algorithmInterface() {
		strategy.algorithmInterface();
	}

}
```

1. 运行测试

```haxe
复制代码package com.lijie;

class ClientTestStrategy {
	public static void main(String[] args) {
		Context context;
		//使用支付逻辑A
		context = new Context(new PayStrategyA());
		context.algorithmInterface();
		//使用支付逻辑B
		context = new Context(new PayStrategyB());
		context.algorithmInterface();
		//使用支付逻辑C
		context = new Context(new PayStrategyC());
		context.algorithmInterface();
	}
}
```

## 观察者模式

### 1.什么是观察者模式

- 先讲什么是行为性模型，行为型模式关注的是系统中对象之间的相互交互，解决系统在运行时对象之间的相互通信和协作，进一步明确对象的职责。
- 观察者模式，是一种行为性模型，又叫发布-订阅模式，他定义对象之间一种一对多的依赖关系，使得当一个对象改变状态，则所有依赖于它的对象都会得到通知并自动更新。

### 2.模式的职责

- 观察者模式主要用于1对N的通知。当一个对象的状态变化时，他需要及时告知一系列对象，令他们做出相应。

**实现有两种方式：**

1. 推：每次都会把通知以广播的方式发送给所有观察者，所有的观察者只能被动接收。
2. 拉：观察者只要知道有情况即可，至于什么时候获取内容，获取什么内容，都可以自主决定。

### 3.观察者模式应用场景

1. 关联行为场景，需要注意的是，关联行为是可拆分的，而不是“组合”关系。事件多级触发场景。
2. 跨系统的消息交换场景，如消息队列、事件总线的处理机制。

### 4.代码实现观察者模式

1. 定义抽象观察者，每一个实现该接口的实现类都是具体观察者。

```aspectj
复制代码package com.lijie;

//观察者的接口，用来存放观察者共有方法
public interface Observer {
    // 观察者方法
    void update(int state);
}
```

1. 定义具体观察者

```aspectj
复制代码package com.lijie;

// 具体观察者
public class ObserverImpl implements Observer {

    // 具体观察者的属性
    private int myState;

    public void update(int state) {
        myState=state;
        System.out.println("收到消息,myState值改为："+state);
    }

    public int getMyState() {
        return myState;
    }
}
```

1. 定义主题。主题定义观察者数组，并实现增、删及通知操作。

```java
复制代码package com.lijie;

import java.util.Vector;

//定义主题，以及定义观察者数组，并实现增、删及通知操作。
public class Subjecct {
	//观察者的存储集合，不推荐ArrayList，线程不安全，
	private Vector<Observer> list = new Vector<>();

	// 注册观察者方法
	public void registerObserver(Observer obs) {
		list.add(obs);
	}
    // 删除观察者方法
	public void removeObserver(Observer obs) {
		list.remove(obs);
	}

	// 通知所有的观察者更新
	public void notifyAllObserver(int state) {
		for (Observer observer : list) {
			observer.update(state);
		}
	}
}
```

1. 定义具体的，他继承继承Subject类，在这里实现具体业务，在具体项目中，该类会有很多。

```pf
复制代码package com.lijie;

//具体主题
public class RealObserver extends Subjecct {
    //被观察对象的属性
	 private int state;
	 public int getState(){
		 return state;
	 }
	 public void  setState(int state){
		 this.state=state;
		 //主题对象(目标对象)值发生改变
		 this.notifyAllObserver(state);
	 }
}
```

1. 运行测试

```reasonml
复制代码package com.lijie;

public class Client {

	public static void main(String[] args) {
		// 目标对象
		RealObserver subject = new RealObserver();
		// 创建多个观察者
		ObserverImpl obs1 = new ObserverImpl();
		ObserverImpl obs2 = new ObserverImpl();
		ObserverImpl obs3 = new ObserverImpl();
		// 注册到观察队列中
		subject.registerObserver(obs1);
		subject.registerObserver(obs2);
		subject.registerObserver(obs3);
		// 改变State状态
		subject.setState(300);
		System.out.println("obs1观察者的MyState状态值为："+obs1.getMyState());
		System.out.println("obs2观察者的MyState状态值为："+obs2.getMyState());
		System.out.println("obs3观察者的MyState状态值为："+obs3.getMyState());
		// 改变State状态
		subject.setState(400);
		System.out.println("obs1观察者的MyState状态值为："+obs1.getMyState());
		System.out.println("obs2观察者的MyState状态值为："+obs2.getMyState());
		System.out.println("obs3观察者的MyState状态值为："+obs3.getMyState());
	}
}
```

## 文章就到这了，没错，没了

察者方法 public void removeObserver(Observer obs) { list.remove(obs); }

```pf
复制代码// 通知所有的观察者更新
public void notifyAllObserver(int state) {
	for (Observer observer : list) {
		observer.update(state);
	}
}
```

}

~~~pf
复制代码4. 定义具体的，他继承继承Subject类，在这里实现具体业务，在具体项目中，该类会有很多。
```java
package com.lijie;

//具体主题
public class RealObserver extends Subjecct {
    //被观察对象的属性
	 private int state;
	 public int getState(){
		 return state;
	 }
	 public void  setState(int state){
		 this.state=state;
		 //主题对象(目标对象)值发生改变
		 this.notifyAllObserver(state);
	 }
}
~~~

1. 运行测试

```reasonml
复制代码package com.lijie;

public class Client {

	public static void main(String[] args) {
		// 目标对象
		RealObserver subject = new RealObserver();
		// 创建多个观察者
		ObserverImpl obs1 = new ObserverImpl();
		ObserverImpl obs2 = new ObserverImpl();
		ObserverImpl obs3 = new ObserverImpl();
		// 注册到观察队列中
		subject.registerObserver(obs1);
		subject.registerObserver(obs2);
		subject.registerObserver(obs3);
		// 改变State状态
		subject.setState(300);
		System.out.println("obs1观察者的MyState状态值为："+obs1.getMyState());
		System.out.println("obs2观察者的MyState状态值为："+obs2.getMyState());
		System.out.println("obs3观察者的MyState状态值为："+obs3.getMyState());
		// 改变State状态
		subject.setState(400);
		System.out.println("obs1观察者的MyState状态值为："+obs1.getMyState());
		System.out.println("obs2观察者的MyState状态值为："+obs2.getMyState());
		System.out.println("obs3观察者的MyState状态值为："+obs3.getMyState());
```



作者：小杰要吃蛋
链接：https://juejin.cn/post/6844904125721772039
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。





# Design  Software

领域驱动设计(DDD)在爱奇艺打赏业务的实践

[飞行的大狗](https://juejin.cn/user/3307791761810670/posts)



2021-03-02 20:01·[高可用架构](https://link.juejin.cn/?target=https%3A%2F%2Fwww.toutiao.com%2Fc%2Fuser%2Ftoken%2FMS4wLjABAAAATJ-zFU1Q7-YExQ2fvoPy2fT9iWRpEc2beEY2J_HL5Xs%2F%3Fsource%3Dtuwen_detail)

领域驱动设计(Domain-Driven Design，以下简称DDD)思潮的形成要追述到30几年前，17年前，Eirc Evans定义了**领域驱动设计**的概念。DDD一直为传统行业的软件工程师提供软件设计的方法论，但是在互联网行业却使用很少。直到近几年，DDD在互联网行业被重新认识，火了起来。究其原因有两点：

- 互联网的行业的**业务越来越复杂**，面临传统行业软件同样的问题；
- **微服务的流行**带火了DDD，来解决微服务拆分问题。

本文主要对第一点“**解决软件复杂性之道**”进行讲解。

### **价值**

详细讲解之前，我们先给出DDD为打赏业务带来的**价值**。

会员业务部门在打赏业务进行了DDD实践后，效率有显著提升：

- 新需求接入开发成本节约**20%** ；
- 更换底层中间件开发成本节约**20%** ；
- 项目熟悉成本节约**30%** (对DDD有基本了解为前提)；
- 单测开发成本指数级**降低**；
- 上线风险、成本**降低**。

了解了DDD流行的背景及业务价值后，下面我们对**DDD是什么、有哪些优势、项目中如何实践**，以及几个关键问题进行叙述。

**领域驱动设计是什么**

## 讨论领域驱动设计是什么

> 之前，我们先看下面一段代码：

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/31f4b4f172ba4527b4a6d2cc55778bae~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

这是一个打赏接口定义，单看这个接口是没有问题的，用户基于活动，选择明星，选择道具进行打赏。

业务逻辑上没问题，但我们会发现一些代码的坏味道。

- 代码编译后方法的参数名会丢失。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/08fefe11075d42debc8653c9531b9b87~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

在编码时如果将明星和道具的参数顺序传错一样可以通过编译，只有运行时才会报业务错误。当然这种问题发现成本不是很高，如果在编译时发现会规避一些风险，降低排查成本。上面方法参数的几个code可以唯一标识出实体，但却丢失了原有实体的实际业务领域意义，为某个明星打赏某种道具。

打赏业务逻辑掺杂了很多与业务核心逻辑无关的前置校验逻辑，影响代码可读性。全部逻辑堆叠在一个方法中，增加了测试用例编写复杂度。

对于以上代码，问题的根本原因是我们对业务的领域没有明确的划分，只是实现了一个操作流程，是需求的直接实现，缺乏领域抽象，方法的参数定义缺少业务领域含义。对于活动校验本质是活动的一些属性判断，活动是否有效是活动自身的属性决定的。可以抽象出活动校验类ActivityValidate，或在实体中增加validate方法，还可以更近一步，将校验逻辑直接放在活动的构造方法中，这样既达到了校验的目的，也避免了漏校验。对于单测的编写，如果我们能将一个大逻辑拆分为多逻辑单元，无疑会大大减少用例数量。优化后的代码如下：

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e7f3c3f7b27740419e7756899e6c47b3~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

对于活动的校验，采用构造函数校验，因此打赏方法中无需再校验活动。活动的校验放到前置构造函数，减少了测试用例数量。

回到最初的问题，什么是领域驱动设计？

#### **1、领域驱动设计基于领域建模而非数据建模**

上面例子中，代码重构前activity实体只有基本的属性和get/set方法，即“失血模型”，从而导致activity这个领域对象退化为数据对象，只用作orm组件的crud，失血模型在项目代码中随处可见。究其原因，跟对象-关系映射(ORM，比如hibernate)持久化机制的流行是有直接关系的，使用ORM将每个类映射到一张数据表，通过实体对象完成crud，久而久之实体成为了orm框架的专用名词，即丧失了领域能力。进行项目设计时，我们应该从业务领域角度出发思考问题，而不应该从数据库角度，我们将在战略设计部分详细讨论。

#### **2、满足六边形架构设计**

六边形架构在后文进行详细介绍，洋葱架构、干净架构与六边形架构类似。

满足以上两点，并对DDD的一些概念进行映射实践，那么你的系统已经符合DDD了。总结，DDD不是一套全新的特殊架构，是任何项目代码经过重构，满足高可维护性、高可扩展性、高可测试性、代码结构清晰之后必将达到的终点。

### **DDD打赏业务实践**

**1、打赏业务简介**

- 观看视频时，选择明星、礼物进行打赏；

- 打赏后屏幕有气泡提示；

- 打赏数据在排行榜进行显示；

- 累计一定的打赏获得某种奖励。

**2、战略设计**

提到战略设计，不得不提的是战略设计的几个核心概念：领域、子域、限界上下文、架构分层。

**领域：** 从广义上讲，领域即是一个组织所做的事情以及其中包含的一切。每个公司或组织都有它自己的业务范围和做事方式，这个业务范围以及其在其中所进行的活动便是领域。当你为某个公司开发软件时，你面对的便是这个公司的领域。这个领域对于你来说应该是明确的，因为你在这个领域中工作。

对于打赏这种业务，打赏本身便是领域，即打赏领域。无论你的打赏对象是一位主播、一部电影或者一篇博文，又无论你的打赏道具是RMB、虚拟币、火箭等等，打赏都是这个领域的核心。

**子域：** 对于领域模型包含“领域”这个词，我们可能会认为整个业务系统创建一个单一的、内聚的、功能全能式的模型。然后这并不是我们使用DDD的目标。正好相反，在DDD中，一个领域被分成若干小的域，这些若干的小的域即子域。事实上，在开发一个领域模型时，我们关注的通常只是这个业务系统的某个方面，对领域的拆分将有助于我们成功。

**限界上下文：** 一个由显示边界限定的特殊职责。领域模型存在于边界之内，在边界内，每一个模型概念，包括它的属性和操作，都具有特殊的含义。

打赏系统搭建之初，需求比较简单，随着业务发展，需求越来越复杂，迭代及领域拆分过程如下：

- 运营及产品需求非常简单，只要实现免费打赏并在界面实现打赏气泡，基于此，只有一个领域；

- 经过一阶段的试水，活动效果很好，需要能同时支持多场打赏活动，增加活动支撑子域；

- 需求方又有了新想法，用户完成一定打赏后给用户发放一些奖品，引入奖励子域；

- 为了提升用户参与感，增加排行功能，引入排行子域。

最终领域划分如下图：

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/253c5c8f0a6b4266878cf27623467887~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

- 打赏核心子域：完成打赏操作。

- 通知子域：实现界面气泡通知能力。

- 奖励子域：奖励策略匹配，奖励发放。

- 排行子域：完成排行功能。

- 活动子域：活动、明星、道具管理。

- 用户子域：完成用户查询、校验等通用能力。

领域的拆分过程并没有上面描述的那么顺利，经历了很多推翻重来的过程，正是经历了这些过程，我们对领域的理解才能更深入，更符合领域建模。

**架构分层：** 分层架构的一个重要原则是——每层只能与位于其下方的层发生聚合。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d518ae04cdb04761b0d7f5760ff4a6ea~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

为了实现接口的定义与实现解耦，接口定义在领域层，实现定义在基础设施层。但这样违背了由顶至底的单项依赖原则。为了解决这个问题，我们此处采用依赖倒置原则,依赖倒置原则内容——**高层模块不应该依赖于低层模块，两者都应该依赖于抽象。抽象不应该依赖于细节，细节应该依赖于抽象。根据此原则，结构调整如下：**

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/37aa719847e74e09ac12917684ff0db4~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

我们将基础设施层放在所有层的最上方，这样它可以实现所有其它层定义的接口。

当我们在分层架构中采用依赖倒置原则时，我们可能会发现，事实上已经不存在分层的概念了。无论是高层还是底层，它们都只依赖于抽象，好像把整个分层推平了一样。推平之后，客户通过“平等”的方式与系统交互。加入新的客户也只是不同的输入、输出，以及不同的展现形式而已，这既是我们即将了解的另一个架构，六边形架构。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b54d8328c95c4bb68a376b17b7d3d008~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

#### 六边形架构

在我们的代码中，有很多直接的外部依赖和实现细节。如mybatis的mapper类、httpclient注入、rocketmq的监听、缓存的直接操作等等。这样的实现有两个比较明显问题，一是当底层更换基础组件时对业务逻辑有直接影响，更换代码改动量及测试范围大大增加。二是不利于功能的复用，如果其他业务有类似逻辑，做不到直接移植复用。

2005年Alistair Cockburn提出了六边形架构，又被称为端口和适配器架构。观察上图我们发现，对于核心的应用程序和领域模型来说，其他的底层依赖或实现都可以抽象为输入和输出两类。组织关系变为了一个二维的内外关系，而不是上下结构。每个io与应用程序之前均有适配器完成隔离工作，每个最外围的边都是一个端口。基于六边形架构设计的系统是DDD追求的最终形态。六边形架构的实践在“DDD的优势”部分进行讲解。

先给出基于六边形架构实践后，项目模块结构：

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d1ce30550f1542e2b9b6d70aa9b7d196~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

| # 模块           | # 说明                       |
| ---------------- | ---------------------------- |
| # Admin-api      | # 配置后台相关               |
| # api            | # 对外用户接口               |
| # application    | # 应用                       |
| # domain         | # 领域                       |
| # infrastructure | # 基础设施                   |
| # query          | # 查询模块                   |
| # task           | # 与使用的中间件相关，可忽略 |
| # worker         | # 处理事件消息模块           |
| # common         | # 基础包                     |

**3、战术设计**

经过战略设计后，领域已经有了清晰的边界，下面我们聊下战术设计。首先对DDD的几个基本概念进行业务映射。

**实体：** 由属性和行为组成，具有唯一标识。

在设计系统时，我们趋向于将重点放在数据上，而不是领域上。对于DDD开发者来说也是如此，因为软件开发中，数据库依然占据着主导地位。首先考虑的是数据的属性和关联关系，而不是富有行为的领域概念。这样做的结果是将数据模型直接反映在对象模型上，导致实体只包含get/set方法，这不是DDD的做法。

只有get/set实体需配合service使用，内聚性、可维护性，以及复用迁移成本均明显高于DDD的做法。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f3c386ed5584594bd6303b96895c66b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6a201452edb74ffca3063e3f22fa370c~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**值对象：** 没有唯一标识，具有可度量或可描述，并满足不变性的对象。

我们应该尽量使用值对象来建模而不是实体对象，因为相比实体，我们可以非常容易地对值对象进行创建、测试、使用、优化和维护。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9429c1ad6534ee8995c2d5484076c1b~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0d2b3b6aa91c4974bb3c09e0a185d36a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

对于第一种实现，用户必须知道需要同时使用amount和currency，并且应该知道如何使用这两个属性，原因在于这两个属性并没有组成一个概念整体。对于PropName值对象的定义，可以带来一些扩展性，如需要对道具名称做大小写转换，此操作可以在PropName的内部实现，对外name的逻辑泄漏到Prop中。

**领域服务：** 领域服务表示一个无状态的操作，它用于实现特定于某个领域的任务。

当某个操作不适合放在聚合和值对象上时，最好的方式便是使用领域服务了。

例如“用户认证”，一种方式是我们可以简单地将认证操作放在实体上。对于这种设计，存在两个问题。首先，用户类需要知道某些认证细节，其次，这种方法也不能显示的表达通用语言。这里我们询问的是一个User“是否被认证了”，而没有表达出“认证”这个过程。在有可能的情况下，我们应该尽量使用建模术语直接地表达出交流语言。

**领域事件：** 领域专家所关心的发生在领域中的一些事件。我们通常将领域事件用于维护事件的一致性，这样可以消除两阶段提交(全局事务)。

**聚合：** 聚合是一组相关对象的组合，作为一个整体被外界访问，聚合根是这个聚合的根节点。

聚合是一个非常重要的概念，核心领域往往都是用聚合来表达。其次，聚合在技术上也有非常高的价值，可以指导详细设计。聚合由根实体、实体、值对象组成。

**工厂：** 工厂提供一个创建对象的接口，该接口封装了所有创建对象的复杂操作过程，同时，它并不需要客户去引用实际被创建的对象。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/0789fdbfad964abf8db0522aab53cb02~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

对于上面例子中方法参入为什么传入资源库对象，我们将在下面六边形架构部分讲解。

**DDD的优势**

**应用DDD的系统符合六边形架构，我们实现了以下目标：**

- 独立于框架：架构不应该依赖某个外部的库或者框架，不应该被框架的结构所束缚；

- 独立于UI：前台展示的样式可能会随时发生变化，但是底层架构不应该随之而变化；

- 独立于底层数据源：软件架构不应该因为不同的底层数据存储而产生巨大改变；

- 独立于外部依赖：无论外部依赖如何变更、升级，业务的核心逻辑不应随之而大幅变化。

实现以上几个目标，六边形架构(洋葱架构、干净架构与此类似)是个不错的选择，下面结合打赏具体业务实现，讲解如何实现以上目标。

先给出项目某个模块的代码包结构：

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/12e90688fe804b47ae303881ccb771c1~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**资源库：** 对于资源库，我们的实践是资源库作为业务与数据的隔离层，屏蔽底层数据表细节，同时完成PO与DO的转化。DO与PO的转化带来的好处是领域层不会直接依赖底层实现，便于后续更换底层实现或功能迁移。资源库接口定义在领域层，接口实现在基础设施层。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4978c01c0c4a4f3a82fbbdfff59b3e63~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b11b11012d754d74b8013fb75d17ca41~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

**RPC：** RPC部分的结构拆分与资源库类似，区别是以领域服务的存在。接口定义放在领域层，具体实现在基础设施层。

![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27257b00aa7d42d394083c5084a9222a~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)![领域驱动设计(DDD)在爱奇艺打赏业务的实践](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efbae78dca95424192c87109083a38c7~tplv-k3u1fbpfcp-zoom-in-crop-mark:1512:0:0:0.awebp)

在遵循六边形架构的大原则下，其他边的拆分也变得清晰简单了，此处不再赘述。

**几个关键问题**

**1、事务**

在上文中“聚合”章节，我们描述了事务，一个事务内只操作一个聚合实例。如果发现一次事务内逻辑过多，可以考虑剥离出独立的聚合，采用最终一致性。基于这个基础，最适合声明事务的层是应用层。

**2、查询**

CQRS 在 DDD 中是一种常常被提及的模式，它的用途在于将领域模型与查询功能进行分离，让一些复杂的查询摆脱领域模型的限制，以更为简单的 DTO 形式展现查询结果。同时分离了不同的数据存储结构，让开发者按照查询的功能与要求更加自由的选择数据存储引擎，CQRS的具体实践可以自行查找资料。

**3、框架无关**

基于六边形架构设计，已经做到了与底层实现、框架、中间件无关。但还有一个最大的框架依赖spring，我们的做法是领域内使用的spring bean通过传参方式，实现领域层框架解耦。

**4、成本**

成本是我们实践DDD时需要考虑的一个很重要的问题，学习成本、改造成本、兼容成本等等都是需要特别关注的。在动手实践之前，建议优先评估好成本。

**结束语**

DDD不是一套全新的特殊架构，是应对软件复杂性的一套方法论。是面向领域建模，基于六边形架构，项目代码经过重构，满足高可维护性、高可扩展性、高可测试性、代码结构清晰之后必将达到的终点。

DDD缺少权威性的实践指导和代码约束，因此应用过程中会碰到很多问题，爱奇艺会员团队通过几个月的实践积累了一定的经验，欢迎交流。



# Develop  principle

- 软件开发中的原则 - SOLID

> 在软件开发中，前人对软件系统的设计和开发总结了一些原则和模式， 不管用什么语言做开发，都将对我们系统设计和开发提供指导意义。本文主要将总结这些常见的原则，和具体阐述意义。@pdai

- 软件开发中的原则 - SOLID
  - 开发原则SOILD
    - S单一职责SRP
      - [定义](#定义)
      - [原则分析](#原则分析)
      - [优点](#优点)
      - [例子](#例子)
    - O开放封闭原则OCP
      - [定义](#定义-1)
      - [原则分析](#原则分析-1)
      - [例子](#例子-1)
    - L里氏替换原则LSP
      - [定义](#定义-2)
      - [原则分析](#原则分析-2)
    - I接口隔离法则
      - [定义](#定义-3)
      - [原则分析:](#原则分析-3)
    - D依赖倒置原则DIP
      - [定义](#定义-4)
      - [原则分析](#原则分析-4)
      - [例子](#例子-2)
  - 其它原则
    - 合成/聚合复用原则
      - [定义](#定义-5)
      - [原则分析:](#原则分析-5)
    - 迪米特法则
      - [定义](#定义-6)
      - [法则分析](#法则分析)
      - [例子](#例子-3)
  - Q&A
    - [面向对象设计其他原则](#面向对象设计其他原则)
    - [你能解释一下里氏替换原则吗?](#你能解释一下里氏替换原则吗)
    - [什么情况下会违反迪米特法则? 为什么会有这个问题?](#什么情况下会违反迪米特法则-为什么会有这个问题)
    - [给我一个符合开闭原则的设计模式的例子?](#给我一个符合开闭原则的设计模式的例子)
    - [什么时候使用享元模式(蝇量模式)?](#什么时候使用享元模式蝇量模式)
  - [参考文章](#参考文章)

## [#](#开发原则soild) 开发原则SOILD

> 面向对象的基本原则(solid)是五个，但是在经常被提到的除了这五个之外还有 `迪米特法则`和`合成复用原则`等， 所以在常见的文章中有表示写六大或七大原则的； 除此之外我还将给出一些其它相关书籍和互联网上出现的原则；

### [#](#s单一职责srp) S单一职责SRP

> Single-Responsibility Principle, 一个类,最好只做一件事,只有一个引起它的变化。单一职责原则可以看做是低耦合,高内聚在面向对象原则的引申,将职责定义为引起变化的原因,以提高内聚性减少引起变化的原因。

#### [#](#定义) 定义

一个对象应该只包含单一的职责，并且该职责被完整地封装在一个类中。(Every object should have a single responsibility, and that responsibility should be entirely encapsulated by the class.)，即又定义有且仅有一个原因使类变更。

#### [#](#原则分析) 原则分析

- 一个类(或者大到模块，小到方法)承担的职责越多，它被复用的可能性越小，而且如果一个类承担的职责过多，就相当于将这些职责耦合在一起，当其中一个职责变化时，可能会影响其他职责的运作。
- 类的职责主要包括两个方面: 数据职责和行为职责，数据职责通过其属性来体现，而行为职责通过其方法来体现。
- 单一职责原则是实现高内聚、低耦合的指导方针，在很多代码重构手法中都能找到它的存在，它是最简单但又最难运用的原则，需要设计人员发现类的不同职责并将其分离，而发现类的多重职责需要设计人员具有较强的分析设计能力和相关重构经验。

#### [#](#优点) 优点

- 降低类的复杂性，类的职责清晰明确。比如数据职责和行为职责清晰明确。
- 提高类的可读性和维护性，
- 变更引起的风险减低，变更是必不可少的，如果接口的单一职责做得好，一个接口修改只对相应的类有影响，对其他接口无影响，这对系统的扩展性、维护性都有非常大的帮助。

> 注意: 单一职责原则提出了一个编写程序的标准，用“职责”或“变化原因”来衡量接口或类设计得是否合理，但是“职责”和“变化原因”都是没有具体标准的，一个类到底要负责那些职责? 这些职责怎么细化? 细化后是否都要有一个接口或类? 这些都需从实际的情况考虑。因项目而异，因环境而异。

#### [#](#例子) 例子

SpringMVC 中Entity,DAO,Service,Controller, Util等的分离。

### [#](#o开放封闭原则ocp) O开放封闭原则OCP

> Open - ClosedPrinciple ,OCP, 对扩展开放，对修改关闭(设计模式的核心原则)

#### [#](#定义-1) 定义

> 一个软件实体(如类、模块和函数)应该对扩展开放，对修改关闭. 意思是,在一个系统或者模块中,对于扩展是开放的,对于修改是关闭的,一个 好的系统是在不修改源代码的情况下,可以扩展你的功能. 而实现开闭原则的关键就是抽象化.

#### [#](#原则分析-1) 原则分析

- 当软件实体因需求要变化时, 尽量通过扩展已有软件实体，可以提供新的行为，以满足对软件的新的需求，而不是修改已有的代码，使变化中的软件有一定的适应性和灵活性 。已有软件模块，特别是最重要的抽象层模块不能再修改，这使变化中的软件系统有一定的稳定性和延续性。
- 实现开闭原则的关键就是抽象化 :在"开-闭"原则中,不允许修改的是抽象的类或者接口,允许扩展的是具体的实现类,抽象类和接口在"开-闭"原则中扮演着极其重要的角色..即要预知可能变化的需求.又预见所有可能已知的扩展..所以在这里"抽象化"是关键!
- 可变性的封闭原则:找到系统的可变因素,将它封装起来. 这是对"开-闭"原则最好的实现. 不要把你的可变因素放在多个类中,或者散落在程序的各个角落. 你应该将可变的因素,封套起来..并且切忌不要把所用的可变因素封套在一起. 最好的解决办法是,分块封套你的可变因素!避免超大类,超长类,超长方法的出现!!给你的程序增加艺术气息,将程序艺术化是我们的目标!

#### [#](#例子-1) 例子

设计模式中模板方法模式和观察者模式都是开闭原则的极好体现。

### [#](#l里氏替换原则lsp) L里氏替换原则LSP

```
里氏替换原则强调，派生类应该能够完全替代其基类，而不会破坏代码的正确性和期望行为。
```



> Liskov Substitution Principle ,LSP: 任何基类可以出现的地方,子类也可以出现；这一思想表现为对继承机制的约束规范,只有子类能够替换其基类时,才能够保证系统在运行期内识别子类,这是保证继承复用的基础。

#### [#](#定义-2) 定义

第一种定义方式相对严格: 如果对每一个类型为S的对象o1，都有类型为T的对象o2，使得以T定义的所有程序P在所有的对象o1都代换成o2时，程序P的行为没有变化，那么类型S是类型T的子类型。

第二种更容易理解的定义方式: 所有引用基类(父类)的地方必须能透明地使用其子类的对象。即子类能够必须能够替换基类能够从出现的地方。子类也能在基类 的基础上新增行为。 (里氏代换原则由2008年图灵奖得主、美国第一位计算机科学女博士、麻省理工学院教授BarbaraLiskov和卡内基.梅隆大学Jeannette Wing教授于1994年提出。其原文如下: Let q(x) be a property provableabout objects x of type T. Then q(y) should be true for objects y of type Swhere S is a subtype of T. )

#### [#](#原则分析-2) 原则分析

- 讲的是基类和子类的关系，只有这种关系存在时，里氏代换原则才存在。正方形是长方形是理解里氏代换原则的经典例子。
- 里氏代换原则可以通俗表述为: 在软件中如果能够使用基类对象，那么一定能够使用其子类对象。把基类都替换成它的子类，程序将不会产生任何错误和异常，反过来则不成立，如果一个软件实体使用的是一个子类的话，那么它不一定能够使用基类。
- 里氏代换原则是实现开闭原则的重要方式之一，由于使用基类对象的地方都可以使用子类对象，因此在程序中尽量使用基类类型来对对象进行定义，而在运行时再确定其子类类型，用子类对象来替换父类对象。

### [#](#i接口隔离法则) I接口隔离法则

> (Interface Segregation Principle，ISL): 客户端不应该依赖那些它不需要的接口。(这个法则与迪米特法则是相通的)

#### [#](#定义-3) 定义

客户端不应该依赖那些它不需要的接口。

另一种定义方法: 一旦一个接口太大，则需要将它分割成一些更细小的接口，使用该接口的客户端仅需知道与之相关的方法即可。 注意，在该定义中的接口指的是所定义的方法。例如外面调用某个类的public方法。这个方法对外就是接口。

#### [#](#原则分析-3) 原则分析:

- 接口隔离原则是指使用多个专门的接口，而不使用单一的总接口。每一个接口应该承担一种相对独立的角色，不多不少，不干不该干的事，该干的事都要干。 
  - 一个接口就只代表一个角色，每个角色都有它特定的一个接口，此时这个原则可以叫做“角色隔离原则”。
  - 接口仅仅提供客户端需要的行为，即所需的方法，客户端不需要的行为则隐藏起来，应当为客户端提供尽可能小的单独的接口，而不要提供大的总接口。
- 使用接口隔离原则拆分接口时，首先必须满足单一职责原则，将一组相关的操作定义在一个接口中，且在满足高内聚的前提下，接口中的方法越少越好。
- 可以在进行系统设计时采用定制服务的方式，即为不同的客户端提供宽窄不同的接口，只提供用户需要的行为，而隐藏用户不需要的行为。

### [#](#d依赖倒置原则dip) D依赖倒置原则DIP

> Dependency-Inversion Principle 要依赖抽象,而不要依赖具体的实现, 具体而言就是高层模块不依赖于底层模块,二者共同依赖于抽象。抽象不依赖于具体,具体依赖于抽象。

#### [#](#定义-4) 定义

高层模块不应该依赖低层模块，它们都应该依赖抽象。抽象不应该依赖于细节，细节应该依赖于抽象。简单的说，依赖倒置原则要求客户端依赖于抽象耦合。原则表述:

1)抽象不应当依赖于细节；细节应当依赖于抽象；

2)要针对接口编程，不针对实现编程。

#### [#](#原则分析-4) 原则分析

- 如果说开闭原则是面向对象设计的目标,依赖倒转原则是到达面向设计"开闭"原则的手段..如果要达到最好的"开闭"原则,就要尽量的遵守依赖倒转原则. 可以说依赖倒转原则是对"抽象化"的最好规范! 我个人感觉,依赖倒转原则也是里氏代换原则的补充..你理解了里氏代换原则,再来理解依赖倒转原则应该是很容易的。
- 依赖倒转原则的常用实现方式之一是在代码中使用抽象类，而将具体类放在配置文件中。
- 类之间的耦合: 零耦合关系，具体耦合关系，抽象耦合关系。依赖倒转原则要求客户端依赖于抽象耦合，以抽象方式耦合是依赖倒转原则的关键。

#### [#](#例子-2) 例子

理解这个依赖倒置，首先我们需要明白依赖在面向对象设计的概念:

依赖关系(Dependency): 是一种使用关系，特定事物的改变有可能会影响到使用该事物的其他事物，在需要表示一个事物使用另一个事物时使用依赖关系。(假设A类的变化引起了B类的变化，则说名B类依赖于A类。)大多数情况下，依赖关系体现在某个类的方法使用另一个类的对象作为参数。在UML中，依赖关系用带箭头的虚线表示，由依赖的一方指向被依赖的一方。

例子: 某系统提供一个数据转换模块，可以将来自不同数据源的数据转换成多种格式，如可以转换来自数据库的数据(DatabaseSource)、也可以转换来自文本文件的数据(TextSource)，转换后的格式可以是XML文件(XMLTransformer)、也可以是XLS文件(XLSTransformer)等。

![img](/images/dev_rules_d_1.png)

由于需求的变化，该系统可能需要增加新的数据源或者新的文件格式，每增加一个新的类型的数据源或者新的类型的文件格式，客户类MainClass都需要修改源代码，以便使用新的类，但违背了开闭原则。现使用依赖倒转原则对其进行重构。

![img](/images/dev_rules_d_2.png)

- 当然根据具体的情况，也可以将AbstractSource注入到AbstractStransformer，依赖注入的方式有以下三种:

```java
/** 
 * 依赖注入是依赖AbstractSource抽象注入的，而不是具体 
 * DatabaseSource 
 * 
 */  
abstract class AbstractStransformer {  
    private AbstractSource source;   
    /** 
     * 构造注入(Constructor Injection): 通过构造函数注入实例变量。 
     */  
    public void AbstractStransformer(AbstractSource source){  
        this.source = source;           
    }  
    /**      
     * 设值注入(Setter Injection): 通过Setter方法注入实例变量。 
     * @param source : the sourceto set        
     */       
    public void setSource(AbstractSource source) {            
        this.source = source;             
    }  
    /** 
     * 接口注入(Interface Injection): 通过接口方法注入实例变量。 
     * @param source 
     */  
    public void transform(AbstractSource source ) {    
        source.getSource();  
        System.out.println("Stransforming ...");    
    }      
}
```

## [#](#其它原则) 其它原则

### [#](#合成-聚合复用原则) 合成/聚合复用原则

> (Composite/Aggregate ReusePrinciple ，CARP): 要尽量使用对象组合,而不是继承关系达到软件复用的目的

#### [#](#定义-5) 定义

经常又叫做合成复用原则(Composite ReusePrinciple或CRP)，尽量使用对象组合，而不是继承来达到复用的目的。

就是在一个新的对象里面使用一些已有的对象，使之成为新对象的一部分；新对象通过向这些对象的委派达到复用已有功能的目的。简而言之，要尽量使用合成/聚合，尽量不要使用继承。

#### [#](#原则分析-5) 原则分析:

- 在面向对象设计中，可以通过两种基本方法在不同的环境中复用已有的设计和实现，即通过组合/聚合关系或通过继承。 
  - 继承复用: 实现简单，易于扩展。破坏系统的封装性；从基类继承而来的实现是静态的，不可能在运行时发生改变，没有足够的灵活性；只能在有限的环境中使用。(“白箱”复用)
  - 组合/聚合复用: 耦合度相对较低，选择性地调用成员对象的操作；可以在运行时动态进行。(“黑箱”复用)
- 组合/聚合可以使系统更加灵活，类与类之间的耦合度降低，一个类的变化对其他类造成的影响相对较少，因此一般首选使用组合/聚合来实现复用；其次才考虑继承，在使用继承时，需要严格遵循里氏代换原则，有效使用继承会有助于对问题的理解，降低复杂度，而滥用继承反而会增加系统构建和维护的难度以及系统的复杂度，因此需要慎重使用继承复用。
- 此原则和里氏代换原则氏相辅相成的,两者都是具体实现"开-闭"原则的规范。违反这一原则，就无法实现"开-闭"原则，首先我们要明白合成和聚合的概念:

> 注意: 聚合和组合的区别是什么?

合成(组合): 表示一个整体与部分的关系，指一个依托整体而存在的关系(整体与部分不可以分开)；比如眼睛和嘴对于头来说就是组合关系，没有了头就没有眼睛和嘴，它们是不可分割的。在UML中，组合关系用带实心菱形的直线表示。

聚合:聚合是比合成关系的一种更强的依赖关系,也表示整体与部分的关系(整体与部分可以分开)；比如螺丝和汽车玩具的关系，螺丝脱离玩具依然可以用在其它设备之上。在UML中，聚合关系用带空心菱形的直线表示。

### [#](#迪米特法则) 迪米特法则

> Law of Demeter，LoD: 系统中的类,尽量不要与其他类互相作用,减少类之间的耦合度

#### [#](#定义-6) 定义

又叫最少知识原则(Least Knowledge Principle或简写为LKP)几种形式定义:

- 不要和“陌生人”说话。英文定义为: Don't talk to strangers.
- 只与你的直接朋友通信。英文定义为: Talk only to your immediate friends.
- 每一个软件单位对其他的单位都只有最少的知识，而且局限于那些与本单位密切相关的软件单位。

简单地说，也就是，一个对象应当对其它对象有尽可能少的了解。一个类应该对自己需要耦合或调用的类知道得最少，你(被耦合或调用的类)的内部是如何复杂都和我没关系，那是你的事情，我就知道你提供的public方法，我就调用这么多，其他的一概不关心。

#### [#](#法则分析) 法则分析

- **朋友类**:

在迪米特法则中，对于一个对象，其朋友包括以下几类:

1. 当前对象本身(this)；
2. 以参数形式传入到当前对象方法中的对象；
3. 当前对象的成员对象；
4. 如果当前对象的成员对象是一个集合，那么集合中的元素也都是朋友；
5. 当前对象所创建的对象。

任何一个对象，如果满足上面的条件之一，就是当前对象的“朋友”，否则就是“陌生人”。

- **狭义法则和广义法则**:

在狭义的迪米特法则中，如果两个类之间不必彼此直接通信，那么这两个类就不应当发生直接的相互作用，如果其中的一个类需要调用另一个类的某一个方法的话，可以通过第三者转发这个调用。

狭义的迪米特法则: 可以降低类之间的耦合，但是会在系统中增加大量的小方法并散落在系统的各个角落，它可以使一个系统的局部设计简化，因为每一个局部都不会和远距离的对象有直接的关联，但是也会造成系统的不同模块之间的通信效率降低，使得系统的不同模块之间不容易协调。

广义的迪米特法则: 指对对象之间的信息流量、流向以及信息的影响的控制，主要是对信息隐藏的控制。信息的隐藏可以使各个子系统之间脱耦，从而允许它们独立地被开发、优化、使用和修改，同时可以促进软件的复用，由于每一个模块都不依赖于其他模块而存在，因此每一个模块都可以独立地在其他的地方使用。一个系统的规模越大，信息的隐藏就越重要，而信息隐藏的重要性也就越明显。

- 迪米特法则的主要用途: 在于控制信息的过载。 
  - 在类的划分上，应当尽量创建松耦合的类，类之间的耦合度越低，就越有利于复用，一个处在松耦合中的类一旦被修改，不会对关联的类造成太大波及；
  - 在类的结构设计上，每一个类都应当尽量降低其成员变量和成员函数的访问权限；
  - 在类的设计上，只要有可能，一个类型应当设计成不变类；
  - 在对其他类的引用上，一个对象对其他对象的引用应当降到最低。

#### [#](#例子-3) 例子

外观模式Facade(结构型)

迪米特法则与设计模式Facade模式、Mediator模式

> 系统中的类,尽量不要与其他类互相作用,减少类之间的耦合度,因为在你的系统中,扩展的时候,你可能需要修改这些类,而类与类之间的关系,决定了修改的复杂度,相互作用越多,则修改难度就越大,反之,如果相互作用的越小,则修改起来的难度就越小..例如A类依赖B类,则B类依赖C类,当你在修改A类的时候,你要考虑B类是否会受到影响,而B类的影响是否又会影响到C类. 如果此时C类再依赖D类的话,呵呵,我想这样的修改有的受了。

## [#](#q-a) Q&A

### [#](#面向对象设计其他原则) 面向对象设计其他原则

封装变化

少用继承 多用组合

针对接口编程 不针对实现编程

为交互对象之间的松耦合设计而努力

类应该对扩展开发 对修改封闭(开闭OCP原则)

依赖抽象，不要依赖于具体类(依赖倒置DIP原则)

密友原则: 只和朋友交谈(最少知识原则，迪米特法则)

说明: 一个对象应当对其他对象有尽可能少的了解，将方法调用保持在界限内，只调用属于以下范围的方法: 该对象本身(本地方法)对象的组件 被当作方法参数传进来的对象 此方法创建或实例化的任何对象

别找我(调用我) 我会找你(调用你)(好莱坞原则)

一个类只有一个引起它变化的原因(单一职责SRP原则)

### [#](#你能解释一下里氏替换原则吗) 你能解释一下里氏替换原则吗?

严格定义: 如果对每一个类型为S的对象o1，都有类型为T的对象o2，使得以T定义的所有程序P在所有的对象用o1替换o2时，程序P的行为没有变化，那么类型S是类型T的子类型。

通俗表述: 所有引用基类(父类)的地方必须能透明地使用其子类的对象。也就是说子类可以扩展父类的功能，但不能改变父类原有的功能。它包含以下4层含义:

子类可以实现父类的抽象方法，但不能覆盖父类的非抽象方法。 子类中可以增加自己特有的方法。 当子类的方法重载父类的方法时，方法的前置条件(即方法的形参)要比父类方法的输入参数更宽松。 当子类的方法实现父类的抽象方法时，方法的后置条件(即方法的返回值)要比父类更严格。

### [#](#什么情况下会违反迪米特法则-为什么会有这个问题) 什么情况下会违反迪米特法则? 为什么会有这个问题?

迪米特法则建议“只和朋友说话，不要陌生人说话”，以此来减少类之间的耦合。

### [#](#给我一个符合开闭原则的设计模式的例子) 给我一个符合开闭原则的设计模式的例子?

开闭原则要求你的代码对扩展开放，对修改关闭。这个意思就是说，如果你想增加一个新的功能，你可以很容易的在不改变已测试过的代码的前提下增加新的代码。有好几个设计模式是基于开闭原则的，如策略模式，如果你需要一个新的策略，只需要实现接口，增加配置，不需要改变核心逻辑。一个正在工作的例子是 Collections.sort() 方法，这就是基于策略模式，遵循开闭原则的，你不需为新的对象修改 sort() 方法，你需要做的仅仅是实现你自己的 Comparator 接口。

### [#](#什么时候使用享元模式-蝇量模式) 什么时候使用享元模式(蝇量模式)?

享元模式通过共享对象来避免创建太多的对象。为了使用享元模式，你需要确保你的对象是不可变的，这样你才能安全的共享。JDK 中 String 池、Integer 池以及 Long 池都是很好的使用了享元模式的例子。

## [#](#参考文章) 参考文章

- 设计模式六大原则 http://www.uml.org.cn/sjms/201211023.asp
- 设计模式原则详解 http://blog.csdn.net/hguisu/article/details/7571617

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/dev-spec/spec/dev-th-solid.html