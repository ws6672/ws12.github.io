---
title: maven迁移仓库
date: 2019-08-20 22:20:12
tags: [tool]
---
# maven迁移仓库
*maven的默认路径是C:\Users\zws\.m2，修改下面的settings.xml配置文件。*
```
  <!-- localRepository
   | The path to the local repository maven will use to store artifacts.
   |
   | Default: ${user.home}/.m2/repository
  <localRepository>/path/to/local/repo</localRepository>
  -->
  <localRepository>E:/wsz6672/bdqy/repository/repository（类似的自定义仓库路径）</localRepository>
```