# Edge

## vim

```
## gh
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
#h
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
#g
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

 Console

## console

- 遍历li
  
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