---
title: Jenkins实战（二）Pipeline
date: 2020-03-25 16:54:15
tags: [CI,tool]
---


### 二、Pipeline
`Jenkins Pipeline`（或简称为 "Pipeline"）是一套插件，将持续交付的实现和实施集成到 Jenkins 中。Pipeline(通过基于版本控制管理的软件（git、svn）实现了持续交付。本质上，Jenkins 是一个自动化引擎，它支持许多自动模式，这个自动化流程被定义在 `Jenkinsfile` 中。

***特点***
+	自动化
+	持续交付

***优点***
+	【自动化】自动地为所有分支创建流水线构建过程并拉取请求。
+	代码复查/迭代
+	审计跟踪
+	源代码可以被项目的多个成员查看和编辑。

***流程***
![Jenkins_sz](/image/jenkins/Jenkins_sz_1.png)
分为多种不同类型的自动化构建过程，可以把“步骤（step）”放入一个执行单一动作的单一的命令。当一个步骤运行成功时继续运行下一个步骤。当任何一个步骤执行失败时，Pipeline的执行结果也为失败。当所有的步骤都执行完成并为成功时，Pipeline的执行结果为成功。


+	SCM插件（需要安装pipeline和git插件）：它用于管理jenkinsfile，即用于管理Pipeline。
+	构建
+	测试
+	部署

##### 1.创建Jenkinsfile
`Jenkinsfile` 是一个描述`Pipeline`流程的文本文件,能使用两种语法进行编写 - 声明式和脚本化。



***Jenkinsfile 声明式模板*** 
```
// 分成三部分，构建、测试、部署
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Building..'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing..'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying....'
            }
        }
    }
}
```
1.agent指令是必需的，它指示 Jenkins 为流水线分配一个执行器和工作区。没有 agent 指令的话，声明式流水线不仅无效，它也不可能完成任何工作！默认情况下，agent 指令确保源代码仓库被检出并在后续阶段的步骤中可被使用。
2.stages 指令指示 Jenkins 要执行什么类型的操作，Build、Test、Deploy
3.steps 指令指示 Jenkins 在哪个阶段执行

***脚本式语法***
```
node {
    checkout scm 
    /* .. snip .. */
}
```

***创建方式***
+	通过 Blue Ocean 
+	通过经典 UI 
+	在源码管理系统中定义（Git、SVN）

##### 2.构建
对于许多项目来说，流水线“工作”的开始就是“构建”阶段。通常流水线的这个阶段包括源代码的组装、编译或打包。Jenkinsfile 文件不能替代现有的构建工具，如 GNU/Make、Maven、Gradle 等，而应视其为一个将项目的开发生命周期的多个阶段（构建、测试、部署等）绑定在一起的粘合层。


假设系统为linux，通过`sh`步骤调用`make`;如果是windows系统，需要修改为bat
```
//	sh 步骤调用 make 命令，只有命令返回的状态码为零时才会继续。任何非零的返回码都将使流水线失败。
//	archiveArtifacts 捕获符合模式（``**/target/*.jar``）匹配的交付件并将其保存到 Jenkins master 节点以供后续获取。

Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                sh 'make' 
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true 
            }
        }
    }
}
```

##### 3.测试
运行自动化测试是任何成功的持续交付过程的重要组成部分，它自带了许多插件可以用于调用常见的测试软件。当测试失败时，Jenkins会 记录这些失败以供汇总、 并为web UI的可视化提供基础数据。

以`JUnit 插件`提供的 junit 步骤为例：

```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent any

    stages {
        stage('Test') {
            steps {
                /* `make check` 在测试失败后返回非零的退出码；
                * 使用 `true` 允许流水线继续进行
                */
                sh 'make check || true' 
                junit '**/target/*.xml' 
            }
        }
    }
}
```


