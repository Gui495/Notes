下面是一份可直接复制保存为 `Verilog点灯入门笔记.md` 的 Markdown 笔记，覆盖前面讨论的知识点、代码模板、引脚约束提醒和常见问题排查。

---

# Verilog 点灯入门笔记（盘古 100Pro+ / MES2L676-100HP）

> 目标：从零理解 Verilog 基础规范，能看懂并写出可综合的点灯/流水灯代码，知道如何仿真、约束引脚、下载到开发板，并能排查常见问题。  
> 注意：具体引脚号、时钟频率、LED 极性、复位按键极性，**必须以官方原理图、实验手册和配套约束文件为准**。本文中 27MHz 仅为常见参考值，使用前请核对。

---

## 目录

1. [硬件与开发背景](#1-硬件与开发背景)
2. [两类代码：RTL 设计代码 vs Testbench 仿真代码](#2-两类代码rtl-设计代码-vs-testbench-仿真代码)
3. [Verilog 固定骨架](#3-verilog-固定骨架)
4. [端口与信号类型：input/output/inout，wire/reg](#4-端口与信号类型inputoutputinoutwirereg)
5. [逻辑描述：assign 与 always](#5-逻辑描述assign-与-always)
6. [时序逻辑与组合逻辑：阻塞赋值和非阻塞赋值](#6-时序逻辑与组合逻辑阻塞赋值和非阻塞赋值)
7. [posedge 和 negedge](#7-posedge-和-negedge)
8. [begin/end 规则](#8-beginend-规则)
9. [parameter 和 localparam](#9-parameter-和-localparam)
10. [常量、位宽、注释、模块例化](#10-常量位宽注释模块例化)
11. [点灯代码示例：单灯闪烁](#11-点灯代码示例单灯闪烁)
12. [流水灯代码示例](#12-流水灯代码示例)
13. [Testbench 仿真代码示例](#13-testbench-仿真代码示例)
14. [引脚约束与下载流程](#14-引脚约束与下载流程)
15. [常见问题排查表](#15-常见问题排查表)
16. [学习路径与检查清单](#16-学习路径与检查清单)
17. [术语速查](#17-术语速查)

---

## 1. 硬件与开发背景

- 开发板：盘古 100Pro+（MES2L676-100HP）。
- 配套：小眼睛 FMC 子卡、摄像头。
- 点灯阶段：**先不接 FMC 和摄像头**，只用开发板本身跑通“编译 → 约束 → 综合 → 布局布线 → 生成位流 → 下载”。
- FPGA 逻辑系统时钟：常见为 **27MHz**，但必须查原理图/手册确认。若实际不是 27MHz，计数器上限要重新计算。
- LED 极性：有些板卡 LED 是**低电平点亮**，有些是高电平点亮。若下载后亮灭相反，把输出取反即可。
- 复位按键：常见为**低电平有效**，即平时为 1，按下为 0，所以代码中常用 `negedge rst_n` 和 `if (!rst_n)`。

---

## 2. 两类代码：RTL 设计代码 vs Testbench 仿真代码

Verilog 不是“两种语言”，而是同一门语言用于两个不同目的。

| 对比项 | RTL 设计代码（“原理图代码”） | Testbench 仿真代码 |
|---|---|---|
| 目的 | 描述真实电路，最终变成 FPGA 内部电路 | 给设计施加虚拟激励，验证逻辑 |
| 是否可综合 | 可综合 | 不可综合，不烧进芯片 |
| 常见语法 | `module/endmodule`、`assign`、`always @(posedge clk)`、`if/else`、`case` | `initial`、`#延时`、`$display`、`$finish`、`forever` |
| 是否有端口 | 有输入输出端口 | 通常没有端口 |
| 运行环境 | PDS 综合、布局布线、下载 | ModelSim 等仿真器 |
| 观察结果 | 板上 LED 亮灭 | 波形图 |

### 关键区别

- RTL 代码里**不要用 `#` 延时**去描述电路。
- RTL 代码里**避免依赖 `initial` 做逻辑初始化**，应使用复位信号。
- Testbench 里离不开 `initial`、`#`、`$finish`。
- RTL 是“造电路”，Testbench 是“质检员”。

> 一句话：`assign` 和 `always @(posedge clk)` 是造电路；`initial` 和 `#10` 是仿真激励。

---

## 3. Verilog 固定骨架

所有可综合设计都长在下面这个骨架里：

```verilog
module 模块名 (
    // 端口列表
);
    // 端口声明、内部信号声明、逻辑描述
endmodule
```

`module` 和 `endmodule` 必须成对出现。模块名建议与文件名一致，以字母或下划线开头。

### 最小完整骨架

```verilog
module blink (
    input  clk,
    input  rst_n,
    output reg led
);
    localparam CNT_MAX = 25'd13_499_999;
    reg [24:0] cnt;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cnt <= 25'd0;
            led <= 1'b0;
        end else if (cnt >= CNT_MAX) begin
            cnt <= 25'd0;
            led <= ~led;
        end else begin
            cnt <= cnt + 1'b1;
        end
    end
endmodule
```

这个骨架包含：端口声明、`localparam`、内部 `reg`、`always` 时序块、`if/else`、`begin/end`、非阻塞赋值。

---

## 4. 端口与信号类型：input/output/inout，wire/reg

### 端口方向

| 方向 | 含义 | 初学注意 |
|---|---|---|
| `input` | 输入 | 只能是 `wire` 型 |
| `output` | 输出 | 可以是 `wire` 或 `reg` |
| `inout` | 双向口 | 初学阶段基本不用 |

现代写法通常在端口列表中直接声明方向、类型、位宽：

```verilog
module led_blink (
    input  clk,
    input  rst_n,
    output reg led
);
```

### wire 和 reg

| 类型 | 本质 | 赋值方式 | 典型用途 |
|---|---|---|---|
| `wire` | 导线，必须被持续驱动 | 只能在 `assign` 中赋值 | 连接子模块、组合逻辑连线 |
| `reg` | 保持最后一次赋值的变量 | 只能在 `always` 或 `initial` 中赋值 | 时序逻辑、组合逻辑过程块 |

> **记忆口诀**：`assign` 左边必须是 `wire`；`always` 里被赋值的信号必须是 `reg`。

### 常见错误

```verilog
wire led;
always @(posedge clk) begin
    led <= ~led;   // 错误：always 里不能给 wire 赋值
end
```

应改为：

```verilog
reg led;
always @(posedge clk) begin
    led <= ~led;
end
```

---

## 5. 逻辑描述：assign 与 always

### assign：连续赋值，适合简单组合逻辑

```verilog
assign y = a & b;
```

只要 `a` 或 `b` 变化，`y` 立刻跟着变化。

### always：过程块，适合时序逻辑或复杂组合逻辑

时序逻辑模板：

```verilog
always @(posedge clk or negedge rst_n) begin
    if (!rst_n)
        led <= 1'b0;
    else
        led <= ~led;
end
```

组合逻辑模板：

```verilog
always @(*) begin
    y = a & b;
end
```

- `always @(*)` 表示敏感列表自动包含块内读取的所有信号。
- 组合逻辑中必须保证所有分支都赋值，否则可能综合出锁存器。
- 时序逻辑中通常使用非阻塞赋值 `<=`。
- 组合逻辑中通常使用阻塞赋值 `=`。

---

## 6. 时序逻辑与组合逻辑：阻塞赋值和非阻塞赋值

| 项目 | 时序逻辑 | 组合逻辑 |
|---|---|---|
| 敏感列表 | `posedge clk` / `negedge rst_n` | `always @(*)` |
| 赋值符号 | 非阻塞 `<=` | 阻塞 `=` |
| 综合结果 | 触发器/寄存器 | 纯组合逻辑 |
| 典型用途 | 计数器、状态机、流水灯 | 译码、简单逻辑运算 |

### 时序逻辑标准写法

```verilog
always @(posedge clk or negedge rst_n) begin
    if (!rst_n) begin
        cnt <= 25'd0;
        led <= 1'b0;
    end else if (cnt >= CNT_MAX) begin
        cnt <= 25'd0;
        led <= ~led;
    end else begin
        cnt <= cnt + 1'b1;
    end
end
```

### 组合逻辑标准写法

```verilog
always @(*) begin
    if (sel)
        y = a;
    else
        y = b;
end
```

> 初学阶段最重要的两条：
> 1. `always @(posedge clk)` 里一律用 `<=`。
> 2. `always @(*)` 里一律用 `=`，并且每个分支都要给输出赋值。

---

## 7. posedge 和 negedge

| 关键字 | 含义 | 跳变方向 |
|---|---|---|
| `posedge` | 上升沿 | 信号从 `0` 变 `1` 的瞬间 |
| `negedge` | 下降沿 | 信号从 `1` 变 `0` 的瞬间 |

可以类比门铃：`posedge` 是按下瞬间，`negedge` 是松开瞬间。电路只在跳变瞬间触发。

### 点灯代码中的含义

```verilog
always @(posedge clk or negedge rst_n) begin
```

意思是：

- `clk` 每次从 0 跳到 1 的瞬间，执行一次；
- 或者 `rst_n` 从 1 跳到 0 的瞬间，执行复位分支。

### 为什么复位常用 negedge？

因为板上复位按键通常是低电平有效：平时为 1，按下变 0。按下瞬间是下降沿，所以用 `negedge rst_n`。

```verilog
if (!rst_n) begin
    // 复位逻辑
end
```

`!rst_n` 表示 `rst_n` 为 0 时成立。

---

## 8. begin/end 规则

`begin ... end` 相当于 C 语言的 `{ ... }`。

### 规则

- `if`、`else`、`always`、`for` 后面如果只有**一条语句**，可以省略 `begin/end`。
- 如果有**两条或以上语句**，必须用 `begin/end` 包起来。
- `if ... else if ... else` 链中，**每个分支各自判断**是否需要 `begin/end`。

### 示例

只有一条语句，可以省略：

```verilog
if (!rst_n)
    led <= 1'b0;
```

多条语句，必须包起来：

```verilog
if (!rst_n) begin
    cnt <= 25'd0;
    led <= 1'b0;
end
```

链式结构：

```verilog
if (a) begin
    x = 1;
    y = 2;
end else if (b) begin
    x = 3;
    y = 4;
end else begin
    x = 0;
end
```

> 建议：初学阶段即使只有一条语句，也可以加 `begin/end`，减少漏写风险。

---

## 9. parameter 和 localparam

两者都是常量，综合后不占用硬件资源，只是让代码可读、好维护。

| 对比项 | `parameter` | `localparam` |
|---|---|---|
| 能否被上层例化修改 | 可以 | 不可以 |
| 典型用途 | 可配置参数：位宽、分频系数、计数上限 | 模块内部固定常量：状态编码、固定计数上限 |
| 作用范围 | 可被上层覆盖 | 只在当前模块内有效 |

### parameter 示例

```verilog
module led_blink #(
    parameter CNT_MAX = 25'd13_499_999
)(
    input clk,
    input rst_n,
    output reg led
);
    // ...
endmodule
```

上层例化时可以覆盖：

```verilog
led_blink #(.CNT_MAX(25'd9)) u_led_blink (
    .clk(clk),
    .rst_n(rst_n),
    .led(led)
);
```

这样仿真时可以把计数上限改得很小，快速看到 LED 翻转。

### localparam 示例

```verilog
localparam IDLE = 2'b00;
localparam WORK = 2'b01;
localparam DONE = 2'b10;
```

状态机编码、模块内部固定值适合用 `localparam`。

> 一句话：`parameter` 是“可配置的默认值”，`localparam` 是“模块内部不许改的常量”。

---

## 10. 常量、位宽、注释、模块例化

### 常量格式

```verilog
8'b1010_1010   // 8 位二进制
4'd9           // 4 位十进制
1'b0           // 1 位二进制 0
25'd13_499_999 // 25 位十进制
```

格式：`位宽'进制数值`。

### 位宽计算

27MHz 时钟下，0.5 秒需要：

```text
27_000_000 × 0.5 = 13_500_000 个时钟周期
```

计数从 0 到 13_499_999，共 13_500_000 个周期。

至少需要 24 位，因为 `2^24 = 16_777_216 > 13_499_999`。代码中可以用 25 位更保险。

> 位宽宁大勿小，但也不要过大浪费资源。

### 注释

```verilog
// 单行注释
/* 多行注释 */
```

### 模块例化

```verilog
led_blink u_led_blink (
    .clk(clk),
    .rst_n(rst_n),
    .led(led)
);
```

- `u_led_blink` 是实例名。
- `.clk(clk)` 表示把上层 `clk` 连接到子模块的 `clk` 端口。
- 例化时不要漏端口。

---

## 11. 点灯代码示例：单灯闪烁

下面代码假设：

- 时钟 27MHz；
- 复位低有效；
- `led` 高电平点亮。若板卡低电平点亮，把 `led <= ~led` 的逻辑反过来，或输出取反。

```verilog
module led_blink #(
    parameter CNT_MAX = 25'd13_499_999   // 27MHz 下约 0.5 秒
)(
    input  clk,
    input  rst_n,
    output reg led
);

    reg [24:0] cnt;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cnt <= 25'd0;
            led <= 1'b0;
        end else if (cnt >= CNT_MAX) begin
            cnt <= 25'd0;
            led <= ~led;
        end else begin
            cnt <= cnt + 1'b1;
        end
    end

endmodule
```

### 如果 LED 是低电平点亮

可以把输出改成：

```verilog
output reg led_n;   // 低有效
// ...
led_n <= ~led_n;
```

或者在顶层取反：

```verilog
assign led = ~led_n;
```

具体以原理图为准。

---

## 12. 流水灯代码示例

```verilog
module led_water #(
    parameter CNT_MAX = 26'd13_500_000   // 27MHz 下约 0.5 秒
)(
    input  clk,
    input  rst_n,
    output reg [7:0] led
);

    reg [25:0] cnt;
    reg [7:0]  led_status;

    always @(posedge clk or negedge rst_n) begin
        if (!rst_n) begin
            cnt <= 26'd0;
            led_status <= 8'b0000_0001;
        end else if (cnt >= CNT_MAX - 1) begin
            cnt <= 26'd0;
            led_status <= {led_status[6:0], led_status[7]}; // 循环左移
        end else begin
            cnt <= cnt + 1'b1;
        end
    end

    always @(*) begin
        led = ~led_status;   // 低电平点亮
        // 若高电平点亮，改为：led = led_status;
    end

endmodule
```

说明：

- `{led_status[6:0], led_status[7]}` 是拼接操作，实现 8 位循环左移。
- `always @(*)` 中给 `led` 赋值，所以 `led` 声明为 `output reg`。
- 若下载后亮灭相反，修改 `led = ~led_status;` 和 `led = led_status;`。

---

## 13. Testbench 仿真代码示例

Testbench 不会被综合，只用于仿真。

```verilog
`timescale 1ns / 1ps

module tb_led_blink;

    reg  clk;
    reg  rst_n;
    wire led;

    // 例化设计模块，并把 CNT_MAX 改小，便于快速仿真
    led_blink #(
        .CNT_MAX(25'd9)
    ) u_led_blink (
        .clk(clk),
        .rst_n(rst_n),
        .led(led)
    );

    // 产生 27MHz 时钟，周期约 37ns，半周期约 18.5ns
    initial begin
        clk = 0;
        forever #18.5 clk = ~clk;
    end

    // 复位和运行控制
    initial begin
        rst_n = 0;
        #100;
        rst_n = 1;
        #1000;
        $finish;
    end

endmodule
```

要点：

- Testbench 模块通常没有端口。
- `initial` 和 `#` 延时是仿真专用。
- `forever` 产生周期时钟。
- `$finish` 结束仿真。
- `$display` 可用于打印信息。
- 仿真时把 `CNT_MAX` 改小，可以避免仿真 0.5 秒真实时间过长。

---

## 14. 引脚约束与下载流程

代码写完后必须做引脚约束，否则无法正确生成位流或下载后不工作。

### 需要约束的信号

- `clk`：系统时钟引脚。
- `rst_n`：复位按键引脚。
- `led` 或 `led[7:0]`：LED 引脚。

### 获取引脚号的方法

1. 查阅官方实验指导手册中的“接口列表”和“工程约束”章节。
2. 打开板卡原理图，查找 FPGA 与 LED、晶振、复位按键连接的页码。
3. 参考小眼睛官方例程中的约束文件。PDS 工程常见约束文件扩展名以实际工程为准，如 `.fdc` / `.uce` 等。
4. 在 PDS 的 User Constraint Editor 中添加引脚约束。

> 不要照抄网上的引脚号，不同板卡版本可能不同。

### 下载流程概览

1. 新建/打开 PDS 工程。
2. 添加 Verilog 源文件。
3. 设置顶层模块 Top Module。
4. 添加引脚约束。
5. 综合。
6. 布局布线。
7. 生成位流文件。
8. 连接下载器，下载到 FPGA。
9. 观察 LED。

### 点灯阶段关于 FMC 和摄像头

- 先不接 FMC 子卡和摄像头。
- 先确认板卡本身、下载器、PDS 工具链正常。
- 点灯稳定后，再通过 FMC 接口做摄像头采集等进阶实验。

---

## 15. 常见问题排查表

| 问题 | 可能原因 | 排查方法 |
|---|---|---|
| LED 完全不亮 | 引脚约束错误、下载失败、LED 极性反、复位一直按下 | 查约束、重新下载、改极性、检查复位按键 |
| LED 常亮或常灭 | 计数器没翻转、复位未释放、极性反 | 仿真看 `led` 是否翻转；检查 `rst_n` |
| 闪烁太快/太慢 | 时钟频率不是 27MHz、`CNT_MAX` 算错、位宽溢出 | 查原理图确认时钟；重新计算计数上限；加宽计数器 |
| 综合报错 “cannot assign to wire” | 在 `always` 里给 `wire` 赋值 | 把被赋值信号改成 `reg`，或改用 `assign` |
| 综合警告 “latch inferred” | 组合逻辑 `always @(*)` 分支不全 | 补全 `else` 或 `default`，确保所有分支都赋值 |
| 仿真波形无变化 | 没释放复位、没产生时钟、仿真时间太短、`CNT_MAX` 太大 | 检查 `rst_n`、时钟、`$finish` 时间；把 `CNT_MAX` 改小 |
| 仿真正确但板上不对 | Testbench 时钟频率与板卡不同、复位极性不同、LED 极性不同 | 按板卡实际时钟和极性修改 |
| 多位宽常量警告 | 常量位宽与信号位宽不匹配 | 显式写位宽，如 `25'd13_499_999` |
| 多个 always 驱动同一 reg | 同一信号在多个 `always` 中被赋值 | 保证一个信号只在一个 `always` 中赋值 |
| RTL 中用了 `#` 延时 | 不可综合或仿真/综合不一致 | RTL 中删除 `#`，延时只放 Testbench |
| RTL 中依赖 `initial` 初始化 | 综合结果不可靠 | 使用复位信号初始化 |
| 顶层模块设置错误 | PDS 中 Top Module 选错 | 检查工程顶层设置 |
| 约束未生效 | 约束文件未加入、引脚号错、未重新综合 | 检查约束文件、重新综合和布局布线 |
| FMC 摄像头不工作 | 点灯阶段未接或未配置 | 先跑通基础点灯，再查 FMC 和摄像头例程 |
| 复位行为不对 | 复位高有效/低有效判断错 | 低有效用 `negedge rst_n` 和 `!rst_n`；高有效用 `posedge rst` 和 `if (rst)` |
| `begin/end` 漏写 | 多语句分支没包起来 | 每个多语句分支加 `begin/end` |
| `parameter` 覆盖不生效 | 例化时没写 `#(.参数名(...))` | 检查例化参数语法 |

---

## 16. 学习路径与检查清单

### 推荐学习顺序

1. 看懂 `module/endmodule` 骨架。
2. 写单灯闪烁，跑通综合和下载。
3. 改 `CNT_MAX`，观察闪烁速度变化。
4. 写 8 位流水灯。
5. 加按键控制流水灯方向或启停。
6. 写 Testbench，用波形验证计数器。
7. 再接触 FMC、摄像头等外设。

### 下载前检查清单

- [ ] 顶层模块设置正确。
- [ ] `clk`、`rst_n`、`led` 都已约束引脚。
- [ ] 时钟频率与 `CNT_MAX` 匹配。
- [ ] LED 极性确认。
- [ ] 复位按键极性确认。
- [ ] 没有在 RTL 中使用 `#` 延时。
- [ ] `always @(posedge clk)` 中使用 `<=`。
- [ ] `always @(*)` 中分支完整，无锁存器警告。
- [ ] 没有多个 `always` 驱动同一信号。
- [ ] 综合、布局布线无严重错误。
- [ ] 下载器连接正常，位流下载成功。

---

## 17. 术语速查

| 术语 | 含义 |
|---|---|
| RTL | 寄存器传输级，可综合的设计描述 |
| Testbench | 仿真测试平台，不可综合 |
| `module/endmodule` | Verilog 模块的固定骨架 |
| `wire` | 线网，必须被持续驱动，`assign` 赋值 |
| `reg` | 过程块中赋值的变量，可在 `always` 中赋值 |
| `assign` | 连续赋值，常用于组合逻辑 |
| `always` | 过程块，可描述时序或组合逻辑 |
| `posedge` | 上升沿，0→1 瞬间 |
| `negedge` | 下降沿，1→0 瞬间 |
| `begin/end` | 相当于 C 语言的 `{ }` |
| `parameter` | 可被上层例化覆盖的常量 |
| `localparam` | 模块内部固定常量 |
| 阻塞赋值 `=` | 组合逻辑常用 |
| 非阻塞赋值 `<=` | 时序逻辑常用 |
| 锁存器推断 | 组合逻辑分支不全导致的意外存储元件 |
| 引脚约束 | 把端口绑定到 FPGA 真实引脚 |
| 位流 | 最终下载到 FPGA 的配置文件 |

---

> 备注：本文中的 27MHz、LED 极性和复位极性均为常见参考，实际以盘古 100Pro+（MES2L676-100HP）官方原理图、实验手册和配套约束文件为准。点灯阶段先不接 FMC 和摄像头，优先跑通基础工具链。