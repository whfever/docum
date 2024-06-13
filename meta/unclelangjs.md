# 狼叔 Node.js

## 简介

1. 早起架构：
   1. Chrome V8 是 Google 发布的开源 JavaScript 引擎，采用 C/C++ 编写，在 Google 的 Chrome 浏览器中被使用。Chrome V8 引擎可以独立运    行，也可以用来嵌入到 C/C++ 应用程序中执行。
   2. Event Loop 事件循环（由 libuv 提供）
   3. Thread Pool 线程池（由 libuv 提供）
2. 进阶学习
   1. 面向过程 OO  函数式编程
   2. zeal 离线文档
   3. 

## 核心 异步流程控制

1. 异步流程控制学习重点
   1. 迭代  ：推荐：使用Async函数 + Promise组合，
      ![promise](https://i5ting.github.io/How-to-learn-node-correctly/media/14913280187332/Screen%20Shot%202017-04-05%20at%2008.43.08.png)
      
      > 而Promise是对回调地狱的思考，或者说是改良方案。目前使用非常普遍，可以说是在async函数普及之前唯一一个通用性规范，甚至 Node.js 社区都在考虑 Promise 化，可见其影响之大。
2. Api写法：Error-first Callback 和 EventEmitter
3. 中流砥柱：Promise

```javascript
promise.then(function(text){
    console.log(text)// Stuff worked!
    return Promise.reject(new Error('我是故意的'))
}).catch(function(err){
    console.log(err)
})
```

4. 终极解决方案：Async/Await
   
   ```javascript
   const Promise = require('bluebird');
   const fs = Promise.promisifyAll(require("fs"));
   ```

async function main(){
    const contents = await fs.readFileAsync("myfile.js", "utf8")
    console.log(contents);
}

main();

```

## web

带视图的传统Web应用和面向Api接口应用，到通过 RPC 调用封装对数据库的操作，到提供前端 Api 代理和网关，服务组装等，统称为后端开发，不再是以往只有和数据库打交道的部分才算后端。这样，就可以让前端工程师对开发过程可控，更好的进行调优和性能优化。

框架名称    特性    点评
Express    简单、实用，路由中间件等五脏俱全    最著名的Web框架
Derby.js && Meteor    同构    前后端都放到一起，模糊了开发便捷，看上去更简单，实际上上对开发来说要求更高
Sails、Total    面向其他语言，Ruby、PHP等    借鉴业界优秀实现，也是 Node.js 成熟的一个标志
MEAN.js    面向架构    类似于脚手架，又期望同构，结果只是蹭了热点
Hapi和Restfy    面向Api && 微服务    移动互联网时代Api的作用被放大，故而独立分类。尤其是对于微服务开发更是利器
ThinkJS    面向新特性    借鉴ThinkPHP，并慢慢走出自己的一条路，对于Async函数等新特性支持，无出其右，新版v3.0是基于Koa v2.0的作为内核的
Koa    专注于异步流程改进    下一代Web框架
Egg    基于Koa，在开发上有极大便利    企业级Web开发框架

### Web编程核心

异步流程控制（前面讲过了）
基本框架 Koa或Express，新手推荐Express，毕竟资料多，上手更容易。如果有一定经验，推荐Koa，其实这些都是为了了解Web编程原理，尤其是中间件机制理解。
数据库 mongodb或mysql都行，mongoose和Sequelize、bookshelf，TypeOrm等都非常不错。对于事务，不是Node.js的锅，是你选的数据库的问题。另外一些偏门，想node连sqlserver等估计还不成熟，我是不会这样用的。
模板引擎， ejs，jade，nunjucks。理解原理最好。尤其是extend，include等高级用法，理解布局，复用的好处。其实前后端思路都是一样的。