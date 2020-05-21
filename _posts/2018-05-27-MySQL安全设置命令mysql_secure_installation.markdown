---
layout:     post
title:      "MySQL安全设置命令mysql_secure_installation"
subtitle:   "这篇文章主要介绍如何使用安全设置命令mysql_secure_installation修改mysql密码及进行相关配置"
date:       2018-05-27
author:     "Malcolm Suen"
header-img: "img/post-bg-2015.jpg"
header-mask:  0.3
catalog: true
tags:
    - MySQL
    - 数据库
---

## 安装MySQL

通过`Homebrew`安装，执行`brew install mysql`即可。

安装完成后会显示`Caveats `警告，如下所示：

```
We've installed your MySQL database without a root password. To secure it run:
    mysql_secure_installation

MySQL is configured to only allow connections from localhost by default

To connect run:
    mysql -uroot

To have launchd start mysql now and restart at login:
  brew services start mysql
Or, if you don't want/need a background service you can just run:
  mysql.server start
```

我们可以根据提示进行简单的配置。

## 安全设置向导mysql_secure_installation

使用brew命令安装完mysql后，根据提示我们可以知道当前root是没有密码的，我们可以通过执行`mysql_secure_installation`命令来进行安全设置。

同时这种方法也同样可以用来解决使用`mysql -u root -p`登录时的`Access denied `问题。

首先执行命令`mysql_secure_installation`，会出现如下的错误：

```
Error: Can't connect to local MySQL server through socket '/tmp/mysql.sock' (2)
```

当前`/tmp/` 目录下并没有`mysql.sock`这个文件，它在mysql服务启动时才会创建，所以需要提前执行`mysql.server start `命令。提示如下：

```
Starting MySQL
. SUCCESS! 
```

这时候就可以正常执行`mysql_secure_installation`命令了。

#### 建立密码验证插件

```
Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD PLUGIN can be used to test passwords and improve security. It checks the strength of password and allows the users to set only those passwords which are secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: y 
```

#### 选择密码规则

```
There are three levels of password validation policy:

LOW    Length >= 8
#长度大于等于8
MEDIUM Length >= 8, numeric, mixed case, and special characters
#长度大于等于8，数字、大小写字母、特殊符号
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file
#长度大于等于8，数字、大小写字母、特殊符号和字典文件（慎选！）

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1
Please set the password for root here.

New password: （输入你的密码）
Re-enter new password: （再次输入你的密码）
```

#### 创建符合规则的新密码

```
Estimated strength of the password: 50 		#密码强度
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y
```

#### 删除匿名用户

```
By default, a MySQL installation has an anonymous user, allowing anyone to log into MySQL without having to have a user account created for them. This is intended only for testing, and to make the installation go a bit smoother.
You should remove them before moving into a production environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y
Success.
```

#### 禁止远程登录

```
Normally, root should only be allowed to connect from 'localhost'. This ensures that someone cannot guess at the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.
```

#### 删除测试数据表

```
By default, MySQL comes with a database named 'test' that anyone can access. This is also intended only for testing, and should be removed before moving into a production environment.

Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.
```

#### Done

```
Reloading the privilege tables will ensure that all changes made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
#是否重新加载权限表
Success.

All done! 
```

现在可以使用`mysql -u root -p`进行密码链接了。

## 可能会出现的问题

#### Your password does not satisfy the current policy requirements.

如果你在选择密码规则的时候不小心选择了2，也就是数字、大小写字母、特殊符号和字典文件的组合。这时候设置密码会出现如下提示：

```
Your password does not satisfy the current policy requirements.
```

这时候重新运行`mysql_secure_installation`也不会再给你机会重新设置了。手动微笑，mmp。

解决方案如下：

使用命令`mysql -uroot`登陆，执行：

```
set global validate_password_policy=0;  
#将密码规则设置为LOW，就可以使用纯数字纯字母密码
```

 LOW强度下密码位数的最低要求为8位，如果你想用诸如123456这类的密码，执行：

```
set global validate_password_length=4;  
#最低位数为4位
```

 这个时候重新运行`mysql_secure_installation`就可以安心设置了。

## 相关参数

```
validate_password_dictionary_file：插件用于验证密码强度的字典文件路径。

validate_password_length：密码最小长度。

validate_password_mixed_case_count：密码至少要包含的小写字母个数和大写字母个数。

validate_password_number_count：密码至少要包含的数字个数。

validate_password_policy：密码强度检查等级，0/LOW、1/MEDIUM、2/STRONG。

validate_password_special_char_count：密码至少要包含的特殊字符数。
```