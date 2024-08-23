# Settings

> C:\ProgramData\Microsoft\Windows\Start Menu
>

## Install

```shell
winget
winget  install  feishu --location D:\natural\win
oh-my-posh
sudo apt install `<deb name>`
sudo dpkg -i .deb
```

## Edge

```
extend

Octotree  scihub zenhub  react vue

Tampermonkey  buautifyread   itab

vim globalspeed   eagle yiban chenjinshi
```

### vim

```
## gh
map go createTab url="http://www.google.com"
map gls createTab url="http://www.lsdhss.com/"
map gc createTab url="http://www.chinadaily.com.cn/"
map jc createTab url="https://chat.openai.com/"
map gt createTab url="https://www.tadu.com/"
map gr createTab url="https://www.reddit.com/"
map gl createTab url="https://leetcode.cn/problemset/all/"
map gn createTab url="https://www.nowcoder.com/exam/oj?page=1"
map gr createTab url="https://weread.qq.com/web/category/newrating_publish"
map gp createTab url="https://tophub.today/apple/podcasts"
map gm createTab url="https://medium.com/"
map gj createTab url="https://web.okjike.com/"
map gb createTab url="https://www.bilibili.com/"
map gy createTab url="https://www.youtube.com/"
map gti createTab url="https://time.geekbang.org/"
map gs createTab url="https://web.skype.com/"
map gg createTab url="https://github.com/"
map gx createTab url="https://www.xiaohongshu.com/explore"
map gw createTab url="https://weread.qq.com/"
map gy createTab url="https://www.yuque.com/search?q="
map gf createTab url="https://cupfox.app/"
map gtm createTab url="https://mp.weixin.qq.com/cgi-bin/appmsg?begin=0&count=10&type=77&action=list_card&token=1236187514&lang=zh_CN"
#h
map h1 createTab url="https://m.weibo.cn/"
map h2 createTab url="https://www.toutiao.com/"
map h3 createTab url="https://www.jianshu.com/"
map h4 createTab url="https://www.xiaohongshu.com/explore"
map h5 createTab url="https://www.zhihu.com/"
map h6 createTab url="https://www.newrank.cn/ranklist/weixin"
map h7 createTab url="https://baijiahao.baidu.com/builder/rc/monetizationTask/home?app_id=1763053681787176"
map h8 createTab url=""
map h9 createTab url=""
map h0 createTab url="https://www.ximalaya.com/"
#g
map g1 createTab url="https://instant.1point3acres.cn/feed"
map g2 createTab url="https://www.v2ex.com/"
map g3 createTab url="https://36kr.com/"
map g4 createTab url=""
map g5 createTab url=""
map g6 createTab url=""
map g7 createTab url=""
map g8 createTab url=""
map g9 createTab url=""
map g0 createTab url=""
```

 Console

### console

- 遍历li
  
  ```js
  // 假设你的<ul>元素有一个类名"my-list"
  const ulElement = document.querySelector('.relation-list');



```js
// 获取所有<li>元素
const li = ulElement.querySelectorAll('li');
const li = Array.from(ulElement.querySelectorAll('li'));
  function me() {
  for (var i = 0; i< li.length; i++) {
    console.log(li.item(i));
    setTimeout(me,1000);
     li[i].querySelector('.follow-select').setAttribute("class","icon icon-follow-watched icon-follow-selected");
   console.log(li.item(i));
  }
  }


var ul1 = document.querySelector('.relation-list');
// 获取第一个li元素
var li1 = ul1.querySelector('li');

while (li) {
    <!-- document.querySelector('.relation-list'); -->
    console.log(li.textContent); // 打印当前li的文本内容
    li = li.nextElementSibling; // 移动到下一个同级元素
}

// 获取ul元素
var d = document.querySelector('.relation-list');

// 遍历ul的所有子元素

  for (var i = 0; i < d.children.length; i++) {
    var child = d.children[i];
// 检查当前子元素是否是li
if (child.tagName === 'LI') {
    // 获取li元素下的所有div元素
    var divs = child.getElementsByTagName('div');

    // 为第一个div元素添加属性
    if (divs.length > 0) {
  
       divs[0].setAttribute('class', 'icon icon-follow-watched icon-follow-selected'); // 这里可以根据需要修改属性名和值
console.log('这段代码将在1000毫秒后执行');
            }
}
        for (var i = 0; i < d.children.length; i++) {
    var child = d.children[i];
            // 检查当前子元素是否是li
if (child.tagName === 'LI') {
    // 获取li元素下的所有div元素
    var divs = child.getElementsByTagName('div');

    // 为第一个div元素添加属性
    if (divs.length > 0) {

      setTimeout(() => {
         console.log('这段代码将在' + ms + '毫秒后执行');
        divs[0].setAttribute('class', 'icon icon-follow-watched icon-follow-selected'); // 这里可以根据需要修改属性名和值);
          
          }, 1000)；     
        }
    }
}
```


​           

## Steam

shortkeys

1. ALt enter alt F4 大小屏幕



## git

```bash

```sh
git config --global user.name zl
git config --global user.email whfever@163.com
git config --global alias.st status  # st 替换status
git config --global alias.co checkout # co 替换checkout
ssh-keygen -t rsa -C "whfever@163.com"

git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy https://127.0.0.1:7890
git config --global http.https://github.com.proxy socks5://127.0.0.1:7890
git config --global https.https://github.com.proxy socks5://127.0.0.1:7890
git config --global -l
```

## node

```js
npm config set prefix "D:\\doc\natural\ware\node\prefix"    （全局安装模块）
npm config set cache "D:\\doc\natural\ware\node\precache"        （缓存模块）

npm install -g cnpm --registry=https://registry.npm.taobao.org
npm set registry https://registry.npm.taobao.org/
npm config set registry https://registry.npmjs.org
```

## Word

1.img  ^g 替换 全部



## Vscode



### user snippets markdown

```json
1. settings.json
    "[markdown]": {
        "editor.unicodeHighlight.ambiguousCharacters": false,
        "editor.unicodeHighlight.invisibleCharacters": false,
        "diffEditor.ignoreTrimWhitespace": false,
        "editor.wordWrap": "on",
        // "editor.quickSuggestions":true
        "editor.quickSuggestions": {
            "comments": "true",
            "strings": "true",
            "other": "true"
        
        }
    }
2. markdown.json
	"Print to ```js": {
		"prefix": "```js",
		"body": [
			"```js",
			"$1",
			"$2",
			"```",
		],
		"description": "js代码片段"
	},
```

### EdgeTODO

settings.json

```json
"todo-tree.highlights.defaultHighlight": {
    "type": "tag",
    "gutterIcon": true,
    "foreground": "white"
},
"todo-tree.highlights.customHighlight": {
  "TODO": {
    "icon": "checklist",
    "background": "orange",
    "iconColour": "orange"
  },
  "FIXME": {
    "icon": "bug",
    "background": "red",
    "iconColour": "red"
  }
},
```