---
title: springmvc（一）定时任务
date: 2019-10-26 16:38:49
tags: [javaweb]
---

在实际使用中，定时任务是很有用的，通过定时任务可以在指定时间进行爬虫、数据处理、邮件提醒等。
 

### 1.通过注解使用定时任务
 
```
	@Component //实例化
	@EnableScheduling #启用定时任务
	public class SzZhsjptDdrbbZhsjb {
		// 定时方法标注
		//    @Scheduled(cron="0 25 1 * * ? ")
		@Scheduled(cron = "0 */10 * * * ?") # 配置定时器，需要配置在方法上
		public void start()throws InterruptedException {
			SzZhsjptDdrbbZhsjb zhsj = new SzZhsjptDdrbbZhsjb();//启动火狐
			zhsj.execute();
			Thread.sleep(3000);
			zhsj.quit();
		}
	}
```

### 2.定时任务自动扫描

在springmvc.xml配置task的命名空间
	```
	xmlns:task="http://www.springframework.org/schema/task"   
	http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.1.xsd  
	```

启用注解驱动
	`<task:annotation-driven scheduler="myScheduler"/>`
配置定时任务的线程池
	`<task:scheduler id="myScheduler" pool-size="5"/>  `

```
	@Component //实例化
	//@EnableScheduling #扫描定时器
	public class SzZhsjptDdrbbZhsjb {
		// 定时方法标注
		//    @Scheduled(cron="0 25 1 * * ? ")
		@Scheduled(cron = "0 */10 * * * ?") # 配置定时器，需要配置在方法上
		public void start()throws InterruptedException {
			SzZhsjptDdrbbZhsjb zhsj = new SzZhsjptDdrbbZhsjb();//启动火狐
			zhsj.execute();
			Thread.sleep(3000);
			zhsj.quit();
		}
	}
```

### 3.cron表达式
在 springmvc 中，它使用的定时任务是使用cron表达式。它使用的语法与*crontab*命令相似，但在细节上又有些许区别。
> crontab命令常见于Unix和类Unix的操作系统之中，用于设置周期性被执行的指令。该命令从标准输入设备读取指令，并将其存放于“crontab”文件中，以供之后读取和执行。该词来源于希腊语chronos（χρόνος），原意是时间。通常，crontab储存的指令被守护进程激活，crond常常在后台运行，每一分钟检查是否有预定的作业需要执行。这类作业一般称为cron jobs。


*语法*
例如，每十分钟运行一次的语法如下：
	【0	*/10	* 	*	*	?】

|0|*/10|* |* |* |* |?|
|:--|:--|:--|:--|:--|:--|:--|
|秒（0~59）|分钟（0~59）|小时（0~23）|日 （0~31）|月（0~11）|星期（1~7）|年|


+	【,】 【\* 10,20】 表示10和20分时候分别都运行一次

+	【-】 【? 10-20】 表示是10到20分之间每分钟都运行一次

+	【?】 不设置该字段(表示忽略)

+	【\*】 【\* 10】表示每分钟

+	【/】 【开始时间/间隔时间】 ,例如【0/5】0分钟开始，每5分钟运行一次; 【\*/5】在分钟位置表示每五分钟一次


# 参考文献
> [Cron_维基百科](https://zh.wikipedia.org/wiki/Cron)
[SpringMVC中使用Cron表达式的定时器](https://www.cnblogs.com/leeyes999/p/5742287.html)