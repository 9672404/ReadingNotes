# Docker 常用命令

```shell

## mysql
docker search mysql  #从仓库查看 mysql所有镜像
docker images #查看本地镜像
docker ps [-a]  #查看正在运行[已停止]的镜像
docker rmi -f mysql:5.7.24 # 删除指定版本的镜像文件
docker stop 镜像id

docker run -p 3306:3306 --name mysqlcloud -e MYSQL_ROOT_PASSWORD=root -d  mysql:5.7.24
#运行一个Mysql镜像

## rabbitMQ
docker exec -it myrabbitmq /bin/bash  #启动; myrabbitmq 为docker 里 mq的名称
/usr/lib/rabbitmq/bin/rabbitmqctl     #docker 客户端的管理工具所在路径
./rabbitmqctl list_users              # 查看所有用户
./rabbitmqctl status                  # 查看状态


```

