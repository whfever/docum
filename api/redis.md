# Redis
## natural
```bash
apt-get update -y 
apt-get install redis-server -y
apt-cache policy redis-server
systemctl start redis-server
systemctl enable redis-server
systemctl status redis-server

nano /etc/redis/redis.conf
#bind 127.0.0.1 -::1
requirepass securepassword


```