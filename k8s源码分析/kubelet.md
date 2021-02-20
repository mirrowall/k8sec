# Kubelet 源码分析

main.main (cmd/kubelet/kubelet.go)
    -> NewKubeletCommand
    -> Run
        -> options.ValidateKubeletServer
        -> initConfigz
        -> BuildAuth
        -> cgroupRoots
        -> cadvisor.New
        -> cm.NewContainerManager
        -> checkPermissions
        -> kubelet.PreInitRuntimeService
        -> RunKubelet
            -> createAndInitKubelet (pkg/kubelet/kubelet.go)
            -> startKubelet
                -> Run ()