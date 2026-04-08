# MemPalace

<div align="center">

<img src="https://github.com/milla-jovovich/mempalace/raw/main/assets/mempalace_logo.png" alt="MemPalace" width="280">

### 史上最高分的 AI 记忆系统基准测试。而且是免费的。

</div>

<br>

每次与 AI 的对话——每一个决策、每一次调试会话、每一次架构辩论——都会在会话结束时消失。六个月的工作，白费了。每次都重新开始。

其他记忆系统试图通过让 AI 决定什么值得记住来解决这个问题。它提取"用户偏好 Postgres"，却抛弃了你解释*为什么*的对话。MemPalace 采用了不同的方法：**存储一切，然后让它可查找。**

**宫殿（The Palace）** — 古希腊演说家通过将观点放置在虚构建筑的房间里来记忆整篇演讲。在建筑中行走，找到观点。MemPalace 将同样的原理应用于 AI 记忆：你的对话被组织成侧厅（人物和项目）、大厅（记忆类型）和房间（具体想法）。没有 AI 决定什么重要——你保留每个词，结构给你一张可导航的地图，而不是扁平的搜索索引。

**原始逐字存储** — MemPalace 将你的实际对话存储在 ChromaDB 中，不进行摘要或提取。96.6% 的 LongMemEval 结果来自这种原始模式。我们不会燃烧 LLM 来决定什么"值得记住"——我们保留一切，让语义搜索找到它。

**AAAK（实验性）** — 一种有损的缩写方言，用于在规模化时将重复实体压缩到更少的 token 中。可被任何读取文本的 LLM 读取——Claude、GPT、Gemini、Llama、Mistral——无需解码器。**AAAK 是一个独立的压缩层，不是存储默认值**，在 LongMemEval 基准测试中，目前它相比原始模式有所退化（84.2% vs 96.6%）。我们正在迭代。请参阅上面的[说明](#来自-milla--ben-的说明--2026年4月7日)。

**本地、开源、可适配** — MemPalace 完全在你的机器上运行，在本地任何数据上运行，不使用任何外部 API 或服务。它已经过对话测试——但可以适配不同类型的数据存储。这就是我们开源它的原因。

<br>

[![][version-shield]][release-link]
[![][python-shield]][python-link]
[![][license-shield]][license-link]
[![][discord-shield]][discord-link]

<br>

[快速开始](#快速开始) · [宫殿](#宫殿) · [AAAK 方言](#aaak-方言-实验性) · [基准测试](#基准测试) · [MCP 工具](#mcp-服务器)

<br>

### 史上最高 LongMemEval 分数——无论免费还是付费。

<table>
<tr>
<td align="center"><strong>96.6%</strong><br><sub>LongMemEval R@5<br><b>原始模式</b>，零 API 调用</sub></td>
<td align="center"><strong>500/500</strong><br><sub>测试问题数<br>独立复现</sub></td>
<td align="center"><strong>$0</strong><br><sub>无订阅<br>无云。仅本地。</sub></td>
</tr>
</table>

<sub>可复现——运行器在 <a href="https://github.com/milla-jovovich/mempalace/tree/main/benchmarks/">benchmarks/</a>。<a href="https://github.com/milla-jovovich/mempalace/blob/main/benchmarks/BENCHMARKS.md">完整结果</a>。96.6% 来自<b>原始逐字模式</b>，不是 AAAK 或房间模式（那些分数较低——见<a href="#来自-milla--ben-的说明--2026年4月7日">上方说明</a>）。</sub>

---

## 来自 Milla & Ben 的说明 — 2026年4月7日

> 社区在发布后几小时内就发现了这个 README 中的实际问题，我们希望直接回应。
>
> **我们错了的地方：**
>
> - **AAAK token 示例不正确。** 我们使用粗略的启发式方法（`len(text)//3`）来计算 token 数量，而不是使用实际的 tokenizer。通过 OpenAI 的 tokenizer 的真实计数：英文示例是 66 个 token，AAAK 示例是 73 个。AAAK 在小规模下不能节省 token——它是为*规模化重复实体*设计的，README 示例是错误的演示。我们正在重写。
>
> - **"30倍无损压缩"被夸大了。** AAAK 是一个有损的缩写系统（实体代码、句子截断）。独立基准测试显示 AAAK 模式在 LongMemEval 上得分 **84.2% R@5 vs 原始模式的 96.6%**——下降了 12.4 个百分点。诚实的表述是：AAAK 是一个实验性压缩层，以保真度换取 token 密度，**96.6% 的头条数字来自原始模式，不是 AAAK**。
>
> - **"+34% 宫殿提升"具有误导性。** 这个数字比较了未过滤搜索与 wing+room 元数据过滤。元数据过滤是 ChromaDB 的标准功能，不是新颖的检索机制。真实有用，但不是护城河。
>
> - **"矛盾检测"** 作为一个独立工具（`fact_checker.py`）存在，但目前没有像 README 暗示的那样接入知识图谱操作。
>
> - **"使用 Haiku 重新排序后 100%"** 是真实的（我们有结果文件），但重新排序管道不在公共基准测试脚本中。我们正在添加。
>
> **仍然正确且可复现的：**
>
> - **原始模式下 LongMemEval 的 96.6% R@5**，500 个问题，零 API 调用——由 [@gizmax](https://github.com/milla-jovovich/mempalace/issues/39) 在 M2 Ultra 上在 5 分钟内独立复现。
> - 本地、免费、无订阅、无云、无数据离开你的机器。
> - 架构（wings、rooms、closets、drawers）是真实有用的，即使它不是神奇的检索提升。
>
> **我们正在做的：**
>
> 1. 使用真实 tokenizer 计数重写 AAAK 示例，并展示 AAAK 真正能展示压缩的场景
> 2. 在基准测试文档中清楚添加 `mode raw / aaak / rooms`，使权衡可见
> 3. 将 `fact_checker.py` 接入 KG 操作，使矛盾检测声明变为现实
> 4. 将 ChromaDB 固定到经过测试的范围（[Issue #100](https://github.com/milla-jovovich/mempalace/issues/100)），修复 hooks 中的 shell 注入（[#110](https://github.com/milla-jovovich/mempalace/issues/110)），解决 macOS ARM64 段错误（[#74](https://github.com/milla-jovovich/mempalace/issues/74)）
>
> **感谢每一位为此挑刺的人。** 残酷的诚实批评正是开源工作的关键，这也是我们要求的。特别感谢 [@panuhorsmalahti](https://github.com/milla-jovovich/mempalace/issues/43)、[@lhl](https://github.com/milla-jovovich/mempalace/issues/27)、[@gizmax](https://github.com/milla-jovovich/mempalace/issues/39)，以及在过去 48 小时内提交 issue 或 PR 的每个人。我们在倾听，我们在修复，我们宁愿正确而不是令人印象深刻。
>
> — *Milla Jovovich & Ben Sigman*

---

## 快速开始

```bash
pip install mempalace

# 设置你的世界——你和谁工作、你的项目是什么
mempalace init ~/projects/myapp

# 挖掘你的数据
mempalace mine ~/projects/myapp                    # 项目 — 代码、文档、笔记
mempalace mine ~/chats/ --mode convos              # 对话 — Claude、ChatGPT、Slack 导出
mempalace mine ~/chats/ --mode convos --extract general  # 一般 — 分类为决策、里程碑、问题

# 搜索你曾经讨论过的任何内容
mempalace search "为什么我们切换到 GraphQL"

# 你的 AI 记住了
mempalace status
```

三种挖掘模式：**projects**（代码和文档）、**convos**（对话导出）和 **general**（自动分类为决策、偏好、里程碑、问题和情感上下文）。一切都留在你的机器上。

---

## 本地开发说明

### 项目结构

```
mempalace/
├── mempalace/                 # 核心包
│   ├── cli.py                 # CLI 入口点
│   ├── mcp_server.py          # MCP 服务器（19个工具）
│   ├── knowledge_graph.py     # 时序实体关系图
│   ├── palace_graph.py        # 房间导航图
│   ├── dialect.py             # AAAK 压缩
│   ├── miner.py               # 项目文件摄取
│   ├── convo_miner.py         # 对话摄取
│   ├── searcher.py            # 语义搜索
│   ├── onboarding.py          # 引导设置
│   └── ...
├── benchmarks/                # 可复现的基准测试运行器
│   ├── README.md              # 复现指南
│   ├── BENCHMARKS.md          # 完整结果和方法论
│   ├── longmemeval_bench.py   # LongMemEval 运行器
│   ├── locomo_bench.py        # LoCoMo 运行器
│   └── membench_bench.py      # MemBench 运行器
├── hooks/                     # Claude Code 自动保存钩子
│   ├── README.md              # 钩子设置指南
│   ├── mempal_save_hook.sh    # 每 N 条消息保存
│   └── mempal_precompact_hook.sh # 压缩前紧急保存
├── examples/                  # 使用示例
│   ├── basic_mining.py
│   ├── convo_import.py
│   └── mcp_setup.md
├── tests/                     # 测试套件
├── assets/                    # logo 和品牌资产
└── pyproject.toml             # 包配置 (v3.0.0)
```

### 系统要求

- Python 3.9+
- `chromadb>=0.4.0`
- `pyyaml>=6.0`

### 本地运行

项目完全在本地运行，不需要：
- API 密钥
- 安装后的互联网连接
- 云服务

```bash
pip install mempalace
```

### 数据存储位置

默认情况下，MemPalace 将数据存储在 `~/.mempalace/` 目录下：

```
~/.mempalace/
├── config.json          # 全局配置
├── wing_config.json     # 侧厅配置（人物和项目映射）
├── identity.txt         # 身份信息（L0 层）
├── palace/              # ChromaDB 数据库
│   └── ...
└── agents/              # 专家代理配置
    └── ...
```

---

## 你实际如何使用它

完成一次性设置（安装 → init → mine）后，你不手动运行 MemPalace 命令。你的 AI 为你使用它。有两种方式，取决于你使用哪个 AI。

### 使用 Claude、ChatGPT、Cursor、Gemini（MCP 兼容工具）

```bash
# 连接 MemPalace 一次
claude mcp add mempalace -- python -m mempalace.mcp_server
```

现在你的 AI 通过 MCP 拥有 19 个可用工具。问它任何问题：

> *"上个月我们关于 auth 做了什么决定？"*

Claude 自动调用 `mempalace_search`，获取逐字结果，并回答你。你再也不需要输入 `mempalace search` 了。AI 处理一切。

MemPalace 还可以原生与 **Gemini CLI** 配合使用（它自动处理服务器和保存钩子）——请参阅 [Gemini CLI 集成指南](https://github.com/milla-jovovich/mempalace/blob/main/examples/gemini_cli_setup.md)。

### 使用本地模型（Llama、Mistral 或任何离线 LLM）

本地模型通常还不支持 MCP。有两种方法：

**1. 唤醒命令** — 将你的世界加载到模型的上下文中：

```bash
mempalace wake-up > context.txt
# 将 context.txt 粘贴到你的本地模型的系统提示中
```

这给你的本地模型 ~170 个关键事实的 token（在 AAAK 中如果你喜欢的话），然后你才问第一个问题。

**2. CLI 搜索** — 按需查询，将结果输入到你的提示中：

```bash
mempalace search "auth decisions" > results.txt
# 在你的提示中包含 results.txt
```

或使用 Python API：

```python
from mempalace.searcher import search_memories
results = search_memories("auth decisions", palace_path="~/.mempalace/palace")
# 注入到你的本地模型的上下文中
```

无论哪种方式——你的整个记忆栈都离线运行。ChromaDB 在你的机器上，Llama 在你的机器上，AAAK 用于压缩，零云调用。

---

## 问题

决策现在发生在对话中。不在文档中。不在 Jira 中。在与 Claude、ChatGPT、Copilot 的对话中。推理、权衡、"我们尝试了 X 但因为 Y 失败了"——都困在会话结束时消失的聊天窗口中。

**六个月的日常 AI 使用 = 1950 万个 token。** 这就是每一个决策、每一次调试会话、每一次架构辩论。消失了。

| 方法 | 加载的 token | 年度成本 |
|----------|--------------|-------------|
| 粘贴一切 | 1950万 — 超出任何上下文窗口 | 不可能 |
| LLM 摘要 | ~65万 | ~$507/年 |
| **MemPalace wake-up** | **~170 token** | **~$0.70/年** |
| **MemPalace + 5 次搜索** | **~13,500 token** | **~$10/年** |

MemPalace 在唤醒时加载 170 个关键事实 token——你的团队、你的项目、你的偏好。然后只在需要时搜索。每年 $10 记住一切 vs $507/年 丢失上下文的摘要。

---

## 工作原理

### 宫殿

布局相当简单，尽管花了很长时间才走到这一步。

它从**侧厅（wing）**开始。每个你正在归档的项目、人或主题都在宫殿中获得自己的侧厅。

每个侧厅都有连接到它的**房间**，信息被分成与该侧厅相关的主题——所以每个房间都是你的项目包含的不同元素。项目想法可能是一个房间，员工可能是另一个，财务报表是另一个。可以有无限数量的房间将侧厅分成多个部分。MemPalace 安装会自动为你检测这些，当然你可以用任何你认为合适的方式进行个性化。

每个房间都有一个连接到它的**壁橱（closet）**，这就是事情变得有趣的地方。我们开发了一种名为 **AAAK** 的 AI 语言。不要问——这是一个完整的故事。你的代理每次唤醒时都会学习 AAAK 速记。因为 AAAK 本质上是英语，但是一个非常截断的版本，你的代理在几秒钟内就知道如何使用它。它作为安装的一部分出现，内置在 MemPalace 代码中。在我们的下一次更新中，我们将直接把 AAAK 添加到壁橱中，这将是一个真正的游戏改变者——壁橱中的信息量会更大，但占用空间更少，你的代理阅读时间也更短。

在那些壁橱里面是**抽屉（drawer）**，那些抽屉是你的原始文件所在的地方。在这个第一个版本中，我们没有使用 AAAK 作为壁橱工具，但即便如此，摘要在我们跨多个基准测试平台进行的所有基准测试中显示了 **96.6% 的召回率**。一旦壁橱使用 AAAK，搜索会更快，同时保留每个单词。但即使现在，壁橱方法对存储在较小空间中的信息量也是一个巨大的推动——它用于轻松地将你的 AI 代理指向你的原始文件所在的抽屉。你不会丢失任何东西，所有这些都在几秒钟内发生。

还有**大厅（hall）**，连接侧厅内的房间，还有**隧道（tunnel）**，连接不同侧厅的房间。所以找到东西变得真正毫不费力——我们给 AI 一个清晰有序的方式，知道从哪里开始搜索，而不必在巨大的文件夹中查找每个关键词。

你说你在找什么，然后哇，它已经知道去哪个侧厅了。就*那本身*就会产生很大的不同。但这是美丽的、优雅的、有机的，最重要的是，高效的。

```
  ┌─────────────────────────────────────────────────────────────┐
  │  WING: Person                                              │
  │                                                            │
  │    ┌──────────┐  ──hall──  ┌──────────┐                    │
  │    │  Room A  │            │  Room B  │                    │
  │    └────┬─────┘            └──────────┘                    │
  │         │                                                  │
  │         ▼                                                  │
  │    ┌──────────┐      ┌──────────┐                          │
  │    │  Closet  │ ───▶ │  Drawer  │                          │
  │    └──────────┘      └──────────┘                          │
  └─────────┼──────────────────────────────────────────────────┘
            │
          tunnel
            │
  ┌─────────┼──────────────────────────────────────────────────┐
  │  WING: Project                                             │
  │         │                                                  │
  │    ┌────┴─────┐  ──hall──  ┌──────────┐                    │
  │    │  Room A  │            │  Room C  │                    │
  │    └────┬─────┘            └──────────┘                    │
  │         │                                                  │
  │         ▼                                                  │
  │    ┌──────────┐      ┌──────────┐                          │
  │    │  Closet  │ ───▶ │  Drawer  │                          │
  │    └──────────┘      └──────────┘                          │
  └─────────────────────────────────────────────────────────────┘
```

**侧厅（Wings）** — 一个人或一个项目。需要多少就有多少。
**房间（Rooms）** — 侧厅内的特定主题。Auth、计费、部署——无尽的房间。
**大厅（Halls）** — *在同一侧厅内*相关房间之间的连接。如果房间 A（auth）和房间 B（security）相关，大厅将它们链接。
**隧道（Tunnels）** — *侧厅之间*的连接。当人物 A 和项目都有关于"auth"的房间时，隧道自动交叉引用它们。
**壁橱（Closets）** — 指向原始内容的摘要。（在 v3.0.0 中这些是纯文本摘要；AAAK 编码的壁橱将在未来更新中添加——见 [Task #30](https://github.com/milla-jovovich/mempalace/issues/30)。）
**抽屉（Drawers）** — 原始逐字文件。精确的词语，从不摘要。

**大厅（Halls）** 是记忆类型——在每个侧厅中相同，充当走廊：
- `hall_facts` — 做出的决策，锁定的选择
- `hall_events` — 会话、里程碑、调试
- `hall_discoveries` — 突破、新见解
- `hall_preferences` — 习惯、喜好、意见
- `hall_advice` — 建议和解决方案

**房间（Rooms）** 是有名字的想法——`auth-migration`、`graphql-switch`、`ci-pipeline`。当同一个房间出现在不同的侧厅时，它会创建一个**隧道**——跨领域连接相同的主题：

```
wing_kai       / hall_events / auth-migration  → "Kai debugged the OAuth token refresh"
wing_driftwood / hall_facts  / auth-migration  → "team decided to migrate auth to Clerk"
wing_priya     / hall_advice / auth-migration  → "Priya approved Clerk over Auth0"
```

同一个房间。三个侧厅。隧道连接它们。

### 为什么结构重要

在 22,000+ 个真实对话记忆上测试：

```
搜索所有壁橱:          60.9%  R@10
在侧厅内搜索:          73.1%  (+12%)
搜索侧厅 + 大厅:       84.8%  (+24%)
搜索侧厅 + 房间:       94.8%  (+34%)
```

侧厅和房间不是装饰性的。它们是 **34% 的检索改进**。宫殿结构就是产品。

### 记忆栈

| 层级 | 内容 | 大小 | 时机 |
|-------|------|------|------|
| **L0** | 身份 — 这是什么 AI？ | ~50 token | 始终加载 |
| **L1** | 关键事实 — 团队、项目、偏好 | ~120 token (AAAK) | 始终加载 |
| **L2** | 房间回忆 — 最近的会话、当前项目 | 按需 | 话题出现时 |
| **L3** | 深度搜索 — 跨所有壁橱的语义查询 | 按需 | 明确询问时 |

你的 AI 用 L0 + L1（~170 token）唤醒，知道你的世界。搜索只在需要时触发。

### AAAK 方言（实验性）

AAAK 是一个有损的缩写系统——实体代码、结构标记和句子截断——旨在在规模化时将重复实体和关系打包到更少的 token 中。它**可被任何读取文本的 LLM 读取**（Claude、GPT、Gemini、Llama、Mistral）无需解码器，因此本地模型可以使用它而无需任何云依赖。

**诚实的状态（2026年4月）：**

- **AAAK 是有损的，不是无损的。** 它使用基于正则表达式的缩写，不是可逆压缩。
- **在小规模下不能节省 token。** 短文本已经高效地进行 token 化。AAAK 开销（代码、分隔符）比在几个句子上的节省更多。
- **在规模化时可以节省 token** — 在有许多重复实体的场景中（一个团队被提及数百次，相同的项目跨数千次会话），实体代码可以分摊。
- **AAAK 目前在 LongMemEval 上退化** vs 原始逐字检索（84.2% R@5 vs 96.6%）。96.6% 的头条数字来自**原始模式**，不是 AAAK 模式。
- **MemPalace 存储默认值是 ChromaDB 中的原始逐字文本**——基准测试的胜利来自这里。AAAK 是用于上下文加载的独立压缩层，不是存储格式。

我们正在迭代方言规范，添加真实的 tokenizer 用于统计，并探索更好的断点来决定何时使用它。在 [Issue #43](https://github.com/milla-jovovich/mempalace/issues/43) 和 [#27](https://github.com/milla-jovovich/mempalace/issues/27) 中跟踪进度。

### 矛盾检测（实验性，尚未接入 KG）

一个独立的工具（`fact_checker.py`）可以检查针对实体事实的断言。它目前没有被知识图谱操作自动调用——这正在修复中（跟踪在 [Issue #27](https://github.com/milla-jovovich/mempalace/issues/27)）。启用后，它会捕获如下内容：

```
输入:  "Soren finished the auth migration"
输出: 🔴 AUTH-MIGRATION: attribution conflict — Maya was assigned, not Soren

输入:  "Kai has been here 2 years"
输出: 🟡 KAI: wrong_tenure — records show 3 years (started 2023-04)

输入:  "The sprint ends Friday"
输出: 🟡 SPRINT: stale_date — current sprint ends Thursday (updated 2 days ago)
```

事实根据知识图谱检查。年龄、日期和任期动态计算——不是硬编码的。

---

## 实际例子

### 跨多个项目的独立开发者

```bash
# 挖掘每个项目的对话
mempalace mine ~/chats/orion/  --mode convos --wing orion
mempalace mine ~/chats/nova/   --mode convos --wing nova
mempalace mine ~/chats/helios/ --mode convos --wing helios

# 六个月后: "为什么我在这里使用 Postgres？"
mempalace search "database decision" --wing orion
# → "Chose Postgres over SQLite because Orion needs concurrent writes
#    and the dataset will exceed 10GB. Decided 2025-11-03."

# 跨项目搜索
mempalace search "rate limiting approach"
# → 在 Orion 和 Nova 中找到你的方法，显示差异
```

### 管理产品的团队负责人

```bash
# 挖掘 Slack 导出和 AI 对话
mempalace mine ~/exports/slack/ --mode convos --wing driftwood
mempalace mine ~/.claude/projects/ --mode convos

# "Soren 上个 sprint 做了什么？"
mempalace search "Soren sprint" --wing driftwood
# → 14 个壁橱: OAuth 重构、深色模式、组件库迁移

# "谁决定使用 Clerk？"
mempalace search "Clerk decision" --wing driftwood
# → "Kai recommended Clerk over Auth0 — pricing + developer experience.
#    Team agreed 2026-01-15. Maya handling the migration."
```

### 挖掘前：拆分超大文件

一些转录导出将多个会话合并成一个巨大的文件：

```bash
mempalace split ~/chats/                      # 拆分为每个会话文件
mempalace split ~/chats/ --dry-run            # 首先预览
mempalace split ~/chats/ --min-sessions 3     # 仅拆分有 3+ 个会话的文件
```

---

## 知识图谱

时序实体关系三元组——像 Zep 的 Graphiti，但使用 SQLite 而不是 Neo4j。本地和免费。

```python
from mempalace.knowledge_graph import KnowledgeGraph

kg = KnowledgeGraph()
kg.add_triple("Kai", "works_on", "Orion", valid_from="2025-06-01")
kg.add_triple("Maya", "assigned_to", "auth-migration", valid_from="2026-01-15")
kg.add_triple("Maya", "completed", "auth-migration", valid_from="2026-02-01")

# Kai 在做什么？
kg.query_entity("Kai")
# → [Kai → works_on → Orion (current), Kai → recommended → Clerk (2026-01)]

# 一月份什么是真的？
kg.query_entity("Maya", as_of="2026-01-20")
# → [Maya → assigned_to → auth-migration (active)]

# 时间线
kg.timeline("Orion")
# → 项目的按时间顺序的故事
```

事实有有效窗口。当某事停止为真时，使其失效：

```python
kg.invalidate("Kai", "works_on", "Orion", ended="2026-03-01")
```

现在对 Kai 当前工作的查询不会返回 Orion。历史查询仍然会。

| 功能 | MemPalace | Zep (Graphiti) |
|---------|-----------|----------------|
| 存储 | SQLite (本地) | Neo4j (云) |
| 成本 | 免费 | $25/月+ |
| 时序有效性 | 是 | 是 |
| 自托管 | 始终 | 仅企业版 |
| 隐私 | 一切本地 | SOC 2, HIPAA |

---

## 专家代理

创建专注于特定领域的代理。每个代理在宫殿中获得自己的侧厅和日记——不在你的 CLAUDE.md 中。添加 50 个代理，你的配置大小保持不变。

```
~/.mempalace/agents/
  ├── reviewer.json       # 代码质量、模式、bug
  ├── architect.json      # 设计决策、权衡
  └── ops.json            # 部署、事件、基础设施
```

你的 CLAUDE.md 只需要一行：

```
You have MemPalace agents. Run mempalace_list_agents to see them.
```

AI 在运行时从宫殿中发现其代理。每个代理：

- **有焦点** — 它关注什么
- **保持日记** — 用 AAAK 书写，跨会话持久化
- **建立专业知识** — 阅读自己的历史以保持其领域的敏锐

```
# 代理在代码审查后写入日记
mempalace_diary_write("reviewer",
    "PR#42|auth.bypass.found|missing.middleware.check|pattern:3rd.time.this.quarter|★★★★")

# 代理读回其历史
mempalace_diary_read("reviewer", last_n=10)
# → 最后 10 个发现，用 AAAK 压缩
```

每个代理都是你数据的一个专业镜头。审查者记住它见过的每个 bug 模式。架构师记住每个设计决策。运维代理记住每个事件。它们不共享草稿纸——它们各自维护自己的记忆。

Letta 为代理管理的记忆收取 $20-200/月。MemPalace 用一个侧厅完成同样的事。

---

## MCP 服务器

```bash
claude mcp add mempalace -- python -m mempalace.mcp_server
```

### 19 个工具

**宫殿（读取）**

| 工具 | 功能 |
|------|------|
| `mempalace_status` | 宫殿概览 + AAAK 规范 + 记忆协议 |
| `mempalace_list_wings` | 带计数的侧厅列表 |
| `mempalace_list_rooms` | 侧厅内的房间 |
| `mempalace_get_taxonomy` | 完整的 wing → room → count 树 |
| `mempalace_search` | 带 wing/room 过滤器的语义搜索 |
| `mempalace_check_duplicate` | 归档前检查 |
| `mempalace_get_aaak_spec` | AAAK 方言参考 |

**宫殿（写入）**

| 工具 | 功能 |
|------|------|
| `mempalace_add_drawer` | 文件逐字内容 |
| `mempalace_delete_drawer` | 按 ID 删除 |

**知识图谱**

| 工具 | 功能 |
|------|------|
| `mempalace_kg_query` | 带时间过滤的实体关系 |
| `mempalace_kg_add` | 添加事实 |
| `mempalace_kg_invalidate` | 标记事实已结束 |
| `mempalace_kg_timeline` | 按时间顺序的实体故事 |
| `mempalace_kg_stats` | 图谱概览 |

**导航**

| 工具 | 功能 |
|------|------|
| `mempalace_traverse` | 从一个房间跨侧厅行走图 |
| `mempalace_find_tunnels` | 找到连接两个侧厅的房间 |
| `mempalace_graph_stats` | 图连接概览 |

**代理日记**

| 工具 | 功能 |
|------|------|
| `mempalace_diary_write` | 写 AAAK 日记条目 |
| `mempalace_diary_read` | 读最近的日记条目 |

AI 从 `mempalace_status` 响应中自动学习 AAAK 和记忆协议。无需手动配置。

---

## 自动保存钩子

两个用于 Claude Code 的钩子，在工作期间自动保存记忆：

**保存钩子** — 每 15 条消息触发一次结构化保存。主题、决策、引用、代码更改。还重新生成关键事实层。

**预压缩钩子** — 在上下文压缩前触发。窗口收缩前的紧急保存。

```json
{
  "hooks": {
    "Stop": [{"matcher": "", "hooks": [{"type": "command", "command": "/path/to/mempalace/hooks/mempal_save_hook.sh"}]}],
    "PreCompact": [{"matcher": "", "hooks": [{"type": "command", "command": "/path/to/mempalace/hooks/mempal_precompact_hook.sh"}]}]
  }
}
```

---

## 基准测试

在标准学术基准上测试——可复现、发布的数据集。

| 基准 | 模式 | 分数 | API 调用 |
|-----------|------|-------|-----------|
| **LongMemEval R@5** | Raw (仅 ChromaDB) | **96.6%** | 零 |
| **LongMemEval R@5** | Hybrid + Haiku 重新排序 | **100%** (500/500) | ~500 |
| **LoCoMo R@10** | Raw, session level | **60.3%** | 零 |
| **Personal palace R@10** | Heuristic bench | **85%** | 零 |
| **Palace structure impact** | Wing+room 过滤 | **+34%** R@10 | 零 |

96.6% 原始分数是已发布的最高 LongMemEval 结果，不需要 API 密钥、不需要云、不需要任何阶段的 LLM。

### vs 已发布系统

| 系统 | LongMemEval R@5 | 需要 API | 成本 |
|--------|----------------|--------------|------|
| **MemPalace (hybrid)** | **100%** | 可选 | 免费 |
| Supermemory ASMR | ~99% | 是 | — |
| **MemPalace (raw)** | **96.6%** | **无** | **免费** |
| Mastra | 94.87% | 是 (GPT) | API 成本 |
| Mem0 | ~85% | 是 | $19–249/月 |
| Zep | ~85% | 是 | $25/月+ |

---

## 所有命令

```bash
# 设置
mempalace init <dir>                              # 引导入职 + AAAK 引导

# 挖掘
mempalace mine <dir>                              # 挖掘项目文件
mempalace mine <dir> --mode convos                # 挖掘对话导出
mempalace mine <dir> --mode convos --wing myapp   # 用侧厅名称标记

# 拆分
mempalace split <dir>                             # 拆分连接的转录
mempalace split <dir> --dry-run                   # 预览

# 搜索
mempalace search "query"                          # 搜索一切
mempalace search "query" --wing myapp             # 在侧厅内
mempalace search "query" --room auth-migration    # 在房间内

# 记忆栈
mempalace wake-up                                 # 加载 L0 + L1 上下文
mempalace wake-up --wing driftwood                # 项目特定

# 压缩
mempalace compress --wing myapp                   # AAAK 压缩

# 状态
mempalace status                                  # 宫殿概览
```

所有命令都接受 `--palace <path>` 来覆盖默认位置。

---

## 配置

### 全局（`~/.mempalace/config.json`）

```json
{
  "palace_path": "/custom/path/to/palace",
  "collection_name": "mempalace_drawers",
  "people_map": {"Kai": "KAI", "Priya": "PRI"}
}
```

### 侧厅配置（`~/.mempalace/wing_config.json`）

由 `mempalace init` 生成。将你的个人和项目映射到侧厅：

```json
{
  "default_wing": "wing_general",
  "wings": {
    "wing_kai": {"type": "person", "keywords": ["kai", "kai's"]},
    "wing_driftwood": {"type": "project", "keywords": ["driftwood", "analytics", "saas"]}
  }
}
```

### 身份（`~/.mempalace/identity.txt`）

纯文本。成为第 0 层——每个会话加载。

---

## 文件参考

| 文件 | 功能 |
|------|------|
| `cli.py` | CLI 入口点 |
| `config.py` | 配置加载和默认值 |
| `normalize.py` | 将 5 种聊天格式转换为标准转录 |
| `mcp_server.py` | MCP 服务器 — 19 个工具，AAAK 自动教学，记忆协议 |
| `miner.py` | 项目文件摄取 |
| `convo_miner.py` | 对话摄取 — 按交换对分块 |
| `searcher.py` | 通过 ChromaDB 进行语义搜索 |
| `layers.py` | 4 层记忆栈 |
| `dialect.py` | AAAK 压缩 — 30倍无损 |
| `knowledge_graph.py` | 时序实体关系图（SQLite） |
| `palace_graph.py` | 基于房间的导航图 |
| `onboarding.py` | 引导设置 — 生成 AAAK 引导 + 侧厅配置 |
| `entity_registry.py` | 实体代码注册表 |
| `entity_detector.py` | 从内容中自动检测人和项目 |
| `split_mega_files.py` | 将连接的转录拆分为每个会话文件 |
| `hooks/mempal_save_hook.sh` | 每 N 条消息自动保存 |
| `hooks/mempal_precompact_hook.sh` | 压缩前紧急保存 |

---

## 项目结构

```
mempalace/
├── README.md                  ← 你在这里
├── mempalace/                 ← 核心包 (README)
│   ├── cli.py                 ← CLI 入口点
│   ├── mcp_server.py          ← MCP 服务器 (19 个工具)
│   ├── knowledge_graph.py     ← 时序实体图
│   ├── palace_graph.py        ← 房间导航图
│   ├── dialect.py             ← AAAK 压缩
│   ├── miner.py               ← 项目文件摄取
│   ├── convo_miner.py         ← 对话摄取
│   ├── searcher.py            ← 语义搜索
│   ├── onboarding.py          ← 引导设置
│   └── ...                    ← 见 <a href="https://github.com/milla-jovovich/mempalace/blob/main/mempalace/README.md">mempalace/README.md</a>
├── benchmarks/                ← 可复现的基准测试运行器
│   ├── README.md              ← 复现指南
│   ├── BENCHMARKS.md          ← 完整结果 + 方法论
│   ├── longmemeval_bench.py   ← LongMemEval 运行器
│   ├── locomo_bench.py        ← LoCoMo 运行器
│   └── membench_bench.py      ← MemBench 运行器
├── hooks/                     ← Claude Code 自动保存钩子
│   ├── README.md              ← 钩子设置指南
│   ├── mempal_save_hook.sh    ← 每 N 条消息保存
│   └── mempal_precompact_hook.sh ← 紧急保存
├── examples/                  ← 使用示例
│   ├── basic_mining.py
│   ├── convo_import.py
│   └── mcp_setup.md
├── tests/                     ← 测试套件 (README)
├── assets/                    ← logo + 品牌资产
└── pyproject.toml             ← 包配置 (v3.0.0)
```

---

## 要求

- Python 3.9+
- `chromadb>=0.4.0`
- `pyyaml>=6.0`

无需 API 密钥。安装后无需互联网。一切本地。

```bash
pip install mempalace
```

---

## 贡献

欢迎 PR。参见 [CONTRIBUTING.md](https://github.com/milla-jovovich/mempalace/blob/main/CONTRIBUTING.md) 了解设置和指南。

## 许可证

MIT — 参见 [LICENSE](https://github.com/milla-jovovich/mempalace/blob/main/LICENSE)。

<!-- Link Definitions -->
[version-shield]: https://img.shields.io/badge/version-3.0.0-4dc9f6?style=flat-square&labelColor=0a0e14
[release-link]: https://github.com/milla-jovovich/mempalace/releases
[python-shield]: https://img.shields.io/badge/python-3.9+-7dd8f8?style=flat-square&labelColor=0a0e14&logo=python&logoColor=7dd8f8
[python-link]: https://www.python.org/
[license-shield]: https://img.shields.io/badge/license-MIT-b0e8ff?style=flat-square&labelColor=0a0e14
[license-link]: https://github.com/milla-jovovich/mempalace/blob/main/LICENSE
[discord-shield]: https://img.shields.io/badge/discord-join-5865F2?style=flat-square&labelColor=0a0e14&logo=discord&logoColor=5865F2
[discord-link]: https://discord.com/invite/ycTQQCu6kn
