---
title: git
date: 2020-03-15 23:22:20
tags: [tool]
---

### Git 分支开发规范指南

分支类型
+	master 主分支，用于部署生产环境的分支，需确保主分支的稳定性
+	develop 开发分支
+	feature  开发新功能时，以`develop`为基础创建feature分支
	+	命名规范：feature/user_module
+	release 预上线分支
+	hotfix 修复分支


日志规范（commit messages）
编写良好的Commit messages可以达到3个重要的目的：

+	加快review的流程
+	帮助我们编写良好的版本发布日志
+	让之后的维护者了解代码里出现特定变化和feature被添加的原因

参考规范：[Angular Git Commit Guidelines](https://github.com/angular/angular.js/blob/master/DEVELOPERS.md#-git-commit-guidelines)

```
	<type>: <subject>
	<BLANK LINE>
	<body>
	<BLANK LINE>
	<footer>
type: 本次 commit 的类型，诸如 bugfix docs style 等
scope: 本次 commit 波及的范围
subject: 简明扼要的阐述下本次 commit 的主旨，在原文中特意强调了几点 1. 使用祈使句，是不是很熟悉又陌生的一个词，来传送门在此 祈使句 2. 首字母不要大写 3. 结尾无需添加标点
body: 同样使用祈使句，在主体内容中我们需要把本次 commit 详细的描述一下，比如此次变更的动机，如需换行，则使用 |
footer: 描述下与之关联的 issue 或 break change，详见案例
```

Type的类别
+	feat: 添加新特性
+	fix: 修复bug
+	docs: 仅仅修改了文档
+	style: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑
+	refactor: 代码重构，没有加新功能或者修复bug
+	perf: 增加代码进行性能测试
+	test: 增加测试用例
+	chore: 改变构建流程、或者增加依赖库、工具等


> [后端必备的 Git 分支开发规范指南](https://blog.csdn.net/weixin_38405253/article/details/102830187)