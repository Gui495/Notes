## Hive 4.1.0 安装步骤（JDK 17 + Hadoop 3.3.6）

### 一、前置条件

确保以下组件已正常运行：

| 组件       | 版本    | 验证命令                  |
| :--------- | :------ | :------------------------ |
| **JDK**    | 17.0.12 | `java -version`           |
| **Hadoop** | 3.3.6   | `hdfs dfs -ls /`          |
| **MySQL**  | 8.0     | `systemctl status mysqld` |

### 二、下载与解压 Hive 4.1.0

```shell
# 下载 Hive 4.1.0
https://hive.apache.org/general/downloads/

# 解压到指定目录
tar -zxvf apache-hive-4.1.0-bin.tar.gz -C /opt/soft/
cd /opt/soft/
mv apache-hive-4.1.0-bin hive-4.1.0
```



### 三、配置环境变量

编辑 `~/.bashrc` 或 `/etc/profile`：

```shell
export HIVE_HOME=/opt/soft/hive-4.1.0
export PATH=$PATH:$HIVE_HOME/bin
```



执行 source /etc/profile 使生效。

### 四、创建 HDFS 数据仓库目录

bash

```shell
hdfs dfs -mkdir -p /hive/warehouse
hdfs dfs -chmod 777 /hive/warehouse
```



### 五、配置 hive-site.xml

bash

```shell
cd $HIVE_HOME/conf
cp hive-default.xml.template hive-site.xml
```



编辑 `hive-site.xml`，清空原内容后填入：

xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>

    <!-- HDFS 数据仓库目录 -->
    <property>
        <name>hive.metastore.warehouse.dir</name>
        <value>/hive/warehouse</value>
    </property>

    <!-- Metastore 服务地址 -->
    <property>
        <name>hive.metastore.uris</name>
        <value>thrift://localhost:9083</value>
    </property>

    <!-- MySQL 连接 URL（注意修改密码） -->
    <property>
        <name>javax.jdo.option.ConnectionURL</name>
        <value>jdbc:mysql://127.0.0.1:3306/hive?createDatabaseIfNotExist=true&amp;useSSL=false&amp;serverTimezone=Asia/Shanghai</value>
    </property>

    <!-- MySQL JDBC 驱动（MySQL 8.0 使用 com.mysql.cj.jdbc.Driver） -->
    <property>
        <name>javax.jdo.option.ConnectionDriverName</name>
        <value>com.mysql.cj.jdbc.Driver</value>
    </property>

    <!-- MySQL 用户名 -->
    <property>
        <name>javax.jdo.option.ConnectionUserName</name>
        <value>root</value>
    </property>

    <!-- MySQL 密码（替换为实际密码） -->
    <property>
        <name>javax.jdo.option.ConnectionPassword</name>
        <value>你的MySQL密码</value>
    </property>

    <!-- 显示当前数据库名称 -->
    <property>
        <name>hive.cli.print.current.db</name>
        <value>true</value>
    </property>

    <!-- 显示查询结果列名 -->
    <property>
        <name>hive.cli.print.header</name>
        <value>true</value>
    </property>
    <!-- hive默认使用hadoop mapreduce引擎 -->
	<property>
  		<name>hive.execution.engine</name>
  		<value>mr</value>
	</property>
       <property>
        <name>hive.server2.thrift.port</name>
        <value>10000</value>
  </property>
  <property>
        <name>hive.server2.thrift.bind.host</name>
        <value>0.0.0.0</value>   <!-- 允许所有地址访问 -->
  </property>
  <property>
    <name>hive.server2.authentication</name>
    <value>NONE</value>
  </property>
  <property>
        <name>hive.server2.thrift.client.user</name>
        <value>root</value>
  </property>
  <property>
        <name>hive.server2.thrift.client.password</name>
        <value>root</value>
   </property>
    <!--免去检查-->
   <property>
    	<name>hive.server2.active.passive.ha.enable</name>
    	<value>true</value>
   </property>
</configuration>
```



### 六、下载并放置 MySQL JDBC 驱动

bash

```shell
# 下载 MySQL 8.0 驱动
mysql-connector-java-8.0.33.jar

# 复制到 Hive lib 目录
cp mysql-connector-java-8.0.33.jar $HIVE_HOME/lib/
```



### 七、解决 Guava 版本冲突（Hive 4.1.0 + Hadoop 3.3.6）

Hive 4.1.0 自带的 Guava 版本可能与 Hadoop 3.3.6 不一致，建议统一：

bash

```shell
# 删除 Hive 中的旧版本 Guava
rm -f $HIVE_HOME/lib/guava-*.jar

# 复制 Hadoop 中的 Guava 到 Hive lib
cp $HADOOP_HOME/share/hadoop/common/lib/guava-*-jre.jar $HIVE_HOME/lib/
```



### 八、初始化 Metastore 元数据

bash

```shell
schematool -dbType mysql -initSchema
```



成功标志：日志显示 `"schemaTool completed"`。

如提示 schema 已存在，可先删除 hive 数据库再重新初始化：

sql

```sql
DROP DATABASE IF EXISTS hive;
CREATE DATABASE hive CHARACTER SET utf8mb4;
```



### 九、启动与验证

**启动顺序**：

1. Hadoop：`start-dfs.sh && start-yarn.sh`
2. MySQL：确保已启动
3. Metastore 服务：`hive --service metastore &`
4. 如果有第三方的链接需要启动  hive --service hiveserver2
5. Hive CLI：`hive`

**验证测试**：

sql

```sql
-- 查看数据库
SHOW DATABASES;

-- 创建测试表
CREATE TABLE test (id INT, name STRING) ROW FORMAT DELIMITED FIELDS TERMINATED BY '\t';

-- 插入测试数据
INSERT INTO test VALUES (1, 'hello'), (2, 'world');

-- 查询
SELECT * FROM test;
```



------

## 常见问题与解决

| 问题                                      | 解决方案                                                     |
| :---------------------------------------- | :----------------------------------------------------------- |
| **NoSuchMethodError (Guava)**             | 按第七步替换 Guava jar 包                                    |
| **Metastore schema version mismatch**     | 检查 `hive.metastore.schema.verification` 是否设为 false（Hive 4.x 默认已放宽校验） |
| **Access denied for user**                | 检查 MySQL 密码和权限，确保允许远程访问                      |
| **Hive 启动报错 NoClassDefFoundError**    | 检查 JDBC 驱动是否已放入 `$HIVE_HOME/lib/`                   |
| **Hive 4.1.0 与 Hadoop 3.3.6 兼容性问题** | 参考 HIVE-28683 方案，将高版本 Hadoop 依赖打包到 Hive 环境中 |

------

## 参考资源

- [Apache Hive 4.1.0 下载页面](https://hive.apache.org/downloads.html)
- [HIVE-28683: Hive4 与低版本 Hadoop 安装指南[reference:12\]](https://issues.apache.org/jira/browse/HIVE-28683)
- [Hive 4.0.1 Release Notes - 确认与 Hadoop 3.3.6 兼容[reference:13\]](https://issues.apache.org/jira/secure/ReleaseNote.jspa?version=12354786)