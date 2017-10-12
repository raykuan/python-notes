## python3内置的urllib有五个的部分,常用的主要是parse、request、error：
```
urllib.parse
urllib.request
urllib.resopnse
urllib.error
urllib.robotparser
```

## urllib.parse
#### urllib.parse.urlparse
```
>>> url = r'https://docs.python.org/3.5/search.html?q=parse&check_keywords=yes&area=default'
>>> parseResult = urllib.parse.urlparse(url)
>>> parseResult
ParseResult(scheme='https', netloc='docs.python.org', path='/3.5/search.html', params='', query='q=parse&check_keywords=yes&area=default', fragment='')
```

#### urllib.parse.parse_qs
```
>>> param_dict = urllib.parse.parse_qs(parseResult.query)
>>> param_dict
{'q': ['parse'], 'check_keywords': ['yes'], 'area': ['default']}
>>> q = param_dict['q'][0]
>>> q
'parse'
#注意：加号会被解码，可能有时并不是我们想要的
>>> urllib.parse.parse_qs('proxy=183.222.102.178:8080&task=XXXXX|5-3+2')
{'proxy': ['183.222.102.178:8080'], 'task': ['XXXXX|5-3 2']}
```

#### urllib.parse.urlencode
```
>>> query = {
  'name': 'walker',
  'age': 99,
  'email': rayk@qq.com,
  }
>>> urllib.parse.urlencode(query)
'name=walker&age=99&email=rayk@qq.com'
```

#### urllib.parse.quote/quote_plus
```
>>> urllib.parse.quote('a&b/c')  #不编码斜线
'a%26b/c'
>>> urllib.parse.quote_plus('a&b/c')  #编码斜线
'a%26b%2Fc'
```

#### urllib.parse.unquote/unquote_plus
```
>>> urllib.parse.unquote('1+2')  #不解码加号
'1+2'
>>> urllib.parse.unquote_plus('1+2')  #把加号解码为空格
'1 2'
```

## urllib.request
#### urllib.request.urlopne
```
urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)
url：需要打开的网址
data：Post提交的数据
timeout：设置网站的访问超时时间

>>> response = urllib.request.urlopen(r'http://python.org/')  #<http.client.HTTPResponse object at 0x048BC908> HTTPResponse类型
>>> page = response.read()  #page的数据格式为bytes类型，需要decode()解码，转换成str类型
>>> page = page.decode('utf-8')

urlopen返回的是一个对象实例，有如下方法：
read(), readline(), readlines(), fileno(), close()  #对HTTPResponse类型数据进行操作
info()：返回HTTPMessage对象，表示远程服务器返回的头信息
getcode()：返回Http状态码，如果是http请求是200、、400、404等
geturl()：返回请求的url

当data参数不为空的时候，urlopen()提交方式为Post
>>> data = {
  'name': 'walker',
  'age': 99,
  'email': rayk@qq.com,
  }
>>> data = urllib.parse.urlencode(data).encode('utf-8')  #先对data做urlencode操作
>>> req = urllib.request.Request(url, headers=headers, data=data)
>>> page = urllib.request.urlopen(req).read()
>>> page = page.decode('utf-8')
```
#### urllib.request.Request
```
urllib.request.Request(url, data=None, headers={}, method=None)
先用request()来包装请求，再通过urlopen()获取页面

>>> url = r'http://www.python.org/Python/?labelWords=label'
>>> headers = {
    'User-Agent': r'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) 
    'Referer': r'http://www.lagou.com/zhaopin/Python/?labelWords=label',
    'Connection': 'keep-alive'
    'Authorization': 'Token 473cfacf14f1e6aace79cfd0339d45cff44b5bb2'
}
>>> req = urllib.request.Request(url, headers=headers)
>>> page = urllib.request.urlopen(req).read()
>>> page = page.decode('utf-8')

User-Agent ：这个头部可以携带如下几条信息：浏览器名和版本号、操作系统名和版本号、默认语言
Referer：可以用来防止盗链，有一些网站图片显示来源http://***.com，就是检查Referer来鉴定的
Connection：表示连接状态，记录Session的状态。
Authorization：表示授权信息，通常出现在对服务器发送的WWW-Authenticate头的应答中
```

## urllib.error
```
import urllib.parse
import urllib.request
import urllib.error

def get_page(url):
    headers = {
        'User-Agent': r'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36',
        'Referer': r'http://www.python.org/Python/?labelWords=label',
        'Connection': 'keep-alive'
    }

    data = {
        'timestamp': str(time.time()).split('.')[0],
        'device_sn': 'rrkksn-123',
        'device_type': 'Mozila 6.5',
        'random_num': 'random',
    }

    data = urllib.parse.urlencode(data).encode('utf-8')
    req = urllib.request.Request(url, headers=headers)
    try:
        page = urllib.request.urlopen(req, data=data).read()
        page = page.decode('utf-8')
    except urllib.error.HTTPError as e:
        print(e.code())
        print(e.read().decode('utf-8'))
    return page
```

