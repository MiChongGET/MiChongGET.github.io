---
title: 'Docker（二）基础使用'
date: 2018-05-22 23:20:23
tags: 
- Docker
- Linux
categories:
- 运维
- 服务器
---

#### 一、基础配置命令

```shell
# 查看所有容器
docker ps -a
# 查看运行中的容器
docker ps
# 启动容器
docker start 容器名或ID
# 进入容器
docker attach 容器名或ID
```

##### dokcer run命令

```shell
docker run <相关参数> <镜像 ID> <初始命令>

-i：表示以“交互模式”运行容器
-t：表示容器启动后会进入其命令行
-v：表示需要将本地哪个目录挂载到容器中，格式：-v <宿主机目录>:<容器目录>

Usage: docker run [OPTIONS] IMAGE [COMMAND] [ARG...]  

  -d, --detach=false         指定容器运行于前台还是后台，默认为false   
  -i, --interactive=false   打开STDIN，用于控制台交互  
  -t, --tty=false            分配tty设备，该可以支持终端登录，默认为false  
  -u, --user=""              指定容器的用户  
  -a, --attach=[]            登录容器（必须是以docker run -d启动的容器）
  -w, --workdir=""           指定容器的工作目录 
  -c, --cpu-shares=0        设置容器CPU权重，在CPU共享场景使用  
  -e, --env=[]               指定环境变量，容器中可以使用该环境变量  
  -m, --memory=""            指定容器的内存上限  
  -P, --publish-all=false    指定容器暴露的端口  
  -p, --publish=[]           指定容器暴露的端口 
  -h, --hostname=""          指定容器的主机名  
  -v, --volume=[]            给容器挂载存储卷，挂载到容器的某个目录  
  --volumes-from=[]          给容器挂载其他容器上的卷，挂载到容器的某个目录
  --cap-add=[]               添加权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  --cap-drop=[]              删除权限，权限清单详见：http://linux.die.net/man/7/capabilities  
  --cidfile=""               运行容器后，在指定文件中写入容器PID值，一种典型的监控系统用法  
  --cpuset=""                设置容器可以使用哪些CPU，此参数可以用来容器独占CPU  
  --device=[]                添加主机设备给容器，相当于设备直通  
  --dns=[]                   指定容器的dns服务器  
  --dns-search=[]            指定容器的dns搜索域名，写入到容器的/etc/resolv.conf文件  
  --entrypoint=""            覆盖image的入口点  
  --env-file=[]              指定环境变量文件，文件格式为每行一个环境变量  
  --expose=[]                指定容器暴露的端口，即修改镜像的暴露端口  
  --link=[]                  指定容器间的关联，使用其他容器的IP、env等信息  
  --lxc-conf=[]              指定容器的配置文件，只有在指定--exec-driver=lxc时使用  
  --name=""                  指定容器名字，后续可以通过名字进行容器管理，links特性需要使用名字  
  --net="bridge"             容器网络设置:
                                bridge 使用docker daemon指定的网桥     
                                host     //容器使用主机的网络  
                                container:NAME_or_ID  >//使用其他容器的网路，共享IP和PORT等网络资源  
                                none 容器使用自己的网络（类似--net=bridge），但是不进行配置 
  --privileged=false         指定容器是否为特权容器，特权容器拥有所有的capabilities  
  --restart="no"             指定容器停止后的重启策略:
                                no：容器退出时不重启  
                                on-failure：容器故障退出（返回值非零）时重启 
                                always：容器退出时总是重启  
  --rm=false                 指定容器停止后自动删除容器(不支持以docker run -d启动的容器)  
  --sig-proxy=true           设置由代理接受并处理信号，但是SIGCHLD、SIGSTOP和SIGKILL不能被代理  
```

#### 二、Docker命令

```shell
镜像操作：
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    images    List images
    load      Load an image from a tar archive or STDIN
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rmi       Remove one or more images
    search    Search the Docker Hub for images
    tag       Tag an image into a repository
    save      Save one or more images to a tar archive 
    history   显示某镜像的历史
    inspect   获取镜像的详细信息

    容器及其中应用的生命周期操作：
    create    创建一个容器
    kill      Kill one or more running containers
    inspect   Return low-level information on a container, image or task
    pause     Pause all processes within one or more containers
    ps        List containers
    rm        删除一个或者多个容器
    rename    Rename a container
    restart   Restart a container
    run       创建并启动一个容器
    start     启动一个处于停止状态的容器
    stats     显示容器实时的资源消耗信息
    stop      停止一个处于运行状态的容器
    top       Display the running processes of a container
    unpause   Unpause all processes within one or more containers
    update    Update configuration of one or more containers
    wait      Block until a container stops, then print its exit code
    attach    Attach to a running container
    exec      Run a command in a running container
    port      List port mappings or a specific mapping for the container
    logs      获取容器的日志

    容器文件系统操作：
    cp        Copy files/folders between a container and the local filesystem
    diff      Inspect changes on a container's filesystem
    export    Export a container's filesystem as a tar archive
    import    Import the contents from a tarball to create a filesystem image

    Docker registry 操作：
    login     Log in to a Docker registry.
    logout    Log out from a Docker registry.

    Volume 操作
    volume    Manage Docker volumes

    网络操作
    network   Manage Docker networks

    Swarm 相关操作
    swarm     Manage Docker Swarm
    service   Manage Docker services
    node      Manage Docker Swarm nodes

    系统操作：
    version   Show the Docker version information
    events    持续返回docker 事件
    info      显示Docker 主机系统范围内的信息
```

```shell
# 查看运行中的容器
docker ps

# 查看所有容器
docker ps -a

# 退出容器
按Ctrl+D 即可退出当前容器【但退出后会停止容器】

# 退出不停止容器：
组合键：Ctrl+P+Q

# 启动容器
docker start 容器名或ID

# 进入容器
docker attach 容器名或ID

# 停止容器
docker stop 容器名或ID

# 暂停容器
docker pause 容器名或ID

#继续容器
docker unpause 容器名或ID

# 删除容器
docker rm 容器名或ID

# 删除全部容器--慎用
docker stop $(docker ps -q) & docker rm $(docker ps -aq)

#保存容器，生成镜像
docker commit 容器ID 镜像名称

#从 host 拷贝文件到 container 里面
docker cp /home/soft centos:/webapp
```

