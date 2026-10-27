---
type: entity
status: budding
created: 2026-10-27
updated: 2026-10-27
sources:
  - raw/Artificial_Intelligence/Chapter_1_Introduction_to_Artificial_Intelligence.md
  - raw/Artificial_Intelligence/Chapter_3_Basic_Processes_and_Methods.md
  - raw/Artificial_Intelligence/Chapter_4_Machine_Learning.md
  - raw/Artificial_Intelligence/Chapter_6_Reinforcement_Learning.md
tags:
  - ai
  - reinforcement-learning
---

AlphaGo 是 DeepMind 开发的围棋程序，2016 年以 4:1 击败李世石，是深度学习与 [[强化学习]] 结合搜索的标志性系统。

## 战绩

- 2016 年围棋 4:1 击败世界冠军李世石。
- 后续 AlphaZero 战胜 AlphaGo（素材未给时间与赛制细节）。
- 素材另一处提到「AlphaGo 完胜柯洁」，未标注年份。

## 技术构成

三份素材给出的拆解互相补充：

- **两大网络**：策略网络（走棋网络）给出当前状态 s 采取动作 a 的概率 $p_{\sigma,\rho}(a|s)$；价值网络（估值网络）给出状态 s' 的整盘获胜概率 $v_\theta(s')$。
- **搜索**：蒙特卡洛树搜索，每步执行百万级模拟平行对弈；节点扩展用 $\frac{Q(v)}{N(v)}+\frac{P(v|v_0)}{1+N(v)}$ 选择后继，模拟阶段同时参考仿真结果与价值网络输出。
- **训练路线**：先基于监督学习（大量棋谱）训练策略网络，再基于强化学习（自我博弈）继续训练。
- **算力**：1920 个 CPU + 280 个 GPU；素材称训练一次耗电量相当于一个家庭 10 年用电量（该说法未给出处，未核实）。

素材第六章把这一组合概括为「三大法宝」：深度学习感知棋面、强化学习自我博弈、蒙特卡洛树搜索采样学习。

## 与其他页面的关系

搜索部分见 [[2026-10-27-搜索求解与逻辑推理]]（MCTS 四步：选择、扩展、模拟、反向传播）；学习部分见 [[2026-10-27-强化学习与Q-Learning]]；它同时是「领域人工智能」的典型例子，见 [[人工智能]]。素材来源：[[raw/Artificial_Intelligence/Chapter_1_Introduction_to_Artificial_Intelligence]]、[[raw/Artificial_Intelligence/Chapter_3_Basic_Processes_and_Methods]]、[[raw/Artificial_Intelligence/Chapter_4_Machine_Learning]]、[[raw/Artificial_Intelligence/Chapter_6_Reinforcement_Learning]]。
