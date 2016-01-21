title: 使用PIL获取图片主色调
date: 2016-01-21 14:25:29
tags:
  - python
  - PIL
---

最近看到`Pocket casts`这个APP中有些页面是取了上一层入口处点击的图片中的颜色作为背景，所以就在想这个是怎么实现的，在网上发现了一篇文章[获取图片的主色调](http://www.cnblogs.com/michaelhuwei/p/3701535.html)，是用c#实现的，我按原作者的思路用Python做了一个类似的版本，原作者的思路是：
```
我想了一个简单的办法，就是根据图片中每个像素的色调值去判断哪些像素符合“醒目”这个特性。

分三步进行

1.计算整个图片的色调的平均值 (avg_hue)

2.遍历每个像素，计算该像素的色调值与 avg_hue 的色差（即将二者相减后取绝对值），如果该色差大于一个阈值（本文中取 30），

   则将该像素加入到“醒目像素”的列表

3.计算整个“醒目像素列表”的颜色均值，得到的结果即为该图片的主色调。
```


python版本代码如下：

```python
import colorsys
from PIL import Image


def get_accent_color(path):
    im = Image.open(path)
    if im.mode != "RGB":
        im = im.convert('RGB')
    delta_h = 0.3
    avg_h = sum(t[0] for t in[colorsys.rgb_to_hsv(*im.getpixel((x,y))) for x in range(im.size[0]) for y in range(im.size[1])])/(im.size[0]*im.size[1])
    beyond = filter(lambda x: abs(colorsys.rgb_to_hsv(*x)[0]- avg_h)>delta_h ,[im.getpixel((x,y)) for x in range(im.size[0]) for y in range(im.size[1])])
    r = sum(e[0] for e in beyond)/len(beyond)
    g = sum(e[1] for e in beyond)/len(beyond)
    b = sum(e[2] for e in beyond)/len(beyond)
    for i in range(im.size[0]/2):
        for j in range(im.size[1]/10):
            im.putpixel((i,j), (r,g,b))
    im.save('res'+path)
    return r, g, b
```

  1. 当图片的`mode`不是`RGB`时需要将其转换成`RGB`。因为如果是png图片，其`mode`可能是`P`,`getpixel`返回的是一个整数而不是一个rgb的tuple，导致`rgb_to_hsv`不能使用。
  2. `rgb_to_hsv`的结果中`h`的取值范围是0-1，所以相应的色差值我们改为`0.3`。
  3. `putpixel((x, y), (r, g, b))`在图片的(x, y)坐标画一个颜色为`(r, g, b)`的点。
  4. 如果要提高执行效率，可以生成缩略图后再处理。

效果展示：
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-21/86402189.jpg)
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-21/5371713.jpg)
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-21/49477423.jpg)
从结果看前两张效果不错，后边的效果太差，所以说如果要更好的效果那可能就需要什么Machine Learning什么的高深技术。

参考链接
[获取图片的主色调](http://www.cnblogs.com/michaelhuwei/p/3701535.html)
[python判断、获取一张图片主色调的2个实例](http://www.jb51.net/article/48875.htm)
