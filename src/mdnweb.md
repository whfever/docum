# MDN
## Vue
1. 创建组件 导入导出   数据传递
```js
{{}}  props
:checked="isDone"  //function
:done="item.done"  //data
v-for="item in items"  :key="item.id"      //data

```
2.  渲染组件
```js
v-for="item in items"  :done="item.done"


v-if  v-else
```

3. 事件

子组件  @change="$emit('checkbox-changed') "->父组件@checkbox-changed="updateDoneStatus(item.id)"
```js
v-on methods:{}  @submit="obsubmit"    v-model="label"
  this.$emit('todo-added', this.label)

// 计算属性
    computed: {
  listSummary() {
    const numberFinishedItems = this.ToDoItems.filter((item) =>item.done).length
    return `${numberFinishedItems} out of ${this.ToDoItems.length} items completed`
  }
},

```
