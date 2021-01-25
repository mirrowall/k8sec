# Harbor 布署

同步源码:
> https://github.com/goharbor/harbor

从这里下载最新的 release 版本, 解压
然后运行:
> ./install.sh --with-clair 

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

