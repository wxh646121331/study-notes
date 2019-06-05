# 1 在Docker中搭建Zookeeper集群

- 下载Zookeeper镜像

  > docker pull zookeeper

- 启动zookeeper镜像

  > docker run —name zookeeper01 -d zookeeper

- 启动日志

  > docker logs -f zookeeper01

- 进入容器

  > docker exec -it zookeeper01 /bin/bash

- 访问容器

  > docker run -it --rm --link zookeeper01:zookeeper zookeeper zkCli.sh -server zookeeper

  - 含义：
    - 启动一个zookeeper镜像，并运行这个镜像内的zkCli.sh命令，命令的参数是"-server zookeeper"；
    - 将我们先前启动的名为zookeeper01的容器链接(link)到我们新建的这个容器上，并其主机名命名为zookeeper

- 上传文件

  > docker cp test.txt zookeeper1:/test.txt

- 查看zookeeper的状态

  > /bin/zkServer.sh status

- 