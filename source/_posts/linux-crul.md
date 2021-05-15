---
title: linux学习指南（二）crul
date: 2019-09-10 22:57:37
tags: linux
---

# 一、什么是curl 
curl是一个非常实用的、用来与服务器之间传输数据的工具；支持的协议包括 (DICT, FILE, FTP, FTPS, GOPHER, HTTP, HTTPS, IMAP, IMAPS, LDAP, LDAPS, POP3, POP3S, RTMP, RTSP, SCP, SFTP, SMTP, SMTPS, TELNET and TFTP)，curl设计为无用户交互下完成工作。


curl提供了一大堆非常有用的功能，包括代理访问、用户认证、ftp上传下载、HTTP POST、SSL连接、cookie支持、断点续传...。


# 二、语法

语法如下：
```
curl [options] [URL...]
```

### 1. 请求头

**添加**

curl -H "name:value"
curl --header "name:value"

**删除**

curl -H "name:"
curl --header "name:"

**设置【User-Agent】**

curl -A "string"
curl --user-agent "string"

**设置访问来源**

有一些网站有防盗链之类的拦截，需要从指定页面跳转过去，才能够访问。即，需要告知HTTP服务是从哪个页面进入该页面的。
curl -e <URL>
curl --referer <URL>


### 2. 响应头

**输出响应头**

用于HTTP服务时，获取页面的http头；用于FTP/FILE时，将会获取文件大小、最后修改时间；
curl -I
curl --head

**输出响应头及内容**

-i
--include

**转储http响应头到指定文件**

-D <file>
--dump-header <file>


### 3. cookie

**发送数据到服务器**

-b name=data
--cookie name=data


**保存到指定位置**

-c filename
--cookie-jar file name

**清理session【重启浏览器】**

-j
--junk-session-cookies





