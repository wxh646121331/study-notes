# 1 Docker简介

# 2 Docker安装

## 2.1 Docker的基本组成

## 2.2 安装步骤

## 2.3永远的HelloWorld

## 2.4阿里云镜像加速

- 是什么：https://dev.aliyun.com/search.html
- 注册阿里云账号
- 获得加速器地址: https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors 链接中有相关配置说明
- 重启Docker后台服务：service docker restart
- 检查加速器是否生效

## 2.5 启动Docker后台容器(测试运行hello-world)

## 2.6 底层原理



# 3 Docker常用命令

## 3.1 帮助命令

- docker -version
- docker info
- docker --help

## 3.1 镜像命令

###3.1.1 查看本地库所有的镜像

- docker images [options]

![](./images/docker/docker images.png)

- 字段说明

  | 字段       | 说明         |
  | ---------- | ------------ |
  | REPOSITORY | 镜像的仓库源 |
  | TAG        | 镜像的标签   |
  | IMAGE ID   | 镜像ID       |
  | CREATED    | 镜像创建时间 |
  | SIZE       | 镜像大小     |

- REPOSITORY:TAG 定义不同版本的镜像

- OPTIONS说明

  | OPTION      | 说明                             |
  | ----------- | -------------------------------- |
  | -a          | 列出本地所有的镜像(含中间镜像层) |
  | -q          | 只显示镜像ID                     |
  | -\-digests  | 显示镜像的摘要信息               |
  | -\-no-trunc | 显示完整的镜像信息               |

###3.1.2 查找镜像

- docker search [OPTIONS] 镜像名

- 网站：https://hub.docker.com

- OPTIONS说明

  | OPTION       | 说明                            |
  | ------------ | ------------------------------- |
  | --no-trunc   | 显示完整的镜像描述              |
  | -s           | 列出收藏数不小于指定值的镜像    |
  | -\-automated | 只列出automated build类型的镜像 |

###3.1.3 拉取镜像

- docker pull 镜像名:tag

- 默认为latest版本

###3.1.4 删除镜像

- 删除单个：docker rmi -f imageID
- 删除多个：docker rmi -f 镜像名1:TAG 镜像名2:TAG
- 删除全部：docker rmi -f $(docker images -qa)

## 3.2 容器命令

### 3.2.1 新建并启动容器

- docker run [option] image [command] [arg...]

- option说明

  | 选项              | 说明                                                         |
  | ----------------- | ------------------------------------------------------------ |
  | --name="容器名称" | 为容器指定一个名称                                           |
  | -d                | 后台运行容器，并返回容器ID，即启动守护式容器                 |
  | -i                | 以交互模式运行容器，通常与-t同时使用                         |
  | -t                | 为容器重新分配一个伪输入终端，通常与-i同时使用               |
  | -P                | 随机端口映射                                                 |
  | -p                | 指定端口映射，4种格式：ip:hostPort:containerPort, ip::containerPort, hostPort:contianerPort, containerPort |


### 3.2.2 列出当前所有正在运行的所有容器

- docker ps [options]

- 选项说明

  | 选项         | 说明                                      |
  | ------------ | ----------------------------------------- |
  | -a           | 列出当前所有正在运行的容器+历史上运行过的 |
  | -l           | 上次运行的容器                            |
  | -n           | 前n次运行的容器                           |
  | -q           | 静默显示，只显示容器id                    |
  | \-\-no-trunc | 不截断输出                                |

###3.2.3 退出容器

- exit —— 容器停止退出
- ctrl+P+Q——容器不停止退出

###3.2.4 启动容器

- docker start 容器id/容器名

###3.2.5 重启容器

- docker restart 容器id/容器名

###3.2.6 停止容器

- docker stop 容器id/容器名

### 3.2.7 强制停止容器

- docker kill 容器id/容器名

###3.2.8 删除已停止的容器

- docker rm 容器id
- 删除多个容器
  - docker rm -f $(docker ps -a -q)
  - docker ps -a -q | xargs docker rm

###3.2.9 重要

- 后台运行

  - docker run -d 容器id
  - 注意：docker 容器后台运行，就必须有一个前台进行。容器运行的命令如果不是那些一直挂起的命令(如：top, tail)，就是会自动退出的。最佳的解决方案是程序以前台进行的形式运行
  - 例：docker run -d centos /bin/sh -c "while true;do echo hello zzyy;sleep 2;done"

- 查看容器日志

  - docker logs -f -t -\-taill 容器id

  - 选项说明

    | 选项     | 说明               |
    | -------- | ------------------ |
    | -t       | 加入时间戳         |
    | -f       | 跟随最新的日志打印 |
    | \-\-tail | 数字显示最后多少条 |

- 查看容器内运行的进程

  - docker top 容器ID

- 查看容器内部细节

  - docker inspect 容器ID

- 进入正在运行的容器并以命令行交互

  - docker attach 容器ID
    - 直接进入容器启动命令的终端，不会启动新的进程
  - docker exec -t 容器id bin/bash
    - 在是容器中打开新的终端，并且可以启动新的进程

- 从容器内拷贝文件到主机上

  - docker cp 容器ID:容器内路径 目的主机路径

# 4 Docker镜像

# 5 Docker容器数据卷

# 6 DockerFile解析

# 7 Docker常用安装

## 7.1 总体步骤

- 搜索镜像
- 拉取镜像
- 查看镜像
- 启动镜像
- 停止容器
- 移除容器

## 7.1 安装MySql

- docker search mysql

- docker pull mysql:5.6

- docker images

- docker run -d -p 12345:3306 --name mysql -v E:/docker/mysql/conf:/etc/mysql/conf.d -v E:/docker/mysql/logs:/logs -v E:/docker/mysql/data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6

  - 命令说明

    >-p 12345:3306   将主机的12345端口映射到docker容器的3306端口
    >
    >--name myssql  运行服务名字
    >
    >-v E:/docker/mysql/conf:/etc/mysql/conf.d 将主机E:/docker/mysql/conf目录下的my.cnf挂载到容器的/etc/mysql/conf.d
    >
    >-v E:/docker/mysql/logs:/logs 将主机E:/docker/mysql/logs目录挂载到容器的/logs
    >
    >-v E:/docker/mysql/data:/var/lib/mysql  将主机E:/docker/mysql/data挂载到容器的/var/lib/mysql
    >
    >-e MYSQL_ROOT_PASSWORD=123456   初始化root用户的密码
    >
    >-d mysql:5.6  后台程序运行mysql5.6

- docker exec -it 容器id /bin/bash

- mysql -uroot -p

- show databases;

- create database db01;

- use db01;

- create table t_book(id int not null primary key, book_name varchar(20));

- show tables;

- 备份数据库：docker exec 容器ID sh -c ' exec mysqldump --all-databases -uroot -p"123456" ' > E:/docker/mysql/all-databases.sql

### 7.2.1 主从复制搭建

- 在docker启动两个mysql容器

  ~~~
  //创建第一台mysql
  docker run --name mysql_1 -v ~/docker/mysql/mysql_1/custom:/etc/mysql/conf.d -v ~/docker/mysql/mysql_1/data:/var/lib/mysql -p 3309:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
  
  //创建第二台mysql
  docker run --name mysql_2 -v ~/docker/mysql/mysql_2/custom:/etc/mysql/conf.d -v ~/docker/mysql/mysql_2/data:/var/lib/mysql -p 3310:3306 -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0
  ~~~

- 查看mysql的ip地址

  ~~~
  admindeMacBook-Pro:~ wuxinhong$ docker inspect --format='{{.NetworkSettings.IPAddress}}' mysql_1
  172.17.0.3
  admindeMacBook-Pro:~ wuxinhong$ docker inspect --format='{{.NetworkSettings.IPAddress}}' mysql_2
  172.17.0.4
  ~~~

- 在容器中安装vim

  ~~~
  apt-get update
  apt-get install vim
  ~~~

  

- 配置主服务器，修改/etc/mysql/my.cnf

  ~~~
  [mysqld]
  ## 设置server_id，一般设置为IP，同一局域网内注意要唯一
  server_id=100
  ## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
  binlog-ignore-db=mysql
  ## 开启二进制日志功能，可以随便取，最好有含义（关键就是这里了）
  log-bin=edu-mysql-bin
  ## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
  binlog_cache_size=1M
  ## 主从复制的格式（mixed,statement,row，默认格式是statement）
  binlog_format=mixed
  ## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
  expire_logs_days=7
  ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
  ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
  slave_skip_errors=1062
  ~~~

- 重启容器

  ~~~
  docker restart mysql_1
  ~~~

- 创建数据同步用户

  ~~~
  mysql -uroot -p123456
  CREATE USER 'slave'@'%' IDENTIFIED BY '123456';
  GRANT REPLICATION SLAVE, REPLICATION CLIENT ON *.* TO 'slave'@'%';
  ~~~

- 配置从服务器

  ~~~
  [mysqld]
  ## 设置server_id，一般设置为IP,注意要唯一
  server_id=101  
  ## 复制过滤：也就是指定哪个数据库不用同步（mysql库一般不同步）
  binlog-ignore-db=mysql  
  ## 开启二进制日志功能，以备Slave作为其它Slave的Master时使用
  log-bin=edu-mysql-slave1-bin  
  ## 为每个session 分配的内存，在事务过程中用来存储二进制日志的缓存
  binlog_cache_size=1M  
  ## 主从复制的格式（mixed,statement,row，默认格式是statement）
  binlog_format=mixed  
  ## 二进制日志自动删除/过期的天数。默认值为0，表示不自动删除。
  expire_logs_days=7  
  ## 跳过主从复制中遇到的所有错误或指定类型的错误，避免slave端复制中断。
  ## 如：1062错误是指一些主键重复，1032错误是因为主从数据库数据不一致
  slave_skip_errors=1062  
  ## relay_log配置中继日志
  relay_log=edu-mysql-relay-bin  
  ## log_slave_updates表示slave将复制事件写进自己的二进制日志
  log_slave_updates=1  
  ## 防止改变数据(除了特殊的线程)
  read_only=1
  ~~~

- 重启从服务器

  ~~~
  docker restart mysql_2
  ~~~

- 进入master

  ~~~
  mysql> show master status;
  +----------------------+----------+--------------+------------------+-------------------+
  | File                 | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
  +----------------------+----------+--------------+------------------+-------------------+
  | edu-mysql-bin.000004 |      433 |              | mysql            |                   |
  +----------------------+----------+--------------+------------------+-------------------+
  1 row in set (0.00 sec)
  ~~~

- 进入slave

  ~~~
  change master to master_host='172.17.0.2', master_user='slave', master_password='123456', master_port=3306, master_log_file='edu-mysql-bin.000001', master_log_pos=929, master_connect_retry=30;  
  ~~~

  命令解释：

  ~~~
  master_host: Master 的IP地址
  master_user: 在 Master 中授权的用于数据同步的用户
  master_password: 同步数据的用户的密码
  master_port: Master 的数据库的端口号
  master_log_file: 指定 Slave 从哪个日志文件开始复制数据，即上文中提到的 File 字段的值
  master_log_pos: 从哪个 Position 开始读，即上文中提到的 Position 字段的值
  master_connect_retry: 当重新建立主从连接时，如果连接失败，重试的时间间隔，单位是秒，默认是60秒。
  ~~~

  查看主从同步状态

  ~~~
  show slave status \G
  ~~~

  开启主从同步

  ~~~
  start slave;
  ~~~

  

## 7.2 安装Redis

# 8 本地镜像发布到阿里云

