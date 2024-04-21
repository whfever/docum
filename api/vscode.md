# vscode
## file icon
![](https://raw.githubusercontent.com/PKief/vscode-material-icon-theme/main/images/fileIcons.png)
Folder cicon
![](https://raw.githubusercontent.com/PKief/vscode-material-icon-theme/main/images/folderIcons.png)

 Markdown
```
.vscode
 ┗ extensions
   ┗ icons
     ┗ sample.svg
```
*123* 

```
.vscode
 ┗ extensions
   ┗ icons
     ┣ folder-sample.svg
```
- 
```
.vscode
 ┗ extensions
   ┗ icons
     ┣ folder-sample.svg
     ┗ folder-sample-open.svg
```
## background
1. extension
2. settings.json
```
 "background.customImages": [
    "file:///D:\\storage\\picture\\wallpaper\\man.png"
    
  ],
```

## batch replace
所有行后添加逗号
^\s*(?=\r?$)\n
第一步：调起替换窗口，快捷键：CTRL + H
第二部：开启正则表达式，快捷键：ALT + R
第三部：在替换窗口的查找输入框输入$，这时全文的所有行最后都一个选中标记
第四部：输入要在行末尾加入的字符
第五步：选择全部替换

## keyboard short



1. ctrl + t 打开搜索框，
    直接打字符串是搜索文件名。    
    @ + 字符串是搜索本文件内的符号    

     + 字符串是搜索工程内的所有符号  
2.  markdown all in one   Markdown PreviewInhance
    1.  ctrl + shift +V  view；
        1.  ctrl p ctrl t/s
    2.  ctrl +shift+p  content 创建目录
3. Ctrl + k + Ctrl + p	当前打开的文档
   1. Ctrl + k + Ctrl + l	折叠  
4. 

### table
```js

    
Ctrl + k + s	查看与更改键盘快捷方式
Ctrl + b	加粗 加粗
Ctrl + m	放大 放 大 放大放大
Ctrl + g	跳转到?行
Ctrl + h	查找与替换
Ctrl + j	调出电脑 cmd 控制终端
Ctrl + l	向下选中
Ctrl + w	关闭当前文件
Ctrl + i	斜体 斜体
Ctrl + o	打开文件
Ctrl + Tab	在不同工程之间切换
Ctrl + k + z	全屏显示当前工程
Ctrl + k + c	VScode 剪切板
Ctrl + k + m	选择语言模式
Ctrl + k + d	只读模式(会指示你未保存的更改)
Ctrl + k + f	关闭当前实例的工作区或文件夹
Ctrl + k + e	打开资源管理器
Ctrl + k + r	打开当前文件所在文件夹
Ctrl + k + u	关闭当前工程之外的所有工程
Ctrl + k + o	当前工程在新窗口开启
Ctrl + Shift + x	扩展商店
Ctrl + Shift + c	进入当前工程文件目录下 cmd 控制终端
Ctrl + Shift + v	检视当前 Markdown 文件的预览
Ctrl + Shift + b	配置生成任务
Ctrl + Shift + n	打开新的 VScode 窗口
Ctrl + Shift + m	打开 VScode 的调试窗口 问题 一栏
Ctrl + Shift + d	打开 运行 和 调试 侧栏
Ctrl + Shift + g	打开 源代码管理 侧栏
Ctrl + Shift + h	搜索和替换
Ctrl + Shift + k	删除当前行
Ctrl + Shift + w	关闭 VScode
Ctrl + Shift + e	打开资源管理器
Ctrl + Shift + r	重构操作
Ctrl + Shift + y	打开 VScode 的调试窗口 调试控制台 一栏
Ctrl + Shift + u	打开 VScode 的调试窗口 输出 一栏
Ctrl + Shift + o	查看 Markdown 章节目录
Ctrl + Shift + p	搜索 VScode 功能
Ctrl + k + Ctrl + c/u/q	注释 或 取消注释
Ctrl + k + Ctrl + b	一个类似于书签的东西
Ctrl + k + Ctrl + m	扩展商店
Ctrl + k + Ctrl + l	展开或收起当前章节
Ctrl + k + Ctrl + e	打开资源管理器
Ctrl + k + Ctrl + r	打开键盘快捷方式 PDF
Ctrl + k + Ctrl + t	打开颜色主题
Ctrl + k + Ctrl + p	当前打开的文档

原生快捷键
通用操作
Ctrl + C, 复制当前文本
Ctrl + V, 粘贴当前文本
Ctrl + Z, 撤销
Ctrl + Shift + Z, 反撤销
Shift + Alt + F, 整理代码
Ctrl + /, 将当前行注释 / 反注释, 当多行文本被选中时, 将多行文本注释


光标操作
Ctrl + ← 将光标向左移动一个单词
Ctrl + → 将光标向右移动一个单词
Ctrl + Alt + ↑, 向上加入一个光标
Ctrl + Alt + ↓, 向下加入一个光标
Ctrl + Alt + U, 撤销上次光标操作


界面移动
Ctrl + ↑ 向上移动当前界面
Ctrl + → 向下移动当前界面


选中操作
Shift + ← 向左选中 / 反选中一个字符(重要)
Shift + → 向右选中 / 反选中一个字符(重要)
Ctrl + Shift + ← 向左选中 / 反选中一个单词(重要)
Ctrl + Shift + → 向右选中 / 反选中一个单词(重要)
Ctrl + D当前有选中文本时, 将下一个与其相同的文本加入选中 (重要)


文本行操作
Ctrl + C当前无选中文本时, 复制当前行
Shift + Alt + ↑ 向上复制当前行, 当多行文本被选中时, 向上复制多行 (重要)
Shift + Alt + ↓ 向下复制当前行, 当多行文本被选中时, 向下复制多行 (重要)
Alt + ↑ 向上移动当前行, 当多行文本被选中时, 将当前多行文本向上移动 (重要)
Alt + ↓ 向下移动当前行, 当多行文本被选中时, 将当前多行文本向下移动 (重要)


插件增加的快捷键
Markdown 语法
Ctrl + B当前有选中文本时, 将文本加粗
Ctrl + I当前有选中文本时, 将文本变为斜体
Ctrl + M 进入数学公式模式 (加入美元符)


图片粘贴
Ctrl + Alt + V 粘贴剪贴板图片 (本地)
Ctrl + Alt + V 粘贴剪贴板图片 (图床)


光标操作
Ctrl + Alt + U 将多选光标变为单选


选中操作
Shift + Alt + ← 向左复制当前选中文本 (重要)
Shift + Alt + → 向右复制当前选中文本 (重要)
Alt + ← 向左移动当前选中文本一个字符(重要)
Alt + → 向右移动当前选中文本一个字符(重要)
Ctrl + Alt + ← 向左移动当前选中文本一个单词(重要)
Ctrl + Alt + → 向右移动当前选中文本一个单词(重要)


计算器功能
Ctrl + Shift + Alt + E 计算当前选中表达式, 用等号连接并输出
Ctrl + Shift + Alt + R 计算当前选中表达式, 并替换当前选中表达式
Ctrl + Shift + Alt + D 定义当前选中表达式, 无输出


```
## VIM


### extend markdown  all  in one