# Vue

 的这种编程方式，称之为 “数据驱动” 编程。
抽象了大部分对 DOM 的直接操作

如果在一个页面上频繁且大量的操作真实 DOM ，频繁的触发浏览器回流（ Reflow ）与重绘（ Repaint ），会带来很大的性能开销，从而造成页面卡顿，在大型项目的性能上很是致命。

而 Vue 则是通过操作虚拟 DOM （ Virtual DOM ，简称 VDOM ），
数据绑定，数据传递：组件

*<u>每一次数据更新都通过 Diff 算法找出需要更新的节点，</u>*只更新对应的虚拟 DOM ，再去映射到真实 DOM 上面渲染，以此避免频繁或大量的操作真实 DOM 。
## 实现简单的Vue
### 实现Observer

```js
Obeject.defineProperty()

// ... 省略
function defineReactive(data, key, val) {
	var dep = new Dep();
    observe(val); // 监听子属性

    Object.defineProperty(data, key, {
        // ... 省略
        set: function(newVal) {
        	if (val === newVal) return;
            console.log('哈哈哈，监听到值变化了 ', val, ' --> ', newVal);
            val = newVal;
            dep.notify(); // 通知所有订阅者
        }
    });
}

function Dep() {
    this.subs = [];
}
Dep.prototype = {
    addSub: function(sub) {
        this.subs.push(sub);
    },
    notify: function() {
        this.subs.forEach(function(sub) {
            sub.update();
        });
    }
};
```
1. 转换getter setter
   //JS的浏览器单线程特性，保证这个全局变量在同一时间内，只会有同一个监听器使用
    //利用闭包的特性,修改value,get取值时也会变化
2. 监听队列

### 实现Compiler

```javascript   

function getter(scope) {
    return  scope.a + Math.PI + scope.b + scope.fn(scope.a);
}


```

### 实现watcher

```js
// Observer.js
// ...省略
Object.defineProperty(data, key, {
	get: function() {
		// 由于需要在闭包内添加watcher，所以通过Dep定义一个全局target属性，暂存watcher, 添加完移除
		Dep.target && dep.addDep(Dep.target);
		return val;
	}
    // ... 省略
});

// Watcher.js
Watcher.prototype = {
	get: function(key) {
		Dep.target = this;
		this.value = data[key];	// 这里会触发属性的getter，从而添加订阅者
		Dep.target = null;
	}
}
```

###  实现MVVM
实现数据绑定的做法有大致如下几种：

1. 发布者-订阅者模式（backbone.js）

```js
function User(uid){
    var binder = new DataBinder(uid),

        user = {
            atttibutes: {},

            //属性设置器使用数据绑定器PubSub来发布变化   

            set: function(attr_name,val){
                this.attriures[attr_name] = val;
                binder.trigger(uid + ":change", [attr_name, val, this]);
            },

            get: function(attr_name){
                return this.attributes[attr_name];
            },

            _binder: binder
        };

        binder.on(uid +":change",function(vet,attr_name,new_val,initiator){
            if(initiator !== user){
                user.set(attr_name,new_val);
            }
        })
}
```

2. 脏值检查（angular.js）
   angular.js 是通过脏值检测的方式比对数据是否有变更，来决定是否更新视图，最简单的方式就是通过 setInterval() 定时轮询检测数据变动，当然Google不会这么low，angular只有在指定的事件触发时进入脏值检测，大致如下：

3. 数据劫持（vue.js）
   vue.js 则是采用数据劫持结合发布者-订阅者模式的方式，通过Object.defineProperty()来劫持各个属性的setter，getter，在数据变动时发布消息给订阅者，触发相应的监听回调。

![mvvm](https://github.com/DMQ/mvvm/raw/master/img/2.png)
## 核心功能

1. 声明式渲染：抽象了大部分对 DOM 的直接操作
 v-if v-for  声明一个同名的 ref：

2. 响应性：Vue 会自动跟踪 JavaScript 状态并在其发生变化时响应式地更新 DOM。
   v-bind  v-on  v-model  响应式依赖 computed   watch  模版语法  :class :style 绑定样式

3. 生命周期钩子 ：特定时机执行自己的代码    
**beforeCreate**：在组件实例创建之前执行，此时还没有初始化数据和事件。可以在这里做一些初始化的操作，比如设置 loading 状态，或者获取一些全局的配置信息。
**created**：在组件实例创建之后执行，此时可以访问数据和方法，但还没有编译模板和挂载到 DOM。可以在这里做一些异步请求，或者结束 loading 状态。
**beforeMount：**在组件模板编译和挂载之前执行，此时可以修改数据，但还没有渲染出真实的 DOM。可以在这里做一些性能优化，比如使用虚拟 DOM 或者服务端渲染。
**mounted**：在组件模板编译和挂载之后执行，此时可以访问真实的 DOM，进行一些 DOM 操作或异步请求。可以在这里使用一些第三方的插件，比如图表，滚动，动画等。
**beforeUpdate**：在组件数据更新之前执行，此时可以获取旧的数据，但还没有重新渲染 DOM。可以在这里做一些数据的校验，或者阻止不必要的更新。
**updated：**在组件数据更新之后执行，此时可以获取新的数据和 DOM。可以在这里做一些数据的处理，或者触发一些事件。
**beforeDestroy**：在组件实例销毁之前执行，此时可以进行一些清理操作，比如取消事件监听，移除定时器，释放资源等。
**destroyed：**在组件实例销毁之后执行，此时组件已经从 DOM 中移除，不再具有响应式的特性。可以在这里做一些统计或者日志的记录。
4. 组件
  嵌套组件   
  props 父组件传递参数   
  emit  子组件传递事件
  slot   插槽出口
## 语法基础
1. ref  模版引用
虽然 Vue 的声明性渲染模型为你抽象了大部分对 DOM 的直接操作，但在某些情况下，我们仍然需要直接访问底层 DOM 元素。要实现这一点，我们可以使用特殊的 ref attribute：

```js
<script setup>
import { ref, onMounted } from 'vue'

// 声明一个 ref 来存放该元素的引用
// 必须和模板里的 ref 同名
const input = ref(null)

onMounted(() => {
  input.value.focus()
})
</script>

<template>
  <input ref="input" />
</template>

```
## Vue Router
1. 局部更新页面   简化跳转：起别名 路由传参： activated：生命周期钩子  
2. 路由守卫：权限控制  工作模式：hash history

## Vuex
Vuex 是一个专为 Vue.js 应用程序开发的状态管理模式 + 库。它采用集中式存储管理应用的所有组件的状态，并以相应的规则保证状态以一种可预测的方式发生变化。
1. 响应式Vuex 的状态存储是响应式的。
当 Vue 组件从 store 中读取状态的时候，若 store 中的状态发生变化，那么相应的组件也会相应地得到高效更新。

你不能直接改变 store 中的状态。改变 store 中的状态的唯一途径就是显式地提交 (commit) mutation。这样使得我们可以方便地跟踪每一个状态的变化，从而让我们能够实现一些工具帮助我们更好地了解我们的应用。
为了处理异步操作，让我们来看一看 Action

store.dispatch 可以处理被触发的 action 的处理函数返回的 Promise，并且 store.dispatch 仍旧返回 Promise：

Vuex 允许我们将 store 分割成模块（module）。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：
## Pinia
1. 为什么从Vux 转Pinia
**Srore**
需要解决的问题：当我们有多个组件共享一个共同的状态时
解决办法1：，一个可行的办法是将共享状态“提升”到共同的祖先组件上去，再通过 props 传递下来。然而在深层次的组件树结构中这么做的话，很快就会使得代码变得繁琐冗长。这会导致另一个问题：*Prop 逐级透传问题*。 
```javascript
// Providde
<script setup>
import { provide } from 'vue'

provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
</script>

import { provide } from 'vue'

export default {
  setup() {
    provide(/* 注入名 */ 'message', /* 值 */ 'hello!')
  }
}
import { createApp } from 'vue'

const app = createApp({})

app.provide(/* 注入名 */ 'message', /* 值 */ 'hello!')

```

     action mutation patter 复杂性 
**Vuex**：
     通过定义和隔离状态管理中的各种概念并通过强制规则维持视图和状态间的独立性，我们的代码将会变得更结构化且易维护。
