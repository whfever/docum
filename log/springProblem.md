# SpringProblem

## Caused by: java.lang.NoClassDefFoundError: org/springframework/dao/support/DaoSupport



```sql
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-jdbc</artifactId>
    <version>4.1.5.RELEASE</version>
  </dependency>
```



## java.lang.IllegalStateException: Failed to load ApplicationContext
	at org.springframework.test.context.cache.DefaultCacheAwareContextLoaderDelegate.loadContex
	



## org.springframework.beans.factory.xml.XmlBeanDefinitionStoreException

```
xml
```



## org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'sqlSessionFactory' 

因为我在mybatis里面这段代码没有删除导致spring和mybatis都扫描了一遍xml，就出错了

，删spring里面的或者mybatis的都行

```xml
　　<mappers>
        <mapper resource="cn/dao/WorkinggMapper.xml"/>
    </mappers>
        <!-- 配置SQL映射文件信息 -->
        <property name="mapperLocations">
            <list>
                <value>classpath:cn/dao/**/*.xml</value>
            </list>
        </property>
```



## org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'org.example.service.UserService' available:

```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    UserMapper userMapper;
    @Override
    public List<User> selectAll() {
        System.out.println("users = ");
        List<User> users = userMapper.findAll();

        System.out.println("users = " + users);
        return users;
    }
}

```

## java.lang.NoSuchMethodError: org.springframework.core.annotation.AnnotatedElementUtils.getAnnotationAttributes

```
pom.xml里面的springframework版本要统一啊
```



## java.lang.ClassNotFoundException: com.mysql.cj.jdbc.Driver

```xml
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.11</version>
        </dependency>
```





## java.lang.NullPointerException

```
@Configuration
@ComponentScan("org.example")
public class AppConfig {
}
```



## java.lang.IllegalArgumentException: Property 'sqlSessionFactory' or 'sqlSessionTemplate' are required

```java
public class MybatisConfig {
    //定义bean：SqlSessionFactoryBean，用于产生SqlSessionFactory对象
    @Bean
    public SqlSessionFactoryBean sqlSessionFactory(@Autowired DataSource dataSource){
        SqlSessionFactoryBean ssfb = new SqlSessionFactoryBean();
        //设置模型类的别名扫描
        ssfb.setTypeAliasesPackage("org.example.entity");
        //设置数据源
        ssfb.setDataSource(dataSource);
        return ssfb;
    }
    }

@Import({DataSourceConfig.class, MybatisConfig.class})
```



## SpringApplication.class类文件具有错误的版本 61.0, 应为 52.0
    
     boot 3.0.0版本要求jdk17以上，自己的版本是jdk8