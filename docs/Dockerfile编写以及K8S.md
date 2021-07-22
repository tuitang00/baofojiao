# Dockerfile的编写以及K8S



## Dockerfile

### 组成部分

- FROM: 基础镜像
- MAINTAINER: 维护者`LABEL maintainer="maintainer@gmail.com"`
- RUN: 需要基于基础镜像进行的操作. 由于每一次RUN都会新增一层镜像, 所以在编写的时候需要规避无意义的多次执行, 这样只会导致镜像体积的膨胀. 但是分层也有好处, 在重复build的情况下, 已经build成功的镜像分层不需要重复构建, 可以节省时间. 
- ENV: 设置环境变量
- ADD: 拷贝文件
- WORKDIR: 工作路径切换
- VOLUME: 目录挂载, 一般会挂载宿主机的部分目录空间
- EXPORT: 声明对外开放的端口
- CMD: 启动命令, 在执行docker run的时候会执行的内容. 但是需要注意, 如果CMD后面的命令执行结束了, 容器也会对应的退出. 



常用命令

- `docker build`根据dockerfile构建镜像, `-f`指定dockerfile文件, `-t <name>:<tag>`指定镜像名称以及版本 
- `docker images`查看所有的docker镜像
- `docker run`启动容器
- `docker ps -a`查看运行中的容器

| 查看                                                         |                                                              |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| docker images                                                | 列出所有镜像(images)                                         |
| docker ps                                                    | 列出正在运行的容器(containers)                               |
| docker ps -a                                                 | 列出所有的容器                                               |
| docker pull centos                                           | 下载centos镜像                                               |
| docker top ‘container’                                       | 查看容器内部运行程序                                         |
| 容器                                                         |                                                              |
| docker exec -it 容器ID sh                                    | 进入容器                                                     |
| docker stop ‘container’                                      | 停止一个正在运行的容器，‘container’可以是容器ID或名称        |
| docker start ‘container’                                     | 启动一个已经停止的容器                                       |
| docker restart ‘container’                                   | 重启容器                                                     |
| docker rm ‘container’                                        | 删除容器                                                     |
| docker run -i -t -p :80 LAMP /bin/bash                       | 运行容器并做http端口转发                                     |
| docker exec -it ‘container’ /bin/bash                        | 进入ubuntu类容器的bash                                       |
| docker exec -it /bin/sh                                      | 进入alpine类容器的sh                                         |
| docker rm `docker ps -a -q`                                  | 删除所有已经停止的容器                                       |
| docker kill $(docker ps -a -q)                               | 杀死所有正在运行的容器，$()功能同``                          |
| 镜像                                                         |                                                              |
| docker build -t wp-api .                                     | 构建1个镜像,-t(镜像的名字及标签) wp-api(镜像名) .(构建的目录) |
| docker run -i -t wp-api                                      | -t -i以交互伪终端模式运行,可以查看输出信息                   |
| docker run -d -p 80:80 wp-api                                | 镜像端口 -d后台模式运行镜像                                  |
| docker rmi [image-id]                                        | 删除镜像                                                     |
| docker rmi $(docker images -q)                               | 删除所有镜像                                                 |
| docker rmi $(sudo docker images --filter "dangling=true" -q --no-trunc) | 删除无用镜像                                                 |



## K8S文件

