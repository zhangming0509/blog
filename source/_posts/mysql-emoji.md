title: mysql存储emoji
date: 2016-02-24 11:59:20
tags:
  - mysql
  - python
  - Flask
---

mysql使用utf8编码时不能存储emoji,因为emoji是占4个字节的，而普通字符是3个字节。myslq从5.5.3版本之后引入了一种新的编码方式`utf8mb4`，支持emoji包括火星文。下边的例子是基于`flask`的一个新建的项目中使用`utf8mb4`编码。

新建数据库设定字符集
--------------

```bash
$ mkdir emoji
$ touch run.py
$ cd emoji
$ touch __init__.py
$ mkdir templates
$ touch templates/test.html
```

<!--more-->

emoji/__init__.py

```python
# -*- coding:utf-8 -*-

from flask import Flask, request, render_template, flash, redirect, url_for
from flask.ext.sqlalchemy import SQLAlchemy
from flask.ext.wtf import Form
from wtforms import StringField, SubmitField
from wtforms.validators import Required
from flask.ext.bootstrap import Bootstrap

# Create the Flask application
app = Flask(__name__)

# Create the Flask-SQLAlchemy object
db = SQLAlchemy(app)

# Create Bootstrap object
bootstrap = Bootstrap(app)

# Config database uri
app.config['SQLALCHEMY_DATABASE_URI'] = \
    'mysql+pymysql://root:abc123..@localhost/emoji?charset=utf8mb4'
# config secret key
app.config['SECRET_KEY'] = 'kd8273f784jkkdjkj(jdj3'


class Info(db.Model):
    id = db.Column(db.Integer(), primary_key=True)
    content = db.Column(db.Unicode(40), nullable=False)


class InfoForm(Form):
    content = StringField('content', validators=[Required()])
    submit = SubmitField('submit')


@app.route('/', methods=['GET', 'POST'])
def index():
    form = InfoForm()
    if form.validate_on_submit():
        info = Info(content=form.content.data)
        db.session.add(info)
        db.session.commit()
        flash(u'添加成功')
        return redirect(url_for('index'))
    infos = [{"content": info.content} for info in Info.query.order_by(Info.id.desc())]
    return render_template('test.html', form=form, infos=infos)


if __name__ == '__main__':
    app.run(debug=True)
```
说明
- `bootstrap = Bootstrap(app)`不能忘记，否则会找不到`bootstrap/base.html`模板
- 数据库uri要加上相应的字符集参数`utf8mb4`


emoji/templates/test.html

```html
{% extends "bootstrap/base.html"  %}
{% import "bootstrap/wtf.html" as wtf %}
{% block title %}test emoji{% endblock %}

{% block content %}
<div class="container">
    <div class="page-header">
        <h1>Add info</h1>
    </div>
    <div class="col-md-12">
        {{ wtf.quick_form(form) }}
    </div>
    <div>
        <ul>
            {% for info in infos %}
                <li>{{ info.content }}</li>
            {% endfor %}
        </ul>
    </div>
</div>
{% endblock %}
```

run.py

```python
# -*- coding:utf-8 -*-
from emoji import app
from werkzeug.contrib.fixers import ProxyFix

app.wsgi_app = ProxyFix(app.wsgi_app)

if __name__ == '__main__':
    app.run(debug=True)
```

创建数据库`emoji`, 使用`utf8mb4`

```bash
➜  ~ mysql -u root -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 149
Server version: 5.5.47-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database emoji default charset utf8mb4;
Query OK, 1 row affected (0.00 sec)
```

通过`db.create_all`创建表

```bash
➜  ~ python       
Python 2.7.6 (default, Jun 22 2015, 17:58:13)
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> from emoji import db
/home/ming/.virtualenvs/kingmv/local/lib/python2.7/site-packages/flask_sqlalchemy/__init__.py:800: UserWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True to suppress this warning.
  warnings.warn('SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True to suppress this warning.')
>>> db.create_all()
>>>
```

查看生成了表`info`，其编码格式为`utf8mb4`

```bash
mysql> show tables;
+-----------------+
| Tables_in_emoji |
+-----------------+
| info            |
+-----------------+
1 row in set (0.00 sec)

mysql> show create table info;
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                                                                                                                                                                  |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| info  | CREATE TABLE `info` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `content` varchar(40) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8mb4 |
+-------+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)

mysql>
```

启动应用

```bash
$ python run.py
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 * Restarting with stat
```

打开手机浏览器,因为ubuntu系统无法输入imoji，输入'http://127.0.0.1:5000/', 测试“苹果”，“草莓”等可以显示，如下所示
![](http://7xkbsf.com1.z0.glb.clouddn.com/16-2-24/9653694.jpg?imageView/2/w/320/q/90)

修改已存在数据库的字符集
------------------

另一种情况是数据库已经存在，即一个运行中的项目需要修改字符集设置

首先查看字符集设置如下，`character_set_system`为`utf8`，需要将其修改为`utf8mb4`

```bash
mysql> SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
+--------------------------+-------------------+
| Variable_name            | Value             |
+--------------------------+-------------------+
| character_set_client     | latin1            |
| character_set_connection | latin1            |
| character_set_database   | latin1            |
| character_set_filesystem | binary            |
| character_set_results    | latin1            |
| character_set_server     | latin1            |
| character_set_system     | utf8              |
| collation_connection     | latin1_swedish_ci |
| collation_database       | latin1_swedish_ci |
| collation_server         | latin1_swedish_ci |
+--------------------------+-------------------+
```

按照[Using Emojis in Django Model Fields](blog.manbolo.com/2014/03/31/using-emojis-in-django-model-fields)的指引

```
# For each database:
ALTER DATABASE database_name CHARACTER SET = utf8mb4 COLLATE utf8mb4_unicode_ci;
# For each table:
ALTER TABLE table_name CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
# For each column:
ALTER TABLE table_name CHANGE column_name column_name VARCHAR(191) CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci NOT NULL;
# (Don’t blindly copy-paste this! The exact statement depends on the column     type, maximum length, and other properties. The above line is just an example for a `VARCHAR` column.)
```
需要修改数据库，表和相应的列的字符集格式，实际测试只需修改数据库和表的字符集格式即可。同时相应的也要修改数据库uri的字符集参数。

最后要修改数据库配置文件`/etc/mysql/my.cnf `为如下：

```
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

重启mysql服务即可。


参考链接
- [django，mysql存储emoji表情，utf8mb4](http://www.aichengxu.com/view/42443)
- [Using Emojis in Django Model Fields](blog.manbolo.com/2014/03/31/using-emojis-in-django-model-fields)
- [How to support full Unicode in MySQL databases](https://mathiasbynens.be/notes/mysql-utf8mb4)
- [TemplateNotFound: bootstrap/wtf.html](http://m.douban.com/group/topic/73189860/)
