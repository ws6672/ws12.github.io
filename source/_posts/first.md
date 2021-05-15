---
title: 博客搭建（一）Github Page
date: 2019-08-18 17:55:10
tags: 博客搭建
---
这是博客搭建的开端，通过这个系列的文章你可以了解到如何在没有站点以及域名的情况下，通过Github Page快速搭建属于自己的博客。  
  
  
  
## 博客搭建（一）Github Page
### 什么是Github Page
github page是面向用户、组织和项目开放的公共静态页面搭建托管服务，站点可以被免费托管在 Github 上，你可以选择使用 Github Pages 默认提供的域名 github.io 或者自定义域名来发布站点，更便利地是你直接从你的GitHub存储库托管。只需编辑和推送你的blog，并且是实时更改。


### 创建存储库
转到GitHub并创建一个格式为`username .github.io` 的新存储库，其中username是GitHub上的用户名（或组织名称）。
如果存储库的第一部分与您的用户名不完全匹配，则无法正常工作，因此请务必正确使用。  
![](/image/19819/name1.png)

  
### Linux下安装GitHub
``` 
--安装git
$ yum -y install git git-core git-doc

-- 配置用户信息
git config --global user.name "ws6672"
git config --global user.email "ws6672s@gmail.com"

# git clone https://github.com/ws6672/ws6672.github.io
# cd ws6672.github.io/
# echo "hello wsz" > index.html

-- 提交修改
# git add --all
# git commit -m"test"
# git push -u origin master
Username for 'https://github.com': ws6672
Password for 'https://ws6672@github.com': 
```


### 自定义域名
在项目中添加CNAME的文件，这样我们就可以对项目进行自定义配置，这时候选择setting会发现多了许多选项。

在自定义域名处填写自己的域名
![](/image/19819/name2.png)

免费域名我是在freenom获取的，很早之前就申请了，当时没有做笔记，请自行百度。
由于freenom是国外的网站，所以在国内使用时，它的域名解析速度不快，我用的是dnspod代理域名解析。

+ 域名申请:[dnspod](https://www.dnspod.cn/)
+ 添加域名
![](/image/19819/b1.png)
+ 自动扫描，获取记录值
![](/image/19819/b2.png)
+ 在freenom配置代理的解析服务器，将记录值配置在feednom中
![](/image/19819/b3.png)
+ 在DNSPod中配置网站所在服务器，由于本文是用Github Page搭建博客，所以直接 ping username.github.io 可以获取对应的IP。
![](/image/19819/b4.png)
+ 修改后等待最多72小时

### Jekyll 

使用Jekyll，您可以使用漂亮的Markdown语法进行博客，而无需处理任何数据库。
您可以通过编辑站点的_config.yml文件将Gekyll主题添加到GitHub页面站点。  


#### 什么是 Jekyll
Jekyll 是一个简单的博客形态的静态站点生产机器。它有一个模版目录，其中包含原始文本格式的文档，通过一个转换器（如 Markdown）和我们的 Liquid 渲染器转化成一个完整的可发布的静态网站，你可以发布在任何你喜爱的服务器上。Jekyll 也可以运行在 GitHub Page 上，也就是说，你可以使用 GitHub 的服务来搭建你的项目页面、博客或者网站，而且是完全免费的。——官网  


#### jekyll 安装
  
  
```
# 安装bundler，bundler通过gemfile文件来管理gem包
gem install  bundler

# 创建一个新的Jekyll项目，并命名为myblog
jekyll new myblog

# 进入myblog目录
cd myblog

# 在Jekyll自带的服务器上预览你的项目，默认的运行地址为http://localhost:4000
# bundle exec 表示在当前项目依赖的上下文环境中执行命令 jekyll serve
bundle exec jekyll serve
```

#### 目录结构
![](/image/19819/name3.png)  



#### jekyll 基本用法  

```
$ jekyll build
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中。

$ jekyll build --destination <destination>
# => 当前文件夹中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --source <source> --destination <destination>
# => 指定源文件夹<source>中的内容将会生成到目标文件夹<destination>中。

$ jekyll build --watch
# => 当前文件夹中的内容将会生成到 ./_site 文件夹中，
#    查看改变，并且自动再生成。
```

  
  
  
启动后，默认可以通过http://localhost:4000访问首页：
![](/image/19819/name4.png)  