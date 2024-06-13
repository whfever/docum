## 

# JS demo

## puppteer

![img](https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/12/31/16f59e4f188d10c3~tplv-t2oaga2asx-zoom-in-crop-mark:4536:0:0:0.image)

```js
const puppeteer = require("puppeteer");
const url = "https://www.v2ex.com/?tab=hot";

(async function run() {
    let browser = await puppeteer.launch({
        executablePath: "./chrome-mac/Chromium.app/Contents/MacOS/Chromium",
        headless: true
    });//使用设置参数运行一个浏览器
    let newPage = await browser.newPage();// 浏览器创建一个新tab页
    newPage.setDefaultTimeout(1000 * 10);// 设置默认请求超时时间
    await Promise.all([
        newPage.setJavaScriptEnabled(false)// 设置禁止js，这样会快一些
    ]);
    await newPage.goto(url, {waitUntil: "domcontentloaded"});//请求页面
    let list = await newPage.?eval(`.item_title a`, eles => eles.map(ele => {
        return {title: ele.innerText, url: ele.href}
    }));// 获取Dom中我们想要的数据
    if (list) {
        list.forEach((item) => {
            console.log(item)// 打印结果
        })
    }
    browser.close();// 关闭浏览器
})();
```

> 当然如果想做一个成品的聚合站，还有很多工作要做，比如**定时爬取、守护进程、缓存和数据库管理、写Server、写前端**等等。篇幅所限，写到这里我们只把爬虫最核心的地方写完了，剩下的相信聪明的你会有很多花(姿)样(势)要玩：）