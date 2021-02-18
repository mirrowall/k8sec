# aggregator 
## 代码结构
cron    定时计算
db      处理数据库的操作
g       处理配置文件及全局变量
http    处理http请求
sdk     提供http请求数据处理


## 代码分析
db.Init()　　        打开mysql数据库，　并设置连接参数
http.Start()        启动http请求
cron.UpdateItems()  
    -> db.ReadClusterMonitorItems
    -> deleteNoUseWorker
    -> createWorkerIfNeed
        -> NewWorker  modules/aggregator/cron/run.go
            -> WorkerRun
                -> /api/v1/graph/lastpoint
                -> 经过一系列的计算，　将数据放至　MetaDataQueue

sender.StartSender
    -> MetaDataQueue 从这个数据队列里拉取数据，　然后POST在PostPushUrl