# 1. MySQL的架构介绍

## 1.1 MySQL简介

- mysql内核
- sql优化工程师
- mysql服务器的优化
- 各种参数常量设定
- 查询语句优化
- 主从复制
- 软硬件升级
- 容灾备份
- sql编程

## 1.2 MySql安装

- 下载地址：

- 查看安装时创建的mysql用户和mysql组

  > cat /etc/passwd | grep mysql
  >
  > cat /etc/group | grep mysql
  >
  > mysqladmin --version

- 启、停mysql服务端

  > service mysql start
  >
  > service mysql stop
  >
  > service mysql restart


## 1.3 MySql配置文件

> cp /usr/share/mysql/my-default.cnf /etc/my.cnf

## 1.4 MySQL逻辑架构介绍

## 1.5 MySQL存储引擎

# 2. 索引优化分析

## 2.1 性能下降SQL慢

## 2.2 常见通用的join查询

## 2.3 索引简介

## 2.4 性能分析

## 2.5 索引优化

# 3. 查询截取分析

## 3.1 查询优化

## 3.2 慢查询日志

## 3.3 批量数据脚本

## 3.4 Show Profile

## 3.5 全局查询日志

# 4. MySQL锁机制

## 4.1 表锁

## 4.2 行锁

## 4.3 页锁

# 5. 主从复制

## 5.1 复制的基本原则

## 5.2 复制的最大问题

## 5.3 一主一从常见配置

