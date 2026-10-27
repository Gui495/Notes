---
created: 2026-10-14
updated: 2026-10-27
tags:
  - index
---

# wiki 总索引

本文由 agent 维护。约定见 [AGENTS.md](../AGENTS.md)；人类的日常入口在 `areas/`。

> 说明：本页是索引而非内容页，故 frontmatter 不含 `type` / `status` / `sources`。依据见 AGENTS.md §4.1。

## 统计

| 类型 | 数量 |
|---|---|
| `summaries/` | 36 |
| `concepts/` | 64 |
| `entities/` | 19 |
| `comparisons/` | 5 |
| `synthesis/` | 3 |
| **内容页合计** | **127** |
| `_Index_Wiki.md` | 1 |

**编译覆盖**：`raw/` 现有 38 个文件 = 37 篇知识素材 + 1 个前端库（`axios.min.js`，非知识素材，排除）。37 篇素材对应 36 篇摘要（`CentOS-7-reg.repo.md` 并入 [[2026-10-27-CentOS7安装MySQL]]）。**未编译素材：0 篇。**

> **2026-10-27 大规模整合**：owner 删除根目录 `index.md` / `log.md` / `schema.md` / `_Index_All.md` 与 `raw/` 下 4 个 `_Index_*.md`；同时新增 29 篇素材（人工智能 9、Java 9、MySQL 5、大数据生态 5、FPGA 1）。 wiki 由 44 页扩至 113 页（+69）：摘要 +27、概念 +30、实体 +11、对比 +1。

> **2026-10-27 第二轮（owner 指令 m00782）**：按 AGENTS.md 新增的 §4.3 补 12 个「无 raw 素材、依据外部资料编写」的概念页（`provenance: external`，status 上限 `budding`），并新增 3 篇 `synthesis/` 跨主题综合。wiki 由 113 页扩至 128 页（+15）：概念 +12、综合 +3。`raw/` 未增删，编译覆盖仍为 36/37。

---

## 一、按学科浏览

### 1. College_Math（3 篇素材）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-14-标准高斯积分]] | [[高斯积分]]、[[雅可比行列式]]、[[配方法]] |
| [[2026-10-14-二维正态分布边缘密度]] | [[二维正态分布]]、[[边缘概率密度]]、[[一维正态分布]] |
| [[2026-10-14-离散与连续随机变量的边缘分布]] | （对比型素材，未新增概念页） |

对比：[[一维与二维正态分布]]、[[离散与连续随机变量的边缘分布]]

### 2. Microcomputer_Principles（4 篇素材）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-14-80x86微处理器]] | [[字长]]、[[CPU组成]]、[[80x86寄存器]]、[[实地址模式]]、[[物理地址计算]]、[[标志寄存器]] |
| [[2026-10-14-汇编语言指令集（一）]] | [[汇编指令格式]]、[[寻址方式]]、[[小端存储]]、[[标志寄存器]] |
| [[2026-10-14-原码反码补码]] | [[原码反码补码]] |
| [[2026-10-14-二进制与十六进制算术及逻辑运算]] | [[位运算]] |

实体：[[80x86]]　对比：[[实模式与保护模式]]

### 3. Linux（2 篇素材）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-14-ArchLinux安装流程]] | [[磁盘分区与挂载]]、[[UEFI双系统引导]]、[[持久化块设备命名]]、[[Linux中文显示与locale]]、[[pacman故障排查]] |
| [[2026-10-14-ArchLinux安装问题排查]] | 同上 |

实体：[[Arch Linux]]、[[pacman]]、[[systemd-boot]]、[[GRUB]]、[[iwd]]、[[fcitx5]]、[[NJUPT_Fastlogin]]

对比：[[GRUB与systemd-boot]]

### 4. Artificial_Intelligence（9 篇素材，课程《人工智能导论》八章 + 一门 ML 库课）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-27-人工智能绪论]] | [[人工智能]]、[[图灵测试]] |
| [[2026-10-27-人工智能应用]] | （承接 [[图灵测试]]、[[机器学习]]） |
| [[2026-10-27-搜索求解与逻辑推理]] | [[专家系统]]、[[知识表示]] |
| [[2026-10-27-机器学习]] | [[机器学习]]、[[神经网络]] |
| [[2026-10-27-深度学习与神经网络]] | [[深度学习]]、[[卷积神经网络]] |
| [[2026-10-27-强化学习与Q-Learning]] | [[强化学习]] |
| [[2026-10-27-博弈安全]] | （承接纳什均衡、对抗样本，未单独立页） |
| [[2026-10-27-大语言模型与低空经济]] | （承接 [[深度学习]]） |
| [[2026-10-27-机器学习科学计算库]] | （工具型素材，未新增概念页） |

实体：[[图灵]]、[[达特茅斯会议]]、[[AlphaGo]]、[[人工智能导论]]

外部来源补页：[[K均值聚类]]、[[感受野与残差连接]]（`provenance: external`，课程素材未展开，本库自行补）

### 5. Java（9 篇素材，按学习顺序）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-27-Java基础语法]] | （变量、运算符、流程控制、数组） |
| [[2026-10-27-类与对象]] | [[面向对象]]、[[类与对象]] |
| [[2026-10-27-方法与重载]] | [[方法重载]] |
| [[2026-10-27-输入输出处理]] | [[IO流]] |
| [[2026-10-27-集合框架与泛型（一）]] | [[集合框架]] |
| [[2026-10-27-集合框架与泛型（二）]] | [[泛型]] |
| [[2026-10-27-多线程]] | [[多线程]] |
| [[2026-10-27-反射机制]] | [[反射机制]] |
| [[2026-10-27-实用类与枚举]] | [[枚举]] |

实体：[[Java]]

### 6. MySQL（5 篇素材，两条安装路线）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-27-MySQL8安装]] | （二进制包装 8.0.30） |
| [[2026-10-27-MySQL数据管理]] | [[DML数据管理]] |
| [[2026-10-27-MySQL事务索引与备份]] | （事务、索引、备份，未单独立页） |
| [[2026-10-27-DQL查询]] | [[DQL查询]] |
| [[2026-10-27-CentOS7安装MySQL]] | （yum 装 5.6/5.7，含 `CentOS-7-reg.repo`） |

实体：[[MySQL]]、[[CentOS]]

### 7. 大数据生态（3 篇素材）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-27-大数据安装]] | [[大数据4V]]、[[分布式系统]]、[[HDFS]]、[[MapReduce]]、[[YARN]] |
| [[2026-10-27-Hive]] | [[数据仓库]] |
| [[2026-10-27-Apache Zeppelin]] | （可视化前端，未新增概念页） |

实体：[[Hadoop]]、[[Hive]]、[[Apache Zeppelin]]

外部来源补页：[[Hadoop集群模式]]、[[Hive分区与分桶]]（`provenance: external`）

### 8. Embedded_System（1 篇素材）

| 素材摘要 | 概念页 |
|---|---|
| [[2026-10-27-Verilog点灯入门]] | [[Verilog模块]]、[[wire与reg]]、[[阻塞与非阻塞赋值]]、[[引脚约束与位流]] |

外部来源补页：[[时序约束]]（`provenance: external`，素材只讲语法不讲综合后时序）

实体：[[FPGA]]　对比：[[RTL设计与Testbench]]

---

## 二、按类型浏览

### summaries（36）

- [[2026-10-14-标准高斯积分]] ← [[raw/College_Math/标准高斯积分]]
- [[2026-10-14-二维正态分布边缘密度]] ← [[raw/College_Math/二维正态分布边缘密度]]
- [[2026-10-14-离散与连续随机变量的边缘分布]] ← [[raw/College_Math/离散_连续随机变量的边缘分布]]
- [[2026-10-14-80x86微处理器]] ← [[raw/Microcomputer_Principles/80x86微处理器]]
- [[2026-10-14-汇编语言指令集（一）]] ← [[raw/Microcomputer_Principles/汇编语言指令集（一）]]
- [[2026-10-14-原码反码补码]] ← [[raw/Microcomputer_Principles/原码、反码、补码]]
- [[2026-10-14-二进制与十六进制算术及逻辑运算]] ← [[raw/Microcomputer_Principles/B、H的算术和逻辑运算]]
- [[2026-10-14-ArchLinux安装流程]] ← [[raw/Linux/ArchLinux Installation Guide]]
- [[2026-10-14-ArchLinux安装问题排查]] ← [[raw/Linux/Installation_Problems]]
- [[2026-10-27-人工智能绪论]] ← [[raw/Artificial_Intelligence/Chapter_1_Introduction_to_Artificial_Intelligence]]
- [[2026-10-27-人工智能应用]] ← [[raw/Artificial_Intelligence/Chapter_2_Applications_of_Artificial_Intelligence]]
- [[2026-10-27-搜索求解与逻辑推理]] ← [[raw/Artificial_Intelligence/Chapter_3_Search_Solving_and_Logical_Reasoning]]
- [[2026-10-27-机器学习]] ← [[raw/Artificial_Intelligence/Chapter_4_Machine_Learning]]
- [[2026-10-27-深度学习与神经网络]] ← [[raw/Artificial_Intelligence/Chapter_5_Deep_Learning_and_Neural_Networks]]
- [[2026-10-27-强化学习与Q-Learning]] ← [[raw/Artificial_Intelligence/Chapter_6_Reinforcement_Learning]]
- [[2026-10-27-博弈安全]] ← [[raw/Artificial_Intelligence/Chapter_7_Game_Theory_and_Security]]
- [[2026-10-27-大语言模型与低空经济]] ← [[raw/Artificial_Intelligence/Chapter_8_Development_and_Challenges]]
- [[2026-10-27-机器学习科学计算库]] ← [[raw/Artificial_Intelligence/Machine_Learning (Scientific_Computing_Library)]]
- [[2026-10-27-Java基础语法]] ← [[raw/Frontend_Backend/Java_Basics]]
- [[2026-10-27-类与对象]] ← [[raw/Frontend_Backend/Classes_and_Objects]]
- [[2026-10-27-方法与重载]] ← [[raw/Frontend_Backend/Methods_and_Method_Overloading]]
- [[2026-10-27-输入输出处理]] ← [[raw/Frontend_Backend/Input_and_Output_Processing]]
- [[2026-10-27-集合框架与泛型（一）]] ← [[raw/Frontend_Backend/Set_Framework_and_Generics1]]
- [[2026-10-27-集合框架与泛型（二）]] ← [[raw/Frontend_Backend/Set_Framework_and_Generics2]]
- [[2026-10-27-多线程]] ← [[raw/Frontend_Backend/Multithreading]]
- [[2026-10-27-反射机制]] ← [[raw/Frontend_Backend/Reflex_Mechanism]]
- [[2026-10-27-实用类与枚举]] ← [[raw/Frontend_Backend/Practical_Introduction]]
- [[2026-10-27-MySQL8安装]] ← [[raw/Frontend_Backend/MySQL8]]
- [[2026-10-27-MySQL数据管理]] ← [[raw/Frontend_Backend/MySQL_Data_Management]]
- [[2026-10-27-MySQL事务索引与备份]] ← [[raw/Frontend_Backend/MySQL_Operations]]
- [[2026-10-27-DQL查询]] ← [[raw/Frontend_Backend/DQL_Command]]
- [[2026-10-27-CentOS7安装MySQL]] ← [[raw/Frontend_Backend/Centos7_Install_MySQL]]、[[raw/Frontend_Backend/CentOS-7-reg.repo]]
- [[2026-10-27-大数据安装]] ← [[raw/Frontend_Backend/大数据安装]]
- [[2026-10-27-Hive]] ← [[raw/Frontend_Backend/Hive]]
- [[2026-10-27-Apache Zeppelin]] ← [[raw/Frontend_Backend/Apache_Zeppelin]]
- [[2026-10-27-Verilog点灯入门]] ← [[raw/Embedded_System/fpga/led_flash]]

### concepts（64）

**数学（6）**：[[高斯积分]]、[[雅可比行列式]]、[[配方法]]、[[二维正态分布]]、[[一维正态分布]]、[[边缘概率密度]]

**微机原理（11）**：[[原码反码补码]]、[[位运算]]、[[字长]]、[[CPU组成]]、[[80x86寄存器]]、[[实地址模式]]、[[物理地址计算]]、[[标志寄存器]]、[[寻址方式]]、[[汇编指令格式]]、[[小端存储]]

**Linux（5）**：[[磁盘分区与挂载]]、[[UEFI双系统引导]]、[[持久化块设备命名]]、[[Linux中文显示与locale]]、[[pacman故障排查]]

**人工智能（11）**：[[人工智能]]、[[图灵测试]]、[[机器学习]]、[[深度学习]]、[[神经网络]]、[[卷积神经网络]]、[[强化学习]]、[[专家系统]]、[[知识表示]]、[[K均值聚类]]、[[感受野与残差连接]]

**Java（12）**：[[面向对象]]、[[类与对象]]、[[方法重载]]、[[集合框架]]、[[泛型]]、[[多线程]]、[[反射机制]]、[[枚举]]、[[IO流]]、[[继承与多态]]、[[异常处理]]、[[JVM内存与垃圾回收]]

**MySQL（5）**：[[DQL查询]]、[[DML数据管理]]、[[事务隔离级别]]、[[B+树索引]]、[[binlog与增量恢复]]

**大数据（8）**：[[大数据4V]]、[[分布式系统]]、[[HDFS]]、[[MapReduce]]、[[YARN]]、[[数据仓库]]、[[Hadoop集群模式]]、[[Hive分区与分桶]]

**FPGA（5）**：[[Verilog模块]]、[[wire与reg]]、[[阻塞与非阻塞赋值]]、[[引脚约束与位流]]、[[时序约束]]

**运维（1）**：[[版本生命周期与EOL]]

> **外部来源补课页（12，`provenance: external`）**：[[版本生命周期与EOL]]、[[继承与多态]]、[[异常处理]]、[[JVM内存与垃圾回收]]、[[事务隔离级别]]、[[B+树索引]]、[[binlog与增量恢复]]、[[Hadoop集群模式]]、[[Hive分区与分桶]]、[[K均值聚类]]、[[感受野与残差连接]]、[[时序约束]]。
> 依据 AGENTS.md §4.3 生成：status 上限 `budding`，页内含固定警示行与 `## 来源`（URL + 访问日期）。它们**不计入**「已编译素材」，`raw/` 覆盖数不受影响。

### entities（19）

- 微机原理：[[80x86]]
- Linux：[[Arch Linux]]、[[pacman]]、[[systemd-boot]]、[[GRUB]]、[[iwd]]、[[fcitx5]]、[[NJUPT_Fastlogin]]
- 人工智能：[[图灵]]、[[达特茅斯会议]]、[[AlphaGo]]、[[人工智能导论]]
- Java/MySQL：[[Java]]、[[MySQL]]、[[CentOS]]
- 大数据：[[Hadoop]]、[[Hive]]、[[Apache Zeppelin]]
- FPGA：[[FPGA]]

### comparisons（5）

- [[离散与连续随机变量的边缘分布]] —— 离散求和 vs 连续积分，含最小算例
- [[一维与二维正态分布]] —— 常数、交叉项与边缘关系的逐项对照
- [[实模式与保护模式]] —— 地址空间、内存管理、安全性、多任务，并登记素材未覆盖项
- [[GRUB与systemd-boot]] —— 安装方式、配置载体、Windows 探测、各自踩过的坑
- [[RTL设计与Testbench]] —— 可综合与否、语法边界、运行环境、四条不可越界规则

### synthesis（3，跨主题整合）

- [[概率与机器学习之间的一段路]] —— 高斯积分/正态/边缘化 ↔ 机器学习/神经网络，附四个接口与素材缺口
- [[从裸机到Hive-Metastore]] —— CentOS 7 → JDK 17 → Hadoop → MySQL 8.0 → Hive → Zeppelin 的版本与依赖链
- [[寄存器时钟与分频]] —— 微机原理的寄存器/标志/位运算 ↔ FPGA 的 reg/wire/always，含三个本质差别

---

## 三、素材错误与裁定登记（已裁定 28 条）

裁定原则：`raw/` 永不改写；wiki 层同时记录「正确值 + 原文错值 + 判定证据」，状态标「已裁定」而**不删除痕迹**，以便日后回溯（AGENTS.md §4.2）。

### 3.1 数学 / 微机原理 / Linux（4 条）

| # | 内容 | 判定 |
|---|---|---|
| 1 | `20A0H:0F6H` 原文写作 **21B60H** | 正确值 **20AF6H**；21B60H 恰为 `2100H×16+0B60H`，属抄录串行 → [[物理地址计算]] |
| 2 | 设备名 `/dev/mem0n1`、`/dev/mem1` | 应为 `/dev/nvme0n1`、`/dev/nvme1n1`；`mem*` 不符内核 NVMe 命名 → [[持久化块设备命名]] |
| 3 | 状态标志列 4 项 vs 6 项 | 以**六项**为准（CF/PF/AF/ZF/SF/OF），四项为精简表 → [[标志寄存器]] |
| 4 | 《B、H的算术和逻辑运算》逻辑运算例题残缺 | **2026-10-27 关闭**：owner 删除该章例题并要求不再追补；残留的掩码/置位两行可唯一还原（`10101100 AND 11110000 = 10100000`；`10000011 OR 00001000 = 10001011`）→ [[2026-10-14-二进制与十六进制算术及逻辑运算]] |

### 3.2 大数据生态（11 条，见各页正文）

| # | 原文错值 | 正确值 |
|---|---|---|
| 5 | HBase 解压目录 `hbase-2.6.0-bin` | 目录是 `hbase-2.6.0`，`-bin` 只是包名后缀 |
| 6 | 示例 IP `192.168.80.128` | 全文其余处为 `192.168.220.129`，照抄示例必失败 |
| 7 | 要求 JDK 17.0.16+，路径全为 `jdk17012` | 版本与路径不一致，按路径为准 |
| 8 | `hive.server2.active.passive.ha.enable` 注释「免去检查」 | 该项是启用 HS2 主备 HA，不是跳过检查 |
| 9 | Zeppelin bin-all 体积 1.6GB / 799MB | 0.11.1 实约 843MB / 577MB（1.5G 是 0.10.1 量级） |
| 10 | `zeppelin-daemon.sh start` 称前台启动 | 它是后台守护；前台应 `bin/zeppelin.sh` |
| 11 | Zeppelin 4.3 节 XML 缺首个 `<property>` 开标签 | 直接粘贴会解析失败，需补全 |
| 12 | `mapped-site.xml`；发布日期「2024 年 3 月」 | `mapred-site.xml`；0.11.1 实际发布 **2024-04-02** |
| 13 | MySQL 临时密码路径 `/var/log/mysql.log` | 应为 `/var/log/mysqld.log` |
| 14 | `character-set-server` 写在 `[mysql]` 段（且该段重复两次） | 它是服务端变量，必须写进 `[mysqld]`，否则修乱码不生效 |
| 15 | 「Hive 4.1.0 + Hadoop 3.3.6 + JDK17」成一整套 | 4.1.0 官方配套 Hadoop 3.4.1；配 3.3.6 的是 4.0.1；Zeppelin 的 JDK17 支持要到 0.13.0 |

### 3.3 Java / MySQL（5 条）

| # | 原文错值 | 正确值 |
|---|---|---|
| 16 | 「关系运算符只支持数值类型比较」 | `> < >= <=` 仅数值；`==`/`!=` 可用于 boolean 与引用类型 → [[2026-10-27-Java基础语法]] |
| 17 | Constructor/Field/Modifier 属 `java.lang.*` | 在 `java.lang.reflect.*`，仅 `Class` 在 `java.lang` → [[2026-10-27-反射机制]] |
| 18 | EXPLAIN 连接类型 `eq_reg`、`indexhe` | 应为 `eq_ref`、`index` → [[2026-10-27-MySQL事务索引与备份]] |
| 19 | 「全文索引只能用于 MyISAM」 | MySQL 5.6.4 起 InnoDB 也支持 `FULLTEXT`；8.0 默认引擎即 InnoDB |
| 20 | DQL 骨架 `[ left\|out\|inner join ]` | `out` 应为 `right`；SQL 无 `OUT JOIN` → [[2026-10-27-DQL查询]] |
| 27 | MySQL 5.x 线与 8.0 线（原待裁决 A）：字符集 `utf8` vs `utf8mb4`、授权 `GRANT ... IDENTIFIED BY` vs `CREATE USER` | 2026-10-27 owner 裁定「版本分家」：两套语法各自只对自家版本成立，不得互抄；[[2026-10-27-CentOS7安装MySQL]] 已加页级「仅适用 5.x」横幅 → [[MySQL]] |

### 3.4 人工智能（6 条）

| # | 原文错值 | 正确值 |
|---|---|---|
| 21 | 「2012 AlexNet 正确率超越人类」 | AlexNet top-5 错误率 15.3%，人类基线约 5.1%；首个低于人类的是 2015 ResNet（3.57%） |
| 22 | 「露西约 400 万年前」 | 约 320 万年（3.2 Ma），1974 年埃塞俄比亚 Hadar，完整度约 40% |
| 23 | 「1990 年第五代工程宣告失败」 | 计划期 1982–1992，1992 年结束；「失败」是 90 年代初的评价 |
| 24 | 「囚徒困境是不完全信息博弈」 | 应为**完全信息**：收益矩阵是共同知识 |
| 25 | 「微软 Phi 4、14 亿参数」 | 140 亿（14B） |
| 26 | 「2024-06 UCSD 实验 500 人 / GPT-4 54% / 人类 67%，已超 30% 标准」 | 方向属实（Jones & Bergen）：发表版 49.7% vs 66%，但论文明确 GPT-4 最佳提示词未显著优于随机（p=0.864），「已超过 30%」不成立 |
| 28 | 图灵测试「30% 误判率」被素材写成通过标准（原待裁决 B） | 2026-10-27 owner 裁定：30% 是图灵 1950 年论文的预测性阈值，非共识判据；引用时写「图灵的原话阈值」→ [[图灵测试]] |

---

## 四、未裁决冲突与待补

### 4.1 原待裁决项的处理（2 条，2026-10-27 已全部裁定）

| # | 冲突 | 处置 | 落点 |
|---|---|---|---|
| A | **MySQL 5.x 线 vs 8.0 线**（字符集、授权语法） | owner 裁定「版本分家」：[[2026-10-27-CentOS7安装MySQL]] 加页级「仅适用 5.x」横幅，两套语法不得互抄 | CentOS7 页首 + [[MySQL]] 实体页 |
| B | **图灵测试 30% 的性质**（素材写成标准，概念页判定为预测） | owner 裁定：30% 是图灵 1950 年论文的预测性阈值，非共识判据，引用写「图灵的原话阈值」 | [[图灵测试]]、[[2026-10-27-人工智能绪论]]、[[2026-10-27-机器学习科学计算库]] |

### 4.2 素材内部前后不一致（未裁定，14 条）

- **MySQL8**：`my.cnf` 的 socket 在第 6 步是 `/usr/local/mysql/data/mysql.sock`，第 7 步软链到 `/usr/local/mysql/mysql.sock`（少 `data/`）；服务注册同时建 `/etc/init.d/mysql` 软链与复制 `/etc/init.d/mysqld`，两套并存；全程 `--user=root` 跑 mysqld，与 8.0 建议的专用 `mysql` 用户相悖。
- **集合框架（一）**：体系图把 Set 写成「不唯一无序」，同篇正文却写「Set 唯一无序」——应为元素唯一。
- **MySQL_Data_Management**：「TRUNCATE TABLE 不会对表名有影响」语义不明（TRUNCATE 本就不改表名），疑为「表结构」笔误。
- **输入输出处理**：由两篇素材合并而成，节号重复。
- **人工智能 Ch1**：称达特茅斯报告「七大议题」却列 8 条；达特茅斯提案四人名单与会议实际参与者需区分。
- **人工智能 Ch7**：「无针对性攻击」的示例实为有目标攻击（「使其被错误识别为数字 3」），有目标攻击段又以「同样属于黑盒攻击」收尾，与损失项作用接不上。
- **人工智能 Ch2**：自动驾驶 SAE 分级 L4 与 L5 的说明文字完全相同，L4 应限定在设计运行范围（ODD）内。
- **人工智能 Ch5**：把「无激活函数」与「Softmax」并列为对照，两者不在同一层（前者缺非线性，后者是输出归一化）；Softmax 算例 0.9525/0.0471/0.0004 与直算 0.9523/0.0474/0.0003 有取整差。
- **大数据安装**：ZooKeeper `dataDir` 在 `/usr/local/software/...` 而 `ZOOKEEPER_HOME` 在 `/opt/soft/...`，两套根目录无法判断真实路径；`chown -R root:root /var/lib/mysql` 与 EL7 包默认 `mysql:mysql` 相反，照抄 mysqld 起不来；Zeppelin 的 Hadoop 路径出现三套（`/opt/soft`、`/usr/local/softwave`、`/opt/soft/jdk17`）。

### 4.3 无法核实（标「待核」，不作定论）

- 低空经济「2030 年 2 万亿、2035 年超 5.1 万亿」无出处与口径；「通信行业碳排放占全球约 4%」常见口径为 ICT 全行业；Llama 2 只取 700 亿最大型号、DeepSeek 6710 亿未标型号。
- dev.mysql.com 两个官方文档页编译时 `web_fetch` 返回 **HTTP 403**，未取得正文，页内已注明「仅作条目指向」。

### 4.4 素材缺失（待 owner 决定是否补素材）

- **Java**：无继承/接口/多态、无系统异常处理、无注解、无 Lambda/Stream、无 JVM 内存与 GC、无 Maven/Gradle；面向对象只讲到封装。
- **MySQL**：事务无隔离级别与并发异常；索引无 B+ 树与最左前缀；EXPLAIN 无真实输出；备份无 binlog 与增量恢复；DQL 只有 SELECT，无 DDL/DCL/视图/存储过程/触发器；例题用到的 `student`/`subject`/`result`/`grade` 四张表**无建表语句**，无法复现。
- **大数据**：只有伪分布式，缺多节点真分布式、HDFS 配额、YARN 队列与 HA、Hive 分区/分桶建模；整条栈踩在 EOL 版本上（CentOS 7 已于 2024-06-30 EOL、MySQL 5.6/5.7、Hive 3.x）。
- **人工智能**：专家系统架构分层与 MYCIN 等经典系统未展开；K-means 只给步骤未给目标函数；CNN 缺卷积数学定义、感受野、BN/残差；反向传播完整推导缺失。
- **FPGA**：只覆盖点灯/流水灯，缺时序约束、跨时钟域、仿真覆盖率。
- **微机原理**：`80x86微处理器` 素材含单选题/判断题/填空题三组例题，本库摘要保留答案速查（例题仍在原文中）。

---

## 五、尚未覆盖 / 后续可做

- `areas/` 已实现 8 个学科的 MOC（人类入口层）。
- `synthesis/` 仍未启用：目前缺跨学科整合性论述。可考虑：把「码制 → 位运算 → 标志寄存器」串成一条数据流综述；或抽象出数学「换元求不变量」与微机「段址换算」的共同模式（后者需更多素材）。
- 跨学科真实交叉点已出现 3 处，可作为 `synthesis/` 的首批候选：① [[Linux中文显示与locale]] 与 MySQL 字符集都表现为「中文乱码」，但分属 locale/字体层与数据库字符集层；② [[位运算]] 的微机原理语境与 Java 位运算符语境；③ [[人工智能]] 的「判别/生成方法」（概率建模）与生成式 AI（产品形态）是同词不同层。
- 根目录历史学科文件夹（`微机原理/` 等，见 `.obsidian/workspace.json` 记录）已不存在。
- **自动蒸馏链路仍故障**，详见下节。

## 六、自动蒸馏链路故障（2026-10-14 确诊，2026-10-27 未修复）

| 项 | 结论 |
|---|---|
| 现象 | `~/.dsh/dsh-kb/queue.json` 中 12 条记录全部 `failed`，`attempts: 2` |
| 报错 | `turn/end → {"kind":"error","error":{"message":"format v4 message requires a producer-owned source kind"}}` |
| 关键证据 | 任取一个 `kb-distill-*` 会话记录，`createdAt` 有值但 **events 为空、turns 为 0、inbox 全空、`isSeeded: false`**——插件注入的首条消息在进入模型前就被 v4 会话格式校验拒绝，模型从未被调用 |
| 根因定位 | 故障在**插件构造蒸馏消息**这一侧（缺 producer-owned source kind），不是素材、不是模型、不是权限 |
| 影响面 | 新素材不会自动编译，且失败是静默的；`log.md` 永远为空 |
| 已采取的兜底 | 本库所有内容改为**交互会话编译**；编译覆盖以本页 §6.2 台账为准，不依赖队列状态 |

**待上游修复**：报错位于 app.asar 内的打包代码，本会话无法打补丁。

### 6.1 恢复方案（四步，需人类执行第 1 步）

| 步骤 | 操作 | 归属 |
|---|---|---|
| 1 | 修上游 bug：给 kb 插件构造的蒸馏消息补上 producer-owned source kind | 人类 / 上游 |
| 2 | 复位队列：`queue.json` 里 12 条记录的 `status` 改回 `pending`、`attempts` 归 0、清空 `retryAt` | 人类 |
| 3 | 触发：往 `raw/` 丢一篇新素材，观察队列是否被消费 | 人类 |
| 4 | 验证三连：`status` 变 `done`、`pages` 非空、`log.md` 出现 `kb-bot` 行 | 人类 |

**为什么第 1 步必须先行**：当前 12 条已 `attempts: 2` 并终止，不会无限重试；但只要上游 bug 在，复位后重跑必然同样失败。

### 6.2 raw 编译台账（37 篇素材 → 36 篇摘要）

自动链路不可靠时，本表是「哪些已编译」的唯一权威来源。新增素材后请在此追加一行。

| # | raw 文件 | 状态 | 产物 |
|---|---|---|---|
| 1 | `raw/College_Math/标准高斯积分.md` | ✅ | [[2026-10-14-标准高斯积分]] + 3 概念 + 1 对比 |
| 2 | `raw/College_Math/二维正态分布边缘密度.md` | ✅ | [[2026-10-14-二维正态分布边缘密度]] + 3 概念 + 1 对比 |
| 3 | `raw/College_Math/离散_连续随机变量的边缘分布.md` | ✅ | [[2026-10-14-离散与连续随机变量的边缘分布]] + 1 对比 |
| 4 | `raw/Microcomputer_Principles/原码、反码、补码.md` | ✅ | [[2026-10-14-原码反码补码]] + [[原码反码补码]] |
| 5 | `raw/Microcomputer_Principles/B、H的算术和逻辑运算.md` | ✅ | [[2026-10-14-二进制与十六进制算术及逻辑运算]] + [[位运算]] |
| 6 | `raw/Microcomputer_Principles/80x86微处理器.md` | ✅ | [[2026-10-14-80x86微处理器]] + 6 概念 + 1 对比 + [[80x86]] |
| 7 | `raw/Microcomputer_Principles/汇编语言指令集（一）.md` | ✅ | [[2026-10-14-汇编语言指令集（一）]] + 3 概念 |
| 8 | `raw/Linux/ArchLinux Installation Guide.md` | ✅ | [[2026-10-14-ArchLinux安装流程]] + 5 概念 + 6 实体 + 1 对比 |
| 9 | `raw/Linux/Installation_Problems.md` | ✅ | [[2026-10-14-ArchLinux安装问题排查]] + 1 实体 |
| 10 | `raw/Artificial_Intelligence/Chapter_1_Introduction_to_Artificial_Intelligence.md` | ✅ | [[2026-10-27-人工智能绪论]] + [[图灵]] + [[达特茅斯会议]] |
| 11 | `raw/Artificial_Intelligence/Chapter_2_Applications_of_Artificial_Intelligence.md` | ✅ | [[2026-10-27-人工智能应用]] |
| 12 | `raw/Artificial_Intelligence/Chapter_3_Search_Solving_and_Logical_Reasoning.md` | ✅ | [[2026-10-27-搜索求解与逻辑推理]] + [[专家系统]] + [[知识表示]] |
| 13 | `raw/Artificial_Intelligence/Chapter_4_Machine_Learning.md` | ✅ | [[2026-10-27-机器学习]] + [[机器学习]] |
| 14 | `raw/Artificial_Intelligence/Chapter_5_Deep_Learning_and_Neural_Networks.md` | ✅ | [[2026-10-27-深度学习与神经网络]] + [[深度学习]] + [[神经网络]] + [[卷积神经网络]] |
| 15 | `raw/Artificial_Intelligence/Chapter_6_Reinforcement_Learning.md` | ✅ | [[2026-10-27-强化学习与Q-Learning]] + [[强化学习]] |
| 16 | `raw/Artificial_Intelligence/Chapter_7_Game_Theory_and_Security.md` | ✅ | [[2026-10-27-博弈安全]] |
| 17 | `raw/Artificial_Intelligence/Chapter_8_Development_and_Challenges.md` | ✅ | [[2026-10-27-大语言模型与低空经济]] |
| 18 | `raw/Artificial_Intelligence/Machine_Learning (Scientific_Computing_Library).md` | ✅ | [[2026-10-27-机器学习科学计算库]] |
| 19 | `raw/Frontend_Backend/Java_Basics.md` | ✅ | [[2026-10-27-Java基础语法]] |
| 20 | `raw/Frontend_Backend/Classes_and_Objects.md` | ✅ | [[2026-10-27-类与对象]] + [[面向对象]] + [[类与对象]] |
| 21 | `raw/Frontend_Backend/Methods_and_Method_Overloading.md` | ✅ | [[2026-10-27-方法与重载]] + [[方法重载]] |
| 22 | `raw/Frontend_Backend/Input_and_Output_Processing.md` | ✅ | [[2026-10-27-输入输出处理]] + [[IO流]] |
| 23 | `raw/Frontend_Backend/Set_Framework_and_Generics1.md` | ✅ | [[2026-10-27-集合框架与泛型（一）]] + [[集合框架]] |
| 24 | `raw/Frontend_Backend/Set_Framework_and_Generics2.md` | ✅ | [[2026-10-27-集合框架与泛型（二）]] + [[泛型]] |
| 25 | `raw/Frontend_Backend/Multithreading.md` | ✅ | [[2026-10-27-多线程]] + [[多线程]] |
| 26 | `raw/Frontend_Backend/Reflex_Mechanism.md` | ✅ | [[2026-10-27-反射机制]] + [[反射机制]] |
| 27 | `raw/Frontend_Backend/Practical_Introduction.md` | ✅ | [[2026-10-27-实用类与枚举]] + [[枚举]] + [[Java]] |
| 28 | `raw/Frontend_Backend/MySQL8.md` | ✅ | [[2026-10-27-MySQL8安装]] |
| 29 | `raw/Frontend_Backend/MySQL_Data_Management.md` | ✅ | [[2026-10-27-MySQL数据管理]] + [[DML数据管理]] |
| 30 | `raw/Frontend_Backend/MySQL_Operations.md` | ✅ | [[2026-10-27-MySQL事务索引与备份]] + [[MySQL]] |
| 31 | `raw/Frontend_Backend/DQL_Command.md` | ✅ | [[2026-10-27-DQL查询]] + [[DQL查询]] |
| 32 | `raw/Frontend_Backend/Centos7_Install_MySQL.md` | ✅ | [[2026-10-27-CentOS7安装MySQL]] + [[CentOS]] |
| 33 | `raw/Frontend_Backend/CentOS-7-reg.repo.md` | ✅（并入 32） | 见 [[2026-10-27-CentOS7安装MySQL]] |
| 34 | `raw/Frontend_Backend/大数据安装.md` | ✅ | [[2026-10-27-大数据安装]] + 5 概念 + [[Hadoop]] |
| 35 | `raw/Frontend_Backend/Hive.md` | ✅ | [[2026-10-27-Hive]] + [[数据仓库]] + [[Hive]] |
| 36 | `raw/Frontend_Backend/Apache_Zeppelin.md` | ✅ | [[2026-10-27-Apache Zeppelin]] + [[Apache Zeppelin]] |
| 37 | `raw/Embedded_System/fpga/led_flash.md` | ✅ | [[2026-10-27-Verilog点灯入门]] + 4 概念 + [[FPGA]] + 1 对比 |
| — | `raw/Frontend_Backend/axios.min.js` | ⏭ 排除 | 前端库源码，非知识素材 |

**覆盖率：37 / 37 篇素材已编译（36 篇摘要，1 篇并入），0 篇待编译。**
