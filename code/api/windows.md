# windows
## TOOL
1. honeycam bandzip honeyview screenToGif
2. bitcomet
## winger
windows package
1. grep awk curl  sed
[](https://winstall.app/generate)
## PowerShell
> ctrl shift control panel
### install extend
```shell
      # 1. 安装 PSReadline 包，该插件可以让命令行很好用，类似 zsh
      Install-Module -Name PSReadLine  -Scope CurrentUser

      # 2. 安装 posh-git 包，让你的 git 更好用
      Install-Module posh-git  -Scope CurrentUser

      # 3. 安装 oh-my-posh 包，让你的命令行更酷炫、优雅
      Install-Module oh-my-posh -Scope CurrentUser

      # 设置主题：
      Set-PoshPrompt -Theme "Paradox"
      # C:\Users\whfever\Documents\WindowsPowerShell\Modules\oh-my-posh
      $env:PSModulePath += ";C:\Users\whfever\Documents\WindowsPowerShell\Modules\oh-my-posh"
      # 重新导入
      Import-Module oh-my-posh

      ```

      ```shell
      Install-Module -Name PSReadLine -Scope AllUsers -Force -SkipPublisherCheck
Install-Module posh-git -Scope AllUsers
Install-Module oh-my-posh -Scope AllUsers
```
安装后，C:\Users\用户名\AppData\Local\oh-my-posh，如果这个目录下面是没有themes文件夹的话，执行Get-PoshThemes会导致报错，找不到themes

在C:\Users\用户名\AppData\Local\Programs\oh-my-posh这个文件夹下面，找到themes文件夹，复制到上面的文件夹中即可


### Command
1. code $profile
```shell
Import-Module PSReadLine  ## 这个工具主要做命令提示管理等操作，默认集成在了 PowerShell 中，不需要安装
Set-PSReadlineKeyHandler -Key Tab -Function Complete  ## 设置 Tab 键补全
Set-PSReadLineKeyHandler -Key "Ctrl+d" -Function MenuComplete  ## 设置 Ctrl+D 为菜单补全和 Intellisense
Set-PSReadLineKeyHandler -Key "Ctrl+z" -Function Undo  ## 设置 Ctrl+Z 为撤销
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward  ## 设置向上键为后向搜索历史记录
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward  ## 设置向下键为前向搜索历史记录
oh-my-posh init pwsh --config "C:\Users\whfever\AppData\Local\oh-my-posh\themes\xtoys.omp.json" | Invoke-Expression
```

# 路径C:\Users\whfever\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1
    
### profile
1. oh my posh
2. power shell
                "commandline": "%SystemRoot%\\System32\\WindowsPowerShell\\v1.0\\powershell.exe -NoLogo",
   1. 
## Kail

1. sudo apt install `<deb name>`
2. sudo dpkg -i .deb

xhsell connect vm

1. ifconfig **inet: addresss**
2. sudo /etc/init.d/ssh status/start

## EveryThing

1. local http

## Clash
1. 忽略代理 settings  system proxy
   TUN模式，真正的系统代理。Allow LAN，同局域网机器（比如 swtich）共享你的代理。
```js
bypass: 
  - "example.com" # 下面字段可不删除
  - 127.0.0.1
  - "*.abc.com"
  - ...
```
```json
{
DOMAIN-SUFFIX: 域名后缀匹配,
DOMAIN: 域名匹配,
DOMAIN-KEYWORD: 域名关键字匹配,
IP-CIDR: IP段匹配,
SRC-IP-CIDR: 源IP段匹配,
GEOIP: GEOIP数据库（国家代码）匹配,
DST-PORT: 目标端口匹配,
SRC-PORT: 源端口匹配
PROCESS-NAME: 源进程名匹配
RULE-SET: Rule Provider规则匹配
MATCH: 全匹配
}

```
###游戏代理
1. [](https://docs.reiz.link/游戏代理/clash-tun%2f)
## Power keys

1. ctrl space preview

## Edge

1. mode ctrl shift , .
2. alt e+x
3.


### plugin
1. edge: //flags/#use-angle  **webgl**
1. icon : material

Octotree
Octotree 是一个 GitHub 增强扩展，提供 GitHub 项目的树形视图，在线预览一个层级复杂的项目的文件结构很方便。

Octotree

ZenHub
ZenHub 为 GitHub 提供了一系列增强功能，包括但不限于搜索项目作者的其它项目、Issue 分类及增强、 拖拽上传文件、“+1”功能、任务分类等等，也是 GitHub 官方推荐插件之一。

ZenHub

Isometric Contributions
Isometric Contributions 可以根据 GitHub Commit 信息提供了一个 3D 视图。

GitHub Hovercard
GitHub Hovercard 为 GitHub 项目、用户、组织甚至一些 commit 提供了详细的 hover 卡片。

GitHub Hovercard

Porter Plug
Porter Plug 提供 GitHub 项目的最近新闻以及项目的 star 数目变化。

Porter Plug

Pretty Pull Requests
Pretty Pull Requests 为 GitHub 项目提供文件结构树形图。

Postman
REST API 测试工具

Postman Interceptor
自动监控网络请求，并发送给 Postman。

JSON Formatter
JSONViewer 用于格式化 JSON 请求数据。

LiveReload
LiveReload 是 livereload.com 出品的 Chrome 插件，支持众多第三方应用， 和 Python 自动刷新工具 LiveReload 配合使用非常方便。

Cross Share AirDrop
Cross Share AirDrop 是一个为 Chrome 添加 AirDrop 分享功能的插件，需要配合 Mac OS X App Cross Share 使用。

桌面 App 和 Chrome 插件的源代码都托管在了 GitHub， 还提供了 Alfred Workflow 支持。

使用方法
打开 crossshare: //<service>?<query> 的链接时会自动启动 CrossShare。

service 可选值包括 airdrop, email, message, twitter, facebook
query 可选值包括 ``text, url, image```
你可以在浏览器地址栏直接输入 CrossShare 链接，或者在控制台使用 open 命令: 

open "crossshare: //airdrop?url=http: //windrunner.me"
open "crossshare: //airdrop?image=\<image file path\>"
open "crossshare: //email?text=body"
Tampermonkey
Tampermonkey 俗称油猴脚本，是一个可以编写脚本增强网页功能的插件。你可以自己编写，可以在脚本分享网站上寻找需要的网站。这里有一些我自己编写的脚本: kxxoling/monkey_scripts。

Image Search Options
Image Search Options 提供相似图片搜索功能，支持 Google、SauceNAO、TinEye、IQDB 等网站。

Image Search Options

Ugoira2GIF
Ugoira2GIF 能够将 P 站的图片转换成 gif 格式。

Ugoira2GIF

Proxy SwitchyOmega
Proxy SwitchyOmega 是一个浏览器代理设置和管理插件，前身是 Switchy Sharp。

web 开发相关

React Developer Tools   Redux DevTools   Vue.js devtools
## 问题
1. 鼠标消失 : 修改windows 文本选择 鼠标样式

# Idea
## extend
1. icon : material
2. 
