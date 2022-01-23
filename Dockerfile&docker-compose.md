# Dockerfile&docker-compose

## 区别

1.dockerfile: 构建镜像；

2.docker run: 启动容器；

3.docker-compose: 负责构建和运行由多个容器组成的应用程序的工具；

## 指令

### Dockfile

| 指令       | 说明                                                         | 示例                                 |
| ---------- | ------------------------------------------------------------ | ------------------------------------ |
| FROM       | 设置基础镜像，必须为Dockerfile的第一个指令                   | FROM ubuntu:18.04                    |
| MIANTAINER | 说明作者信息                                                 | MIANTAINER author \<author@mail.com> |
| RUN        | 设置容器内执行的指令                                         | RUN apt-get update -y                |
| COPY       | 用于构建环境的上下文复制文件至镜像<br>当使用本地目录为源目录时，推荐使用 COPY | COPY HostFile DockerFile             |
| ADD        | 添加上下文文件或远程文件到镜像，对压缩文件会自动解压         | ADD HostFile DockerFile              |
| CMD        | 用来指定启动容器时默认执行的命令。<br>每个 Dockerfile 只能有 CMD 命令 如果指定了多条命令，只有最后一条会被执行 | CMD  ['./shell.sh']                  |
| VOLUME     | 指定数据卷<br>改指令之后的所以命令不可以对改数据卷有任何修改 | VOLUME  /data                        |
| USER       | 设定容器运行时的UID或用户名                                  | USER　TOM                            |
| WORKDIR    | 设定后续RUN、CMD等指令的工作目录                             | WORKDIR /data                        |

```dockerfile
#指定基础镜像
FROM python:3.6   
#设置容器内执行的指令
RUN groupadd -r uwsgi &&\
    useradd -r -g uwsgi uwsgi&&\
    apt-get update -y &&\
    apt-get install python-dev  -y &&\
    rm -rf /var/lib/apt/lists/*&&\
    #pip install --upgrade pip &&\    
    pip install Flask==0.10.1 uWSGI redis==2.10.3 requests==2.5.1
#设置appw为工作目录
WORKDIR /app
#把本地app、cmd.sh文件复制到镜像中
COPY app /app
COPY cmd.sh /
#说明容器会监听的端口，该指令本身不会对网络有实质性的改变
EXPOSE 9090 9191
USER uwsgi
CMD ["/cmd.sh"]
```

### docker-compose

| 指令  | 说明                                                 | 示例                 |
| ----- | ---------------------------------------------------- | -------------------- |
| up    | 启动自定义的容器，通常结合-d使用                     | docker-compose up -d |
| build | 重新构建由Dockerfile构建的镜像<br>需要更新镜像时使用 | docker-compose build |
| stop  | 停止容器                                             | docker-compose stop  |

compose会保留原来容器中所有的旧的数据卷，这意味着即使容器更新后，数据库或缓存也依旧在容器内，着很可能造成混淆，因此要特别小心。

```yaml
docker_build_by_compose: #声明构建容器的名称
  build: .               #说明镜像由当前目录`.`下的Dockerfile构建。相当于docker build .
  ports:                 #声明开放的端口，相当于docker run -p 
          - "5000:5000"
  environment:           #声明容器环境变量，相当于docker run -e 
          ENV : DEV
  volumes:			     #声明数据卷，相当于docker run -v
          - ./app:/app
  links:                 #声明容器互联，相当于docker run --link dnmonster:dnmonster
          - dnmonster
          - redis
dnmonster:               #定义新容器，且声明镜像来源
          image: amouat/dnmonster:1.0

redis:
          image: redis:3.0
```

