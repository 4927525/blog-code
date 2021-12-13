---
title: "Docker学习笔记"
date: 2021-12-13T17:12:42+08:00
Description: ""
Tags: ["docker", "Dockerfile", "docker-compose"]
Categories: ["docker"]
DisableComments: false
---
# Docker安装

## 使用官方安装脚本自动安装

安装命令如下：

```
curl -fsSL https://get.docker.com | bash -s docker --mirror aliyun
```

也可以使用国内 daocloud 一键安装命令：

```
curl -sSL https://get.daocloud.io/docker | sh
```

## 手动安装

### 卸载旧版本

较旧的 Docker 版本称为 docker 或 docker-engine 。如果已安装这些程序，请卸载它们以及相关的依赖项。

```shell
yum remove docker \
         docker-client \
         docker-client-latest \
         docker-common \
         docker-latest \
         docker-latest-logrotate \
         docker-logrotate \
         docker-engine
```

### 安装 Docker Engine-Community

### 使用 Docker 仓库进行安装

在新主机上首次安装 Docker Engine-Community 之前，需要设置 Docker 仓库。之后，您可以从仓库安装和更新 Docker。

**设置仓库**

安装所需的软件包。yum-utils 提供了 yum-config-manager ，并且 device mapper 存储驱动程序需要 device-mapper-persistent-data 和 lvm2。

```bash
yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

使用以下命令来设置稳定的仓库。

## 使用官方源地址（比较慢）

```bash
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

可以选择国内的一些源地址：

## 阿里云

```bash
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```

### 安装 Docker Engine-Community

安装最新版本的 Docker Engine-Community 和 containerd，或者转到下一步安装特定版本：

```
$ sudo yum install docker-ce docker-ce-cli containerd.io
```

如果提示您接受 GPG 密钥，请选是。

Docker 安装完默认未启动。并且已经创建好 docker 用户组，但该用户组下没有用户。

**要安装特定版本的 Docker Engine-Community，请在存储库中列出可用版本，然后选择并安装：**

1、列出并排序您存储库中可用的版本。此示例按版本号（从高到低）对结果进行排序。

```bash
$ yum list docker-ce --showduplicates | sort -r

docker-ce.x86_64  3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64  3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64  18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64  18.06.0.ce-3.el7                    docker-ce-stable
```

2、通过其完整的软件包名称安装特定版本，该软件包名称是软件包名称（docker-ce）加上版本字符串（第二列），从第一个冒号（:）一直到第一个连字符，并用连字符（-）分隔。例如：docker-ce-18.09.1。

```
$ sudo yum install docker-ce-<VERSION_STRING> docker-ce-cli-<VERSION_STRING> containerd.io
```

启动 Docker。

```
$ sudo systemctl start docker
```

通过运行 hello-world 映像来验证是否正确安装了 Docker Engine-Community 。

```
$ sudo docker run hello-world
```

### 卸载 docker

删除安装包：

```
yum remove docker-ce
```

删除镜像、容器、配置文件等内容：

```
rm -rf /var/lib/docker
```

# Docker常用命令

## 帮助命令

```bash
docker version
docker info
docker 命令 --help
```

帮助文档的地址：https://docs.docker.com/engine/reference/commandline/docker/

## 镜像命令

`docker images`查看所有的本地的主机上的镜像

```bash
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
hello-world   latest    d1165f221234   3 months ago   13.3kB
# 解释
REPOSITORY 镜像的仓库原
TAG        镜像的标签
IMAGE ID   镜像的ID
CREATED    镜像的创建时间
SIZE       镜像的大小

# 可选项
-a -all    # 列出所有的镜像
-q -quiet  # 只显示镜像的ID
```

`docker search`搜索镜像

```bash
[root@localhost ~]# docker search mysql
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql                             MySQL is a widely used, open-source relation…   11038     [OK]
mariadb                           MariaDB Server is a high performing open sou…   4184      [OK]

# 可选项
--filter=STARS=3000 # 搜索出来的镜像就是STARS大于3000的
[root@localhost ~]#  docker search mysql --filter=STARS=3000
NAME      DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
mysql     MySQL is a widely used, open-source relation…   11038     [OK]
mariadb   MariaDB Server is a high performing open sou…   4184      [OK]
```

`docker pull` 下载镜像

```bash
# 下载镜像 docker pull 镜像名[:tag]
[root@localhost ~]# docker pull mysql
Using default tag: latest # 如果不屑 tag，默认就是latest
latest: Pulling from library/mysql
b4d181a07f80: Pull complete # 分层下载，docker images的核心 联合文件系统
a462b60610f5: Pull complete
578fafb77ab8: Pull complete
524046006037: Pull complete
d0cbe54c8855: Pull complete
aa18e05cc46d: Pull complete
32ca814c833f: Pull complete
9ecc8abdb7f5: Pull complete
ad042b682e0f: Pull complete
71d327c6bb78: Pull complete
165d1d10a3fa: Pull complete
2f40c47d0626: Pull complete
Digest: sha256:52b8406e4c32b8cf0557f1b74517e14c5393aff5cf0384eff62d9e81f4985d4b # 签名
Status: Downloaded newer image for mysql:latest
docker.io/library/mysql:latest # 真实地址
```

```bash
[root@localhost ~]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
mysql         latest    5c62e459e087   47 hours ago   556MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
```

`docker rmi`删除镜像

```bash
[root@localhost ~]# docker rmi -f 容器ID #删除指定的容器
[root@localhost ~]# docker rmi -f 容器ID 容器ID 容器ID #批量删除容器
[root@localhost ~]# docker rmi -f $(docker images -aq) #批量删除容器
```

## 容器命令

> 说明：我们有了镜像才可以创建容器，linux，下载一个centos镜像来测试学习

```bash
docker pull centos        
```

新建容器并启动

```bash
docker run [可选参数] image

# 参数说明
--name="Name"    容器名字 tomcat01 tomcat02，用来区分容器
-d                后台方式运行
-it                使用交互方式运行，进入容器查看内容
-p                指定容器的端口 -p 8080:8080
    -p    ip:主机端口:容器端口
    -p    主机端口:容器端口(常用)
    -p    容器端口
    容器端口
-p                随机指定端口

# 测试 启动并进入容器
docker run -it centos /bin/bash
[root@localhost /]# docker run -it centos /bin/bash
[root@b57c4eb8d3b4 /]# ls # 查看容器内的centos 基础版本，很多命令都是不完善的
bin  dev  etc  home  lib  lib64  lost+found  media  mnt  opt  proc  root  run  sbin  srv  sys  tmp  usr  var

# 从容其中退回主机
[root@b57c4eb8d3b4 /]# exit
exit
[root@localhost /]# ls
Applications  boot  dev  home  lib64  mnt  proc  run   srv  tmp  var
bin           data  etc  lib   media  opt  root  sbin  sys  usr
```

**列出所有正在运行的容器**

```bash
# docker ps 命令
   # 列出当前正在运行的容器
-a # 列出曾经运行过的容器
-n=? # 显示最近创建的容器
-q # 只显示容器的编号

[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
You have new mail in /var/spool/mail/root
[root@localhost /]# docker ps -a
CONTAINER ID   IMAGE         COMMAND       CREATED          STATUS                        PORTS     NAMES
b57c4eb8d3b4   centos        "/bin/bash"   56 minutes ago   Exited (130) 15 minutes ago             adoring_herschel
d88793e060ec   hello-world   "/hello"      4 hours ago      Exited (0) 4 hours ago                  quirky_meninsky
7bd1df5679f3   hello-world   "/hello"      4 hours ago      Exited (0) 4 hours ago                  great_satoshi
ce120eb6b569   hello-world   "/hello"      5 hours ago      Exited (0) 5 hours ago                  jolly_ritchie
```

**退出容器**

```ba
exit # 直接容器退出
Ctrl + P + Q #容器不停止退出
[root@localhost /]# docker run -it centos /bin/bash
[root@8066da4521e3 /]# [root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
8066da4521e3   centos    "/bin/bash"   17 seconds ago   Up 17 seconds             naughty_engelbart
639d14d7aec3   centos    "/bin/bash"   30 seconds ago   Up 29 seconds             awesome_hellman
[root@localhost /]#
```

**删除容器**

```bash
docker rm 容器ID # 删除指定容器  不能删除正在运行的容器  -f 可以强行删除
docker rm -f $(docker ps -aq) # 删除所有容器 
docker ps -a -q | xargs docker rm # 删除所有容器 
```

**启动和停止**

```bash
docker start 容器id
docker restart 容器id
docker stop 容器id
docker kill 容器id
```

## 常用其他命令

**后台启动容器**

```bash
# 命令 docker run -d 镜像名
[root@localhost /]# docker run -d centos
d267786cc3dd79b0b4316954a0720707ecc5177e16686e6f4c5a623d01edac9c

# 问题 docker ps 发现centos 停止了

# 常见的坑 docker 容器使用后台运行，就必须要有一个前台进程。docker发现没有应用，就会自动停止
# nginx 容器启动后，发现自己没有提供服务，就会立即停止，就是没有程序了            
```

**查看日志**

```bash
docker logs -f -t --tail 容器，没有日志

# 自己编写一段shell脚本
"while true;do echo abc;sleep 1;done"

docker run -d centos /bin/sh -c "while true;do echo abc;sleep 1;done"
a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS        PORTS     NAMES
a01f18c02028   centos    "/bin/sh -c 'while t…"   2 seconds ago   Up 1 second             interesting_visvesvaraya

# 显示日志
-tf # 显示日志
--tail number # 显示日志条数
root@localhost /]# docker logs -f -t --tail 10  a01f18c02028
2021-06-25T07:24:26.380555173Z abc
2021-06-25T07:24:27.385373738Z abc
2021-06-25T07:24:28.387426503Z abc
2021-06-25T07:24:29.392712447Z abc
2021-06-25T07:24:30.395028922Z abc
2021-06-25T07:24:31.398863425Z abc
2021-06-25T07:24:32.401880781Z abc
2021-06-25T07:24:33.407703428Z abc
2021-06-25T07:24:34.411654343Z abc
2021-06-25T07:24:35.416725109Z abc
2021-06-25T07:24:36.420730978Z abc
2021-06-25T07:24:37.423037029Z abc
2021-06-25T07:24:38.428198303Z abc
```

**查看容器重进程信息 **ps

```bash
[root@localhost /]# docker top a01f18c02028
UID                 PID                 PPID                C                   STIME               TTY                 TIME                CMD
root                40580               40560               0                   00:23               ?                   00:00:00            /bin/sh -c while true;do echo abc;sleep 1;done
root                41188               40580               0                   00:28               ?                   00:00:00            /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/sleep 1
```

**查看镜像的源数据**

```bash
# 命令
docker inspect 容器ID
[root@localhost /]# docker inspect a01f18c02028
[
    {
        "Id": "a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69",
        "Created": "2021-06-25T07:23:48.851080754Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true;do echo abc;sleep 1;done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 40580,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2021-06-25T07:23:49.21616444Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
        "Image": "sha256:300e315adb2f96afe5f0b2780b87f28ae95231fe3bdd1e16b9ba606307728f55",
        "ResolvConfPath": "/var/lib/docker/containers/a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69/resolv.conf",
        "HostnamePath": "/var/lib/docker/containers/a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69/hostname",
        "HostsPath": "/var/lib/docker/containers/a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69/hosts",
        "LogPath": "/var/lib/docker/containers/a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69/a01f18c02028a82a18d1a29b1a27397292f4621a4a365068389c234c98226b69-json.log",
        "Name": "/interesting_visvesvaraya",
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
                "LowerDir": "/var/lib/docker/overlay2/1ce659a6b62b9112f176ee07fd72265459ac054a59332678ea112907a0e7a75f-init/diff:/var/lib/docker/overlay2/bbdf8d4647beacf80398f3d8d0a6d53d9f990e329cd388a4b5c921eb1f4238f8/diff",
                "MergedDir": "/var/lib/docker/overlay2/1ce659a6b62b9112f176ee07fd72265459ac054a59332678ea112907a0e7a75f/merged",
                "UpperDir": "/var/lib/docker/overlay2/1ce659a6b62b9112f176ee07fd72265459ac054a59332678ea112907a0e7a75f/diff",
                "WorkDir": "/var/lib/docker/overlay2/1ce659a6b62b9112f176ee07fd72265459ac054a59332678ea112907a0e7a75f/work"
            },
            "Name": "overlay2"
        },
        "Mounts": [],
        "Config": {
            "Hostname": "a01f18c02028",
            "Domainname": "",
            "User": "",
            "AttachStdin": false,
            "AttachStdout": false,
            "AttachStderr": false,
            "Tty": false,
            "OpenStdin": false,
            "StdinOnce": false,
            "Env": [
                "PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
            ],
            "Cmd": [
                "/bin/sh",
                "-c",
                "while true;do echo abc;sleep 1;done"
            ],
            "Image": "centos",
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
            "SandboxID": "35ce57dad7db28ce0cce7e31cc91da80e5f3e37d82aac77f17114070bbcf51cc",
            "HairpinMode": false,
            "LinkLocalIPv6Address": "",
            "LinkLocalIPv6PrefixLen": 0,
            "Ports": {},
            "SandboxKey": "/var/run/docker/netns/35ce57dad7db",
            "SecondaryIPAddresses": null,
            "SecondaryIPv6Addresses": null,
            "EndpointID": "3704db025e2a0adcaa4646e492f5738eb1604e6ef91a2e4da247d6527e4a13de",
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
                    "NetworkID": "ff38593198097ea96091cc6fb96038d598897789f1c1b5126ce3ff61e228a42b",
                    "EndpointID": "3704db025e2a0adcaa4646e492f5738eb1604e6ef91a2e4da247d6527e4a13de",
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
```

**进入当前正在运行的容器**

```bash
# 我们通常容器都是使用后台方式运行的，需要进入容器，需要修改一些配置

# 命令
docker exec -it 容器id bashShell

# 测试
[root@localhost /]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS     NAMES
a01f18c02028   centos    "/bin/sh -c 'while t…"   9 minutes ago   Up 9 minutes             interesting_visvesvaraya
You have new mail in /var/spool/mail/root
[root@localhost /]# docker exec -it a01f18c02028 /bin/bash
[root@a01f18c02028 /]# ps -ef
UID         PID   PPID  C STIME TTY          TIME CMD
root          1      0  0 07:23 ?        00:00:00 /bin/sh -c while true;do echo abc;sleep 1;done
root        596      0  0 07:33 pts/0    00:00:00 /bin/bash
root        620      1  0 07:33 ?        00:00:00 /usr/bin/coreutils --coreutils-prog-shebang=sleep /usr/bin/s
root        621    596  0 07:33 pts/0    00:00:00 ps -ef


# 方式二
docker attach 容器ID

# 测试
[root@localhost ~]# docker attach a01f18c02028
正在执行当前的代码....


# docker exec   # 进入容器后开启一个新的终端，可以在里面操作（常用）
# docker attach # 进入容器正在执行的终端，不会启动新的进程
```

**从容器内拷贝文件到主机中**

```bash
docker cp 容器id:容器内路径   目的的主机路径 

# 查看当前主机目录下
[root@localhost ~]# cd /home/
[root@localhost home]# ls
elastic  root1
[root@localhost home]# docker ps
CONTAINER ID   IMAGE     COMMAND       CREATED          STATUS          PORTS     NAMES
0326384667af   centos    "/bin/bash"   44 seconds ago   Up 42 seconds             xenodochial_hofstadter

# 进入docker容器内部
[root@localhost home]# docker attach 0326384667af
[root@0326384667af /]#  cd /home
[root@0326384667af home]# ls
# 再容器内新建一个文件
[root@0326384667af home]# touch test.java
[root@0326384667af home]# ls
test.java
[root@0326384667af home]# exit
exit
You have new mail in /var/spool/mail/root
[root@localhost home]# docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
[root@localhost home]# docker ps -a
CONTAINER ID   IMAGE     COMMAND       CREATED              STATUS                     PORTS     NAMES
0326384667af   centos    "/bin/bash"   About a minute ago   Exited (0) 5 seconds ago             xenodochial_hofstadter
# 将这个文件拷贝到主机上
[root@localhost home]# docker cp 0326384667af:/home/test.java /home/
[root@localhost home]#
[root@localhost home]# ls
abc.java  elastic  root1  test.java


# 拷贝是一个手动过程，未来我们使用 -v 卷的技术， 可以实现
```

## 小结

![image20210625155153641](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210625155153641.13btvklxon1s.png)

## Docker 命令大全

```bash
  attach      当前shell下attch链接指定运行镜像
  build       通过Dockerfile定制映像
  commit      提交当前容器为新的镜像
  cp          从容器中拷贝指定文件或者目录到宿主机中
  create      创建一个新的容器，同run，但不启动容器
  diff        查看docker容器变化
  events      从docker服务获取容器实时事件
  exec        在已存在的容器上运行命令
  export      导出容器的内容流作为一个tar归档文件【对应import】
  history     展示一个镜像形成历史
  images      列出系统当前镜像
  import      从tar包中的内容创建一个新的文件系统镜像
  info        显示系统相关信息
  inspect     查看容器详细信息
  kill        kill指定docker容器
  load        从一个tar包中加载一个镜像【对应save】
  login       注册或者登陆一个docker源服务器
  logout      从当前docker registry退出
  logs        输出当前容器日志信息
  pause       暂停容器
  port        查看映射端口对应的容器内部源端口
  ps          列出容器列表
  pull        从docker镜像源服务器拉取指定镜像或者库镜像
  push        推送指定镜像或者库镜像至docker源服务器
  rename      重命名一个容器
  restart     重新启动一个或多个容器
  rm          移除一个或多个容器
  rmi         移除一个或多个镜像【无容器使用该镜像才可以删除，否则需要删除相关容器才可以继续或 -f 强制删除】
  run         创建一个新的容器并运行一个命令
  save        保存一个镜像为一个tar包【对应load】
  search      再docker hub 重搜索镜像
  start       启动容器
  stats       显示容器资源使用统计的实时流
  stop        停止容器
  tag         给源重镜像打标签
  top         查看容器中运行的进程信息
  unpause     取消暂定容器
  update      更新一个或多个容器的配置
  version     查看版本号
  wait        做取容器停止时的退出状态值
```

## 安装 nginx

```bash
[root@localhost home]# docker serach nginx
[root@localhost home]# docker pull nginx
[root@localhost home]# docker images
REPOSITORY    TAG       IMAGE ID       CREATED        SIZE
nginx         latest    4f380adfc10f   2 days ago     133MB
mysql         latest    5c62e459e087   2 days ago     556MB
hello-world   latest    d1165f221234   3 months ago   13.3kB
centos        latest    300e315adb2f   6 months ago   209MB
# -d 后台 --name 给容器命名  -p 宿主机端口:容器端口   镜像
[root@localhost home]# docker run -d --name nginx01 -p 3344:80 nginx
ed81afedeef714506184db9bade227891b6e98ee958f626dc22d35069cf53e92
You have new mail in /var/spool/mail/root
[root@localhost home]# docker ps
CONTAINER ID   IMAGE     COMMAND                  CREATED         STATUS         PORTS                                   NAMES
ed81afedeef7   nginx     "/docker-entrypoint.…"   4 seconds ago   Up 3 seconds   0.0.0.0:3344->80/tcp, :::3344->80/tcp   nginx01
# 测试
[root@localhost home]# curl localhost:3344
公网 访问http://192.168.240.131:3344/

# 进入容器
[root@localhost home]# docker exec -it nginx01 /bin/bash
root@ed81afedeef7:/# whereis nginx
nginx: /usr/sbin/nginx /usr/lib/nginx /etc/nginx /usr/share/nginx
root@ed81afedeef7:/# cd /etc/nginx/
root@ed81afedeef7:/etc/nginx#
root@ed81afedeef7:/etc/nginx# ls
conf.d  fastcgi_params  mime.types  modules  nginx.conf  scgi_params  uwsgi_params
root@ed81afedeef7:/etc/nginx#
```

端口暴露的概念

![image20210625161813431](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210625161813431.5mdyi5p4fxs0.png)

思考问题∶我们每次改动nginx配置文件，都需要进入容器内部?十分的麻烦，我要是可以在容器外部提供一个映射路径，达到在容器修改文件名，容器内部就可以自动修改? -v 数据卷!

## 安装 tomcat

```bash
# 官方的使用
docker run -it --rm tomcat:9.0

# 我们之前的启动都是后台，停止了容器之后，容器还是可以查到 docker run -it --rm tomcat:9.0 一般用来测试，用完即删

# 下载再启动
docker pull tomcat

# 启动运行
docker run -d -p 3355:8080 --name tomcat01 tomcat

# 测试访问没有问题

# 进入容器
[root@localhost home]# docker exec -it tomcat01 /bin/bash

# 发现问题：1 linux命令太少 2.没有webapps 
# 保证最小可运行的环境
```

## 安装es+kibana

```bash
# es 暴露的端口很多
# es 十分耗内存
# es 的数据一般需要放置到安全目录 挂载

# --net somenetwoker ? 网络配置

# 启动es
docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" elasticsearch:7.13.2


# 启动了linux就卡住了  dcoker stats 查看 cpu状态

# 查看 docker stats

# 测试
[root@localhost home]#  curl localhost:9200
{
  "name" : "a068ccb5112a",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "E8_VUE5KQTm6cjblW4CdPQ",
  "version" : {
    "number" : "7.13.2",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "4d960a0733be83dd2543ca018aa4ddc42e956800",
    "build_date" : "2021-06-10T21:01:55.251515791Z",
    "build_snapshot" : false,
    "lucene_version" : "8.8.2",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}

# 赶紧关闭，增加内存的限制 修改配置文件 -e 环境配置修改
docker run -d --name elasticsearch02 -p 9200:9200 -p 9300:9300 -e ES_JAVA_OPTS="-Xms64m -Xmx512m" -e "discovery.type=single-node" elasticsearch:7.13.2
```

使用kibana链接es？思考网络如何才能连接过去

![image20210625171337104](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210625171337104.66wxibqavig0.png)

## 可视化

**portainer**

```bash
docker run -d -p 8088:9000 \
--restart=always -v /var/run/docker.sock:/var/run/docker.sock --privileged=true portainer/portainer
```

访问测试:http://192.168.240.131:8088/

![image20210625173943064](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210625173943064.5q2al4glxz80.png)

# Docker镜像讲解

## commit镜像

```shell
docker commit 提交容器成为一个新的副本

# 命令和git原理类似
docker commit -m='提交的描述信息' -a='作者' 容器id 目标敬香茗：[TAG]
```

实战测试

```shell
# 1.启动一个默认的tomcat

# 2.发现这个默认的tomcsat 是没有webapps应用，镜像的原因，官方的景象默认 webapps下面是没有文件的

# 3.我自己拷贝进去了基本的文件

# 4.将我们操作过的容器通过commit提交为一个镜像，我们以后就是用我们修改过的镜像即可，这就是我们自己的一个修改的镜像
```

![image20210628172730264](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210628172730264.4bni57ct4po0.png)

```shell
如果你想要保存当前容器的状态，就可以通过commit来提交，获得一个镜像
就好比我们以前学习VM的时候，快照
```

到这里才算是入门

# 容器数据卷

## 什么事容器数据卷

**docker的理念回顾**

将应用和环境打包成一个镜像

数据？如果数据都在容器中，那么我们容器删除，数据将会丢失 **需求：数据可以持久化**

Mysql，容器删了，删库跑路。 **需求：mysql数据可以存储在本地**

容器之间可以有一个数据共享的技术，docker容器中产生的数据，同步到本地

这就是卷技术，目录的挂在，将我们容器内的目录，挂载到linux上面

![image20210628174622225](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210628174622225.1suzoul05juo.png)

**总结一句话：容器的持久化和同步操作，容器间也是可以数据共享的**

## 使用数据卷

> 方式一：直接使用命令来挂载 -v

```shell
docker run -it -v 主机目录：容器内目录

# 测试
[root@localhost ~]# cd /home/
[root@localhost home]# ls
abc.java  elastic  root1  test.java
[root@localhost home]# docker run -it -v /home/test:/home/ centos /bin/bash

# 启动起来的时候我们可以通过 docker inspect 容器id
```

![image20210628180027068](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210628180027068.7c6r3eq0d380.png)

测试文件的同步

![image20210628180224155](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210628180224155.572usmnr2c00.png)

![image20210628180230124](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210628180230124.4tprdcj2vyg0.png)

再来测试

1. 停止容器
2. 宿主机上修改文件
3. 启动容器
4. 容器内的数据依旧是同步的

好处：我们以后修改只需要在本地修改即可，容器内会自动同步

## 实战MySQL

思考：Mysql的数据持久化问题

```shell
# 获取镜像
docker pull mysql:5.7

# 运行容器，需要做数据挂载  # 安装启动mysql，需要配置密码的。
# 官方测试
$ docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
# 启动
-d 后台运行
-p 端口映射
-v 卷挂载
-e 环境配置
--name  容器名字
[root@localhost test]# docker run -d -p 3310:3306 -v /home/mysql/conf:/etc/mysql/conf.d -v /home/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql01 mysql:5.7


# 启动成功之后，我们在本地使用navicat连接测试
# navicat连接到服务器的3310  ---- 3310和容器内的3306映射，这个时候我们就可以连接上了

# 在本地测试创建一个数据库，查看一下我们映射的路径是否ok
```

假设我们将容器删除

发现，我们挂载到本地的数据卷依旧没有丢失，这就实现了容器数据持久化的功能

![image20210629092932038](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629092932038.6kdrmrg7stg0.png)

## 具名和 匿名挂载

```shell
# 建议使用具名挂载

# 匿名挂载
-P 随机映射
-v 容器内的路径
[root@localhost data]# docker run -d -P --name nginx02 -v /etc/nginx nginx
# 查看所有的volume情况
[root@localhost data]# docker volume ls
DRIVER    VOLUME NAME
local     35bd8a161b962620ea47f660664701c704fc8bae25a4ad7179d9bce8952746bf
local     b09f887966023130ee85f9c8958a7b69c8b40fd7b2e6dce3a3773df01f6ba5cf

# 这里发现，这种就是匿名挂载，我们在 -v 只有写了容器内的路径，没有写容器外的路径

# 具名挂载
[root@localhost data]# docker run -d -P --name nginx03 -v juming-nginx:/etc/nginx nginx
50272f657e3fcf1a23272bf33bb8724a6f3c2691073cc918d4ecce79c4ff1945
You have new mail in /var/spool/mail/root
[root@localhost data]# docker volume ls
DRIVER    VOLUME NAME
local     35bd8a161b962620ea47f660664701c704fc8bae25a4ad7179d9bce8952746bf
local     b09f887966023130ee85f9c8958a7b69c8b40fd7b2e6dce3a3773df01f6ba5cf
local     juming-nginx

# 通过 -v 卷名：容器内路径
# 查看一下这个卷
[root@localhost data]# docker volume inspect juming-nginx
[
    {
        "CreatedAt": "2021-06-28T18:40:42-07:00",
        "Driver": "local",
        "Labels": null,
        "Mountpoint": "/var/lib/docker/volumes/juming-nginx/_data", # 主机路径
        "Name": "juming-nginx",
        "Options": null,
        "Scope": "local"
    }
]
[root@localhost data]#
```

所有的docker容器内的卷，没有指定目录的情况下都是在`/var/lib/docker/volume/xxx/_data`

我们通过具名挂载可以方便的找到我们的一个卷，大多数情况再使用的`具名挂载`

```shell
# 如何确定是具名挂载还是匿名挂在，还是指定路径挂载
-v 容器内路径 # 匿名挂载
-v 卷名：容器内路径 # 具名挂载
-v /宿主机路径：容器内路径 # 指定路径挂载
```

拓展：

```shell
# 通过 -v 容器内的路径：ro rw 改变读写权限
ro readonly # 只读
rw readwrite # 可读可写

# 一旦这个设置了容器权限，容器对我们挂在出来的内容就有限定了
[root@localhost _data]# docker run -d -P --name nginx04 -v juming-ngixn:/etc/nginx:ro nginx
[root@localhost _data]# docker run -d -P --name nginx04 -v juming-ngixn:/etc/nginx:rw nginx

# ro 只要看到ro这就说明了这个路径只能通过宿主机来操作，容器内部是无法操作的
```

## 初识Dockerfile

Dockerfile 就是用来构建docker镜像的构建文件。 命令脚本，先来体验一下

通过这个脚本可以生成镜像，镜像是一层一层的，脚本一个个的命令，每个命令都是一层

```shell
# 创建一个dockerfile文件，名字可以随机 建议dockerifle
# 文件中的内容 指令（大写） 参数
[root@localhost docker-test-volume]# pwd
/home/docker-test-volume
[root@localhost docker-test-volume]# vim dockerfile1
You have new mail in /var/spool/mail/root
[root@localhost docker-test-volume]# cat dockerfile1
FROM centos
VOLUME ["/volume01", "/volume02"] # 匿名挂载
CMD echo "----end----"
CMD /bin/bash

# 这里的每个命令，就是一层一层的镜像


[root@localhost docker-test-volume]# docker build -f dockerfile1 -t lh/centos:1.1 .
Sending build context to Docker daemon  2.048kB
Step 1/4 : FROM centos
 ---> 300e315adb2f
Step 2/4 : VOLUME ["/volume01", "/volume02"]
 ---> Running in ccd7105825f2
Removing intermediate container ccd7105825f2
 ---> 420469c0ffed
Step 3/4 : CMD echo "----end----"
 ---> Running in f2fcf7700844
Removing intermediate container f2fcf7700844
 ---> a15b37af0524
Step 4/4 : CMD /bin/bash
 ---> Running in c6abbf588738
Removing intermediate container c6abbf588738
 ---> 8bd96f852c19
Successfully built 8bd96f852c19
Successfully tagged lh/centos:1.1
```

```shell
# 启动自己写的容器
[root@localhost docker-test-volume]# docker run -it 8bd96f852c19 /bin/bash
[root@8627375f2a88 /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var       volume02
dev  home  lib64  media       opt  root  sbin  sys  usr  volume01

volume01 volume02 这个目录就是我们生成镜像的时候自动改在的，数据卷目录
```

这个卷和外部一定有一个同步的目录

```shell
[root@localhost docker-test-volume]# docker run -it 8bd96f852c19 /bin/bash
[root@ebadcf29a00f /]# cd volume01
[root@ebadcf29a00f volume01]# touch container.txt
[root@ebadcf29a00f volume01]# ls
container.txt
[root@ebadcf29a00f volume01]# exit
exit
[root@localhost docker-test-volume]# docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED             STATUS             PORTS                                     NAMES
5fcbb050d4f5   8bd96f852c19   "/bin/bash"              30 minutes ago      Up 30 minutes                                                keen_hopper
You have new mail in /var/spool/mail/root
[root@localhost docker-test-volume]# docker inspect 5fcbb050d4f5
```

查看一下卷挂载的路径

![image20210629103630550](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629103630550.l39sfhschds.png)

```shell
[root@localhost docker-test-volume]# cd /var/lib/docker/volumes/c870b50d3c8371078d54547c666a5542c327ba71df30e63c9446c8214d344419/_data
You have new mail in /var/spool/mail/root
[root@localhost _data]# ls
container.ext
```

测试一下刚才的文件是否同步出去了

这种方式我们未来使用的是分舵，因为我们通常会构建自己的镜像

假设构建镜像的时候没有挂在卷，要手动镜像挂载， -v 卷名：容器内路径

## 数据卷容器

多个mysql同步数据

![image20210629105754453](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629105754453.35hvmaujun00.png)

```shell
# 启动三个容器，通过我们刚才自己写的镜像启动
```

启动docker01

```shell
[root@localhost _data]# docker run -it --name docker01 test/centos:1.0
[root@9c626ae570bb /]# ls
bin  etc   lib    lost+found  mnt  proc  run   srv  tmp  var       volume02
dev  home  lib64  media       opt  root  sbin  sys  usr  volume01
[root@9c626ae570bb /]#
```

双向绑定，类似mysql主从复制

```shell
# docker02 挂载docker01 ，这样就可以实现同步了
# 只要通过 --volumes-from 就可以实现同步
[root@localhost _data]# docker run -it --name docker02 --volumes-from docker01 test/centos:1.0
```

```shell
# 测试：可以删除docker01，查看一下docker02和docker03是否还可以访问这个文件
# 测试依旧可以访问
```

![image20210629112553046](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629112553046.1ng89k10o15s.png)

多个mysql实现数据共享

```shell
[root@localhost test]# docker run -d -p 3310:3306 -v /etc/mysql/conf.d -v /var/lib/mysql -e MYSQL_ROOT_PASSWORD=root --name mysql01 mysql:5.7

[root@localhost test]# docker run -d -p 3310:3306 -e MYSQL_ROOT_PASSWORD=root --name mysql02 volumes-from mysql01 mysql:5.7

# 这个时候，可以实现两个容器数据同步
```

**结论**

容器之间配置信息的传递，数据卷容器的生命周期一直持续到没有容器使用为止

但是一旦你持久化到了本地，这个时候，本地的数据是不会丢失的

# DockerFile

dockerfile是用来构建docker镜像的文件，命令参数脚本

构建步骤：

1 编写一个dockerfile文件

2 docker build 构建成为一个镜像

3 docker run 运行镜像

4 docker push 发布镜像（dockerhub aliyun镜像仓库）

## DockerFile 构建过程

**基础知识**

1. 每个保留关键字（指令）都是必须是大写字母
2. 执行从上到下的顺序执行
3. #表示注释
4. 每个指令都会创建提交一个新的镜像层，并提交

![image20210629114559685](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629114559685.2gc6bmz14al.png)

dockerfile是面向开发的，我们以后要发布项目，做镜像，就需要编写dockerifle文件，这个文件十分简单

Docker镜像逐渐成为企业交付的标准，必须要掌握

步骤：开发 部署 运维 缺一不可

Dockerfile：构建文件，定义了一切的步骤，源代码

DockerImages：通过DockerFile构建生成了镜像，最终发布和运行的产品

Docker容器：容器就是镜像运行起来提供的容器

## DockerFile的指令

以前的话我们就是是用别人的，现在我们知道了这些指令后，我们来练习自己写一个镜像

```shell
FROM             # 基础镜像，且从这里开始构建 centos
MAINTAINER          # 镜像是谁写的，姓名+邮箱
RUN             # 镜像构建的时候需要运行的命令
ADD             # 步骤，tomcat镜像，这个tomcat压缩包就是添加的内容
WORKDIR         # 镜像的工作目录
VOLUME             # 挂在的目录
EXPOSE             # 保留端口配置


CMD             # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT         # 指定这个容器启动的时候要运行的命令，可以追加命令
ONBUILD         # 当构建一个被继承 DockerFile 这个时候就会运行 ONBUILD 的指令，触发指令
COPY            # 类似ADD命令，将我们的文件拷贝到镜像中
ENV                # 构建的时候设置环境变量
```

![image20210629134941709](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629134941709.4japlc3hlz00.png)

## 实战测试

DockerHub中99%镜像都是从这个基础镜像中FROM scratch，然后配置需要的软件配置来进行的构建

##

dockerhub中centos 镜像dockerfile

```shell
FROM scratch
ADD centos-8-x86_64.tar.xz /
LABEL org.label-schema.schema-version="1.0"     org.label-schema.name="CentOS Base Image"     org.label-schema.vendor="CentOS"     org.label-schema.license="GPLv2"     org.label-schema.build-date="20201204"
CMD ["/bin/bash"]
```

> 创建一个自己的centos

```shell
# 1.编写Dockerfile的文件
[root@localhost dockerfile]# cat mydockerfile
FROM centos
MAINTAINER 4927525<hzbskak@gmai.com>

ENV MYPATH /usr/local
WORKDIR $MYPATH

RUN yum -y install vim
RUN yum -y install net-tools

EXPOSE 80

CMD echo $MYPATH

CMD echo "---end---"

CMD /bin/bash

# 2.通过这个文件构建镜像
# 命令 docker build -f dockerfile文件路径 -t 镜像名-
[root@localhost dockerfile]# docker build -f mydockerfile -t mycentos:0.1 .

Successfully built ffb9077ccb22
Successfully tagged mycentos:0.1
# 3. 测试运行
[root@localhost dockerfile]# docker run -it mycentos:0.1
```

我们可以列出本地镜像的变更历史

```shell
[root@localhost dockerfile]# docker history ffb9077ccb22
IMAGE          CREATED         CREATED BY                                      SIZE      COMMENT
ffb9077ccb22   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "/bin…   0B
223fa2f4bde3   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B
bef017bd3047   5 minutes ago   /bin/sh -c #(nop)  CMD ["/bin/sh" "-c" "echo…   0B
85fcbce5fd8b   5 minutes ago   /bin/sh -c #(nop)  EXPOSE 80                    0B
7084b37fb63a   5 minutes ago   /bin/sh -c yum -y install net-tools             25.8MB
9c34db8acb38   5 minutes ago   /bin/sh -c yum -y install yum                   47.5MB
7feb4f1278c9   5 minutes ago   /bin/sh -c #(nop) WORKDIR /usr/local            0B
c5c9a67494a5   5 minutes ago   /bin/sh -c #(nop)  ENV MYPATH=/usr/local        0B
0a71ab45d13a   5 minutes ago   /bin/sh -c #(nop)  MAINTAINER 4927525<hzbska…   0B
300e315adb2f   6 months ago    /bin/sh -c #(nop)  CMD ["/bin/bash"]            0B
<missing>      6 months ago    /bin/sh -c #(nop)  LABEL org.label-schema.sc…   0B
<missing>      6 months ago    /bin/sh -c #(nop) ADD file:bd7a2aed6ede423b7…   209MB
```

我们平时拿到一个镜像，可以研究一下他是怎么做到的

> CMD和 ENTRYPOINT的区别

```shell
CMD             # 指定这个容器启动的时候要运行的命令，只有最后一个会生效，可被替代
ENTRYPOINT         # 指定这个容器启动的时候要运行的命令，可以追加命令
```

测试cmd

```shell
# 编写 dockerfile文件
[root@localhost dockerfile]# vim dockerfile-cmd-test
FROM centos
CMD ["ls","-a"]
# 构建镜像
[root@localhost dockerfile]# docker build -f dockerfile-cmd-test -t cmdtest .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM centos
 ---> 300e315adb2f
Step 2/2 : CMD ["ls","-a"]
 ---> Running in 3b7ae0862736
Removing intermediate container 3b7ae0862736
 ---> d7fe8935d69a
Successfully built d7fe8935d69a
Successfully tagged cmdtest:latest
You have new mail in /var/spool/mail/root
# run 运行
[root@localhost dockerfile]# docker run d7fe8935d69a
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var

# 项追加一个命令 -l ls -al
[root@localhost dockerfile]# docker run d7fe8935d69a -l
docker: Error response from daemon: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "-l": executable file not found in $PATH: unknown.
ERRO[0000] error waiting for container: context canceled

# cmd的命令下 -l 替换了CMD["ls","-a"] 命令， -l 不是命令所以报错
[root@localhost dockerfile]# docker run d7fe8935d69a ls -al
total 0
drwxr-xr-x.   1 root root   6 Jun 29 06:28 .
drwxr-xr-x.   1 root root   6 Jun 29 06:28 ..
-rwxr-xr-x.   1 root root   0 Jun 29 06:27 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jun 29 06:28 dev
drwxr-xr-x.   1 root root  66 Jun 29 06:27 etc
drwxr-xr-x.   2 root root   6 Nov  3  2020 home
lrwxrwxrwx.   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4  2020 lost+found
drwxr-xr-x.   2 root root   6 Nov  3  2020 media
drwxr-xr-x.   2 root root   6 Nov  3  2020 mnt
drwxr-xr-x.   2 root root   6 Nov  3  2020 opt
dr-xr-xr-x. 159 root root   0 Jun 29 06:28 proc
dr-xr-x---.   2 root root 162 Dec  4  2020 root
drwxr-xr-x.  11 root root 163 Dec  4  2020 run
lrwxrwxrwx.   1 root root   8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3  2020 srv
dr-xr-xr-x.  13 root root   0 Jun 17 02:57 sys
drwxrwxrwt.   7 root root 145 Dec  4  2020 tmp
drwxr-xr-x.  12 root root 144 Dec  4  2020 usr
drwxr-xr-x.  20 root root 262 Dec  4  2020 var
```

测试 ENTRYPONIT

```shell
# 编写 dockerfile文件
[root@localhost dockerfile]# vim dockerfile-entrypoint-test
FROM centos
ENTRYPOINT ["ls", "-a"]
# 构建镜像
[root@localhost dockerfile]# docker build -f dockerfile-entrypoint-test -t entrypoint-test .
Sending build context to Docker daemon  4.096kB
Step 1/2 : FROM centos
 ---> 300e315adb2f
Step 2/2 : ENTRYPOINT ["ls", "-a"]
 ---> Running in e6cabfa40cbf
Removing intermediate container e6cabfa40cbf
 ---> 36f3e7745ea3
Successfully built 36f3e7745ea3
Successfully tagged entrypoint-test:latest
# run 运行
[root@localhost dockerfile]# docker run 36f3e7745ea3
.
..
.dockerenv
bin
dev
etc
home
lib
lib64
lost+found
media
mnt
opt
proc
root
run
sbin
srv
sys
tmp
usr
var
# 我们的追加命令，是直接拼接在我们的 ENTRYPOINT 命令的后面
[root@localhost dockerfile]# docker run 36f3e7745ea3 -l
total 0
drwxr-xr-x.   1 root root   6 Jun 29 06:22 .
drwxr-xr-x.   1 root root   6 Jun 29 06:22 ..
-rwxr-xr-x.   1 root root   0 Jun 29 06:22 .dockerenv
lrwxrwxrwx.   1 root root   7 Nov  3  2020 bin -> usr/bin
drwxr-xr-x.   5 root root 340 Jun 29 06:22 dev
drwxr-xr-x.   1 root root  66 Jun 29 06:22 etc
drwxr-xr-x.   2 root root   6 Nov  3  2020 home
lrwxrwxrwx.   1 root root   7 Nov  3  2020 lib -> usr/lib
lrwxrwxrwx.   1 root root   9 Nov  3  2020 lib64 -> usr/lib64
drwx------.   2 root root   6 Dec  4  2020 lost+found
drwxr-xr-x.   2 root root   6 Nov  3  2020 media
drwxr-xr-x.   2 root root   6 Nov  3  2020 mnt
drwxr-xr-x.   2 root root   6 Nov  3  2020 opt
dr-xr-xr-x. 162 root root   0 Jun 29 06:22 proc
dr-xr-x---.   2 root root 162 Dec  4  2020 root
drwxr-xr-x.  11 root root 163 Dec  4  2020 run
lrwxrwxrwx.   1 root root   8 Nov  3  2020 sbin -> usr/sbin
drwxr-xr-x.   2 root root   6 Nov  3  2020 srv
dr-xr-xr-x.  13 root root   0 Jun 17 02:57 sys
drwxrwxrwt.   7 root root 145 Dec  4  2020 tmp
drwxr-xr-x.  12 root root 144 Dec  4  2020 usr
drwxr-xr-x.  20 root root 262 Dec  4  2020 var
[root@localhost dockerfile]#
```

DockerFile中很多命令都十分的相似，我们需要了解他们的区别，进行测试

## 实战:Tomcat镜像

1. 准备镜像文件 tomcat 压缩包，jdk的压缩包
  
  ```shell
  [root@localhost build]# ls
  apache-tomcat-9.0.48.tar.gz  jdk-8u11-linux-x64.tar.gz
  [root@localhost build]# touch read.txt
  [root@localhost build]# ls
  apache-tomcat-9.0.48.tar.gz  jdk-8u11-linux-x64.tar.gz  read.txt
  [root@localhost build]# pwd
  /home/hzbskak/build
  ```
  
2. 编写一个Dockefile文件， 官方命令 `Dockerfile`，build 会自动寻找这个文件，就不要 -f 指定了
  

```shell
FROM centos
MAINTAINER hzbskak<hzbskak@gmail.com>

COPY readme.txt /usr/local/readme.txt

ADD jdk-8u11-linux-x64.tar.gz /usr/local/
ADD apache-tomcat-9.0.48.tar.gz /usr/local/

RUN yum -y install vim

ENV MYPATH /usr/local
WORKDIR $PATH

ENV JAVA_HOME /usr/local/jdk1.8.0_11
ENV CLASSPATH $JAVA_HOME/lib/dt.jar:$JAVA_HOME/ib/tools.jar
ENV CATALINA_HOME /usr/local/apache-tomcat-9.0.48
ENV CATALINA_BASH /usr/local/apache-tomcat-9.0.48
ENV PATH $PATH:$JAVA_HOME/bin:$CATALINA_HOME/lib$CATALINA_HOME/bin

EXPOSE 8080

CMD /usr/local/apache-tomcat-9.0.48/bin/startup.sh && tail -F /usr/local/apache-tomcat-9.0.48/bin/logs/catalina.out
```

3. 构建镜像
  
  ```shell
  [root@localhost build]# docker build -t diytomcat .
  Successfully built a5a99033ad77
  Successfully tagged diytomcat:latest
  ```
  
4. 启动镜像
  
5. 访问测试
  
6. 发布项目（由于做了卷挂在，我们直接在本地编写项目就可以发布了）
  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd" id="WebApp_ID" version="2.5">
  <display-name>db</display-name>
  <welcome-file-list>
    <welcome-file>index.html</welcome-file>
    <welcome-file>index.htm</welcome-file>
    <welcome-file>index.jsp</welcome-file>
    <welcome-file>default.html</welcome-file>
    <welcome-file>default.htm</welcome-file>
    <welcome-file>default.jsp</welcome-file>
  </welcome-file-list>
</web-app>
```

```jsp
<html>
<head><title>Hello World</title></head>
<body>
Hello World!<br/>
<%
System.out.println("---my test web logs---");
%>
</body>
</html>
```

发现：项目部署成功，可以直接访问ok

我们以后开发的步骤：需要掌握Dockerfile的编写，我们之后的一切都是使用docker经i想来发布运行

## 发布自己的镜像

> DockerHUb

1. 地址：https://hub.docker.com/ 注册账号
2. 确定这个账号可以登录
3. 在我们服务器上提交自己的镜像

```shell
[root@localhost test]# docker login --help

Usage:  docker login [OPTIONS] [SERVER]

Log in to a Docker registry.
If no server is specified, the default is defined by the daemon.

Options:
  -p, --password string   Password
      --password-stdin    Take the password from stdin
  -u, --username string   Username


[root@localhost test]# docker login -u hzbskak
Password:
Error response from daemon: Get https://registry-1.docker.io/v2/: unauthorized: incorrect username or password
[root@localhost test]# docker login -u hzbskak
Password:
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
```

4. 登陆完毕后就可以提交镜像了，docker push

```shell
# push 自己的镜像到服务器上
[root@localhost test]# docker push diytomcat
Using default tag: latest
The push refers to repository [docker.io/library/diytomcat]
6fbb0898d558: Preparing
4fd2367d013d: Preparing
d16afef30535: Preparing
ad73d498d7b9: Preparing
43c516115976: Preparing
2653d992f4ef: Waiting
denied: requested access to the resource is denied # 拒绝

# push 镜像的问题
[root@localhost test]# docker push hzbskak/diytomcat:1.0
The push refers to repository [docker.io/hzbskak/diytomcat]
An image does not exist locally with the tag: hzbskak/diytomcat

# 解决，增加一个tag
[root@localhost test]# docker push hzbskak/diytomcat:1.0

# docker push 上去即可，自己发布的镜像尽量带上版本号
[root@localhost test]# docker push hzbskak/tomcat:1.0
The push refers to repository [docker.io/hzbskak/tomcat]
6fbb0898d558: Pushed
4fd2367d013d: Pushed
d16afef30535: Pushed
ad73d498d7b9: Pushed
43c516115976: Pushed
2653d992f4ef: Pushed
1.0: digest: sha256:2ba2966627689fe12d896227d5a306d405917b738bb44c793912ca7be9c959c6 size: 1580
```

提交的时候也是按照镜像的层级来进行发布的

## 小结

![image20210629164932988](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629164932988.1peq9umfk0hs.png)

# Docker网络

## 理解Docker

清空所有环境

> 测试

![image20210629170706355](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629170706355.3dhsc1eu1ss0.png)

三个网络

```shell
# 问题：docker 是如何处理容器网络访问的？    
```

![image20210629171548882](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629171548882.3g8n7tqkxtk0.png)

```shell
[root@localhost test]# docker run -d -P --name tomcat01 tomcat

# 查看容器的内部网络地址 ip addr ， 发现容器启动的时候会得到一个 eth0@if131 ip地址。docker分配的
[root@localhost test]# docker exec -it tomcat01 ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
130: eth0@if131: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever

# 思考。 linux能不能ping通容器颞部
[root@localhost test]#  ping 172.17.0.2
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.050 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.045 ms


# linux 可以 ping 通 docker 容器内部 
```

> 原理

1. 我们每启动一个docker容器，docker就会给docker容器分配一个ip，我们只要安装了docker，就会有一个网卡docker0
  
  桥接模式，使用的技术是veth-pair技术
  
  在测试ip addr
  
  ```shell
  [root@localhost test]# ip addr
  1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
      link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
      inet 127.0.0.1/8 scope host lo
         valid_lft forever preferred_lft forever
      inet6 ::1/128 scope host
         valid_lft forever preferred_lft forever
  2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
      link/ether 00:0c:29:01:cb:92 brd ff:ff:ff:ff:ff:ff
      inet 192.168.240.131/24 brd 192.168.240.255 scope global noprefixroute dynamic ens33
         valid_lft 1129sec preferred_lft 1129sec
      inet 192.168.240.3/24 brd 192.168.240.255 scope global secondary noprefixroute ens33
         valid_lft forever preferred_lft forever
      inet6 fe80::2716:8358:e585:ac04/64 scope link noprefixroute
         valid_lft forever preferred_lft forever
  3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
      link/ether 02:42:0d:11:06:60 brd ff:ff:ff:ff:ff:ff
      inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
         valid_lft forever preferred_lft forever
      inet6 fe80::42:dff:fe11:660/64 scope link
         valid_lft forever preferred_lft forever
  # 多了个131
  131: veth4be9a62@if130: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
      link/ether ee:59:72:a1:93:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
      inet6 fe80::ec59:72ff:fea1:93bf/64 scope link
         valid_lft forever preferred_lft forever
  ```
  
  2. 在启动一个容器测试，发现又多了一对网卡
    
    ```shell
    [root@localhost test]# docker run -d -P --name tomcat02 tomcat
    9391f02636daa46cde8ab621e114b080b9a975d9102be0bd9f02c45aa8ee45e2
    You have new mail in /var/spool/mail/root
    [root@localhost test]# ip adr
    Object "adr" is unknown, try "ip help".
    [root@localhost test]# ip addr
    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
        link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
        inet 127.0.0.1/8 scope host lo
           valid_lft forever preferred_lft forever
        inet6 ::1/128 scope host
           valid_lft forever preferred_lft forever
    2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
        link/ether 00:0c:29:01:cb:92 brd ff:ff:ff:ff:ff:ff
        inet 192.168.240.131/24 brd 192.168.240.255 scope global noprefixroute dynamic ens33
           valid_lft 985sec preferred_lft 985sec
        inet 192.168.240.3/24 brd 192.168.240.255 scope global secondary noprefixroute ens33
           valid_lft forever preferred_lft forever
        inet6 fe80::2716:8358:e585:ac04/64 scope link noprefixroute
           valid_lft forever preferred_lft forever
    3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
        link/ether 02:42:0d:11:06:60 brd ff:ff:ff:ff:ff:ff
        inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
           valid_lft forever preferred_lft forever
        inet6 fe80::42:dff:fe11:660/64 scope link
           valid_lft forever preferred_lft forever
    131: veth4be9a62@if130: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
        link/ether ee:59:72:a1:93:bf brd ff:ff:ff:ff:ff:ff link-netnsid 0
        inet6 fe80::ec59:72ff:fea1:93bf/64 scope link
           valid_lft forever preferred_lft forever
    # 发现又多了一对网卡
    133: vetha0bd6b8@if132: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master docker0 state UP group default
        link/ether 62:3a:c4:f5:34:f0 brd ff:ff:ff:ff:ff:ff link-netnsid 1
        inet6 fe80::603a:c4ff:fef5:34f0/64 scope link
           valid_lft forever preferred_lft forever
    ```
    
    ```shell
    # 我们发现这个容器带来的网卡，都是一对一对的
    # veth-pair 就是一对的虚拟设备接口，他们都是成对出现的，一段连着协议，一段彼此相连
    # 正因为有了这个特性，veth-pair 充当一个桥梁，连接各种虚拟网络设备
    # OpenStac，Docker容器之间的连接，OVS的连接，都是veth-pair技术
    ```
    
    3. 我们来测试下tomcat01和tomcat02是否可以ping通
      
      ```sh
      You have new mail in /var/spool/mail/root
      [root@localhost test]# docker exec -it tomcat02 ping 172.17.0.2
      PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
      64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.089 ms
      64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.060 ms
      
      # 结论：容器和容器之间是可以互相ping通
      ```
      

![image20210629174921730](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629174921730.72ni97t3kg40.png)

结论： tomcat01 和tomcat02 是公用的一个路由器，docker0

所有的容器不指定网络的情况下，都是docker0路由器，docker会给我们的容器分配一个默认的可用ip

> 小结

Docker使用的是LInux的桥接，宿主机中是一个Docker容器的网桥 Docker0

![image20210629175919723](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210629175919723.4rzj48m54f00.png)

Docker中的所有的网络接口都是虚拟的，虚拟的转发效率高（内网传递文件）

只要让其删除，对应的网桥一对就没了

![image20210630135838328](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image-20210630135838328.5vw7w994w3o0.png)

## --link

> 思考一个场景，我们编写了一个微服务，database url=ip:，项目部重启，数据库ip换掉了，我们希望可以通过名字来进行访问容器

```shell
[root@localhost test]# docker exec -it tomcat02 ping tomcat01
ping: tomcat01: Name or service not known


# 如何解决
# 通过 --link 可以解决网络联通问题
[root@localhost test]# docker run -d -P --name tomcat03 --link tomcat02 tomcat
83129fe1c7f36df4e0d362982603b257188d110cd45764f6132baed54f47e5de
You have new mail in /var/spool/mail/root
[root@localhost test]# docker exec -it tomcat03 ping tomcat02
PING tomcat02 (172.17.0.3) 56(84) bytes of data.
64 bytes from tomcat02 (172.17.0.3): icmp_seq=1 ttl=64 time=0.200 ms
64 bytes from tomcat02 (172.17.0.3): icmp_seq=2 ttl=64 time=0.301 ms

# 反向可以ping通吗？
[root@localhost test]# docker exec -it tomcat02 ping tomcat03
ping: tomcat03: Name or service not known
```

**探究：inspect**

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.1fxcs8i0tva8.png)

其实这个tomcat03就是在本地配置了tomcat02的配置

```shell
# 查看 hosts 配置，在这里原理发现
[root@localhost test]# docker exec -it tomcat03 cat /etc/hosts
127.0.0.1       localhost
::1     localhost ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
172.17.0.3      tomcat02 9391f02636da
172.17.0.4      83129fe1c7f3
```

本质探究：--link 就是我们在 hosts 配置中增加了一个 172.17.0.3 tomcat02 9391f02636da

我们现在玩docker已经不建议使用 -- link了

自定义网络，不适用docker0

docker0问题：它不支持容器名连接访问

## 自定义网络

> 查看所有的docker网络

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.gnrda8mbvwo.png)

**网络模式**

bridge:桥接 docker （默认，自己搭建也用）

none:不配置网络

host:和宿主机共享网络

container:容器网络连通（用的少，局限性大）

**测试**

```shell
# 我们直接启动的命令 --net bridge， 而这个就是我们的docker0
[root@localhost test]# docker run -d -P --name tomcat01 tomcat
[root@localhost test]# docker run -d -P --name tomcat01 --net bridge tomcat

# docker0特点，默认，域名不能访问， --link 可以打通连接

# 我们可以自定义一个网络
# --driver bridge 模式 桥接
# --subnet 192.168.0.0/16  子网掩码 255*255个
# --gateway 192.168.0.1 网关
[root@localhost test]# docker network create --driver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 mynet
0c9fe3872f2812a119ddb2e1bf797c742549226c78f2a66ce2733b390f35dc30
[root@localhost test]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
ff3859319809   bridge    bridge    local
138846a95e07   host      host      local
0c9fe3872f28   mynet     bridge    local
0202174ed4f9   none      null      local
```

那么自己的网络就创建好了

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.7a25bc2gt280.png)

```shell
[root@localhost test]# docker run -d -P --name tomcat-net-01 --net mynet tomcat^C
You have new mail in /var/spool/mail/root
[root@localhost test]# docker network ls
NETWORK ID     NAME      DRIVER    SCOPE
ff3859319809   bridge    bridge    local
138846a95e07   host      host      local
0c9fe3872f28   mynet     bridge    local
0202174ed4f9   none      null      local
[root@localhost test]# docker run -d -P --name tomcat-net-01 --net mynet tomcat
bba5282a18cfb1b124543c8ba1ae883c0a0b4755f933a65228c9a9554c210183
[root@localhost test]# docker run -d -P --name tomcat-net-02 --net mynet tomcat
32a6f39d45e21cca35d8fbfd39cb8b756ca0504bb9965a66fb6edbc5a1039583
[root@localhost test]# docker inspect mynet
[
    {
        "Name": "mynet",
        "Id": "0c9fe3872f2812a119ddb2e1bf797c742549226c78f2a66ce2733b390f35dc30",
        "Created": "2021-06-29T23:46:32.683682838-07:00",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "192.168.0.0/16",
                    "Gateway": "192.168.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {
            "32a6f39d45e21cca35d8fbfd39cb8b756ca0504bb9965a66fb6edbc5a1039583": {
                "Name": "tomcat-net-02",
                "EndpointID": "dcf263735e351318040ac020b7326348885325923a5548636fc670ccb283faa2",
                "MacAddress": "02:42:c0:a8:00:03",
                "IPv4Address": "192.168.0.3/16",
                "IPv6Address": ""
            },
            "bba5282a18cfb1b124543c8ba1ae883c0a0b4755f933a65228c9a9554c210183": {
                "Name": "tomcat-net-01",
                "EndpointID": "ac38a305ac7ba7ea2fa28c33be61b68d41f255f6af804b00b98c5be76d4695d6",
                "MacAddress": "02:42:c0:a8:00:02",
                "IPv4Address": "192.168.0.2/16",
                "IPv6Address": ""
            }
        },
        "Options": {},
        "Labels": {}
    }
]



# 再次测试ping连接
[root@localhost test]# docker exec -it tomcat-net-01 ping 192.168.0.3
PING 192.168.0.3 (192.168.0.3) 56(84) bytes of data.
64 bytes from 192.168.0.3: icmp_seq=1 ttl=64 time=0.052 ms
64 bytes from 192.168.0.3: icmp_seq=2 ttl=64 time=0.046 ms


# 现在不适用 --link 也可以ping名字
[root@localhost test]# docker exec -it tomcat-net-01 ping tomcat-net-02
PING tomcat-net-02 (192.168.0.3) 56(84) bytes of data.
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=1 ttl=64 time=0.030 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=2 ttl=64 time=0.046 ms
64 bytes from tomcat-net-02.mynet (192.168.0.3): icmp_seq=3 ttl=64 time=0.050 ms
^C
--- tomcat-net-02 ping statistics ---
3 packets transmitted, 3 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.030/0.042/0.050/0.008 ms
```

我们自定义的网络docker都已经帮我们维护好了对应的关系，推荐我们平时这样使用网络

好处：

redis-不同的集群使用不同的网络，保证集群是安全和健康的

mysql-不同的集群使用不同的网络，保证集群是安全和健康的

## 网络联通

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.c4i8ze2fzhk.png)

```shell
# 测试打通 tomcat01 - mynet
[root@localhost test]# docker network connect mynet tomcat01
# 连通之后就是将 tomcat01 放到了 mynet网络下

# 一个容器两个ip地址
# 阿里云服务， 公网ip 私网ip
```

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.bvqrr19prn4.png)

```shell
# 01 连接成功
[root@localhost test]# docker exec -it tomcat01 ping tomcat-net-01
PING tomcat-net-01 (192.168.0.2) 56(84) bytes of data.
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=1 ttl=64 time=0.034 ms
64 bytes from tomcat-net-01.mynet (192.168.0.2): icmp_seq=2 ttl=64 time=0.073 ms

# 02 依旧是打不通的
[root@localhost test]# docker exec -it tomcat02 ping tomcat-net-01
ping: tomcat-net-01: Name or service not known
```

结论：假设要跨网络操作别人，就需要使用 docker network connect 连通

## 实战：Redis集群部署

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.1ybbtw4p3z6o.png)

shell脚本

```shell
# 创建网卡
[root@localhost test]# docker network create redis --subnet 172.38.0.0/16

# 通过脚本创建六个redis配置
for port in $(seq 6); \
do \
mkdir -p /mydata/redis/node-${port}/conf
touch /mydata/redis/node-${port}/conf/redis.conf
port 6379
bind 0.0.0.0
cluster-enabled yes
cluster-config-file nodes.conf
cluster-node-timeout 5000
cluster-announce-ip 172.38.0.1${port}
cluster-announce-port 6379
cluster-announce-bus-port 16379
appendonly yes
EOF
done


# 启动六个redis容器
docker run -p 6371:6379 -p 16371:16379 --name redis-1 \
-v /mydata/redis/node-1/data:/data \
-v /mydata/redis/node-1/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.11 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6372:6379 -p 16372:16379 --name redis-2 \
-v /mydata/redis/node-2/data:/data \
-v /mydata/redis/node-2/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.12 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6373:6379 -p 16373:16379 --name redis-3 \
-v /mydata/redis/node-3/data:/data \
-v /mydata/redis/node-3/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.13 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6374:6379 -p 16374:16379 --name redis-4 \
-v /mydata/redis/node-4/data:/data \
-v /mydata/redis/node-4/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.14 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6375:6379 -p 16375:16379 --name redis-5 \
-v /mydata/redis/node-5/data:/data \
-v /mydata/redis/node-5/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.15 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf

docker run -p 6376:6379 -p 16376:16379 --name redis-6 \
-v /mydata/redis/node-6/data:/data \
-v /mydata/redis/node-6/conf/redis.conf:/etc/redis/redis.conf \
-d --net redis --ip 172.38.0.16 redis:5.0.9-alpine3.11 redis-server /etc/redis/redis.conf


# 创建集群
redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1


[root@localhost /]# docker exec -it redis-1 /bin/sh
/data # redis-cli --cluster create 172.38.0.11:6379 172.38.0.12:6379 172.38.0.13:6379 172.38.0.14:6379 172.38.0.15:6379 172.38.0.16:6379 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 172.38.0.15:6379 to 172.38.0.11:6379
Adding replica 172.38.0.16:6379 to 172.38.0.12:6379
Adding replica 172.38.0.14:6379 to 172.38.0.13:6379
M: 176e15fcce0982ec955bbcf272b2293430f7d22b 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
M: 2579705092fd084be7c2e6dd756215138174a0b1 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
M: c321cebfca76809337da13f08d8a8ce00288184e 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
S: 566f5b703bff6eb3662a1e9a23275ccb4505def5 172.38.0.14:6379
   replicates c321cebfca76809337da13f08d8a8ce00288184e
S: ed1418dc9388fb65ed4b3f8047698dedefff370d 172.38.0.15:6379
   replicates 176e15fcce0982ec955bbcf272b2293430f7d22b
S: 0d62df88c7fc0b026da261bef202bc6a29ea48ee 172.38.0.16:6379
   replicates 2579705092fd084be7c2e6dd756215138174a0b1
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
...
>>> Performing Cluster Check (using node 172.38.0.11:6379)
M: 176e15fcce0982ec955bbcf272b2293430f7d22b 172.38.0.11:6379
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: ed1418dc9388fb65ed4b3f8047698dedefff370d 172.38.0.15:6379
   slots: (0 slots) slave
   replicates 176e15fcce0982ec955bbcf272b2293430f7d22b
M: 2579705092fd084be7c2e6dd756215138174a0b1 172.38.0.12:6379
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
M: c321cebfca76809337da13f08d8a8ce00288184e 172.38.0.13:6379
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 0d62df88c7fc0b026da261bef202bc6a29ea48ee 172.38.0.16:6379
   slots: (0 slots) slave
   replicates 2579705092fd084be7c2e6dd756215138174a0b1
S: 566f5b703bff6eb3662a1e9a23275ccb4505def5 172.38.0.14:6379
   slots: (0 slots) slave
   replicates c321cebfca76809337da13f08d8a8ce00288184e
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
/data #

# 进入 redis
/data # redis-cli -c
127.0.0.1:6379>
```

docker 搭建redis集群完成

从机redis-4替代主机redis-3成为master

![](https://cdn.jsdelivr.net/gh/4927525/images@master/20210630/image.67qkgqnd7fg0.png)

我们使用了docker之后，所有的技术都会慢慢的变得简单起来

# SpringBoot微服务打包Docker镜像

1. 构建springboot项目
2. 打包应用
3. 编写dockerfile
4. 构建镜像
5. 发布运行

以后我们使用了docker之后，给别人交付的就是一个镜像即可

# Docker Compose

## 简介

Docker

DockerFile build run 手动操作，单个容器！

微服务。100个微服务！有依赖关系。

Docker Compose 来轻松的管理容器。定义运行多个容器。

> 官方介绍 https://docs.docker.com/compose/

​ Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see [the list of features](https://docs.docker.com/compose/#features).

Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in [Common Use Cases](https://docs.docker.com/compose/#common-use-cases).

**三步骤：**

Using Compose is basically a three-step process:

1. Define your app’s environment with a `Dockerfile` so it can be reproduced anywhere.
  - Dockerfile 保证我们的项目在任何地方可以运行
2. Define the services that make up your app in `docker-compose.yml` so they can be run together in an isolated environment.
  - services 什么是服务
  - docekr-compose.yml 这个文件怎么写！
3. Run `docker compose up` and the [Docker compose command](https://docs.docker.com/compose/cli-command/) starts and runs your entire app. You can alternatively run `docker-compose up` using the docker-compose binary.
  - 启动项目

作用：容器批量编排

> 我自己理解

Compose是Docker官方的开源项目。需要安装！

`Dockerfile` 让程序在任何地方运行。 web服务。 redis,mysql,redis,nginx...多个容器。 run

Compose

```shell
version: "3.9"  # optional since v1.27.0
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
      - logvolume01:/var/log
    links:
      - redis
  redis:
    image: redis
volumes:
  logvolume01: {}
```

docker-compose up 100个服务

Compose：重要的概念

- 服务services：容器。应用。（web,redis,mysql...）
- 项目project：一组关联的容器。博客。web，mysql

## 安装

https://docs.docker.com/compose/install/

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.6c7k26pixjc0.png)

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.3eh8fkc4klo0.png)

## 体验

https://docs.docker.com/compose/gettingstarted/

python应用，redis计数器

1. 应用 app.py
2. Dockerfile 应用打包为镜像
3. Docker-compose.yaml文件（定义整个服务，需要的环境。web redis） 完整的上限服务！
4. 启动compose项目（docker-compose up）

流程：

1. 创建网络
2. 执行Docker-compose yaml
3. 启动任务

Docker-compose yaml

Creating composetest_web_1 ... done
Creating composetest_redis_1 ... done

1. 文件名 composetest
  
2. 服务
  
  ```shell
  version: "3.9"
  services:
    web:
      build: .
      ports:
        - "5000:5000"
      volumes:
        - .:/code
    redis:
      image: "redis:alpine"
  ```
  
  自动的默认规则？
  

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.6nsgc670d0s0.png)

docker images

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.1kw9il124pvk.png)

```bash
PS C:\Users\windows>  docker service ls
Error response from daemon: This node is not a swarm manager. Use "docker swarm init" or "docker swarm join" to connect this node to swarm and try again.
```

默认的服务名 文件名 _ 服务名 _ num

多个服务器。集群。 A B _num 副本数量

服务redis服务 => 4个副本。

集群状态。服务不可能只有一个运行实例。 弹性， 10HA 高并发

kubectl service 负载均衡。

3. 网络规则
  
  ![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.44t5arssg7m0.png)
  

10个服务=>项目（项目中的内容都在同个网络下。域名访问）

mysql:3306

10个容实例：mysql

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.6t8hudxd9eg0.png)

如果在同一个网络下，我们可以直接通过域名访问

HA!

停止：docker-composer down CTRL+C

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.2uhd3m0amla0.png)

docker-compose

以前都是单个docker run 启动容器

docker-compose。 通过docker-compose编写yaml配置文件，可以通过compose一键启动所有服务，停止

**Docker小结**

1. Docker镜像。 run=>容器
2. Dockerfile构建镜像（服务打包）
3. docker-compose启动项目 （编排，多个微服务/环境）
4. Docker网络

## yaml规则

docker-compose yaml 核心。

https://docs.docker.com/compose/compose-file/compose-file-v3/#compose-file-structure-and-examples

```shell
# 3层

version: '' # 版本
services: # 服务
    服务1：web
    # 服务配置
    iamges
    build 
    network
    ...
    服务2：redis
    ...

# 其他配置 网络/卷，全局规则
volumes:
networks:
configs:
```

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.2jla36joyxk0.png)

## 开源项目

### 博客

https://docs.docker.com/samples/wordpress/

compose 应用。 => 一键启动

1. 下载项目（docker-compose.yaml）
2. 如果需要文件。Dockerfile
3. 文件准备齐全（直接一键启动项目）

前台启动

docker -d

docker-compose up -d

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.162zecih8qqk.png)

## 实战

1. 编写项目微服务
2. dockerfile构建镜像
3. docker-compose.yaml编排项目
4. 丢到服务器 docker-compose up

**小结**

未来项目只要有docker-compose文件，按照这个规则，启动编排容器

公司：docker-compose 直接启动

网上开源项目：docker-compose一键搞定

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211122/image.7bifk25mb8o0.png)

```bash
docker-compose up --build # 重新构建
```

**总结：**

**工程 服务 容器**

项目 compose:三层

- 工程project
- 服务 服务
- 容器 运行实例 docker k8s 容器