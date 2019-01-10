---
title: MongoDB学习笔记（一）
date: 2017-07-12 15:23:21
tags: Mongo
categories: 数据库
summary:  "mongoDB是介于关系型和非关系型之间的，基于分布式文件存储的数据库，它数据格式是类似于json的bson，能够存储比较负责的数据类型。"
---
### 对mongoDB的了解
+ mongoDB是介于关系型和非关系型之间的，基于分布式文件存储的数据库，它数据格式是类似于json的bson，能够存储比较负责的数据类型。
<!-- more -->
1. mongoDB安装和卸载
+ 安装mongo
```
sudo apt-get install mongodb        
```
+ 判断mongo是否安装成功
```
mongo -version
```
+ 启动和关闭mongo
```
service mongodb start
service mongodb stop
```
+ 判断mongo是否启动成功，如果输出端口号，即启动成功
```
pgrep mongo -l
```
+ 卸载mongo(卸载还没试过)
```
sudo apt-get --purge remove mongodb mongodb-clients mongodb-server
```
+ 数据库相关
```
show dbs:显示数据库列表
show collections：显示当前数据库中的集合（类似关系数据库中的表table）
show users：显示所有用户
use yourDB：切换当前数据库至yourDB
db.help() ：显示数据库操作命令
db.yourCollection.help() ：显示集合操作命令，yourCollection是集合名 
```
2. mongoDB常用操作

+ 插入数据，mongo存入数据有两种方式
```
db.person.insert({_id:1, name: 'zhangsan', age: 21})
db.person.insert({_id:1,name:'nanxuan',age:21)
```
insert和save的区别之处在于如果，_id已经存在，insert不做操作，save做更新操作；如果不加_id字段，两者作用相同都是插入数据。
参考链接：[http://blog.csdn.net/flyfish111222/article/details/51886787](http://blog.csdn.net/flyfish111222/article/details/51886787)

+ 查询数据
db.student.find([查询条件],[显示的部分数据])
注意第一个参数（查询条件）不可以省略，如果查询条件为空可以用{}填充。
```
db.person.find() #查询所有记录
db.person.find({name:'nanxuan'}) #相当于select sname from person where name = 'nanxuan' 
```
+ 修改数据
db.student.update([查询条件],[更新操作符],[若不存在update记录，是否继续插入，默认是false],[是否更新找到的全部记录，默认是不更新即为false])
```
 db.person.update({_id:1},{$set:{name:'zhiqiang'}},false,true) # 相当于update person set name='zhiqiang' where id=1
```
+ 删除数据
db.student.remove([查询条件])
```
db.person.remove({_id:1}) # delete from person where _id=1
```
