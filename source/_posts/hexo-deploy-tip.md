title: Hexo部署相关
date: 2015-07-12 11:12:13
tags: hexo
---
记录一下hexo部署时遇到的问题, 执行了

```bash
$ hexo deploy
```

报错`hexo Deployer not found`, 这是因为hexo 3.0后需要单独安装`delpoyer package`,如果要部署在github则需要安装`hexo-deployer-git`, 其它部署方式所需要相应的deployer请参看[hexo deployment](https://hexo.io/docs/deployment.html)

```bash
$ npm install hexo-deployer-git --save
```

然后就是编辑config的delpoy部分

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:zhangming0509/zhangming0509.github.io.git
```

* repo的名字是你的github用户名加`.github.io`,每个用户只能创建一个[github page](https://pages.github.com/)
* 编辑config时要注意yml的语法,冒号后要有空格

<!--more-->

配置完成后执行

```bash
$ hexo d -g
```

提示`INFO  Deploy done: git`,现在访问`http://你的github用户名.github.io/`就可以看到新的文章了.

另外,也可使用`blog.git`的分支`gh-pages`去部署页面,相应的config修改为:

```
# Deployment
## Docs: http://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: git@github.com:zhangming0509/blog.git
  branch: gh-pages
```

`blog.git`就是你`hexo init blog`生成的项目对应的github仓库, gh-pages的开启需要在setting里边设置.
`gh-pages`会使用`http://你的github用户名.github.io/blog/`作为页面的地址. 现在使用`hexo d -g`会将`public`目录上传只`blog.git`项目的`gh-pages`分支. blog项目保存了markdown文件,将更新后的文件提交到blog.git

```bash
$ git cimmit -m "commit info"
```

推荐使用第二种方法,可以省下一个远端的repo.

tag使用
------------

hexo文章的头部文件是用YAML来写的，比如文章要同时标记多个tags，就需要用

```
tags: [tag1, tag2]
```

或者

```
tags:
  - tag1
  - tag2
```

发布显示更多
---------------

在你觉得适合的位置插入`<!--more-->`就会将之前的部分生成摘要。点击”阅读全文“才会看到全文。
