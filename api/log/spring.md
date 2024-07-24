# spring

## create
1. maven
2. idea
3. starter.spring.io

## 简化开发
前言
```java



UserDaoImpl userDao = new UserDaoImpl();
UserSericeImpl userService = new UserServiceImpl();
userService.setUserDao(userDao);
List<User> userList = userService.findUserList();


```

1. XML配置  
```xml
    <bean id="userService" class="org.example.UserServiceImpl">
        <property name="userDao" ref="userDao"/>
        <!-- additional collaborators and configuration for this bean go here -->
    </bean>


```

2. java配置
```java
@Configuration
public class BeansConfig {
    @Bean("userDao")
    public UserDaoImpl userDao(){
        return new UserDaoImpl();
    }
    @Bean("userservice")
    public  UserServiceImpl  userService(){
        UserServiceImpl userService = new UserServiceImpl();
        userService.setUserDao(userDao());
        return userService;

    }
}


```
3. annotion
```java
@Repository
//@Service
public class StudentServiceImpl {
    /**
     * user dao impl.
     */
    @Autowired
    private StudentDaoImpl studentDao;
    
    public void setUserDao(StudentDaoImpl studentDao) {
        this.studentDao = studentDao;
    }
    public List<Student> findUserList() {
        return this.studentDao.findUserList();
    }
}

```


## iC 方式
```java
    @Autowired // 这里@Autowired也可以省略
    public UserServiceImpl(final UserDaoImpl userDaoImpl) {
        this.userDao = userDaoImpl;
    }


```

推荐构造器注入方式:
1. 如果使用构造器注入，在spring项目启动的时候，就会抛出：BeanCurrentlyInCreationException：Requested bean is currently in creation: Is there an unresolvable circular reference？从而提醒你避免循环依赖，如果是field注入的话，启动的时候不会报错，在使用那个bean的时候才会报错。
2. 如果使用setter注入，缺点显而易见，对于IOC容器以外的环境，除了使用反射来提供它需要的依赖之外，无法复用该实现类。而且将一直是个潜在的隐患，因为你不调用将一直无法发现NPE的存在。
3. 依赖不可变：其实说的就是final关键字。

## @Autowired
1、@Autowired是Spring自带的注解，通过AutowiredAnnotationBeanPostProcessor 类实现的依赖注入
2、@Autowired可以作用在CONSTRUCTOR、METHOD、PARAMETER、FIELD、ANNOTATION_TYPE
3、@Autowired默认是根据类型（byType ）进行自动装配的
4、如果有多个类型一样的Bean候选者，需要指定按照名称（byName ）进行装配，则需要配合@Qualifier。


## AOP思想
对于这个过程，一般分为动态织入和静态织入：
1. 动态织入的方式是在运行时动态将要增强的代码织入到目标类中，这样往往是通过动态代理技术完成的，如Java JDK的动态代理(Proxy，底层通过反射实现)或者CGLIB的动态代理(底层通过继承实现)，Spring AOP采用的就是基于运行时增强的代理技术ApectJ采用的就是静态织入的方式。
2. ApectJ主要采用的是编译期织入，在这个期间使用AspectJ的acj编译器(类似javac)把aspect类编译成class字节码后，在java目标类编译时织入，即先编译aspect类再编译目标类。

JDK的动态代理
```java
/**
 * Jdk Proxy Service.
 *
 * @author pdai
 */
public interface IJdkProxyService {

    void doMethod1();

    String doMethod2();

    String doMethod3() throws Exception;
}
/**
 * @author pdai
 */
@Service
public class JdkProxyDemoServiceImpl implements IJdkProxyService {

    @Override
    public void doMethod1() {
        System.out.println("JdkProxyServiceImpl.doMethod1()");
    }

    @Override
    public String doMethod2() {
        System.out.println("JdkProxyServiceImpl.doMethod2()");
        return "hello world";
    }

    @Override
    public String doMethod3() throws Exception {
        System.out.println("JdkProxyServiceImpl.doMethod3()");
        throw new Exception("some exception");
    }
}


```

## Spring MVC
用一种业务逻辑、数据、界面显示分离的方法，将业务逻辑聚集到一个部件里面

![](https://i-blog.csdnimg.cn/blog_migrate/a98d13f331dc64c1e1495255fb1b73fb.png)


## Log 


## SSM 
1. from maven-webapp 
2. java  resource 
3. pom.xml 
4. tomcat
apache-tomcat-9.0.43\conf\logging.properties配置文件
```xml
java.util.logging.ConsoleHandler.encoding = GBK
```

```bash
service install
service remove
startup.bat 启动tomcat服务

shutdown.bat 关闭tomcat服务
```


