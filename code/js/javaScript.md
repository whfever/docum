# Js

## What

> Job ，EcmaScript Specifition 

![image-20230802103850769](https://article.biliimg.com/bfs/article/e1eeb7287ee7f5a04f999f9a0c726ae22ce89aa7.png)

```
console.log('hello');
```

### Development Environment

> Node  Vscode





## Basic

### In  Browswer

### In  Node

```
node index.js
```



### Seperation





## Variables

> 声明：let  ， 
>
> 使用： 

### Constants

```
const interRate=1；
```





### Primitive Types

![image-20230802110754408](https://article.biliimg.com/bfs/article/efbc17b149a25cc791eb929f877ba1d42cd5623f.png)

```
typeof name
```




### Functions



```
function greet(){
	console.log('hello');
}
function greet(name,lastName){
	console.log('hello'+namea);
}
```

#### arrow function

> function {return} = >

![image-20230802132432384](https://article.biliimg.com/bfs/article/d5d9b798539049387f7330cdb5af598606678418.png)

> this window

![image-20230805164220168](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805164220168.png)

#### nested function





type  of functions

![image-20230802113356447](https://article.biliimg.com/bfs/article/6e608467faaabfa2efac5044e71fa793797266ab.png)



#### settimeout

![image-20230802141900011](https://article.biliimg.com/bfs/article/b96066258725acb5198f8fffaa86bf009964bd1c.png)

```
cleartimeout（）；
```





#### setInterval

![image-20230802142224153](https://article.biliimg.com/bfs/article/476e21cb486b7144656a218e0b991fd59bf69060.png)





#### asynchrnous

![image-20230802142912673](https://article.biliimg.com/bfs/article/4a72f2a76cd64314a764379f656b3055971cb9e8.png)

> timer

![image-20230802143302263](https://article.biliimg.com/bfs/article/519e5f3ae344ef992520dda16fade4f8a3f5bd28.png)

> async functions

![image-20230802145748849](https://article.biliimg.com/bfs/article/89a05128d2cbe2f7da5a419dd0db5b7c3b500c54.png)

### Objects

> Dynamic typing

> . 
> []

![image-20230802133449200](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230802133449200.png)



![image-20230802111903172](https://article.biliimg.com/bfs/article/2c6e47f54c33e8dabed0bfa95f6ddb9950ac1516.png)

```
let person={
	name:'<'
	age:1;
}
```

#### arrays

```0
let colors=['red','blue']
colors[2]='green';
colors[2]= 2;
console.log(colors[0])
//原型
console.log(colors.length)

```

#### Object  argments

![image-20230802140837665](https://article.biliimg.com/bfs/article/85c228ece2f0a3dd58f6379eaec58269776c8964.png)

#### Array  of objects

#### anoymous objects 

![image-20230802141307406](https://article.biliimg.com/bfs/article/b22afbe3178ce6266ec8a502a6f25ed573ae57c5.png)

#### Date

![image-20230802142459206](https://article.biliimg.com/bfs/article/024c37edfb1ae9563ab0e23c7278dd8fa72ef19f.png)

> cloack program

![image-20230802142659900](https://article.biliimg.com/bfs/article/581f86e607430e7c403a84f69dcbaf701c975733.png)

## Error  Hadleing

> try  catch finally

![image-20230802141657926](https://article.biliimg.com/bfs/article/167428f70ef463e0ad5a8f4b95627ea9a5106f39.png)

# advance

while  for  

if switch 





## Spread operator

![image-20230802125214363](https://article.biliimg.com/bfs/article/5df4d46bb346d4627dc85ce2b5011c7150f78ad6.png)



![image-20230802125245669](https://article.biliimg.com/bfs/article/ea48d63d6846f4a807f57d637fd6f48eee23c350.png)



![image-20230802125346407](https://article.biliimg.com/bfs/article/f5f9ca02afd143bd6c52eda76a24dbdc02e16e1e.png)



![image-20230802125452664](https://article.biliimg.com/bfs/article/9a2ac303b01a76441f5e28ac8f81bc1f4099a58b.png)

## format currency



## template literals

```
${}
```

![image-20230802124634429](https://article.biliimg.com/bfs/article/e07b82105099986ce3c7fc9b287f006bfa0c01f5.png)



## Promise

|                              |                                                              |      |
| ---------------------------- | ------------------------------------------------------------ | ---- |
| then  catch                  | ![image-20230802145031869](https://article.biliimg.com/bfs/article/63834db19a0ad4ec1d022f7bf5e9047030dd968a.png) |      |
| sertTimeout（resolve，time） | ![image-20230802145323002](https://article.biliimg.com/bfs/article/a824449aa9264cda4637cb34c4a1ca63a2269751.png) |      |
| sertTimeout（resolve，time） | ![image-20230802145446374](https://article.biliimg.com/bfs/article/3b559fa29b54033cecd926e077a87b00e8976c17.png) |      |

### await

![image-20230802150127569](https://article.biliimg.com/bfs/article/d185e9ec8dbb21bca566ac05feda5e5d57da7a83.png)

## Object

### Array

|         |                                                              |      |
| ------- | ------------------------------------------------------------ | ---- |
| foreach | ![image-20230802131130369](https://article.biliimg.com/bfs/article/7fb9f42d7e07fe247f22ecac69d0c05f2ceee0d7.png) |      |
| map     | ![image-20230802131500213](https://article.biliimg.com/bfs/article/01a3219abdcdadf4a3c8daa6ad4d5163e567c2a2.png) |      |
| filter  | ![image-20230802131600590](https://article.biliimg.com/bfs/article/5e91ac1115b6f449874687cfde1261b06fa5dce1.png) |      |
| reduce  | ![image-20230802131730914](https://article.biliimg.com/bfs/article/4413c2277a5cc65b5fc2ca52c150ab60102137fc.png) |      |
| sort    | ![image-20230802131854226](https://article.biliimg.com/bfs/article/06faf6cba4f1cfd1fea81e94a205a8f5c6223946.png) |      |
| shuffle |                                                              |      |
|         |                                                              |      |





### Math







### Map

![image-20230802133240482](https://article.biliimg.com/bfs/article/8b787eda530b88fb8358ce91ac330aa8df1db768.png)

```


```

## Modules



![image-20230802150442615](https://article.biliimg.com/bfs/article/7fd50f90bd8f0d86e1b6ad89262850c5d67c7715.png)

# Dom

![image-20230802150702221](https://article.biliimg.com/bfs/article/9e0fc3712c473f96b5bee48874505eeb524c70b9.png)





## element selectors

![image-20230802150956882](https://article.biliimg.com/bfs/article/fdc956c85da6d6517601596d7b229df52d703b0a.png)



## Dom traversal

![image-20230802151203790](https://article.biliimg.com/bfs/article/413f762a90ac96ab8f086835c19ae1126937239a.png)



### window

![image-20230802153325979](https://article.biliimg.com/bfs/article/ffbbfb902890d23b8f8951543b5251f0739b7d98.png)



## cookie

![image-20230802154103251](https://article.biliimg.com/bfs/article/2cd3bc4b2429786a93b5d91caa18a46a58f9849f.png)

## add change html



![image-20230802151334494](https://article.biliimg.com/bfs/article/9dcda4c5b8b30e40c603dc7286c81d882666ccee.png)



### show hide html

![image-20230802152143076](https://article.biliimg.com/bfs/article/23024435fa09e16c45258dc7ed75b94b99581a24.png)

## add change css

![image-20230802151454952](https://article.biliimg.com/bfs/article/c6b6d0bb1806cd0d48b8e9c75a48f4cf99b24b2c.png)



### animation

![image-20230802152636654](https://article.biliimg.com/bfs/article/54f2ee7ead0235b19a07ee5bd0b638b12205c7f6.png)

## canvas

![image-20230802153007874](https://article.biliimg.com/bfs/article/04f17a1ce9aaa5c63d9d07578ac90eb561bd15e6.png)

## events

![image-20230802151842287](https://article.biliimg.com/bfs/article/586e79817ded25d5a1b6181108862951b4956fbb.png)

### addEventlistener

![image-20230802152038327](https://article.biliimg.com/bfs/article/8069fca3808b97961c949aaa0ce2ff9f4e36db73.png)



### detect key presses

![image-20230802152451822](https://article.biliimg.com/bfs/article/3c8b90ad94602e0ae0043352c4a9f73545a6fef9.png)



# JsObject

## CallBack

> JavaScript 按从上到下的顺序运行代码。但是，在有些情况下，必须在某些情况发生之后，代码才能运行（或者说必须运行），这就不是按顺序运行了。这是异步编程。

![image-20230802130556471](https://article.biliimg.com/bfs/article/f2e1a34549c7c6088538dca37ff36e522c7bcc19.png)



## this

![image-20230802133713750](https://article.biliimg.com/bfs/article/09d2158c00296e3a6861597d0c6916f0421a3ecc.png)





## class

![image-20230802134521984](https://article.biliimg.com/bfs/article/17e9346982d6cec36ae273f3912a4387e84d8eb7.png)





### constructor

![image-20230802134615823](https://article.biliimg.com/bfs/article/8b8c062b47f6b1301623b2d75f0df15cfcc7897f.png)





### static

![image-20230802134939234](https://article.biliimg.com/bfs/article/4f8f4b1b2c912d60812d0c887b16a8ca0719ba04.png)



![image-20230802135653729](https://article.biliimg.com/bfs/article/b597982219d4fe6993dee742cf21be9f30756405.png)





### inheritance

![image-20230802135849377](https://article.biliimg.com/bfs/article/53c7fc97fcf3860068bf9faf0b83b19f733cb27d.png)



#### super

![image-20230802140058413](https://article.biliimg.com/bfs/article/b66ba15414b6c15085800800c16b624189d55f9f.png)



#### get set

![image-20230802140432074](https://article.biliimg.com/bfs/article/601f98dae5d69dba85ad5423a93c426f60f9ec11.png)





# Jsquery

## 原型

> js  constrouctor  浪费内存

![image-20230805153746567](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805153746567.png)



> 原型作用： 共享内存

![image-20230805154157668](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805154157668.png)



### prototype



> not  constructor

```
    
    constructor： star
    
    
```

![image-20230805154449884](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805154449884.png)



![image-20230805154843948](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805154843948.png)



###   _ proto_

> 示例对象可是指向原型对象的原因：  有一个对象原型属性指向原型对象

```
  ldh =new  Star（）；
  ldh._proto_=Star.prototype
```

![image-20230805155315967](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805155315967.png)

—

**![image-20230805155545174](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805155545174.png)**



## 闭包

> 实现数据的私有

![image-20230805163451503](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805163451503.png)

## 变量，函数提升

> var  作用域提升到最前面

![image-20230805163723129](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805163723129.png)

## Gc

- 引用计数法
- 标记清除法



> 标记清除法：解决了内部循环引用，全局没有引用  根部寻找

![image-20230805163155118](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805163155118.png)

# Apply



## debounce

![image-20230805161852022](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805161852022.png)

  

```
if (timer) cleartimeout(){

}
timer =settimeout(f(){},t)
```



## throttle

> 节流：单位时间内只执行一次

![image-20230805161803821](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805161803821.png)

# jsVscode

## 代码规范  eslint prettier

> 校验   代码格式化
