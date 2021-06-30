+++



title = "Docker 安装 nginx"

date = "2021-06-30"

description = "Docker 安装 nginx"

featured = false

categories = [

  "nginx"

]

tags = [

  "Docker"

]

images = [



"https://busgo.github.io/images/docker01.png"



]



+++

在操作之前必须使用 *docker search * 命令进行 *nginx* 镜像搜索。

```shell
docker search nginx 
```

下载 *nginx* 镜像

```shell
docker pull nginx 
```

启动 nginx 

```shell
docker run -d --name nginx_80  -p 8002:80  nginx
```

验证服务是否成功

```shell
curl http://localhost:8002
```

