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