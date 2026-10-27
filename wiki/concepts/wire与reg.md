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

`wire` 是必须被持续驱动的连线，`reg` 是过程块中保存最后一次赋值的变量；两者的分界不是「是否映射成寄存器」，而是「由谁赋值」。

## 对照

| 类型 | 本质 | 赋值方式 | 典型用途 |
|---|---|---|---|
| `wire` | 导线，必须被持续驱动 | 只能在 `assign` 中赋值（或由模块端口驱动） | 连接子模块、组合逻辑连线 |
| `reg` | 保持最后一次赋值的变量 | 只能在 `always` 或 `initial` 中赋值 | 时序逻辑、组合逻辑过程块 |

**记忆口诀**：`assign` 左边必须是 `wire`；`always` 里被赋值的信号必须是 `reg`。

## 最常见的两个误解

1. **`reg` 不等于触发器。** 只有在 `always @(posedge clk)` 里被赋值的 `reg` 才综合成触发器；在 `always @(*)` 里赋值的 `reg` 综合出来是组合逻辑。
2. **`wire` 不能进 `always`。** 下面这样写会报 `cannot assign to wire`：

```verilog
wire led;
always @(posedge clk) begin
    led <= ~led;   // 错误
end
```

改成 `reg led;`，或改用 `assign led = ...`。

## 关联

- [[Verilog模块]]（端口方向决定可选类型）
- [[阻塞与非阻塞赋值]]（`reg` 在时序块里配 `<=`，在组合块里配 `=`）
- [[2026-10-27-Verilog点灯入门]]

来源：[[raw/Embedded_System/fpga/led_flash]]
