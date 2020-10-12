# 1 基本操作

## 1.1 新增索引

- 使用put添加索引

~~~
PUT /lib/          //索引名称
{
  "settings": {
    "index":{
      "number_of_shards":3,    //分片，一旦确认，不能修改
      "number_of_replicas":0   //备份数量
    }
  }
}
~~~

~~~
PUT lib2
~~~

- 查看索引配置

~~~
GET /lib/_settings  //查看索引lib的配置
GET _all/_settings  //查看所有索引的配置
~~~

## 1.2 在索引下添加文档

- PUT方式，需要指定ID

~~~
PUT /lib/user/1
{
  "firstName":"Jane",
  "lastName":"Smith",
  "age":20
}
~~~

- POST方式，不用指定ID，由ElasticSearch自动生成id

~~~
POST /lib/user
{
  "firstName":"Jane",
  "lastName":"Smith",
  "age":20
}
~~~

- 查询文档

~~~
GET /lib/user/1

GET /lib/user/_search

GET /lib/user/1?_source=firstName,lastName  // 查询指定字段
~~~

## 1.3 更新文档

- 全文档覆盖

~~~
PUT /lib/user/1
{
  "firstName":"Jane",
  "lastName":"Smith",
  "age":30
}
~~~

- 部分修改

~~~
POST /lib/user/1/_update
{
  "doc": {
    "age":33
  }
}
~~~

## 1.4 删除索引

~~~
DELETE /lib/user/1
~~~