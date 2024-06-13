# python

## 爬虫

1. selenium  测试自动库
2. beautifulsoup html解析库
3. request 下载  urlib
4. you-get  下载视频
5. multiprocessing  多线程  ThreadPool
6. Python-tesseract  ocr
7. pymysql  mysqlclient 

```python
# -*- coding: utf-8 -*-
import urllib
from bs4 import BeautifulSoup

rawurl='https://www.esrl.noaa.gov/psd/cgi-bin/db_search/DBListFiles.pl?did=118&tid=40290&vid=2227'
content = urllib.request.urlopen(rawurl).read().decode('ascii')  # 获取页面的HTML

soup = BeautifulSoup(content, 'lxml')
url_cand_html=soup.find_all(id='content') # 定位到存放url的标号为content的div标签
list_urls=url_cand_html[0].find_all("a") # 定位到a标签，其中存放着文件的url
urls=[]

for i in list_urls[1:]:
    urls.append(i.get('href')) # 取出链接

for i,url in enumerate(urls):
    print("This is file"+str(i+1)+" downloading! You still have "+str(142-i-1)+" files waiting for downloading!!")
    file_name = "./ncfile/"+url.split('/')[-1] # 文件保存位置+文件名
    urllib.request.urlretrieve(url, file_name)
```

### 分块下载

```python
import requests
url = 'https://buildmedia.readthedocs.org/media/pdf/python-guide/latest/python-guide.pdf'
r = requests.get(url, stream = True)
with open("PythonBook.pdf", "wb") as Pypdf:
    for chunk in r.iter_content(chunk_size = 1024): # 1024 bytes
        if chunk:
            Pypdf.write(chunk)
```

### 并行下载

```python
import os
import requests
from time import time
from multiprocessing.pool import ThreadPool

def url_response(url):
    path, url = url
    r = requests.get(url, stream=True)
    with open(path, 'wb') as f:
        for ch in r:
            f.write(ch)

urls = [("Event1", "https://www.python.org/events/python-events/805/"),
        ("Event2", "https://www.python.org/events/python-events/801/"),
        ("Event3", "https://www.python.org/events/python-events/790/"),
        ("Event4", "https://www.python.org/events/python-events/798/"),
        ("Event5", "https://www.python.org/events/python-events/807/"),
        ("Event6", "https://www.python.org/events/python-events/807/"),
        ("Event7", "https://www.python.org/events/python-events/757/"),
        ("Event8", "https://www.python.org/events/python-user-group/816/")]

start = time()
ThreadPool(9).imap_unordered(url_response, urls)
print(f"Time to download: {time() - start}")

# Time to download: 0.0064961910247802734`

# 多线程
def main(url):
    html = request_douban(url)
    soup = BeautifulSoup(html, 'lxml')
    save_content(soup)

if __name__ == '__main__':
    start = time.time()
    urls = []
    pool = multiprocessing.Pool(multiprocessing.cpu_count())
    for i in range(0, 10):
        url = 'https://movie.douban.com/top250?start=' + str(i * 25) + '&filter='
        urls.append(url)
    pool.map(main, urls)
    pool.close()
    pool.join()
```

### ip代理库

[python](https://github.com/Python3WebSpider/ProxyPool )

### 免登录

1. Cookie
   
   ```python
   import requests
   ```

headers = {
    # 假装自己是浏览器    'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Ubuntu Chromium/73.0.3683.75 Chrome/73.0.3683.75 Safari/537.36',
    # 把你刚刚拿到的Cookie塞进来
    'Cookie': 'eda38d470a662ef3606390ac3b84b86f9; Hm_lvt_f1d3b035c559e31c390733e79e080736=1553503899; biihu__user_login=omvZVatKKSlcXbJGmXXew9BmqediJ4lzNoYGzLQjTR%2Fjw1wOz3o4lIacanmcNncX1PsRne5tXpE9r1sqrkdhAYQrugGVfaBICYp8BAQ7yBKnMpAwicq7pZgQ2pg38ZzFyEZVUvOvFHYj3cChZFEWqQ%3D%3D; Hm_lpvt_f1d3b035c559e31c390733e79e080736=1553505597',
}

session = requests.Session()
response = session.get('https://biihu.cc/people/wistbean%E7%9C%9F%E7%89%B9%E4%B9%88%E5%B8%85', headers=headers)

print(response.text)

```
2. selenum

```python
username = WAIT.until(EC.presence_of_element_located((By.CSS_SELECTOR, "帐号的selector")))
password = WAIT.until(EC.presence_of_element_located((By.CSS_SELECTOR, "密码的selector")))
submit = WAIT.until(EC.element_to_be_clickable((By.XPATH, '按钮的xpath')))

username.send_keys('你的帐号')
password.send_keys('你的密码')submit.click()

cookies = webdriver.get_cookies()
```

#### 验证码

1. Python-tesseract ocr

```python
captcha = Image.open("captcha1.png")
result = pytesseract.image_to_string(captcha)
print(result)


# 灰度处理 二值化  降噪处理
def convert_img(img,threshold):
    img = img.convert("L")  # 处理灰度
    pixels = img.load()
    for x in range(img.width):
        for y in range(img.height):
            if pixels[x, y] > threshold:
                pixels[x, y] = 255
            else:
                pixels[x, y] = 0
    return img
```

#### Appium 爬取你的微信朋友圈

### matplotlib

数据分析 pyecharts

```python
再来画个词云图

words = [
    ("Sam S Club", 10000),
    ("Macys", 6181),
    ("Amy Schumer", 4386),
    ("Jurassic World", 4055),
    ("Charter Communications", 2467),
    ("Chick Fil A", 2244),
    ("Planet Fitness", 1868),
    ("Pitch Perfect", 1484),
    ("Express", 1112),
    ("Home", 865),
    ("Johnny Depp", 847),
    ("Lena Dunham", 582),
    ("Lewis Hamilton", 555),
    ("KXAN", 550),
    ("Mary Ellen Mark", 462),
    ("Farrah Abraham", 366),
    ("Rita Ora", 360),
    ("Serena Williams", 282),
    ("NCAA baseball tournament", 273),
    ("Point Break", 265),
]


def wordcloud_base() -> WordCloud:
    c = (
        WordCloud()
        .add("", words, word_size_range=[20, 100])
        .set_global_opts(title_opts=opts.TitleOpts(title="WordCloud-基本示例"))
    )
    return c

# 需要安装 snapshot_selenium
make_snapshot(driver, wordcloud_base().render(), "WordCloud.png")
```

### 爬虫框架

scrapy

```python
 scrapy startproject qiushibaike
```

## 反爬

### css 加密

1. 存储字典编码

### js 加密

生成sign

```python
n.md5("fanyideskweb" + e + i + "@6f#X3=cCuncYssPsuRUE")
```