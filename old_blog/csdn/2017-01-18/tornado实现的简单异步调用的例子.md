tornado是python实现的一个异步web框架
除了写web服务实现web api供人调用之外，自己想写一个简单的http访问，于是有了下面的代码，尽供自己记录用，其实也可以用来平时写tornado代码简单进行功能调试使用参考
test.py
 <!--more-->  
```
#coding=utf-8
import tornado.ioloop
from tornado import gen
from tornado.httpclient import AsyncHTTPClient
from urllib import urlencode
from tornado.netutil import Resolver

AsyncHTTPClient.configure("tornado.curl_httpclient.CurlAsyncHTTPClient")
Resolver.configure('tornado.platform.caresresolver.CaresResolver')
PHONE_NUMS = [
'13000000000',
'13000000001',
'13000000002',
'13000000003'
]
uri = 'http://testurl/send'
@gen.coroutine
def test_send():
    http_client = AsyncHTTPClient(max_clients=32)
    req = []
    for phone in PHONE_NUMS:
        _mesg = '验证码：%s' %phone[-6:]
        _query = dict(phone=phone, mesg=_mesg)
        query = urlencode(_query)
        _req = http_client.fetch(uri, method='POST',
                                 body=query)
        req.append(_req)
    resp = yield req
    print "resp:", resp


def test_run():
    #tornado.ioloop.PeriodicCallback(sms_send,60000).start()
    #tornado.ioloop.IOLoop.instance().start()
    ioloop = tornado.ioloop.IOLoop.current()
    ioloop.run_sync(test_send) 

if __name__ == "__main__":
    test_run()
```
本文中的例子在使用httpclient时候，使用了curl_httpclient这个实现，相比与tornado默认的simple_httpclient，curl_httpclient效率更高，并且还有其他的一些好处，可以参见tornado官方的文档介绍使用。
但是在使用curl_httpclient时，需要注意要配合使用异步的dns
上面的代码使用的一个异步的dns解析，在使用时需要另外安装pycares模块
```
pip install pycares
```
