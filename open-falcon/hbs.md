# heartbeat server

## 代码结构
cache   从数据库初始数据
db      数据库操作
g       解析配置文件
http    
rpc     进行rpc连接的解析，　包括hbs, agent

## 代码调用逻辑
db.Init
    打开 mysql 数据库，　并设置相应的连接设置

cache.Init
    -> GroupPlugins
    -> GroupTemplates
    -> HostGroupsMap
    -> HostMap
    -> TemplateCache
    -> Strategies
    -> HostTemplateIds
    -> ExpressionCache
    -> MonitoredHosts
http.Start
    初始化http的请求信息
rpc.Start
    -> agent    modules/hbs/rpc/agent.go
    -> hbs      modules/hbs/rpc/hbs.go