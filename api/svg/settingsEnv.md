# windows

> Idea Vscode Edge Tabby Marktext

## path

C:\ProgramData\Microsoft\Windows\Start Menu

## Service

- bandzip bandview everything  snipaste  powerkeys
  
  - q-dir clash  bookxnote   windwosTop  unigetUI

- vscode marktext  xmind  mindatom
  
  - dbever tabby xshell git-cmd  debian
    Win+1 ：idea
    Win+2 ：shell，tabby
    Win+3 ：Vscode
    Win+4 : Edge
    Win+5  ：Marktext/Wps
    F3：boo，eng，xm ，qdir，kimi
    F4：xshell，pycharm，dbever，idea
    F6： SystemProxy
    F7:pip
    F9：snicky
    F10：transparent
    Alt+ Q ~ : Alt  1 2 3 4
    shell: startup  C:\Users\Administrator\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup\
  - moderncsv 

## disk

- picture
  
  - wall

- doc
  
  - bakup
  - docum
  - commit
  - work

- video

- natural
  
  - floor
  - instr
  - env

- down

- buffer
  
  ## Install

```shell
winget
winget  install  feishu --location D:\natural\win
oh-my-posh
sudo apt install `<deb name>`
sudo dpkg -i .deb
```

## Edge

extend

- Octotree  scihub zenhub  react vue
- Tampermonkey  buautifyread   itab
- vim globalspeed   eagle yiban chenjinshi

## Steam

shortkeys

1. ALt enter alt F4 大小屏幕

# Command

## windows

### wsl

```bash
shutdown /r /fw   重启进入bios
wsl -l --all -v
wsl --export Ubuntu-20.04 d:\wsl-ubuntu20.04.tar
wsl --unregister kali-linux
wsl --import Ubuntu-20.04 d:\wsl-ubuntu20.04 d:\wsl-ubuntu20.04.tar --version 2
ubuntu2004 config --default-user Username
6.删除tar文件(可选)
del d:\wsl-ubuntu20.04.tar
```

### apt  下载源

```bash
su vim /etc/apt/sources.list

# 官方源
# deb http://http.kali.org/kali kali-rolling main non-free contrib
# deb-src http://http.kali.org/kali kali-rolling main non-free contrib
#根据需要自己选一个，中科大的还可以
#中科大
deb https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye main non-free contrib
deb https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb-src https://mirrors.aliyun.com/debian-security/ bullseye-security main
deb https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-updates main non-free contrib
deb https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib
deb-src https://mirrors.aliyun.com/debian/ bullseye-backports main non-free contrib



cat > /etc/apt/sources.list << EOF
deb http://mirrors.tencentyun.com/debian/ buster main contrib non-free
deb-src http://mirrors.tencentyun.com/debian/ buster main contrib non-free

deb http://mirrors.tencentyun.com/debian/ buster-updates main contrib non-free
deb-src http://mirrors.tencentyun.com/debian/ buster-updates main contrib non-free

deb http://mirrors.tencentyun.com/debian/ buster-backports main contrib non-free
deb-src http://mirrors.tencentyun.com/debian/ buster-backports main contrib non-free

deb http://mirrors.tencentyun.com/debian-security/ buster/updates main contrib non-free
deb-src http://mirrors.tencentyun.com/debian-security/ buster/updates main contrib non-free
EOF

cat > /etc/apt/sources.list << EOF
deb http://mirrors.cloud.aliyuncs.com/debian/ buster main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster main contrib non-free

deb http://mirrors.cloud.aliyuncs.com/debian/ buster-updates main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster-updates main contrib non-free

deb http://mirrors.cloud.aliyuncs.com/debian/ buster-backports main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian/ buster-backports main contrib non-free

deb http://mirrors.cloud.aliyuncs.com/debian-security/ buster/updates main contrib non-free
deb-src http://mirrors.cloud.aliyuncs.com/debian-security/ buster/updates main contrib non-free
EOF

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
npm config set prefix "D:\\natural\floor\\prefix"    （全局安装模块）
npm config set cache "D:\\natural\\floor\\precache"        （缓存模块）

npm install -g cnpm --registry=https://registry.npm.taobao.org
npm set registry https://registry.npm.taobao.org/
npm config set registry https://registry.npmjs.org
```

## Word

1.img  ^g 替换 全部

## backups

### service

1. ventory  
2. tech： terminus  nextssh  typora finalshell utools  tengcentDesktop   quicklook
3. wireshark 
4. 

## EdgeTODO

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