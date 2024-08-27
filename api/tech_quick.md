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
npm config set registry https://registry.npmmirror.com
npm update
# 假设本地代理端口为8080
npm config set proxy "http://localhost:7890"
npm config set https-proxy "http://localhost:7890"

# 有用户密码的代理
npm config set proxy "http://username:password@localhost:7890"
npm confit set https-proxy "http://username:password@localhost:7890"

npm config delete proxy
npm config delete https-proxy
```

## wsl

```bash
# install
wsl -l -o
# export
wsl -t Ubuntu-22.04
wsl -l --all -v
wsl --export Ubuntu-20.04 d:\wsl-ubuntu20.04.tar
wsl --unregister Ubuntu-20.04
wsl --import Ubuntu-20.04 d:\wsl-ubuntu20.04 d:\wsl-ubuntu20.04.tar --version 2
ubuntu2004 config --default-user Username
del d:\wsl-ubuntu20.04.tar
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

## Anacoda

https://developer.nvidia.com/cudnn-downloads cudnn

https://pytorch.org/get-started/locally/  版本

```bash
#activate
cat >> ~/.bashrc << EOF
export PYTHONIOENCODING=UTF-8
EOF
conda init cmd.exe,bash
conda config --set show_channel_urls yes
conda clean -i    
onda create -n python310 python=3.10
conda activate python310
conda remove --name python310 --all
conda install -n python310 requests
conda remove -n python310 requests
conda deactivate
 conda remove -n env_name --all 
# package
conda list
pip list
conda update package_name --all
conda remove package_name
pip uninstall package_name
conda search keyword
#txt
pip freeze > requirements.txt
conda list -e > requirements.txt
conda install --yes --file requirements.txt 
#yml
conda env export > environment.yml
conda env create -f environment.yml -p /user/username/anaconda3/envs/env_name

#pytorch
nvidia-smi 12.4
conda install pytorch torchvision torchaudio pytorch-cuda=12.4 -c pytorch -c nvidia
conda install --offline pytorch-2.1.1-py3.9_cuda11.8_cudnn8_0.tar.bz2
import torch
torch.cuda.is_available()
```



```yml
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  deepmodeling: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/
```





## Mirror

https://mirrors.tuna.tsinghua.edu.cn/help/pypi/



# linux

```bash
ps -ax | less
ps -u pungki
ps -f -C getty  #格式化
#thread
ps -L 1213 
apt-get install psmisc
pstree -ap
cat /proc/${pid}/status # ls /proc/${pid}/task
top -H -p ${pid} 或者 top -p ${pid} 然后 shitf + H
ps -hH -p ${pid} | wc -l
watch -n 1 ‘ps -aux --sort -pmem, -pcpu |head 20
 watch -n 1 ‘ps -aux -U pungki u --sort -pmem, -pcpu | head 20’’ 

#Top c
lmctzZ  ij
f < > space enter esc


# kill
kill $(ps -ef | grep 'process_name' | grep -v grep | awk '{print $2}')

# nc
sudo apt-get -y install netcat-traditional 
```
