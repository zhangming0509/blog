title: 使用PIL生成缩略图
date: 2016-01-19 14:45:27
tags:
  - python
  - PIL
---

两种生成缩略图方法的区别
-------------------------

使用PIL生成缩略图用两种方式，`resize`和`thumbnail`,区别在于使用`reszie`会返回一个新对象，
而使用`thumbnail`则会在原对象上进行修改，即`thumbnail`会覆盖原图。

```bash
>>> from PIL import Image
>>> im = Image.open('a.jpg')
>>> im.size   # 原图尺寸
(3264, 2448)
>>> id(im)
140253860921640
>>> resize_im = im.resize((100,100))
>>> resize_im
<PIL.Image.Image image mode=RGB size=100x100 at 0x7F8F65A0A518>
>>> id(resize_im)
140253862077720
>>> thumb_im = im.thumbnail((100, 100))
>>> thumb_im
>>> im.size   # 使用thumbnail后的原图尺寸改变，resize后的结果不一定等于指定的尺寸，因为是按比例缩放的
(100, 75)
```

<!--more-->

缩略图与原图方向不一致的分析
--------------------------

部分图片产生的缩略图会与原图不一致，如下所示：

原图
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-19/43179932.jpg?imageView/2/w/400/q/90)

缩略图
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-19/91777846.jpg?imageView/2/w/400/q/90)
在上边的代码中获得的原图的尺寸`image.size`是(3264, 2448)，并非图片浏览器中的2448X3264,只是图片浏览器自动
为我们对图片进行了旋转，使其能正确的显示给我们，而图片浏览器也是根据原图的`Exif`信息中的`Orientation`这个Tag
的值来对图片进行旋转的。我们产生的缩略图是对未经过旋转的原图进行缩略，所以会造成如上所示的方向不一致，为了使原图
和缩略图看起来方向一致则需要在生成缩略图之前对原图进行旋转，下边的代码用于获取原图中的`Exif`信息。

```python
from PIL.ExifTags import TAGS
from PIL import Image

def get_exif(im_obj):
    info = im_obj._getexif()
    ret = dict()
    if info:
        for tag, value in info.items():
            decoded = TAGS.get(tag, tag)
            ret[decoded] = value
    return ret
```
获取`a.jpg`的`Exif`信息
```bash
>>> import pprint
>>> from PIL import Image
>>> im = Image.open('a.jpg')
>>> pprint.pprint(get_exif(im))
{'DateTime': u'2015:11:01 17:38:18',
 'DateTimeOriginal': u'2015:11:01 17:38:22',
 'ExifOffset': 414,
 'Flash': 0,
 'FocalLength': (459, 100),
 'GPSInfo': {1: u'N',
             2: ((22, 1), (32, 1), (49, 1)),
             3: u'E',
             4: ((114, 1), (3, 1), (28, 1))},
 'ImageLength': 2448,
 'ImageWidth': 3264,
 'JpegIFByteCount': 6716,
 'JpegIFOffset': 444,
 'LightSource': 0,
 'Make': u'Meizu GPS',
 'MeteringMode': 1,
 'Model': u'MX(35)\x00',
 'Orientation': 6,
 'WhiteBalance': 0}
```
此处我们获取到`Orientation`的值为`6`，`Orientation`取值的含义参考下图。
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-19/36949893.jpg?imageView/2/w/400/q/90)
我们最终要旋转的角度就是`Encoded JPEG`到`Screen View`的角度，由图可知当`Orientation`
的值为`6`时需要逆时针旋转90度。对应PIL中的`rotate`函数需要传入参数`-90`，获取具体旋转角度的代码如下：
```python
def get_rotate_degree(im_obj):
    degree = 0
    if hasattr(im_obj, '_getexif'):
        info = im_obj._getexif()
        ret = dict()
        degree_dict = {1: 0, 3: 180, 6: -90, 8: 90}
        if info:
            orientation = info.get(274, 0)
            degree = degree_dict.get(orientation, 0)
    return degree
```
调用此函数计算原图应旋转的角度
```bash
>>> get_rotate_degree(im)
-90
```
旋转后生成缩略图
```bash
>>> im = Image.open('a.jpg')
>>> new_img = im.rotate(get_rotate_degree(im))
>>> new_img.thumbnail((600, 800))
>>> new_img.save('rotated_thumbnail.jpg', 'JPEG')
>>> new_img.size
(600, 800)
```
最终结果如下：
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-19/27325129.jpg?imageView/2/w/400/q/90)

裁剪后缩略
----------

我们生成缩略图用于头像或图片预览，我们想要的是一个正方形的缩略图，而非等比缩放的缩略图，上边提到了缩略结果
与我们指定的参数不同，要生成一个正方形就要在原图中裁剪出一个最大的正方形然后再进行缩略。通过下边的代码
我们可以获取到原图的裁剪区域。
```
def get_crop_region(width, height):
    if width < height:
        left = 0
        upper = (height - width) / 2
        right = width
        lower = upper + width
    elif width > height:
        left = (width - height) / 2
        upper = 0
        right = left + height
        lower = height
    else:
        left = 0
        upper = 0
        right = width
        lower = height
    return (left, upper, right, lower)
```
生成正方形缩略图：
```bash
>>> im = Image.open('a.jpg')
>>> im.size
(3264, 2448)
>>> im = im.rotate(get_rotate_degree(im))
>>> im.size
(2448, 3264)
>>> get_crop_region(im.size[0], im.size[1])
(0, 408, 2448, 2856)
>>> copy = im.crop(get_crop_region(im.size[0], im.size[1]))
>>> copy.size
(2448, 2448)
>>> copy.thumbnail((600, 600))
>>> copy.size
(600, 600)
>>> copy.save('square_thumbnail.jpg', 'JPEG')
```
正方形缩略图如下所示：
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-19/35458890.jpg?imageView/2/w/400/q/90)


参考链接：
[JPEG Rotation and EXIF Orientation](http://www.impulseadventure.com/photo/exif-orientation.html)
[Python读取图片EXIF信息](http://www.jb51.net/article/52063.htm)
