# Prometheus 布署

同步源码:
> https://github.com/prometheus/prometheus.git

如里需要编译的话, 直接运行: 
> make build

即可在目录下生成: prometheus  promtool

布署时使用K8S直接布署, 同步布署代码工程: 
> https://github.com/prometheus-operator/kube-prometheus.git

布署的脚本在 manifests 目录下:
> kubectl apply -f manifests/setup

> kubectl apply -f manifests

运行完成后, k8s会拉取相应的镜像并生成相应的pods.

修改 prometheus-service.yaml 中的 type 为 NodePort, 并设置其nodePort为:30200, 这样便可以通过 http://10.240.53.31:32000 访问 prometheus

同样修改 grafana-service.yaml 中的 type NodePort, 并设置其nodePort为:30100, 这样便可以通过 http://10.240.53.31:30100 访问 grafana