# Emacs

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

# 21 [emacs](https://book.emacs-china.org/)


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


# tmux

Tmux 基本概念
session 是 Tmux 的最高层级，一个 session 可以包含多个 window；
window 是 Tmux 的第二层级，一个 window 可以包含多个 pane；
Tmux 中 buffer 概念和 Vim 类似，用于保存复制或剪切的文本或命令输出，使用 Prefix+[ 进入复制模式时就会遇到选择缓冲区和剪贴板缓冲区。
Pane（面板）如果你用过 Vim 之类的编辑器肯定不会对 Pane 概念感到陌生，Tmux 也支持类似的概念，支持横向和纵向切割面板功能。

功能	命令
水平切割（上下两半）	Prefix + "
竖直切割（左右）	Prefix + %
调整面板宽度／高度	Prefix - 方向键
窗口（window）
窗口的层级要高于面板，作用类似于标签页，默认会在终端的底部显示窗口列表。

功能	命令
创建新窗口	Prefix + c
重命名窗口	Prefix + $
切换到某个窗口	Prefix + 窗口 ID
会话（session）
会话的层级更高于窗口，在终端输入 tmux 会创建并进入一个新的会话，你可以使用会话来区分 使用者或者任务。

功能	命令
创建并进入新会话	tmux
进入未关闭的会话	tmux attach 会话名
退出但保留当前会话	Prefix + d
列出所有会话	Prefix + s
重命名当前会话	Prefix + $
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
100idesu [ESC]
## windwow
1. :sp file 上面的这个命令将会上下分割当前文件和新打开的 file 。
2. 
   Ctrl + w l 将当前的光标定位到右边的屏幕
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
1. ctrl v d
2. ctrl v I [ESC] [ESC]
3. ctrl v J join to one line

##  f

1. ctrl f /ctrl b h j k l w e   i,I,a,A,o,O,s,S
2.  
      0 移动到行头
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
5. 
    % : 匹配括号移动，包括 (, {, [. （陈皓注：你需要把光标先移到括号上）
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




