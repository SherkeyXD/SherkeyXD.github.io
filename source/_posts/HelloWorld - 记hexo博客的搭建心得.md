---
title: HelloWorld - 记hexo博客的搭建心得
date: 1919-08-10 11:45:14
tags: blog
category: Programming
---
既然你现在已经看到了这篇文章，那么说明我的博客搭建至少不算失败。hexo博客的搭建并不算麻烦，但也需要够的耐心，和一点点的node使用经验与yml语法知识

## 安装node及npm

对于已经熟悉nodejs的程序员来讲，这并不是什么新鲜事。但对于只是想写点文章记录自己想法的人来说，就有点陌生了。作为一个程序员，你应该深知版本控制的重要性。因此，我们将使用nvm或者n来安装nodejs以及npm

### Windows(with nvm-windows)

[nvm-windows](https://github.com/coreybutler/nvm-windows)算是windows上为数不多的版本控制选择了

首先，从[github release](https://github.com/coreybutler/nvm-windows/releases)，下载并打开最新版本的nvm-setup.exe，一路安装

然后设置镜像，这里以阿里的镜像站为例
```bash
nvm npm_mirror https://npmmirror.com/mirrors/npm/
nvm node_mirror https://npmmirror.com/mirrors/node/
```
放心，这里的npm只是下载npm的镜像，并不是安装npm包的镜像，所以不用担心npm源出问题

然后，用`nvm list available`来列出目前的可用版本，然后用`nvm install 版本号`来安装node及npm。建议安装lts版本，这样不容易有奇怪bug

最后用`nvm use 版本`来设置node版本，设置完成后后可以用`node -v`确认

### Linux(with n)

如果你想在Linux上使用nvm，我不会有什么意见，毕竟爱折腾自己的人并不少。但既然你选择了Linux，那么我还是推荐你使用n来进行node的版本控制，这要简单易用的多。

首先安装npm，这里以ubuntu/debian为例：
```bash
sudo apt-get update && sudo apt-get install node npm
```
然后，npm切换至阿里的源，并安装n
```bash
npm config set registry https://registry.npmmirror.com/
npm install -g n
```
> 如果出现 npm WARN config global --global, --local are deprecated. Use --location ，不用担心，升级npm至最新版本即可解决

**这里不用cnpm的原因是因为cnpm不认package-lock.json，可能会有版本上的问题**

然后，使用清华源安装node，并确认node版本
```bash
export NODE_MIRROR=https://mirrors.tuna.tsinghua.edu.cn/nodejs-release/
sudo n lts
sudo n
```

设置完成后后可以用`node -v`确认node版本

## 选择hexo主题并配置

你可以在[github topics](https://github.com/topics/hexo-theme)中寻找自己想用的主题。各个主题会有不同的配置文件说明，这个直接跟着文档走就行，这里只讲一些通用的修改

首先打开_config.yml，修改如下内容

````yml
# Site
title: 你的博客网页标题
subtitle: 这个有些主题要写
description: 简介
keywords: 会被搜索引擎收录，以逗号隔开
author: 你的名字
language: zh
timezone: 'Asia/Shanghai' 

# URL
## Set your site url here. For example, if you use GitHub Page, set url as 'https://username.github.io/project'
url: 你的博客网址
````

其他的可以先不用动，或者跟着主题走

## 博客文章写作

可以参考[写作 | Hexo](https://hexo.io/zh-cn/docs/writing)

写新文章时，直接在博客根目录下执行

```bash
hexo new post 文章名字
```

## 博客发布

在博客根目录下执行如下命令

```bash
hexo clean && hexo generate && hexo server
```

然后 hexo会提示你博客已经在localhost上打开，你可以看看效果

要部署到github，还需执行 `hexo deploy`，但在此之前还得做些配置

首先，在博客根目录下执行

```
npm install hexo-deployer-git --save
```

然后，在_config.yml的最后部分

```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repo: 你的github仓库地址，https与ssh都可以
  branch: 分支名字，建议为 gh-pages
```

由于hexo支持对命令的缩写，故一般的部署流程可以简写为

```
hexo clean && hexo g -d
```

等待部署完成就可以了

## 一些小技巧

### 短链接的设置

我选用的主题没有短链接修改，我又嫌hexo默认的短链接太丑，于是找到了hexo-abbrlink这个包

在hexo博客根目录下，执行

```bash
npm install hexo-abbrlink --save
```

然后找到_config.yml 中 permalink 部分，把原先的删掉换上新的（绿色部分）

```diff
-permalink: :year/:month/:day/:title/
-permalink_defaults:
-pretty_urls:
-  trailing_index: true # Set to false to remove trailing 'index.html' from permalinks
-  trailing_html: true # Set to false to remove trailing '.html' from permalinks
+# permalink: :year/:month/:day/:title/
+permalink: posts/:abbrlink.html  # posts 是自定义的前缀
+abbrlink:
+    alg: crc16   #算法： crc16(default) and crc32
+    rep: hex     #进制： dec(default) and hex
```

效果大概如下

```
crc16 & hex
https://username.github.io/posts/66c8.html
crc16 & dec
https://username.github.io/posts/65535.html
crc32 & hex
https://username.github.io/posts/8ddf18fb.html
crc32 & dec
https://username.github.io/posts/1690090958.html
```

### 快速渲染修改

通过hexo-browsersync插件，我们可以在hexo server打开的情况下将对博文的修改同步至server上，算是非常方便了

在博客根目录下执行

```
npm install hexo-browsersync
```

下次hexo server启动时就可以直接修改了

### 创建新文章时自动打开文本编辑器

在博客根目录下新建名为scripts的文件夹，在里面新建一个js文件（文件名随意），内容如下

```js
var spawn = require('child_process').exec;
hexo.on('new', function(data){
  spawn('start  "这个地方填文本编辑器路径" ' + data.path);
});
```

### yml语法的学习

在配置hexo博客的过程中我们或或多少会涉及到yaml语言的使用，你可以在[Learn yaml in Y Minutes (learnxinyminutes.com)](https://learnxinyminutes.com/docs/yaml/)上找到基本的yaml语法知识

这里提一下有个需要注意的点：yml中字符串并不一定要用引号括起来，但是个人建议还是要这么做，避免奇怪的问题