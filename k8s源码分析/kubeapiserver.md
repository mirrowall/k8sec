# kube-apiserver 分析

cmd/kube-apiserver/appserver.go
    - main()

从源码上分析, 包括三部分:
cmd/kube-apiserver/app/server.go
cmd/kube-apiserver/app/apiextensions.go
cmd/kube-apiserver/app/aggregator.go

server分析:
    NewAPIServerCommand 创建命令行
    Run 主要的执行函数, 是由 cobra.Command 包所定义
    CreateServerChain Run函数中的首要函数 
    CreateKubeAPIServer 创建CreateKubeAPIServer
    CreateKubeAPIServerConfig  获取KubeAPIServer 的配置
从文件结构上来说, 主要是APIServer的实现, 结合 main.main 上看, 此文件也是main入口的主要函数

aggregator分析:
暴露的功能类似于一个七层负载均衡，将来自用户的请求拦截转发给其他服务器，并且负责整个 APIServer 的 Discovery 功能
    makeAPIService 创奸定个APIServer
    apiServicesToRegister 注册
    createAggregatorServer 创建一个聚合
    createAggregatorConfig 创建Aggregator的Config

apiextensions分析:
APIExtensionServer 作为 Delegation 链的最后一层，是处理所有用户通过 Custom Resource Definition 定义的资源服务器。

    createAPIExtensionsConfig 获取配置
    createAPIExtensionsServer 创建server

pkg/apiserver/server/option 包, 定义了一些常用的Options 


## kube-apiserver 启动流程分析
1.初始化Scheme资源注册表
> 代码路径：pkg/api/legacyscheme/scheme.go
```
	Scheme = runtime.NewScheme()
	Codecs = serializer.NewCodecFactory(Scheme)
	ParameterCodec = runtime.NewParameterCodec(Scheme)
```

调用 Complete 获取一些配置 -> ServerRunOptions 定义了相当多的选项
    -> DefaultAdvertiseAddress 获取默认的IP
    -> getServiceIPAndRanges 获取服务IP范围
    -> MaybeDefaultWithSelfSignedCerts 判断是否指定证书, 如果没指定, 会尝试生成一个
    -> 处理 serveraccount 的私有证书

1、调用 CreateServerChain 构建服务调用链并判断是否启动非安全的 http server，http server 链中包含 apiserver 要启动的三个 server，以及为每个 server 注册对应资源的路由；

CreateServerChain
    -> CreateNodeDialer 创建一个连接node的连接器, 其中会判断是否可以用 ssh 来连接, 返回为 tunneler.Tunneler, *http.Transport, error
    -> CreateKubeAPIServerConfig 获取 APIServer 的配置
        -> buildGenericConfig 创建配置, 
            -> NewConfig  Config(vendor/k8s.io/apiserver/pkg/server/config.go)
            -> DefaultOpenAPIConfig
            -> NewStorageFactoryConfig
            -> BuildAuthorizer
            -> buildServiceResolver
        -> controlplane.Config
        -> GetClientCAContentProvider  (serving-cert::/var/run/kubernetes/serving-kube-apiserver.crt::/var/run/kubernetes/serving-kube-apiserver.key)
        -> PublicKeysFromFile
    
    -> createAPIExtensionsConfig
    -> createAPIExtensionsServer (apiextensionsConfig.Complete().New(delegateAPIServer))
        -> customresourcedefinition.NewREST
        -> customresourcedefinition.NewStatusREST
        -> InstallAPIGroup
            -> getOpenAPIModels  /apis
            -> installAPIResources
        -> NewCustomResourceDefinitionHandler 创建一个CRD的handler, 定义Add/Update/Delete的handler, (/vendor/k8s.io/apiextensions-apiserver/pkg/apiserver/customresource_handler.go)
        -> NewDiscoveryController
        -> NewNamingConditionController
        -> NewConditionController
        -> NewKubernetesAPIApprovalPolicyConformantConditionController
        -> NewCRDFinalizer
        -> AddPostStartHookOrDie  以上都使用使用了 Infomer 的 HOOK机制, 将相应的事件均HOOK至相应的handler中
    -> CreateKubeAPIServer (kubeAPIServerConfig.Complete().New(delegateAPIServer))
        -> GenericConfig.New
            -> NewAPIServerHandler
            -> GenericAPIServer
            -> 处理 PostStartHooks, PreShutdownHooks
            -> installAPI
        -> NewOpenIDMetadata
        -> Instance
        -> InstallLegacyAPI (bootstrap-controller)
        -> RESTStorageProvider
        -> InstallAPIs
        -> installTunneler
        -> AddPostStartHookOrDie
    -> createAggregatorConfig
  - -> createAggregatorServer
    - -> NewWithDelegate
      - -> APIAggregator
      - -> NewRESTStorage
      - -> InstallAPIGroup
      - -> NewAPIServiceRegistrationController (APIServiceRegistrationController)
      - -> NewAvailableConditionController (AvailableConditionController)
      - -> AddPostStartHookOrDie ()
    - -> NewAutoRegisterController (autoRegisterController)
    - -> apiServicesToRegister
    - -> NewCRDRegistrationController
    - -> AddPostStartHook(kube-apiserver-autoregistration)
      - -> autoRegistrationController.Run
PrepareRun
    -> AddPostStartHookOrDie("apiservice-openapi-controller"_
        -> openAPIAggregationController.Run
    -> GenericAPIServer.PrepareRun
        -> s.installHealthz()
        -> s.installLivez()
        -> s.installReadyz()
    -> openapiaggregator.NewDownloader()
    -> openapicontroller.NewAggregationController


2、调用 server.PrepareRun 进行服务运行前的准备，该方法主要完成了健康检查、存活检查和OpenAPI路由的注册工作；
3、调用 prepared.Run 启动 https server；