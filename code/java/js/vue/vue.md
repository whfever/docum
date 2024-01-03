# Vue
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
