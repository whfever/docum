# python
## 爬虫
1. selenium  测试自动库
2. beautifulsoup html解析库
3. request 下载  urlib
4. you-get  下载视频
5. multiprocessing  多线程  ThreadPool

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

## ip代理库
[python](https://github.com/Python3WebSpider/ProxyPool )