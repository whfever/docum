# Edge
## vim
### gh
```
map go createTab url="http://www.google.com"
map gls createTab url="http://www.lsdhss.com/"
map gc createTab url="http://www.chinadaily.com.cn/"
map jc createTab url="https://chat.openai.com/"
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
    - { name: '香港 IEPL 04', type: ss, server: je0fqks9w0ylugbjiepl.gamesub.cc, port: 35001, cipher: '
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

# Edge Console

## 遍历 LI 

```js
function me() {
  for (var i = 0; i< li.length; i++) {
    console.log(li.item(i));
    setTimeout(me,1000);
    li[i].setAttribute("class","icon icon-follow-watched icon-follow-selected");
   console.log(li.item(i));
}
}
```