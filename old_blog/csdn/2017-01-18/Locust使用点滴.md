[locust](http://locust.io/)的基本使用方法可以直接参考[官方文档](http://docs.locust.io/en/latest/quickstart.html)  
 <!--more-->  
这里主要记下一些使用时候遇到的细节需要注意的地方

### 1. --no-web的使用
参考官方文档，写了一个简单测试，但是运行时候总是无法发起请求：
```
(locust) liujinliu@liujinliu-OptiPlex-7010:~/code/servertest/locust$ locust --host=http://{ip}:{port} --print-stats
/home/liujinliu/venv/locust/local/lib/python2.7/site-packages/locust/rpc/__init__.py:6: UserWarning: WARNING: Using pure Python socket RPC implementation instead of zmq. If running in distributed mode, this could cause a performance decrease. We recommend you to install the pyzmq python package when running in distributed mode.
  warnings.warn("WARNING: Using pure Python socket RPC implementation instead of zmq. If running in distributed mode, this could cause a performance decrease. We recommend you to install the pyzmq python package when running in distributed mode.")
[2017-01-14 18:23:46,358] liujinliu-OptiPlex-7010/INFO/locust.main: Starting web monitor at *:8089
[2017-01-14 18:23:46,358] liujinliu-OptiPlex-7010/INFO/locust.main: Starting Locust 0.7.5
 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                              0     0(0.00%)                                       0.00

 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                              0     0(0.00%)                                       0.00

 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                              0     0(0.00%)                                       0.00

 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                              0     0(0.00%)                                       0.00

^C[2017-01-14 18:23:53,095] liujinliu-OptiPlex-7010/ERROR/stderr: KeyboardInterrupt
[2017-01-14 18:23:53,095] liujinliu-OptiPlex-7010/INFO/locust.main: Shutting down (exit code 0), bye.
 Name                                                          # reqs      # fails     Avg     Min     Max  |  Median   req/s
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
 Total                                                              0     0(0.00%)                                       0.00

Percentage of the requests completed within given times
 Name                                                           # reqs    50%    66%    75%    80%    90%    95%    98%    99%   100%
--------------------------------------------------------------------------------------------------------------------------------------------
--------------------------------------------------------------------------------------------------------------------------------------------
```
在最后加上```--no-web```的参数就好了  
即正确写法是这样：
```
locust --host=http://{ip}:{port} --print-stats --no-web
```
