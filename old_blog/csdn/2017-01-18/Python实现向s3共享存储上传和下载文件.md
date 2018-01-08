主要是使用boto模块 
<!--more-->  
#### 建立连接
```
import boto
import boto.s3.connection
from boto.s3.key import Key

conn = boto.connect_s3(
            aws_access_key_id = 'xxxxxxx',
            aws_secret_access_key = 'xxxxxxxx',
            host = 's3.xxxxx.com', is_secure = False,
            calling_format = boto.s3.connection.OrdinaryCallingFormat())
```
#### 创建/获取bucket
```
bucket = conn.create_bucket('test')  ----创建一个新的bucket
# bucket = conn.get_bucket('test')   ----获取一个已经存在的bucket
k = Key(bucket)
k.key = 'test_file'
```
#### 将文件内容上传到共享存储
```
filename = 'testfile'
k.set_contents_from_filename(filename)
```
#### 将共享存储的内容存到文件
```
filename2 = 'testfile2'
k.get_contents_to_filename(filename2)
```
#### 删除key/bucket
```
bucket.delete_key('test_file')
conn.delete_bucket(bucket.name)
```
上面介绍的是使用boto2模块，但现在官方推荐使用boto3，boto3提供了更高层的抽象(resource)。这使得很多操作变的更便捷，可以参考[官方文档](https://boto3.readthedocs.io/en/latest/guide/migrations3.html)的介绍，对于一些可能遇到的坑这里提下
## boto3使用中可能遇到问题
### 1. 使用resource层级的接口，如何上传一段内存内容
这个目前我还没找到方法，官方意思是提供一个filelike-obj的就可以，但是需要这个filelike-obj提供read方法，貌似文件的read方法没那么好模拟，我简单写了一个类，return一个bytes，结果在使用中，调用的时候直接死循环了。所以这里还是使用boto3提供的client这一层的接口吧。
```
class PersisDb(object):
    def __init__(self, *args, **kargs):
        self.s3 = boto3.resource('s3', *args, **kargs)
        self.bkt = kargs.get('s3_bucket', 'test')
        self.client = boto3.client('s3', *args, **kargs)

    def insert(self, user_id, date, doc):
        bkt = self.bkt
        key = '{user_id}/{date}'.format(user_id=user_id, date=date)
        LOG.debug('writing doc to %s:%s' % (bkt, key))
        self.client.put_object(Bucket=bkt, Key=key, Body=doc)
```
### 2. 使用resource层级接口，如何下载一个文件到内存
这个还是可以实现的，就是用StringIO这个类型来做filelike-object
```
    def select(self, user_id, date):
        bkt = self.bkt
        key = '{user_id}/{date}'.format(user_id=user_id, date=date)
        f = StringIO.StringIO()
        self.s3.Bucket(bkt).download_fileobj(key, f)
        return f.getvalue()
```
另外这里说一点，采用s3作为后端存储，系统设计时候需要注意的是bucket数量不能太多(<100)，key这一层可以认为没有限制。s3的存储很像文件系统，bucket可以抽象为一个物理设备，而key则构成了设备上的目录(采用aws s3 ls命令也可以很明显的发现这一点)
