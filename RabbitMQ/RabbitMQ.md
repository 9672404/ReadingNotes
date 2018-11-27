# RabbitMQ

## 1. 使用Docker安装

```shell
#下载镜像
docker exec -it myrabbitmq /bin/bash  #启动; myrabbitmq 为docker 里 mq的名称
/usr/lib/rabbitmq/bin/rabbitmqctl     #docker 客户端的管理工具所在路径
./rabbitmqctl list_users              # 查看所有用户
./rabbitmqctl status                  # 查看状态

rabbitmqctl add_user mango 123456     #添加一个用户
rabbitmqctl set_user_tags mango admin #给用户赋予角色（damin）
```

需要关闭防火墙和设置阿里云白名单才能访问web控制台。

> 控制台地址：http://ip:15673/#/

