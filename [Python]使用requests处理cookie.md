
![requests](https://camo.githubusercontent.com/5e4574f4d470db274e80e7cb1464e426e643e084/687474703a2f2f646f63732e707974686f6e2d72657175657374732e6f72672f656e2f6d61737465722f5f7374617469632f72657175657374732d736964656261722e706e67)

## 0x00 需求

常见的 `application/json` 请求，如果token进行验证，我们可以在header或者body中直接添加，对于使用cookie进行验证的请求，虽然可以自己维护cookie，但是会比token麻烦很多。

之前的忘了请求都是使用python3的urllib进行，当处理cookie时，发现比较困难，因此着手另寻他法，这样就发现了requests。简单看了下，使用起来比urllib方便了好多，好多。不紧紧实现了手动添加cookie的需求，其他接口使用也是相当友好。有时间打算吧之前的代码重构一下。

下面简单介绍下requests的使用。


## 0x01 requests

### Request

- GET

    - 无参数
    
        ```    
>>> r = requests.get('https://api.github.com/events')`
        ```
                
    - url参数
    
        ```
>>> payload = {'key1': 'value1', 'key2': 'value2'}
>>> r = requests.get('http://httpbin.org/get', params=payload)
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
        ```
        
        输出：
        
        ```
>>> print(r.url)
http://httpbin.org/get?key2=value2&key1=value1
        ```
    
- POST

    
    - form参数
    
        ```
    r = requests.post('http://httpbin.org/post', data = {'key':'value'})
        ```
    
    - json参数

       ```
>>> import json
>>> url = 'https://api.github.com/some/endpoint'
>>> payload = {'some': 'data'}
>>> r = requests.post(url, data=json.dumps(payload))
       ``` 
    
    - file
    
        `TODO`

- Other: 

    - PUT: `r = requests.put('http://httpbin.org/put', data = {'key':'value'})`
    - DELETE: `r = requests.delete('http://httpbin.org/delete')`
    - HEAD: `r = requests.head('http://httpbin.org/get')`
    - OPTIONS: `r = requests.options('http://httpbin.org/get')`
       
- HEADER

	``` shell
>>> url = 'https://api.github.com/some/endpoint'
>>> headers = {'user-agent': 'my-app/0.0.1'}
>>> r = requests.get(url, headers=headers)
	```
     
- COOKIES
    
    请求：
    
    ``` shell
>>> url = 'http://httpbin.org/cookies'
>>> cookies = dict(cookies_are='working')
>>> r = requests.get(url, cookies=cookies)
    ```
    
   请求的数据：
    
   ``` shell
GET /cookies HTTP/1.1
Host: httpbin.org
Connection: keep-alive
User-Agent: python-requests/2.12.4
Accept-Encoding: gzip, deflate
Accept: */*
Cookie: cookies_are=working
    ```
    
### Response

```
>>> import requests
>>> response = requests.get('https://api.github.com/events')
```

- response.url 
- response.encoding 编码
- response.status_code 状态吗
- response.headers headers
- response.text 返回的文本
- response.json() json
- response.raw 原始数据
- response.raise_for_status() 错误请求信息
- response.cookies
- response.history


## 0x03 总结

使用过urllib的话，来看requests你会发现简单好多，同时支持python2 和 python3。


## 0xFF 参考

- https://pypi.python.org/pypi/requests
- http://docs.python-requests.org/en/master/
- https://github.com/kennethreitz/requests

![Create by ttdevs](https://raw.githubusercontent.com/ttdevs/ttdevs.github.io/common/images/logo.png)


