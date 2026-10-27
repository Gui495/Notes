# Notes

一个由人类与 AI agent 共同维护的 Obsidian 知识库。

采用 **LLM Wiki** 模式：原始素材进入 `raw/`，由 agent 编译为结构化的 `wiki/` 页面，`areas/` 作为人类的日常入口。知识随每次投喂而**复利式增长**。

---
## 核心理念

传统笔记工具的问题在于：笔记越写越多，但知识没有变聪明。每次查询都要重新翻找，旧的结论不会被新的理解更新，跨主题的连接全靠人脑。

本仓库的做法是把知识库当成一个**代码库**：

- **原始素材 = 源码**，只增不改，作为可追溯的证据
- **AI agent = 编译器**，持续把源码编译为结构化的知识页面
- **wiki = 编译产物**，包含概念页、实体页、摘要、对比、综合论述
- **areas = 用户界面**，你日常从这里进入知识库

每次投喂新素材，agent 不只索引它，而是**通读现有 wiki，把它整合进已有知识网络**——更新概念页、修订摘要、标注矛盾、建立双链。这是「自生长」的本质：**知识库随使用而变聪明，而不是越用越乱**。

---
## 三层架构

| 层 | 谁维护 | 放什么 | 作用 |
|---|---|---|---|
| `raw/` | 人类只写不改 | 网页剪藏、PDF、课堂速记、代码 dump、聊天记录 | 证据仓库，可溯源 |
| `wiki/` | agent 拥有 | 概念页、实体页、摘要、对比、综合论述 | 编译产物，知识网络 |
| `areas/` | 人类主导，agent 出草稿 | 学科 MOC、学习地图、进度、待办 | 日常入口，驾驶舱 |

三者的关系：

```
raw/  ──编译──▶  wiki/  ──汇总──▶  areas/
原料            知识             入口
```

- `raw/` 是只增不减的档案室，你不需要天天打开
- `wiki/` 是 agent 的工作区，你定期审校即可
- `areas/` 是你每天打开 Obsidian 后进入的第一个地方

---
## 目录结构

```
Notes/
├── AGENTS.md                  # agent 操作契约，必须先读
├── README.md                  # 本文件
├── raw/                       # 原始素材层（只读）
│   ├── College_Math/
│   ├── Linux/
│   └── Microcomputer_Principles/
├── wiki/                      # 编译知识层（agent 拥有）
│   ├── concepts/              # 概念页：一个概念一页
│   ├── entities/              # 实体页：人物、工具、论文、课程
│   ├── summaries/             # 摘要页：对单个 raw 素材的提炼
│   ├── comparisons/           # 对比分析：多个概念的横向比较
│   ├── synthesis/             # 综合论述：跨主题的整合
│   └── _Index_Wiki.md         # wiki 总索引
├── areas/                     # 人类入口层
│   ├── College_Math.md        # 每个学科一个 MOC
│   ├── Linux.md
│   ├── Microcomputer_Principles.md
│   └── _drafts/               # agent 生成的草稿，人类审阅后移出
└── _Index_All.md              # 全量索引（人类维护）
```

历史遗留的学科文件夹（`College_Math/` 等）暂留根目录，逐步分流到 raw 或 areas。

---
## 工作流

### 投喂

把学习资料丢进 `raw/` 对应学科目录。可以是：

- 网页剪藏的 markdown
- 课堂速记
- PDF 的文字提取
- 代码片段与注释
- 与 AI 对话中值得保留的部分

### 编译

对 agent 说：

```
处理 raw/ 里的新素材
```

agent 会：

1. 扫描 `raw/` 中未被编译的文件
2. 为每个素材在 `wiki/summaries/` 创建摘要页
3. 更新或创建相关 `wiki/concepts/`、`wiki/entities/` 页面
4. 标注与现有页面的矛盾
5. 更新 `wiki/_Index_Wiki.md`
6. 输出变更报告

### 查询

直接向 agent 提问。优先检索 `wiki/`，未覆盖时回退到 `raw/`，并给出双链引用。

### 审查

定期对 agent 说：

```
review
```

agent 会报告：孤立页面、停滞的 seedling、未解决矛盾、未编译的 raw 素材。

### 审校

打开 agent 生成的 wiki 页面，检查：

- 提炼是否准确（有没有编造 raw 中不存在的内容）
- 双链是否指向真实存在的页面
- frontmatter 是否完整
- 是否有该合并的重复页

审校意见可以回写进 `AGENTS.md`，让 agent 下一次做得更好。

---
## 页面规范

每个 wiki 页面带 frontmatter：

---
type: concept | entity | summary | comparison | synthesis
status: seedling | budding | evergreen
created: YYYY-MM-DD
updated: YYYY-MM-DD
sources:
  - raw/College_Math/xxx.md
tags:
  - math
---

**成熟度**：

- `seedling` — 只有定义和零散要点，链接 < 2
- `budding` — 有结构化解释，链接 2-5
- `evergreen` — 完整、交叉验证、链接 ≥ 5，可独立阅读

**命名**：中文为主，专有名词保留英文。不用空格和特殊符号。

---
## 使用方式

### Obsidian

用 Obsidian 打开本仓库根目录作为 vault。

推荐插件：

- **Smart Connections** — 被动语义召回，写笔记时自动推送相关页面
- **Copilot for Obsidian** — vault 级对话与语义搜索

### AI agent

用 dsh（或 Claude Code / Codex）打开仓库根目录。agent 启动后必须先读 `AGENTS.md`，然后按权限操作：

- `raw/` 只读
- `wiki/` 可读写
- `areas/` 只读，可写草稿到 `_drafts/`

---
## 权限约定

| 目录 | agent 权限 |
|---|---|
| `raw/` | 只读，禁止修改、移动、删除 |
| `wiki/` | 读写，agent 拥有 |
| `areas/` | 只读，可写 `_drafts/` |
| `_Index_All.md` | 只读 |
| 根目录历史学科文件夹 | 只读 |

越权操作视为错误。不确定时先询问。

---
## 参考

- [Andrej Karpathy 关于 LLM Wiki 的论述](https://github.com/karpathy)
- [obsidian-llm-wiki-kit](https://github.com/) — 开箱即用的 LLM Wiki 模板
- [NOUZ](https://github.com/) — Obsidian 语义 MCP 服务器
- [Tiago Forte, Building a Second Brain](https://www.buildingasecondbrain.com/) — PARA 与 CODE

---
## 状态

早期搭建阶段。核心规则见 `AGENTS.md`。