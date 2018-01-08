转自：http://www.cnblogs.com/ccdc/p/4069112.html
 <!--more-->  
### 安装
```
#尤其重要，否则会报错
yum install python-devel

yum install libjpeg libjpeg-devel zlib zlib-devel freetype freetype-devel lcms lcms-devel

yum install python-imaging
```
### 下载PIL所需源码包
http://www.pythonware.com/products/pil/#pil117 下载最新版的PIL安装程序
http://www.ijg.org 下载jpeg库
http://www.gzip.org/zlib/ 下载支持压缩功能的zlib库

### 解压安装jpeg包
```
#解压
tar zxf jpegsrc.v9b.tar.gz
cd jpeg-9b/
./configure && make && make test && make install
```
修改Image的配置文件
```
#修改配置文件setup.py
TCL_ROOT = "/usr/lib/"    
JPEG_ROOT = "/usr/lib/"    
ZLIB_ROOT = "/usr/lib/"    
TIFF_ROOT = "/usr/lib/"    
FREETYPE_ROOT = "/usr/lib/"   
LCMS_ROOT = "/usr/lib/"
```
安装Image
```
tar xvfz Imaging-1.1.7.tar.gz
cd cd Imaging-1.1.7/
python setup.py build_ext -i
python setup.py install
```
