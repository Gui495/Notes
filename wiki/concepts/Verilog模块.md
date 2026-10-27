---
type: concept
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Embedded_System/fpga/led_flash.md
tags:
  - verilog
  - fpga
---

`module ... endmodule` 是 Verilog 唯一的设计封装单元，端口、内部信号、逻辑描述全部写在里面，必须成对出现。

## 骨架

```verilog
module 模块名 (
    // 端口列表
);
    // 端口声明、内部信号声明、逻辑描述
endmodule
```

- 模块名建议与文件名一致，以字母或下划线开头。
- 现代写法在端口列表里直接写方向、类型、位宽，省掉重复声明。

## 端口方向

| 方向 | 含义 | 初学注意 |
|---|---|---|
| `input` | 输入 | 只能是 `wire` 型 |
| `output` | 输出 | 可以是 `wire` 或 `reg` |
| `inout` | 双向口 | 初学阶段基本不用 |

## 例化（调用子模块）

```verilog
led_blink #(.CNT_MAX(25'd9)) u_led_blink (
    .clk(clk),
    .rst_n(rst_n),
    .led(led)
);
```

- `u_led_blink` 是实例名；`.clk(clk)` 把上层 `clk` 接到子模块同名端口。
- 例化时不要漏端口。
- `#(...)` 用来覆盖子模块的 `parameter`，不改 `localparam`。

## 与 wire/reg 的关系

端口方向决定可选类型：`input` 只能是 `wire`，`output` 要看在 `assign` 里赋值（`wire`）还是在 `always` 里赋值（`reg`）。详见 [[wire与reg]]。

## 关联

- [[阻塞与非阻塞赋值]]
- [[RTL设计与Testbench]]
- [[时序约束]]（综合后时序是否收敛，决定代码能否真正跑到板子上）
- [[2026-10-27-Verilog点灯入门]]

来源：[[raw/Embedded_System/fpga/led_flash]]
