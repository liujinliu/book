celery是实现一个简单，灵活可靠的分布式任务队列系统的好选择
tornado则不用过多介绍
在开发机上安装rabbitmq这里就不介绍了
首先是task文件的编写
task.py
```
#coding=utf-8
from celery import Celery
from celery.bin import worker as celery_worker
import celeryconfig

broker = 'amqp://'
backend = 'amqp'
app = Celery('celery_test', backend=backend, broker=broker)
app.config_from_object(celeryconfig)

@app.task
def mytask0(task_name):
    print "task0:%s" %task_name
    return task_name 

@app.task
def mytask1(task_name):
    print "task1:%s" %task_name
    return task_name 

def worker_start():
    worker = celery_worker.worker(app=app)
    worker.run(broker=broker, concurrency=4,
               traceback=False, loglevel='INFO')

if __name__ == "__main__":
    worker_start()
```
celeryconfig.py文件中包含对celery的配置
```
#coding=utf-8
from kombu import Queue
CELERY_DEFAULT_QUEUE = 'mytask0'
CELERY_QUEUES = (
    Queue('mytask0',    routing_key='task.mytask0'),
    Queue('mytask1',    routing_key='task.mytask1'),
)
CELERY_DEFAULT_EXCHANGE_TYPE = 'direct'
CELERY_DEFAULT_ROUTING_KEY = 'task.mytask0'
CELERY_TIMEZONE = 'Asia/Shanghai'
CELERY_ROUTES = {
    'task.mytask0': {
        'queue': 'mytask0',
        'routing_key': 'task.mytask0',
    },
    'task.mytask1': {
        'queue': 'mytask1',
        'routing_key': 'task.mytask1',
    },
}
```
执行python task.py将会启动worker
### tornado调用celery将阻塞任务变为非阻塞
这会使用到tcelery模块，即tornado下的一个非阻塞的broker实现
app.py
```
#coding=utf-8
from tornado import web
import task

class TestHandler(tornado.web.RequestHandler):

    @web.asynchronous
    def get(self):
        task.mytask0.apply_async(
            args=['task0'],
                  queue='mytask0',
                  routing_key='task.mytask0',
                  callback=self.on_success)
    def on_success(self, result):
        self.finish({'task':result.result})
```
start.py
用于实现tornado服务的启动
```
#coding=utf-8
import tornado
from tornado.options import define, options, parse_command_line
from tornado.log import enable_pretty_logging
import tcelery
from app import TestHandler
import tornado.httpserver

define("port", default=8000, help="run on the given port", type=int)
define("debug", default=False, help="run in debug mode")

urls = [(r"/api/task/test", TestHandler)]

def server_start():
    app = tornado.web.Application(urls, debug=options.debug)
    enable_pretty_logging()
    parse_command_line()
    server = tornado.httpserver.HTTPServer(app)
    server.bind(options.port)
    server.start(2)
    tcelery.setup_nonblocking_producer(limit=2)
    tornado.ioloop.IOLoop.current().start()

if __name__ == "__main__":
    server_start()
```
执行python start.py即可启动服务