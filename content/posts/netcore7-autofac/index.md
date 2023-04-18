+++
title = "在asp.net core中使用autofac"
tags = ["c-sharp","编程","aspnet-core","AutoFac"]
date = "2023-04-18"
update = "2023-04-18"
enableGitalk = false
draft = false
weight = 3
+++


### 关于autofac

关于此项目：[项目地址](https://github.com/autofac/Autofac)

在asp.net core7 webapi中的使用方法
``` C#
var builder = WebApplication.CreateBuilder(args);
builder.Host.UseServiceProviderFactory(new AutofacServiceProviderFactory()).ConfigureContainer<ContainerBuilder>(builder =>
{
	//实现层
	Assembly service = Assembly.Load("xxx.Service");
	//接口层
	Assembly iservice = Assembly.Load("xxx.IService");
	builder.RegisterAssemblyTypes(service, iservice)
		.Where(t => (t.FullName.EndsWith("Service")
			|| t.FullName.EndsWith("Notify")
			|| t.FullName.EndsWith("Execute")) && !t.IsAbstract) //类名以service结尾，且类型不能是抽象的
		.AsImplementedInterfaces()
		.InstancePerDependency();
	// 从程序集中注册所有实现了IService接口的服务
});
```


以上便完成了对上述两个程序集的自动注入


{{< notice note >}}
**请注意**，如上只是一个简单的示例,在不同的项目需要注入的方式不同，如果是Transient/Scoped/Singleton 需要单独配置
{{< /notice >}}