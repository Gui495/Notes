---
type: entity
status: seedling
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Embedded_System/fpga/led_flash.md
tags:
  - fpga
  - embedded
---

FPGA（Field-Programmable Gate Array，现场可编程门阵列）是通过下载位流改变内部互连与逻辑的可编程逻辑器件；本库的具体对象是盘古 100Pro+ 开发板。

## 板卡信息

| 项 | 内容 |
|---|---|
| 型号 | 盘古 100Pro+（MES2L676-100HP） |
| 配套子卡 | 小眼睛 FMC 子卡、摄像头 |
| 系统时钟 | 常见 27MHz（须以原理图为准） |
| 工具链 | PDS（综合、布局布线、User Constraint Editor、下载） |
| 仿真器 | ModelSim 等 |

## 入门阶段的分层

1. **点灯层**：只用开发板本体，跑通编译→约束→综合→布局布线→位流→下载。
2. **外设层**：点灯稳定后再接 FMC 子卡与摄像头，做采集类实验。

## 学习路径

骨架 → 单灯闪烁 → 改计数上限看速度 → 8 位流水灯 → 按键控制方向/启停 → Testbench 波形验证 → FMC 与摄像头。

## 关联

- [[引脚约束与位流]]
- [[RTL设计与Testbench]]
- [[Verilog模块]]
- [[2026-10-27-Verilog点灯入门]]

来源：[[raw/Embedded_System/fpga/led_flash]]
