---
title: Egg.js错误日志
date: 2019-04-07 11:15:21
tags:
categories:
---

# 一分钟总结
*  连接数据库失败，主要都是权限问题

>
```mysql
$ 先修改密码强度设置
$ GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
$ FLUSH PRIVILEGES;
```

* 修改默认端口

>
```js
config.cluster = {
    listen: {
      port: 80
    }
  };
```

* 使用nunjucks直接输出html而不是字符串

> 
```
{{ xxx | safe }}
```

* config中的安全码

> 先在Linux中egg-init一个项目，复制其中的`security keys`,再粘贴到自己的项目中

<!--more-->
# Mysql权限设置
无脑教程

```mysql
mysql -u root -p

> set global validate_password_policy=0; 
> set global validate_password_mixed_case_count=0;  
> set global validate_password_number_count=3; 
> set global validate_password_special_char_count=0; 
> set global validate_password_length=3; 
> GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root' WITH GRANT OPTION;
> FLUSH PRIVILEGES;
```

直接`grant`，如果你的密码太简单，会报`ERROR 1819 (HY000): Your password does not satisfy the current policy requiremen`的错误，因为你设置的密码不符合mysql的密码验证规则所致。

```
> select @@validate_password_policy;    查看mysql密码验证级别（0/LOW/1/MEDIUM/2/STRONG）；
> SHOW VARIABLES LIKE 'validate_password%'; 查看具体的设置项；
```
> validate_password_dictionary_file     用于验证密码强度的字典路径；
validate_password_length                密码的最少长度；
validate_password_mixed_case_count      密码的字母大小写个数；
validate_password_number_count          密码的数字个数；
validate_password_policy                密码的级别；
validate_password_special_char_count    密码的特殊字符个数；

# 阿里云http服务访问不了
> 需要开放端口
> [开放阿里云服务器端口](https://blog.csdn.net/niexia_/article/details/80362498)





