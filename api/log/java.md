# Java

## jar
-  制作只含有字节码文件的jar包
```mf
META-INF
Hello.class
Main-Class: Hello 
```

```java
1 class Hello{
2     public static void main(String[] agrs){
3         System.out.println("hello");
4     }
5 }
```
```bash
javac Hello.java 
jar -cvf hello.jar Hello.class 
 jar -cvfm hello.jar META-INF\MENIFEST.MF Hello.class Tom.class #Hello使用Tom

javac Hello.java -d target 

# 打包 war 包
jar -cvf <WAR包名称>.war -C <项目根目录>/ .
```
- 制作含有jar文件的jar包

制作含有jar文件的jar包
```bash
  javac -cp tom.jar Hello.class 
jar -cvfm 命名 MENIFEST文件 要打包的文件1 要打包的文件2
```
- 制作含有资源文件的jar包
```java
 InputStream is = hello.getClass().getResourceAsStream("text.txt");

```

```bash
jar -tf xxx.jar 可以查看 jar 文件内容

jar -xf xxx.jar 可以解压 jar 文件

```