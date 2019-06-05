# 1 Docker简介

# 2 Docker安装

## Docker的基本组成

## 安装步骤

## 永远的HelloWorld

### 1. 阿里云镜像加速

- 是什么：https://dev.aliyun.com/search.html
- 注册阿里云账号
- 获得加速器地址: https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors
- 重启Docker后台服务：service docker restart
- 检查加速器是否生效

### 2. 网易云加速

### 3. 启动Docker后台容器(测试运行hello-world)

## 底层原理

# 3 Docker常用命令

## 3.1 镜像命令

- docker pull centos

## 3.2 容器命令

- 新建并启动容器

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

  - 

- 列出当前所有正在



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

## 7.2 安装Redis

# 8 本地镜像发布到阿里云

