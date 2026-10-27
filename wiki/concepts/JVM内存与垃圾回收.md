---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
provenance: external
sources:
  - https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html
  - https://docs.oracle.com/en/java/javase/21/gctuning/garbage-collector-implementation1.html
  - https://docs.oracle.com/en/java/javase/21/gctuning/factors-affecting-garbage-collection-performance1.html
  - https://docs.oracle.com/en/java/javase/21/gctuning/available-collectors.html
  - https://docs.oracle.com/en/java/javase/21/gctuning/other-considerations.html
  - https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/System.html
  - https://openjdk.org/jeps/122
  - https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html
tags:
  - java
---

JVM 内存与垃圾回收是「规范规定的运行时数据区 + HotSpot 实现上的分代收集策略」两层叠加的话题：前者回答对象放在哪，后者回答谁还活着、什么时候回收。

> 📌 来源性质：本页无 raw 素材，依据外部资料编写，待有 raw 素材时再收编。

## 运行时数据区

`JVMS` 第 2 章规定的运行时数据区共六块，判据是「线程私有还是全线程共享」以及「溢出时报什么错」：

| 区域 | 归属 | 存放内容 | 溢出表现 |
| --- | --- | --- | --- |
| 程序计数器（pc 寄存器） | 每线程私有 | 当前执行字节码的行号；执行 native 方法时值未定义 | 规范未定义溢出 |
| Java 虚拟机栈 | 每线程私有 | 栈帧，每帧对应一次方法调用 | 栈深度超限 `StackOverflowError`；无法申请到内存 `OutOfMemoryError` |
| 本地方法栈 | 每线程私有 | native 方法的调用状态（C 栈） | `StackOverflowError` 或 `OutOfMemoryError` |
| 堆 | 全线程共享 | 所有类实例与数组 | `OutOfMemoryError` |
| 方法区 | 全线程共享 | 每类的运行期常量池、字段与方法数据、方法与构造器代码 | `OutOfMemoryError` |
| 运行期常量池 | 每类一份，从方法区分配 | 编译期常量与符号引用 | 随方法区 |

要补的一条是「直接内存」：`ByteBuffer.allocateDirect` 一类分配的缓冲区在堆外，**不属于** JVMS 定义的数据区，因而不受 `-Xmx` 约束，这也是「堆没满却 OOM」的常见来源（其上限选项与对应消息串本页未取得官方原文，待核）。

还有两个规范层面的判据值得先立住。其一，JVMS 明确规定**不规定**对象在堆内的布局，也不规定用哪种 GC 算法，所以「堆分几代、怎么标记」是实现选择而非语言要求——换收集器不改变程序语义。其二，运行期常量池属于方法区，JDK 8 之后这块元数据搬到了本地内存（Metaspace），「方法区」在 HotSpot 里不再是一段固定的堆内空间，见下文。

## 对象分配与 -Xms / -Xmx

JVMS 把堆定义为「所有类实例与数组分配内存的来源」，因此 `new` 的对象默认落在堆上（JIT 的逃逸分析可能把它们拆到栈上或直接消除，属 HotSpot 优化，待核）。

`-Xmx` 决定启动时**预留**的地址空间总量，`-Xms` 决定其中**立即提交**的部分。Oracle 调优文档的表述是：VM 初始化时就预留整个堆空间，预留量由 `-Xmx` 指定；若 `-Xms` 小于 `-Xmx`，则未提交的那部分处于 virtual 状态，各代可在其中按需增长。这解释了一个常见误解：`-Xmx` 约束的是虚拟地址空间上限，不等于常驻内存。

未显式指定时，两个值按机器内存计算：默认最大堆为物理内存的 1/4，初始堆为 1/64（该计算原本用于 Parallel，现在被所有收集器沿用）。默认情况下堆还会在每次收集后涨缩，把空闲比例维持在 `-XX:MinHeapFreeRatio`（默认 40%）与 `-XX:MaxHeapFreeRatio`（默认 70%）之间。把 `-Xms` 与 `-Xmx` 设为相同值可以提高可预测性——文档同时提醒，代价是放弃了 VM 的弹性补偿能力。

年轻代比例由 `-XX:NewRatio` 控制（默认 2，即年轻代约占堆的 1/3；文档示例 `-XX:NewRatio=3` 对应年轻代占 1/4），survivor 与 eden 之比由 `-XX:SurvivorRatio` 控制（默认 8）。默认值随收集器与平台版本有差异，以本机 `-XX:+PrintFlagsFinal` 为准。

## 栈帧与局部变量表

每次方法调用新建一个帧，方法正常或异常结束即销毁。帧内含三样东西：局部变量数组、操作数栈，以及指向当前类运行期常量池的引用。判据有两条：局部变量数组长度与操作数栈最大深度在**编译期**就已确定（所以不会在运行期「栈帧变大」）；实例方法的 0 号局部变量槽固定留给 `this`，`long`/`double` 各占两个连续槽。

这三条直接解释了 `StackOverflowError`：栈大小固定，递归过深时帧数超过上限。它也关系到可达性——栈帧里局部变量引用的对象就是 GC Roots 的一部分，方法返回后这些引用随之消失，对象才可能被回收。所以「方法里的临时大对象」通常不构成泄漏，而「静态字段里塞进去的对象」才是。

## 可达性分析与 GC Roots

HotSpot 的判活方式是从根出发做可达性分析，而不是引用计数。OC 调优文档的定义是：对象从任何其它活对象的引用都不可达时才算垃圾；更严格的说法是「从 GC root 到该对象不存在路径」，且文档明确 GC Roots 包含**活动线程的引用与 JVM 内部引用**。这直接得出两个判据：循环引用不会导致泄漏（引用计数式算法的经典缺陷在此不存在）；一个对象是否泄漏，取决于是否有活线程或 JVM 内部结构还摸得到它。

GC Roots 的**完整清单**在 Oracle 文档里没有封闭枚举——官方只给出上面那句概括。工程上通常还包含：类的静态字段、运行期常量池中的引用、JNI 全局与局部引用、被同步监视器持有的对象、类加载器等 JVM 内部结构。这些属 HotSpot 实现与业界共识，不能当作规范结论引用（待核）。

分代收集还引入一层细节：文档指出年轻代与老年代独立收集，因此**彼此互为 roots**。这就是跨代引用必须被单独跟踪的原因（具体机制如卡表属实现细节，本页不展开，待核）。

## 分代假说与新生代/老年代

分代的依据是 weak generational hypothesis：「大多数对象只存活很短时间」。文档用一个分布图描述对象生命期——左侧尖峰是被快速回收的短命对象，右侧拖尾是活到进程结束的长命对象，中间是中等寿命对象；「绝大多数对象死在年轻时」这个经验事实，使「只扫年轻代」的收集成本远低于「每次扫全堆」。

因此堆被划分为年轻代与老年代（HotSpot 早期还包含永久代，JDK 8 起被 Metaspace 取代）。年轻代内部由 eden 与两个 survivor 组成：对象先在 eden 分配；一次 minor collection 只收年轻代，存活对象被复制到一个空的 survivor，另一个 survivor 与 eden 清空；下一轮两个 survivor 角色互换。对象在 survivor 之间反复复制，达到某个次数阈值或 survivor 空间不足时被**晋升**到老年代，这个复制计数过程称为 aging。老年代填满时发生 major collection，回收整个堆，停顿显著长于 minor——因为要处理多得多的活对象。

尺寸选择上文档给了明确的权衡而非答案：年轻代越大，minor 越少，但固定堆下老年代被压缩、major 反而更频繁；年轻代越小，单次停顿越短，但吞吐下降。文档原话是没有唯一正确的代大小，取决于应用的对象生命期分布。

**版本差异**：eden + 两个 survivor 的连续布局是 Serial 收集器的默认排布（文档明说该章以 Serial 为例）。G1 及之后的收集器把堆切成大小相等的 region，按 region 回收，因此上面对「物理连续的三段空间」的描述不适用于它们；G1 的 young-only / space-reclamation 阶段划分属该收集器文档，本页未逐字核验（待核）。

## 常见收集器

判据只有一句话：文档建议先让 VM 自选，再调整堆大小，仍不满足再换收集器。

| 收集器 | 一句话定位 | 启用参数 | 典型适用 |
| --- | --- | --- | --- |
| Serial | 单线程、全程 stop-the-world，最简单也最省内存 | `-XX:+UseSerialGC` | 单处理器、小数据集（文档给出的量级是约 100MB 以内） |
| Parallel | 吞吐优先的多线程收集器（throughput collector） | `-XX:+UseParallelGC` | 中型到大型数据集、能容忍较长停顿的批处理 |
| CMS | 并发标记清除，以低停顿为目标，会产生浮动垃圾 | 已弃用/移除 | 历史方案，见下 |
| G1 | mostly concurrent，多数硬件的默认，按暂停时间目标调度并兼顾吞吐 | `-XX:+UseG1GC` | 大堆、需要可预测停顿的通用服务 |
| ZGC | 最大停顿声称不超过 1 毫秒，代价是牺牲吞吐，停顿与堆大小无关 | `-XX:+UseZGC`（分代模式 `-XX:+ZGenerational`） | 极大堆（文档量级：几百 MB 到 16TB）、停顿敏感 |

CMS 值得单独说一句，因为它是「并发收集」的代价样本：它与应用线程并发标记，标记结束后应用可能已把某些对象改成不可达，这些「已死但未被回收」的对象就是 floating garbage；文档建议按约 20% 预留老年代空间来吸收它。CMS 已被弃用并从较新 JDK 移除（JEP 291 弃用、JEP 363 移除，具体版本本页未逐条抓取原文，待核），新项目不应再选它。

ZGC 的分代模式在 JDK 21 起是默认行为，`-XX:-ZGenerational` 可退回非分代实现；这是版本差异明显的一处，跨版本迁移时须核对。

## Minor / Major / Full GC 的触发

三种说法对应三种范围，触发点各不相同。minor collection 在年轻代填满时发生，只收年轻代，成本与「被收集的活对象数」成正比，因此装满死对象的年轻代收得极快。major collection 在老年代填满时发生，回收整个堆。full collection 在 GC 日志里表现为 `Pause Full`，文档示例中由 `System.gc()` 触发。此外类元数据提交量达到高水位时也会诱发一次 GC 以卸载类，高水位初值取自 `-XX:MetaspaceSize`。

还有一条容易忽略的路径：若一次回收后进程把约 98% 的时间花在 GC 上、且每次回收不到 2% 的堆、连续 5 次如此，就会抛 `OutOfMemoryError: GC overhead limit exceeded`——此时堆并未真正耗尽，而是「活着的数据刚好塞满堆」。该检查可用 `-XX:-UseGCOverheadLimit` 关闭。

术语上要留一个余量：minor / major / full 是传统分代收集器的语言，G1 之后日志改称 young / full 并细分阶段，不同收集器的触发条件并不完全对齐（待核具体阶段判据）。

## OutOfMemoryError 的几种形态

判断 OOM 类型只看异常末尾的 detail message，它比栈更重要：

| 消息 | 含义 | 常见判据 |
| --- | --- | --- |
| `Java heap space` | 堆内分配失败且堆无法再扩展 | 可能是堆配小了，也可能是对象被意外长期持有（真泄漏） |
| `GC overhead limit exceeded` | 回收时间占比过高、几乎回收不到空间 | 活数据接近堆上限，先加堆或查泄漏 |
| `Metaspace` | 类元数据耗尽（JDK 8 起在本地内存，默认无上限） | 动态生成类过多；用 `-XX:MaxMetaspaceSize` 设上限 |
| `Compressed class space` | 压缩类指针空间耗尽 | 该空间默认 1GB（`-XX:CompressedClassSpaceSize`） |
| `Requested array size exceeds VM limit` | 数组长度超过实现上限，与剩余堆无关 | 参数或长度计算错误 |
| `request size bytes for reason. Out of swap space?` | 本地堆/交换空间不足，并非堆内对象分配失败 | 属系统级问题，需看 fatal error log |
| `Direct buffer memory` | 直接缓冲区超出上限 | 与 `-XX:MaxDirectMemorySize` 相关（本页未核验其默认值，待核） |
| `StackOverflowError`（不是 OOM） | 线程请求的栈深度超过限制 | 递归无终止、深调用链；调 `-Xss` 只是掩盖问题 |

Metaspace 的默认值有两个易错点：一是「默认无上限」意味着它受机器物理内存约束，而不是受 `-Xmx` 约束，所以「堆还有余量但进程被 OOM killer 杀掉」是典型症状；二是 `-XX:MetaspaceSize` 只是触发首次元数据 GC 的高水位初值（默认随平台在 12MB 到 20MB 之间），并不是容量上限——真正的上限是 `-XX:MaxMetaspaceSize`。开启压缩类指针时，类元数据被拆成两块本地内存，`-XX:MaxMetaspaceSize` 约束的是两者之和。

## System.gc() 只是建议

`System.gc()` 的 javadoc 用词是「suggests」：它建议 JVM 尽力回收，但不保证回收任何数量的对象、不保证在方法返回前完成，效果等价于 `Runtime.getRuntime().gc()`。调优文档的立场更明确：显式 GC 可能在不必要时强制一次 major collection（例如本来一次 minor 就够），因此通常应避免；`-XX:+DisableExplicitGC` 会让 VM 直接忽略 `System.gc()` 调用。

但也有正当场景：RMI 的分布式 GC 需要周期性触发本地整堆回收，否则远端对象无法被判定为不可达，其频率可用 `-Dsun.rmi.dgc.client.gcInterval` 调整。这条恰好说明「显式 GC 是协议需要，而不是清理资源的工具」。

由此得到的工程判据是：凡是依赖 `System.gc()`「顺便」释放文件句柄、直接缓冲区的代码都是错的，因为回收时机不可控。资源释放应走 try-with-resources 或 `Cleaner`；`finalize()` 这条老路已在 JDK 9 弃用、JDK 18 起弃用待移除（JEP 421），而 `Cleaner` 同样由 GC 调度、存在无界延迟，不能用于「必须及时释放」的资源。

## 关联

- [[Java]]
- [[面向对象]]
- [[多线程]]
- [[集合框架]]
- [[异常处理]]
- [[2026-10-27-多线程]]
- [[2026-10-27-Java基础语法]]

## 来源

- Java 虚拟机规范 SE 21 第 2 章（运行时数据区、帧、对象表示）: https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-2.html （访问日期 2026-10-27）
- HotSpot GC 调优指南, Garbage Collector Implementation: https://docs.oracle.com/en/java/javase/21/gctuning/garbage-collector-implementation1.html （访问日期 2026-10-27）
- HotSpot GC 调优指南, Factors Affecting Garbage Collection Performance: https://docs.oracle.com/en/java/javase/21/gctuning/factors-affecting-garbage-collection-performance1.html （访问日期 2026-10-27）
- HotSpot GC 调优指南, Available Collectors: https://docs.oracle.com/en/java/javase/21/gctuning/available-collectors.html （访问日期 2026-10-27）
- HotSpot GC 调优指南, Other Considerations（GC Roots 概括、显式 GC、Class Metadata）: https://docs.oracle.com/en/java/javase/21/gctuning/other-considerations.html （访问日期 2026-10-27）
- Java SE 21 API, java.lang.System#gc(): https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/System.html （访问日期 2026-10-27）
- JEP 122: Remove the Permanent Generation: https://openjdk.org/jeps/122 （访问日期 2026-10-27）
- Java SE 8 Troubleshooting Guide, Understand the OutOfMemoryError Exception: https://docs.oracle.com/javase/8/docs/technotes/guides/troubleshoot/memleaks002.html （访问日期 2026-10-27）

版本差异说明：堆默认值、Metaspace 行为、收集器可用性均随 JDK 版本变化；OOM 消息清单与 `-XX:MetaspaceSize` 默认区间取自 JDK 8 故障排查文档与 JDK 21 调优文档，跨版本使用时以对应版本文档为准。标「待核」的条目（GC Roots 完整清单、直接内存上限与消息、CMS 移除版本、G1 阶段划分、JIT 逃逸分析细节）本页未取得一手原文，引用前须复核。
