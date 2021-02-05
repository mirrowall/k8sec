# kube-controller-manager

main.main (cmd/kube-controller-manager/controller-manager.go)
    -> CreateControllerContext
        -> NewSharedInformerFactory
        -> NewMemCacheClient
    -> NewControllerInitializers
    -> StartControllers
        ->  startEndpointController
            startEndpointSliceController
            startEndpointSliceMirroringController
            startReplicationController
            startPodGCController
            startResourceQuotaController
            startNamespaceController
            startServiceAccountController
            startGarbageCollectorController
            startDaemonSetController
            startJobController
            startDeploymentController
            startReplicaSetController
            startHPAController
            startDisruptionController
            startStatefulSetController
            startCronJobController
            startCSRSigningController
            startCSRApprovingController
            startCSRCleanerController
            startTTLController
            startBootstrapSignerController
            startTokenCleanerController
            startNodeIpamController
            startNodeLifecycleController
            startServiceController
            startRouteController
            startCloudNodeLifecycleController
            startPersistentVolumeBinderController
            startAttachDetachController
            startVolumeExpandController
            startClusterRoleAggregrationController
            startPVCProtectionController
            startPVProtectionController
            startTTLAfterFinishedController
            startRootCACertPublisher
            startEphemeralVolumeController
            startStorageVersionGCController


NodeLifecycleController 主要是监控 node 状态，当 node 异常时驱逐 node 上的 pod，其行为与其他组件有一定关系，node 的状态由 kubelet 上报，node 异常时为 node 添加 taint 标签后，scheduler 调度 pod 也会有对应的行为。为了保证由于网络等问题引起的 pod 驱逐行为，NodeLifecycleController 会为 node 进行分区并会为每个区设置不同的驱逐速率，即实际上会以 rate-limited 的方式添加 taint，在某些情况下可以避免 pod 被大量驱逐。

此外，NodeLifecycleController 还会对外暴露多个 metrics，包括 zoneHealth、zoneSize、unhealthyNodes、evictionsNumber 等，便于用户查看集群下 node 的状态。

NewNodeLifecycleController (pkg/controller/nodelifecycle/node_lifecycle_controller.go)
    -> Controller
    -> podInformer.Informer().AddEventHandler
    -> nodeInformer.Informer().AddEventHandler

Run
    -> nc.leaseInformerSynced, nc.nodeInformerSynced, nc.podInformerSynced, nc.daemonSetInformerSynced
    -> nc.taintManager.Run
    -> nc.doNodeProcessingPassWorker
    -> nc.doPodProcessingWorker,
    -> nc.monitorNodeHealth

JOB
NewController
    -> Controller
    -> jobInformer.Informer().AddEventHandler
    -> podInformer.Informer().AddEventHandler

Run
    -> jm.worker
    -> syncJob