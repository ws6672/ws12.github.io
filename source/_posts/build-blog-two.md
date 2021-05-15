---
title: 博客搭建（二）hexo的使用
date: 2019-08-19 22:46:31
tags: 博客搭建
---
新版的Jekyll对于Github Page不太兼容，所以我改用了Hexo。

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

注：以下的步骤都需要先在cmd或者Linux的窗口中CD到项目目录下
# Hexo  
  
  
## 安装
安装 Hexo 相当简单。然而在安装前，您必须检查电脑中是否已安装下列应用程序：
+ Node.js (Should be at least nodejs 6.9)
+ Git

hexo安装步骤如下：
```
npm install hexo-cli -g
hexo init blog
cd blog
npm install
hexo server
```
---


## 自定义主题
创建 Hexo 主题非常容易，您只要在 themes 文件夹内，新增一个任意名称的文件夹，并修改 _config.yml 内的 theme 设定，即可切换主题。一个主题可能会有以下的结构：
```
├── _config.yml 主题的配置文件。修改时会自动更新，无需重启服务器。
├── languages 语言文件夹
├── layout 布局
├── scripts 脚本
└── source 存放文档以及资源
```

### 主题的发布
当您完成主题后，可以考虑将它发布到 主题列表，让更多人能够使用您的主题。在发布前建议先进行 主题单元测试，确保每一项功能都能正常使用。发布主题的步骤和 更新文档 非常类似。

+ Fork hexojs/site

+ 把库（repository）复制到电脑上，并安装所依赖的插件。
```
$ git clone https://github.com/<username>/site.git
$ cd site
$ npm install
```

+ 编辑 source/_data/themes.yml，在文件中新增您的主题，例如：
```
- name: landscape
  description: A brand new default theme for Hexo.
  link: https://github.com/hexojs/hexo-theme-landscape
  preview: http://hexo.io/hexo-theme-landscape
  tags:
    - official
    - responsive
    - widget
    - two_column
    - one_column
```
+ 在 source/themes/screenshots 新增同名的截图档案，图片必须为 800x500 的 PNG 文件。
+ 推送（push）分支。
+ 建立一个新的合并申请（pull request）并描述改动。

而我用的主题是别人发布出来的主题BlueLake，地址在下面，推荐使用。

### 安装主题和渲染：
```
$ git clone https://github.com/chaooo/hexo-theme-BlueLake.git themes/BlueLake
$ npm install hexo-renderer-jade@0.3.0 --save
$ npm install hexo-renderer-stylus --save
```

### 启用
在Hexo配置文件（hexo/_config.yml）中把主题设置修改为BlueLake。
theme: BlueLake

### 项目文件结构
![](/image/19819/name6.png)


## 写作
```
$ hexo new [layout] <title>
```

您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。

### 布局（Layout）
Hexo 有三种默认布局：post、page 和draft。在创建者三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。

```
布局	路径
post	source/_posts
page	source
draft	source/_drafts
```


## 部署
本地编写完成后，要同步Github Pages，运行以下三个命令即可：
```
`hexo g`生成静态页面文件
`hexo s`启动本地服务器，进行查看，直接打开 https://127.0.0.1:4000  查看
`hexo d`部署静态页面到服务器（如 github ）
```

### 使用hexo d命令前需要在_config.yml文件中进行配置
```
deploy:
  type: git
  repo: 仓库网址 #这里的网址填你自己的
  branch: gh-pages 
```

### hexo d后 ERROR Deployer not found: git 
```npm install --save hexo-deployer-git```

## 运行测试

输入命令 hexo s，启动服务器，然后访问https://127.0.0.1:4000，可以看到类似如下的页面：
![](/image/19819/name5.png)
