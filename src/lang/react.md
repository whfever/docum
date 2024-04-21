# React

## what

> 组件化  ：不用再查询和更新Dom

![image-20230805214628195](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805214628195.png)

![image-20230805214537825](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805214537825.png)





## Env

> premetieer  

```
  npm  create vite@5.1.0
  React  Vue
```



### Project  Structure

## Base

### React Component|  Jsx

 

![image-20230805222508591](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805222508591.png)





![image-20230805224533353](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805224533353.png)

#### Fragment

![image-20230805234758327](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805234758327.png)



#### Rending  Lists

```react
function List() {
    const items=["wh",'zl']
  return (
    <div>
        <h1></h1>
        <ul className="list-group">
            {
                items.map((item)=>(
                    <li>{item}</li>
                ))
            }
          <li className="list-group-item">An item</li>
          <li className="list-group-item">A second item</li>
          <li className="list-group-item">A third item</li>
          <li className="list-group-item">A fourth item</li>
          <li className="list-group-item">And a fifth one</li>
        </ul>
    </div>
  );
}

export default List;

```



> 条件渲染   

![image-20230805235804186](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230805235804186.png)

### Create

> 1.  cra
> 2.  vite



### React  Ecosystem

> 擅长创建动态和交互式用户界面

> 用Routing  转到另一个页面





```
npm i bootstrap@4.1.0
```

###  hot  module

> hmr



```
          <li key={item} onClick={console.log(item)}>{item}</li>

```



### handing  events



### managing  state

> 因为函数组件是纯粹的 JavaScript 函数，每次函数执行完毕后，所有的本地变量都会被销毁，因此无法保持状态的持久性

```
import React, { useState } from 'react';

const MyComponent = () => {
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount(count + 1);
    // 使用 setCount 来更新 count 的值，触发组件的重新渲染
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
};

export default MyComponent;

```



### passing  data

##### props

> 传递时可以直接解构

![image-20230806005615336](C:\Users\whfever\AppData\Roaming\Typora\typora-user-images\image-20230806005615336.png)





##### context

> 反复嵌套

```
import Child from "./child";
// 创建一个上下文
import { createContext } from "react";

const DataContext = createContext("default");

function Parent() {
  const data = "Hello Child";

  return (
    <DataContext.Provider value={data}>
      <p>Hello Parent</p>
      <Child />
    </DataContext.Provider>
  );
}

export default Parent;
export { DataContext };

```





interface

> 通过使用接口，您可以明确指定组件的属性和状态类型，使代码更加健壮和易于维护。TypeScript 在进行类型检查时会基于接口提供更准确的错误提示，帮助您在开发过程中尽早发现潜在的类型错误，提高代码质量。





## react-router





## Redux
