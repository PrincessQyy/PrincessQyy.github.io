---
layout:     post
title:      "【MySQL】查看表"
date:       2017-01-17 11:39:00
author:     "Yuki"

---



好久之前学的数据库，一直没用差点不会操作了:)Σ( ° △ °)︴
看来一些简单的指令也还是要随时记录下来才方便查看呀~

## 如何查看已有的表 ##
打开cmd窗口，输入命令

- net start mysql  

启动MySQL服务。




输入命令

- mysql -u root -p

按下回车，按照提示输入密码，密码显示为******链接你自己的数据库，当然，你也可以直接在-p后面输入你的密码，不过此时是明文密码。

输入命令

- show databases

可以查看有哪些数据库

用命令

- use databaseName

来进入某个数据库，该操作会返回所有数据库的名称

用命令

- show tables

查看当前数据库有哪些表，该操作会返回所有表的名称，然后就可以用 

- desc tableName
 
来查看表结构了。

还有一个方法，不进入数据库，用命令
- show tables from databaseName

也可以返回指定数据库下所有表的名称。








