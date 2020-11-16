### 常用linux指令



| 指令          | 作用           |
| ------------- | -------------- |
| netstat -ntlp | 查看在用端口号 |
| chmod +x ./*  | 赋予权限       |
| du -sh *            | 查看文件大小               |





Docker

| 指令                                                         | 说明                 |
| ------------------------------------------------------------ | -------------------- |
| docker  build -t  文件名：文件标记                           | 创建镜像             |
| docker images                                                | 查看镜像             |
| docker rmi REPOSITORY:TAG                                    | 删除镜像             |
| docker tag IMAGEID REPOSITORY:TAG                            | 重命名镜像           |
| docker rm CONTAINERID                                        | 删除容器             |
| docker ps                                                    | 查看所有正在运行容器 |
| docker stop containerId                                      | 停止容器             |
| docker ps -a                                                 | 查看所有容器         |
| docker ps -a -q                                              | 查看所有容器ID       |
| docker stop                                                  | 停止所有容器         |
| docker rmi                                                   | 删除所有容器         |
| docker image inspect --format='{{.RepoTags}} {{.Id}} {{.Parent}}'  $(docker image ls -q --filter since=dc98e8e62cd1) | 分析镜像依赖         |
| docker logs --tail  300 -f  containId                        | 查看容器日志         |
| docker ``rm` `$(docker ``ps` `-a -q)                         | 删除所有容器         |
| docker stop $(docker ``ps` `-a -q)                           | 停止所有容器         |
| sudo docker exec -it containId  /bin/bash                    | 进入容器             |
|                                                              |                      |
|                                                              |                      |
|                                                              |                      |













