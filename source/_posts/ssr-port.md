---
title: SSR的1080端口被占用
date: 2019-08-30 12:24:06
tags: [tool]
---

一劳永逸的方法是修改使用的端口，到ShadowsocksR的根目录下，打开gui-config.json配置文件，找到配置项（"localPort" : 1080）,将值修改为其他端口，并重启程序。