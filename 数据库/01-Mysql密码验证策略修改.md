系统环境为：**ubuntu 22.04 wsl**，Mysql 版本为：**8.0.31**。

## Mysql 修改密码
在安装完毕 Mysql 后，一般需要执行 `sudo mysql_secure_installation` 来修改数据库的安全设置，它的作用如下：

1. 可以为root帐户设置密码。
2. 可以删除外部访问的root帐户。
3. 可以删除匿名帐户。
4. 可以删除测试数据库（默认情况下，所有用户，包括匿名用户都可以访问该数据库）以及相关权限。

在给 root 设置密码时，可能会失败，错误信息如下所示：
```
Error: SET PASSWORD has no significance for user 'root'@'localhost' as the authentication method used doesn't store authentication data in the MySQL server. Please consider using ALTER USER instead if you want to change authentication parameters.
```
即需要使用 `ALTER USER` 来设置账号密码，命令如下：
```
# 首次登录 mysql
sudo mysql

# 设置root账号密码
mysql> alter user 'root'@'localhost' identified by 'youpassword';

# 刷新权限
mysql> flush privileges;
```

上面的 mysql 语句可能会执行失败，因为设置的密码可能不满足密码验证策略，报错如下：
```
ERROR 1819 (HY000): Your password does not satisfy the current policy requirements
```
即**您的密码不符合当前策略要求**。


## 修改验证策略
Mysql 的密码验证策略分为 3 级，分别如下：
```
There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG
```
默认使用的 `MEDIUM`，也就是说密码长度要大于等于 8，且要有数字、大小写混合、特殊字符。

可以使用 mysql 语句查询当前 mysql-server 所使用的密码验证策略，语句如下：
```
mysql> SHOW VARIABLES LIKE 'validate_password%';
+--------------------------------------+--------+
| Variable_name                        | Value  |
+--------------------------------------+--------+
| validate_password.check_user_name    | ON     |
| validate_password.dictionary_file    |        |
| validate_password.length             | 8      |
| validate_password.mixed_case_count   | 1      |
| validate_password.number_count       | 1      |
| validate_password.policy             | MEDIUM |
| validate_password.special_char_count | 1      |
+--------------------------------------+--------+
```

下面列出以上有关密码策略的参数意义：
- dictionary_file，指定密码验证的文件路径
- length，整个密码的总长度
- mixed_case_count，整个密码中**至少**要包含大/小写字母的总个数
- number_count，整个密码中**至少**要包含阿拉伯数字的个数
- policy，密码策略，默认为 **MEDIUM**
- special_char_count，整个密码中**至少**要包含特殊字符的个数

一般在开发环境会使用很简单的密码，例如 123456，所以需要修改 `policy` 和 `length`，需要注意的是，Mysql 5.7 和 Mysql 8 使用的修改语句有所不同。

- Mysql 5.7
```
mysql> set global validate_password_policy=0;
mysql> set global validate_password_length=1;
```

- Mysql 8
```
mysql> set global validate_password.policy=0;
mysql> set global validate_password.length=1;
```