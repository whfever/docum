# C++_practice

## 源

| I        | e                                                            |
| -------- | ------------------------------------------------------------ |
| 书籍推荐 | ![image-20230131113243097](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230131113243097.png) |

### C++对c的增强

增加面向对象的机制，扩充功能

## 基

主函数（入口），命名空间(区分)，预处理编译，返回值，标识符和关键字（自定义）

常量与变量（作用域）

### 标准

#### 标准命名空间std

standard 标准类

![image-20230201095611521](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201095611521.png)

引用命名空间的成员using

作用域限定符：：



#### 标准输入输出

cin,cout 》《流对象

get()  getline()

put()  printf()

![image-20230202104733608](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230202104733608.png)

### 函数



声明，调用，输出结果的封装

#### 初始化

定义，声明，调用

传参，返回值

#### 编译预处理

|          |        |        |
| -------- | ------ | ------ |
| #define  | 宏定义 | 短字符 |
| #include |        |        |
|          |        |        |



#### 作用域

变量，函数，文件

#### 重载

函数名多用，![image-20230201162108370](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201162108370.png)

#### 递归，内联



内联函数

![img](https://res.weread.qq.com/wrepub/epub_26846805_551)

### 数据类型

数据基本类型，结构体类型（加），类（操作），枚举，共用体，推断

类型别名

![image-20230201101543531](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201101543531.png)

#### 声明和定义

exetern

#### 内存空间区别

静态，栈

#### 对象的生命周期



### 运算符和表达式

算术运算符的优先级，关系，位，逻辑，条件，赋值



## 核

### 流程控制

<u>*流程控制：程序语句执行的次序*</u>

控制语句，空语句，执行语句

![img](https://res.weread.qq.com/wrepub/epub_26846805_301)

赋值语句和执行表达式

![image-20230201150503487](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201150503487.png)

#### 顺序

#### 选择if,switch



#### 循环for while

顺序：开始，条件，结束-》当

do while

#### 跳转

| statement | effect       |              |
| --------- | ------------ | ------------ |
| goto      | 无条件转移   |              |
| break     | 结束循环     | 循环或switch |
| continue  | 终止当前循环 |              |

### 数组引用，指针

#### 数组

1. 一二维数组的定义与初始化

2. 字符数组

结束标志计算长度

![image-20230201153936849](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201153936849.png)

3. 结构体数组

#### 引用

变量的第二个标签

作用，增加函数传递数据的功能

![image-20230201154555957](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201154555957.png)

#### 指针

1. 数组指针 指针数组

![image-20230201155731874](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201155731874.png)

2. 字符串指针

3. const 指针

   修改指针指向的数据（常量），指向其他地方（变量）；都不能修改用const

### 类和对象

#### 初始化

定义，创建，访问

<u>*成员函数*</u> ，访问修饰符

构造函数，析构函数（释放资源）

#### 动态内存

![image-20230201165903898](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201165903898.png)

new delete 申请和释放堆中分配的内存空间

new|malloc

![image-20230201170358607](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230201170358607.png)

this指针，static 静态成员

静态成员在类的所有对象中是共享的

静态成员函数

常量成员：共享，不被修改

友元函数：类的友元函数是定义在类外部，但有权访问类的所有私有（private）成员和保护（protected）成员。尽管友元函数的原型有在类的定义中出现过，但是友元函数并不是成员函数。

友元类



#### 继承与派生

基类，派生类

继承方式：public private protected

基类与派生类的转换，切片

多重继承

#### 多态与重载

多态：一个事物的多种形态

![image-20230202100646232](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230202100646232.png)

virtual 希望派生类改变的函数



抽象基类：实例化会编译错误

函数的重载：一名多用

运算符的重载：定义函数

## 核2

### 文件操作（数据存储）

#### 文件流类，对象

istream  ostream fstream



ofstream ,ifstream:打开只读

打开关闭文件 open(),close()

读写文本文件：

1. 流运算符>> <<
2. 流函数 put() get() getline()

读写二进制文件：

read();write();

文件的数据定位：文件指针，随机读写

![img](https://res.weread.qq.com/wrepub/epub_26846805_910)

### 容器（数据类型）

![image-20230202114631022](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230202114631022.png)

#### 顺序容器

|        |                                                  |                  |
| ------ | ------------------------------------------------ | ---------------- |
| vector | 数组，[]下标访问，高效的尾部的插入和删除         |                  |
| list   | 双向链表，允许快速的插入和删除，随机访问比较慢   |                  |
| deque  | 数组，双向队列，允许在头部，尾部，快速插入和删除 | 多个连续的存储块 |

#### 关联容器

关联容器：通过关键字保存和访问

##### map

标准模板库的关联容器map,不允许重复的关键字

#include<utility> pair

map使用：插入，遍历，查找，删除

1. insert pair
2. insert value_type
3. 数组

##### set

set:单纯键的集合

#### 容器适配器

| adaptive       | statement1     |                          |
| -------------- | -------------- | ------------------------ |
| stack          | 先进后出       |                          |
| queue          | 先进先出       |                          |
| priority_queue | 带优先级的删除 | 包含最大值的元素位于队首 |

正确选择容器：

![image-20230202155723531](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230202155723531.png)

通用容器如下：![img](https://res.weread.qq.com/wrepub/epub_26846805_1015)

### 模板（代码重用率）

C++模板：支持参数化多态的工具，泛型编程的基础；功能相似，数据类型不同的函数或类设计为通用的函数或类模板

![image-20230202160803145](https://cdn.jsdelivr.net/gh/whfever/figure/2022/science/image-20230202160803145.png)

#### 语法

```c++
//函数模板
template <T >
int add(T x,T y){
return x+y;
}
//类模板
template <class T>
```

#### 函数模板

通用函数

#### 类模板

![img](https://res.weread.qq.com/wrepub/epub_26846805_1045)

#### 模板特化

特殊类型的数据处理

函数模板特化：

![img](https://res.weread.qq.com/wrepub/epub_26846805_1045)

类模板特化：

![img](https://res.weread.qq.com/wrepub/epub_26846805_1058)

### 标准库

可扩展的基础性框架：成本，质量，编程风格

#### 迭代器

<iterator>

#### STL（算法）

<algorithm>提供算法

##### 编辑

数据编辑算法：对容器内的数据进行填充，赋值，合并，删除等操作

|           |                                                             |
| --------- | ----------------------------------------------------------- |
| fill()    | ![img](https://res.weread.qq.com/wrepub/epub_26846805_1074) |
| copy()    |                                                             |
| merge()   |                                                             |
| remove()  |                                                             |
| replace() |                                                             |

##### 查找

|            |                        |
| ---------- | ---------------------- |
| find（）   | 返回值是一个迭代器类型 |
| search（） |                        |
|            |                        |

##### 比较

|            |                          |
| ---------- | ------------------------ |
| equal()    |                          |
| mismatch() | 两个元素第一个不同的地方 |
|            |                          |

##### 排序

|           |      |
| --------- | ---- |
| sort()    |      |
| reverse() |      |
|           |      |

##### 计算

|                     |      |
| ------------------- | ---- |
| count（）           |      |
| max_element() ,min_ |      |
| for_each()          |      |
| transform()         |      |

#### 字符串库

|            |                          |
| ---------- | ------------------------ |
| strcat_s() | 指定返回字符串str1的大小 |
| strcpy_s() | 保证缓冲区大小           |
| strcmp()   | 比较，返回0，+，-        |
| strlen()   | 实际长度                 |

#### 字符串类

String 重载运算符，连接，索引和复制

### 异常

#### 程序错误

|            |                          |
| ---------- | ------------------------ |
| 语法错误   | 编译器按顺序查找语法错误 |
| 逻辑错误   |                          |
| 运行时错误 |                          |

#### 异常处理

| 异常处理语句块 |          |
| -------------- | -------- |
| throw          | 抛出异常 |
| try  catch     | 捕获异常 |
|                |          |

## 应

### 游戏：推箱子

|          |                                                |
| -------- | ---------------------------------------------- |
| 图画模块 | 二维数组转换为图形模式                         |
| 移动模块 | 键盘数据改变二维数据的值，实现任务和箱子的移动 |
| 菜单模块 |                                                |

### 金融：ATM

|          |                                  |
| -------- | -------------------------------- |
| 业务模块 | 查询，取款，存款，转账，修改密码 |
| 菜单模块 |                                  |
|          |                                  |

### 移动互联网：局域网聊天

利用socket，多线程，TCP协议实现局域网聊天系统的开发

服务端：客户端
