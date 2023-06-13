# GORM设计原理与实践

## 1. GORM介绍

### 1.1 背景知识

设计简洁、功能强大、自由扩展的全功能ORM(持久层))框架

ORM全称是：Object Relational Mapping **(对象关系映射)**，其主要作用是在编程中，把面向对象的概念跟数据库中表的概念对应起来。举例来说就是，我定义一个对象，那就对应着一张表，这个对象的实例，就对应着表中的一条记录。

GORM是**Golang目前比较热门的数据库ORM操作库**，它文档齐全，对开发者也比较友好，使用非常方便简单，支持主流数据库。使用上主要就是把struct类型和数据库表记录进行映射，操作数据库的时候不需要直接手写Sql代码。

### 1.2 惯例约定

约定优于配置

*   表名为struct name的snake\_cases复数格式
*   字段名为field name的snake\_case单数格式
*   ID/ld字段为主键，如果为数字，则为自增主键
*   CreatedAt字段，创建时,保存当前时间
*   updatedAt字段，创建、更新时，保存当前时间
*   gorm.DeletedAt字段，默认开启soft delete模式

一切都可配置

*   <https://gorm.io/docs/conventions.html>

## 2. GORM基础使用

应用程序 -> GORM *操作接口* database/sql *连接、操作接口* 数据库

### 2.1 SQL生成

GORM STATEMENT

![GORM SQL.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/188d752e727c49d0bbe1a5f0794d2a12~tplv-k3u1fbpfcp-watermark.image?)

MySQL:
**DELETE FROM** XXX **WHERE** XXX **ORDER BY** XXX **desc LIMIT** 100

### 2.2 插件扩展

插件是怎么工作的？

```go
//注册新Callback
db.Callback().Create().Register("myplugin"，func(*gorm.DB){})
//删除Callback
db.Callback(.Create(.Remove("gorm:begin_transaction")
//替换Callback
db.Callback().Create().Replace(" gorm:before_create"， func(*gorm.DB){})
//查询注册的Callback
db . Callback. Create.Get(" gorm: create")
//指定Callback顺序
db.Callback().Create().Before(" gorm:create ").After("myplugin").Register("myplugin2"，func(*gorm.DB){})
//注册到所有服务之前
db.Callback().Create().Before("*").Register("my_plugin:new_callback"，func( *gorm.DB){})
//注册时检查条件
enableTransaction := func(db *gorm.DB) bool { return !db.SkipDefaultTransaction }
db.Callback().Create().Match(enableTransaction).Reaister( "caorm:beain_transaction". BeainTransaction)
```

多租户 多数据库、读写分离

### 2.3 ConnPool

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fa35d958fd2d409ab833119959faed6a~tplv-k3u1fbpfcp-watermark.image?)
ConnPool是什么？

查找缓存的预编译SQL  
未找到，将收到的SQL和Vars预编译  
使用缓存的预编译SQL执行  

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f18d142446b442caa0475f6d6fb37acf~tplv-k3u1fbpfcp-watermark.image?)

### 2.4 Dialector

定制SQL生成  
定制GORM插件  
定制ConnPool  
定制企业特性逻辑  
