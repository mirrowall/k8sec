# ETCD 存储分析

+ RESTStorage       (实现了RESTful风格的对外资源存储服务的API接口)
  > 代码路径: vendor/k8s.io/apiserver/pkg/registry/rest/rest.go

  Kubernetes的每种资源实现的RESTStorage接口一般定义在pkg/registry/<资源组>/<资源>/storage/storage.go中
  以Deployment资源为例, 
  > pkg/registry/apps/deployment/storage/storage.go

+ RegistryStore     (实现了资源存储的通用操作)

    当通过RegistryStore存储了一个资源对象时，RegistryStore中定义了如下两种函数。●
    + Before Func：也称Strategy预处理，它被定义为在创建资源对象之前调用，做一些预处理工作。
    + After Func：它被定义为在创建资源对象之后调用，做一些收尾工作。

  > 代码路径：vendor/k8s.io/apiserver/pkg/registry/generic/registry/store.go

  RegistryStore定义了4种Strategy预处理方法，分别是CreateStrategy（创建资源对象时的预处理操作）、UpdateStrategy （更新资源对象时的预处理操作）、DeleteStrategy（删除资源对象时的预处理操作）、ExportStrategy（导出资源对象时的预处理操作）

  RegistryStore定义了3种创建资源对象后的处理方法，分别是AfterCreate （创建资源对象后的处理操作）、AfterUpdate （更新资源对象后的处理操作）、AfterDelete（删除资源对象后的处理操作）

+ Storage.Interface (通用存储接口，该接口定义了资源的操作方法)
  > 代码路径：vendor/k8s.io/apiserver/pkg/storage/interfaces.go
  ```
  ● Versioner：资源版本管理器，用于管理Etcd集群中的数据版本对象。
  ● Create：创建资源对象的方法。
  ● Delete：删除资源对象的方法。
  ● Watch：通过Watch机制监控资源对象变化方法，只应用于单个key。
  ● WatchList：通过Watch机制监控资源对象变化方法，应用于多个key（当前目录及目录下所有的key）。
  ● Get：获取资源对象的方法。
  ● GetToList：获取资源对象的方法，以列表（List）的形式返回。
  ● List：获取资源对象的方法，以列表（List）的形式返回。
  ● GuaranteedUpdate：保证传入的tryUpdate函数运行成功。
  ● Count：获取指定key下的条目数量。
  ```
+ CacherStorage     (带有缓存功能的资源存储对象)
  CacherStoraeg缓存层并非会为所有操作都缓存数据。对于某些操作，为保证数据一致性，没有必要在其上再封装一层缓存层，例如Create、Delete、Count等操作，通过UnderlyingStorage直接向Etcd集群发出请求即可。只有Get、GetToList、List、GuaranteedUpdate、Watch、WatchList等操作是基于缓存设计的。__其中Watch操作的事件缓存机制（即watchCache）使用缓存滑动窗口来保证历史事件不会丢失，设计较为巧妙__。

  > 代码路径：vendor/k8s.io/apiserver/pkg/storage/cacher/cacher.go


+ UnderlyingStorage (底层存储，也被称为BackendStorage（后端存储），是真正与Etcd集群交互的资源存储对象)

  UnderlyingStorage（底层存储），也称为BackendStorage（后端存储），是真正与Etcd集群交互的资源存储对象, UnderlyingStorage对Etcd的官方库进行了封装
  > 代码路径：vendor/k8s.io/apiserver/pkg/storage/storagebackend/factory/factory.go
  
