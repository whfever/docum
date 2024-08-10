# Command

## windows

shutdown /r /fw   重启进入bios

## Git

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

git config --global --unset http.proxy 
git config --global --unset https.proxy 
git config --global --unset http.https://github.com.proxy 
git config --global --unset https.https://github.com.proxy 
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

# Quick

## 

```shell
npm config ls
```

## python

```shell
python -m site
# /python/Lib/Site.py USER_SITE USER_BASE
# test 
python -c "import wordcloud"
# env
pip  show pip
#
```
