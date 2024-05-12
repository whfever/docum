# MDN
## HTML
1. 创建元素 ： 标签，属性，内容
```js
1. meta utf-8 description
<p>Japanese example: <span lang="ja">ご飯が熱い。</span>.</p>
结构化文档： h p 强调span strong  em b,i,u
链接  ：
a <a href="https://www.mozilla.org/zh-CN/">Mozilla 主页</a>的链接。
<a href="https://developer.mozilla.org/zh-CN/">
  <img src="mdn_logo.svg" alt="MDN Web 文档主页" />
</a>

引用： <link rel="stylesheet" href="my-css-file.css" />
        <script src="my-js-file.js" defer></script>

```
2.  网站架构
 nav header main（aside）  footer

# CSS
基于规则的语言
1. 选择器
结构  类 ID 伪类 伪元素 关系选择器
```js
body h1 + p .special {
  color: yellow;
  background-color: black;
  padding: 5px;
}
a:hover {
}
p::first-line {
}
article > p {
}
li[class^="a"]匹配了任何值开头为a的属性，于是匹配了前两项。
li[class$="a"]匹配了任何值结尾为a的属性，于是匹配了第一和第三项。
li[class*="a"]匹配了任何值的字符串中出现了a的属性，于是匹配了所有项。

```
1. 样式

```js
.box {
  margin: 30px;
  width: 100px;
  height: 100px;
  background-color: rebeccapurple;
  transform: rotate(0.8turn);
}

```
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
