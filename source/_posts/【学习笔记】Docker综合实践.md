---
title: 【学习笔记】Docker综合实践
date: 2021-04-24 22:16:40
math: false
index_img: /img/docker.jpg
tags: ['Docker']
categories: 
- notes
---
Datawhale Docker学习笔记第六篇
<!--more--->

在没有学习docker之前，部署项目都是直接启动文件，比如java项目就是java –jar xxxx.jar的方式，python项目就是python xxxx.py。如果采用docker的方式去部署这些项目，一般有两种方式，以jar包项目为例

# 方式一、挂载部署

这种方式类似于常规部署，通过数据卷的方式将宿主机的jar包挂载到容器中，然后执行jar包的jdk选择容器中的而非采用本地的。

1. 将jar包上传到服务器的指定目录，比如/root/docker/jar。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yobtp2tj30o007njt5.jpg)

2. 通过docker pull openjdk:8命令获取镜像

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yord00xj30o006bju3.jpg)

3. 编写docker-compose.yml文件

```yaml
version:'3.0'

services:

  java:
    image: docker.io/openjdk
    restart:always
    container_name: myopenjdk
    ports:
      - 8080:8001
    volumes:
      - /root/docker/jar/xxxx.jar:/root:z
      - /etc/localtime:/etc/localtime
    environment:
      - TZ="Asia/Shanghai"
    entrypoint: java -jar /root/xxxx.jar
    mynetwork:
      ipv4_address: 192.168.1.13

networks:
  mynetwork:
   ipam:
     config:
      - subnet: 192.168.1.0/24
```

参数解释：

- build 指定dockerfile所在文件夹的路径 context指定dockerfile文件所在路径 dockerfile指定文件的具体名称

- container_name 指定容器名称

- volumes 挂载路径  z是用来设置selinux，或者直接在linux通过命令临时关闭或者永久关闭

- ports 暴露端口信息

- networks是用来给容器设置固定的ip

4. 执行命令docker-compose up –d启动jar包, 可以通过docker ps查看容器是否在运行，需要注意的是默认查看所有运行中的容器，如果想查看所有容器，需要添加参数-a

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yp7mo2wj30o0057aay.jpg)

5. 注意如果容器启动失败或者状态异常，可以通过docker logs查看日志

6. 通过docker inspect myopenjdk查看容器详细信息，可以看到容器ip已经设置成功

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9ypl8wgdj30o009wabq.jpg)

7. 然后在虚拟机中打开浏览器输入jar包项目的访问地址，就可以看到运行的项目，需要注意访问端口是映射过的端口而非项目实际端口

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yq1vynoj30o00dytaj.jpg)

## 方式二、构建镜像部署

1. 将jar包上传到服务器的指定目录，比如/root/docker/jar。

2. 在该目录下创建Dockerfile文件，通过vim等编辑工具在Dockerfile中编辑以下内容

```dockerfile
FROM java:8
MAINTAINER YHF
LABEL description=”learn docker”
ADD xxx.jar
EXPOSE 8001
ENTRYPOINT [“java”,”-jar”,”xxxx.jar”]
```

参数解释：

- FROM java:8 指定所创建镜像的基础镜像

- MAINTAINER yhf 指定作者为yhf

- LABEL 为生成的镜像添加元数据标签信息

- ADD xxxx.jar 添加内容到镜像

- EXPOSE 8080 声明镜像内服务监听的端口

- ENTRYPOINT 指定镜像的默认入口命令，支持两种格式ENTRYPOINT[“java”,”-jar”,”xxxx.jar”]；ENTRYPOINT java –jar xxxx.jar。注意每个dokcerfile中只能有一个ENTRYPOINT，如果指定多个只有最后一个生效。

4. Dockerfile构建完成以后可以通过命令docker build构建镜像，然后再运行容器，这里咱们用docker-compose命令直接编排构建镜像和运行容器。

5. 编写docker-compose.yml文件

 ```yaml
version: '3'

services:

  java_2:
    restart: always
    image: yhfopenjdk:latest
    container_name: myopenjdk
    ports:
      - 8080:8001
    volumes:
      - /etc/localtime:/etc/localtime
    environment:
      - TZ="Asia/Shanghai"
    entrypoint: java -jar /root/datawhale-admin-1.0.0.jar
    networks:
      mynetwork:
         ipv4_address: 192.168.1.13

networks:
  mynetwork:
   ipam:
     config:
      - subnet: 192.168.1.0/24
 ```

参数解释同方式一：

6. 执行docker-compose up –d直接启动基于文件构建的自定义镜像，如果镜像不存在会自动构建，如果已存在那么直接启动。如果想重新构建镜像，则执行docker-compose build。如果想在执行compose文件的时候重构，则执行docker-compose up –d –build。

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yqi1lv0j30o007ign6.jpg)

此使通过dockerfile文件构建的镜像已经创建

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yqxa7t1j30o005qdi5.jpg)

通过镜像运行的容器已经正常启动，可以通过docker ps查看容器是否在运行，需要注意的是默认查看所有运行中的容器，如果想查看所有容器，需要添加参数-a

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yra1jg1j30o0013q33.jpg)

7. 在浏览器中输入访问路径可以看到项目已经正常运行

![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9yrmnnzzj30o00d975v.jpg)