---
layout: post
title: 'Docker命令记录'
date: 2019-10-28
author: wyg1997
color: rgb(255,210,32)
cover: '../assets/imgs/docker.png'
tags: docker
---

**注：原文章在[我的csdn博客](https://blog.csdn.net/wyg1997/article/details/102780039)上。**

# Docker命令记录
平时使用docker的时候总是忘了一些命令，每次使用忘了都要重新查，比较麻烦。现在整理一下。

**文中命令中大写字母为要替换的变量。**

* any list
{:toc}

---

### 添加用户到docker组
非root用户使用docker时总是要在命令前加`sudo`，并且要输入密码，使用比较麻烦。可以创建一个`docker`组，并把用户加入到这个组内，这样以后使用时就可以不加`sudo`啦。

#### 步骤

1. 检查当前计算机中是否有名为`docker`的组。

    ```sh
    sudo cat /etc/group | grep docker
    ```

2. 如果上一步没有`docker`组的话就创建一个。

    ```sh
    # 指定ID为999
    sudo groupadd -g 999 docker
    # 也可以不指定
    sudo groupadd docker
    ```

3. 添加相应的用户到`docker`组内。

    ```sh
    # xxx替换为要添加的用户名
    sudo usermod -aG docker xxx
    ```

4. 检查一下是否添加成功。

    ```sh
    cat /etc/group
    ```

5. 重新登陆当前用户或重启docker-daemon。

    ```sh
    sudo systemctl restart docker
    ```

6. 检查是否成功。

    ```sh
    docker info
    ```

7. 如果有错误，看错误提示是`docker.sock`没有权限，给一下权限即可。

    ```sh
    sudo chmod a+rw /var/run/docker.sock
    ```

---

### 拉取镜像
```sh
docker pull [OPTIONS] NAME[:TAG|@DIGEST]
```

---

### 通过DockerFile创建镜像
```sh
docker build -t REPOSITORY:TAG -f ./DOCKERFILE_PATH .
```

说明：

- \-t：生成镜像的仓库名和TAG名。
- \-f：从指定DOCKERFILE文件中创建。
- 最后的路径是指：指定镜像构建过程中的上下文环境的目录。

---

### 查看计算机中有的`Docker Images`
```sh
docker imags
```
其中`REPOSITORY:TAG`组成了`IMAGE_NAME`，`IMAGE_NAME`和第三列`IMAGE_ID`都可以表示一个独一无二的镜像。

---

### 创建容器
使用以下命令创建容器后，会返回一个容器的id，也是和容器一一对应的。
#### 两个命令
1. `docker create`：创建容器，处于**停止**状态。
2. `docker run`：创建并启动容器。

#### 后台型容器
```sh
docker run --name=USER_MISSION -d IMAGE_ID/IMAGE_NAME /bin/bash -c "while true; do echo hello world; sleep 1; done"
```

说明：

- \-\-name：表示容器的名字，本人比较喜欢`用户_任务`的方式命名，一眼就知道这个容器的作用，管理起来非常方便。也可以不指定，docker会随机的分配一个名字。
- \-d：表示容器在后台运行。
- \-c：表示容器的CPU优先级，默认情况下，所有的容器拥有相同的CPU优先级和CPU调度周期，但你可以通过Docker来通知内核给予某个或某几个容器更多的CPU计算周期。比如，我们使用-c或者–cpu-shares =0启动了C0、C1、C2三个容器，使用-c/–cpu-shares=512启动了C3容器。这时，C0、C1、C2可以100%的使用CPU资源（1024），但C3只能使用50%的CPU资源（512）。如果这个主机的操作系统是时序调度类型的，每个CPU时间片是100微秒，那么C0、C1、C2将完全使用掉这100微秒，而C3只能使用50微秒。(引自[这里](https://blog.csdn.net/u010246789/article/details/53958662))
- 中间的`IMAGE_NAME`或`IMAGE_ID`指定特定的镜像。
- 最后的bash命令是一个命令循环，使容器一直执行某个任务。

#### 前台型容器
##### 创建一个正常的容器

```sh
docker run -it --name=USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

##### 挂载本地目录

  本来当容器关闭时，容器启动时改动的文件就都没有了，如果内部有什么数据也只能专门提前移动出来。做路径映射就相当于在docker的内部和外部都对路径中的数据做管理，即使docker非正常关闭，数据也不会丢失。还是**推荐**做这个操作的，毕竟容器的目的就是做操作，而不是数据管理。

```sh
docker run -it -v LOCAL_PATH:CONTAINER_PATH --name USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

说明：

- \-v：表示目录挂载，后面要跟映射关系，`LOCAL_PATH`表示本地路径，`CONTAINER_PATH`表示docker容器内的路径，==必须==使用绝对路径(以`/`开头)。如果容器内不存在，则会自动创建。

##### 添加端口映射
有的容器需要和外部网络通信，需要有一些可访问的端口，比如vscode的Remote-ssh、jupyter-lab，这时就需要添加端口映射了。

```sh
docker run -it -p 22 -p 8888 -p 8080 -v LOCAL_PATH:CONTAINER_PATH --name USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

说明：

  - \-p：端口映射的参数，后面要跟上**映射表**。有多种方式进行映射，上面的写法表示映射本地的一个*随机*端口到容器的指定端口，本人比较喜欢这种写法。也可以写为`LOCAL_PORT:CONTAINER_PORT`的形式，表示本地的指定端口映射到容器的指定端口。
  - \-P：这个参数是对容器暴露的端口进行随机映射，后面**不需要**跟参数。容器暴露的端口可以在`DockerFile`中指定。

##### 设置网络连接类型
~~之前使用Docker时从来没有注意过网络问题，直到有一次发现Docker内怎么也无法联网，后来是通过设置这个参数才成功的。~~ 问题已经解决，详情见下一条。改成host模式是可以解决问题，但是外挂端口的时候还会有坑，还是bridge好用。

```sh
docker run -it --net="bridge" --name USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

说明：
- \-\-net：设置网络连接类型，支持 bridge/host/none/container: 四种类型。

##### 配置DNS
上一条说的bridge模式无法联网的问题找到了，是DNS没有配置，在启动时加上DNS信息或直接配置Docker默认的DNS即可。

```sh
docker run -it --dns=114.114.114.114 --name USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

##### 设置共享内存(shared memory)大小
这个问题主要是最近在跑pytorch框架的时候，把`dataloader`中的`workers`设置成大于0的数就会出错。手动设置下共享内存的大小就可以解决。
```sh
docker run -it --shm-size=8G IMAGE_NAME/IMAGE_ID /bin/bash
```

##### 添加gdb调试权限
```sh
docker run -it --cap-add=SYS_PTRACE --name USER_MISSION IMAGE_NAME/IMAGE_ID /bin/bash
```

---

### 查看容器
#### 查看正在运行的容器
```sh
docker ps
```

#### 查看所有容器，包括关闭的
```sh
docker ps -a
```

#### 查看容器的超详细信息
会输出一个json形式的文本，里面包括非常详细的配置信息。
```sh
docker inspect IMAGE_ID/CONTAINER_ID
```

---

### 退出容器
这里指的是在容器内使用终端命令的时候做的操作。

#### 退出关关闭容器
使用快捷键`Ctrl`+`D`即可。

#### 暂退容器
如果想暂时退出容器，但又不希望容器停止，使用快捷键`Ctrl`+`p`+`q`。

---

### 停止正在运行的容器
```sh
docker stop CONTAINER_ID
```

---

### 终端打开已启动的容器
```sh
docker attach CONTAINER_ID
```

---

### 启动已停止的容器
```sh
docker start CONTAINER_ID
```

---

### 保存容器
```sh
docker commit -a "wyg1997" -m "hello world" CONTAINER_ID REPOSITORY:TAG
```

说明：

- \-a：镜像作者。
- \-m：注释信息。
- REPOSITORY：镜像仓库。
- TAG：标签，表示镜像的版本，==非常不建议==使用`latest`。

---

### 删除操作
#### 删除镜像
```sh
docker rmi IMAGE_ID
```

#### 删除容器
```sh
# 如果容器还在运行状态记得关一下
# docker stop CONTAINER_ID
docker rm CONTAINER_ID
```

#### 批量删除名为none的镜像
这些镜像可能是后面新的镜像`REPOSITORY:TAG`有重复，前面的就会被改名为\<none\>:\<none\>。

```sh
docker rmi $(docker images | grep "none" | awk '{print $3}')
```

#### 批量删除已经停止的容器
```sh
docker rm $(docker ps -a | grep "Exited" | awk '{print $1 }')
```

