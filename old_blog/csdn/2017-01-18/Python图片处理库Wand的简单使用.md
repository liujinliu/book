#  ImageMagick安装
 <!--more-->  
ubuntu下安装
sudo apt-get install libmagickwand-dev
其他系统安装方法可以参考
http://docs.wand-py.org/en/0.4.2/guide/install.html

# Wand安装
pip install Wand

# 简单使用
### 图片缩放
```
from wand.image import Image
from wand.drawing import Drawing
from wand.color import Color
def resize_photo(filename, width, 
                     height, target):
    with Image(filename=filename) as img:
        img.resize(width, height)
        img.save(filename=target)
```
图片filename将被缩放到width*height大小，并且重新保存成target

### 图片拼接
这里讲一种拼接方式是先画一张纯白背景，然后将一张图片放到这个背景的某个位置
```
from wand.image import Image
from wand.drawing import Drawing
from wand.color import Color

# 画一个纯白背景，并保存成filename
def draw_rec(width, height, filename):
    with Image(width=width,height=height,
               background=Color('White')) as img:
        img.save(filename=filename)
        
# 将图片img放在img_back的上面，具体位置是距左边界left个像素， 
# 距离上边界top个像素，生成的新图片保存成filename
def composite(img_back, img, left, top, target):
    with Image(filename = img_back) as w:
        with Image(filename = img) as r:
            with Drawing() as draw:
                draw.composite(operator = 'atop',
                        left = left, top = top,
                        width = r.width,
                        height = r.height,
                        image = r)
                draw(w)

if __name__ == '__main__':
    draw_rec(640, 800, 'background.jpg')
    composite('background.jpg', 'front.jpg', 50, 100, 'final.jpg')
```
### 命令行的简单使用
composite  
实现两张图片的拼接(一张拼到另一张的上面)
```
composite -gravity northwest -geometry +{left}+{top} {front.jpg} {background.jpg} {target.jpg}
northwest: 表示以左上角为坐标原点
{left}:距离左边界的距离(像素)
｛top}:距离上边界的距离(像素)
{front.jpg}: 上面的图片
{background.jpg}: 下面的图片
{target.jpg}: 生成的图片
```
