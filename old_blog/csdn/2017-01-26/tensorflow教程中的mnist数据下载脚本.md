阅读tensorflow教程过程中，最初的例子使用mnist，官方提供的下载脚本链接已经失效，从网上一直没有找到正确的input_data.py的脚本，查看官方教程，发现这个脚本在[这里](https://github.com/tensorflow/tensorflow/blob/master/tensorflow/contrib/learn/python/learn/datasets/mnist.py)但是下载下来之后发现，运行过程会报错，因为通过pip安装的tensorflow包里是不包含这部分代码的，同时这个文件依赖也在pip安装的tensor版本里也是没有的。  
 <!--more-->  
通过反复折腾，终于搞出来一个版本   
https://github.com/liujinliu/tensor_mnist
大家可以直接fork我这份工程，或是直接把代码down下来，这里边包含了mnist图片文件。main.py就是描述的[这篇教程](http://www.tensorfly.cn/tfdoc/tutorials/mnist_beginners.html)的第一部分的内容。  可以在安装tensorflow之后直接运行这个文件。
 
####另外安利我写的两个模块，欢迎使用或贡献代码：
[一个命令行版本的zookeeper cli](https://pypi.python.org/pypi/zoo_cmd)
[一个tornado web app框架快速生成工具](https://pypi.python.org/pypi/twapp)
