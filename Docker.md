# Docker概览

## Docker在开发和运维中的优势

1. 快速部署与交付

   容器是轻量的，使用镜像构造标准化的开发、测试环境，实现开发、测试环境的统一，便于之后的快速迭代和部署，并且整个过程可见。

2. 资源的高效利用

   相对于传统的虚拟化方式，容器与主机的操作系统共享资源，docker容器在操作系统层实现虚拟化，避免像传统虚拟化技术在硬件层进行虚拟化，使得多出了虚拟机管理程序和虚拟机操作系统层，提高了资源利用率和容器相应速度。

3.  便捷的更新管理方式

   可通过修改dockerfile自动化实现容器的修改管理。

## 三大核心

### 镜像

镜像类似虚拟机镜像，提供一个只读模板。

镜像由多个不同等“层”组成，每一个层都是只读的，dockerfile中的RUN、COPY、ADD都会添加层数，层数过多会导致镜像臃肿，为此要减低层数。诸如把多个UNIX命令放在同一个RUN指令中。

**注意：**镜像本身是只读的，容器在启动之后会在镜像的最上层构建一个可写层。

#### 常规命令

##### 1.增

| 指令                         | 说明     | 示例               |
| ---------------------------- | -------- | ------------------ |
| docker pull  \<image\<:tag>> | 拉取镜像 | docker pull ubuntu |

##### 2.删

| 指令                                             | 说明                                                         | 示例                    |
| ------------------------------------------------ | ------------------------------------------------------------ | ----------------------- |
| docker rmi  \<image><br>docker image rm \<image> | 删除镜像                                                     | docker rmi nginx:latest |
| docker image prune -f                            | 使用Docker一段时间后，系统中可能会遗留一些临时的镜像文件，以及一些没有被使用的镜像，可以通过 该命令来进行清理。 |                         |

当镜像存在容器依赖，要删除该镜像时，需先`docker rm <containerID>`删除容器，再删除镜像`docker rmi <imageID>`

##### 3.改

| 指令                                       | 说明                               | 示例                        |
| ------------------------------------------ | ---------------------------------- | --------------------------- |
| docker tag \<image:Tag> \<NewImage:NewTag> | 为镜像添加标签dokcer tag d55 ql:v1 | dokcer tag mysql:last ql:v1 |

##### 4.查

| 指令                                                         | 说明               | 示例                                                         |
| ------------------------------------------------------------ | ------------------ | ------------------------------------------------------------ |
| docker image<br>docker image ls                              | 查看本地已有镜像   |                                                              |
| docker inspect  \<image:tag><br>docker inspect -f \<image:tag> | 查看镜像的详细情况 | docker inspect mysql:5.7<br>docker inspect  -f {{".Id"}} mysql:5.7 |
| dokcer search \<image>                                       | 搜寻镜像           | docker search --filter=is-official=true nginx                |

镜像大小信息只是表示了该镜像的逻辑体积大小，实际上由于相同的镜像层本地只会存储一份，物理上占用的存储空间会小于各镜像逻辑体积之和。

##### 5.创建镜像

```bash
#基于已有的容器创建
docker commit -m "this is message you add" -a "this is author message you add"  nginx:v1

#基于dockerfile创建
docker build -t MyImage:MyTag .
#-t 镜像名:标签  设定镜像的信息
```

##### 6.载入和保存镜像

```bash
#保存镜像
docker save -o nginx_v1.tar nginx:v1
#载入镜像
docker load -i nginx_v1.tar
#or
docker load < nginx_v1.tar
```

### 容器

容器是镜像的一个运行实例。

#### 常规指令

##### 1.增

| 指令                                     | 说明                                                   | 示例                            |
| ---------------------------------------- | ------------------------------------------------------ | ------------------------------- |
| docker create -it \<image:tag>           | 创建容器<br>-i: 保持标准输入打开<br>-t: 分配一个伪终端 | docker create -it nginx:latest  |
| docker run -it \<containerID> /bin/bash  | 运行容器                                               | docker run -it ubuntu /bin/bash |
| docker exec -it \<containerID> /bin/bash | 進入容器                                               |                                 |



##### 2.删

| 指令                        | 說明     | 示例 |
| --------------------------- | -------- | ---- |
| docker rm -f \<containerID> | 刪除容器 |      |

-f，--force=false:是否强行终止并删除一个运行中的容器;
		-l，--link=false:删除容器的连接，但保留容器;
		-v，--volumes=false:删除容器挂载的数据卷。

##### 3.改

| 指令                                                         | 说明     | 示例                                   |
| ------------------------------------------------------------ | -------- | -------------------------------------- |
| docker pause \<containerID>                                  | 暫停容器 | docker pause nginx                     |
| docker stop \<containerID>                                   | 終止容器 | docker stop nginx                      |
| docker restart \<containerID>                                |          | docker restart nginx                   |
| docker export -o \<container.tar> \<containerID>             | 导出容器 | docker export -o nginx_v1.tar nginx:v1 |
| docker import  [-c]--change[=[]]] [-m] --message[=MESSAGE]] file  [REPOSITORY [:TAG]] | 导入容器 | docker import  nginx_v1.tar -nginx:v1  |

##### 4.查

| 指令                       | 说明             | 示例            |
| -------------------------- | ---------------- | --------------- |
| docker logs \<containerID> | 查看容器输出信息 | docker logs ce5 |
| docker ps -qa              | 查看所有容器ID   |                 |

### 仓库

共有仓库和私有仓库

## 数据管理

### 数据卷

数据卷可以在容器之间**共享和重用**，容器间传递数据将变得高效与方便； 

对数据卷内数据的**修改会立马生效**，无论是容器内操作还是本地操作； 

对数据卷的**更新不会影响镜像**，解摘开应用和数据 ；

卷会一直存在 ，直到没有容器使用，**可安全地卸载**。

```bash
docker run -it --rm -p 80:8080 --name testfile -v /hostdata:/dockerdata busybox
 #Docker 载数据卷的默认权限是读写rw ，用户也可以 ro 定为只读
docker run -it --rm -p 80:8080 --name testfile -v /hostdata:/dockerdata:ro busybox
```

### 数据卷容器

数据卷容器也是一个容器，但是它的目的是专门提供数据卷给其他容器挂载。

```bash
#构建数据卷容器
docker run  -it --rm -v /dockerdata  --name db busybox 
#其他容器挂载数据卷容器的数据卷
docker run -it --rm --volumes-from db --name client1 busybox
```

#### 数据备份与恢复

```bash
#构建数据卷容器
docker run  -it --rm -v /dockerdata  --name db busybox 

#备份
docker run -it --rm --volumes-from db --name client1  -v $(pwd):/backup  busybox tar cvf /backup/backup.tar /dockerdata 
#--volumes-from 挂载数据卷容器的数据卷
#-v $(pwd):/backup 挂载本地数据卷到client1容器的backup目录上
# tar cvf /backup/backup.tar /dockerdata  把db容器上的数据打包备份到clienr1的/bakup目录下，即本机的目录下。

#恢复
docker run -it --rm --volumes-from db --name client2  -v $(pwd):/backup  busybox  tar xvf /backup/backup.tar
```

## 端口映射

```bash
#-p HostPort:DockerPort
docker run --rm -d  --name nginx -p 80:80 nginx
```

## 容器互联

```bash
#创建容器A
docker run -it --rm --name db   busybox 
#创建容器B并与容器A进行交互
#--link  容器名：别名
docker run -it --rm -p 80:80 --name client --link db:dockerdb busybox
```

​       

