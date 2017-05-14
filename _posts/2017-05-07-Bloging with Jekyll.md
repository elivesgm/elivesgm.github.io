---
layout: post
title: "Blogging with jekyll"
date: 2017-05-07 18:15:06 
description: "Blogging with jekyll"
tag: Tools
---


### 1 Blogging with jekyll

#### a 创建项目
在github上创建一个空项目

#### b 本地clone项目
>* git clone git@github.com:elivesgm/elivesgm.github.io.git

#### c 创建gh-pages分支
>* git checkout -b gh-pages
>* github上只有在该分支中的页面才会生成网页。以下操作都在该分支进行。</p>

#### d 创建jekyll配置文件
在项目根目录下，建立一个名为_config.yml的文本文件。它是jekyll的设置文件，我们在里面填入如下内容，其他设置都可以用默认选项，具体解释参见官方网页。
    baseurl: /jekyll_demo

#### d 创建模板文件
在项目根目录下，创建一个_layouts目录，用于存放模板文件。
>* $ mkdir _layouts
进入该目录，创建一个default.html文件，作为Blog的默认模板。并在该文件中填入以下内容。


#### e 创建Blog
在项目根目录，创建一个_posts目录，用于存放blog文章。
>* $ mkdir _posts

进入该目录，创建第一篇文章。文章就是普通的文本文件，文件名假定为2012-08-25-hello-world.html。(注意，文件名必须为"年-月-日-文章标题.后缀名"的格式。如果网页代码采用html格式，后缀名为html；如果采用markdown格式，后缀名为md。）

在该文件中，填入以下内容：（注意，行首不能有空格）


#### f 创建首页

它的Yaml文件头表示，首页使用default模板，标题为"我的Blog"。

#### g 发布内容


上传成功之后，等10分钟左右，访问http://username.github.com/jekyll_demo/就可以看到Blog已经生成了（将username换成你的用户名）。

{{ page.date | date_to_string }}

