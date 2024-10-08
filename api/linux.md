# Linux

## Emacs

HOME | Links | About | Read
Emacs笔记
不是教程，非常不完整 文末我贴了几个不错的链接

Table of Contents
markdown-mode编辑快捷键
常用的Emacs缓冲区命令
常用的Emacs编辑命令
文本调换
转换大小写
缩进和填充文本
高级文本操作
命令行选项
书签
窗口操作
Emacs框架操作命令
链接
markdown-mode编辑快捷键
`C-c C-t n`插入hash样式的标题，其中n为1-5,表示第一级到第五级标题
`C-c C-t t`插入ubderline样式的一级标题
`C-c C-t s`插入underline样式的二级标题
`C-c C-a l`插入链接
`C-c C-i i`插入图像
`C-c C-s b`插入引用内容
`C-c C-s c`插入代码
`C-c C-p b`加粗
`C-c C-p i`斜体
`C-c -` 插入水平线
S-TAB可以在大纲视图，目录视图及正常视图之间切换

常用的Emacs缓冲区命令
`C-x C-s`保存当前缓冲区
`C-x C-c`要求将未保存的缓冲区保存，并退出
`C-x C-c`挂起Emacs并使成为后台进程
`C-x k`杀死一个缓冲区
`C-x C-q`切换当前状态的可读状态
`C-x i`在插入点插入文件的内容
`C-x b`切换缓冲区
连续按两次`C-x C-z`会出现异常

常用的Emacs编辑命令
`C-d`删除插入点的字符
`M-d`从插入点开始向后删除字符，直到单词结尾
`M-Backspace\M-del`从插入点开始向前删除字符，直到单词开头
`C-\C-x u`撤销上一次的键入或者操作
`C-g C-`恢复键入和操作
`C-u 次数 命令`按总的次数（默认4次）连续执行命令
`M-m`跳转到行首的非空字符
`M->\M-<`跳转到文末和文首
`C-u 次数 命令`如果命令为移动命令，就可以快速的移动光标

文本调换
`C-t`调换字母
`M-t`调换单词
`C-x C-t`调换行
`C-u 2 C-x C-t`连续调换两次
转换大小写
`M-u`将光标处到光标词尾的所有文本转换为大写
`M-l`将光标处到光标词尾的所有文本转换为小写
`M-c`转换所在处字母大小写并调到词尾
`C-o`在光标下方打开一个新行而不移动光标

缩进和填充文本
`C-s 【字符串】【C-w】【C-y】`向前搜索字符串（默认为上一次的搜索字符串），`C-w`使用光标到词尾的文本，`C-y`使用光标到行尾的文本。
`C-r 【字符串】【C-w】【C-y】`向后，同上
`C-s Enter C-w 单词或者短语`向前搜索单词和短语（不管他们之间的间隔）
`C-r Enter C-w 单词或者短语`向后，同上
`C-M-s/C-M-r`向前和向后搜索正则表达式
`M-%`向后搜索字符串，并询问替换
`C-M-%`向后搜索正则表达式，并询问替换
高级文本操作
`M-n 【操作】`重复执行操作n次
`M-4 space`输入4个空格
`M-5 C-n`向下移动5行
执行一个命令，然后`C-x z`重复执行一次，接着就可以通过键入z来反复执行这个命令。 `C-u n`也可以重复执行一个命令

命令行选项
`emacs +number 文件名`打开文件移动到number的行
`emacs +3:12 文件名`打开文件，并移动到第3行的第12个字符
`emacs -nw`以终端模式启动
书签
`C-x r m bookmark`设置一个名为bookmark的书签
`C-x r l`列出所有已保存的书签
`函数bookmark-delete`删除一个书签
`C-x r b bookmark`跳转到名为bookmark的书签所设置的位置
`函数bookmark-save`将所有的书签保存到书签文件~/.emacs.bmk
窗口操作
`C-x 2`水平划分窗口
`C-x 4 c-o`在另一个窗口显示一个缓冲区（同时保留当前窗口为活动）
`C-x 4 b`将窗口水平划为2半，并将其作为活动区
`C-x 4 f`在新的缓冲区打开新的文件，水平划分
`C-x 4 r`在新的只读缓冲区打开新的文件，水平划分
`C-M-v`滚动到下一个由`C-x o`切换到的窗口
`C-x o`光标移动到下一个窗口
`C-x b`切换到当前窗口的另一个缓冲区
`C-x 0`删除当前窗口，并将光标移动到使用`C-x o`切换到下一个窗口
`C-x 1`删除当前窗口之外的所有窗口
`C-x 4 0`删除当前窗口，并剪切它的缓冲区
`C-x 3`垂直划分窗口
`C-x `使当前窗口增加一行的高度
`C-x }/C-x {`使当前窗口减少或者增加一列的宽度
`C-x -`缩小当前窗口到最小尺寸
`C-x +`平衡所有窗口尺寸，使它们大小大致相等
`函数compare-windows`当前窗口和下一个窗口进行比较
Emacs框架操作命令
`C-x 5 2`生成一个新的Emacs框架，并使其成为活动框架
`C-x 5 b`在另一个框架中打开指定的缓冲区，不存在其他框架，则创建
`C-x 5 f`在另一个框架打开指定文件，不存在其他框架，则创建
`C-x 5 r`在另一个框架只读缓冲区打开指定文件，不存在其他框架，则创建
`C-x 5 o`移动到下一个框架，并使下一个框架成为活动框架
`C-x 5 0`删除当前框架，并使其成为活动框架
`C-x 5 1`删除当前框架之外的所有框架
`C-z`图标化当前框架，如果已图标化，则取消它的图标化
链接
Emacs的org-mode org-mode是最好的编辑利器，没有之一

## app

1. monsterWm 平铺窗口管理器
   
   ## start

核心理念：

* 文本编辑
* 备份  ~
* undo redo
* extend
  [evil](https://github.com/jiacai2050/dotfiles/commit/db5c7c28a355e21250e6dba8e13c56c71f00c81e):keyboard
  [org-mode](https://orgmode.org/) md
  [Magit](https://magit.vc/)
  [Dired](https://www.gnu.org/software/emacs/manual/html_node/emacs/Dired.html) 批量删除文件
  [Ivy/Conunsel/swiper](https://github.com/abo-abo/swiper) 补全
  [lsp-mode](https://github.com/emacs-lsp/lsp-mode) 高亮
  [ace-jump-mode](https://github.com/winterTTr/ace-jump-mode)
  [treemacs](https://github.com/Alexander-Miller/treemacs)

## 21 [emacs](https://book.emacs-china.org/)

## blog

Emacs *布道*
这里主要是一些布道内容，用于展示 Emacs 的强大：

玩游戏 2048贪吃蛇：M-x snake。（这是 Emacs 内置游戏，按键解释见后面）
俄罗斯方块：M-x snake。（这是 Emacs 内置游戏，按键解释见后面）
任务管理 org-mode（建议上 YouTube 搜索大师的 org-mode 展示）
浏览网页 WebKit  偷偷上 Twitter  国内论坛  v2ex-mode  
在线聊天 Slack：emacs-slack
git 可视化Magit：GUI 的简单 + CLI 的强大！
内置 Terminal
（看完官网不理解这些插件的强大之处的话，建议上 YouTube 观看大师的使用方式。）

基本快捷键
Emacs 快捷键基本中的基本是这几个：

C（Ctrl 或者 control 键）
M（Meta，PC 中对应 Alt，Mac 上对应 option），Meta 键来自 Solaris，常见 PC 都不具备该键
S（uper，PC 对应 Win 键，Mac 对应 command）

1. 由于 Emacs 的快捷键通常都比较长，或者某个后台任务进程不够 nice 导致 Emacs 假死的时候也可以尝试使用 C-g。

2. 相对于快捷键功能，Emacs 更多的功能并没有绑定快捷键，这些功能都对应了 Elisp 的函数，你可以按下 M-x 来运行这些函数。

3. 对于需要在 Shell 中完成的任务，你可以按下 M-! 通常也就是 Meta-Shift-1，这样就能在 Emacs 中打开一个终端而不用单独开启 Shell 会话。

4. 你可以直接输入 emcas somefile 直接打开文件，也可以启动 Emacs 后输入 C-x C-j 输入文件地址来打开指定文件。

5. 保存文件命令通常是 S-s，在 macOS 中就是 CMD-s，和大多数 Mac 程序操作一致。

6. 如果你想保存并退出 Emacs，可以输入** C-x C-c**，它会提示保存当前 buffer 并在那之后退出 Emacs。

学会使用文档
Emacs 强大的自文档也是其优势之一，任何时候你想要查找文档都无需离开 Emacs 自身。查看帮助文档相关命令的快捷键都是以 C-h 为 prefix 键：

C-h k：查看某个快捷键（组合）的用处或者绑定的函数。
C-h f：查看某个函数的作用，以及它绑定的快捷键。
C-h v：查看某个 Emacs 变量的值。
Shell 相关命令
正如前面提到的，Emacs 的很多操作/控制命令都被 Shell（Bash/Zsh）所借鉴，并命名之“Emacs 模式”，可以看出 Emacs 究竟由多大的影响力！（很多 Shell 也支持 Vim 模式，不过通常默认都是 Emacs 模式。）

文本编辑命令
Ctrl + a：移到命令行首
Ctrl + e：移到命令行尾
Ctrl + f：按字符前移（右向）
Ctrl + b：按字符后移（左向）
Meta + f：按单词前移（右向）
Meta + b：按单词后移（左向）
Ctrl + xx：在命令行首和光标之间移动
Ctrl + u：从光标处删除至命令行首
Ctrl + k：从光标处删除至命令行尾
Ctrl + w：从光标处删除至字首
Meta + d：从光标处删除至字尾
Ctrl + d：删除光标处的字符
Ctrl + h：删除光标前的字符
Ctrl + y：粘贴至光标后
Meta + c：从光标处更改为首字母大写的单词
Meta + u：从光标处更改为全部大写的单词
Meta + l：从光标处更改为全部小写的单词
Ctrl + t：交换光标处和之前的字符
Meta + t：交换光标处和之前的单词
控制命令
Ctrl + r：向后搜索
Ctrl + g：退出某个模式
Ctrl + p/n：上下选择
配置脚本 Emacs Lisp
Elisp 作为 Lisp 最流行的方言之一，拥有强大的表现能力和灵活性。如果你不曾学习过类似的编程语言也不用担心 Elisp 是一门非常简单、学习曲线非常平坦的编程语言，只需几分钟到几小时（取决与你的编程经验）就可以入门，虽然还不足以读懂高手的配置，但已经足够编写和修改简单的配置脚本了。

另外，Elisp 作为最正统的函数式编程语言之一，对于真正热爱编程的人来说决定是非常值得学习的一门编程语言。

推荐教程：Learn X language in Y minute.。文档很短，找个在线 Elisp REPL 环境相信很快就能了解它的基本概念。

高手配置
和 Vim 一样，GitHub 上早有开源的高手们的配置了，并且支持灵活的扩展。比如：

purcell/emacs.d 对流行编程语言都有基本的支持，更高级的功能需要自己扩展。Purcell 的配置的另一个问题是文档太多稀缺，作为新手实在是有点难以下手。
redguardtoo/emacs.d 国人的 Emacs 配置，fork 自上面的库，内置了中文支持等众多自定义设置。
bbatsov/prelude 相对全功能，但在试用了一段时间后发现自定义设置和我的个人习惯有较大冲突，遂弃。
Spacemacs 结合了 Emacs 和 Vim 的优点，漂亮的 UI 即使不想要任何 Vim 特性也值得你一试。

## keyboard

编辑命令
Ctrl + a ：移到命令行首
Ctrl + e ：移到命令行尾
Ctrl + f ：按字符前移（右向）
Ctrl + b ：按字符后移（左向）
Alt + f ：按单词前移（右向）
Alt + b ：按单词后移（左向）
Ctrl + xx：在命令行首和光标之间移动
Ctrl + u ：从光标处删除至命令行首
Ctrl + k ：从光标处删除至命令行尾
Ctrl + w ：从光标处删除至字首
Alt + d ：从光标处删除至字尾
Ctrl + d ：删除光标处的字符
Ctrl + h ：删除光标前的字符
Ctrl + y ：粘贴至光标后
Alt + c ：从光标处更改为首字母大写的单词
Alt + u ：从光标处更改为全部大写的单词
Alt + l ：从光标处更改为全部小写的单词
Ctrl + t ：交换光标处和之前的字符
Alt + t ：交换光标处和之前的单词
Alt + Backspace：与 Ctrl + w ~~相同~~类似，分隔符有些差别 [感谢 rezilla 指正]
重新执行命令
Ctrl + r：逆向搜索命令历史
Ctrl + g：从历史搜索模式退出
Ctrl + p：历史中的上一条命令      
Ctrl + n：历史中的下一条命令
Alt + .：使用上一条命令的最后一个参数
控制命令
Ctrl + l：清屏
Ctrl + o：执行当前命令，并选择上一条命令
Ctrl + s：阻止屏幕输出
Ctrl + q：允许屏幕输出
Ctrl + c：终止命令
Ctrl + z：挂起命令
Bang (!) 命令
!!：执行上一条命令
!blah：执行最近的以 blah 开头的命令，如 !ls
!blah:p：仅打印输出，而不执行
!$：上一条命令的最后一个参数，与 Alt + . 相同
!$:p：打印输出 !$ 的内容
!*：上一条命令的所有参数
!*:p：打印输出 !* 的内容
^blah：删除上一条命令中的 blah
^blah^foo：将上一条命令中的 blah 替换为 foo
^blah^foo^：将上一条命令中所有的 blah 都替换为 foo
_友情提示_：

以上介绍的大多数 Bash 快捷键仅当在 emacs 编辑模式时有效，若你将 Bash 配置为 vi 编辑模式，那将遵循 vi 的按键绑定。Bash 默认为 emacs 编辑模式。如果你的 Bash 不在 emacs 编辑模式，可通过 set -o emacs 设置。

^S、^Q、^C、^Z 是由终端设备处理的，可用 stty 命令设置。

{ via }

## deepin

# MAC

## Shell

1. iTerm2 Warp

## tmux

Tmux 基本概念
session 是 Tmux 的最高层级，一个 session 可以包含多个 window；
window 是 Tmux 的第二层级，一个 window 可以包含多个 pane；
Tmux 中 buffer 概念和 Vim 类似，用于保存复制或剪切的文本或命令输出，使用 Prefix+[ 进入复制模式时就会遇到选择缓冲区和剪贴板缓冲区。
Pane（面板）如果你用过 Vim 之类的编辑器肯定不会对 Pane 概念感到陌生，Tmux 也支持类似的概念，支持横向和纵向切割面板功能。

功能    命令
水平切割（上下两半）    Prefix + "
竖直切割（左右）    Prefix + %
调整面板宽度／高度    Prefix - 方向键
窗口（window）
窗口的层级要高于面板，作用类似于标签页，默认会在终端的底部显示窗口列表。

功能    命令
创建新窗口    Prefix + c
重命名窗口    Prefix + $
切换到某个窗口    Prefix + 窗口 ID
会话（session）
会话的层级更高于窗口，在终端输入 tmux 会创建并进入一个新的会话，你可以使用会话来区分 使用者或者任务。

功能    命令
创建并进入新会话    tmux
进入未关闭的会话    tmux attach 会话名
退出但保留当前会话    Prefix + d
列出所有会话    Prefix + s
重命名当前会话    Prefix + $
Tmux Prefix（prefix）
Tmux 使用 Prefix 以将自身的快捷键与其它应用区分，运行 Tmux 快捷键时首先按下这个 Prefix（默认是 Ctrl-b 组合键），松手后紧接着按下对应操作的快捷键。

比如，如果我想要列出所有的 Tmux 会话（对应快捷键是 s）需要这样：

按下 Ctrl-b 组合键（默认 Prefix ）；
放开 Ctrl-b；
按下 s 键。
Tmux 配置文件的默认地址是 ~/.tmux.conf，每次启动 Tmux 时都会加载该文件。

修改 Prefix（ Prefix ）
Tmux 的配置未见位置是 ~/.tmux.conf，修改 Tmux Prefix 首先需要取消绑定原有的的 Prefix ； 再设置新的 Prefix，这里以 Ctrl-w 为例：

unbind C-b
set -g prefix C-w
Tmux 基本操作
Tmux 基本操作是以 Prefix + 操作 的方式进行的，比如 Prefix+c 会创建一个新的窗口，或者 Prefix+: 进入命令模式直接输入命令。大体可以分为：

会话操作：
增加会话：tmux new -s 会话名
列出会话：tmux ls
进入会话：tmux attach -t 会话名
退出会话：tmux detach
窗口操作：
创建窗口：Prefix + c
切换窗口：Prefix + 窗口 ID
重命名窗口：Prefix + $
关闭窗口：Prefix + &
面板操作：
水平切割：Prefix + "
竖直切割：Prefix + %
切换面板：Prefix + 方向键
关闭面板：Prefix + x
调整面板宽度／高度：Prefix + 方向键
其它：
重新加载配置文件：Prefix + r
列出所有快捷键：Prefix + ?
列出所有会话：Prefix + s
安装插件：Prefix + I
卸载插件：Prefix + u
窗口/面板转换：Prefix + !
横向/纵向布局转换：Prefix + space
更多操作可以参考 Tmux CheatSheet。

绑定快捷键
Tmux 快捷键绑定的命令是 bind 快捷键 作用，即可将“作用”绑定在 Prefix ＋快捷键 上， 下面这行配置会将“重新加载”配置文件的操作绑定在快捷键 R 上：

bind R source-file ~/.tmux.conf \; display-message "Config reloaded..."
类 Vim 的文字选择和复制方式
选中和复制文字
你需要添加以下配置：

#（进入复制模式后）输入 'v' 开始选择
bind-key -t vi-copy v begin-selection
 将选中文字添加到系统的剪贴板中
bind-key -t vi-copy y copy-pipe "reattach-to-user-namespace pbcopy"
结对编程
tmux 有个特性，不管多少人连进同一个 tmux 会话，他们看到和操作的都是同一个东西，会话的长宽取决于输出的长款的最小值，因此可以用来进行结对编程练习。如果你想要临时和别人一起使用 tmux，可以使用 tmate 来创建共享会话：

Tmate 与远程会话共享
首先你需要安装 Tmate：

brew install tmate
Ubuntu：

sudo apt-get install software-properties-common && \
sudo add-apt-repository ppa:tmate.io/archive    && \
sudo apt-get update                             && \
sudo apt-get install tmate
输入 tmate 将会创建一个公开的远程会话（会话的底部会出现提示“[tmate] Remote session: ssh [some hash]@ny.tmate.io”），将 ssh 的地址发送给你的朋友就可以分享你的会话了！

Vim 兼容问题
主题冲突问题
如果你跟我一样使用 Vim 作为编辑器，可能同样会遇到输出黑块的问题。解决方案是在 Vim 的配置文件中加入：

if exists('$TMUX')
  set term=screen-256color
endif
在 Tmux 中遇到 Vim 主题颜色问题也可以在 .tmux.conf 配置中加入：

 set -g default-terminal "screen-256color"
set-option -sa terminal-overrides ",xterm*:Tc"
UI 主题
推荐一些主题：

Powerline Blue
Catppuccin
关闭终端时自动保存和恢复会话
关闭 Tmux 会话所在 Terminal 窗口时不会自动关闭会话，可以在新窗口中使用 tmux attach 命令恢复会话。

插件安装和管理
推荐使用 TPM（Tmux Plugin Manager）来管理插件，TMP 的安装方法见 https://github.com/tmux-plugins/tpm#installation 。

配置示例和推荐
一个简单的配置
 配置 Vim 颜色兼容
set-option -sa terminal-overrides ",xterm*:Tc"
 配置鼠标控制，可能需要 terminal 支持
set -g mouse on

 插件，默认是从 GitHub 寻找对应 repo
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

set -g @plugin 'catppuccin/tmux'
set -g @plugin 'christoomey/vim-tmux-navigator'

 控制区管理插件
set -g @plugin 'tmux-plugins/tmux-yank'

 set vi-mode
set-window-option -g mode-keys vi
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi C-v send-keys -X rectangle-toggle
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

 窗口切换快捷键
bind-key -n C-H previous-window
bind-key -n C-L next-window

 设置窗口从 1 开始而不是默认的 0，这样就可以使用 1-9 来切换窗口了，0 离其它键太远了
set -g base-index 1
set -g pane-base-index 1
set-window-option -g pane-base-index 1

 新建窗口时设置为当前目录
bind '"' split-window -v -c "#{pane_current_path}"
bind % split-window -h -c "#{pane_current_path}"

 Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)
run '~/.tmux/plugins/tpm/tpm'
Oh My Tmux
也可以试试 Oh My Tmux! 这个开箱即用的方案，或者仅作为个人配置的参考。

参考链接
官网：Tmux
Tmux 快捷键 & 便条
A Tmux crash course: tips and tweaks
持久保存 Tmux 会话

# vim

> 1. 移动we 命令:! 重复.  替换
> 2. 块v 分屏w 提示p 宏录制
>    100idesu [ESC]

## windwow

1. :sp file 上面的这个命令将会上下分割当前文件和新打开的 file 。
2. Ctrl + w l 将当前的光标定位到右边的屏幕
   Ctrl + w c 上面这个命令是关闭当前的分屏
   Ctrl + w q 上面的这个命令也是关闭当前的分屏，如果是最后一个分屏将会退出 VIM 。

1、：quit&close二者都能实现关闭窗口的功能，但是，quit会关闭最后一个窗口，而close不会关闭最后一个窗口。用close不用担心不小心退出vim。

2、：only此命令可以关闭其他所有窗口。这个命令关闭除当前窗口外的所有窗口。如果要关闭的窗口中有一个没有存盘，Vim 会显示一个错误信息，并且那个窗口不会被关闭。

3、Ctrl-w如果长按Ctrl-w那么光标会不停地在窗口之间跳转。如果眼力好的话可以使用。如果需要精确定位的话可以再加上hlkj（左右上下）像在vim当中是一样的。

4、：split此命令是分割当前窗口的，所以在使用的时候要注意把光标跳转到你需要分割的哪个窗口上之后再使用此命令。

5、：qall全部退出，如果任何一个窗口没有存盘，Vim 都不会退出。同时光标会自动跳到那个窗口你可以用 ":write" 命令保存该文件或者 ":quit!" 放弃修改。

6、:wall此命令表示 "write all" (全部保存)。但实际上，它只会保存修改过的文件。

7、vim -o one.txt two.txt three.txt这个命令就是在终端下使用的,就是一次性打开3个文件并使用分割形式显示。

## repeat

1. . 100idesu [ESC] 
   
   ## block

2. ctrl v d

3. ctrl v I [ESC] [ESC]

4. ctrl v J join to one line

## f

1. ctrl f /ctrl b h j k l w e   i,I,a,A,o,O,s,S

2. 0 移动到行头
   ^ 移动到本行的第一个不是 blank 字符
   $ 移动到行尾
   g_ 移动到本行最后一个不是 blank 字符的位置
   w 光标移动到下一个单词的开头
   e 光标移动到下一个单词的结尾
   fa 移动到本行下一个为 a 的字符处，fb 移动到下一个为 b 的字符处
   nfa 移动到本行光标处开始的第 n 个 字符为 a 的地方（n 是 1，2，3，4 ... 数字）
   Fa 同 fa 一样，光标移动方向同 fa 相反
   nFa 同 nfa 类似，光标移动方向同 nfa相反
   nG 光标定位到第 n 行的行首
   
   gg 光标定位到第一行的行首
   G 光标定位到最后一行的行首
   H 光标定位到当前屏幕的第一行行首
   M 光标移动到当前屏幕的中间
   L 光标移动到当前屏幕的尾部
   zt 把当前行移动到当前屏幕的最上方，也就是第一行
   zz 把当前行移动到当前屏幕的中间
   zb 把当前行移动到当前屏幕的尾部
   % 匹配括号移动，包括 ( , { , [ 需要把光标先移动到括号上

3. dd yy x p
   
        d 是删除的意思，通常搭配一个字符 ( 删除范围 ) 实现删除功能，常用的如下：
        dw 删除一个单词
        dnw 删除 n 个单词，
        dfa 删除光标处到下一个 a 的字符处（ fa 定位光标到 a 处 ）
        dnfa 删除光标处到第 n 个 a 的字符处
        dd 删除一整行
        ndd 删除光标处开始的 n 行
        重复命令 10dd 删除十次

4. u ctrl r

5. % : 匹配括号移动，包括 (, {, [. （陈皓注：你需要把光标先移到括号上）
   
   * 和 #:  匹配光标当前所在的单词，移动光标到下一个（或上一个）匹配单词（*是下一个，#是上一个）

## :

1. %s/1/2  i 表示大小写不敏感查找，I 表示大小写敏感：c 表示需要确认，
2. ! linux命令
   :r !date  执行 date 命令显示时间，并且添加命令结果到文本中。
3. map:map ^M I#<ESC>上面的例子也就是通过快捷键 Ctrl + m 在文件光标处所在行的行首插入 # （ # 代表注释）。
4. :ab email xxxx@gmail.com
5. :set nu  该命令会显示行号。

## pattern

1. Ctrl+n 或者 Ctrl+p 会有代码提示功能

## vimrc

[vim](https://razeen.me/posts/my-macvim-vimrc/) vim配制

# shell

### 自启动

1. 如果你想让脚本在每次开机后都能后台运行，你可以将它添加到系统的启动项中。下面是一些通用的方法来实现这个目标：
   
        Windows系统：
           
        将你的脚本存储在一个特定的文件夹中，比如C:\Scripts。按下Win + R，输入"shell:startup"，然后按回车键。
        在打开的文件夹中创建一个快捷方式，指向你的脚本文件。这样，每次Windows启动时，该脚本将自动在后台运行。
        macOS系统：
           
        将你的脚本存储在一个特定的文件夹中，比如~/Documents/Scripts。打开系统设置，点击"用户与群组"，然后选择启动项。
        点击"+"按钮，选择你的脚本文件。这样，每次macOS启动时，该脚本将自动在后台运行。
           
        Linux系统：
        将你的脚本存储在一个特定的文件夹中，比如~/Scripts。打开终端，输入以下命令：
        bash  crontab -e   在打开的编辑器中添加一行，指定在系统启动时运行你的脚本：
        bash   @reboot /path/to/your/script.sh   保存并关闭编辑器。
        这样，每次Linux系统启动时，该脚本将自动在后台运行。

# Manjaro

## Short keys

1. alt F2  :terminal 0

## pcman

1. 切换系统软件源 
   sudo pacman-mirrors -i -c China -m rank

2. 添加archlinuxcn源，获得更多的软件包
   
   # 修改/etc/pacman.conf
   
   echo -e "\n[archlinuxcn]\nServer = https://mirrors.tuna.tsinghua.edu.cn/archlinuxcn/\$arch\n\n" | sudo tee -a /etc/pacman.conf

# 导入GPG key

sudo pacman -Sy archlinuxcn-keyring
3. update system
sudo pacman -Syyu

　　尽管manjaro下的软件已经够多了，安装方式已经够灵活，官方的首推pacman，再推yay，实在不行还有aur，为了安装Xmind-for-Linux-amd64bit-22.11.3656.deb，先去官方下载包

1、安装debtap　

　　yay -S debtap

2、更新debtap

　　sudo debtap -u

3、安装deb包，产生一个压缩包

　　sudo debtap [name].deb

　　例如：sudo debtap Xmind-for-Linux-amd64bit-22.11.3656.deb就会产生一个压缩包，xmind-vana-22.11.3656-1-x86_64.pkg.tar.zst

4、pacman安装软件包

　　sudo pacman -U [package_name]

　　例如：sudo pacman -U xmind-vana-22.11.3656-1-x86_64.pkg.tar.zst

2. Q
3. input chinese  ： fcitx5  software
4. glamereader ocr
5. flameshot  ：pacman -S flameshot
6. yay ""  yay -S
7. deb 转。tar.zst   
8. 文档？
9. yay -S clash-for-windows-chinese

```
pacman:
Manjaro包管理常用命令

    对整个系统进行更新

sudo pacman -Syu   

    升级软件包

sudo pacman -Syu

    安装或者升级单个软件包，或者一列软件包（包含依赖包），使用如下命令：

sudo pacman -S package_name1 package_name2 ...

    与上面命令不同的是，该命令将在同步包数据库后再执行安装

sudo pacman -Sy package_name

    安装本地包

sudo pacman -U local_package_name#其扩展名为pkg.tar.gz或pkg.tar.xz

    安装一个远程包

sudo pacman -U url#不在 pacman 配置的源里面，例：pacman -U http://www.example.com/repo/example.pkg.tar.xz

    在仓库中搜索含关键字的包sudo

sudo pacman -Ss keyword
```
# Linux 
## ssh
```bash
cd /usr/local/src
wget -c http://www.openssl.org/source/openssl-3.3.1.tar.gz
tar xzvf openssl-3.3.1.tar.gz
 cd openssl-0.9.8d
 apt install perl
 apt  install gcc automake autoconf libtool make
 ./Configure --prefix=/usr/local/openssl
 make
 make test
 make install

```

## locale

```bash
sudo apt install locales
sudo dpkg-reconfigure locales

sudo apt-get install -y locales && sed -i '/^[^#[:space:]]/ s/^\([^#]\)/# \1/' /etc/locale.gen && sed -i '/^# en_US\.UTF-8 UTF-8/ s/^# //' /etc/locale.gen && sudo locale-gen en_US.UTF-8 && sudo update-locale LANG=en_US.UTF-8 && source /etc/default/locale

```

## man 
```bash
vi /etc/manpath.config
man -M /usr/share/man/zh_CN
```

## net-tools

```bash
apt install net-tools
curl ifconfig.me
ip addr show eth0
ip addr | grep 'inet ' | grep -v '127.0.0.1' | awk '{print $2}' | cut -f1 -d'/'


```

## nginx
```bash
apt install nginx
whereis nginx

```