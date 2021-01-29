# Kubernetes 的扩展点概览
1. Admission Control
    + Initializers: kubernetes-initializer-tutorial
    + GenericAdmissionWebhook ImagePolicyWebhook
2. Device Plugins(1.8)
3. CloudProvider
4. CNI 插件
5. Volume 插件
6. CustomResourceDefinitions

https://www.wenjiangs.com/doc/n4xxzmhz2

Kubernetes 的源码概览
Kubernetes 源码目录结构
Kubernetes 构建脚本
Kubernetes 的源码生成机制
https://github.com/kubernetes/gengo
https://github.com/kubernetes/code-generator
Kubernetes 的 CloudProvider
CloudProvider 的作用
CloudProvider 案例解读
Kubernetes 的 CNI 插件
CNI 的规范
CNI 的运行原理-通过 CNI script 分析
Kubernetes 的 Volume 插件
Volume Provisioner 接口
Flex Volume 接口
Volume Plugin 案例解读
CustomResourceDefinitions
CRD 应用场景
CRD 规范以及机制
CRD 案例讲解
https://github.com/openshift-evangelists/crd-code-generation

按扩展点分类
１.kubectl命令行工具扩展
通过为kubectl提供插件并配置扩展kubectl，属于二进制扩展。这种扩展只对集群中单个kubectl有效，不是全局范围。详细参考kubectl plugins。
https://kubernetes.io/docs/tasks/extend-kubectl/kubectl-plugins/

２.apiserver扩展
apiserver预留的挂载点，可以与远程服务挂钩，将请求转发出去由远程服务处理，自定义实现比如认证、其于内容的拦截、请求内容转换、删除处理等，详细参考API Access Extensions，这种属于webhook扩展类型。
https://kubernetes.io/docs/concepts/overview/extending#api-access-extensions

３.自定义资源扩展
kubernetes除了可以处理系统内建的资源类型如pod，也可以处理用户自定义的资源类型，这种资源称为custom resources，经常需要与其它扩展点组合，这应该也是属于这种属于webhook扩展类型。
https://kubernetes.io/docs/concepts/overview/extending#user-defined-types

４.调度器扩展
就是扩展kubernetes scheduler，详细参考Scheduler Extensions，这属于控制器扩展类型。
https://kubernetes.io/docs/concepts/overview/extending#scheduler-extensions

５.控制器扩展
无需解释，属于控制器扩展类型。

６.kubelet网络扩展
与kubectl一样，属于二进制扩展类型，参考Network Plugins。
https://kubernetes.io/docs/concepts/overview/extending#network-plugins

７.kubelet存储扩展
与６同，不解释。

8.动态准入控制扩展
当创建某种类型的资源时，可以在标准流程之外加挂额外处理实现扩展，比如创建之前的初始化工作，称为“Dynamic Admission Control”。参考Admission Control、Image Policy webhook、 Admission webhook、Initializers。