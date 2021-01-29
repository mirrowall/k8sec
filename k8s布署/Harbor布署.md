# Harbor 布署

同步源码:
> https://github.com/goharbor/harbor

从这里下载最新的 release 版本, 解压
修改 docker-compose.yaml 里面的 hostname 为服务器的IP地址

然后运行:
> ./install.sh --with-clair 

如果需要修改参数, 在修改 docker-compose.yaml 文件后, 调用
> ./prepare --with-clair --with-chartmuseum --with-trivy

安装, 便可以把 clair 也安装进去

## 使用harbor上传下载镜像
> docker login http://10.240.53.31:30800

Error response from daemon: Get https://10.240.53.31:30800/v2/: http: server gave HTTP response to HTTPS client
```
# cd /etc/docker/
# vi daemon.json
{
 "insecure-registries" : ["www.harbor2.com"]
}

systemctl restart docker
```

Docker服务器给镜像打标签
> docker tag mysql:5.7 10.240.53.31:30800/mysql:5.7

上传
> docker push 10.240.53.31:30800/antiy-cloud/mysql:5.7


如果k8s需要下载私有仓库的镜像, 须生成secret用于调用
首先生成harbor的授权令牌
> cat /root/.docker/config.json | base64 -w 0


```
apiVersion: v1
kind: Secret
metadata:
  name: harbor-sec
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: ewoJImF1dGhzIjogewoJCSIxMC4yNDAuNTMuMzE6MzA4MDAiOiB7CgkJCSJhdXRoIjogImNHRnVaM2hzT2taMVkydEFNVEl6IgoJCX0KCX0KfQ==
```

导入后, 在需要下载镜像的yaml里增加:
```
imagePullSecrets:
- name: harbor-sec
```