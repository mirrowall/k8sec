# graph
## 代码结构
api     提供api接口
cron    定时清除
g       解析cfg文件，　及其它全局
http
index
proc
rrdtool
store

## 代码分析
g.InitDB()  创建mysql连接，　并创建连接池
rrdtool.InitChannel()   初始化rrd
rrdtool.Start()

api.Start()
    -> Graph    modules/graph/api/graph.go

index.Start()
    定时和mysql交互，　插入更新数据

http.Start()
    初始化http服务请求

cron.CleanCache()
    DeleteInvalidItems()   // 删除无效的GraphItems
    DeleteInvalidHistory() // 删除无效的HistoryCache