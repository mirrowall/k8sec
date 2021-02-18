# falcon-agent

## 代码结构：
cron  一些定时的操作，　基本上所有循环定时操作的入口都在这里
funcs 所有的功能函数都在这里
g  全局变量，共用函数在此定义
http 提供http定义的ＵＲＬ处理
plugins 插件的定义以及调度的定义
public agent的web界面

## 代码调用顺序：
如果设置了check，　则只是为了收集信息，调用后就退出
> modules/agent/funcs/checker.go
通过　nux 包，　获取CPU, DISK, PROC 信息
https://github.com/toolkits/nux 

InitDataHistory 定时计算CPU,DISK的信息，　不返回，　将数据存在全局变量中

ReportAgentStatus　定时上报客户端信息，　包括版本，IP, plugin version, hostname
SyncMinePlugins  定时同步plugins, 根据　plugin文件夹中，　首先删除		plugins.DelNoUsePlugins(desiredAll)，　然后再行　plugins.AddNewPlugins(desiredAll)，
每个一个plugin都会生成　NewPluginScheduler，　然后调用　PluginScheduler.Schedule 进行调度，　PluginRun(), 直接运行，　然后获取输出的json格式，　然后采用rpc上传至服务器，　在这里会随机选一个地址上传，　成功则退出，　不成功则继续随机

SyncBuiltinMetrics　从心跳服务器请求Agent.BuiltinMetrics，　然后根据返回，　进行数据的设置

SyncTrustableIps　从心跳服务器请求　Agent.TrustableIps，　然后根据返回设置

Collect　根据设置的mapper，　进行数据的上传，　数据的定义在modules/agent/funcs/funcs.go，这里可以看到进行上传哪些数据，　其中有些上传的数据，　比如　ports 等，　是由上面的函数进行定时设置

启动http监听，　可以根据URL读取相应的信息，供第三方调用



