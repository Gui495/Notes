# Apache Zeppelin 安装与配置指南（Hadoop 3.3.6 + Hive 4.1.0 + JDK 17）

------

## 一、Zeppelin 简介

**Apache Zeppelin** 是一个基于 Web 的交互式数据分析和协作笔记本，支持 SQL、Scala、Python、R 等多种语言。它通过 **解释器（Interpreter）** 机制与后端引擎交互，可将 SQL 查询结果直接可视化，非常适合数据探索和团队协作。

### 版本选择说明

| 组件         | 版本       | 说明                                              |
| :----------- | :--------- | :------------------------------------------------ |
| **Zeppelin** | **0.11.1** | 最新稳定版（2024年3月发布），对 JDK 17 有较好支持 |
| **JDK**      | 17.0.12    | ✅ 已验证兼容                                      |
| **Hadoop**   | 3.3.6      | ✅ 已验证兼容                                      |
| **Hive**     | 4.1.0      | ✅ 已验证兼容                                      |

> **⚠️ 重要提示**：Zeppelin **0.10.0 及以上版本**已不再提供独立的 Hive 解释器，而是将 Hive 功能合并到 **JDBC 解释器**中。本文档将使用 JDBC 解释器连接 Hive。

------

## 二、前置条件

在安装 Zeppelin 之前，请确保以下组件已正常运行：

| 组件                       | 状态检查命令                 |
| :------------------------- | :--------------------------- |
| **JDK 17**                 | `java -version`              |
| **Hadoop 3.3.6**           | `hdfs dfs -ls /`             |
| **Hive 4.1.0 + Metastore** | `netstat -tlnp | grep 9083`  |
| **HiveServer2**            | `netstat -tlnp | grep 10000` |

> **⚠️ 必须确认 HiveServer2 已启动**，Zeppelin 通过 JDBC 连接 HiveServer2 执行查询。启动命令参考：
>
> bash
>
> ```shell
> nohup hive --service metastore >> logs/metastore.log 2>&1 &
> nohup hive --service hiveserver2 >> logs/hiveserver2.log 2>&1 &
> ```

------

## 三、下载 Zeppelin

### 3.1 下载安装包

Zeppelin 提供两种安装包：

| 安装包              | 大小    | 说明                                              |
| :------------------ | :------ | :------------------------------------------------ |
| **bin-all**（推荐） | ~1.6 GB | 包含所有解释器，开箱即用                          |
| **bin-netinst**     | ~799 MB | 仅包含 Spark、Python 等核心解释器，其他需在线安装 |

**推荐下载 `bin-all` 版本**，避免后续手动安装解释器的麻烦。

bash

```shell
# 下载 Zeppelin 0.11.1（全量版）
https://mirrors.pku.edu.cn/apache/zeppelin/
zeppelin-0.11.1-bin-all.tgz
```



### 3.2 解压安装

bash

```shell
# 解压到指定目录
tar -zxvf zeppelin-0.11.1-bin-all.tgz -C /opt/soft/
cd /opt/soft/
mv zeppelin-0.11.1-bin-all zeppelin-0.11.1
```



### 3.3 配置环境变量（可选）

bash

```shell
export ZEPPELIN_HOME=/opt/soft/zeppelin-0.11.1
export PATH=$PATH:$ZEPPELIN_HOME/bin
```



------

## 四、配置 Zeppelin

### 4.1 配置 zeppelin-site.xml

bash

```shell
cd $ZEPPELIN_HOME/conf
cp zeppelin-site.xml.template zeppelin-site.xml
```



编辑 `zeppelin-site.xml`，主要配置 Zeppelin 服务端口：

xml

```xml
<configuration>
    <!-- 使用扩展图形功能 -->
    <property>
        <name>zeppelin.helium.registry</name>
        <value>helium</value>
    </property>
    <!-- Zeppelin 服务端口（默认 8080） -->
    <property>
        <name>zeppelin.server.port</name>
        <value>8080</value>
    </property>
    <!-- 允许远程访问（如需外网访问） -->
    <property>
        <name>zeppelin.server.addr</name>
        <value>0.0.0.0</value>
    </property>
</configuration>
```



### 4.2 配置 zeppelin-env.sh（环境变量）

bash

```shell
cp zeppelin-env.sh.template zeppelin-env.sh
```



编辑 `zeppelin-env.sh`，添加 Java 和 Hadoop 环境：

bash

```shell
# Java 环境
export JAVA_HOME=/opt/soft/jdk17  # 替换为实际路径
export JRE_HOME=$JAVA_HOME

# Hadoop 环境
export HADOOP_HOME=/opt/soft/hadoop-3.3.6
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop

# Hive 环境（用于 JDBC 连接）
export HIVE_HOME=/opt/soft/hive-4.1.0
export HIVE_CONF_DIR=$HIVE_HOME/conf
```

### 4.3 修正Hadoop配置 Core-site.xml

```xml
<!--添加hadoop允许第三方使用任意用户和用户地址登录-->
<name>hadoop.proxyuser.root.groups</name>
  <value>*</value>
</property>
<property>
  <name>hadoop.proxyuser.root.hosts</name>
  <value>*</value>
</property>

```

### 4.4 修正Hadoop配置 mapped-site.xml

```xml
<property>
    <name>yarn.app.mapreduce.am.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/soft/hadoop-3.3.6</value>
</property>
<property>
    <name>mapreduce.map.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/soft/hadoop-3.3.6</value>
</property>
<property>
    <name>mapreduce.reduce.env</name>
    <value>HADOOP_MAPRED_HOME=/opt/soft/hadoop-3.3.6</value>
</property>
```



------

## 五、配置 JDBC 解释器连接 Hive

> **核心原理**：Zeppelin 0.10.0+ 使用 **JDBC 解释器**连接 Hive，而非独立的 Hive 解释器。

### 5.1 准备 Hive JDBC 驱动

Zeppelin 默认不包含 Hive JDBC 驱动，需要手动添加。

bash

```shell
# 1. 进入 JDBC interpreter 目录（如果不存在，先创建）
mkdir -p $ZEPPELIN_HOME/interpreter/jdbc
cd $ZEPPELIN_HOME/interpreter/jdbc

# 2. 复制 Hive 的全部 lib 包（包含所有我们遇到过的：httpclient, curator, thrift, netty, woodstox, commons-collections, hadoop-shaded-guava 等）
cp $HIVE_HOME/lib/*.jar .

# 3. （可选）补充 Hadoop 中可能被 shade 的包（通常 Hive 已带，但加一下无妨）
cp $HADOOP_HOME/share/hadoop/common/lib/hadoop-shaded-*.jar .
# 同时把hive-site.xml 复制到 zeppelin-0.11.1/conf/
```



### 5.2 通过 Web UI 配置 JDBC 解释器

1. **启动 Zeppelin**：

   bash

   ```shell
   cd $ZEPPELIN_HOME
   bin/zeppelin-daemon.sh start
   ```

   

2. **访问 Web 界面**：打开浏览器访问 `http://<服务器IP>:8080`

3. **进入解释器配置页面**：点击右上角 **Anonymous** → **Interpreter**

4. **创建新的 JDBC 解释器**：

   - 点击 **+ Create** 按钮
   - **Interpreter Name**：输入 `hive`（这个名称将用于 Notebook 中 `%hive` 引用）
   - **Interpreter Group**：选择 **jdbc**

5. **配置连接参数**：

   | 属性名             | 值                                     | 说明                       |
   | :----------------- | :------------------------------------- | :------------------------- |
   | `default.driver`   | `org.apache.hive.jdbc.HiveDriver`      | Hive JDBC 驱动类           |
   | `default.url`      | `jdbc:hive2://localhost:10000/default` | HiveServer2 连接地址       |
   | `default.user`     | `root`                                 | 连接用户名（根据实际设置） |
   | `default.password` | （留空）                               | 密码（如未设置）           |

   > **⚠️ 重要**：`default.user` 需与启动 HiveServer2 的用户一致，或使用 Hive 认证配置的用户。

------

## 六、启动 Zeppelin

### 6.1 启动服务

bash

```shell
cd $ZEPPELIN_HOME

# 前台启动（用于调试）
bin/zeppelin-daemon.sh start

# 后台启动（推荐）
nohup bin/zeppelin-daemon.sh start >> logs/zeppelin.log 2>&1 &
```



### 6.2 验证服务状态

bash

```shell
# 检查进程
ps -ef | grep zeppelin

# 检查端口
netstat -tlnp | grep 8080
```



### 6.3 停止服务

bash

```shell
bin/zeppelin-daemon.sh stop
```



------

## 七、验证 Hive 连接

### 7.1 创建 Notebook

1. 访问 `http://<服务器IP>:8080`
2. 点击 **Notebook** → **Create new note**
3. 输入名称，**Default Interpreter** 选择 `jdbc`

### 7.2 执行 Hive 查询

在 Notebook 的段落中输入：

text

```sql
%hive
SHOW DATABASES;
```



点击 **运行** 按钮（或按 `Shift+Enter`），应显示 Hive 中的数据库列表。

**更多测试**：

sql

```sql
%hive
SHOW TABLES;

%hive
SELECT * FROM test LIMIT 10;
```



### 7.3 多连接配置（可选）

如需同时连接多个数据源，可使用前缀区分：

| 属性名         | 值                                     |
| :------------- | :------------------------------------- |
| `hive.driver`  | `org.apache.hive.jdbc.HiveDriver`      |
| `hive.url`     | `jdbc:hive2://localhost:10000/default` |
| `mysql.driver` | `com.mysql.cj.jdbc.Driver`             |
| `mysql.url`    | `jdbc:mysql://localhost:3306/mydb`     |

使用时：

text

```
%jdbc(hive)
SHOW DATABASES;

%jdbc(mysql)
SELECT * FROM users;
```



------

## 八、完整安装流程汇总

bash

```shell
# 1. 下载 Zeppelin
wget https://archive.apache.org/dist/zeppelin/zeppelin-0.11.1/zeppelin-0.11.1-bin-all.tgz

# 2. 解压
tar -zxvf zeppelin-0.11.1-bin-all.tgz -C /usr/local/softwave/
cd /usr/local/softwave/
mv zeppelin-0.11.1-bin-all zeppelin-0.11.1

# 3. 配置环境变量（编辑 ~/.bashrc）
export ZEPPELIN_HOME=/usr/local/softwave/zeppelin-0.11.1
export PATH=$PATH:$ZEPPELIN_HOME/bin
source ~/.bashrc

# 4. 拷贝配置文件模板
cd $ZEPPELIN_HOME/conf
cp zeppelin-site.xml.template zeppelin-site.xml
cp zeppelin-env.sh.template zeppelin-env.sh

# 5. 编辑 zeppelin-env.sh（设置 JAVA_HOME、HADOOP_HOME、HIVE_HOME）

# 6. 复制 Hive JDBC 驱动
cp $HIVE_HOME/lib/hive-jdbc-*.jar $ZEPPELIN_HOME/lib/
cp $HIVE_HOME/lib/hive-service-*.jar $ZEPPELIN_HOME/lib/
cp $HADOOP_HOME/share/hadoop/common/lib/guava-*-jre.jar $ZEPPELIN_HOME/lib/

# 7. 启动 Zeppelin
nohup bin/zeppelin-daemon.sh start >> logs/zeppelin.log 2>&1 &

# 8. 访问 http://<服务器IP>:8080 配置 JDBC 解释器
```



------

## 九、常见问题与解决

| 问题现象                                                  | 可能原因                            | 解决方案                                                     |
| :-------------------------------------------------------- | :---------------------------------- | :----------------------------------------------------------- |
| **启动失败 / 端口被占用**                                 | 8080 端口已被占用                   | 修改 `zeppelin-site.xml` 中 `zeppelin.server.port`           |
| **NoClassDefFoundError: org/apache/hive/jdbc/HiveDriver** | Hive JDBC 驱动未添加到 classpath    | 将 `hive-jdbc-*.jar` 复制到 `$ZEPPELIN_HOME/lib/`            |
| **Invalid SessionHandle**                                 | HiveServer2 会话超时                | 添加 `default.validationQuery=select 1` 和 `default.testOnBorrow=true` |
| **Connection refused**                                    | HiveServer2 未启动                  | 确认 `netstat -tlnp | grep 10000` 有监听                     |
| **Access denied / 认证失败**                              | 用户名或密码错误                    | 检查 `default.user` 是否正确，或尝试使用 `hive` 用户         |
| **Guava 版本冲突**                                        | Zeppelin 与 Hadoop Guava 版本不一致 | 复制 Hadoop 的 `guava-*-jre.jar` 到 `$ZEPPELIN_HOME/lib/` 覆盖 |
| **SQL 执行超时**                                          | 查询耗时过长                        | 在解释器配置中增加 `common.max_result` 或调整超时参数        |
| **Zeppelin 启动慢**                                       | 首次启动加载所有解释器              | 等待 1-2 分钟，查看 `logs/zeppelin.log`                      |

------

## 十、参考资源

- [Apache Zeppelin 官方下载页](https://zeppelin.apache.org/download.html)
- [Zeppelin JDBC 解释器文档](https://zeppelin.apache.org/docs/latest/interpreter/jdbc.html)
- [Zeppelin Hive 解释器说明](https://zeppelin.apache.org/docs/latest/interpreter/hive.html)

------

**文档版本**：1.0
**适用环境**：Hadoop 3.3.6 + Hive 4.1.0 + JDK 17.0.12 + Zeppelin 0.11.1
**最后更新**：2026-08-03