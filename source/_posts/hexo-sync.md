---
title: 多PC同步hexo
date: 2021-05-16 13:03:07
tags: [博客搭建]
---

# 在不同机器同步hexo

hexo项目生成的文档占据了主分支，还需要另外一个分支存放源码

1. Github上新建分支
2. clone指定分支
```
	git clone -b 指定分支 https://github.com/用户名/仓库.git
```
3. 删除“.git”（这是一个默认隐藏文件）之外的所有文件
4. 移动项目文件到这里面，更新
```
	$ git add .
	$ git commit -m "test"
	$ git push origin dir
```

现在，存在两个分支，主分支存放文章，第二分支存放源码


unable to access 'https://github.com/XX/XX.git: OpenSSL SSL_read: Connection was reset, errno 10054

```
-- 关闭证书认证
git config --global http.sslVerify "false"
```



【hexo】`TypeError [ERR_INVALID_ARG_TYPE]: The mode argument must be integer. Received an instance of Object`

1. nodejs版本过高，降级即可
[nvm 包管理器](https://github.com/coreybutler/nvm-windows/releases)

2. 切换node版本
```
nvm install 10.16.0
nvm uninstall 10.16.0


nvm use 10.16.0//临时版本
nvm alias default 10.16.0//永久版本

```
