# javaProblem
## c3p0找不到
```xml
    <dependency>
      <groupId>c3p0</groupId>
      <artifactId>c3p0</artifactId>
      <version>0.9.0.2</version>
    </dependency>

```


## mybatis创建SqlSessionFactory的bean实例失败的排查思路


## Cannot find class [org.springframework.jdbc.datasource.DataSourceTransactionManager]

## Error creating bean with name 'org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerMapping': Invocation of init method failed;

## Cannot find class [src/main/java/controller/TestController] for bean with name '/hello.do' defined in file [D:\doc\commit\Nwgid\springdemo03ssm\target\classes\spring-web.xml]; nested exception is java.lang.ClassNotFoundException: src/main/java/controller/TestController

## 无法加载配置类
Cannot find class [src/main/java/controller/TestController] for bean with name '/hello.do' defined in file [D:\doc\commit\Nwgid\springdemo03ssm\target\classes\spring-web.xml]; nested exception is java.lang.ClassNotFoundException: src/main/java/controller/TestController

## java.lang.NoClassDefFoundError:javax/servlet/jsp/jstl/core/Config
TestController 已经打印日志 ， 返回Modelandview错误

## java.lang.ClassNotFoundException: controller/TestController

```xml
    <bean id="/hello.do" class="controller.TestController"></bean>


```

## http://localhost:9090/springdemo03ssm/hello.do    404
/springdemo03ssm/page/hello.jsp
HTTP Status 404 - /springdemo03ssm/page/hello.jsp

@RestController
@RestController 注解相当于@ResponseBody ＋ @Controller合在一起的作用, 需要注意的是
Spring 扫描bean
<context:component-scan base-package="controller"/>  

## jsp  不渲染Etl
```xml
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<%@ page isELIgnored="false" %>

```
## java.io.FileNotFoundException: class path resource [spring.xml] cannot be opened because it does not exist


## IDEA关于Lombok的一些问题( java: 找不到符号 符号)的解决
pom.xml 提高Lombok 版本