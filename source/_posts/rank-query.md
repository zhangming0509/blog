title: SQL和sqlalchemy实现根据点赞和评论数量排序
date: 2016-01-11 13:49:13
tags:
  - SQL
  - sqlalchemy
---

表结构
------

表结构如下所示， 用户可以点赞和评论，最后需要综合点赞和评论数量对结果降序排列：
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-11/18995526.jpg?imageView/2/w/720/q/90)

<!--more-->

使用sqlalchemy建模
---------------------

```python
from flask.ext.sqlalchemy import SQLAlchemy

db = SQLAlchemy(app)


class User(db.Model):
    __tablename__ = 'user'
    id = Column(Integer, primary_key=True)
    username = Column(String(16), unique=True, nullable=False)
    password = Column(String(255), nullable=False)
    email = Column(String(255))
    modified = Column(DateTime, default=datetime.now, onupdate=datetime.now)
    created = Column(DateTime, default=datetime.now)


class DatingShow(db.Model):
    __tablename__ = 'dating_show'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id'), nullable=False)
    user = relationship('User')
    title = Column(String(45), nullable=False)
    content = Column(String(45), nullable=False)
    photo = Column(String(45))
    modified = Column(DateTime, default=datetime.now, onupdate=datetime.now)
    created = Column(DateTime, default=datetime.now)


class DatingShowComment(db.Model):
    __tablename__ = 'dating_show_comment'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id'), nullable=False)
    user = relationship('User')
    datingshow_id = Column(
        Integer, ForeignKey('dating_show.id'), nullable=False)
    dating_show = relationship(
        'DatingShow', backref=backref(
            'comments', lazy='dynamic', cascade="all, delete-orphan"))
    content = Column(String(255), nullable=False)
    photo = Column(String(33*9))
    modified = Column(DateTime, default=datetime.now, onupdate=datetime.now)
    created = Column(DateTime, default=datetime.now)


class DatingShowLike(db.Model):
    __tablename__ = 'dating_show_like'
    id = Column(Integer, primary_key=True)
    user_id = Column(Integer, ForeignKey('user.id'), nullable=False)
    user = relationship('User')
    datingshow_id = Column(
        Integer, ForeignKey('dating_show.id'), nullable=False)
    dating_show = relationship(
        'DatingShow', backref=backref(
            'likes', lazy='dynamic', cascade="all, delete-orphan"))
    modified = Column(DateTime, default=datetime.now, onupdate=datetime.now)
    created = Column(DateTime, default=datetime.now)
```

* 对于`modified`和`created`的default值需要传入的是`datetime.now`这个函数，并非`datetime.now()`,
后者是一个具体的值，不会动态变化，取值为模块第一次载入时的时间
* 字段`modified`设置了属性`onupdate`，当任意字段值更新后`modified`字段值会更新为当前时间
* `DatingShowLike`和`DatingShowComment`分别对引用关系设置了`cascade`属性，
`all`中包含了`delete`，意思是父对象删除时子对象也跟着删除，`delete-orphan`设置当引用关系解除时，子对象被删除


查询晒约榜的评论的点赞数量
------------------------------------

评论算2票，点赞算1票，按得票总数降序排列。

sql版本如下：

```sql
select
    *
from
    (select
        d.id,
            sub_comment.comment_count as sub_c_count,
            sub_like.like_count as sub_like_count,
            (sub_comment.comment_count * 2 + sub_like.like_count) as total_count
    from
        dating_show as d
    left join (select
        count(*) as comment_count, dc.datingshow_id as dcid
    from
        dating_show_comment as dc
    group by dc.datingshow_id
    order by comment_count) as sub_comment ON sub_comment.dcid = d.id
    left join (select
        count(*) as like_count, di.datingshow_id as diid
    from
        dating_show_like as di
    group by di.datingshow_id
    order by like_count) as sub_like ON sub_like.diid = d.id
    -- where d.created between date_sub(curdate(), interval 1 day) and curdate()
    order by sub_comment.comment_count desc) as sub
order by sub.total_count desc;
```


* `ifnull`,当查询的某一字段结果为null时指定一个默认值替换，下边是两种情况下的结果
  + 不使用`ifnull`的查询结果如下，有一个字段为空，结果则为空：
  ![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-12/57662675.jpg)
  + 使用了`ifnull`的结果如下，空的字段被当做0处理：
  ![](http://7xkbsf.com1.z0.glb.clouddn.com/16-1-12/12566949.jpg)
* `curdate`,返回当前日期字符串
  ```
  mysql> select curdate();
  +------------+
  | curdate()  |
  +------------+
  | 2016-01-12 |
  +------------+
  1 row in set (0.00 sec)
  ```
* `date_sub`,日期减法操作
  ```
  mysql> select date_sub(curdate(), interval 1 day);
  +-------------------------------------+
  | date_sub(curdate(), interval 1 day) |
  +-------------------------------------+
  | 2016-01-11                          |
  +-------------------------------------+
  1 row in set (0.00 sec)
  ```

sqlalchemy 版本


```python
from sqlalchemy import case
from sqlalchemy.sql import func
import datetime

sub_like = s.query(DatingShowLike.datingshow_id, func.count('*').label('like_count')).group_by(DatingShowLike.datingshow_id).subquery()

sub_comment = s.query(DatingShowComment.datingshow_id, func.count('*').label('comment_count')).group_by(DatingShowComment.datingshow_id).subquery()

sub = s.query(
  DatingShow.id, DatingShow.created,
  case(
    [(sub_comment.c.comment_count == None, 0)],
    else_=sub_comment.c.comment_count).label('comment_count'),
  case(
    [(sub_like.c.like_count == None, 0)],
    else_=sub_like.c.like_count).label('like_count')).outerjoin(sub_comment,
  DatingShow.id==sub_comment.c.datingshow_id).outerjoin(
  sub_like, DatingShow.id==sub_like.c.datingshow_id).subquery()

end_subquery = s.query(
  sub.c.id, sub.c.created, sub.c.like_count, sub.c.comment_count, (2*sub.c.comment_count+sub.c.like_count).label('total_count')).subquery()

for res in s.query(end_subquery).order_by(end_subquery.c.total_count.desc(), end_subquery.c.id.desc()):
    print res.id, res.created.date(), res.comment_count, res.like_count, res.total_count
```

* `label`方法用于给查询的字段起一个别名，同sql中的`as`
* `case`函数等同于sql中的`ifnull`
* `func.count`等同于sql中的`count`函数

输出结果如下，对比sql查询结果一致：
```bash
>>> for res in s.query(end_subquery).order_by(end_subquery.c.total_count.desc(), end_subquery.c.id.desc()):
...     print res.id, res.created.date(), res.comment_count, res.like_count, res.total_count
...
68 2015-12-29 3 4 10
65 2015-12-28 2 4 8
69 2015-12-30 2 2 6
62 2015-12-27 2 2 6
63 2015-12-27 2 1 5
59 2015-12-26 1 3 5
78 2015-12-30 0 3 3
72 2015-12-30 1 1 3
71 2015-12-30 0 3 3
70 2015-12-30 0 3 3
61 2015-12-26 0 3 3
56 2015-12-26 0 3 3
51 2015-12-25 1 1 3
80 2016-01-04 0 2 2
76 2015-12-30 0 2 2
75 2015-12-30 0 2 2
60 2015-12-26 0 2 2
58 2015-12-26 0 2 2
```

参考链接：
[mysql中时间比较的实现](http://www.cnblogs.com/weiwang/p/3245915.html)
