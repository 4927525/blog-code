---
title: "docker swarm 搭建与服务更新"

date: 2021-12-15T17:54:53+08:00

Description: ""

Tags: ["docker", "swarm", "docker-compose"]

Categories: ["docker"]

DisableComments: false

---

# 序

研究了好久的docker集群部署终于是在本地服务器跑通了。于是趁着热乎赶紧码下来。    

![src=http---pic1](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/src=http---pic1.zhimg.com-50-v2-a0b6937057b1ca1f6b695929017695e0_hd.jpg&refer=http---pic1.zhimg.24mok2d62g1s.jpg)





## 1. 创建集群

```bash
docker swarm init --advertise-addr 192.168.101.67
```

将本机ip 作为默认leader节点创建。

```bash
  Swarm: active #已激活
  NodeID: p9cf3rg9ygyqxe3vr0psm0t7a #节点id 
  Is Manager: true
  ClusterID: pdch8cn7rh82vw9lzx81k7bcn
  Managers: 1
  Nodes: 1 #节点数
  Default Address Pool: 10.0.0.0/8

通过
```



通过命令在本机获取join-token，将其他服务器以manager或worker节点加入集群中

```bash
docker swarm join-token [worker|manager]
```



![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.7jjwzkm59ic0.png)



在需要加入的节点中输入这行命令就可以直接加入。

所有服务器执行完成后，可以在manager节点中输入`docker info` 查看

显示当前有四个节点，其中三个节点是管理节点

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.4q2b71vmnno0.png)



离开集群

```bash
docker swarm leave
```

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.3bntdykzcfw0.png)

# 2. 使用yml文件编排服务

> Docker stack 也是一个yaml文件，和一份docker-compose.yml文件差不多，指令也基本一致。但是与compose相比其不支持build、links和network_mode。Docker stack有一个新的指令deploy。



[docker stack | Docker Documentation](https://docs.docker.com/engine/reference/commandline/stack/)



stack不支持的指令:

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.4ruiccj1fh80.png)



> Deploy是用来指定swarm服务部署和运行时的相关配置，并且只有使用docker stack deploy 部署swarm集群时才会生效。如果使用docker-compose up 或者docker-compose run时，该选项会被忽略。要使用deploy选项，compose-file中version版本要在3或3+。



```shell
version: '3.3'
services:
    nginx:
        image: registry.xxx.com/nginx:5.6-alpine
        ports:
            - "80:80"
            - "443:443"
        env_file:
            - .env
        environment:
            - NGINX_HOST=${NGINX_HOST}
            - TZ=Asia/Shanghai
        command: /bin/sh -c "envsubst '$$NGINX_HOST' < /etc/nginx/conf.d/default.template > /etc/nginx/conf.d/default.conf && nginx -g 'daemon off;'"
        depends_on:
            - php
            - mysql
        networks:
            phal_net:
                aliases:
                    - nginx
        deploy:
            replicas: 4
            update_config:
                parallelism: 2
                failure_action: rollback
            restart_policy:
                condition: on-failure
                delay: 5s
                max_attempts: 3
                window: 120s
    php:
        image: registry.xxx.com/php:8.0-fpm-alpine
        environment:
          - TZ=Asia/Shanghai
        networks:
            phal_net:
                aliases:
                    - php
        deploy:
            replicas: 4
            update_config:
                parallelism: 2
                failure_action: rollback
            restart_policy:
                condition: on-failure
                delay: 5s
                max_attempts: 3
                window: 120s
    redis:
        image: registry.xxx.com/redis-test
        volumes:
            - redis-volume:/data
            - redis-volume:/redis/conf
            - redis-volume:/redis/logs
        ports:
            - 6379:6379
        networks:
            phal_net:
                aliases:
                    - redis
        deploy:
            replicas: 4
            update_config:
                parallelism: 2
                failure_action: rollback
            restart_policy:
                condition: on-failure
                delay: 5s
                max_attempts: 3
                window: 120s
    mongo:
        image: mongo:latest
        ports:
            - 27017:27017
        volumes:
            - mongo-volume:/data/configdb/mongo.conf
            - mongo-volume:/data/db
            - mongo-volume:/data/log/config.log
            - mongo-volume:/data/backup
        networks:
            phal_net:
                aliases:
                    - mongo
        deploy:
            replicas: 1
            update_config:
                parallelism: 1
                failure_action: rollback
            restart_policy:
                condition: on-failure
                delay: 5s
                max_attempts: 3
                window: 120s
    visualizer:
        image: dockersamples/visualizer:latest
        ports:
            - "8080:8080"
        stop_grace_period: 1m30s
        volumes:
            - "/var/run/docker.sock:/var/run/docker.sock"
        deploy:
            replicas: 1
            placement:
                constraints: [node.role == manager]
    portainer:
        image: portainer/portainer
        ports:
            - "8090:9000"
        volumes:
            - "/var/run/docker.sock:/var/run/docker.sock"
        deploy:
            replicas: 1
            placement:
                constraints: [node.role == manager]
networks:
  phal_net:

volumes:
    mongo-volume:
    redis-volume:
    mysql-volume:




```

*visualizer，portainer 为服务管理与监听服务*



1，mode ：global 全局（每个群集节点只有一个容器）replicated 副本（指定容器的数量）。默认值：replicated。

2，replicas：副本模式下每个节点启动副本的数量

3，endpoint_mode：指定swarm服务发现的模式

- vip - Docker为swarm集群服务分配一个虚拟IP(VIP)，作为客户端到达集群服务的“前端”。Docker 在客户端和可用工作节点之间对服务的请求进行路由。而客户端不用知道有多少节点参与服务或者是这些节点的IP/端口。（这是默认模式）
- dnsrr - DNS轮询（DNSRR）服务发现不使用单个虚拟IP。 Docker为服务设置DNS条目，使得服务名称的DNS查询返回一个IP地址列表，并且客户端直接连接到其中的一个。如果您想使用自己的负载平衡器，或者混合Windows和Linux应用程序，则DNS轮询功能非常有用。

4，labels：指定服务的标签。这些标签仅在服务上设置，而不在服务的任何容器上设置

5，resources：设置服务资源分配

- limits：最大使用限制
- reservations：表示预留，即最小使用
- cpus: '0.50' 表示最大或预留50%
- memory: 20M：表示最大或预留20M

6，restart_policy：配置在容器退出时是否并如何重启容器。取代docker-compose 中的 restart指令。

- condition ：none、on-failure和any（默认any）
- delay ：在重启尝试之间等待多久（默认0）
- max_attempts ：尝试重启的次数（默认一直重启，直到成功）
- window ： 在确实一个重启是否成功前需要等待的窗口时间　

7，update_config ：配置更新服务，用于无缝更新应用（rolling update)

- parallelism：同一时间升级的容器数量
- delay：容器升级间隔时间
- failure_action：升级失败后的动作（continue、rollback和pause。默认pause）。
- monitor：更新完成后确实成功的时间（ns|us|ms|s|m|h）。（默认0s）
- max_failure_ratio：更新期间允许的失败率
- order： 更新期间的操作顺序。停止优先（旧任务在开始新任务之前停止）或者先启动（首先启动新任务，并且正在运行的任务短暂重叠）（默认停止优先）注意：只支持v3.4及更高版本。

# 3. docker stack 的一些命令

### 3.1 部署新的堆栈或更新现有堆栈

```bash
docker stack deploy [OPTIONS] STACK
```

*私有仓库需要携带"--with-registry-auth"参数，否则提示*

```shell
mage registry.xxx.com/xxx:latest could not be accessed on a registry to record 
its digest. Each node will access registry.xxx.com/ejiyuan/xxx:latest,
possibly leading to different nodes running different


```



先登录到私有仓库

```bash
docker login registry.xxx.com
```

执行命令开始使用 docker-stack.yml 文件部署服务堆，堆名为“phal”



```bash
 docker stack deploy -c docker-stack.yml phal--with-registry-auth
```



会先创建网络，然后创建服务， 服务名(stack名+下划线+service名)

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.6x2xp5oa9ds0.png)

### 3.2 列出现有堆栈和服务数

```bash
docker stack ls
```



![](C:\Users\windows\AppData\Roaming\marktext\images\2021-12-15-20-48-39-image.png)



### 3.3 列出堆栈中的服务

```basic
docker stack services [OPTIONS]
```

![](C:\Users\windows\AppData\Roaming\marktext\images\2021-12-15-20-50-21-image.png)



### 3.4 列出堆栈中的任务

```bash
docker stack ps [OPTIONS]
```

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.2wwemav04js0.png)



### 3.5 删除一个或多个堆栈

```bash
docker stack rm STACK [STACK...] [flags]
```

# 4. 服务升级

### 4.1 更新镜像

```bash
docker service update --image registry.xxx.com/php:8.0-fpm-alpine  phal_php --with-registry-auth
```



因为有4个副本，parallelism设置同时更新2个。先更新两个，等待delay后在更新两个

![](C:\Users\windows\AppData\Roaming\marktext\images\2021-12-15-20-57-28-image.png)

注：即使设置了update_config.order: start-first，服务会先启动在停止，但是php启动 项目需要一定时间，这段时间服务是不可用的，但是服务状态是Runing的，所以，这里启用两个副本，每次更新一个，等待一个服务启动完成后，在执行另一个更新，主要目的是为了，无缝的升级系统，具体update_config.delay设置为多少可以参考php的启动时间

### 4.2 更新节点数量

```bash
docker service scale phal_redis=5
```

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.5qzh5nrthmc0.png)

### 4.3 添加或者更新一个对外端口

```bash
docker service update --publish-add 9001 phal_portainer
```

![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.53cga1wu3d00.png)

# 5. 更新节点

```bash
docker node update [OPTIONS] NODE [flags]
```

参数：

- --availability 节点的可用性(有效/暂停/耗尽)
- --label-add 添加或更新节点标签(key = value)
- --label-rm 删除节点标签(如果存在)
- --role 节点的作用(worker / manager)



![image](https://cdn.jsdelivr.net/gh/4927525/images@master/20211215/image.22y4kbdhck1s.png)



# # 5. 注意

如果一台机器启用多个服务注意，合理分配cpu与内存资源，因mongo在启动编译时会很吃内存，且docker是多线程启动的，所有最好是限定一下（设置resources.limits）否者会导致内存在同一时刻用光，某些服务启动失败当然也可是设置出错重启（restart_policy.condition:on-failure），另外设置resources.reservations要注意，不要超出总内存或cpu百分比，否者会导致后面服务无法获取cpu或内存资源出现“no suitable node (insufficien”错误（这个错误很奇怪，某个service不启动，也不输出日志，使用“**docker stack ps [xxxx]**”查看状态会显示此错误）无法启动





其实正常走下来觉得还是比较容易的，但是期间没少踩坑，每次编译文件都要好长时间。真的麻了。就比如编译一个php的dockerfile，要半个多小时。

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Finews.gtimg.com%2Fnewsapp_bt%2F0%2F13371439711%2F1000.jpg&refer=http%3A%2F%2Finews.gtimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1642166629&t=40180c298c83d82ef6eefb647bea898c "点击查看源网页")

后来才知道原来可以再第一次编译的时候就把这个镜像打包到私有仓库，以后可以在这个基础上加ext cmd。

![](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Finews.gtimg.com%2Fnewsapp_match%2F0%2F9335157602%2F0.jpg&refer=http%3A%2F%2Finews.gtimg.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1642166680&t=eda8e8524ba6d9f68493a83df61e9cb8 "点击查看源网页")


























































