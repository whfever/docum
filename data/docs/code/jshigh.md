# js gj

从 JavaScript 的起源开始，逐步讲解到新出现的技术，其中重点介绍 ECMAScript 和 DOM 标准。在此基础上，接下来的各章揭示了 JavaScript 的基本概念，包括类、期约、迭代器、代理，等等。另外，书中深入探讨了客户端检测、事件、动画、表单、错误处理及 JSON。本书同时也介绍了近几年来涌现的重要新规范，包括 Fetch API、模块、工作者线程、服务线程以及大量新 API

- 后来 Ajax 因为 jQuery 火了而变得更加流行，这才打开了通向新世界的大门，可靠、稳定的应用程序应运而生
- 前端模型、数据绑定、路由管理、反应式视图，全都爆发出来了。

1. 1995 年，JavaScript 问世。当时，它的主要用途是代替 Perl 等服务器端语言处理输入验证
2. JavaScript 终于踏上了标准化的征程 。也就是 ECMAScript（发音为“ek-ma-script”）这个新的脚本语言标准
3. 万维网联盟（W3C，World Wide Web Consortium）开始了制定 DOM 标准的进程
4. IE3 和 Netscape Navigator 3 提供了浏览器对象模型（BOM） API，用于支持访问和操作浏览器的窗口。

JavaScript 是一门用来与网页交互的脚本语言，包含以下三个组成部分。
 ECMAScript：由 ECMA-262 定义并提供核心功能。
 文档对象模型（DOM）：提供与网页内容交互的方法和接口。
 浏览器对象模型（BOM）：提供与浏览器交互的方法和接口《

## base

在 JavaScript 早期，网景公司的工作人员希望在将 JavaScript 引入 HTML 页面的同时，不会导致页面在其他浏览器中渲染出问题

1. 将 JavaScript 插入 HTML 的主要方法是使用\<script>\元素

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Example HTML Page</title>
  </head>
  <body>
    <!-- 这里是页面内容 -->
    <!-- defer 推迟执行 -->
    <!-- async 异步执行 -->
    <script defer src="example1.js"></script>
    <script async src="example2.js"></script>
  </body>
</html>

<!-- 动态添加脚本 -->

let script = document.createElement('script'); script.src = 'gibberish.js';
document.head.appendChild(script);
```

&lt; = <

```xhtml
<script type="text/javascript">
  function compare(a, b) {
  if (a &lt; b) {
  console.log("A is less than B");
  } else if (a > b) {
  console.log("A is greater than B");
  } else {
  console.log("A is equal to B");
  }
  }
</script>
```

## ECMAScript

1. 注释，语句；，关键字
2. 变量
   1. 数据类型 ：：Undefined、Null、Boolean、Number、String 和 Symbol Object 复杂类型
      **数值转换**
   2. 操作符：操作符，包括数学操作符（如加、减）、位操作符、关系操作符和相等操作符等。
      1. 一元，按位与，或，异或
      2. 赋值 逗号
   3. 作用域，垃圾回收
      1. 原始值（primitive value）就是最简单的数据，引用值（reference value）则是由多个值构成的对象
      2. 执行上下文与作用域
      3. 垃圾回收：标记清楚，引用计数
         1. 因为 const 和 let 都以块（而非函数）为作用域，所以相比于使用 var，使用这两个新关键字可能会更早地让垃圾回
            收程序介入，尽早回收应该回收的内存。在块作用域比函数作用域更早终止的情况下，这就有可能发生

```js
当然，解决方案就是避免 JavaScript 的“先创建再补充”（ready-fire-aim）式的动态属性赋值，并在
构造函数中一次性声明所有属性，如下所示：
function Article(opt_author) {
 this.title = 'Inauguration Ceremony Features Kazoo Band';
 this.author = opt_author;
}
let a1 = new Article();
let a2 = new Article('Jake');
这样，两个实例基本上就一样了（不考虑 hasOwnProperty 的返回值），因此可以共享一个隐藏类，
从而带来潜在的性能提升。
```

            2.

1. 语句
   1. if switch
   2. while do while
   3. for of - 可迭代 for in
   4. 标签语句
      1. start: for (let i = 0; i < count; i++) { console.log(i); }
   5. break continue
   6. with 1. 上面代码中的每一行都用到了 location 对象。如果使用 with 语句，就可以少写一些代码：
      with(location) { let qs = （location.）search.substring(1); let hostName = hostname; let url = href; }
1. 函数

```js
// var块作用域 let函数作用域
if (true) {
  var name = "Matt";
  console.log(name); // Matt
}
console.log(name); // Matt
if (true) {
  let age = 26;
  console.log(age); // 26
}
console.log(age); // ReferenceError: age 没有定义

// 全局声明
var name = "Matt";
console.log(window.name); // 'Matt'
let age = 26;
console.log(window.age); // undefined
```

const 声明

```js
const 的行为与 let 基本相同，唯一一个重要的区别是用它声明变量时必须同时初始化变量，且
尝试修改 const 声明的变量会导致运行时错误。
```

null 是一个假值

```javascript
let message = null;
let age;
if (message) {
  // 这个块不会执行
}
if (!message) {
  // 这个块会执行
}

if (age) {
  // 这个块不会执行
}
if (!age) {
  // 这个块会执行
}

// 模板字面量标签函数
let a = 6;
let b = 9;
function simpleTag(strings, aValExpression, bValExpression, sumExpression) {
  console.log(strings);
  console.log(aValExpression);
  console.log(bValExpression);
  console.log(sumExpression);
  return "foobar";
}
let untaggedResult = `${a} + ${b} = ${a + b}`;
let taggedResult = simpleTag`${a} + ${b} = ${a + b}`;
// ["", " + ", " = ", ""]
// 6
// 9
// 15
console.log(untaggedResult); // "6 + 9 = 15"
console.log(taggedResult); // "foobar"
```

## 引用类型

1. 基本引用类型

```js
let now = new Date();
let stop = Date.now(),
  result = stop - start;
/**
 * 2. RegExp
 *
 *  */
// 匹配字符串中的所有"at"
let pattern1 = /at/g;
// 匹配第一个"bat"或"cat"，忽略大小写
let pattern2 = /[bc]at/i;
// 匹配所有以"at"结尾的三字符组合，忽略大小写
let pattern3 = /.at/gi;

/**
 *
 *3.包装类型
 */

let numberObject = new Number(10);
let stringObject = new String("hello world");

/**
 * 4.单例内置对象
 * Global 对象隐式。包括 isNaN()、isFinite()、parseInt()和 parseFloat()，实际上都是 Global 对象的方法。除
了这些，Global 对象上还有另外一些方法

- window
-Math
 */
let uri = "http://www.wrox.com/illegal value.js#start";
// "http://www.wrox.com/illegal%20value.js#start"
console.log(encodeURI(uri));
// "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.js%23start"
console.log(encodeURIComponent(uri));

let uri = "http%3A%2F%2Fwww.wrox.com%2Fillegal%20value.js%23start";
// http%3A%2F%2Fwww.wrox.com%2Fillegal value.js%23start
console.log(decodeURI(uri));
// http:// www.wrox.com/illegal value.js#start
console.log(decodeURIComponent(uri));

eval("console.log('hi')");
//上面这行代码的功能与下一行等价：
console.log("hi");

let num = Math.floor(Math.random() * 10 + 1);
```

2. 集合引用类型
1. object
1. Array ： 栈方法 队列方法 排序 搜索 迭代 归并
   1.  every()：对数组每一项都运行传入的函数，如果对每一项函数都返回 true，则这个方法返回 true。
       filter()：对数组每一项都运行传入的函数，函数返回 true 的项会组成数组之后返回。
       forEach()：对数组每一项都运行传入的函数，没有返回值。
       map()：对数组每一项都运行传入的函数，返回由每次函数调用的结果构成的数组。
       some()：对数组每一项都运行传入的函数，如果有一项函数返回 true，则这个方法返回 true。
      这些方法都不改变调用它们的数组
1. Map WeakMap
1. Set

```javascript
/**
 * 1. object
 * 2. Array
 *
 */
let person = new Object();
person.name = "Nicholas";
person.age = 29;
let person = {
  name: "Nicholas",
  age: 29,
};

let colors = new Array(3);
let colors = Array(3); // 创建一个包含 3 个元素的数组
let colors = ["red", "blue", "green"];
const a1 = [1, 2, 3, 4];
const a2 = Array.from(a1, (x) => x ** 2);
const a3 = Array.from(
  a1,
  function (x) {
    return x ** this.exponent;
  },
  { exponent: 2 }
);
console.log(a2); // [1, 4, 9, 16]
console.log(a3); // [1, 4, 9, 16]
console.log(Array.of(1, 2, 3, 4)); // [1, 2, 3, 4]

//迭代器
for (const [idx, element] of a.entries()) {
  alert(idx);
  alert(element);
}
// 0
// foo
// 1
// bar
// 2
// baz
// 3
// qux
const zeroes = [0, 0, 0, 0, 0];
// 用 5 填充整个数组
zeroes.fill(5);

let colors = ["red", "blue", "green"]; // 创建一个包含 3 个字符串的数组
alert(colors.toString()); // red,blue,green
alert(colors.valueOf()); // red,blue,green
alert(colors); // red,blue,green
let colors = ["red", "green", "blue"];
alert(colors.join(",")); // red,green,blue
alert(colors.join("||")); // red||green||blue

let numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];
let everyResult = numbers.every((item, index, array) => item > 2);
alert(everyResult); // false
let someResult = numbers.some((item, index, array) => item > 2);
alert(someResult); // true

const m = new Map().set("key1", "val1");
m.set("key2", "val2").set("key3", "val3");
alert(m.size); // 3

for (let pair of m.entries()) {
  alert(pair);
}
// [key1,val1]
// [key2,val2]
// [key3,val3]
```

## 面向对象编程

1. 对象： 属性数据，访问器属性
   1. 属性值简写，简写方法名
   2. 对象结构
      1. 创建对象：工厂，构造函数模式 原型
      2. 对象迭代：
   3. 继承
   4. 类
      1. 类定义 构造函数 实例 原型方法 类成员
      2. 迭代器方法  继承

```js
let book = {};
Object.defineProperties(book, {
  year_: {
    value: 2017,
  },
  edition: {
    value: 1,
  },
  year: {
    get: function () {
      return this.year_;
    },
    set: function (newValue) {
      if (newValue > 2017) {
        this.year_ = newValue;
        this.edition += newValue - 2017;
      }
    },
  },
});
let descriptor = Object.getOwnPropertyDescriptor(book, "year_");
console.log(descriptor.value); // 2017
console.log(descriptor.configurable); // false
console.log(typeof descriptor.get); // "undefined"
let descriptor = Object.getOwnPropertyDescriptor(book, "year");
console.log(descriptor.value); // undefined
console.log(descriptor.enumerable); // false
console.log(typeof descriptor.get); // "function"


let person = { 
 sayName(name) { 
 console.log(`My name is ${name}`); 
 } 
}; 

// 使用对象解构
let person = { 
 name: 'Matt', 
 age: 27 
}; 
let { name: personName, age: personAge } = person; 


// 嵌套结构
let person = { 
 name: 'Matt',  
 age: 27, 
 job: { 
 title: 'Software engineer' 
 } 
}; 
// 声明 title 变量并将 person.job.title 的值赋给它
let { job: { title } } = person; 
console.log(title); // Software engineer
```

原型模式
```js
let Person = function() {}; 
Person.prototype.name = "Nicholas"; 
Person.prototype.age = 29; 
Person.prototype.job = "Software Engineer"; 
Person.prototype.sayName = function() { 
 console.log(this.name); 
}; 
let person1 = new Person(); 
person1.sayName(); // "Nicholas" 
let person2 = new Person(); 
person2.sayName(); // "Nicholas" 
console.log(person1.sayName == person2.sayName); // true
/** 
 * 如前所述，构造函数有一个 prototype 属性
 * 引用其原型对象，而这个原型对象也有一个
 * constructor 属性，引用这个构造函数
 * 换句话说，两者循环引用：
 */ 
console.log(Person.prototype.constructor === Person); // true 
/** 
 * 正常的原型链都会终止于 Object 的原型对象
 * Object 原型的原型是 null 
 */ 
console.log(Person.prototype.__proto__ === Object.prototype); // true 
console.log(Person.prototype.__proto__.constructor === Object); // true 
console.log(Person.prototype.__proto__.__proto__ === null); // true 
console.log(Person.prototype.__proto__); 
// { 
// constructor: f Object(), 
// toString: ... 
// hasOwnProperty: ... 
// isPrototypeOf: ... 
// ... 
// } 
```
原型层级
```js 
function Person() {} 
Person.prototype.name = "Nicholas"; 
Person.prototype.age = 29; 
Person.prototype.job = "Software Engineer"; 
Person.prototype.sayName = function() { 
 console.log(this.name); 
}; 
let person1 = new Person(); 
let person2 = new Person(); 
person1.name = "Greg"; 
console.log(person1.name); // "Greg"，来自实例
console.log(person2.name); // "Nicholas"，来自原型
delete person1.name; 
console.log(person1.name); // "Nicholas"，来自原型

```
只要通过对象可以访问，in 操作符就返回 true，而 hasOwnProperty()只有属性存在于实例上
时才返回 true。因此，只要 in 操作符返回 true 且 hasOwnProperty()返回 false，就说明该属性
是一个原型属性。来看下面的例子
```js
function Person() {} 
Person.prototype.name = "Nicholas"; 
Person.prototype.age = 29; 
Person.prototype.job = "Software Engineer"; 

Person.prototype.sayName = function() { 
 console.log(this.name); 
}; 
let person = new Person(); 
console.log(hasPrototypeProperty(person, "name")); // true 
person.name = "Greg"; 
console.log(hasPrototypeProperty(person, "name")); // false
```

属性枚举顺序
```js
let k1 = Symbol('k1'), 
 k2 = Symbol('k2'); 
let o = { 
 1: 1, 
 first: 'first', 
 [k1]: 'sym2', 
 second: 'second', 
 0: 0 
}; 
o[k2] = 'sym2'; 
o[3] = 3; 
o.third = 'third'; 
o[2] = 2; 
console.log(Object.getOwnPropertyNames(o)); 
// ["0", "1", "2", "3", "first", "second", "third"] 
console.log(Object.getOwnPropertySymbols(o)); 
// [Symbol(k1), Symbol(k2)]
```

对象迭代
```javascript
const o = { 
 foo: 'bar', 
 baz: 1, 
 qux: {} 
}; 
console.log(Object.values(o));
// ["bar", 1, {}] 
console.log(Object.entries((o))); 
// [["foo", "bar"], ["baz", 1], ["qux", {}]]
```

原型的问题:
我们知道，原型上的所有属性是在实例间共享的，这对函数来说比较合适。另外包含原始值的属性
也还好，如前面例子中所示，可以通过在实例上添加同名属性来简单地遮蔽原型上的属性。真正的问题
来自包含引用值的属性。来看下面的例子：

```js

function Person() {} 
Person.prototype = { 
 constructor: Person, 
 name: "Nicholas", 
 age: 29, 
 job: "Software Engineer", 
 friends: ["Shelby", "Court"],
 sayName() { 
 console.log(this.name); 
 } 
}; 
let person1 = new Person(); 
let person2 = new Person(); 
person1.friends.push("Van"); 
console.log(person1.friends); // "Shelby,Court,Van" 
console.log(person2.friends); // "Shelby,Court,Van" 
console.log(person1.friends === person2.friends); // true

```

TODO: 
FIX:
继承是面向对象编程中讨论最多的话题。很多面向对象语言都支持两种继承：接口继承和实现继承。前者只继承方法签名，后者继承实际的方法。
接口继承在 ECMAScript 中是不可能的，因为函数没有签名。**实现继承**是 ECMAScript 唯一支持的继承方式，而这主要是通过原型链实现的。

重温一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型有一个属性指回构造函数，而实例有一个内部指针指向原型。如果原型是另一个类型的实例呢？那就意味着这个原型本身有一个内部指针指向另一个原型，相应地另一个原型也有一个指针指向另一个构造函数。这样就在实例和原型之间构造了一条原型链。这就是原型链的基本构想。

```js
function SuperType() { 
 this.property = true; 
} 
SuperType.prototype.getSuperValue = function() { 
 return this.property; 
}; 
function SubType() { 
 this.subproperty = false; 
} 
// 继承 SuperType 
SubType.prototype = new SuperType(); 
SubType.prototype.getSubValue = function () {


    return this.subproperty; 
}; 
let instance = new SubType(); 
console.log(instance.getSuperValue()); // true

```

实际上，原型链中还有一环。默认情况下，所有引用类型都继承自 Object，这也是通过原型链实
现的。任何函数的默认原型都是一个 Object 的实例，这意味着这个实例有一个内部指针指向
Object.prototype。这也是为什么自定义类型能够继承包括 toString()、valueOf()在内的所有默
认方法的原因。

```js
//覆盖方法
function SuperType() { 
 this.property = true; 
} 
SuperType.prototype.getSuperValue = function() { 
 return this.property; 
}; 
function SubType() { 
 this.subproperty = false; 
} 
// 继承 SuperType 
SubType.prototype = new SuperType(); 
// 新方法
SubType.prototype.getSubValue = function () { 
 return this.subproperty; 
}; 
// 覆盖已有的方法
SubType.prototype.getSuperValue = function () { 
 return false; 
}; 
let instance = new SubType(); 
console.log(instance.getSuperValue()); // false 

```
在使用原型实现继承时，原型实际上变成了另一个类型的实例。这意味着原先的实例属性摇身一变成为了原型属性。

```js
function SuperType() { 
 this.colors = ["red", "blue", "green"]; 
} 
function SubType() {} 
// 继承 SuperType 
SubType.prototype = new SuperType(); 
let instance1 = new SubType(); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
let instance2 = new SubType(); 
console.log(instance2.colors); // "red,blue,green,black"
```
**盗用构造函数**
为了解决原型包含引用值导致的继承问题，一种叫作“盗用构造函数”（constructor stealing）的技术在开发社区流行起来（这种技术有时也称作“对象伪装”或“经典继承”）。
```js
function SuperType() { 
 this.colors = ["red", "blue", "green"]; 
} 
function SubType() { 
 // 继承 SuperType 
 SuperType.call(this); 
} 
let instance1 = new SubType(); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
let instance2 = new SubType(); 
console.log(instance2.colors); // "red,blue,green"
```

组合继承
```javascript
function SuperType(name){ 
 this.name = name; 
 this.colors = ["red", "blue", "green"]; 
} 
SuperType.prototype.sayName = function() { 
 console.log(this.name); 
}; 
function SubType(name, age){ 
 // 继承属性
 SuperType.call(this, name); 
 this.age = age; 
} 
// 继承方法
SubType.prototype = new SuperType(); 
SubType.prototype.sayAge = function() { 
 console.log(this.age); 
}; 
let instance1 = new SubType("Nicholas", 29); 
instance1.colors.push("black"); 
console.log(instance1.colors); // "red,blue,green,black" 
instance1.sayName(); // "Nicholas"; 
instance1.sayAge(); // 29 
let instance2 = new SubType("Greg", 27); 
console.log(instance2.colors); // "red,blue,green" 
instance2.sayName(); // "Greg"; 
instance2.sayAge(); // 27 
```


2006 年，Douglas Crockford 写了一篇文章：《JavaScript 中的原型式继承》（“Prototypal Inheritance in JavaScript”）。这篇文章介绍了一种不涉及严格意义上构造函数的继承方法。他的出发点是即使不自定义类型也可以通过原型实现对象之间的信息共享。文章最终给出了一个函数：
```js
function object(o) { 
 function F() {} 
 F.prototype = o; 
 return new F(); 
} 
// 这个 object()函数会创建一个临时构造函数，将传入的对象赋值给这个构造函数的原型，然后返回这个临时类型的一个实例。本质上，object()是对传入的对象执行了一次浅复制。来看下面的例子：
let person = { 
 name: "Nicholas", 
 friends: ["Shelby", "Court", "Van"] 
}; 
let anotherPerson = object(person); 
anotherPerson.name = "Greg"; 
anotherPerson.friends.push("Rob"); 
let yetAnotherPerson = object(person); 
yetAnotherPerson.name = "Linda"; 
yetAnotherPerson.friends.push("Barbie"); 
console.log(person.friends); // "Shelby,Court,Van,Rob,Barbie" 

```

与原型式继承比较接近的一种继承方式是**寄生式继承**（parasitic inheritance），也是 Crockford 首倡的一种模式。寄生式继承背后的思路类似于寄生构造函数和工厂模式：创建一个实现继承的函数，以某种方式增强对象，然后返回这个对象。基本的寄生继承模式如下：
```js
function createAnother(original){ 
 let clone = object(original); // 通过调用函数创建一个新对象
 clone.sayHi = function() { // 以某种方式增强这个对象
 console.log("hi"); 
 }; 
 return clone; // 返回这个对象
} 

在这段代码中，createAnother()函数接收一个参数，就是新对象的基准对象。这个对象 original会被传给 object()函数，然后将返回的新对象赋值给 clone。接着给 clone 对象添加一个新方法sayHi()。最后返回这个对象。可以像下面这样使用 createAnother()函数：
let person = { 
 name: "Nicholas", 
 friends: ["Shelby", "Court", "Van"] 
}; 
let anotherPerson = createAnother(person); 
anotherPerson.sayHi(); // "hi"
```

> 类：正因为如此，实现继承的代码也显得非常冗长和混乱。为解决这些问题，ECMAScript 6 新引入的 class 关键字具有正式定义类的能力。
>
把类当成特殊函数ECMAScript 中没有正式的类这个类型。从各方面来看，ECMAScript 类就是一种特殊函数。声明一
个类之后，通过 typeof 操作符检测类标识符，表明它是一个函数

迭代器方法
```js
也可以只返回迭代器实例：
class Person { 
 constructor() { 
 this.nicknames = ['Jack', 'Jake', 'J-Dog']; 
 } 
 [Symbol.iterator]() { 
 return this.nicknames.entries(); 
 } 
} 
let p = new Person(); 
for (let [idx, nickname] of p) { 
 console.log(nickname); 
} 
// Jack 
// Jake 
// J-Dog
```

## 代理与反射
ECMAScript 6 新增的代理和反射为开发者**提供了拦截并向基本操作嵌入额外行为的能力**。具体地说，可以给目标对象定义一个关联的代理对象，而这个代理对象可以作为抽象的目标对象来使用。在对目标对象的各种操作影响目标对象之前，可以在代理对象中对这些操作加以控制
1. 代理是目标对象的*抽象*
2. get 捕获器 反射API
3. 代理模式 


>代理对象

```js
const target = { 
 id: 'target' 
}; 
const handler = {}; 
const proxy = new Proxy(target, handler); 
// id 属性会访问同一个值
console.log(target.id); // target 
console.log(proxy.id); // target 
// 给目标属性赋值会反映在两个对象上
// 因为两个对象访问的是同一个值
target.id = 'foo'; 
console.log(target.id); // foo 
console.log(proxy.id); // foo 
// 给代理属性赋值会反映在两个对象上
// 因为这个赋值会转移到目标对象
proxy.id = 'bar'; 
console.log(target.id); // bar 
console.log(proxy.id); // bar 
// hasOwnProperty()方法在两个地方
// 都会应用到目标对象
console.log(target.hasOwnProperty('id')); // true 
console.log(proxy.hasOwnProperty('id')); // true 
// Proxy.prototype 是 undefined 
// 因此不能使用 instanceof 操作符
console.log(target instanceof Proxy); // TypeError: Function has non-object prototype 
'undefined' in instanceof check 
console.log(proxy instanceof Proxy); // TypeError: Function has non-object prototype 
'undefined' in instanceof check 
// 严格相等可以用来区分代理和目标
console.log(target === proxy); // false
```

> get 捕获器
>
```javascript
const target = { 
 foo: 'bar' 
}; 
const handler = { 
 // 捕获器在处理程序对象中以方法名为键
 get() { 
 return 'handler override'; 
 } 
}; 
const proxy = new Proxy(target, handler); 
console.log(target.foo); // bar 
console.log(proxy.foo); // handler override 
console.log(target['foo']); // bar 
console.log(proxy['foo']); // handler override 
console.log(Object.create(target)['foo']); // bar 
console.log(Object.create(proxy)['foo']); // handler override

```
> get捕获器参数 添加额外动作
```javascript
const target = { 
 foo: 'bar', 
 baz: 'qux' 
}; 
const handler = { 
 get(trapTarget, property, receiver) { 
 let decoration = ''; 
 if (property === 'foo') { 
 decoration = '!!!'; 
 } 
 return Reflect.get(...arguments) + decoration; 
 } 
}; 
const proxy = new Proxy(target, handler); 
console.log(proxy.foo); // bar!!! 
console.log(target.foo); // bar 
console.log(proxy.baz); // qux 
console.log(target.baz); // qux 
```

> 代理模式
1. 跟踪属性访问  隐藏属性  属性验证
2. 函数与构造函数参数验证   数据绑定与可观察对象
```js

const user = { 
 name: 'Jake' 
}; 
const proxy = new Proxy(user, { 
 get(target, property, receiver) { 
 console.log(`Getting ${property}`); 
 return Reflect.get(...arguments); 
 }, 
 set(target, property, value, receiver) { 
 console.log(`Setting ${property}=${value}`); 
 return Reflect.set(...arguments); 
 } 
}); 
proxy.name; // Getting name 
proxy.age = 27; // Setting age=27 

// 隐藏属性
const hiddenProperties = ['foo', 'bar']; 
const targetObject = { 
 foo: 1, 
 bar: 2, 
 baz: 3 
}; 
const proxy = new Proxy(targetObject, { 
 get(target, property) { 
 if (hiddenProperties.includes(property)) { 
 return undefined; 
 } else { 
 return Reflect.get(...arguments); 
 } 
 }, 
 has(target, property) {
   if (hiddenProperties.includes(property)) { 
 return false; 
 } else { 
 return Reflect.has(...arguments); 
 } 
 } 
}); 
// get() 
console.log(proxy.foo); // undefined 
console.log(proxy.bar); // undefined 
console.log(proxy.baz); // 3 
// has() 
console.log('foo' in proxy); // false 
console.log('bar' in proxy); // false 
console.log('baz' in proxy); // true 


const target = { 
 onlyNumbersGoHere: 0 
}; 
const proxy = new Proxy(target, { 
 set(target, property, value) { 
 if (typeof value !== 'number') { 
 return false; 
 } else { 
 return Reflect.set(...arguments); 
 } 
 } 
}); 
proxy.onlyNumbersGoHere = 1; 
console.log(proxy.onlyNumbersGoHere); // 1 
proxy.onlyNumbersGoHere = '2'; 
console.log(proxy.onlyNumbersGoHere); // 1 

```

> 数据绑定 与可观察对象
>
```js
const userList = []; 
class User { 
 constructor(name) { 
 this.name_ = name; 
 } 
} 
const proxy = new Proxy(User, { 
 construct() { 
 const newUser = Reflect.construct(...arguments); 
 userList.push(newUser); 
 return newUser; 
 } 
}); 
new proxy('John'); 
new proxy('Jacob'); 
new proxy('Jingleheimerschmidt'); 
console.log(userList); // [User {}, User {}, User{}]

```

## 函数
1. 函数表达式、函数声明提升及箭头函数 
2. 函数参数，函数作为值 默认参数及扩展操作符
3. 函数内部 arguments this
4.  使用函数实现递归
5. 使用闭包实现私有变量

> 箭头函数
```js
let ints = [1, 2, 3]; 
console.log(ints.map(function(i) { return i + 1; })); // [2, 3, 4] 
console.log(ints.map((i) => { return i + 1 })); // [2, 3, 4] 
```
> ECMAScript 函数不能像传统编程那样重载。在其他语言比如 Java 中，一个函数可以有两个定义，只要签名（接收参数的类型和数量）不同就行。如前所述，ECMAScript 函数没有签名，因为参数是由包含零个或多个值的数组表示的。没有函数签名，自然也就没有重载.如果在 ECMAScript 中定义了两个同名函数，则后定义的会覆盖先定义的。来看下面的例子：
```js
function addSomeNumber(num) { 
 return num + 100; 
} 
function addSomeNumber(num) { 
 return num + 200; 
} 
let result = addSomeNumber(100); // 300
```
>函数声明提升
```js
ECMAScript 函数不能像传统编程那样重载。在其他语言比如 Java 中，一个函数可以有两个定义，
只要签名（接收参数的类型和数量）不同就行。如前所述，ECMAScript 函数没有签名，因为参数是由
包含零个或多个值的数组表示的。没有函数签名，自然也就没有重载。
如果在 ECMAScript 中定义了两个同名函数，则后定义的会覆盖先定义的。来看下面的例子：
function addSomeNumber(num) { 
 return num + 100; 
} 
function addSomeNumber(num) { 
 return num + 200; 
} 
let result = addSomeNumber(100); // 300
```

>使用 arguments.callee 就可以让函数逻辑与函数名解耦：
```js
function factorial(num) { 
 if (num <= 1) { 
 return 1; 
 } else { 
 return num * arguments.callee(num - 1); 
 } 
} 
```

> 前面提到过，ECMAScript 中的函数是对象，因此有属性和方法。每个函数都有两个属性：length和 prototype。其中，length 属性保存函数定义的命名参数的个数，如下例所示：
```js
function sayName(name) { 
 console.log(name); 
} 
function sum(num1, num2) { 
 return num1 + num2; 
} 
function sayHi() { 
 console.log("hi"); 
} 
console.log(sayName.length); // 1 
console.log(sum.length); // 2 
console.log(sayHi.length); // 0 
```

> 函数属性与方法
```js
function sum(num1, num2) { 
 return num1 + num2; 
} 
function callSum1(num1, num2) { 
 return sum.apply(this, arguments); // 传入 arguments 对象
} 
```

> 匿名函数经常被人误认为是闭包（closure）。闭包指的是那些引用了另一个函数作用域中变量的函
```js
function createComparisonFunction(propertyName) { 
 return function(object1, object2) { 
 let value1 = object1[propertyName]; 
 let value2 = object2[propertyName]; 
 if (value1 < value2) { 
 return -1; 
 } else if (value1 > value2) { 
 return 1; 
 } else { 
 return 0; 
 } 
 }; 
} 
```

模块模式
```js
let application = function() { 
 // 私有变量和私有函数 
 let components = new Array(); 
 // 初始化
 components.push(new BaseComponent()); 
 // 公共接口
 return { 
 getComponentCount() { 
 return components.length; 
 }, 
 registerComponent(component) { 
 if (typeof component == 'object') { 
 components.push(component); 
 } 
 } 
 }; 
}(); 

模块模式是在单例对象基础上加以扩展，使其通过作用域链来关联私有变量和特权方法。模块模式
的样板代码如下：
let singleton = function() { 
 // 私有变量和私有函数
 let privateVariable = 10; 
 function privateFunction() { 
 return false; 
 } 
 // 特权/公有方法和属性
 return { 
 publicProperty: true, 
 publicMethod() { 
 privateVariable++; 
 return privateFunction(); 
 } 
 }; 
}();

```
### 期约与异步函数
1. 异步编程 异步返回值  失败处理
2. 期约 异步函数的抽象  Promise   async、await


> 异步编程
```js
function double(value) { 
 setTimeout(() => setTimeout(console.log, 0, value * 2), 1000); 
} 
function double(value, callback) { 
 setTimeout(() => callback(value * 2), 1000); 
} 
double(3, (x) => console.log(`I was given: ${x}`)); 
// I was given: 6（大约 1000 毫秒之后）
```
显然，随着代码越来越复杂，回调策略是不具有扩展性的。“回调地狱”这个称呼可谓名至实归。
嵌套回调的代码维护起来就是噩梦。

但直到十几年以后，Barbara Liskov 和 Liuba Shrira 在 1988 年发表了论文“Promises: Linguistic Support for 
Efficient Asynchronous Procedure Calls in Distributed Systems”，这个概念才真正确立下来。同一时期的计算机科学家还使用了“终局”（eventual）、“期许”（future）、“延迟”（delay）和“迟付”（deferred）等术语指代同样的概念。

期约主要有两大用途。首**先是抽象地表示一个异步操作**。期约的状态代表期约是否完成。“待定”
表示尚未开始或者正在执行中。“兑现”表示已经成功完成，而“拒绝”则表示没有成功完成。

通过执行函数控制期约状态
```js
let p1 = new Promise((resolve, reject) => resolve()); 
setTimeout(console.log, 0, p1); // Promise <resolved> 
let p2 = new Promise((resolve, reject) => reject()); 
setTimeout(console.log, 0, p2); // Promise <rejected>
// Uncaught error (in promise)
```

如果给期约添加了多个处理程序，当期约状态变化时，相关处理程序会按照添加它们的顺序依次执
行。无论是 then()、catch()还是 finally()添加的处理程序都是如此。
```js
let p1 = Promise.resolve(); 
let p2 = Promise.reject(); 
p1.then(() => setTimeout(console.log, 0, 1)); 
p1.then(() => setTimeout(console.log, 0, 2)); 
// 1 
// 2 
p2.then(null, () => setTimeout(console.log, 0, 3)); 
p2.then(null, () => setTimeout(console.log, 0, 4)); 
// 3 
// 4 
p2.catch(() => setTimeout(console.log, 0, 5)); 
p2.catch(() => setTimeout(console.log, 0, 6)); 
// 5 
// 6 
p1.finally(() => setTimeout(console.log, 0, 7)); 
p1.finally(() => setTimeout(console.log, 0, 8)); 
// 7 
// 8 
```

期约有向图
```js
// A 
// / \ 
// B C 
// /\ /\ 
// D E F G 
let A = new Promise((resolve, reject) => { 
 console.log('A'); 
 resolve(); 
}); 
let B = A.then(() => console.log('B')); 
let C = A.then(() => console.log('C')); 
B.then(() => console.log('D')); 
B.then(() => console.log('E')); 
C.then(() => console.log('F')); 
C.then(() => console.log('G')); 
// A 
// B 
// C 
// D 
// E 
// F 
// G 
```

期约进度通知
 ```js
 let p = new TrackablePromise((resolve, reject, notify) => { 
 function countdown(x) { 
 if (x > 0) { 
 notify(`${20 * x}% remaining`); 
 setTimeout(() => countdown(x - 1), 1000); 
 } else { 
 resolve(); 
 } 
 } 
 countdown(5); 
}); 
p.notify((x) => setTimeout(console.log, 0, 'progress:', x)); 
p.then(() => setTimeout(console.log, 0, 'completed')); 
// （约 1 秒后）80% remaining 
// （约 2 秒后）60% remaining 
// （约 3 秒后）40% remaining 
// （约 4 秒后）20% remaining 
// （约 5 秒后）completed 
```

async 关键字用于声明异步函数
因为异步函数主要针对不会马上完成的任务，所以自然需要一种暂停和恢复执行的能力。使用 await关键字可以暂停异步函数代码的执行，等待期约解决

```js
let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3)); 
p.then((x) => console.log(x)); // 3 
使用 async/await 可以写成这样：
async function foo() { 
 let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3)); 
 console.log(await p); 
} 
foo(); 
// 3 
```

注意，await 关键字会暂停执行异步函数后面的代码，让出 JavaScript 运行时的执行线程。这个行为与生成器函数中的 yield 关键字是一样的。await 关键字同样是尝试“解包”对象的值，然后将这个值传给表达式，再异步恢复异步函数的执行。

```js
//异步等待
async function foo() { 
 let p = new Promise((resolve, reject) => setTimeout(resolve, 1000, 3)); 
 console.log(await p); 
} 
foo(); 
// 3
```

#### 实现sleep

很多人在刚开始学习 JavaScript 时，想找到一个类似 Java 中 Thread.sleep()之类的函数，好在程
序中加入非阻塞的暂停。以前，这个需求基本上都通过 setTimeout()利用 JavaScript 运行时的行为来
实现的。
有了异步函数之后，就不一样了。一个简单的箭头函数就可以实现 sleep()：
```javascript

async function sleep(delay) { 
 return new Promise((resolve) => setTimeout(resolve, delay)); 
} 
async function foo() { 
 const t0 = Date.now(); 
 await sleep(1500); // 暂停约 1500 毫秒
 console.log(Date.now() - t0); 
} 
foo(); 
// 1502 
```
> 如果顺序不是必需保证的，那么可以先一次性初始化所有期约，然后再分别等待它们的结果。比如：
```js
async function randomDelay(id) { 
 // 延迟 0~1000 毫秒
 const delay = Math.random() * 1000; 
 return new Promise((resolve) => setTimeout(() => { 
 setTimeout(console.log, 0, `${id} finished`); 
 resolve(); 
 }, delay)); 
} 
async function foo() { 
 const t0 = Date.now(); 
 const p0 = randomDelay(0); 
 const p1 = randomDelay(1); 
 const p2 = randomDelay(2); 
 const p3 = randomDelay(3); 
 const p4 = randomDelay(4); 
 await p0; 
 await p1; 
 await p2; 
 await p3; 
 await p4; 
 setTimeout(console.log, 0, `${Date.now() - t0}ms elapsed`); 
} 
foo(); 
// 1 finished 
// 4 finished 
// 3 finished 
// 0 finished 
// 2 finished 
// 877ms elapsed
```

> 用数组和 for 循环再包装一下就是
```js
async function randomDelay(id) { 
 // 延迟 0~1000 毫秒
 const delay = Math.random() * 1000; 
 return new Promise((resolve) => setTimeout(() => { 
 console.log(`${id} finished`); 
 resolve(); 
 }, delay)); 
} 
async function foo() { 
 const t0 = Date.now(); 
 const promises = Array(5).fill(null).map((_, i) => randomDelay(i)); 
 for (const p of promises) { 
 await p; 
 } 
 console.log(`${Date.now() - t0}ms elapsed`); 
} 
foo(); 
// 4 finished 
// 2 finished 
// 1 finished 
// 0 finished 
// 3 finished 
// 877ms elapsed

```

## BOM
BOM 是使用 JavaScript 开发 Web 应用程序的核心。BOM 提供了与网页无关的浏览器功能对象
1. window
2. location
3. navigator
4. screen
5. history

location 是最有用的 BOM 对象之一，提供了当前窗口中加载文档的信息，以及通常的导航功能。
这个对象独特的地方在于，它既是 window 的属性，也是 document 的属性。也就是说，
window.location 和 document.location 指向同一个对象。

```js
window.open("http://www.wrox.com/", "topFrame"); 
```
### DOM
文档对象模型（DOM，Document Object Model）是 HTML 和 XML 文档的编程接口。DOM 表示由多层节点构成的文档，通过它开发者可以添加、删除和修改页面的各个部分。脱胎于网景和微软早期的动态 HTML（DHTML，Dynamic HTML），DOM 现在是真正跨平台、语言无关的表示和操作网页的方式
1. Document Element Attr
2. Dom编程  MutationObserver
3. DOM扩展 selector API  HTML5 DOM 扩展
4. 事件类型 处理
5. 使用 WebGL 绘制 3D 图形
6. 表单脚本
7. 


DOM编程
```js
let script = document.createElement("script"); 
script.appendChild(document.createTextNode("function sayHi(){alert('hi');}")); 
document.body.appendChild(script); 

```

使用NodeList
理解 NodeList 对象和相关的 NamedNodeMap、HTMLCollection，是理解 DOM 编程的关键。这
3 个集合类型都是“实时的”，意味着文档结构的变化会实时地在它们身上反映出来，因此它们的值始终
例如，下面的代码会导致无穷循环：
```js
let divs = document.getElementsByTagName("div"); 
for (let i = 0; i < divs.length; ++i){ 
 let div = document.createElement("div"); 
 document.body.appendChild(div); 
} 
```
任何时候要迭代 NodeList，最好再初始化一个变量保存当时查询时的长度，然后用循环变量与这
个变量进行比较，如下所示：
```js
let divs = document.getElementsByTagName("div"); 
for (let i = 0, len = divs.length; i < len; ++i) { 
 let div = document.createElement("div"); 
 document.body.appendChild(div); 
} 

```

MutationObserver 的实例要通过调用 MutationObserver 构造函数并传入一个回调函数来创建：
```js

let observer = new MutationObserver(() => console.log('<body> attributes changed')); 
observer.observe(document.body, { attributes: true }); 
document.body.className = 'foo'; 
console.log('Changed body class'); 
// Changed body class 
// <body> attributes changed

如果想在变化记录中保存属性原来的值，可以将 attributeOldValue 属性设置为 true：
let observer = new MutationObserver( 
 (mutationRecords) => console.log(mutationRecords.map((x) => x.oldValue))); 
observer.observe(document.body, { attributeOldValue: true }); 
document.body.setAttribute('foo', 'bar'); 
document.body.setAttribute('foo', 'baz'); 
document.body.setAttribute('foo', 'qux'); 
// 每次变化都保留了上一次的值
// [null, 'bar', 'baz'] 

```

JavaScript 库中最流行的一种能力就是根据 CSS 选择符的模式匹配 DOM 元素。比如，jQuery 就完全
以 CSS 选择符查询 DOM 获取元素引用，而不是使用 getElementById()和 getElementsByTagName()。
Selectors API Level 1 的核心是两个方法：querySelector()和 querySelectorAll()

自 HTML4 被广泛采用以来，Web 开发中一个主要的变化是 class 属性用得越来越多，其用处是为
元素添加样式以及语义信息。自然地，JavaScript 与 CSS 类的交互就增多了，包括动态修改类名，以及
根据给定的一个或一组类名查询元素，等等。为了适应开发者和他们对 class 属性的认可，HTML5 增
加了一些特性以**方便使用 CSS 类**
getElementsByClassName()是 HTML5 新增的最受欢迎的一个方法，暴露在 document 对象和所有 HTML 元素上。

实际上，DOM2
和 DOM3 是按照模块化的思路来制定标准的，每个模块之间有一定关联，但分别针对某个 DOM 子集

### js api
1.  Atomics 与 SharedArrayBuffer  线程安全
2. 跨上下文消息
3.  Encoding API 
4.  File API 与 Blob API 
5. 拖放、
6.  Notifications API 、
7.   Page Visibility API 、
8.  Streams API 
9.   计时 API 、
10.  Web 组件
11. Web Cryptography API 

XDM
```js
let iframeWindow = document.getElementById("myframe").contentWindow; 
iframeWindow.postMessage("A secret", "http://www.wrox.com"); 
```

### XHR 网络请求
```javascript
let xhr = new XMLHttpRequest(); 
xhr.onreadystatechange = function() { 
 if (xhr.readyState == 4) { 
 if ((xhr.status >= 200 && xhr.status < 300) || xhr.status == 304) { 
 alert(xhr.responseText); 
 } else { 
 alert("Request was unsuccessful: " + xhr.status); 
 } 
 } 
}; 
xhr.open("get", "example.txt", true); 
xhr.send(null); 
```
Fetch API 能够执行 XMLHttpRequest 对象的所有任务，但更容易使用，接口也更现代化，能够在
Web 工作线程等现代 Web 工具中使用。XMLHttpRequest 可以选择异步，而 Fetch API 则必须是异步。

### 客户端存储
sessionStorage 对象只存储会话数据，这意味着数据只会存储到浏览器关闭。这跟浏览器关闭时
会消失的会话 cookie 类似。
1. cookie

## 附录
1. 框架 
   1. React  Flux  React 是 Facebook 开发的框架，专注于模型视图控制器（MVC，Model-View-Controller）模型中
的“视图”。专注的范围让它可以与其他框架或 React 扩展合作，实现 MVC 模式。React 使用单向数据
流，是声明性和基于组件的，基于虚拟 DOM 高效渲染页面，提供了在 JavaScript 包含 HTML 标记的 JSX
语法。Facebook 也维护了一个 React 的补充框架，叫作 Flux
   2.  Angular  Angular 1.x 和 Angular 2。前者是最初的 AngularJS 项目，后者则是基于 ES6 语法和 TypeScript 完全重新设计的框架。 
谷歌在 2010 年首次发布的 Angular 是基于模型视图视图模型（MVVM）架构的全功能 Web 应用
程序框架。
   3. Vue
2. 库
3. 动画特效：
   1. three.js  moo.fxlight.bx