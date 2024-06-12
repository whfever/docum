# typescript

## Introduction

>  JavaScript with  type checking

|              |                                                                                                                    |     |
| ------------ | ------------------------------------------------------------------------------------------------------------------ | --- |
| static-typed | ![image-20230804110556521](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804110556521.png) |     |
| tsc          | ![image-20230804112323273](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804112323273.png) |     |
|              |                                                                                                                    |     |

> development environment

```
sudo npm i -g typeScript
tsc -v
tsc -- init
```

### env

### debug

![image-20230804113949112](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804113949112.png)

## variables

### Built intypes

```
let age:number =20;
let course = 'TypeScript';
```

![image-20230804114508179](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804114508179.png)

> built in types

### any

```
level=1;
level='a';
```

```
"noImplicitAny": false,      
```

### numbers

![image-20230804115703496](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804115703496.png)

### Tuple

```
let user:[number,string]= [1,"M"];
user.push(2 );
```

### Enum

![image-20230804122839216](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804122839216.png)

### functions

![image-20230804123142949](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804123142949.png)

### Objects

![image-20230804123433754](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804123433754.png)

```
let  employee:{id:number,name?:string}={id:1}
let  employee2:{readonly id:number,name?:string}={id:1}
```

### advaced types

```
type  pm={
    id:number,
    name?:string

}
let  employee3:pm={id:1}
```

![image-20230804123837382](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804123837382.png)

### Union types

![image-20230804124733255](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804124733255.png)

### Intersection types

![image-20230804124938126](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804124938126.png)

```
interface IPerson {
  id: string;
  age: number;
}

interface IWorker {
  companyId: string;
}

type IStaff = IPerson & IWorker;

const staff: IStaff = {
  id: 'E1006',
  age: 33,
  companyId: 'EXE'
};

console.dir(staff)
```

### Literal types

![image-20230804125522763](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804125522763.png)

### nubble types

> strictNUllChecks

![image-20230804125732319](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804125732319.png)

### Option Chaning

![image-20230804130417168](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804130417168.png)

```
let get1=get(null );
console.log(get1?.charAt(2))
let get2=get ("123" );
console.log(get2?.charAt(2))
```

## 核心

1. TypeScript的核心原则之一是对值所具有的结构进行类型检查。 它有时被称做“鸭式辨型法”或“结构性子类型化”。 
   在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码**定义契约**
   接口描述了类的公共部分，而不是公共和私有两部分。 它不会帮你检查类是否具有某些私有成员
   和类一样，接口也可以相互继承

2. 传统的JavaScript程序使用函数和基于原型的继承来创建可重用的组件，但对于熟悉使用面向对象方式的程序员来讲就有些棘手，因为他们用的是基于类的继承并且对象是由类构建出来的。
   
   在像C#和Java这样的语言中，可以使用泛型来创建可重用的组件
   相比于操作any所有类型，我们想要限制函数去处理任意带有.length属性的所有类型。 只要传入的类型有这个属性，我们就允许，就是说至少包含这一属性。 为此，我们需要列出对于T的约束要求
   
   ```js
   interface Lengthwise {
    length: number;
   }
   ```

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}

```
3.  装饰器

```js
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```