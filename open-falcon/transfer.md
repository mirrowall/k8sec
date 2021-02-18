# transfer 分析
## 代码结构
g       常用的函数及全局变量
http    提供的http服务
proc    全局变量
receiver    接收器，　包括rpc 和socket
sender  提供发送队列及发送连接池

## 代码调用结构
proc.Start() 利用　https://github.com/toolkits/proc　这个工程，　初始化了一系列的全局变量

sender创建了四个连接池，　modules/transfer/sender/sender.go
	JudgeConnPools     *backend.SafeRpcConnPools
	TsdbConnPoolHelper *backend.TsdbConnPoolHelper
	GraphConnPools     *backend.SafeRpcConnPools
	TransferConnPools  *backend.SafeRpcConnPools

sender.Start() 
    -> initConnPools　初始化这四个池子
    -> initSendQueues 创建发送的缓存队列　modules/transfer/sender/sender.go
        TsdbQueue     *nlist.SafeListLimited
        JudgeQueues   = make(map[string]*nlist.SafeListLimited)
        GraphQueues   = make(map[string]*nlist.SafeListLimited)
        TransferQueue *nlist.SafeListLimited
        InfluxdbQueue *nlist.SafeListLimited
    -> initNodeRings 服务节点的一致性哈希环
    
    -> startSendTasks
        -> forward2JudgeTask
        -> forward2GraphTask
        -> forward2TsdbTask
        -> forward2TransferTask
        -> forward2InfluxdbTask

    -> startSenderCron
        -> startProcCron
        -> startLogCron
receiver.Start()
    rpc.StartRpc()
        -> modules/transfer/receiver/rpc/rpc_transfer.go
        -> Push2GraphSendQueue
        -> Push2JudgeSendQueue
        -> Push2TsdbSendQueue
        -> Push2TransferSendQueue
        -> Push2InfluxdbSendQueue

    socket.StartSocket()
        -> socketTelnetHandle
            -> Push2GraphSendQueue
            -> Push2JudgeSendQueue
            -> Push2TransferSendQueue
http.Start() 提供一系列的http接口