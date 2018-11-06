# Docker 常用命令

```shell
docker search mysql  #从仓库查看 mysql所有镜像
docker images #查看本地镜像
docker ps [-a]  #查看正在运行[已停止]的镜像
docker rmi -f mysql:5.7.24 # 删除指定版本的镜像文件
docker stop 镜像id

docker run -p 3306:3306 --name mysqlcloud -e MYSQL_ROOT_PASSWORD=root -d  mysql:5.7.24
#运行一个Mysql镜像


```

