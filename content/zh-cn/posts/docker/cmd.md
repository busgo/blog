+++

title = "Docker 基本命令"
date = "2021-06-29"
description = "Docker 基本命令"
featured = false
categories = [
  "Docker"
]
tags = [
  "Docker"
]
images = [

"https://busgo.github.io/images/docker01.png"

]

+++



#### 帮助命令

```shell
docker version  # 查看docker 当前版本信息
docker info  #  查看docker 基本信息 
docker -h   # docker 命令帮助信息
```

命令文档地址: https://docs.docker.com/engine/reference/run/

### 镜像命令

####  查看镜像列表

```shell
# docker images 查看目前存在的镜像列表
apple@appledeMacBook-Pro ~ % docker images
REPOSITORY   TAG       IMAGE ID   CREATED   SIZE
# REPOSITORY 镜像仓库源
# TAG  镜像的标签
# IMAGE ID 镜像的 id
# CREATED 创建镜像的时间
# SIZE 镜像大小

# 可选参数
Options:
  -a, --all             Show all images (default hides intermediate images)
      --digests         Show digests
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print images using a Go template
      --no-trunc        Don't truncate output
  -q, --quiet           Only show image IDs

```

####   搜索镜像

```shell
# 搜索镜像  docker search 镜像名称
apple@appledeMacBook-Pro ~ % docker  search  mysql  
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11063     [OK]       
mariadb                           MariaDB Server is a high performing open sou…   4193      [OK]       
mysql/mysql-server                Optimized MySQL Server Docker images. Create…   822                  [OK]

## 可选参数
Options:
  -f, --filter filter   Filter output based on conditions provided
      --format string   Pretty-print search using a Go template
      --limit int       Max number of search results (default 25)
      --no-trunc        Don't truncate output
```

#### 下载镜像

```shell
# 下载镜像 docker pull 镜像名:tag
apple@appledeMacBook-Pro ~ % docker pull  mysql
Using default tag: latest
latest: Pulling from library/mysql
b4d181a07f80: Downloading [===================>                               ]  10.65MB/27.15MB
a462b60610f5: Download complete 
578fafb77ab8: Download complete 
524046006037: Download complete 
d0cbe54c8855: Download complete 
aa18e05cc46d: Downloading [==============>                                    ]  3.955MB/13.45MB
32ca814c833f: Download complete 
9ecc8abdb7f5: Waiting 
ad042b682e0f: Waiting 
71d327c6bb78: Waiting 
165d1d10a3fa: Waiting 
2f40c47d0626: Waiting 

# 如果不添加tag 则直接使用 latest 版本
# 指定版本下载镜像
apple@appledeMacBook-Pro ~ % docker pull mysql:5.7
5.7: Pulling from library/mysql
```

#### 删除镜像

```shell
# 格式  docker rmi -f  容器id 多个 空格隔开 
apple@appledeMacBook-Pro ~ % docker rmi -f 5c62e459e087
Untagged: mysql:latest
Untagged: mysql@sha256:52b8406e4c32b8cf0557f1b74517e14c5393aff5cf0384eff62d9e81f4985d4b
Deleted: sha256:5c62e459e087e3bd3d963092b58e50ae2af881076b43c29e38e2b5db253e0287
Deleted: sha256:b92a81bddd621ceee73e48583ed5c4f0d34392a5c60adf37c0d7acc98177e414
Deleted: sha256:265829a9fa8318ae1224f46ab7bc0a10d12ebb90d5f65d71701567f014685a9e
Deleted: sha256:2b9144b43d615572cb4a8fb486dfad0f78d1748241e49adab91f6072183644e9
Deleted: sha256:944ffc10a452573e587652116c3217cf571a32c45a031b79fed518524c21fd4f
Deleted: sha256:b9108f19e3abf550470778a9d91959ce812731d3268d7224e328b0f7d8a73d26

# 说明
# 在使用 docker rmi 命令的时候必须添加 `-f` 参数 强制删除操作。
```

### 容器命令

在使用容器命令的时候我们必须存在镜像才能新建容器

> docker pull centos 

#### 创建容器并启动

```shell
#  启动容器   docker run [可选参数]  image
# 可选参数说明
--name    容器名字 
-it 			使用交互式运行 能够进入容器
-d  			后台方式运行
-p 				指定端口   port1:port2
-P 				随机绑定端口

# 创建容器并进入容器[非后台运行] 
apple@appledeMacBook-Pro ~ % docker images
REPOSITORY   TAG       IMAGE ID       CREATED        SIZE
centos       latest    300e315adb2f   6 months ago   209MB
apple@appledeMacBook-Pro ~ % docker run -it 300e315adb2f /bin/bash
[root@5385bfdb8473 /]# 

```

#### 后台启动容器

```shell
# 后台启动容器  docker run -it -d 镜像id
apple@appledeMacBook-Pro ~ % docker run -d  -it 300e315adb2f /bin/bash
f328a564c9efad5d020de59e434b5f836695488e7d2ceaf801a0fdbc9f57dfff
```



#### 查看运行中的容器列表

```shell
# 查看运行容器列表
apple@appledeMacBook-Pro ~ % docker ps 
CONTAINER ID   IMAGE          COMMAND       CREATED          STATUS          PORTS     NAMES
f328a564c9ef   300e315adb2f   "/bin/bash"   31 seconds ago   Up 31 seconds             laughing_mendeleev
# CONTAINER ID  容器id
# IMAGE 				镜像id
# COMMAND   		启动执行的命令
# CREATED   		创建时间
# STATUS 				状态
#	PORTS					映射端口
#	NAMES     		容器名称
```

#### 退出容器

```shell
# 退出容器  exit
直接在容器里面执行命令 `exit`
```

#### 删除容器

```shell
# docker rm   容器id
apple@appledeMacBook-Pro ~ % docker rm -f f328a564c9ef
f328a564c9ef
# 如果强制删除 要添加 `-f` 参数强制移除
```

#### 重启/停止容器

```shell
# 停止容器  docker stop 容器id  或者 docker kill 容器id
apple@appledeMacBook-Pro ~ % docker stop fdc986471165
fdc986471165

#	重启容器 docker restart 容器id
apple@appledeMacBook-Pro ~ % docker restart 16f113adb432
16f113adb432
```

### 其他命令

#### 查看容器日志命令

```shell
# 查看容器日志命令 docker logs -f -t 
apple@appledeMacBook-Pro ~ % docker logs -f -t 16f113adb432
2021-06-29T14:11:26.449041924Z [root@16f113adb432 /]# exit
# 参数
# --tail  number 
# -tf     格式化添加时间戳
```

#### 查看容器内部进程信息

```shell
# 查看容器内部进程信息  docker top  16f113adb432
apple@appledeMacBook-Pro ~ % docker top  16f113adb432
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                3830                3801                0                   14:11               ?                   00:00:00            /bin/bash
```

#### 查看容器内部信息

```shell
apple@appledeMacBook-Pro ~ % docker inspect 16f113adb432
[
    {
        "Id": "16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be",
        "Created": "2021-06-29T14:11:11.74918956Z",
        "Path": "/bin/bash",
        "Args": [],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3830,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-29T14:11:26.779386787Z",
            "FinishedAt": "2021-06-29T14:11:26.450332121Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be/hostname",
        "HostsPath": "/var/lib/docker/containers/16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be/hosts",
        "LogPath": "/var/lib/docker/containers/16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be/16f113adb432971bea9366b6806363c6e3b8f9fb3b8196e0676c4a439d02f4be-json.log",
        "Name": "/vigilant_perlman",
        "RestartCount": 0,
        "Driver": "overlay2",
        "Platform": "linux",
        "MountLabel": "",
        "ProcessLabel": "",
        "AppArmorProfile": "",
        "ExecIDs": null,
        "HostConfig": {
            "Binds": null,
            "ContainerIDFile": "",
            "LogConfig": {
                "Type": "json-file",
                "Config": {}
            },
            "NetworkMode": "default",
            "PortBindings": {},
            "RestartPolicy": {
                "Name": "no",
                "MaximumRetryCount": 0
            },
            "AutoRemove": false,
            "VolumeDriver": "",
            "VolumesFrom": null,
            "CapAdd": null,
            "CapDrop": null,
            "CgroupnsMode": "host",
            "Dns": [],
            "DnsOptions": [],
            "DnsSearch": [],
            "ExtraHosts": null,
            "GroupAdd": null,
            "IpcMode": "private",
            "Cgroup": "",
            "Links": null,
            "OomScoreAdj": 0,
            "PidMode": "",
            "Privileged": false,
            "PublishAllPorts": false,
            "ReadonlyRootfs": false,
            "SecurityOpt": null,
            "UTSMode": "",
            "UsernsMode": "",
            "ShmSize": 67108864,
            "Runtime": "runc",
            "ConsoleSize": [
                0,
                0
            ],
            "Isolation": "",
            "CpuShares": 0,
            "Memory": 0,
            "NanoCpus": 0,
            "CgroupParent": "",
            "BlkioWeight": 0,
            "BlkioWeightDevice": [],
            "BlkioDeviceReadBps": null,
            "BlkioDeviceWriteBps": null,
            "BlkioDeviceReadIOps": null,
            "BlkioDeviceWriteIOps": null,
            "CpuPeriod": 0,
            "CpuQuota": 0,
            "CpuRealtimePeriod": 0,
            "CpuRealtimeRuntime": 0,
            "CpusetCpus": "",
            "CpusetMems": "",
            "Devices": [],
            "DeviceCgroupRules": null,
            "DeviceRequests": null,
            "KernelMemory": 0,
            "KernelMemoryTCP": 0,
            "MemoryReservation": 0,
            "MemorySwap": 0,
            "MemorySwappiness": null,
            "OomKillDisable": false,
            "PidsLimit": null,
            "Ulimits": null,
            "CpuCount": 0,
            "CpuPercent": 0,
            "IOMaximumIOps": 0,
            "IOMaximumBandwidth": 0,
            "MaskedPaths": [
                "/proc/asound",
                "/proc/acpi",
                "/proc/kcore",
                "/proc/keys",
                "/proc/latency_stats",
                "/proc/timer_list",
                "/proc/timer_stats",
                "/proc/sched_debug",
                "/proc/scsi",
                "/sys/firmware"
            ],
            "ReadonlyPaths": [
                "/proc/bus",
                "/proc/fs",
                "/proc/irq",
                "/proc/sys",
                "/proc/sysrq-trigger"
            ]
        },
        "GraphDriver": {
            "Data": {
                "LowerDir": "/var/lib/docker/overlay2/c115fc909f04da008acb5492cffb371cd9c434e58ad2e028d371c2ccff680735-init/diff:/var/lib/docker/overlay2/a03dddb97f534806e0b2ba00ebfb69db3d058a195d0e86fd590021f17ebd5c4c/diff",
                "MergedDir": "/var/lib/docker/overlay2/c115fc909f04da008acb5492cffb371cd9c434e58ad2e028d371c2ccff680735/merged",
                "UpperDir": "/var/lib/docker/overlay2/c115fc909f04da008acb5492cffb371cd9c434e58ad2e028d371c2ccff680735/diff",
                "WorkDir": "/var/lib/docker/overlay2/c115fc909f04da008acb5492cffb371cd9c434e58ad2e028d371c2ccff680735/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "16f113adb432",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": true,
            "OpenStdin": true,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/bash"
            ],
            "Image": "300e315adb2f",
            "Volumes": null,
            "WorkingDir": "",
            "Entrypoint": null,
            "OnBuild": null,
            "Labels": {
                "org.label-schema.build-date": "20201204",
                "org.label-schema.license": "GPLv2",
                "org.label-schema.name": "CentOS Base Image",
                "org.label-schema.schema-version": "1.0",
                "org.label-schema.vendor": "CentOS"
            }
        },
        "NetworkSettings": {
            "Bridge": "",
            "SandboxID": "987372635d9b2c5537e0aeb15e43874dc84098978caa27e24d255dc1cb450ec7",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/987372635d9b",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "9c1359c52530151adf4a6429f1bcf9e8bb0b5970fa061e91878d1c45eefb3c1d",
            "Gateway": "172.17.0.1",
            "GlobalIPv6Address": "",
            "GlobalIPv6PrefixLen": 0,
            "IPAddress": "172.17.0.2",
            "IPPrefixLen": 16,
            "IPv6Gateway": "",
            "MacAddress": "02:42:ac:11:00:02",
            "Networks": {
                "bridge": {
                    "IPAMConfig": null,
                    "Links": null,
                    "Aliases": null,
                    "NetworkID": "b63ddd2e5e8ff0d212d4b05a97fc36d6467df9fa85310cb2ca3296bb14c5f29e",
                    "EndpointID": "9c1359c52530151adf4a6429f1bcf9e8bb0b5970fa061e91878d1c45eefb3c1d",
                    "Gateway": "172.17.0.1",
                    "IPAddress": "172.17.0.2",
                    "IPPrefixLen": 16,
                    "IPv6Gateway": "",
                    "GlobalIPv6Address": "",
                    "GlobalIPv6PrefixLen": 0,
                    "MacAddress": "02:42:ac:11:00:02",
                    "DriverOpts": null
                }
            }
        }
    }
]
apple@appledeMacBook-Pro ~ % 
```

#### 进入运行中的容器

```shell
# 进入运行中的容器 docker exec -it 容器id /bin/bash
apple@appledeMacBook-Pro ~ % docker exec   -it 16f113adb432 /bin/bash 
[root@16f113adb432 /]# 
# 方式二
# docker attach 容器id /bin/bash 不是创建一个新的终端
apple@appledeMacBook-Pro ~ % docker attach  16f113adb432          
[root@16f113adb432 /]# ls
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var
```

#### 从容器复制至主机

```shell
# 从容器复制至主机  docker cp 容器id:/目录   目标地址
apple@appledeMacBook-Pro ~ % docker cp 147f83591e9f:/1.txt ~/Desktop/wp 
apple@appledeMacBook-Pro ~ % ls  ~/Desktop/wp 
1.txt	code	doc	img	soft
apple@appledeMacBook-Pro ~ % 
```

