http://stackoverflow.com/questions/13292407/how-to-improve-the-performance-of-the-combination-of-gevent-and-tornado?rq=1  
在stackoverflow上看到有人给出了将tornado和gevent结合的编码例子，记在这里，便于以后查阅。细节待有时间在补充吧
这是第一段
 <!--more-->  
```
# Gevent monkeypath
from gevent import monkey
monkey.patch_all()

# Gevent imports
import gevent

# Python immports
import functools

# Tornado imports
import tornado.ioloop
import tornado.web
import tornado.httpserver

# Request imports
import requests


# Asynchronous gevent decorator
def gasync(func):
    @tornado.web.asynchronous
    @functools.wraps(func)
    def f(self, *args, **kwargs):
        #self._auto_finish = False
        return gevent.spawn(func, self, *args, **kwargs)
    return f


# Constants
URL_TO_FETCH = 'http://google.co.uk/'

# Global
I = 0


class MainHandler(tornado.web.RequestHandler):
    @gasync
    def get(self):
        global I
        r = requests.get(URL_TO_FETCH)
        I += 1
        print('Got page %d (length=%d)' % (I, len(r.content)))
        self.write("Done")
        self.finish()


# Our URL Mappings
handlers = [
   (r"/", MainHandler),
]


def main():
    # Setup app and HTTP server
    application = tornado.web.Application(handlers)
    http_server = tornado.httpserver.HTTPServer(application)
    http_server.listen(9998)

    # Start ioloop
    tornado.ioloop.IOLoop.instance().start()


if __name__ == "__main__":
    main()
```
这是第二段，可以采用coroutine的方式调用
```
# encoding: utf-8

from gevent import monkey
monkey.patch_all()

# Gevent imports
import gevent

# Python immports
import functools

# Tornado imports
import tornado.ioloop
import tornado.web
import tornado.gen
import tornado.httpserver

# Request imports
import requests
from tornado.concurrent import Future


# Asynchronous gevent decorator
def gfuture(func):
    @functools.wraps(func)
    def f(*args, **kwargs):
        loop = tornado.ioloop.IOLoop.current()
        future = Future()

        def call_method():
            try:
                result = func(*args, **kwargs)
                loop.add_callback(functools.partial(future.set_result, result))
            except Exception, e:
                loop.add_callback(functools.partial(future.set_exception, e))
        gevent.spawn(call_method)
        return future
    return f


# Constants
URL_TO_FETCH = 'http://google.com/'

# Global
I = 0


@gfuture
def gfetch(url, i):
    r = requests.get(url)
    return i


class MainHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    @tornado.gen.coroutine
    def get(self):
        global I
        I += 1
        n = I
        print "=> %s" % n
        n = yield gfetch(URL_TO_FETCH, n)
        print "<= %s" % n
        self.write("Done %s" % n)


# Our URL Mappings
handlers = [(r"/", MainHandler)]


def main():
    # Setup app and HTTP server
    application = tornado.web.Application(handlers)
    http_server = tornado.httpserver.HTTPServer(application)
    http_server.listen(9998)

    # Start ioloop
    tornado.ioloop.IOLoop.instance().start()


if __name__ == "__main__":
    main()
```
另外网上有两个比较好的项目可以供参考：
https://github.com/liujinliu/tornalet/blob/master/tornalet.py(这份是我fork别人的一份工程)  
https://github.com/mongodb/motor
