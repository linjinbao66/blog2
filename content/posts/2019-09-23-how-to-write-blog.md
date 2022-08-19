---

title: GitHub博客教程 
categories: ["教程", "GitHub"] 
tags: ["教程"] 
date: 2019-09-23

---

## GitHub博客

作为一名`程序员`，每天的学习必不可少，但是，如果只是乱糟糟的学习，而不懂得如何去记笔记，那效率可想而知。一个**技术博客**，是大多数程序员的必须品。漂亮的博客，让工作顺心，让学习更有效率，更有动力。今天，我来教大家如何利用GitHub提供的免费服务，打造自己的专属博客，`白嫖`美滋滋。

### 前期准备

1. [GitHub账号](https://github.com/)
2. [Hugo 软件](https://www.gohugo.org/)
3. [Git客户端](https://git-scm.com/downloads)
4. 掌握Markdown语法

   需要准备的东西就是这么多，账号去官网注册就行，几个软件下载安装就可以，具体怎么安装，以及如何跨平台啥的，这里不作阐述。需要注意的是Markdown语法一定要会，起码简单的语法懂得，因为写文章都要用Markdown来写，`.md`是他的标志。

### 第一篇文章

准备好以上条件后，我们来写一篇博客

1. 建立博客目录

在硬盘目录建立一个空文件夹，可以任意命名，我这里是`F:\`，我放在了`F`盘根目录

1. 使用hugo生成站点

打开`cmd`，`cd`到`F:\`目录

```text
hugo new site amrom
```

此时在`F:\`下的目录结构如下：

```text
F:\>tree
卷 新加卷 的文件夹 PATH 列表
卷序列号为 F4A2-CDCD
F:.
└─amrom
    ├─archetypes
    ├─content
    │  ├─posts
    │  └─resources
    │      └─_gen
    │          ├─assets
    │          └─images
    ├─data
    ├─layouts
    ├─public
    ├─static
    │  └─images
    │      └─me
    └─themes
        ├─hyde
        │  ├─archetypes
        │  ├─images
        │  ├─layouts
        │  │  ├─partials
        │  │  └─_default
        │  └─static
        │      └─css
        ├─material-design
        │  ├─archetypes
        │  ├─images
        │  ├─layouts
        │  │  ├─partials
        │  │  └─_default
        │  └─static
        │      ├─css
        │      ├─font
        │      │  ├─material-design-icons
        │      │  └─roboto
        │      ├─images
        │      └─js
        ├─public
        │  ├─categories
        │  └─tags
        └─resources
            └─_gen
                ├─assets
                └─images

F:\>
```

此处重点看文件夹目录和一级二级目录，因为我个目录结构和最初的变化过了，多了很多东西，例如下面的`hyde`和`material-design`是我下载自定义的主题

1. 安装主题

hugo默认是没有主题的，你可以自己写，也可以从[官方](https://www.gohugo.org/theme/)下载主题，步骤如下：

```text
# 创建 themes 目录
$ cd themes
$ git clone https://github.com/spf13/hyde.git
```

1. 写文章

有了主题之后，我们来写一篇文章试试 可以使用命令`hugo new xx.md`创建，也可以用任意编辑器写`md`文档，然后放置到`content`目录下，注意，如果是前者，你需要在**博客根目录**此处是`amrom`执行以上命令，注意格式：

```text
+++
date = "2019-09-23T08:36:54-07:00"
draft = false
title = "关于"

+++

正文内容
```

`draft`如果是`true`，则视作草稿，在编译博客的时候会被忽略。

1. 本地调试

此时，我们已经完成了一篇文章的书写，可以启动`hugo`服务器进行查看，不满意的地方进行修改 在博客根目录，执行`$ hugo server --theme=hyde --buildDrafts`，指定了主题为`hyde`，在浏览器打开[http://localhost:1313](http://localhost:1313)，截图示例： ![&#x793A;&#x4F8B;](https://raw.githubusercontent.com/linjinbao666/resources/master/amromPicture/%E7%A4%BA%E4%BE%8B.png)

1. 部署到`GitHub Pages`上

   只放在本地多没意思，可以用免费的服务器多爽，古语有云：**一时白嫖一时爽，一直白嫖一直爽**

首先，在GitHub上创建一个Repository，命名为：coderzh.github.io （coderzh替换为你的github用户名）； 然后，在站点根目录执行 Hugo 命令生成最终页面：`$ hugo --theme=hyde --baseUrl="http://coderzh.github.io/"` 如果一切顺利，所有静态页面都会生成到 public 目录，将pubilc目录里所有文件 push 到刚创建的Repository的 master 分支。

```text
cd public
git init
git remote add origin https://github.com/coderzh/coderzh.github.io.git
git add -A
git commit -m "first commit"
git push -u origin master
```

浏览器访问：[http://coderzh.github.io/](http://coderzh.github.io/)

1. 结束

到了这里教程结束。总而言之，博客建立容易，**坚持写下去**才是难事。

PS：如果你有钱，那也可以买个VPS挂上去啊 PS：求大佬合租VPS 版权所有©[菜鸡聪](https://linjinbao666.github.io/) **欢迎转载**
