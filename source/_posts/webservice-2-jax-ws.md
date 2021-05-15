---
title: webservice 使用（二）jax-ws
date: 2019-11-30 17:05:28
tags: [javaweb]
---


# 一、什么是JAX-WS

JAX-WS(Java API for XML Web Services)规范是一组XML web services的JAVA API，JAX-WS允许开发者可以选择RPC-oriented或者message-oriented 来实现自己的web services。

### 0. 相关术语

+	JAX-WS【java soap实现规范】	
	+	规范；一组XML web services的JAVA API；
	+	风格
		+	RPC风格，依赖语言，跨语言性不佳
		+	文档风格，难以使用
		
+	RPC【远程过程调用】 ———— Remote Procedure Call；
	+	特点
		+	C/S结构、无状态。从客户端到服务器的每个请求都必须包含理解请求所必需的信息。如果服务器在请求之间的任何时间点重启，客户端不会得到通知。
		+	cache
		+	无状态请求可以由任何可用服务器回答
		+	统一的接口
	+	目标
		+	解决分布式系统中服务的调用
		+	远程调用
		+	隐藏底层细节，调用时可以像调用本地服务一样来调用其它系统中的服务
		
	+	![webservice-rpc](/image/webservice/webservice-rpc.png)
+	RMI【RPC，远程对象调用】
	+	java实现了RPC的一组接口
+	JMS【MQ】
+	EJB【大型分布式，rmi-iiop协议】

##### 0.1 RPC历史
![RPC历史](/image/webservice/webservice-history.png)


### 2. 实例
通过springboot整合webservice，java提供的原生【jax-ws】难以使用，而spring提供了对【jax-ws】的支持。

Spring提供标准Java web services APIs完全支持：
+	使用JAX-WS暴露web业务
+	使用JAX-WS访问web业务

Found option without preceding group in config file 。修改mysql配置文件【my.ini】后无法启动服务。

+	my.ini文件格式需要为ANSI


*配置服务端bean*
```
@Configuration
public class WebServiceConfig {
    @Bean
    public SimpleJaxWsServiceExporter getSimpleJaxWsServiceExporter() {
        SimpleJaxWsServiceExporter exporter = new SimpleJaxWsServiceExporter();
        exporter.setBaseAddress("http://127.0.0.1:8080/wb_server/");
        return exporter;
    }
}
```

*配置服务端业务代码*
```
@Service("dataService")
//webservice地址使用
@WebService(serviceName="dataService")
//防止jdk版本过低导致无法生成wsdl
@SOAPBinding(style=Style.RPC)
public class DataService {
	@WebMethod
	public String getData(String tableName, String pdate) {
		return "success";
	}
}

```

*测试*
wsdl地址：`http://localhost:8080/wb_server/dataService?wsdl`


		
>	[解释什么是RPC](https://www.jianshu.com/p/2accc2840a1b)