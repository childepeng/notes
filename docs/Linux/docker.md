# Docker

## 网络

1. 查看网络列表

   > docker network ls

2. 创建网络

   > docker network create --driver 

## 镜像



## 容器

1. 创建容器 

   > docker run -itd [--name  name]  [--hostname hostname] [--net  network]  [--ip IP]  image  [shell]
   >
   > eg. 
   >
   > ​	docker run -itd  --name hdp01 --hostname hdp01 --net innernet --ip 172.18.0.11 ubuntu  /bin/bash

2. 查看容器

   > 查看容器列表
   >
   > docker container ls  -a
   >
   > 查看指定容器详情
   >
   > docker inspect  containerid/name

3. 启停容器

   > docker  start/stop/rm  containerid/name

