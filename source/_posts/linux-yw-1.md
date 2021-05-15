---
title: linux 运维（一） kdevtmpfsi挖矿病毒处理
date: 2020-01-02 23:14:43
tags: [linux]
---

### CPU 100%的处理——基于kdevtmpfsi挖矿病毒总结排查思路

1. 查看占资源最多的进程
`netstat -natp `
2. 尝试使用kill
`kill 19999 -9`
3. kill进程无效，不断重启；判定存在crontab定时任务；排查定时任务
`crontab -l `
4. 终止守护线程，终止定时任务；删除对应文件
```
systemctl status 23437

ps -aux | grep kinsing

ps -aux | grep kdevtmpfsi

kill  -9   23437

kill  -9   18534

cd  /tmp

ls

 rm -rf kdevtmpfsi 

rm -rf /var/tmp/kinsing  记得这个守护进程的文件也要删掉，找不到的话，也可以用这个命令

find / -name kdevtmpfsi

find / -name kinsing 
```

### 存在挖矿病毒入侵的风险点
1. redis 存在漏洞
2. docker

### 应对方法
1. 白名单机制
2. 不使用常用的端口
3. 不直接使用对外端口，使用代理

# 参考资源
>   [处理kdevtmpfsi挖矿病毒](https://blog.csdn.net/u014589116/article/details/103705690)