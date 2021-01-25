# clair 源码分析

https://github.com/quay/clair.git

Clair主要包括以下模块：
+ 获取器（Fetcher）- 从公共源收集漏洞数据
+ 检测器（Detector）- 指出容器镜像中包含的Feature
+ 容器格式器（Image Format）- Clair已知的容器镜像格式，包括Docker，ACI
+ 通知钩子（Notification Hook）- 当新的漏洞被发现时或者已经存在的漏洞发生改变时通知用户/机器
+ 数据库（Databases）- 存储容器中各个层以及漏洞
+ Worker - 每个Post Layer都会启动一个worker进行Layer Detect

编译方式:
>　go build github.com/coreos/clair/cmd/clair

启动一个pgsql容器作为Clair的Backend DB
> docker run -p 5432:5432 -e POSTGRES_PASSWORD=passw0rd postgres:latest

config.yaml
```
clair:
  database:
    type: pgsql
    options:
      source: postgresql://postgres:root123@postgresql:5432/postgres?sslmode=disable
      cachesize: 16384

  api:
    # API server port
    port: 6060
    healthport: 6061

    # Deadline before an API request will respond with a 503
    timeout: 300s
  updater:
    interval: 12h
```
启动clair
> clair -config config.yaml

## Clair整体处理流程如下：

+ Clair定期从配置的源获取漏洞元数据然后存进数据库。
+ 客户端使用Clair API处理镜像，获取镜像的特征并存进数据库。
+ 客户端使用Clair API从数据库查询特定镜像的漏洞情况，为每个请求关联漏洞和特征，避免需要重新扫描镜像。
+ 当更新漏洞元数据时，将会有系统通知产生。另外，还有webhook用于配置将受影响的镜像记录起来或者拦截其部署。

