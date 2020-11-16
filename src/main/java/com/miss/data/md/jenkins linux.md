# jenkins linux

## 一、环境部署

注：以下操作都是在 /home/deploy/ 路径下

***

### 1、Java

```xml
下载：wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u141-b15/336fa29ff2bb4ef291e347e091f7f4a7/jdk-8u141-linux-x64.tar.gz"
解压：tar zxvf jdk-8u141-linux-x64.tar.gz
重命名：mv jdk1.8.0_141 jdk8
环境变量配置：
vim /etc/profile
加入以下：
export JAVA_HOME=/home/deploy/jdk8
export CLASSPATH=.:$JAVA_HOME/lib:$JAVA_HOME/jre/lib:$CLASSPATH
export PATH=$JAVA_HOME/bin:$JAVA_HOME/jre/bin
保存：:wq
刷新：source /etc/profile
验证：java -version
```

### 2、Maven

```
maven 历史版本：https://archive.apache.org/dist/maven/maven-3/
下载：wget https://archive.apache.org/dist/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
解压：tar zxvf apache-maven-3.3.9-bin.tar.gz
mv apache-maven-3.3.9-bin maven
环境变量配置：
vim /etc/profile
加入以下：
export MAVEN_HOME=/home/deploy/maven
export PATH=$MAVEN_HOME/bin:$PATH
保存：:wq
刷新：source /etc/profile
验证：mvn -version

maven 目录下新建 repository：mkdir repository
maven settings.xml 找开发人员要
```

### 3、Git

```
下载：wget https://github.com/git/git/archive/v2.26.0.zip
下载依赖：yum install curl-devel expat-devel gettext-devel openssl-devel zlib-devel gcc perl-ExtUtils-MakeMaker
解压: unzip v2.26.0.zip
移除旧: yum -y remove git
进入git目录: cd /home/deploy/git-2.26.0
编译git源码: make prefix=/usr/local/git all
安装：make prefix=/home/deploy/git install
环境变量配置：
vim /etc/profile
加入以下：
export PATH=$PATH:/home/deploy/git/bin
保存：:wq
刷新：source /etc/profile
验证：git --version
设置git：
git config --global user.name "name"
git config --global user.email "email"
生成密钥：ssh-keygen -t rsa -C "email@qq.com"
```

### 4、Node JS

```
下载：wget https://nodejs.org/dist/v12.13.0/node-v12.13.0-linux-x64.tar.gz
解压：tar -xvf node-v12.13.0-linux-x64.tar.gz
更改名称：mv node-v12.13.0-linux-x64 nodejs
环境变量配置：
vim /etc/profile
加入以下：
export NODE_HOME=/home/deploy/nodejs
export PATH=$PATH:$NODE_HOME/bin
保存：:wq
刷新：source /etc/profile
验证：node -v
```

### 5、Jenkins

```
下载：wget https://pkg.jenkins.io/redhat-stable/jenkins-2.204.2-1.1.noarch.rpm
安装：rpm -ih jenkins-2.204.2-1.1.noarch.rpm
修改工作目录: vim /etc/sysconfig/jenkins   JENKINS_HOME=/home/deploy/jenkins
修改用户：vim /etc/sysconfig/jenkins   JENKINS_USER="root"
添加java环境：vim  /etc/init.d/jenkins
candidates="
/home/deploy/jdk11/bin/java
"
新建目录 mkdir /home/deploy/jenkins/plugins/
下载启动必要插件：cd /home/deploy/jenkins/plugins/   wget http://ftp.icm.edu.pl/packages/jenkins/plugins/cloudbees-folder/6.12/cloudbees-folder.hpi
修改插件下载源：cd /home/deploy/jenkins/plugins/   vim hudson.model.UpdateCenter.xml   <url>http://mirror.xmission.com/jenkins/updates/update-center.json</url>
/usr/lib/jenkins/jenkins.war    WAR包 
/etc/sysconfig/jenkins       配置文件
/var/lib/jenkins/       默认的JENKINS_HOME目录
/var/log/jenkins/jenkins.log    Jenkins日志文件
启动: service jenkins start
重启: service jenkins restart
停止: service jenkins stop
访问：ip+端口, 第一次登录Jenkins会要求解锁，打开红色标记中的路径，取出password
```



## 二、Jenkins配置

### 1、初始化密码

![image-20200911183024940](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183024940.png)

### 2、安装默认插件

![image-20200911183234782](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183234782.png)

### 3、创建用户登录

![image-20200911183307517](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183307517.png)

### 4、安装插件

![image-20200911183418248](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183418248.png)

```
安装以下插件：
Locale
GitLab
Publish Over SSH
SSH
Maven Integration
NodeJS
Workspace Cleanup
```

### 5、全局工具配置

![image-20200911183512125](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183512125.png)

#### 1）Maven settings 文件配置

![image-20200911183555726](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183555726.png)

#### 2）JDK 配置

![image-20200911183626531](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183626531.png)

#### 3）Git配置

![image-20200911183653627](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183653627.png)

#### 4）Maven 配置

![image-20200911183720597](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183720597.png)

#### 5）Node JS 配置

![image-20200911183748223](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183748223.png)

### 5、其他配置

#### 1）Git 凭证配置

![image-20200911183826573](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183826573.png)

#### 2）配置shell命令

![image-20200911183909517](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183909517.png)

#### 3)添加用户名密码

![image-20200911183946953](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911183946953.png)

#### 4）远程传输文件配置

![image-20200911184027072](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184027072.png)

## 三、Jenkins打包构建

#### 1、基础jar包

#### 1）新建任务

![image-20200911184118480](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184118480.png)

#### 2）源码管理

![image-20200911184143194](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184143194.png)

#### 3）构建触发器

![image-20200911184204119](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184204119.png)

#### 4)构建环境

![image-20200911184229920](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184229920.png)

#### 5）构建配置

![image-20200911184254267](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184254267.png)

注：如果打包失败提示找不到xxx基础依赖，先让开发打包构建后上传到私服后在打包构建

***

### 2、项目war包、jar包构建

#### 1）新建任务

![image-20200911184349501](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184349501.png)

#### 2）源码管理

![image-20200911184411127](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184411127.png)

#### 3）构建环境

![image-20200911184433243](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184433243.png)

#### 4）构建配置

![image-20200911184455080](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184455080.png)

#### 5）构建

![image-20200911184513939](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184513939.png)

pack.sh 脚本代码：

```
#!/bin/bash

name=$1
port=$2

echo "==========================================================================="
echo "`date "+%Y-%m-%d %H:%M:%S"` 删除的旧包为: /home/deploy/${name}/webapps/"
#ls /home/deploy/${name}/webapps/ | grep -v application.properties | xargs rm -rf
rm -fr /home/deploy/${name}/webapps/*
if [ $? -eq 0 ];then
    echo "`date "+%Y-%m-%d %H:%M:%S"` 删除成功"
else
    echo "`date "+%Y-%m-%d %H:%M:%S"` 删除失败" 
fi
echo "`date "+%Y-%m-%d %H:%M:%S"` 删除完毕, 请留意提示是否删除成功"
echo "============================================================================"
echo ""
echo ""

echo "============================================================================"
echo "`date "+%Y-%m-%d %H:%M:%S"` 拷贝新包 ${name} 到 --> ${name}"
cd /home/deploy/jenkins_jobs/workspace/${name}/target/
\cp -rf *.war /home/deploy/${name}/webapps/
echo "============================================================================"
echo ""
echo ""

echo "============================================================================"
echo "端口号为: ${port}"
echo "进程PID为: `netstat -anp|grep ${port}`"
echo "PID: `netstat -tunlp|grep ${port}|awk '{print $7}'|awk -F '/' '{print $1}'|xargs`"
pid="`netstat -anp|grep ${port}`"
echo "`date "+%Y-%m-%d %H:%M:%S"` 杀死进程中...... ${pid}"
sleep 3s
kill -9 $(lsof -i :${port}| awk '{print $2}'| sed -n '2p')
kill -9 $(lsof -i :${port}| awk '{print $2}'| sed -n '2p')

while  [ `netstat -anp|grep ${port}`!="" ]
do
    kill -9 $(lsof -i :${port}| awk '{print $2}'| sed -n '2p')
    echo "`date "+%Y-%m-%d %H:%M:%S"` ${pid} 已杀死"
    sleep 3s
    if [ `netstat -anp|grep ${port}`="" ];then
        echo "进程: `netstat -anp|grep ${port}`"
        echo "`date "+%Y-%m-%d %H:%M:%S"` 确认结束进程"
        break
    fi
done
echo "========================================================================================"
echo ""
echo ""



echo "========================================================================================="
echo "`date "+%Y-%m-%d %H:%M:%S"` 等待启动中......"
sleep 5s
echo "`date "+%Y-%m-%d %H:%M:%S"` 启动 ---> ${name} 项目"
cd /home/deploy/${name}/bin/
sh startup.sh
#sh /home/deploy/${name}/bin/startup.sh

sleep 5s
a=`netstat -anp|grep ${port}|wc -l`
if [ "${a}" -eq "1" ]&&[[ `netstat -anp|grep ${port}`!="" ]] ;then
    echo "`date "+%Y-%m-%d %H:%M:%S"` ${name} 服务启动 --->  成功"
	sleep 5s
	echo "====================================================================================="
	echo ""
	echo ""

	echo "`date "+%Y-%m-%d %H:%M:%S"` 等待日志输出中......"
	sleep 10s
	echo ""
	echo ""

	echo "===========================================================日志输出==========================================================="
	s1="start Server startup in"
	
	for i in $(seq 1 100)
	do
	    s2=`tail -n 20 /home/deploy/${name}/logs/catalina.out`
	    result=$(echo ${s2} | grep "${s1}")
	
	    if [[ "$result" = "" ]]
	    then
		echo "`date "+%Y-%m-%d %H:%M:%S"` 日志输出 --->  正在启动...... ${i}"
		sleep 2s
	
		if [[ "${i}" = "100" ]]&&[[ `lsof -i :${port}`="" ]]
		then
		    echo "`date "+%Y-%m-%d %H:%M:%S"` 启动 ${name} 失败, 详情见下方日志"
		    break
		else
		    echo "`date "+%Y-%m-%d %H:%M:%S"` 日志输出 --->  启动 ${name} 完毕! ---> 请检查以下最后 100 行日志!!!"
		    break
		fi
	    elif [[ "$result" != "" ]]
	    then
	        echo "`date "+%Y-%m-%d %H:%M:%S"` 日志输出 --->  启动 ${name} 完毕! ---> 请检查以下最后 100 行日志!!!"
		break
	    fi
	done
	echo "==========================================================================================================="
	echo ""
	echo ""
	
	sleep 3s
	tail -n 100 /home/deploy/${name}/logs/catalina.out
	echo ""
	echo ""
	echo "=========================== `date "+%Y-%m-%d %H:%M:%S"` ${name} 打包结束, 请检查日志是否启动成功 =============================="
	echo ""
	echo ""
	echo "再次检查: "
        echo "`ps -ef | grep ${port}`"
else
    echo "${time} ${name} 服务启动 --->  失败, 启动命令执行失败"
fi
echo "当前时间: `date "+%Y-%m-%d %H:%M:%S"`"
exit 0
```

### 前端打包构建

#### 1）新建任务

![image-20200911184609867](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184609867.png)

### 2）源码管理

![image-20200911184628809](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184628809.png)

#### 3）构建环境

![image-20200911184647667](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184647667.png)

#### 4）构建配置

![image-20200911184706858](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184706858.png)

注：需要先下载依赖

front.sh 脚本代码：

```
#!/bin/bash

export NODE_HOME=/home/deploy/nodejs
export PATH=$PATH:$NODE_HOME/bin

name=$1
comm=$2

echo ""
echo "`date "+%Y-%m-%d %H:%M:%S"` 开始打包, 请稍等......"
cd /home/deploy/jenkins_jobs/workspace/${name}/
echo `npm run ${comm}`
sleep 10s

echo ""
cd /home/deploy/jenkins_jobs/workspace/${name}/
cp -rf dist /home/deploy/jenkins_jobs/workspace/${name}/${name}

if [ -d "/home/deploy/jenkins_jobs/workspace/${name}/${name}/" ];then
    echo "${name} 文件夹存在, 构建成功"
    echo ""
    
    echo "拷贝新包 ${name} 到 ---> ip:/home/deploy/nginx/html/ 目录下"
    \scp -P port -r /home/deploy/jenkins_jobs/workspace/${name}/${name} root@ip:/home/deploy/nginx/html/
    echo ""

    echo "`date "+%Y-%m-%d %H:%M:%S"` 打包全部完成"
    
    sleep 3s
    echo ""
    echo "`date "+%Y-%m-%d %H:%M:%S"` 打包项目为: ${name}"
    echo "删除 ${name} 下除了 node_modules 依赖外所有文件"
    cd /home/deploy/jenkins_jobs/workspace/${name}/
    ls | grep -v node_modules | xargs rm -fr

else 
    echo "${name} 文件夹不存在, 打包失败"
fi
```

### 四、jenkins问题解决

### 1、找不到基础包

```
Jenkins从git上拉取jar包的时候可能会有个别包没有下载到本地，这时候就要用命令行本地安装一下
jenkins从git拉取基础jar包报错如图：
```

![image-20200911184810483](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184810483.png)

```
黑框里依次为：
组ID：-DgroupId=com.azazar
jar包名称：-DartifactId=bitcoin-json-rpc-client
jar包版本：-Dversion=1.0.1-SNAPSHOT.jar
```

```
maven构建基础jar包命令：
mvn install:install-file -Dfile=D:\apache-maven-3.6.1\repository\bitcoin-json-rpc-client-1.0.1-SNAPSHOT.jar  -DgroupId=com.azazar -DartifactId=bitcoin-json-rpc-client -Dversion=1.0.1-SNAPSHOT -Dpackaging=jar
```

```
解析：
-Dfile=：需要安装的jar包绝对路径，如：
-Dfile=D:\apache-maven-3.6.1\repository\bitcoin-json-rpc-client-1.0.1-SNAPSHOT.jar

-DgroupId=：组ID，如：
-DgroupId=com.azazar

-DartifactId=：jar包名称，如：
-DartifactId=bitcoin-json-rpc-client

-Dversion=：jar包版本，类似以下：
-Dversion=1.0.1-SNAPSHOT.jar

注意：jar包带 SNAPSHOT 为快照版本，不带 SNAPSHOT 为正常版本，注意分辨安装
```

### 2、XXX程序包不存在、XXX找不到符号

![image-20200911184904729](C:\Users\王延\AppData\Roaming\Typora\typora-user-images\image-20200911184904729.png)

开发代码问题，让开发修改好以后重新提交代码，重新打包构建即可。

***

### 3、无法删除工作空间

一般出现在Jenkins服务安装在Windows机器上，并且Jenkins勾选了 "构建前删除当前构建空间" 选项，如下图：

​    ![0](http://note.youdao.com/yws/public/resource/669e23ab3b00bb7f9cbc2c95ad27e886/xmlnote/WEBRESOURCE0c7bf74818fa5a951c5f2c3f1c762314/2702)

解决方法：关闭Jenkins所在服务器所有已打开的文件和文件夹即可

### 4、打包构建后提示为黄色

​    ![0](http://note.youdao.com/yws/public/resource/669e23ab3b00bb7f9cbc2c95ad27e886/xmlnote/WEBRESOURCE53178c7ed9d8b0a60bd89912b0a025d0/2710)

一般为网络不稳定，上传文件失败，或执行本地、远程shell命令失败，如下图：

​    ![0](http://note.youdao.com/yws/public/resource/669e23ab3b00bb7f9cbc2c95ad27e886/xmlnote/WEBRESOURCEca07bcafd5afa220ecfb004028caf7a9/3371)

解决方法：检查shell命令，重新打包。网络问题的话到Jenkins服务器下找到对应项目的工作空间直接拷贝war上传到服务器重启更新

### 5、无法找到XXX文件，****没有那个文件或目录****，XXX****文件或文件夹不存在

一般为shell脚本里文件或文件夹或目录名称写错了

解决方法：检查shell脚本，把shell脚本复制到服务器执行一遍

### 6、打包构建时间过长

一般为网络问题

解决方法：到项目服务器/home/deploy/jenkins/目录下查看上传的项目war包大小变化，太慢的话直接到Jenkins服务器拷贝war到项目服务器重启更新

### 7、其他问题解决

出现问题Jenkins的控制台输出都会有明确提示信息的，在控制台输出页面 “Ctrl + F”，全局搜索 “ERROR”，如下图

​    ![0](http://note.youdao.com/yws/public/resource/669e23ab3b00bb7f9cbc2c95ad27e886/xmlnote/WEBRESOURCEb9b29e663d7c883d33c28c4cc8f810e2/3374)

 ERRO后面就是具体的报错信息，报错信息会明确输出的