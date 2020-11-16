# Harbor镜像仓库搭建及简单使用

## 一、背景

容器应用的开发和运行离不开可靠的镜像管理。从安全和效率等方面考虑，部署在私有环境内的Registry是非常必要的。Project Harbor是由VMware公司中国团队为企业用户设计的Registry server开源项目，包括了权限管理(RBAC)、LDAP、审计、管理界面、自我注册、HA等企业必需的功能，同时针对中国用户的特点，设计镜像复制和中文支持等功能。

## 二、简介

1. 基于角色的访问控制 - 用户与Docker镜像仓库通过“项目”进行组织管理，一个用户可以对多个镜像仓库在同一命名空间（project）里有不同的权限。
2. 镜像复制 - 镜像可以在多个Registry实例中复制（同步）。尤其适合于负载均衡，高可用，混合云和多云的场景。
3. 图形化用户界面 - 用户可以通过浏览器来浏览，检索当前Docker镜像仓库，管理项目和命名空间。
4. AD/LDAP 支持 - Harbor可以集成企业内部已有的AD/LDAP，用于鉴权认证管理。
5. 审计管理 - 所有针对镜像仓库的操作都可以被记录追溯，用于审计管理。
6. 国际化 - 已拥有英文、中文、德文、日文和俄文的本地化版本。更多的语言将会添加进来。
7. RESTful API - RESTful API 提供给管理员对于Harbor更多的操控, 使得与其它管理软件集成变得更容易。
8. 部署简单 - 提供在线和离线两种安装工具， 也可以安装到vSphere平台(OVA方式)虚拟设备。

## 三、环境要求

### 1、docker  （不同Harbor要求的版本不同）

```
# yum install -y yum-utils device-mapper-persistent-data lvm2
# yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# yum -y install docker-ce
# docker --version
Docker version 17.06.2-ce, build cec0b72
# systemctl start docker
# systemctl status docker
//开机自启
# systemctl enable docker
```

### 2、docker -compost （不同Harbor要求的版本不同）

```
# yum -y install python-pip
# pip install --upgrade pip
# pip -V
pip 9.0.1 from /usr/lib/python2.7/site-packages (python 2.7)
# pip install docker-compose
# docker-compose version
docker-compose version 1.16.1, build 6d1ac219
docker-py version: 2.5.1
CPython version: 2.7.5
//OpenSSL更新到最新版本
OpenSSL version: OpenSSL 1.0.1e-fips 11 Feb 2013
```

## 四、Harbor服务搭建

### 1、下载Harbor安装文件

从GitHub上 https://github.com/goharbor/harbor/releases 查看当前可用的harbor版本，一般选择最新稳定版，目前就是这个：
![图片描述](https://img1.sycdn.imooc.com/5e76c74b0001315d08140488.png)

下载该安装包到系统目录中，如下：

```
[root@localhost ~]# mkdir -p /home/temp/
[root@localhost ~]# cd /home/temp/
[root@localhost temp]# wget https://github.com/goharbor/harbor/releases/download/v1.10.1/harbor-offline-installer-v1.10.1.tgz
```

下载速度可能会有点慢，耐心等待下载完成。

### 2、解压安装文件

```
[root@localhost temp]# tar -zxf harbor-offline-installer-v1.10.1.tgz
[root@localhost temp]# tar -zxf harbor.v1.10.1.tar.gz
```

### 3、配置Harbor

```
[root@localhost temp]# ls
[root@localhost temp]# cd harbor
[root@localhost harbor]# ls
[root@localhost harbor]# vi harbor.yml    
```

\#新版本的harbor配置文件已经改为用harbor.yml而不是harbor.cfg

主要修改如下内容：

```
hostname: 你的服务器IP或域名
http:
  # port for http, default is 80. If https enabled, this port will redirect to https port
  port: 你的端口号 #默认是80端口
harbor_admin_password: Harbor12345  #Harbor超级管理员密码
database:
  # The password for the root user of Harbor DB. Change this before any production use.
  password: root123  #数据库管理员密码

data_volume: /data   #配置harbor数据文件，也就是未来镜像文件的存储位置，建议修改，不然直接占用系统盘空间。

#同时注释如下内容，默认启用http，而不是https证书除非你有配置https证书
# https related config
#https:
  # https port for harbor, default is 443
  #port: 443
  # The path of cert and key files for nginx
  #certificate: /your/certificate/path
  #private_key: /your/private/key/path
```

注：其他详细的配置参数详见《harbor.yml参数描述说明》

### 4、安装Harbor

修改完配置文件后，在当前目录执行./install.sh，harbor服务器会自动调用docker-compose分析依赖的镜像并逐个下载，同时自动安装并启动各服务

```
[root@localhost harbor]# ./install.sh
```

若安装时遇到如下错误

![图片描述](https://img1.sycdn.imooc.com/5e76c75e0001156404880083.png)

这就需要升级docker 版本，具体操作详见《docker版本升级简易指南》

升级docker后，重新执行./install.sh，等待程序自动按步骤安装：

![图片描述](https://img1.sycdn.imooc.com/5e76c7680001366909630428.png)

出现如下提示，则表示安装成功：

![图片描述](https://img1.sycdn.imooc.com/5e76c77100018a5306120171.png)

注：安装完后，安装目录下会变成这样，可以看到其中多出一个docker-compose.yml文件，这也是基于harbor.yml生成的供docker-compose调用创建容器的服务编排文件。

![图片描述](https://img1.sycdn.imooc.com/5e76c77c000123b209600044.png)



### 5、查看容器状况

![image-20200917171020643](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200917171020643.png)

容器的作用：

1. harbor-adminserver:harbor系统管理服务
2. harbor-db: 由官方mysql镜像构成的数据库容器
3. harbor-jobservice:harbor的任务管理服务
4. harbor-log:harbor的日志收集、管理服务
5. harbor-ui:harbor的web页面服务
6. nginx:负责流量转发和安全验证
7. registry:官方的Docker registry，负责保存镜像

### 6、登录web

安装完成后，通过配置中设置的IP或域名+端口，即可访问harbor管理控制台，如果端口占用，可以去安装目录下harbor.yml文件中，对应服务的端口映射。

![图片描述](https://img1.sycdn.imooc.com/5e76c7850001763f18970889.png)

测试安装是否成功，使用安装时在harbor.yml中设置的管理员密码，用户名是admin，登录控制台：

![图片描述](https://img1.sycdn.imooc.com/5e76c78e00019c5505970382.png)

登录成功

![图片描述](https://img1.sycdn.imooc.com/5e76c7990001e18319230648.png)

### 7、新建项目

![image-20200917171454630](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200917171454630.png)

新建项目成功，就可以上传镜像。

## 五、镜像上传

### 1、修改docker配置

```
# grep "ExecStart" /usr/lib/systemd/system/docker.service
ExecStart=/usr/bin/dockerd --insecure-registry=192.168.0.1
# systemctl daemon-reload
# systemctl restart docker

注:docker默认使用https传输镜像，这里使用的是http，所以需要指定，无论上传还是下载都需要指定
```



### 2、登录Harbor

```
# docker login -u admin -p Harbor12345 192.168.0.1
Login Succeeded

# docker login 192.168.0.1
Username: admin
Password: 
Login Succeeded

注:第一种方便，第二种安全，实际中请使用第二种
```



### 3、标记镜像并上传

```
# docker tag 9989ji95ffa 192.168.0.1/k8s/test-images:3.0
# docker push 192.168.0.1/k8s/test-images:3.0
The push refers to a repository [192.168.100.100/k8s/test-images]
5f70bf18a086: Pushed 
41ff149e94f2: Pushed 
3.0: digest: sha256:f04288efc7e65a84be74d4fc63e235ac3c6c603cf832e442e0bd3f240b10a91b size: 939
```

### 4、验证上传

![image-20200917172131353](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200917172131353.png)

## 六、镜像下载

### 1、镜像拉取

![image-20200917172309979](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200917172309979.png)

docker pull 192.168.0.1:5000/share/box/box-business:20200917-144609

### 2、运行镜像

docker run --name test-images 192.168.0.1:5000/share/box/box-business:20200917-144609

## 七、相关资料

- Harbor官网  https://goharbor.io/
- Github地址 https://github.com/goharbor/harbor
- 下载地址 https://github.com/goharbor/harbor/releases

