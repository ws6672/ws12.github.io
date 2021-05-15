---
title: Jenkins实战（一）初始化使用
date: 2020-03-25 16:13:51
tags: [CI,tool]
---


### 一、Jenkins配置


##### 命令配置
1.通过docker安装
```
--要使用最新的LTS： 
	docker pull jenkins/jenkins

--要使用最新的每周： 
	docker pull jenkins/jenkins
```

2. 配置环境变量
```
 vim /etc/profile

jenkins_home=$PATH:/var/jenkins_home
export jenkins_home

source /etc/profile
```

3. 启动docker镜像
```
-- 以下命令同时映射 8080、50000端口，适合单词使用
docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins

-- 将docker中的文件映射到真实路径下
sudo docker run -d -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home jenkins/jenkins

```

##### 界面配置

1.登陆密码
`sudo vim /var/jenkins_home/secrets/initialAdminPassword`


2.遇到 `jenkins实例似乎已离线`

```
# 查看ip
ifconfig -a

修改镜像位置，
	http://localhost:8080/pluginManager/advanced

修改站点（update site）：
	https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json
	http://mirror.xmission.com/jenkins/updates/update-center.json
	http://ftp.tsukuba.wide.ad.jp/software/jenkins/updates/current/update-center.json
	http://updates.jenkins.io/update-center.json 


# 查看日志
sudo docker logs d5eb
# Jenkins 的异常，镜像无效
java.net.UnknownHostException: updates.jenkins.io

```


##### 遇到的几个问题
***XXX 不在 sudoers 文件中***

用户没有提权的权限，需要配置：
```
	vim /etc/sudoers
	-- XX可以执行任何用户的任何命令
	XX  ALL=(ALL)       AL
	:wq! 强制保存
```

***docker pull timeout***
修改国内镜像源
```
	vim /etc/docker/daemon.json

	{
	"registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
	}
	:wq

	sudo service docker restart
```

***docker删除镜像失败***
删除镜像需要停止、删除容器，
-- 删除镜像,强制删除
`docker rmi -f <image id>`

***sudo: XXX：找不到命令***
需要为普通用户添加sudo权限，wheel用户组默认有sudo权限
```
useradd jenkins
passwd jenkins
usermod -aG wheel jenkins
```

***Permission denied:cannot touch '/user/jenkins_home/copy_reference_file.log'***
宿主机中的文件是缺省生成的，默认所属root，所以其它用户无法操作，即便拥有sudo权限。需要进入宿主机修改文件权限

```
-- 停止
sudo docker container stop 0e15
-- 启动
sudo docker run -dti --privileged -p 8080:8080 -p 50000:50000 jenkins/jenkins

-- 修改文件所属权限
	-- 解决Docker容器没有权限写入宿主机目录
	-- bash: sudo: command not found
		-- 以管理员身份进入宿主机
		sudo docker exec -ti --user root b41e /bin/bash
		-- 安装sudo
		apt-get update
		apt-get install sudo
		exit
		
-- 普通用户登录
sudo docker exec -ti 6b21 /bin/bash
sudo chown -R 1000:1000 /var/jenkins_home
sudo chmod -R 777 jenkins_home
ls -al
sudo docker run -d -p 8080:8080 -p 50000:50000 -v /var/jenkins_home:/var/jenkins_home jenkins/jenkins

```



### 二、创建第一个Pipeline
Jenkins Pipeline（或简称为 "Pipeline"）是一套插件，将持续交付的实现和实施集成到 Jenkins 中。
持续交付 Pipeline 自动化的表达了这样一种流程：将基于版本控制管理的软件持续的交付到您的用户和消费者手中。Pipeline被定义在文件`Jenkinsfile`中。


Java
```
Jenkinsfile (Declarative Pipeline)
pipeline {
    agent { docker 'maven:3.3.3' }
    stages {
        stage('build') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

### 三、执行多个步骤（Step）
Pipelines 由多个步骤（step）组成，允许你构建、测试和部署应用。 Jenkins Pipeline 允许您使用一种简单的方式组合多个步骤， 以帮助您实现多种类型的自动化构建过程。

可以把“步骤（step）”看作一个执行单一动作的单一的命令。 当一个步骤运行成功时继续运行下一个步骤。 当任何一个步骤执行失败时，Pipeline 的执行结果也为失败。

当所有的步骤都执行完成并且为成功时，Pipeline 的执行结果为成功。


Jenkinsfile (定义Pipeline)，如果是Linux类型的系统，则用`sh`表示调用命令行命令；如果是windows系统，则使用`bat`。Jenkins Pipeline 提供了很多的步骤（step），这些步骤可以相互组合嵌套。

```
pipeline {
    agent any
    stages {
        stage('Deploy') {
            steps {
				// 即便执行失败，也会重复执行三次
                retry(3) {
                    sh './flakey-deploy.sh'
                }
				// 等待 health-check.sh 脚本最长执行3分钟
                timeout(time: 3, unit: 'MINUTES') {
                    sh './health-check.sh'
                }
            }
        }
		// 最后执行的操作
		 post {
			always {
				echo 'This will always run'
			}
			success {
				echo 'This will run only if successful'
			}
			failure {
				echo 'This will run only if failed'
			}
			unstable {
				echo 'This will run only if the run was marked as unstable'
			}
			changed {
				echo 'This will run only if the state of the Pipeline has changed'
				echo 'For example, if the Pipeline was previously failing but is now successful'
			}
		}
    }
}
```

### 四、定义执行环境
`agent` 指令告诉Jenkins在哪里以及如何执行Pipeline或者Pipeline子集

在执行引擎中，agent 指令会引起以下操作的执行：

+	所有在块block中的步骤steps会被Jenkins保存在一个执行队列中。 一旦一个执行器 executor 是可以利用的，这些步骤将会开始执行。
+	一个工作空间 workspace 将会被分配， 工作空间中会包含来自远程仓库的文件和一些用于Pipeline的工作文件
 
我的Jenkins是通过Docker运行的，所以执行环境也定义为Docker。当然，也有[更多的代理方式](https://jenkins.io/doc/book/pipeline/syntax#agent)可以使用。

```
pipeline {
    agent {
        docker { image 'node:7-alpine' }
    }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
            }
        }
    }
}
```

***使用环境变量***

```
pipeline {
    agent any

    environment {
        DISABLE_AUTH = 'true'
        DB_ENGINE    = 'sqlite'
    }

    stages {
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
}
```
### 五、测试与测试结果
***记录测试结果***
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh './gradlew build'
            }
        }
        stage('Test') {
            steps {
                sh './gradlew check'
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'build/libs/**/*.jar', fingerprint: true
            junit 'build/reports/**/*.xml'
			deleteDir() // 清理输出文件
        }
    }
}
```

***电子邮件通知***
```
post {
    failure {
        mail to: 'team@example.com',
             subject: "Failed Pipeline: ${currentBuild.fullDisplayName}",
             body: "Something is wrong with ${env.BUILD_URL}"
    }
}
```

### 六、部署
```
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building'
            }
        }
        stage('Test') {
            steps {
                echo 'Testing'
            }
        }
		// 人工确认满足部署条件
		stage('Sanity check') {
            steps {
                input "Does the staging environment look ok?"
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying'
            }
        }
    }
}
```




相关文章：
> [创建您的第一个Pipeline](https://jenkins.io/zh/doc/pipeline/tour/hello-world/)
Jenkinsfile 的配置：Blue Ocean、经典UI、源码管理系统