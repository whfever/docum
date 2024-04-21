## 结构模式

### 组合模式

> 组合模式是一种结构型设计模式，它允许将对象组合成树形结构，以表示“部分-整体”的层次结构，同时使得客户端统一对待单个对象和组合对象。

```Java
import java.util.ArrayList;
import java.util.List;

//抽象构件角色- 学校课表组件
interface CourseComponent {
    void add(CourseComponent courseComponent); // 添加
    void remove(CourseComponent courseComponent); // 移除
    void display(); // 展示
}

//叶子构件角色 - 学生
class Student implements CourseComponent {
    private String name;

    public Student(String name) {
        this.name = name;
    }

    @Override
    public void add(CourseComponent courseComponent) {
        // 叶子构件角色无法添加子节点
    }

    @Override
    public void remove(CourseComponent courseComponent) {
        // 叶子构件角色无法移除子节点
    }

    @Override
    public void display() {
        System.out.println("- " + name);
    }
}

// 容器构件角色 - 班级
class Class implements CourseComponent {
    private String name;
    private List<CourseComponent> courseComponents = new ArrayList<>();

    public Class(String name) {
        this.name = name;
    }

    @Override
    public void add(CourseComponent courseComponent) {
        courseComponents.add(courseComponent);
    }

    @Override
    public void remove(CourseComponent courseComponent) {
        courseComponents.remove(courseComponent);
    }

    @Override
    public void display() {
        System.out.println(name + "：");
        for (CourseComponent courseComponent : courseComponents) {
            courseComponent.display();
        }
    }
}

//容器构件角色 - 学校课表
class CourseSchedule implements CourseComponent {
    private List<CourseComponent> courseComponents = new ArrayList<>();

    @Override
    public void add(CourseComponent courseComponent) {
        courseComponents.add(courseComponent);
    }

    @Override
    public void remove(CourseComponent courseComponent) {
        courseComponents.remove(courseComponent);
    }

    @Override
    public void display() {
        System.out.println("学校课表：");
        for (CourseComponent courseComponent : courseComponents) {
            courseComponent.display();
        }
    }
}

public class Client {
    public static void main(String[] args) {
        CourseComponent math = new Class("数学班级");
        math.add(new Student("小明"));
        math.add(new Student("小红"));
        math.add(new Student("小华"));

        CourseComponent chinese = new Class("语文班级");
        chinese.add(new Student("小李"));
        chinese.add(new Student("小张"));

        CourseComponent english = new Class("英语班级");
        english.add(new Student("小刘"));
        english.add(new Student("小周"));
        english.add(new Student("小孙"));

        CourseComponent courseSchedule = new CourseSchedule();
        courseSchedule.add(math);
        courseSchedule.add(chinese);
        courseSchedule.add(english);

        courseSchedule.display();
    }
}

```





### 装饰模式

> 装饰模式是一种结构型设计模式，它允许在不修改现有对象结构的情况下，动态地向对象添加功能。 装饰器的主要思想是将装饰器对象与原始对象嵌套在一起，这个过程是透明的，因此无需更改原始对象本身。 装饰器模式旨在提供类别别名用于一些额外的操作。
>
> 装饰模式是一种结构型设计模式，它允许向现有对象动态添加新功能，同时又不改变其结构。装饰模式通过将对象包装在装饰器类中，使得每个装饰器都可以独立于其他装饰器来增加功能。

```java
// 组件接口
interface Window {
    void draw(); // 绘制窗口
}

// 具体组件
class SimpleWindow implements Window {
    @Override
    public void draw() {
        System.out.println("Drawing simple window.");
    }
}

// 抽象装饰器
abstract class WindowDecorator implements Window {
    protected Window window;

    public WindowDecorator(Window window) {
        this.window = window;
    }

    @Override
    public void draw() {
        window.draw();
    }
}

// 具体装饰器 - 边框装饰器
class BorderDecorator extends WindowDecorator {
    public BorderDecorator(Window window) {
        super(window);
    }

    @Override
    public void draw() {
        super.draw();
        drawBorder();
    }

    private void drawBorder() {
        System.out.println("Drawing border for the window.");
    }
}

// 具体装饰器 - 滚动条装饰器
class ScrollDecorator extends WindowDecorator {
    public ScrollDecorator(Window window) {
        super(window);
    }

    @Override
    public void draw() {
        super.draw();
        drawScrollBar();
    }

    private void drawScrollBar() {
        System.out.println("Drawing scroll bar for the window.");
    }
}

// 客户端使用示例
public class DecoratorPatternExample {
    public static void main(String[] args) {
        // 创建一个简单的窗口
        Window window = new SimpleWindow();

        // 添加边框装饰器
        Window decoratedWindow1 = new BorderDecorator(window);
        decoratedWindow1.draw();

        System.out.println("------------");

        // 添加边框和滚动条装饰器
        Window decoratedWindow2 = new ScrollDecorator(new BorderDecorator(window));
        decoratedWindow2.draw();
    }

```

### 外观模式

> 外观模式是一种结构型设计模式，它提供了一个统一的接口，用于访问子系统中一组接口的功能。外观模式通过创建一个高层接口，简化了客户端与子系统之间的交互，使得客户端更容易使用子系统。

```java
// 子系统接口1
interface Subsystem1 {
    void operation1();
}

// 子系统接口2
interface Subsystem2 {
    void operation2();
}

// 子系统1的实现
class ConcreteSubsystem1 implements Subsystem1 {
    @Override
    public void operation1() {
        System.out.println("Subsystem1 operation1");
    }
}

// 子系统2的实现
class ConcreteSubsystem2 implements Subsystem2 {
    @Override
    public void operation2() {
        System.out.println("Subsystem2 operation2");
    }
}

// 外观
class Facade {
    private Subsystem1 subsystem1;
    private Subsystem2 subsystem2;

    public Facade() {
        this.subsystem1 = new ConcreteSubsystem1();
        this.subsystem2 = new ConcreteSubsystem2();
    }

    public void doSomething() {
        subsystem1.operation1();
        subsystem2.operation2();
    }
}

// 客户端使用示例
public class FacadePatternExample {
    public static void main(String[] args) {
        Facade facade = new Facade();
        facade.doSomething();
    }
}

```

### 享元模式




> 享元模式是一种结构型设计模式，它旨在减少内存使用和提高性能。享元模式通过共享对象来有效地支持大量细粒度的对象，从而减少了对象的创建和内存消耗。
>
> 享元模式通常包含以下几个角色：
>
> - 享元接口（Flyweight）：定义了享元对象的接口，通过这个接口可以接收和操作外部状态。
> - 具体享元（Concrete Flyweight）：实现了享元接口，包含内部状态和外部状态。
> - 享元工厂（Flyweight Factory）：负责创建和管理享元对象，可以通过工厂方法获取享元对象。

```java
import java.util.HashMap;
import java.util.Map;

// 享元接口
interface Character {
    void draw();
}

// 具体享元
class ConcreteCharacter implements Character {
    private final char character;

    public ConcreteCharacter(char character) {
        this.character = character;
    }

    @Override
    public void draw() {
        System.out.println("Character: " + character);
    }
}

// 享元工厂
class CharacterFactory {
    private Map<Character, Character> characters = new HashMap<>();

    public Character getCharacter(char c) {
        if (characters.containsKey(c)) {
            return characters.get(c);
        } else {
            Character character = new ConcreteCharacter(c);
            characters.put(c, character);
            return character;
        }
    }
}

// 客户端使用示例
public class FlyweightPatternExample {
    public static void main(String[] args) {
        CharacterFactory characterFactory = new CharacterFactory();

        Character char1 = characterFactory.getCharacter('A');
        char1.draw();

        Character char2 = characterFactory.getCharacter('B');
        char2.draw();

        Character char3 = characterFactory.getCharacter('A');
        char3.draw();

        System.out.println("char1 == char3? " + (char1 == char3));
    }
}

```





### 代理模式

> 代理模式是一种结构型设计模式，它允许通过创建一个代理对象来控制对其他对象的访问。代理模式在访问对象时引入了一定程度的间接性，可以在不改变原始对象的情况下，增加额外的逻辑或控制访问方式。

```java
// 抽象主题
interface Image {
    void display();
}

// 真实主题
class RealImage implements Image {
    private String filename;

    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }

    private void loadFromDisk() {
        System.out.println("Loading image from disk: " + filename);
    }

    @Override
    public void display() {
        System.out.println("Displaying image: " + filename);
    }
}

// 代理
class ProxyImage implements Image {
    private String filename;
    private RealImage realImage;

    public ProxyImage(String filename) {
        this.filename = filename;
    }

    @Override
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);
        }
        realImage.display();
    }
}

// 客户端使用示例
public class ProxyPatternExample {
    public static void main(String[] args) {
        Image image1 = new ProxyImage("image1.jpg");
        image1.display();

        Image image2 = new ProxyImage("image2.jpg");
        image2.display();

        // 第二次调用时直接使用已加载的真实对象
        image2.display();
    }
}

```





## 行为模式

### 策略模式

> 策略模式是一种行为型设计模式，它定义了一系列算法，将每个算法封装到独立的类中，并使它们可以互换。策略模式使得算法可以独立于客户端而变化，客户端可以根据需要在运行时选择不同的算法。
>
> 策略模式通常包含以下几个角色：
>
> - 环境（Context）：维护一个对策略对象的引用，并在需要时调用策略对象的方法。
> - 抽象策略（Strategy）：定义了策略对象的公共接口，可以是抽象类或接口。
> - 具体策略（Concrete Strategy）：实现了抽象策略定义的接口，包含具体的算法实现。

```java
// 抽象策略
interface PaymentStrategy {
    void pay(double amount);
}

// 具体策略 - 信用卡支付
class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;
    private String cvv;

    public CreditCardStrategy(String cardNumber, String cvv) {
        this.cardNumber = cardNumber;
        this.cvv = cvv;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paying " + amount + " using credit card: " + cardNumber);
    }
}

// 具体策略 - 支付宝支付
class AlipayStrategy implements PaymentStrategy {
    private String username;
    private String password;

    public AlipayStrategy(String username, String password) {
        this.username = username;
        this.password = password;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paying " + amount + " using Alipay: " + username);
    }
}

// 具体策略 - 微信支付
class WechatPayStrategy implements PaymentStrategy {
    private String appId;
    private String appSecret;

    public WechatPayStrategy(String appId, String appSecret) {
        this.appId = appId;
        this.appSecret = appSecret;
    }

    @Override
    public void pay(double amount) {
        System.out.println("Paying " + amount + " using WeChat Pay: " + appId);
    }
}

// 环境
class PaymentContext {
    private PaymentStrategy paymentStrategy;

    public void setPaymentStrategy(PaymentStrategy paymentStrategy) {
        this.paymentStrategy = paymentStrategy;
    }

    public void pay(double amount) {
        paymentStrategy.pay(amount);
    }
}

// 客户端使用示例
public class StrategyPatternExample {
    public static void main(String[] args) {
        // 创建支付上下文
        PaymentContext paymentContext = new PaymentContext();

        // 设置信用卡支付策略
        PaymentStrategy creditCardStrategy = new CreditCardStrategy("123456789", "123");
        paymentContext.setPaymentStrategy(creditCardStrategy);
        paymentContext.pay(100.00);

        // 设置支付宝支付策略
        PaymentStrategy alipayStrategy = new AlipayStrategy("username", "password");
        paymentContext.setPaymentStrategy(alipayStrategy);
        paymentContext.pay(200.00);

        // 设置微信支付策略
        PaymentStrategy wechatPayStrategy = new WechatPayStrategy("AppId", "AppSecret");
        paymentContext.setPaymentStrategy(wechatPayStrategy);
        paymentContext.pay(300.00);
    }
}

```



### 责任链模式

> 责任链模式是一种行为型设计模式，它允许将请求沿着处理链进行传递，直到有一个处理者能够处理该请求为止。责任链模式将请求发送者和接收者解耦，使多个对象都有机会处理请求，同时避免了请求的发送者与接收者之间的直接耦合关系。
>
> 责任链模式通常包含以下几个角色：
>
> - 抽象处理者（Handler）：定义了处理请求的接口，并持有下一个处理者的引用。
> - 具体处理者（Concrete Handler）：实现了处理请求的方法，如果自己无法处理，则将请求传递给下一个处理者。
>
> 一个真实的开发场景可以是在一个工作流程系统中，需要依次处理多个环节的任务。每个环节都有不同的处理者，可以根据任务的类型选择合适的处理者进行处理，或者将任务交给下一个处理者。这时候可以使用责任链模式，将每个环节的处理逻辑封装在不同的处理者中，形成一个处理链。

```Java
// 抽象处理者
abstract class Handler {
    protected Handler successor;

    public void setSuccessor(Handler successor) {
        this.successor = successor;
    }

    public abstract void handleRequest(String request);
}

// 具体处理者 - 部门经理
class DepartmentManager extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("Day off")) {
            System.out.println("Department Manager: Approving day off.");
        } else if (successor != null) {
            successor.handleRequest(request);
        }
    }
}

// 具体处理者 - 总经理
class GeneralManager extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("Expense reimbursement")) {
            System.out.println("General Manager: Approving expense reimbursement.");
        } else if (successor != null) {
            successor.handleRequest(request);
        }
    }
}

// 具体处理者 - CEO
class CEO extends Handler {
    @Override
    public void handleRequest(String request) {
        if (request.equals("New project approval")) {
            System.out.println("CEO: Approving new project.");
        } else {
            System.out.println("No one can handle this request.");
        }
    }
}

// 客户端使用示例
public class ChainOfResponsibilityPatternExample {
    public static void main(String[] args) {
        // 创建处理者对象
        Handler departmentManager = new DepartmentManager();
        Handler generalManager = new GeneralManager();
        Handler ceo = new CEO();

        // 设置处理链
        departmentManager.setSuccessor(generalManager);
        generalManager.setSuccessor(ceo);

        // 发送请求
        departmentManager.handleRequest("Day off");
        departmentManager.handleRequest("Expense reimbursement");
        departmentManager.handleRequest("New project approval");
        departmentManager.handleRequest("Others");
    }
}

```





### 命令模式

> 命令模式是一种行为型设计模式，它将请求封装为一个对象，从而使您可以使用不同的请求对客户端参数化。命令模式允许将请求的发送者和接收者解耦，使得发送者不需要知道接收者的具体实现细节。
>
> 命令模式通常包含以下几个角色：
>
> - 命令接口（Command）：定义了执行命令的方法。
> - 具体命令（Concrete Command）：实现了命令接口，持有对接收者对象的引用，并在执行方法中调用接收者的相关操作。
> - 接收者（Receiver）：执行实际操作的对象。
> - 调用者（Invoker）：发送命令并要求执行命令的对象。
>
> 一个真实的开发场景可以是在文本编辑器中，实现撤销（Undo）和重做（Redo）功能。用户在编辑器中进行操作时，可以通过命令模式将每个操作封装为一个命令对象，并将命令对象存储在历史记录中。当用户要求撤销或重做时，可以执行相应的命令。

```Java
// 命令接口
interface Command {
    void execute();
}

// 具体命令 - 打开文档
class OpenDocumentCommand implements Command {
    private DocumentReceiver receiver;

    public OpenDocumentCommand(DocumentReceiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.open();
    }
}

// 具体命令 - 保存文档
class SaveDocumentCommand implements Command {
    private DocumentReceiver receiver;

    public SaveDocumentCommand(DocumentReceiver receiver) {
        this.receiver = receiver;
    }

    @Override
    public void execute() {
        receiver.save();
    }
}

// 接收者 - 文档编辑器
class DocumentReceiver {
    public void open() {
        System.out.println("Opening document");
    }

    public void save() {
        System.out.println("Saving document");
    }
}

// 调用者 - 命令执行器
class CommandExecutor {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void executeCommand() {
        command.execute();
    }
}

// 客户端使用示例
public class CommandPatternExample {
    public static void main(String[] args) {
        // 创建接收者对象
        DocumentReceiver receiver = new DocumentReceiver();

        // 创建具体命令对象并设置接收者
        Command openCommand = new OpenDocumentCommand(receiver);
        Command saveCommand = new SaveDocumentCommand(receiver);

        // 创建调用者对象并设置命令
        CommandExecutor executor = new CommandExecutor();
        executor.setCommand(openCommand);

        // 执行命令
        executor.executeCommand();

        // 切换命令并执行
        executor.setCommand(saveCommand);
        executor.executeCommand();
    }
}

```

#### 使用场景

- 如果你需要通过操作来参数化对象， 可使用命令模式。

 命令模式可将特定的方法调用转化为独立对象。 这一改变也带来了许多有趣的应用： 你可以将命令作为方法的参数进行传递、 将命令保存在其他对象中， 或者在运行时切换已连接的命令等。

举个例子： 你正在开发一个 GUI 组件 （例如上下文菜单）， 你希望用户能够配置菜单项， 并在点击菜单项时触发操作。

-  如果你想要将操作放入队列中、 操作的执行或者远程执行操作， 可使用命令模式。

 同其他对象一样， 命令也可以实现序列化 （序列化的意思是转化为字符串）， 从而能方便地写入文件或数据库中。 一段时间后， 该字符串可被恢复成为最初的命令对象。 因此， 你可以延迟或计划命令的执行。 但其功能远不止如此！ 使用同样的方式， 你还可以将命令放入队列、 记录命令或者通过网络发送命令。

- 如果你想要实现操作回滚功能， 可使用命令模式。

 尽管有很多方法可以实现撤销和恢复功能， 但命令模式可能是其中最常用的一种。

为了能够回滚操作， 你需要实现已执行操作的历史记录功能。 命令历史记录是一种包含所有已执行命令对象及其相关程序状态备份的栈结构。

这种方法有两个缺点。 首先， 程序状态的保存功能并不容易实现， 因为部分状态可能是私有的。 你可以使用[备忘录](https://refactoringguru.cn/design-patterns/memento)模式来在一定程度上解决这个问题。

其次， 备份状态可能会占用大量内存。 因此， 有时你需要借助另一种实现方式： 命令无需恢复原始状态， 而是执行反向操作。 反向操作也有代价： 它可能会很难甚至是无法实现。

### 迭代器模式

> 迭代器模式是一种行为型设计模式，用于提供一种顺序访问聚合对象元素的方法，而无需暴露其内部表示。通过迭代器模式，我们可以在不知道聚合对象内部结构的情况下，逐个访问聚合对象中的元素。
>
> 迭代器模式通常包含以下几个角色：
>
> - 迭代器接口（Iterator）：定义了访问和遍历元素的方法。
> - 具体迭代器（Concrete Iterator）：实现了迭代器接口，维护对聚合对象的引用，并跟踪当前位置。
> - 聚合对象接口（Aggregate）：定义了创建迭代器对象的方法。
> - 具体聚合对象（Concrete Aggregate）：实现了聚合对象接口，返回一个具体迭代器的实例。
>
> 一个真实的开发场景可以是在一个音乐播放器中，我们可以使用迭代器模式来遍历和播放播放列表中的歌曲。播放列表可以被视为聚合对象，而每首歌曲可以被视为聚合对象中的元素。通过使用迭代器模式，我们可以逐个遍历歌曲并播放它们，而无需暴露播放列表的内部实现。
>
> 

```java
// 迭代器接口
interface Iterator {
    boolean hasNext();
    Object next();
}

// 聚合对象接口
interface Aggregate {
    Iterator createIterator();
}

// 具体迭代器
class PlaylistIterator implements Iterator {
    private Playlist playlist;
    private int currentPosition;

    public PlaylistIterator(Playlist playlist) {
        this.playlist = playlist;
        this.currentPosition = 0;
    }

    @Override
    public boolean hasNext() {
        return currentPosition < playlist.getLength();
    }

    @Override
    public Object next() {
        Song song = playlist.getSong(currentPosition);
        currentPosition++;
        return song;
    }
}

// 具体聚合对象
class Playlist implements Aggregate {
    private Song[] songs;
    private int length;

    public Playlist() {
        songs = new Song[10];
        length = 0;
    }

    public void addSong(Song song) {
        if (length < songs.length) {
            songs[length] = song;
            length++;
        }
    }

    public Song getSong(int index) {
        if (index < length) {
            return songs[index];
        }
        return null;
    }

    public int getLength() {
        return length;
    }

    @Override
    public Iterator createIterator() {
        return new PlaylistIterator(this);
    }
}

// 歌曲类
class Song {
    private String title;

    public Song(String title) {
        this.title = title;
    }

    public String getTitle() {
        return title;
    }
}

// 客户端使用示例
public class IteratorPatternExample {
    public static void main(String[] args) {
        // 创建播放列表
        Playlist playlist = new Playlist();

        // 添加歌曲到播放列表
        playlist.addSong(new Song("Song 1"));
        playlist.addSong(new Song("Song 2"));
        playlist.addSong(new Song("Song 3"));

        // 创建迭代器
        Iterator iterator = playlist.createIterator();

        // 遍历播放列表并播放歌曲
        while (iterator.hasNext()) {
            Song song = (Song) iterator.next();
            System.out.println("Playing: " + song.getTitle());
        }
    }
}

```

### 中介者模式

> 中介者模式是一种行为型设计模式，它通过封装一组对象之间的交互方式，来减少对象之间的直接耦合。中介者模式通过引入一个中介者对象，将对象之间的交互转移给中介者处理，从而促进对象之间的松耦合和可维护性。
>
> 中介者模式通常包含以下几个角色：
>
> - 中介者（Mediator）：定义了对象之间交互的接口，维护对各个对象的引用。
> - 具体中介者（Concrete Mediator）：实现了中介者接口，协调各个对象之间的交互。
> - 同事对象（Colleague）：定义了与其他同事对象交互的接口，持有对中介者的引用。
> - 具体同事对象（Concrete Colleague）：实现了同事对象接口，通过中介者与其他同事对象进行交互。
>
> 一个真实的开发场景可以是一个多用户聊天室。聊天室中的每个用户都是一个同事对象，而聊天室本身则是中介者对象。当用户发送消息时，不需要直接与其他用户进行通信，而是通过中介者进行消息的转发和协调。这样可以避免用户之间直接的耦合，同时也方便了聊天室的管理和扩展。

```java
// 中介者接口
interface ChatMediator {
    void sendMessage(String message, User sender);
    void addUser(User user);
}

// 具体中介者
class ChatRoom implements ChatMediator {
    private List<User> users;

    public ChatRoom() {
        this.users = new ArrayList<>();
    }

    @Override
    public void sendMessage(String message, User sender) {
        for (User user : users) {
            if (user != sender) {
                user.receiveMessage(message);
            }
        }
    }

    @Override
    public void addUser(User user) {
        users.add(user);
    }
}

// 同事对象
abstract class User {
    protected ChatMediator mediator;
    protected String name;

    public User(ChatMediator mediator, String name) {
        this.mediator = mediator;
        this.name = name;
    }

    public abstract void send(String message);
    public abstract void receiveMessage(String message);
}

// 具体同事对象
class ChatUser extends User {
    public ChatUser(ChatMediator mediator, String name) {
        super(mediator, name);
    }

    @Override
    public void send(String message) {
        System.out.println(name + " sends: " + message);
        mediator.sendMessage(message, this);
    }

    @Override
    public void receiveMessage(String message) {
        System.out.println(name + " receives: " + message);
    }
}

// 客户端使用示例
public class MediatorPatternExample {
    public static void main(String[] args) {
        // 创建中介者
        ChatMediator chatMediator = new ChatRoom();

        // 创建用户
        User user1 = new ChatUser(chatMediator, "User1");
        User user2 = new ChatUser(chatMediator, "User2");
        User user3 = new ChatUser(chatMediator, "User3");

        // 添加用户到中介者
        chatMediator.addUser(user1);
        chatMediator.addUser(user2);
        chatMediator.addUser(user3);

        // 用户发送消息
        user1.send("Hello, everyone!");
        user2.send("Hi, User1!");
    }
}


```



### 备忘录模式

> 备忘录模式是一种行为型设计模式，用于捕获和恢复对象的内部状态，而不会破坏对象的封装性。备忘录模式通过在对象之外保存和恢复其内部状态，使得对象可以在需要时回到之前的状态。
>
> 备忘录模式通常包含以下几个角色：
>
> - 发起人（Originator）：负责创建备忘录对象，并可以使用备忘录对象恢复其内部状态。
> - 备忘录（Memento）：存储发起人的内部状态。
> - 管理者（Caretaker）：负责保存和管理备忘录对象。
>
> 一个真实的开发场景可以是文本编辑器中的撤销和恢复功能。当用户对文本进行编辑时，可以使用备忘录模式来保存编辑前的状态，以便在需要时撤销编辑操作或恢复到之前的状态。

```Java
// 备忘录类
class TextMemento {
    private String text;

    public TextMemento(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }
}

// 发起人类
class TextEditor {
    private String text;

    public void setText(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    public TextMemento createMemento() {
        return new TextMemento(text);
    }

    public void restoreFromMemento(TextMemento memento) {
        this.text = memento.getText();
    }
}

// 管理者类
class Caretaker {
    private List<TextMemento> mementos = new ArrayList<>();

    public void addMemento(TextMemento memento) {
        mementos.add(memento);
    }

    public TextMemento getMemento(int index) {
        if (index >= 0 && index < mementos.size()) {
            return mementos.get(index);
        }
        return null;
    }
}

// 客户端使用示例
public class MementoPatternExample {
    public static void main(String[] args) {
        // 创建发起人和管理者
        TextEditor editor = new TextEditor();
        Caretaker caretaker = new Caretaker();

        // 编辑文本并保存备忘录
        editor.setText("First version");
        caretaker.addMemento(editor.createMemento());

        editor.setText("Second version");
        caretaker.addMemento(editor.createMemento());

        editor.setText("Third version");

        // 恢复到第二个备忘录
        TextMemento memento = caretaker.getMemento(1);
        editor.restoreFromMemento(memento);

        System.out.println("Current text: " + editor.getText());
    }
}

```



### 观察者模式

> 观察者模式是一种行为型设计模式，用于在对象之间建立一种一对多的依赖关系。当一个对象的状态发生变化时，它会自动通知依赖它的其他对象，并使其能够自动更新。
>
> 观察者模式通常包含以下几个角色：
>
> - 主题（Subject）：负责维护一组观察者对象，提供添加、删除和通知观察者的方法。
> - 观察者（Observer）：定义了接收主题通知和更新的方法。
> - 具体主题（Concrete Subject）：实现了主题接口，维护了观察者对象的列表，并在状态发生变化时通知观察者。
> - 具体观察者（Concrete Observer）：实现了观察者接口，定义了接收通知和更新的具体行为。
>
> 一个真实的开发场景可以是一个新闻发布订阅系统。在该系统中，新闻机构充当主题，而订阅者则充当观察者。当新闻机构发布新闻时，订阅者会自动收到通知并更新自己的状态。

```java
import java.util.ArrayList;
import java.util.List;

// 主题接口
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers(String news);
}

// 具体主题
class NewsAgency implements Subject {
    private List<Observer> observers;
    private String news;

    public NewsAgency() {
        this.observers = new ArrayList<>();
    }

    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }

    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }

    @Override
    public void notifyObservers(String news) {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }

    public void setNews(String news) {
        this.news = news;
        notifyObservers(news);
    }
}

// 观察者接口
interface Observer {
    void update(String news);
}

// 具体观察者
class Subscriber implements Observer {
    private String name;

    public Subscriber(String name) {
        this.name = name;
    }

    @Override
    public void update(String news) {
        System.out.println(name + " received news: " + news);
    }
}

// 客户端使用示例
public class ObserverPatternExample {
    public static void main(String[] args) {
        // 创建主题和观察者
        NewsAgency newsAgency = new NewsAgency();
        Observer subscriber1 = new Subscriber("Subscriber 1");
        Observer subscriber2 = new Subscriber("Subscriber 2");

        // 订阅观察者
        newsAgency.attach(subscriber1);
        newsAgency.attach(subscriber2);

        // 发布新闻
        newsAgency.setNews("Breaking news: COVID-19 vaccine discovered!");

        // 取消订阅一个观察者
        newsAgency.detach(subscriber2);

        // 发布新闻
        newsAgency.setNews("New article published: 10 tips for a healthy lifestyle");

        /*
         * Output:
         * Subscriber 1 received news: Breaking news: COVID-19 vaccine discovered!
         * Subscriber 2 received news: Breaking news: COVID-19 vaccine discovered!
         * Subscriber 1 received news: New article published: 10 tips for a healthy lifestyle
         */
    }
}


```



### 状态模式



> 状态模式是一种行为型设计模式，它允许对象在内部状态改变时改变其行为，看起来好像对象在改变类。
>
> 状态模式通常包含以下几个角色：
>
> - 上下文（Context）：定义客户端所感兴趣的接口，维护一个当前状态对象，并在状态改变时调用状态对象的方法。
> - 抽象状态（State）：定义一个接口或抽象类，描述一个状态的行为。
> - 具体状态（Concrete State）：实现抽象状态定义的接口或抽象类，定义状态的具体行为。
>
> 一个真实的开发场景可以是自动售货机。在自动售货机中，根据当前的状态（如有货、无货、已支付等），自动售货机的行为会有所不同。当用户投入硬币时，如果售货机的状态是无货，则显示"无货"；如果售货机的状态是有货，则显示"请选择商品"；如果售货机的状态是已支付，则显示"请取走商品"。

```java
// 上下文类
class VendingMachine {
    private State state;

    public VendingMachine() {
        state = new NoStockState();
    }

    public void setState(State state) {
        this.state = state;
    }

    public void insertCoin() {
        state.insertCoin(this);
    }

    public void selectItem() {
        state.selectItem(this);
    }

    public void dispenseItem() {
        state.dispenseItem(this);
    }

    public void printStatus() {
        state.printStatus();
    }
}

// 抽象状态类
interface State {
    void insertCoin(VendingMachine machine);
    void selectItem(VendingMachine machine);
    void dispenseItem(VendingMachine machine);
    void printStatus();
}

// 具体状态类
class NoStockState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("无货，请取回硬币");
    }

    @Override
    public void selectItem(VendingMachine machine) {
        System.out.println("无货，请投币后选择");
    }

    @Override
    public void dispenseItem(VendingMachine machine) {
        System.out.println("无货，无法出货");
    }

    @Override
    public void printStatus() {
        System.out.println("当前状态：无货");
    }
}

class HasStockState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("已投币，请选择商品");
    }

    @Override
    public void selectItem(VendingMachine machine) {
        System.out.println("已支付，请取走商品");
        machine.setState(new PaidState());
    }

    @Override
    public void dispenseItem(VendingMachine machine) {
        System.out.println("请选择商品");
    }

    @Override
    public void printStatus() {
        System.out.println("当前状态：有货");
    }
}

class PaidState implements State {
    @Override
    public void insertCoin(VendingMachine machine) {
        System.out.println("已支付，请取走商品");
    }

    @Override
    public void selectItem(VendingMachine machine)
    public void dispenseItem(VendingMachine machine) {
        System.out.println("已支付，请取走商品");
        machine.setState(new NoStockState());
    }

    @Override
    public void printStatus() {
        System.out.println("当前状态：已支付");
    }
}

// 客户端使用示例
public class StatePatternExample {
    public static void main(String[] args) {
        // 创建自动售货机对象
        VendingMachine vendingMachine = new VendingMachine();

        // 测试无货状态
        vendingMachine.insertCoin();  // 无货，请取回硬币
        vendingMachine.selectItem();  // 无货，请投币后选择
        vendingMachine.dispenseItem();  // 无货，无法出货
        vendingMachine.printStatus();  // 当前状态：无货

        // 设置有货状态
        vendingMachine.setState(new HasStockState());

        // 测试有货状态
        vendingMachine.insertCoin();  // 已投币，请选择商品
        vendingMachine.selectItem();  // 已支付，请取走商品
        vendingMachine.dispenseItem();  // 请选择商品
        vendingMachine.printStatus();  // 当前状态：有货

        // 设置已支付状态
        vendingMachine.setState(new PaidState());

        // 测试已支付状态
        vendingMachine.insertCoin();  // 已支付，请取走商品
        vendingMachine.selectItem();  // 已支付，请取走商品
        vendingMachine.dispenseItem();  // 已支付，请取走商品
        vendingMachine.printStatus();  // 当前状态：已支付
    }
}

```

### 模板方法模式

> 模板方法模式是一种行为型设计模式，它定义了一个操作中的算法框架，将某些步骤的具体实现延迟到子类中。模板方法模式通过抽象类和具体子类的组合，使得子类可以在不改变算法结构的情况下重新定义算法中的某些步骤。
>
> 模板方法模式通常包含以下几个角色：
>
> - 抽象类（Abstract Class）：定义了一个模板方法，该方法中包含了算法的框架和一些抽象的步骤，子类必须实现这些抽象的步骤。
> - 具体子类（Concrete Class）：实现抽象类中定义的抽象步骤，完成算法中具体的实现。
>
> 一个真实的开发场景可以是制作咖啡和茶。无论制作咖啡还是茶，它们都有一些共同的步骤，如煮水、冲泡、加入调料等。这些步骤的顺序是固定的，但具体的实现可能不同。通过使用模板方法模式，我们可以定义一个饮料制作的模板类，将公共的步骤定义在模板方法中，而将具体的实现留给子类去完成。

```java
// 抽象类
abstract class BeverageMaker {
    // 模板方法
    public final void prepareBeverage() {
        boilWater();
        brew();
        pourInCup();
        addCondiments();
    }

    // 公共方法
    public void boilWater() {
        System.out.println("煮水");
    }

    public void pourInCup() {
        System.out.println("倒入杯中");
    }

    // 抽象方法，由子类实现
    public abstract void brew();

    public abstract void addCondiments();
}

// 具体子类
class CoffeeMaker extends BeverageMaker {
    @Override
    public void brew() {
        System.out.println("冲泡咖啡");
    }

    @Override
    public void addCondiments() {
        System.out.println("加入牛奶和糖");
    }
}

class TeaMaker extends BeverageMaker {
    @Override
    public void brew() {
        System.out.println("冲泡茶叶");
    }

    @Override
    public void addCondiments() {
        System.out.println("加入柠檬");
    }
}

// 客户端使用示例
public class TemplateMethodPatternExample {
    public static void main(String[] args) {
        BeverageMaker coffeeMaker = new CoffeeMaker();
        coffeeMaker.prepareBeverage();

        System.out.println("------------------");

        BeverageMaker teaMaker = new TeaMaker();
        teaMaker.prepareBeverage();
    }
}

```

### 访问者模式

> 访问者模式是一种行为型设计模式，它将数据结构和对数据的操作分离开来，使得可以在不改变数据结构的情况下定义新的操作。访问者模式通过定义一个访问者类，该类封装了对数据结构中各个元素的不同操作，从而实现对数据结构的透明访问和操作。
>
> 访问者模式通常包含以下几个角色：
>
> - 抽象访问者（Visitor）：定义了访问数据结构中各个元素的方法，每个方法对应一种操作。
> - 具体访问者（Concrete Visitor）：实现抽象访问者中定义的方法，完成对元素的具体操作。
> - 抽象元素（Element）：定义了接受访问者对象的方法，即`accept()`方法。
> - 具体元素（Concrete Element）：实现了抽象元素中定义的`accept()`方法，将自身作为参数传递给访问者对象。
> - 数据结构（Object Structure）：包含了一组具体元素对象，提供访问者访问元素的方法。
>
> 一个真实的开发场景可以是一个电商平台中的订单处理。假设有不同类型的订单，如普通订单、打折订单和促销订单等。每个订单类型可能有不同的处理逻辑，如计算订单总金额、计算优惠金额等。通过使用访问者模式，我们可以将订单作为数据结构，将订单处理逻辑抽象为访问者，从而实现对订单的不同操作。
>
> 下面是一个使用访问者模式的 Java 实现案例代码：

```java
// 抽象访问者
interface OrderVisitor {
    void visit(RegularOrder order);
    void visit(DiscountOrder order);
    void visit(PromotionOrder order);
}

// 具体访问者
class OrderTotalAmountVisitor implements OrderVisitor {
    private double totalAmount = 0;

    @Override
    public void visit(RegularOrder order) {
        totalAmount += order.getAmount();
    }

    @Override
    public void visit(DiscountOrder order) {
        totalAmount += order.getAmount() * (1 - order.getDiscountRate());
    }

    @Override
    public void visit(PromotionOrder order) {
        totalAmount += order.getPromotionAmount();
    }

    public double getTotalAmount() {
        return totalAmount;
    }
}

// 抽象元素
interface Order {
    void accept(OrderVisitor visitor);
}

// 具体元素
class RegularOrder implements Order {
    private double amount;

    public RegularOrder(double amount) {
        this.amount = amount;
    }

    public double getAmount() {
        return amount;
    }

    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}

class DiscountOrder implements Order {
    private double amount;
    private double discountRate;

    public DiscountOrder(double amount, double discountRate) {
        this.amount = amount;
        this.discountRate = discountRate;
    }

    public double getAmount() {
        return amount;
    }

    public double getDiscountRate() {
        return discountRate    ;
    }
            @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}

class PromotionOrder implements Order {
    private double promotionAmount;

    public PromotionOrder(double promotionAmount) {
        this.promotionAmount = promotionAmount;
    }

    public double getPromotionAmount() {
        return promotionAmount;
    }

    @Override
    public void accept(OrderVisitor visitor) {
        visitor.visit(this);
    }
}

// 客户端使用示例
public class VisitorPatternExample {
    public static void main(String[] args) {
        Order regularOrder = new RegularOrder(100);
        Order discountOrder = new DiscountOrder(200, 0.2);
        Order promotionOrder = new PromotionOrder(50);

        OrderVisitor totalAmountVisitor = new OrderTotalAmountVisitor();
        regularOrder.accept(totalAmountVisitor);
        discountOrder.accept(totalAmountVisitor);
        promotionOrder.accept(totalAmountVisitor);

        double totalAmount = ((OrderTotalAmountVisitor) totalAmountVisitor).getTotalAmount();
        System.out.println("Total Amount: " + totalAmount);
    }
}


```

