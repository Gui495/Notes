## 1. 清除原有 MariaDB
```bash
rpm -qa | grep mariadb
rpm -e --nodeps 上步查询到的文件
```

## 2. 下载安装
### 2.1 MySQL 5.6
```bash
wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm
rpm -ivh mysql-community-release-el7-5.noarch.rpm
yum install mysql-server -y
```

### 2.2 MySQL 5.7
```bash
wget -i -c http://dev.mysql.com/get/mysql57-community-release-el7-10.noarch.rpm
yum -y install mysql57-community-release-el7-10.noarch.rpm
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2022
yum install mysql-server -y
```

## 3. 文件授权
```bash
chown -R root:root /var/lib/mysql
chown root /var/lib/mysql/
```

## 4. MySQL 中文乱码处理
编辑 `/etc/my.cnf`：
```ini
[mysql]
character-set-server=utf8

[client]
default-character-set=utf8

[mysql]
default-character-set=utf8
```
重启：
```bash
service mysql restart
```

## 5. 修改 MySQL 登录密码、开放远程登录
### 5.1 MySQL 5.6
直接登录没有密码：
```sql
use mysql;
update user set password=password('ok') where user='root';
```

### 5.2 MySQL 5.7
查看临时密码：
```bash
grep "password" /var/log/mysql.log
```
修改密码：
```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '你的密码';
```
- 密码必须 8 位以上，大小写字母和数字。

### 5.3 修改远程登录
```sql
GRANT ALL PRIVILEGES ON *.* TO root@"%" IDENTIFIED BY "你的密码";
flush privileges;
exit;
```

## 6. 小结
- 先清除 MariaDB。
- 按版本安装 MySQL 5.6/5.7。
- 文件授权。
- 配置字符集解决中文乱码。
- 修改 root 密码和远程登录权限。
- 注意密码复杂度和安全。