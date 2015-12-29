title: ubuntu下使用fiddler对Android应用进行抓包
date: 2015-12-29 09:50:47
tags:
  - fiddler
  - mono
---

安装
-----

之前在ubuntu下尝试了使用wine安装fiddler，装完后打不开，后来发现还有种方法就是mono。
首先安装mono:

```bash
$ sudo apt-get install mono-complete
```

然后下载[fiddler for mono](http://fiddler.wikidot.com/mono),选择`Current Linux build`就行了，下载完成后解压，cd到解压目录执行：

```bash
$ mono Fiddler.exe
```
<!--more-->

![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-29/44673911.jpg?imageView/2/w/720/q/90)

设置
------

连接设置，Tools->Fiddler Options->connection,勾选allow remote computer to connect
![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-29/43193016.jpg?imageView/2/w/720/q/90)

查看本机ip
```bash
$ ifconfig
```
![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-29/95993753.jpg?imageView/2/w/480/q/90)
打开android设备的“设置”->“WLAN”,打开代理服务器，设置地址为上边获取的ip（192.168.1.66）,
端口为connection中设置的8888
![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-29/51468114.jpg?imageView/2/w/320/q/90
)

配置Fiddler允许监听https

默认下，Fiddler不会捕获HTTPS会话，需要设置, 打开Fiddler Tool->Fiddler Options->HTTPS页签。
选中decrypt https traffic和ignore server certificate errors两项
![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-29/56537623.jpg?imageView/2/w/720/q/90)

参考链接：
[Linux(Ubuntu)环境下使用Fiddler](http://www.cnblogs.com/jcli/p/4474332.html)
[利用Fiddler对Android https请求进行监测](http://www.linuxidc.com/Linux/2014-02/97024.htm)
