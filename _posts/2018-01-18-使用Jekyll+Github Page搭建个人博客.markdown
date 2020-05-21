---
layout:     post
title:      "使用Jekyll+Github Page搭建个人博客"
subtitle:   "本篇文章将展示如何通过静态站点生成器Jekyll编写网站源码，然后上传到GitHub，由GitHub生成并托管整个网站。"
date:       2018-01-18
author:     "Malcolm Suen"
header-img: "img/post-bg-2015.jpg"
header-mask:  0.3
catalog: true
tags:
    - Jekyll
    - 个人博客
    - Github
---

## 前言

Github很好的将代码和社区联系在了一起，于是发生了很多有趣的事情，世界也因为他美好了一点点。Github作为现在最流行的代码仓库，已经得到很多大公司和项目的青睐，比如jQuery、Twitter等。为使项目更方便的被人理解，介绍页面少不了，甚至会需要完整的文档站，Github替你想到了这一点，他提供了Github Pages的服务，不仅可以方便的为项目建立介绍站点，也可以用来建立个人博客。我们既拥有绝对管理权，又享受github带来的便利：不管何时何地，只要向主机提交commit，就能发布新文章。而且这一切还是免费的，github提供无限流量，世界各地都有理想的访问速度。

本篇文章将展示如何通过静态站点生成器Jekyll编写网站源码，然后上传到GitHub，由GitHub生成并托管整个网站。

这样做的好处是：

- 免费，无限流量
- 轻量级博客系统，配置简单
- 享受git的版本管理功能，不必担心文章丢失
- 可以绑定自己的域名

当然他也有缺点：

- 有一定技术门槛，需要了解一点git和网页开发
- 它生成的是静态网页，添加动态功能必须使用外部服务
- 它不适合大型网站，因为没有用到数据库，每运行一次都必须遍历全部的文本文件，网站越大，生成时间越长

## 配置和使用Github

### 安装配置Git工具

[GitHub账户的创建和配置](https://git-scm.com/book/zh/v2/GitHub-%E8%B4%A6%E6%88%B7%E7%9A%84%E5%88%9B%E5%BB%BA%E5%92%8C%E9%85%8D%E7%BD%AE)此处不再赘述，其他git相关知识可以参考[git中文教程](https://git-scm.com/book/zh/v2)。

macOS下使用`brew install git`安装，Linux（Debian/Ubuntu）使用`apt-get install git`，Windows[点此](https://git-scm.com/downloads)自行下载。本文主要以macOS为主进行讲解。

安装完成后配置用户名与邮箱：

```shell
### 如果想设置为全局生效，添加 --global 参数
$ git config --global user.name "你的用户名"
$ git config --global user.email "你的邮箱"
```

### SSH Key设置

#### 检查现有的ssh key

```shell
$ cd ~/.ssh
```

如果显示“No such file or directory”，说明你是第一次使用 git。

#### 生成新的SSH Key

输入下面的代码，就可以生成新的key文件，我们只需要默认设置就好，所以当需要输入文件名的时候，回车就好。

```shell
$ ssh-keygen -t rsa -C "邮件地址@youremail.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/Users/your_user_directory/.ssh/id_rsa):#密码不会回显
```

然后系统会要你输入加密串（Passphrase）：

```shell
Enter passphrase (empty for no passphrase):<输入加密串>
Enter same passphrase again:<再次输入加密串>
```

最后看到ssh key success，就成功设置ssh key了.

#### 添加SSH Key到GitHub

在本机设置SSH Key之后，需要添加到GitHub上，以完成SSH链接的设置。

在GitHub的主页上点击设置按钮： github account setting，选择SSH Keys项，把id_rsa.pub中的内容粘贴进去，然后点击Add Key按钮即可：

PS：如果需要配置多个GitHub账号，可以参看这个[多个github帐号的SSH key切换](http://ju.outofmemory.cn/entry/16775)，不过需要提醒一下的是，如果你只是通过这篇文章中所述配置了Host，那么你多个账号下面的提交用户会是一个人，所以需要通过命令`git config --global --unset user.email`删除用户账户设置，在每一个repo下面使用`git config --local user.email '你的github邮箱@mail.com'`命令单独设置用户账户信息

#### 测试

可以输入下面的命令，看看设置是否成功，git@github.com的部分不要修改：

```shell
$ ssh -T git@github.com
```

```shell
Hi <em>username</em>! You've successfully authenticated, but GitHub does not provide shell access.
```

如果显示以上内容，则说明配置成功。此时你可以成功连接GitHub了。

## 安装Jekyll

### Jekyll 究竟是什么？

Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过 Markdown （或者 Textile） 以及 Liquid 转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是**完全免费**的。

### 事先准备

安装 Jekyll 相当简单，但是你得先做好一些准备工作 开始前你需要确保你在系统里已经有如下配置。

- [Ruby](http://www.ruby-lang.org/en/downloads/)
- [RubyGems](http://rubygems.org/pages/download)
- Linux, Unix, or Mac OS X

### 借助RubyGems安装Jekyll

安装 Jekyll 的最好方式就是使用 [RubyGems](http://docs.rubygems.org/read/chapter/3). 你只需要打开终端输入以下命令就可以安装了：

```shell
$ gem install jekyll
```

所有的 Jekyll 的 gem 依赖包都会被自动安装，所以你完全不用去担心。

##### 安装 Xcode Command-Line Tools

如果你是Mac用户，你就需要安装 Xcode 和 Command-Line Tools了。下载方式`Preferences → Downloads → Components`。

### 附加功能

根据每个人使用方式的不同，Jekyll 还支持你安装一些附加功能。包括了对 LaTex 的支持，以及使用动态内容渲染引擎。 查看 [the extras page](https://www.jekyll.com.cn/docs/extras/)获得更多信息。

#### What`s more

你已经安装了所有需要的东西了，开始玩转 Jekyll 博客吧！具体的Jekyll的使用可以参考[Jekyll官方文档](https://www.jekyll.com.cn/docs/home/)。

当然在互联网上有大量的Jekyll模板可供选择，只需要做一些简单的修改就可以使用了，还可以在原有模板的基础上进行个性化的定制，前提是你要对[Liquid](http://docs.shopify.com/themes/liquid-basics)语言有一些了解。在[Jekyll Theme](http://jekyllthemes.org/)上就有丰富的模板。

## 开始搭建博客

与 GitHub 建立好链接之后，就可以方便的使用它提供的 Pages 服务，GitHub Pages 分两种，一种是你的 GitHub 用户名建立的 USER_NAME.github.io 这样的用户&组织页（站），另一种是依附项目的 Pages。

想建立个人博客是用的第一种，形如 https://belikovvv.github.io/ 这样的可访问的站，每个用户名下面只能建立一个，且仓库名形如USER_NAME.github.io。

#### 常用Git命令

```shell
$ git clone [url]	# 下载一个项目和它的整个代码历史
$ git init			# 在当前目录新建一个Git代码库
$ git add .			# 添加当前目录的所有文件到暂存区
$ git commit -m [message]		# 提交暂存区到仓库区
$ git remote add origin https://github.com/USER_NAME/repository_name.git
$ git push -u origin master		# 上传本地指定分支到远程仓库
```

上传成功之后就可以访问自己的博客了。除此之外你还可以根据[Jekyll官方文档](https://www.jekyll.com.cn/docs/home/)对博客进行个性化定制，此处不再赘述。同时很多模板都会详细的写明配置信息，只需要认真阅读模板的说明信息即可。

#### 推送文章更新

可以通过命令行更新，同时也推荐使用GitHub Desktop进行更新，后者的可视化更强一些，尤其针对那些GitHub被墙的地区，只需要配置好全局代理就可以使用。

#### 404页面

GitHub Pages有提供制作404页面的指引：[Custom 404 Pages](https://help.github.com/articles/custom-404-Pages)。

直接在根目录下创建自己的 404.html 或者 404.md 就可以。但是自定义404页面仅对绑定顶级域名的项目才起作用。推荐使用[腾讯公益404](http://www.qq.com/404)。

#### 图床

在博文中难免会用到图片，把图片以文件形式上传难免会影响博客的加载速度，建议使用图床。

推荐使用[七牛图床](https://www.qiniu.com/)或[极简图床](http://jiantuku.com/)。

#### 评论功能

[Disqus](http://disqus.com/)是一个社会化的评论解决方案，调用它的接口非常简单，在自己的页面加载他的一段JS代码即可，如果别人注册了Disqus，那么就可以方便的留言，交流，一处登录，处处方便，而且Disqus也提供了一些spam等策略，并且可以和一些现有的博客系统很好的转换对接。

由于种种原因，Disqus在国内暂时不可用。有包括[来必力](https://livere.com/)等在内的一些其他插件可供选择。

#### 站点统计

这里介绍的站点统计是[Google  Analytics](http://www.google.cn/analytics/)，analytics的使用十分简单，同样的原理，利用注入脚本来实现流量统计的外挂，统计功能十分强大，谁用谁知道。

当然[百度统计](https://tongji.baidu.com/)功能也很强大，比较适合在国内使用。



## 相关问题

对于`Deprecation: The 'gems' configuration option has been renamed to 'plugins'. Please update your config file accordingly.`这个警告，解决办法是打开`_config.yml`将`gems:`改成`plugins:`

 