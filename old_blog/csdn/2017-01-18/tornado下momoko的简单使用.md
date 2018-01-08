最近在项目中实现一个server的时候采用了tornado，需要连接pg数据库，于是接触并使用了momoko，在这里做一些简单记录
model.py
```
#coding=utf-8
from tornado import gen
from tornado.gen import Return
import momoko
class TestMomoko(object):
    def __init__(self):
        self.table = 'testmomoko'
        self.cols = ['id', 'message']
    
    def connect(self, ioloop, host='localhost', 
                dbname='mytest',
                user='test', port=5494, 
                passwd='test'):
        dsn = (('dbname=%s user=%s password=%s '
               'host=%s port=%d') %(dbname, user, passwd, 
               host, port))
        self.db = momoko.Pool(dsn=dsn, size=5, ioloop=ioloop)
        future = self.db.connect()
        ioloop.add_future(future, lambda f: ioloop.stop())
        ioloop.start()
        future.result()

    @gen.coroutine
    def test_insert(self):
        sql = (("INSERT INTO %s (message) "
                "VALUES ('%s')") %(self.table, 'test message'))
        yield self.db.execute(sql)

    @gen.coroutine
    def test_get(self, offset, limit):
        sql = (("SELECT message FROM %s LIMIT %d OFFSET %d") 
               %(self.table, limit, offset))
        cursor = yield self.db.execute(sql) 
        raise Return(cursor.fetchall())

TEST_PG = TestMomoko()

@gen.coroutine
def test_insert():
    yield TEST_PG.test_insert()
```
tornado启动文件
start.py
```
#coding=utf-8
from tornado.options import define, options, parse_command_line
import tornado.ioloop
from tornado.log import enable_pretty_logging
from model import TEST_PG

define("pg_host", default="localhost", help="postgresql host name")
define("pg_port", default=5494, help="postgresql port")
define("pg_db", default="test", help="postgresql database name")
define("pg_user", default="test", help="postgresql user name")
define("pg_pass", default="test", help="postgresql user password")

def server_start():
    enable_pretty_logging()
    parse_command_line()
    io_loop = tornado.ioloop.IOLoop.instance()
    TEST_PG.connect(io_loop, options.pg_host,
                    options.pg_db, options.pg_user,
                    options.pg_port, options.pg_pass)
    io_loop.start()
```
下面这个app.py实现一个简单的http接口来调用momoko进行pg数据库的操作
```
class TestPushWorkHandler(tornado.web.RequestHandler):

    @gen.coroutine
    def get(self):
        yield test_insert()
```