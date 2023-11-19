# typescript

## Introduction

>  JavaScript with  type checking

|              |                                                              |      |
| ------------ | ------------------------------------------------------------ | ---- |
| static-typed | ![image-20230804110556521](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804110556521.png) |      |
| tsc          | ![image-20230804112323273](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230804112323273.png) |      |
|              |                                                              |      |

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

###  any

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

