## Docker安装部署

### 环境

- 系统 CentOS Linux release 7.4.1708

### 部署

#### 安装docker

1、首先查看一下docker版本，如果没安装就是用yum install -y docker 命令安装一下 

```
// docker version
yum install -y docker
```

***

2、如果需要Tomcat，还要用docker拉取一下Tomcat镜像。

```
docker pull tomcat
```

3、 启动 docker服务：

```
service docker start   //启动docker service
```

4、如果想要关闭docker服务：

```
service docker stop
```

5、 获取镜像：

可以通过 docker search [镜像名]	 查看是否有可用镜像

```
sudo docker pull NAME[:TAG]
sudo docker pull centos:latest

// 比如获取tomcat 镜像：
docker search jdk
```

6、查看所有镜像：

```
docker images
```

 7、删除镜像，从本地删除一个已经下载的镜像

```
sudo docker rmi IMAGE [IMAGE...]
sudo docker rmi centos:latest
```

8、启动一个停止的容器：

```
docker start webdemo
```

9、 罗列所有的docker容器：包含了启动的Docker和没有启动的容器Docker

```
docker ps -a;
```

10、启动容器：

```
docker start webdemo
```

11、可能端口会被占用，我们可以先把运行的docker实例停掉，然后再删掉，

```
docker kill webdemo

docker rm webdemo
```

12、进入其中一个容器：**使用docker exec命令**

这个命令使用exit命令后，不会退出后台，一般使用这个命令，使用方法如下

```
docker exec -it webdemo /bin/bash
```

13、拷贝文件

从主机复制到容器 sudo docker cp host_path containerID:container_path

从容器复制到主机 sudo docker cp containerID:container_path host_path

请注意，以上这两个命令都是在主机中执行的，不能再容器中执行

```
docker cp /root/software/docker.war webdemo:/
docker cp demo:/abc.txt  /root/software/
```

14、一个服务器可以有多个容器，但是一个服务器只需要安装一个tomcat,然后每个容器可以挂载到不同的tomcat的端口上面，就是这关系。前台访问

```
http://192.168.163.128:80/docker;  //这是第一个容器webdemo

http://192.168.163.128:81/docker; //这是第二个容器webdemo1
```



#### 创建jdk的Dockerfile

```
#1.指定基础镜像，并且必须是第一条指令
FROM centos:7

#2.指明该镜像的作者和其电子邮件
#MAINTAINER share "share@qq.com"

#3.在构建镜像时，指定镜像的工作目录，之后的命令都是基于此工作目录，如果不存在，则会创建目录
WORKDIR /docker/jdk11

#4.一个复制命令，把jdk安装文件复制到镜像中，语法：ADD <src>... <dest>,注意：jdk*.tar.gz使用的是相对路径
ADD jdk.11.tar.gz /docker/jdk11

#5.设置时区
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo 'Asia/Shanghai' >/etc/timezone

#6.配置环境变量
ENV JAVA_HOME=/docker/jdk11/jdk-11.0.3
ENV CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
ENV PATH=$JAVA_HOME/bin:$PATH

#容器启动时需要执行的命令
#CMD ["java","-version"]

```



#### 创建镜像

在 Dockerfile 文件的存放目录下，执行构建动作。

以下示例，通过目录下的 Dockerfile 构建一个 nginx:test（镜像名称:镜像标签）。

**注**：最后的 **.** 代表本次执行的上下文路径。

```
docker build -t jdk11:jdk11
```



### 创建jar包 dockerfile

```
# Docker image for springboot file run
# 1、基础镜像使用java 必有且必在首部位置
FROM vssware/jdk11

# 2、作者 姓名 邮箱 非必须
# MAINTAINER 

# 3、VOLUME 指定了临时文件目录为/tmp。
# 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp 

# 4、将jar包添加到容器中并更名为***.jar  为了项目路径清晰，可自定路径，如非配置分离，只添加一个jar包至镜像即可
ADD saas-api-0.0.1-SNAPSHOT.jar share/saas-api.jar
ADD lib	share/lib
ADD resources	share/resources

# 5、运行ja
RUN bash -c 'touch /share/saas-api.jar'
ENTRYPOINT ["java","-jar","/share/saas-api.jar"]
```



