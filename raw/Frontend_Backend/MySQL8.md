# MySQL8 安装步骤

##### 1 卸载mariadb

```sh
rpm -qa | grep mariadb
rpm -e --nodeps 文件名
```

##### 2. 安装前置软件

```shell
#根据情况
yum -y install libaio numactl
```

##### 3. 解压文件

```shell
cd /usr/local/
tar -xvf mysql-8.0.30-el7-x86_64.tar
tar -zxvf mysql-8.0.30-el7-x86_64.tar.gz
mv mysql-8.0.30-el7-x86_64/ mysql
mkdir -p mysql/data
```

##### 4.授权

```shell
chown -R root:root /usr/local/mysql/
chmod -R 755 /usr/local/mysql/
```

##### 5.初始化

```shell
cd /usr/local/mysql/bin/
./mysqld --initialize --user=root --datadir=/usr/local/mysql/data --basedir=/usr/local/mysql
#此处要复制你的随机密码
```

##### 6.增加和修改配置文件

```shell
#修改/usr/local/mysql/support-files/mysql.server搜索$bindir/mysqld_safe 在命令上添加用户 其他不动
$bindir/mysqld_safe --user=root 
#增加vim /etc/my.cnf
[client]
port = 3306
socket = /usr/local/mysql/data/mysql.sock
default-character-set = utf8mb4

[mysql]  
default-character-set = utf8mb4

[mysqld]  
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_general_ci
init_connect = 'SET NAMES utf8mb4'

port = 3306
socket = /usr/local/mysql/data/mysql.sock
skip-external-locking
datadir = /usr/local/mysql/data


[mysqldump]
quick
max_allowed_packet = 16M

[mysql]
no-auto-rehash

[myisamchk]
key_buffer_size = 20M
sort_buffer_size = 20M
read_buffer = 2M
write_buffer = 2M

[mysqlhotcopy]
interactive-timeout
```

##### 7.启动mysql 并设置软链接

```shell
/usr/local/mysql/support-files/mysql.server start
ln -s /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql
ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql
ln -s /usr/local/mysql/mysql.sock /var/mysql.sock
service mysql restart
```

##### 8.登录修改mysql密码开放远程

```sql
mysql -uroot -p
输入你的密码
alter user 'root'@'localhost' identified by 'ok';
use mysql;
update user set user.Host='%' where user.User='root';
flush privileges;
```

##### 9.设置开机启动

```
cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysqld
chmod +x /etc/init.d/mysqld
chkconfig --add mysqld
reboot
```

