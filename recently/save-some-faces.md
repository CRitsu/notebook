# 收集一些表情

这是一个插曲。

之前写教程时用到了一些表情图片来增加内容的趣味性。为此我还特地百度了一些表情图片，但是结果不是很满意。

一张一张搜集效率太差，而且我没有那么多时间花在表情包上。

这时我找到了一个有趣的表情包网站，暴走漫画的表情包。

打开浏览器开发者工具看了看网页的结构，嗯，很简单，那么就用Python爬一下吧！

页面结构简单，使用requests和BeautifulSoup很轻松的就拉取了这个网站的所有表情（程序运行完有954个文件）。

使用时需要注意下文件保存的位置。

33行这一块，将`open`后面的`../faces-tar/`换成想存放表情的文件夹就可以了。

```python
def write(url, name):
    r = requests.get(url, user_agent)
    with open('../faces-tar/%s' % name, 'wb') as f:
        f.write(r.content)
```

[完整代码](https://github.com/CRitsu/python.test/blob/master/faces/baozou_faces_crawler.py)


**后续 18.3.27**

抽时间整理了一下表情文件，发现重复图片和低像素图片太多，质量不是很高。不过也够用了。
