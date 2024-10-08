# Problem
## Java
### package打出了jar包idea看不到的问题
ctrl  shift  E ：tree show  exlued files   

### 设置环境变量
printenv |grep  -i java_home

### git bash连接mysql卡死问题
winpty mysql -u root -p

### git bash 启动慢
git config --global core.bigFileThreshold 1G

### egde 卡顿
--disk-cache-size-2147483648
### git bash 扩展

winpty tree.com

### 自动导入
general  anto-import

### 打包不含依赖

在项目结构工件中引入依赖


## Xx.class.getClassLoader().getResource()出现空指针异常

```java
public class Test {
   public static void main(String[] args) {
       String path = Test.class.getClassLoader().getResource("1.xml").getPath();
       System.out.println(path);
   }
}
```

### 自动补全
keymap   ALT /


### 版本
```java
Exception in thread "main" java.lang.UnsupportedClassVersionError: org/apache/catalina/startup/Bootstrap has been compiled by a 

```
Tomcat >8.0

### web.xml
```java
java.io.FileNotFoundException: class path resource xxxxxx cannot be opened because it does not exist

```

```xml
<!DOCTYPE web-app PUBLIC
        "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
        "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
    <display-name>Archetype Created Web Application</display-name>
    <!--1.配置前置控制器 -->
    <servlet>
        <servlet-name>SpringDispatcherServlet</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <!-- 引入spring配置文件 -->
            <param-value>classpath:spring-*.xml</param-value>
        </init-param>
    </servlet>
    <servlet-mapping>
        <servlet-name>SpringDispatcherServlet</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!--2.欢迎文件-->
    <welcome-file-list>
        <welcome-file>/page/index.jsp</welcome-file>
    </welcome-file-list>
</web-app>
```

##  DispatcherServlet
org.springframework.web.servlet.DispatcherServlet.noHandlerFound No mapping for GET /####/


```XML
<resources>
<resource>
  <directory>src/main/java</directory>
  <!--所在的目录-->
  <includes>
    <!--包括目录下的.properties,.xml 文件都会被扫描到-->
    <include>**/*.properties</include>
    <include>**/*.xml</include>
  </includes>
  <filtering>false</filtering>
</resource>
<resource>
  <directory>src/main/resources</directory>
  <includes>
    <include>**/*.*</include>  </includes>/resource></resources>

```

##  XML schema
Configuration problem: Unable to locate Spring NamespaceHandler for XML schema namespace [http://www.springframework.org/schema/tx]

```xml
   <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-tx</artifactId>
      <version>3.2.5.RELEASE</version>
    </dependency>
```

