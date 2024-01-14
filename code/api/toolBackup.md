# Extend
## vim
### gh
```
map go createTab url="http://www.google.com"
map gls createTab url="http://www.lsdhss.com/"
map gc createTab url="http://www.chinadaily.com.cn/"
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
```

### h number
```javascript
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

```
### g number
```js

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
## clash

```yml
mixed-port: 7890
allow-lan: true
bind-address: '*'
mode: rule
log-level: info
external-controller: '127.0.0.1:9090'
dns:
    enable: true
    ipv6: false
    default-nameserver: [223.5.5.5, 119.29.29.29]
    enhanced-mode: fake-ip
    fake-ip-range: 198.18.0.1/16
    use-hosts: true
    nameserver: ['https://doh.pub/dns-query', 'https://dns.alidns.com/dns-query']
    fallback: ['https://doh.dns.sb/dns-query', 'https://dns.cloudflare.com/dns-query', 'https://dns.twnic.tw/dns-query', 'tls://8.8.4.4:853']
    fallback-filter: { geoip: true, ipcidr: [240.0.0.0/4, 0.0.0.0/32] }
proxies:
    - { name: '剩余流量：181.17 GB', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35012, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '距离下次重置剩余：12 天', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35012, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: 套餐到期：2024-01-26, type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35012, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 V', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35012, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL V', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35012, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 I', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35011, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL I', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35011, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 P', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35013, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL P', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35013, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 01', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 02', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35008, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 02', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35008, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 03', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35007, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 03', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35007, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 04', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 04', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 05', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 05', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 06', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35004, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 07', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35005, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 07', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35005, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 08', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35006, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 08', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35006, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 09', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35014, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 09', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35014, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 10', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35016, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 10', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35016, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 01 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35009, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 01 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35009, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 35010, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '香港 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35010, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 34001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 IEPL 01', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 34001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 34002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 34002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 03 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 34003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '台湾 IEPL 03 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 34003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 01 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 IEPL 01 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 03 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 IPEL 03 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 04 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31004, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '新加坡 IEPL 04 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31004, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 01 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 33001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 IEPL 01 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 33001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 33002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 33002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 03 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 33003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 IEPL 03 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 33003, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 04 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 33004, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '美国 IEPL 04 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 33004, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 01 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31401, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 IEPL 01 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31401, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31402, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31402, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 03 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31403, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 IEPL 03 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31403, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 04 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31404, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 IEPL 04 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31404, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 05 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31405, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '韩国 IEPL 05 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 31405, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '日本 01 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 32001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '日本 IEPL 01 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 32001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '日本 02 解锁', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 32002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '日本 IEPL 02 解锁', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 32002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '马来西亚 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 36001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '马来西亚  IEPL 01', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 36001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '马来西亚 02', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 36002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '马来西亚 IEPL 02', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 36002, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '菲律宾 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 39001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '柬埔寨 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31101, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '土耳其 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 37001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '越南 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 38001, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '泰国 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31201, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '俄罗斯 01', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31301, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '俄罗斯 02', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31302, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
    - { name: '俄罗斯 03', type: ss, server: e9e9w765cqzwhgrlxhs.gamesub.cc, port: 31303, cipher: chacha20-ietf-poly1305, password: df8a0f21-ba2c-47c5-a719-7ad4fa2073cc, udp: true }
proxy-groups:
    - { name: 西红柿🍅, type: select, proxies: [自动选择, 故障转移, '剩余流量：181.17 GB', '距离下次重置剩余：12 天', 套餐到期：2024-01-26, '香港 V', '香港 IEPL V', '香港 I', '香港 IEPL I', '香港 P', '香港 IEPL P', '香港 01', '香港 IEPL 01', '香港 02', '香港 IEPL 02', '香港 03', '香港 IEPL 03', '香港 04', '香港 IEPL 04', '香港 05', '香港 IEPL 05', '香港 06', '香港 07', '香港 IEPL 07', '香港 08', '香港 IEPL 08', '香港 09', '香港 IEPL 09', '香港 10', '香港 IEPL 10', '香港 01 解锁', '香港 IEPL 01 解锁', '香港 02 解锁', '香港 IEPL 02 解锁', '台湾 01', '台湾 IEPL 01', '台湾 02 解锁', '台湾 IEPL 02 解锁', '台湾 03 解锁', '台湾 IEPL 03 解锁', '新加坡 01 解锁', '新加坡 IEPL 01 解锁', '新加坡 02 解锁', '新加坡 IEPL 02 解锁', '新加坡 03 解锁', '新加坡 IPEL 03 解锁', '新加坡 04 解锁', '新加坡 IEPL 04 解锁', '美国 01 解锁', '美国 IEPL 01 解锁', '美国 02 解锁', '美国 IEPL 02 解锁', '美国 03 解锁', '美国 IEPL 03 解锁', '美国 04 解锁', '美国 IEPL 04 解锁', '韩国 01 解锁', '韩国 IEPL 01 解锁', '韩国 02 解锁', '韩国 IEPL 02 解锁', '韩国 03 解锁', '韩国 IEPL 03 解锁', '韩国 04 解锁', '韩国 IEPL 04 解锁', '韩国 05 解锁', '韩国 IEPL 05 解锁', '日本 01 解锁', '日本 IEPL 01 解锁', '日本 02 解锁', '日本 IEPL 02 解锁', '马来西亚 01', '马来西亚  IEPL 01', '马来西亚 02', '马来西亚 IEPL 02', '菲律宾 01', '柬埔寨 01', '土耳其 01', '越南 01', '泰国 01', '俄罗斯 01', '俄罗斯 02', '俄罗斯 03'] }
    - { name: 自动选择, type: url-test, proxies: ['剩余流量：181.17 GB', '距离下次重置剩余：12 天', 套餐到期：2024-01-26, '香港 V', '香港 IEPL V', '香港 I', '香港 IEPL I', '香港 P', '香港 IEPL P', '香港 01', '香港 IEPL 01', '香港 02', '香港 IEPL 02', '香港 03', '香港 IEPL 03', '香港 04', '香港 IEPL 04', '香港 05', '香港 IEPL 05', '香港 06', '香港 07', '香港 IEPL 07', '香港 08', '香港 IEPL 08', '香港 09', '香港 IEPL 09', '香港 10', '香港 IEPL 10', '香港 01 解锁', '香港 IEPL 01 解锁', '香港 02 解锁', '香港 IEPL 02 解锁', '台湾 01', '台湾 IEPL 01', '台湾 02 解锁', '台湾 IEPL 02 解锁', '台湾 03 解锁', '台湾 IEPL 03 解锁', '新加坡 01 解锁', '新加坡 IEPL 01 解锁', '新加坡 02 解锁', '新加坡 IEPL 02 解锁', '新加坡 03 解锁', '新加坡 IPEL 03 解锁', '新加坡 04 解锁', '新加坡 IEPL 04 解锁', '美国 01 解锁', '美国 IEPL 01 解锁', '美国 02 解锁', '美国 IEPL 02 解锁', '美国 03 解锁', '美国 IEPL 03 解锁', '美国 04 解锁', '美国 IEPL 04 解锁', '韩国 01 解锁', '韩国 IEPL 01 解锁', '韩国 02 解锁', '韩国 IEPL 02 解锁', '韩国 03 解锁', '韩国 IEPL 03 解锁', '韩国 04 解锁', '韩国 IEPL 04 解锁', '韩国 05 解锁', '韩国 IEPL 05 解锁', '日本 01 解锁', '日本 IEPL 01 解锁', '日本 02 解锁', '日本 IEPL 02 解锁', '马来西亚 01', '马来西亚  IEPL 01', '马来西亚 02', '马来西亚 IEPL 02', '菲律宾 01', '柬埔寨 01', '土耳其 01', '越南 01', '泰国 01', '俄罗斯 01', '俄罗斯 02', '俄罗斯 03'], url: 'http://www.gstatic.com/generate_204', interval: 86400 }
    - { name: 故障转移, type: fallback, proxies: ['剩余流量：181.17 GB', '距离下次重置剩余：12 天', 套餐到期：2024-01-26, '香港 V', '香港 IEPL V', '香港 I', '香港 IEPL I', '香港 P', '香港 IEPL P', '香港 01', '香港 IEPL 01', '香港 02', '香港 IEPL 02', '香港 03', '香港 IEPL 03', '香港 04', '香港 IEPL 04', '香港 05', '香港 IEPL 05', '香港 06', '香港 07', '香港 IEPL 07', '香港 08', '香港 IEPL 08', '香港 09', '香港 IEPL 09', '香港 10', '香港 IEPL 10', '香港 01 解锁', '香港 IEPL 01 解锁', '香港 02 解锁', '香港 IEPL 02 解锁', '台湾 01', '台湾 IEPL 01', '台湾 02 解锁', '台湾 IEPL 02 解锁', '台湾 03 解锁', '台湾 IEPL 03 解锁', '新加坡 01 解锁', '新加坡 IEPL 01 解锁', '新加坡 02 解锁', '新加坡 IEPL 02 解锁', '新加坡 03 解锁', '新加坡 IPEL 03 解锁', '新加坡 04 解锁', '新加坡 IEPL 04 解锁', '美国 01 解锁', '美国 IEPL 01 解锁', '美国 02 解锁', '美国 IEPL 02 解锁', '美国 03 解锁', '美国 IEPL 03 解锁', '美国 04 解锁', '美国 IEPL 04 解锁', '韩国 01 解锁', '韩国 IEPL 01 解锁', '韩国 02 解锁', '韩国 IEPL 02 解锁', '韩国 03 解锁', '韩国 IEPL 03 解锁', '韩国 04 解锁', '韩国 IEPL 04 解锁', '韩国 05 解锁', '韩国 IEPL 05 解锁', '日本 01 解锁', '日本 IEPL 01 解锁', '日本 02 解锁', '日本 IEPL 02 解锁', '马来西亚 01', '马来西亚  IEPL 01', '马来西亚 02', '马来西亚 IEPL 02', '菲律宾 01', '柬埔寨 01', '土耳其 01', '越南 01', '泰国 01', '俄罗斯 01', '俄罗斯 02', '俄罗斯 03'], url: 'http://www.gstatic.com/generate_204', interval: 7200 }
rules:
    - 'DOMAIN,xhs01.apiapi.top,DIRECT'
    - 'DOMAIN-KEYWORD,ding,DIRECT'
    - 'DOMAIN-SUFFIX,services.googleapis.cn,西红柿🍅'
    - 'DOMAIN-SUFFIX,xn--ngstr-lra8j.com,西红柿🍅'
    - 'DOMAIN,safebrowsing.urlsec.qq.com,DIRECT'
    - 'DOMAIN,safebrowsing.googleapis.com,DIRECT'
    - 'DOMAIN,developer.apple.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,digicert.com,西红柿🍅'
    - 'DOMAIN,ocsp.apple.com,西红柿🍅'
    - 'DOMAIN,ocsp.comodoca.com,西红柿🍅'
    - 'DOMAIN,ocsp.usertrust.com,西红柿🍅'
    - 'DOMAIN,ocsp.sectigo.com,西红柿🍅'
    - 'DOMAIN,ocsp.verisign.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,apple-dns.net,西红柿🍅'
    - 'DOMAIN,testflight.apple.com,西红柿🍅'
    - 'DOMAIN,sandbox.itunes.apple.com,西红柿🍅'
    - 'DOMAIN,itunes.apple.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,apps.apple.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blobstore.apple.com,西红柿🍅'
    - 'DOMAIN,cvws.icloud-content.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mzstatic.com,DIRECT'
    - 'DOMAIN-SUFFIX,itunes.apple.com,DIRECT'
    - 'DOMAIN-SUFFIX,icloud.com,DIRECT'
    - 'DOMAIN-SUFFIX,icloud-content.com,DIRECT'
    - 'DOMAIN-SUFFIX,me.com,DIRECT'
    - 'DOMAIN-SUFFIX,aaplimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,cdn20.com,DIRECT'
    - 'DOMAIN-SUFFIX,cdn-apple.com,DIRECT'
    - 'DOMAIN-SUFFIX,akadns.net,DIRECT'
    - 'DOMAIN-SUFFIX,akamaiedge.net,DIRECT'
    - 'DOMAIN-SUFFIX,edgekey.net,DIRECT'
    - 'DOMAIN-SUFFIX,mwcloudcdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,mwcname.com,DIRECT'
    - 'DOMAIN-SUFFIX,apple.com,DIRECT'
    - 'DOMAIN-SUFFIX,apple-cloudkit.com,DIRECT'
    - 'DOMAIN-SUFFIX,apple-mapkit.com,DIRECT'
    - 'DOMAIN-SUFFIX,126.com,DIRECT'
    - 'DOMAIN-SUFFIX,126.net,DIRECT'
    - 'DOMAIN-SUFFIX,127.net,DIRECT'
    - 'DOMAIN-SUFFIX,163.com,DIRECT'
    - 'DOMAIN-SUFFIX,360buyimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,36kr.com,DIRECT'
    - 'DOMAIN-SUFFIX,acfun.tv,DIRECT'
    - 'DOMAIN-SUFFIX,air-matters.com,DIRECT'
    - 'DOMAIN-SUFFIX,aixifan.com,DIRECT'
    - 'DOMAIN-KEYWORD,alicdn,DIRECT'
    - 'DOMAIN-KEYWORD,alipay,DIRECT'
    - 'DOMAIN-KEYWORD,taobao,DIRECT'
    - 'DOMAIN-SUFFIX,amap.com,DIRECT'
    - 'DOMAIN-SUFFIX,autonavi.com,DIRECT'
    - 'DOMAIN-KEYWORD,baidu,DIRECT'
    - 'DOMAIN-SUFFIX,bdimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,bdstatic.com,DIRECT'
    - 'DOMAIN-SUFFIX,bilibili.com,DIRECT'
    - 'DOMAIN-SUFFIX,bilivideo.com,DIRECT'
    - 'DOMAIN-SUFFIX,caiyunapp.com,DIRECT'
    - 'DOMAIN-SUFFIX,clouddn.com,DIRECT'
    - 'DOMAIN-SUFFIX,cnbeta.com,DIRECT'
    - 'DOMAIN-SUFFIX,cnbetacdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,cootekservice.com,DIRECT'
    - 'DOMAIN-SUFFIX,csdn.net,DIRECT'
    - 'DOMAIN-SUFFIX,ctrip.com,DIRECT'
    - 'DOMAIN-SUFFIX,dgtle.com,DIRECT'
    - 'DOMAIN-SUFFIX,dianping.com,DIRECT'
    - 'DOMAIN-SUFFIX,douban.com,DIRECT'
    - 'DOMAIN-SUFFIX,doubanio.com,DIRECT'
    - 'DOMAIN-SUFFIX,duokan.com,DIRECT'
    - 'DOMAIN-SUFFIX,easou.com,DIRECT'
    - 'DOMAIN-SUFFIX,ele.me,DIRECT'
    - 'DOMAIN-SUFFIX,feng.com,DIRECT'
    - 'DOMAIN-SUFFIX,fir.im,DIRECT'
    - 'DOMAIN-SUFFIX,frdic.com,DIRECT'
    - 'DOMAIN-SUFFIX,g-cores.com,DIRECT'
    - 'DOMAIN-SUFFIX,godic.net,DIRECT'
    - 'DOMAIN-SUFFIX,gtimg.com,DIRECT'
    - 'DOMAIN,cdn.hockeyapp.net,DIRECT'
    - 'DOMAIN-SUFFIX,hongxiu.com,DIRECT'
    - 'DOMAIN-SUFFIX,hxcdn.net,DIRECT'
    - 'DOMAIN-SUFFIX,iciba.com,DIRECT'
    - 'DOMAIN-SUFFIX,ifeng.com,DIRECT'
    - 'DOMAIN-SUFFIX,ifengimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,ipip.net,DIRECT'
    - 'DOMAIN-SUFFIX,iqiyi.com,DIRECT'
    - 'DOMAIN-SUFFIX,jd.com,DIRECT'
    - 'DOMAIN-SUFFIX,jianshu.com,DIRECT'
    - 'DOMAIN-SUFFIX,knewone.com,DIRECT'
    - 'DOMAIN-SUFFIX,le.com,DIRECT'
    - 'DOMAIN-SUFFIX,lecloud.com,DIRECT'
    - 'DOMAIN-SUFFIX,lemicp.com,DIRECT'
    - 'DOMAIN-SUFFIX,licdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,luoo.net,DIRECT'
    - 'DOMAIN-SUFFIX,meituan.com,DIRECT'
    - 'DOMAIN-SUFFIX,meituan.net,DIRECT'
    - 'DOMAIN-SUFFIX,mi.com,DIRECT'
    - 'DOMAIN-SUFFIX,miaopai.com,DIRECT'
    - 'DOMAIN-SUFFIX,microsoft.com,DIRECT'
    - 'DOMAIN-SUFFIX,microsoftonline.com,DIRECT'
    - 'DOMAIN-SUFFIX,miui.com,DIRECT'
    - 'DOMAIN-SUFFIX,miwifi.com,DIRECT'
    - 'DOMAIN-SUFFIX,mob.com,DIRECT'
    - 'DOMAIN-SUFFIX,netease.com,DIRECT'
    - 'DOMAIN-SUFFIX,office.com,DIRECT'
    - 'DOMAIN-SUFFIX,office365.com,DIRECT'
    - 'DOMAIN-KEYWORD,officecdn,DIRECT'
    - 'DOMAIN-SUFFIX,oschina.net,DIRECT'
    - 'DOMAIN-SUFFIX,ppsimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,pstatp.com,DIRECT'
    - 'DOMAIN-SUFFIX,qcloud.com,DIRECT'
    - 'DOMAIN-SUFFIX,qdaily.com,DIRECT'
    - 'DOMAIN-SUFFIX,qdmm.com,DIRECT'
    - 'DOMAIN-SUFFIX,qhimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,qhres.com,DIRECT'
    - 'DOMAIN-SUFFIX,qidian.com,DIRECT'
    - 'DOMAIN-SUFFIX,qihucdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,qiniu.com,DIRECT'
    - 'DOMAIN-SUFFIX,qiniucdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,qiyipic.com,DIRECT'
    - 'DOMAIN-SUFFIX,qq.com,DIRECT'
    - 'DOMAIN-SUFFIX,qqurl.com,DIRECT'
    - 'DOMAIN-SUFFIX,rarbg.to,DIRECT'
    - 'DOMAIN-SUFFIX,ruguoapp.com,DIRECT'
    - 'DOMAIN-SUFFIX,segmentfault.com,DIRECT'
    - 'DOMAIN-SUFFIX,sinaapp.com,DIRECT'
    - 'DOMAIN-SUFFIX,smzdm.com,DIRECT'
    - 'DOMAIN-SUFFIX,snapdrop.net,DIRECT'
    - 'DOMAIN-SUFFIX,sogou.com,DIRECT'
    - 'DOMAIN-SUFFIX,sogoucdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,sohu.com,DIRECT'
    - 'DOMAIN-SUFFIX,soku.com,DIRECT'
    - 'DOMAIN-SUFFIX,speedtest.net,DIRECT'
    - 'DOMAIN-SUFFIX,sspai.com,DIRECT'
    - 'DOMAIN-SUFFIX,suning.com,DIRECT'
    - 'DOMAIN-SUFFIX,taobao.com,DIRECT'
    - 'DOMAIN-SUFFIX,tencent.com,DIRECT'
    - 'DOMAIN-SUFFIX,tenpay.com,DIRECT'
    - 'DOMAIN-SUFFIX,tianyancha.com,DIRECT'
    - 'DOMAIN-SUFFIX,tmall.com,DIRECT'
    - 'DOMAIN-SUFFIX,tudou.com,DIRECT'
    - 'DOMAIN-SUFFIX,umetrip.com,DIRECT'
    - 'DOMAIN-SUFFIX,upaiyun.com,DIRECT'
    - 'DOMAIN-SUFFIX,upyun.com,DIRECT'
    - 'DOMAIN-SUFFIX,veryzhun.com,DIRECT'
    - 'DOMAIN-SUFFIX,weather.com,DIRECT'
    - 'DOMAIN-SUFFIX,weibo.com,DIRECT'
    - 'DOMAIN-SUFFIX,xiami.com,DIRECT'
    - 'DOMAIN-SUFFIX,xiami.net,DIRECT'
    - 'DOMAIN-SUFFIX,xiaomicp.com,DIRECT'
    - 'DOMAIN-SUFFIX,ximalaya.com,DIRECT'
    - 'DOMAIN-SUFFIX,xmcdn.com,DIRECT'
    - 'DOMAIN-SUFFIX,xunlei.com,DIRECT'
    - 'DOMAIN-SUFFIX,yhd.com,DIRECT'
    - 'DOMAIN-SUFFIX,yihaodianimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,yinxiang.com,DIRECT'
    - 'DOMAIN-SUFFIX,ykimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,youdao.com,DIRECT'
    - 'DOMAIN-SUFFIX,youku.com,DIRECT'
    - 'DOMAIN-SUFFIX,zealer.com,DIRECT'
    - 'DOMAIN-SUFFIX,zhihu.com,DIRECT'
    - 'DOMAIN-SUFFIX,zhimg.com,DIRECT'
    - 'DOMAIN-SUFFIX,zimuzu.tv,DIRECT'
    - 'DOMAIN-SUFFIX,zoho.com,DIRECT'
    - 'DOMAIN-KEYWORD,amazon,西红柿🍅'
    - 'DOMAIN-KEYWORD,google,西红柿🍅'
    - 'DOMAIN-KEYWORD,gmail,西红柿🍅'
    - 'DOMAIN-KEYWORD,youtube,西红柿🍅'
    - 'DOMAIN-KEYWORD,facebook,西红柿🍅'
    - 'DOMAIN-SUFFIX,fb.me,西红柿🍅'
    - 'DOMAIN-SUFFIX,fbcdn.net,西红柿🍅'
    - 'DOMAIN-KEYWORD,twitter,西红柿🍅'
    - 'DOMAIN-KEYWORD,instagram,西红柿🍅'
    - 'DOMAIN-KEYWORD,dropbox,西红柿🍅'
    - 'DOMAIN-SUFFIX,twimg.com,西红柿🍅'
    - 'DOMAIN-KEYWORD,blogspot,西红柿🍅'
    - 'DOMAIN-SUFFIX,youtu.be,西红柿🍅'
    - 'DOMAIN-KEYWORD,whatsapp,西红柿🍅'
    - 'DOMAIN-KEYWORD,admarvel,REJECT'
    - 'DOMAIN-KEYWORD,admaster,REJECT'
    - 'DOMAIN-KEYWORD,adsage,REJECT'
    - 'DOMAIN-KEYWORD,adsmogo,REJECT'
    - 'DOMAIN-KEYWORD,adsrvmedia,REJECT'
    - 'DOMAIN-KEYWORD,adwords,REJECT'
    - 'DOMAIN-KEYWORD,adservice,REJECT'
    - 'DOMAIN-SUFFIX,appsflyer.com,REJECT'
    - 'DOMAIN-KEYWORD,domob,REJECT'
    - 'DOMAIN-SUFFIX,doubleclick.net,REJECT'
    - 'DOMAIN-KEYWORD,duomeng,REJECT'
    - 'DOMAIN-KEYWORD,dwtrack,REJECT'
    - 'DOMAIN-KEYWORD,guanggao,REJECT'
    - 'DOMAIN-KEYWORD,lianmeng,REJECT'
    - 'DOMAIN-SUFFIX,mmstat.com,REJECT'
    - 'DOMAIN-KEYWORD,mopub,REJECT'
    - 'DOMAIN-KEYWORD,omgmta,REJECT'
    - 'DOMAIN-KEYWORD,openx,REJECT'
    - 'DOMAIN-KEYWORD,partnerad,REJECT'
    - 'DOMAIN-KEYWORD,pingfore,REJECT'
    - 'DOMAIN-KEYWORD,supersonicads,REJECT'
    - 'DOMAIN-KEYWORD,uedas,REJECT'
    - 'DOMAIN-KEYWORD,umeng,REJECT'
    - 'DOMAIN-KEYWORD,usage,REJECT'
    - 'DOMAIN-SUFFIX,vungle.com,REJECT'
    - 'DOMAIN-KEYWORD,wlmonitor,REJECT'
    - 'DOMAIN-KEYWORD,zjtoolbar,REJECT'
    - 'DOMAIN-SUFFIX,9to5mac.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,abpchina.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,adblockplus.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,adobe.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,akamaized.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,alfredapp.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,amplitude.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ampproject.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,android.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,angularjs.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,aolcdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,apkpure.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,appledaily.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,appshopper.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,appspot.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,arcgis.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,archive.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,armorgames.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,aspnetcdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,att.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,awsstatic.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,azureedge.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,azurewebsites.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,bing.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,bintray.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,bit.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,bit.ly,西红柿🍅'
    - 'DOMAIN-SUFFIX,bitbucket.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,bjango.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,bkrtx.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blog.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blogcdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blogger.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blogsmithmedia.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blogspot.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,blogspot.hk,西红柿🍅'
    - 'DOMAIN-SUFFIX,bloomberg.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,box.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,box.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,cachefly.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,chromium.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,cl.ly,西红柿🍅'
    - 'DOMAIN-SUFFIX,cloudflare.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,cloudfront.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,cloudmagic.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,cmail19.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,cnet.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,cocoapods.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,comodoca.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,crashlytics.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,culturedcode.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,d.pr,西红柿🍅'
    - 'DOMAIN-SUFFIX,danilo.to,西红柿🍅'
    - 'DOMAIN-SUFFIX,dayone.me,西红柿🍅'
    - 'DOMAIN-SUFFIX,db.tt,西红柿🍅'
    - 'DOMAIN-SUFFIX,deskconnect.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,disq.us,西红柿🍅'
    - 'DOMAIN-SUFFIX,disqus.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,disquscdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,dnsimple.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,docker.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,dribbble.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,droplr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,duckduckgo.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,dueapp.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,dytt8.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,edgecastcdn.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,edgekey.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,edgesuite.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,engadget.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,entrust.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,eurekavpt.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,evernote.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,fabric.io,西红柿🍅'
    - 'DOMAIN-SUFFIX,fast.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,fastly.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,fc2.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,feedburner.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,feedly.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,feedsportal.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,fiftythree.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,firebaseio.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,flexibits.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,flickr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,flipboard.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,g.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,gabia.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,geni.us,西红柿🍅'
    - 'DOMAIN-SUFFIX,gfx.ms,西红柿🍅'
    - 'DOMAIN-SUFFIX,ggpht.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ghostnoteapp.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,git.io,西红柿🍅'
    - 'DOMAIN-KEYWORD,github,西红柿🍅'
    - 'DOMAIN-SUFFIX,globalsign.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,gmodules.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,godaddy.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,golang.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,gongm.in,西红柿🍅'
    - 'DOMAIN-SUFFIX,goo.gl,西红柿🍅'
    - 'DOMAIN-SUFFIX,goodreaders.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,goodreads.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,gravatar.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,gstatic.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,gvt0.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,hockeyapp.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,hotmail.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,icons8.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ifixit.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ift.tt,西红柿🍅'
    - 'DOMAIN-SUFFIX,ifttt.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,iherb.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,imageshack.us,西红柿🍅'
    - 'DOMAIN-SUFFIX,img.ly,西红柿🍅'
    - 'DOMAIN-SUFFIX,imgur.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,imore.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,instapaper.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ipn.li,西红柿🍅'
    - 'DOMAIN-SUFFIX,is.gd,西红柿🍅'
    - 'DOMAIN-SUFFIX,issuu.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,itgonglun.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,itun.es,西红柿🍅'
    - 'DOMAIN-SUFFIX,ixquick.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,j.mp,西红柿🍅'
    - 'DOMAIN-SUFFIX,js.revsci.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,jshint.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,jtvnw.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,justgetflux.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,kat.cr,西红柿🍅'
    - 'DOMAIN-SUFFIX,klip.me,西红柿🍅'
    - 'DOMAIN-SUFFIX,libsyn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,linkedin.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,line-apps.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,linode.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,lithium.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,littlehj.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,live.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,live.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,livefilestore.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,llnwd.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,macid.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,macromedia.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,macrumors.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mashable.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mathjax.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,medium.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mega.co.nz,西红柿🍅'
    - 'DOMAIN-SUFFIX,mega.nz,西红柿🍅'
    - 'DOMAIN-SUFFIX,megaupload.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,microsofttranslator.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mindnode.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,mobile01.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,modmyi.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,msedge.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,myfontastic.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,name.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,nextmedia.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,nsstatic.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,nssurge.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,nyt.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,nytimes.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,omnigroup.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,onedrive.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,onenote.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ooyala.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,openvpn.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,openwrt.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,orkut.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,osxdaily.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,outlook.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ow.ly,西红柿🍅'
    - 'DOMAIN-SUFFIX,paddleapi.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,parallels.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,parse.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,pdfexpert.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,periscope.tv,西红柿🍅'
    - 'DOMAIN-SUFFIX,pinboard.in,西红柿🍅'
    - 'DOMAIN-SUFFIX,pinterest.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,pixelmator.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,pixiv.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,playpcesor.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,playstation.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,playstation.com.hk,西红柿🍅'
    - 'DOMAIN-SUFFIX,playstation.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,playstationnetwork.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,pushwoosh.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,rime.im,西红柿🍅'
    - 'DOMAIN-SUFFIX,servebom.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,sfx.ms,西红柿🍅'
    - 'DOMAIN-SUFFIX,shadowsocks.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,sharethis.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,shazam.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,skype.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,smartdns西红柿🍅.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,smartmailcloud.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,sndcdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,sony.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,soundcloud.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,sourceforge.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,spotify.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,squarespace.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,sstatic.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,st.luluku.pw,西红柿🍅'
    - 'DOMAIN-SUFFIX,stackoverflow.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,startpage.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,staticflickr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,steamcommunity.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,symauth.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,symcb.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,symcd.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tapbots.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tapbots.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,tdesktop.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,techcrunch.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,techsmith.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,thepiratebay.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,theverge.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,time.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,timeinc.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,tiny.cc,西红柿🍅'
    - 'DOMAIN-SUFFIX,tinypic.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tmblr.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,todoist.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,trello.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,trustasiassl.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tumblr.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,tumblr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tweetdeck.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,tweetmarker.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,twitch.tv,西红柿🍅'
    - 'DOMAIN-SUFFIX,txmblr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,typekit.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,ubertags.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ublock.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,ubnt.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ulyssesapp.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,urchin.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,usertrust.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,v.gd,西红柿🍅'
    - 'DOMAIN-SUFFIX,v2ex.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,vimeo.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,vimeocdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,vine.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,vivaldi.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,vox-cdn.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,vsco.co,西红柿🍅'
    - 'DOMAIN-SUFFIX,vultr.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,w.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,w3schools.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,webtype.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wikiwand.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wikileaks.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,wikimedia.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,wikipedia.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wikipedia.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,windows.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,windows.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,wire.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wordpress.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,workflowy.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wp.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wsj.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,wsj.net,西红柿🍅'
    - 'DOMAIN-SUFFIX,xda-developers.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,xeeno.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,xiti.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,yahoo.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,yimg.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,ying.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,yoyo.org,西红柿🍅'
    - 'DOMAIN-SUFFIX,ytimg.com,西红柿🍅'
    - 'DOMAIN-SUFFIX,telegra.ph,西红柿🍅'
    - 'DOMAIN-SUFFIX,telegram.org,西红柿🍅'
    - 'IP-CIDR,91.108.4.0/22,西红柿🍅,no-resolve'
    - 'IP-CIDR,91.108.8.0/21,西红柿🍅,no-resolve'
    - 'IP-CIDR,91.108.16.0/22,西红柿🍅,no-resolve'
    - 'IP-CIDR,91.108.56.0/22,西红柿🍅,no-resolve'
    - 'IP-CIDR,149.154.160.0/20,西红柿🍅,no-resolve'
    - 'IP-CIDR6,2001:67c:4e8::/48,西红柿🍅,no-resolve'
    - 'IP-CIDR6,2001:b28:f23d::/48,西红柿🍅,no-resolve'
    - 'IP-CIDR6,2001:b28:f23f::/48,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.232.181.162/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.241.147.226/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.253.253.226/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.253.255.162/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.253.255.34/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,120.253.255.98/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,180.163.150.162/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,180.163.150.34/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,180.163.151.162/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,180.163.151.34/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,203.208.39.0/24,西红柿🍅,no-resolve'
    - 'IP-CIDR,203.208.40.0/24,西红柿🍅,no-resolve'
    - 'IP-CIDR,203.208.41.0/24,西红柿🍅,no-resolve'
    - 'IP-CIDR,203.208.43.0/24,西红柿🍅,no-resolve'
    - 'IP-CIDR,203.208.50.0/24,西红柿🍅,no-resolve'
    - 'IP-CIDR,220.181.174.162/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,220.181.174.226/32,西红柿🍅,no-resolve'
    - 'IP-CIDR,220.181.174.34/32,西红柿🍅,no-resolve'
    - 'DOMAIN,injections.adguard.org,DIRECT'
    - 'DOMAIN,local.adguard.org,DIRECT'
    - 'DOMAIN-SUFFIX,local,DIRECT'
    - 'IP-CIDR,127.0.0.0/8,DIRECT'
    - 'IP-CIDR,172.16.0.0/12,DIRECT'
    - 'IP-CIDR,192.168.0.0/16,DIRECT'
    - 'IP-CIDR,10.0.0.0/8,DIRECT'
    - 'IP-CIDR,17.0.0.0/8,DIRECT'
    - 'IP-CIDR,100.64.0.0/10,DIRECT'
    - 'IP-CIDR,224.0.0.0/4,DIRECT'
    - 'IP-CIDR6,fe80::/10,DIRECT'
    - 'DOMAIN-SUFFIX,cn,DIRECT'
    - 'DOMAIN-KEYWORD,-cn,DIRECT'
    - 'GEOIP,CN,DIRECT'
    - 'MATCH,西红柿🍅'
```