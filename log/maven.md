# Maven
## plugin
```xml
<plugin>
  <groupId>org.apache.maven.plugins</groupId>
  <artifactId>maven-surefire-plugin</artifactId>
  <version>3.0.0-M5</version>
</plugin>


```
## packages

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <version>3.2.4</version>
    <configuration>
        <!-- put your configurations here -->
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
        </execution>
    </executions>
</plugin>

<plugin>
     <artifactId>maven-assembly-plugin</artifactId>
     <configuration>
     <minimizeJar>true</minimizeJar>
         <archive>
             <manifest>
                 <mainClass>com.example.demo.DemoApplication</mainClass>
             </manifest>
         </archive>
         <descriptorRefs>
             <descriptorRef>jar-with-dependencies</descriptorRef>
         </descriptorRefs>
     </configuration>
     <!--下面是为了使用 mvn package命令，如果不加则使用mvn assembly-->
     <executions>
         <execution>
             <id>make-assemble</id>
             <!-- 绑定到package生命周期 -->
             <phase>package</phase>
             <goals>
                 <!-- 只运行一次 -->
                 <goal>single</goal>
             </goals>
         </execution>
     </executions>
 </plugin>
```
### surefire
> regex 排除 跳过构建  report
```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <version>${maven-surefire-plugin.version}</version>
    <configuration>
        <useUnlimitedThreads>true</useUnlimitedThreads>
        <rerunFailingTestsCount>1</rerunFailingTestsCount>
        <parallel>methods</parallel>
        <forkedProcessExitTimeoutInSeconds>2</forkedProcessExitTimeoutInSeconds>
    </configuration>
</plugin>
```

## 生命周期

```bash
# 1.validate 验证项目是正确的，所有必要信息都可用，所有依赖项都已下载。
# 2.compile 编译项目源代码。
# 3.test 使用适当的单元测试框架对编译后的源代码运行测试。
# 4.package 将编译后的代码打包成可发布格式，例如JAR。
# 5.install 将包安装到本地存储库中，以便在本地的其他项目中作为依赖项使用。
# 6.deploy 将最终包复制到远程存储库，以便与其他开发人员和项目共享。

```


## 重复下载 快照机制
版本号以[-SNAPSHOT]结尾，每次构建都会去远程仓库检查最新版本。