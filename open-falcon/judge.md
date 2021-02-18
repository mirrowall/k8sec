# judge
## 代码结构
cron
g
http
rpc
store


## 代码分析
g.InitRedisConnPool()  初始化redis
g.InitHbsClient()       初始化心跳服务器连接

http.Start()    启动http监听，　获取策略等
rpc.Start()     
    ->　modules/judge/rpc/receiver.go
    -> store.HistoryBigMap[pk[0:2]].PushFrontAndMaintain
        CheckStrategy(L, firstItem, now)
	    CheckExpression(L, firstItem, now)

cron.SyncStrategies()
    syncStrategies()
    syncExpression()
    syncFilter()

cron.CleanStale()
    cleanStale()