title: python监控小猪巴士票预售
date: 2015-12-31 09:54:02
tags:
  - python
  - crontab
---

数据获取
--------

本片将记录如何使用python查询小猪巴士是否有新票放出，在[ubuntu下使用fiddler对Android应用进行抓包 ](http://zhangming0509.github.io/2015/12/29/ubuntu-mono-fiddler/)
已经记录了如何真机抓包，线路查询接口如下图所示：
![](http://7xkbsf.com1.z0.glb.clouddn.com/15-12-31/31957377.jpg?imageView/2/w/720/q/90)
使用curl尝试发送请求：
```bash
curl -X POST -d "lineId=150716105019140882&lineBaseId=150716105018892878" http://bsapp.pig84.com/customline_app/app_book/getLineAndClassInfo.action
```
响应结果如下：
<!--more-->

```json
{
    "a1": "07:30",
    "a2": "岗头市场南（B班）",
    "a3": "岗头市场南(B班)",
    "a4": "软件产业基地",
    "a5": "22",
    "a6": "60",
    "a7": "6.90",
    "a8": "150129085654531320",
    "a9": "150716105019140882",
    "a10": "6.90",
    "a11": "30",
    "a12": "粤BAP278",
    "a13": "0",
    "a14": "",
    "a15": "",
    "list": [
        {
            "a1": "150817110913979834",
            "a2": "岗头市场南（B班）",
            "a3": "114.067946",
            "a4": "22.665774",
            "a5": "1"
        },
        {
            "a1": "150814204118757900",
            "a2": "岗头市场南(B班)",
            "a3": "114.067897",
            "a4": "22.665882",
            "a5": "1"
        },
        {
            "a1": "150814204118963901",
            "a2": "华为单身公寓北",
            "a3": "114.072399",
            "a4": "22.656271",
            "a5": "1"
        },
        {
            "a1": "150814204119170902",
            "a2": "万科城北",
            "a3": "114.072945",
            "a4": "22.653974",
            "a5": "1"
        },
        {
            "a1": "150814204119335903",
            "a2": "大冲①",
            "a3": "113.958042",
            "a4": "22.546029",
            "a5": "2"
        },
        {
            "a1": "150814204119503904",
            "a2": "高科技中心",
            "a3": "113.952042",
            "a4": "22.543585",
            "a5": "2"
        },
        {
            "a1": "150814204119686905",
            "a2": "深港研究基地",
            "a3": "113.952079",
            "a4": "22.539312",
            "a5": "2"
        },
        {
            "a1": "150814204119826906",
            "a2": "软件产业基地",
            "a3": "113.945932",
            "a4": "22.530629",
            "a5": "2"
        }
    ],
    "list1": [
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2015-12-31",
            "freeSeat": "-1",
            "lineClassid": "151216145140168993",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "1"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-01",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-02",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-03",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-04",
            "freeSeat": "0",
            "lineClassid": "151230103249617270",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-05",
            "freeSeat": "0",
            "lineClassid": "151230103249629272",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-06",
            "freeSeat": "0",
            "lineClassid": "151230103249636274",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-07",
            "freeSeat": "0",
            "lineClassid": "151230103249644276",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-08",
            "freeSeat": "0",
            "lineClassid": "151230103249654278",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-09",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-10",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-11",
            "freeSeat": "0",
            "lineClassid": "151230103249623271",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-12",
            "freeSeat": "0",
            "lineClassid": "151230103249633273",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-13",
            "freeSeat": "0",
            "lineClassid": "151230103249641275",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-14",
            "freeSeat": "0",
            "lineClassid": "151230103249649277",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-15",
            "freeSeat": "0",
            "lineClassid": "151230103249658279",
            "orderSeats": "47",
            "price": "6.90",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-16",
            "freeSeat": "-1",
            "isbook": "0"
        },
        {
            "lineBaseId": "150716105018892878",
            "orderStartTime": "07:30",
            "orderDate": "2016-01-17",
            "freeSeat": "-1",
            "isbook": "0"
        }
    ]
}
```
其中`list`为站点信息，`list1`为预售日期信息，预售期内的日期有返回`price`，所以将`list1`反转，
找到第一次出现`price`的条目并保存，再次请求时取出数据与前一次相比较，若相同则无新票预售。
代码如下：

check_bus.py
```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import requests
import json
import memcache
import datetime

# connect to memcached
mc = memcache.Client(['localhost:11211'], debug=0)


class BusBookChecker():

    def __init__(self, lineId, lineBaseId):
        self.url = 'http://bsapp.pig84.com/customline_app/app_book/getLineAndClassInfo.action'
        self.payload = {"lineId": lineId, "lineBaseId": lineBaseId}
        self.bookable_date = mc.get('bookable_date')
        if not self.bookable_date:
            bookable_date = datetime.date.today().strftime("%Y-%m-%d")
            mc.set('bookable_date', bookable_date)
            self.bookable_date = bookable_date

    def check(self):
        result = requests.post(self.url, self.payload)
        order_dates = result.json()['list1']
        order_dates.reverse()
        for date in order_dates:
            if 'price' in date:
                if self.bookable_date != date['orderDate']:
                    mc.set('bookable_date', date['orderDate'])
                    send_message()
                break
        return self.bookable_date


def send_message():
    print 'send sms to user'


if __name__ == "__main__":
    checker = BusBookChecker(
        '150716105019140882', '150716105018892878')
    print checker.check()
```

修改check_bus.py的权限，增加可执行权限：
```bash
$ chmod u+x check_bus.py
$ ./check_bus.py
当前预售期到2016-01-15
```

定时执行
--------

使用crontab每小时执行一次脚本，编辑任务列表
```bash
$ crontab -e
```
在文件末尾添加一行，每小时执行一次查询。
```bash
0 * * * * /path/check_bus.py
```
查看当前有哪些定时任务：
```bash
$ crontab -l
# Edit this file to introduce tasks to be run by cron.
#
# Each task to run has to be defined through a single line
# indicating with different fields when the task will be run
# and what command to run for the task
#
# To define the time you can provide concrete values for
# minute (m), hour (h), day of month (dom), month (mon),
# and day of week (dow) or use '*' in these fields (for 'any').#
# Notice that tasks will be started based on the cron's system
# daemon's notion of time and timezones.
#
# Output of the crontab jobs (including errors) is sent through
# email to the user the crontab file belongs to (unless redirected).
#
# For example, you can run a backup of all your user accounts
# at 5 a.m every week with:
# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
#
# For more information see the manual pages of crontab(5) and cron(8)
#
# m h  dom mon dow   command
0 * * * * /path/check_bus.py
```
删除所有定时任务，不提示是否确认：
```bash
$ crontab -r
$ crontab -l
no crontab for ming
```

删除任务时提示是否确认：
```bash
$ crontab -ir
crontab: really delete ming's crontab? (y/n) y
$ crontab -l
no crontab for ming
```

参考链接：
[python 链接和操作 memcache](http://www.cnblogs.com/unsea/archive/2013/01/30/2883738.html)
[Ubuntu中memcached的安装、配置和启用关闭](http://nonfu.me/p/2191.html)
[每天一个linux命令（50）：crontab命令](http://www.cnblogs.com/peida/archive/2013/01/08/2850483.html)
