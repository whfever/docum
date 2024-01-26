# growth
全栈应用开发 精益实践
## 基础知识
1. 过去 PHP 是主流的开发，不过现在也是，PHP 为 WEB ⽽⽣。有⼀天 Ruby on Rails出现了，⼀切就变了，变得⾼效，变得更 Powerful。MVC ⼀直很不错，不是么？于是越来越多的框架出现了，如 Django，Laravel 等等。不同的语⾔有着不同的框架，Javascript上也有着合适的框架，如 Angular js。不同语⾔的使⽤者们⽤着他们合适的⼯具，因为学习新的东西，对于多数的⼈来说就是⼀种新的挑战。
**在学⾯向对象语⾔的时候，⼈们很容易把程序写成过程式的。**
2. 如果你不试着去写点博客、整理资料、准备分享，那么你可能并没有意识到你缺少了多少东西。虽然你已经有了很多的实践，**然并卵**。
因为你⼀直在完成功能、完成⼯作，你总会有意、⽆意地漏掉⼀些知识，⽽你也没有意识到这些知识的重要性。
学习内容留存率：  
   1. 被动学习 听讲5 阅读10 视听20  演示30
   2. 主动学习 讨论50 实践75 教授给他人90
   3. 所以，如果下次有⼈问你如果学⼀门新语⾔、技术，那么答案就是写⼀本书。
⽽学习⼀门新的技术的最好实践就是⽤这门技术对现有的系统⾏重写。

3. 我们可以将 Javascript 理解为解决问题的语⾔，html 则是前端显⽰，css 是配置⽂件，这样的话，我们会在那之后学会成为⼀个近
乎专业的程序员
4. css 层叠样式表   选择器
5. javascript

```js
tang=4;
num=3;
document.write(tang*num);
// 这才能叫得上是程序设计，或许你注意到了 “;” 这个符号的存在，我想说的是这是
// 另外⼀个标准，我们不得不去遵守，也不得不去 fuck。
```
### js
1. Object : document.write(typeof document);

```js
var IO=new Object();
function print(result){
document.write(result);
};
IO.print=print;
IO.print("a obejct with function");
IO.print(typeof IO.print);


var store={};
又或者是
var store=new Object{};

```
2. 原型继承
```js
var Chinese=function(){
this.country="China";
}
var Person=function(name,weight,height){
this.name=name;
this.weight=weight;
this.height=height;
this.futrue=function(){
return "future";
}
}
Chinese.prototype=new Person();
var phodal=new Chinese("phodal",50,166);
document.write(phodal.country);

```

3. 三部分
   完整的 Javascript 应该由下列三个部分组成:
• 核⼼ (ECMAScript)——核⼼语⾔功能
• ⽂档对象模型 (DOM)——访问和操作⽹页内容的⽅法和接⼜
• 浏览器对象模型 (BOM)——与浏览器交互的⽅法和接⼜

> ，但是 JS 真正强⼤的，或者说我们最需要的可能就是对 DOM 的操作，这也就是为什么 jQuery 等库可以流⾏的原因之⼀，
```js
var para=document.getElementById("para");
para.style.color="blue";

```

• 我们可以创建⼀个对象或者函数，它可以包含基本值、对象或者函数。
• 我们可以⽤ Javascript 修改页⾯的属性，虽然只是简单的⽰例。
• 我们还可以去解决实际的编程问题。

过去⼈们在 Java 上花费了很多的时间，或在架构上，或在语⾔上，或在模式上。由于这些投⼊，都给了⼈们很多的启发。这些都可以⽤于新的语⾔，新的设计，毕竟没有什么技术是独⽴于旧的技术产⽣出来的。

## 编码
1. 那些所谓的写作逻辑对编程的影响
• 早期的代码是以⾏数算的，⽂章是以字数算的
• 代码是写给⼈看的，⽂章也是写给⼈看的
• 编程同写作⼀样，都由想法开始
• 代码同⽂章⼀样都可以堆砌出来 (ps: 如本⽂)
• 写出好的⽂章不容易，需要反复琢磨，写出好的代码不也是如此么
• 构造⼀个类，好⽐是构造⼀个⼈物的性格特点，多⼀点不⾏，少⼀点又不全
• 代码⽣成，和⽣成诗⼀样，没有情感，过于机械化

**真正的想法都在脑⼦⾥，⽽不在纸上，或者 IDE ⾥。**
### 重构
```java
public class extract {
private String _name;
void printOwing(double amount){
printBanner();
//提炼方法
System.out.println("name:" + _name);
System.out.println("amount" + amount);
}
private void printBanner() {
}
}
```

### SEO
SEO 与编程的最⼤不同之处在于: 编程的核⼼是技术，SEO 的核⼼是内容。
内容才是 SEO 最重要的组成部分，这也就是腾讯复制不了的东西。
