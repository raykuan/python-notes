### python3内置的urllib有五个的部分：
urllib.parse <br>
urllib.request <br>
urllib.resopnse <br>
urllib.error <br>
urllib.robotparser <br>

### 常用的主要是parse、request、error

urllib.parse
```
1. urllib.parse.urlparse
>>> url = r'https://docs.python.org/3.5/search.html?q=parse&check_keywords=yes&area=default'
>>> parseResult = urllib.parse.urlparse(url)
>>> parseResult
ParseResult(scheme='https', netloc='docs.python.org', path='/3.5/search.html', params='', query='q=parse&check_keywords=yes&area=default', fragment='')

2. urllib.parse.parse_qs
>>> param_dict = urllib.parse.parse_qs(parseResult.query)
>>> param_dict
{'q': ['parse'], 'check_keywords': ['yes'], 'area': ['default']}
>>> q = param_dict['q'][0]
>>> q
'parse'
#注意：加号会被解码，可能有时并不是我们想要的
>>> urllib.parse.parse_qs('proxy=183.222.102.178:8080&task=XXXXX|5-3+2')
{'proxy': ['183.222.102.178:8080'], 'task': ['XXXXX|5-3 2']}

3. urllib.parse.urlencode
>>> query = {
  'name': 'walker',
  'age': 99,
  'email': rayk@qq.com,
  }
>>> urllib.parse.urlencode(query)
'name=walker&age=99&email=rayk@qq.com'

4. urllib.parse.quote/quote_plus
>>> urllib.parse.quote('a&b/c')  #不编码斜线
'a%26b/c'
>>> urllib.parse.quote_plus('a&b/c')  #编码斜线
'a%26b%2Fc'

5. urllib.parse.unquote/unquote_plus
>>> urllib.parse.unquote('1+2')  #不解码加号
'1+2'
>>> urllib.parse.unquote_plus('1+2')  #把加号解码为空格
'1 2'
```

urllib.request
```
1. urllib.request.Request


2. urllib.request.urlopne

