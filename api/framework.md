# FrameWork

## Mybatis

mybatis是一个优秀的基于 java 的持久层框架，它内部封装了 jdbc，使开发者只需要关注 sql 语句本身，而不需要花费精力去处理加载驱动、创建连接、创建 statement 等繁杂的过程。

mybatis通过 xml 或注解的方式将要执行的各种**statement配置**起来，并通过java对象和statement 中sql 的动态参数进行映射生成最终执行的 sql 语句，最后由 mybatis 框架执行 sql 并将结果映射为 java 对象并返回。

采用 ORM 思想解决了实体和数据库映射的问题，对 jdbc进行了封装，屏蔽了 jdbc api 底层访问细节，使我们不用与 jdbc api 打交道，就可以完成对数据库的持久化操作。

### Mybatis快速入门

```java
		//1. 读取配置文件
        InputStream is = Resources.getResourceAsStream("mybatis-config.xml");
        /**
         *  <mappers>
         *         <mapper resource="org/example/dao/UserMapper.xml"/>
         *          :::usermapper.xml
         *         <mapper namespace="org.example.dao.UserDao">
         *            <select id="findAll" resultType="org.example.entity.User">
         *                 select * from user
         *            </select>
         *          </mapper>
         *          :::usermapper.xml
         *
         *         <mapper resource="org/example/dao/DownMapper.xml"/>
         *         <mapper class="org.example.dao.StudentDao"/>
         *  </mappers>
         */
        //2. 创建SqlSessionFactory工厂
        SqlSessionFactoryBuilder builder = new SqlSessionFactoryBuilder();
        SqlSessionFactory factory = builder.build(is);
        //3. 使用工厂生产SqlSession对象
        SqlSession sqlSession = factory.openSession();
        //4. 使用SqlSession创建Dao接口的代理对象
        UserDao userMapper = sqlSession.getMapper(UserDao.class);
        //5. 使用代理对象执行方法
        List<User> users = userMapper.findAll();
        for (User user : users) {
            System.out.println(user);
        }
```

![](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/17/16f13e72188384dc~tplv-t2oaga2asx-jj-mark:3024:0:0:0:q75.awebp)

### Mybatis 进阶

1. 连接池及事务控制
2. 动态SQL
3. 分页插件，generator



## Spring

### Spring快速入门

**传统的方式：**
通过new 关键字主动创建一个对象

**IOC方式：**
对象的生命周期由Spring来管理，直接从Spring那里去获取一个对象。 IOC是反转控制 (Inversion Of Control)的缩写，就像控制权从本来在自己手里，交给了Spring。

**DI：Dependency Injection（依赖注入）**

- 指 Spring 创建对象的过程中，将对象依赖属性（简单值，集合，对象）通过配置设值给该对象

**AOP 的目的**

AOP能够将那些与业务无关，却为业务模块所共同调用的逻辑或责任（例如事务处理、日志管理、权限控制等）封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可拓展性和可维护性。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean name="source" class="pojo.Source">
        <property name="fruit" value="橙子"/>
        <property name="sugar" value="多糖"/>
        <property name="size" value="超大杯"/>
    </bean>
    <bean name="juickMaker" class="pojo.JuiceMaker">
        <property name="source" ref="source" />
    </bean>
     <bean name="productService" class="org.example.service.ProductService" />
    <bean id="loggerAspect" class="org.example.aspect.LoggerAspect"/>
    <!-- 配置AOP -->
    <aop:config>
        <!-- where：在哪些地方（包.类.方法）做增加 -->
        <aop:pointcut id="loggerCutpoint"
                      expression="execution(* org.example.service.ProductService.*(..)) "/>

        <!-- what:做什么增强 -->
        <aop:aspect id="logAspect" ref="loggerAspect">
            <!-- when:在什么时机（方法前/后/前后） -->
            <aop:around pointcut-ref="loggerCutpoint" method="log"/>
        </aop:aspect>
    </aop:config>
</beans>

```

```java
package test;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import pojo.JuiceMaker;
import pojo.Source;

public class TestSpring {

    @Test
    public void test(){
        ApplicationContext context = new ClassPathXmlApplicationContext(
                new String[]{"applicationContext.xml"}
        );

        Source source = (Source) context.getBean("source");
        System.out.println(source.getFruit());
        System.out.println(source.getSugar());
        System.out.println(source.getSize());

        JuiceMaker juiceMaker = (JuiceMaker) context.getBean("juickMaker");
        System.out.println(juiceMaker.makeJuice());
    }
}

```

### Spring Mybatis

> 整合思想

将mybatis接口代理对象的创建权交给spring管理，我们就可以把dao的代理对象注入到service中，  此时也就完成了spring与mybatis的整合了。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!--引入外部数据源的配置参数-->
    <context:property-placeholder location="classpath:jdbc.properties"></context:property-placeholder>
<!--    <context:property-placeholder location="classpath:config/jdbc.properties"></context:property-placeholder>-->

<!--    初始化数据库连接池-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="driverClassName" value="${jdbc.driver}"></property>
        <property name="url" value="${jdbc.url}"></property>
        <property name="username" value="${jdbc.username}"></property>
        <property name="password" value="${jdbc.password}"></property>
    </bean>

    <!--sqlSessionFactory-->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
        <!--设置数据库连接池-->
        <property name="dataSource" ref="dataSource"></property>
        <!--设置mybatis全局配置文件位置-->
        <property name="configLocation" value="mybatis-config.xml"></property>
        <!--设置别名的包-->
        <property name="typeAliasesPackage" value="org.example"></property>
        <!--设置mapper.xml文件的位置-->
<!--        <property name="mapperLocations" value="org/example/mapper/*.xml"></property>-->
    </bean>

    <!--mapper接口扫描  生成接口代理对象 同时完成对象的托管-->
    <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
        <property name="basePackage" value="org.example.mapper"></property>
    </bean>

    <!--开启包扫描  base-package  设置需要扫描的包 -->
    <context:component-scan base-package="org.example"></context:component-scan>
</beans>
```



## SpringBoot

### quick

```properties 
# 设置项目启动端口
server.port=8099
# 设置项目访问路径
server.servlet.context-path=/init
```

```xml
	<dependencies>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter</artifactId>
		</dependency>

		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-test</artifactId>
			<scope>test</scope>
		</dependency>
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
		</dependency>
	</dependencies>

	<build>
		<plugins>
			<plugin>
				<groupId>org.springframework.boot</groupId>
				<artifactId>spring-boot-maven-plugin</artifactId>
			</plugin>
		</plugins>
	</build>
```

