---
type: comparison
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Embedded_System/fpga/led_flash.md
tags:
  - verilog
  - fpga
---

RTL 设计代码描述真实电路、可综合；Testbench 代码只给设计施加虚拟激励、不可综合。Verilog 不是两门语言，而是同一门语言的两个用途。

## 逐项对照

| 对比项 | RTL 设计代码（「原理图代码」） | Testbench 仿真代码 |
|---|---|---|
| 目的 | 描述真实电路，最终变成 FPGA 内部电路 | 施加虚拟激励，验证逻辑 |
| 是否可综合 | 可综合 | 不可综合，不烧进芯片 |
| 常见语法 | `module/endmodule`、`assign`、`always @(posedge clk)`、`if/else`、`case` | `initial`、`#延时`、`$display`、`$finish`、`forever` |
| 是否有端口 | 有输入输出端口 | 通常没有端口 |
| 运行环境 | PDS 综合、布局布线、下载 | ModelSim 等仿真器 |
| 观察结果 | 板上 LED 亮灭 | 波形图 |

## 四条不可越界的规则

- RTL 里**不要用 `#` 延时**描述电路。
- RTL 里**避免用 `initial` 做逻辑初始化**，应使用复位信号。
- Testbench 里离不开 `initial`、`#`、`$finish`。
- 仿真时把 `parameter`（如 `CNT_MAX`）覆盖成小值，避免跑真实时长。

## 一句话

`assign` 和 `always @(posedge clk)` 是造电路；`initial` 和 `#10` 是仿真激励。

## 关联

- [[阻塞与非阻塞赋值]]
- [[引脚约束与位流]]
- [[wire与reg]]
- [[2026-10-27-Verilog点灯入门]]

来源：[[raw/Embedded_System/fpga/led_flash]]
