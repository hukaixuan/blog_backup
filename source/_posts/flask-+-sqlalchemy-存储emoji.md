---
title: "flask+sqlalchemy 存储emoji"
date: 2017.04.10 17:44
tags:
 - Flask
 - MySQL
---

  最近在用flask写一个web应用，需要存储用户发表的内容，可能含有 emoji 等内容，但是在存储的时候总是报错 ` (_mysql_exceptions.OperationalError) (1366, "Incorrect string value: '\\xF0\\x9F\\x98\\xB2' for column 'content' at row 1")`，一直以为问题出在数据库端，可是数据库端全部更改为 `utf8mb4`还是报错。终于通过查资料发现问题是在 flask-sqlalchemy 默认的字符集设置是不支持 emoji 的 utf8，解决如下：
`SQLALCHEMY_DATABASE_URI = '数据库连接配置?charset=utf8mb4'`　　在数据库连接后指明编码为`utf8mb4`
ps: 我的配置是通过一个config类来实现的，参考自己的配置，改好数据编码后，在数据库连接URI后 添加 `?charset=utf8mb4` 即可，如：
`app.config['SQLALCHEMY_DATABASE_URI'] = '数据库连接配置?charset=utf8mb4'`
