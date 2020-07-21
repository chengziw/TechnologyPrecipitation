## java项目Jar包部署



#### maven项目

```
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-jar-plugin</artifactId>
            <configuration>
                <!-- 不打包资源文件（配置文件和依赖包分开） -->
                <excludes>
                    <exclude>static/**</exclude>
                    <exclude>templates/**</exclude>
                    <exclude>${profile.active}/**</exclude>
                    <exclude>*.yml</exclude>
                    <exclude>*.properties</exclude>
                    <exclude>*.xml</exclude>
                    <exclude>*.txt</exclude>
                    <exclude>*.json</exclude>
                    <exclude>*.conf</exclude>
                </excludes>
                <archive>
                    <manifest>
                        <addClasspath>true</addClasspath>
                        <classpathPrefix>./lib/</classpathPrefix>
                        <useUniqueVersions>false</useUniqueVersions>
                        <!-- 指定入口类 -->
                        <mainClass>com.sky.server.alert.AlertApplication</mainClass>
                    </manifest>
                    <manifestEntries>
                        <Class-Path>./resources/</Class-Path>
                    </manifestEntries>
                </archive>
                <outputDirectory>${project.build.directory}</outputDirectory>
            </configuration>
        </plugin>

        <!-- 该插件用于复制依赖包到指定的文件夹里 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-dependency-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-dependencies</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-dependencies</goal>
                    </goals>
                    <configuration>
                        <outputDirectory>${project.build.directory}/lib/</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <!-- 该插件用于复制配置文件到指定的文件夹里 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-resources-plugin</artifactId>
            <executions>
                <execution>
                    <id>copy-resources</id>
                    <phase>package</phase>
                    <goals>
                        <goal>copy-resources</goal>
                    </goals>
                    <configuration>
                        <resources>
                            <resource>
                                <directory>src/main/resources/${profile.active}/</directory>
                                <includes>
                                    <!--<include>${profile.active}/**.*</include>-->
                                    <include>**/**.*</include>
                                </includes>
                            </resource>
                        </resources>
                        <outputDirectory>${project.build.directory}/resources</outputDirectory>
                    </configuration>
                </execution>
            </executions>
        </plugin>
        <!-- Spring Boot打包插件，把maven-jar-plugin打成的jar包重新打成可运行jar包 -->
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <!-- 重写包含依赖，包含不存在的依赖，jar里没有pom里的依赖 -->
                <includes>
                    <include>
                        <groupId>null</groupId>
                        <artifactId>null</artifactId>
                    </include>
                </includes>
                <mainClass>com.sky.server.alert.AlertApplication</mainClass>
                <layout>ZIP</layout>
                <!-- 使用外部配置文件，jar包里没有资源文件 -->
                <addResources>true</addResources>
                <outputDirectory>${project.build.directory}/resources</outputDirectory>
            </configuration>
            <executions>
                <execution>
                    <goals>
                        <goal>repackage</goal>
                    </goals>
                    <configuration>
                        <!--配置jar包特殊标识 配置后，保留原文件，生成新文件 *-run.jar -->
                        <!--配置jar包特殊标识 不配置，原文件命名为 *.jar.original，生成新文件 *.jar -->
                        <!--<classifier>run</classifier> -->
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```



jar启动脚本

```
#!/bin/bash
BASE_PATH=$(cd `dirname $0`;pwd)
APP_NAME=`basename $2`
PROFILE="$2"
cd ${BASE_PATH}
echo ${APP_NAME}

JAVA_OPTS="-Xms256m -Xmx512m \
  -XX:+HeapDumpOnOutOfMemoryError \
  -XX:HeapDumpPath=logs/heapdump.hprof \
  -XX:+PrintGCDetails \
  -Xloggc:logs/gc.log"

#使用说明，用来提示输入参数
usage() {
  echo "Usage: sh 执行脚本.sh [start|stop|restart|status|log] jarfilename"
  exit 1
}

#检查程序是否在运行
is_exist(){
  PIDS=`jps -l | grep ${APP_NAME} |awk '{print $1}' `
  #如果不存在返回1，存在返回0
  if [ -z "${PIDS}" ]; then
   return 1
  else
   return 0
  fi
}

#启动方法
start(){
	is_exist
	echo "启动程序：${APP_NAME}"
  if [ ! -d $BASE_PATH"/logs" ]; then
      mkdir $BASE_PATH"/logs"
  fi
  	date=$(date -d "yesterday" +%Y-%m-%d-%H:%M:%S )
  if [ ! -f $BASE_PATH"/logs/catalina.out."$date ]; then
      echo $date
      mv $BASE_PATH"/logs/catalina.out" $BASE_PATH"/logs/catalina.out.${date}"
  fi
  num=`jps -l | grep ${APP_NAME} | wc -l`
  if [ $num -gt 0 ]; then
      echo "程序已启动.pid=${PIDS}"
  else
      echo ${APP_NAME} : ${PROFILE}
      nohup java ${JAVA_OPTS} -jar -Dspring.profiles.active=${PROFILE} "${APP_NAME}" >>$BASE_PATH"/logs/catalina.out" 2>&1 &
  fi

  count=1
  while [ $num -gt 0 ]; do
      is_exist
      if [ $? -eq 0 ]; then
        break
      fi

      echo "等待中... ... "$count
      sleep 1
      let "count++"

      if [ count -ge 5 ]; then
        echo "程序启动失败，请检查原因"
        break
      fi
  done
}

#停止方法
stop(){
	echo "停止程序：${APP_NAME}"
  #kill `jps -v | grep $BASE_PATH|awk '{print $2}'`
  echo `jps -l | grep ${APP_NAME}`
  num=`jps -l | grep ${APP_NAME} | wc -l`
  count=1
  while [ $num -gt 0 ]; do
      kill `jps -l | grep "${APP_NAME}" | awk '{print $1}'`
      echo "等待中... ... $count"
      sleep 1
      num=`jps -l | grep "${APP_NAME}" | wc -l`
      let "count++"

      if [ $count -gt 5 ]; then
        break
      fi
  done

  while [ $num -gt 0 ]; do
      kill -INT `jps -l | grep "${APP_NAME}" | awk '{print $1}'`
      echo "等待中... ... $count"
      sleep 1
      num=`jps -l | grep "${APP_NAME}" | wc -l`
      let "count++"

      if [ $count -gt 10 ]; then
        break
      fi
  done

  if [ $num -gt 0 ]; then
     kill -9 `jps -l | grep "${APP_NAME}" | awk '{print $1}'`
     echo "kill -9"
  fi

  echo "程序退出完成"
}

#输出运行状态
status(){
  is_exist
  if [ $? -eq 0 ]; then
    echo "${APP_NAME} is running. Pid is ${PIDS}"
  else
    echo "${APP_NAME} is NOT running."
  fi
}

#重启
restart(){
  stop
  start
}

#查看日志
log(){
  tail -f logs/catalina.out
}

#根据输入参数，选择执行对应方法，不输入则执行使用说明
case "$1" in
  "start")
    start
    ;;
  "stop")
    stop
    ;;
  "status")
    status
    ;;
  "restart")
    restart
    ;;
  "log")
    log
    ;;
  *)
    usage
    ;;
esac


```


赋值权限语句；

chmod +x ./*