> **本文档汉化自 [github.com/milla-jovovich/mempalace](https://github.com/milla-jovovich/mempalace)，仅供中文用户参考。原始英文版以上游仓库为准。**

<div align="center">

<img src="https://raw.githubusercontent.com/milla-jovovich/mempalace/main/assets/mempalace_logo.png" alt="MemPalace" width="280">

# MemPalace

### 有史以来基准测试得分最高的 AI 记忆系统。而且免费。

<br>

你与 AI 的每一次对话——每一个决策、每一次调试、每一次架构讨论——都在会话结束时消失。六个月的工作，化为乌有。你每次都要从头开始。

其他记忆系统试图通过让 AI 决定什么值得记住来解决这个问题。它提取"用户偏好 Postgres"，却丢弃了你解释*为什么*的对话。MemPalace 采用了不同的方法：**存储一切，然后让它可查找。**

**宫殿（The Palace）**——古希腊演说家通过将想法放置在想象建筑的房间中来记住整篇演讲。走过建筑，找到想法。MemPalace 将同样的原则应用于 AI 记忆：你的对话被组织成翼（人与项目）、厅（记忆类型）和房间（具体想法）。没有 AI 来决定什么重要——你保留每一个字，结构为你提供可导航的地图，而不是扁平的搜索索引。

**原始逐字存储**——MemPalace 将你的实际对话存储在 ChromaDB 中，不进行摘要或提取。96.6% 的 LongMemEval 成绩正是来自这种原始模式。我们不会消耗 LLM 来决定什么"值得记住"——我们保留一切，让语义搜索来找到它。

**AAAK（实验性）**——一种有损缩写方言，用于在大规模场景下将重复实体压缩到更少的 token 中。任何能读取文本的 LLM 都能阅读——Claude、GPT、Gemini、Llama、Mistral——无需解码器。**AAAK 是一个独立的压缩层，不是存储默认值**，在 LongMemEval 基准测试中，它目前相比原始模式有所回退（84.2% vs 96.6%）。我们正在迭代中。请参阅[上方说明](#来自-milla--ben-的说明--2026-年-4-月-7-日)了解真实状态。

**本地、开放、可适配**——MemPalace 完全在你的机器上运行，使用你本地的任何数据，不使用任何外部 API 或服务。它已在对话上经过测试——但可以适配不同类型的数据存储。这就是我们开源它的原因。

<br>

[![][version-shield]][release-link]
[![][python-shield]][python-link]
[![][license-shield]][license-link]
[![][discord-shield]][discord-link]

<br>

[快速开始](#快速开始) · [宫殿](#宫殿) · [AAAK 方言](#aaak-压缩) · [基准测试](#基准测试) · [MCP 工具](#mcp-服务器)

<br>

### 有史以来发布的最高 LongMemEval 得分——无论免费还是付费。

<table>
<tr>
<td align="center"><strong>96.6%</strong><br><sub>LongMemEval R@5<br><b>原始模式</b>，零 API 调用</sub></td>
<td align="center"><strong>500/500</strong><br><sub>测试问题数<br>独立复现</sub></td>
<td align="center"><strong>$0</strong><br><sub>无需订阅<br>无需云端。仅本地。</sub></td>
</tr>
</table>

<sub>可复现——运行脚本在 <a href="https://github.com/milla-jovovich/mempalace/tree/main/benchmarks/">benchmarks/</a>。<a href="https://github.com/milla-jovovich/mempalace/blob/main/benchmarks/BENCHMARKS.md">完整结果</a>。96.6% 来自<b>原始逐字模式</b>，不是 AAAK 或房间模式（那些得分更低——参见<a href="#来自-milla--ben-的说明--2026-年-4-月-7-日">上方说明</a>）。</sub>

</div>

---

## 来自 Milla & Ben 的说明 — 2026 年 4 月 7 日

> 社区在发布后数小时内就发现了此 README 中的真实问题，我们想直接回应。
>
> **我们做错的地方：**
>
> - **AAAK token 示例不正确。** 我们使用了粗略的启发式方法（`len(text)//3`）来计算 token 数，而不是实际的分词器。通过 OpenAI 分词器的真实计数：英文示例为 66 个 token，AAAK 示例为 73 个。AAAK 在小规模下不节省 token——它是为*大规模重复实体*设计的，而 README 示例是对此的糟糕展示。我们正在重写它。
>
> - **"30 倍无损压缩"被夸大了。** AAAK 是一种有损缩写系统（实体编码、句子截断）。独立基准测试显示 AAAK 模式在 LongMemEval 上得分 **84.2% R@5 vs 原始模式的 96.6%**——回退了 12.4 个百分点。诚实的表述是：AAAK 是一个实验性压缩层，以保真度换取 token 密度，**96.6% 的标题数字来自 RAW 模式，不是 AAAK**。
>
> - **"+34% 宫殿提升"具有误导性。** 该数字比较的是未过滤搜索与 wing+room 元数据过滤。元数据过滤是 ChromaDB 的标准功能，不是新颖的检索机制。真实且有用，但不是护城河。
>
> - **"矛盾检测"** 作为一个独立的工具（`fact_checker.py`）存在，但目前并未如 README 所暗示的那样接入知识图谱操作。
>
> - **"使用 Haiku 重排序达到 100%"** 是真实的（我们有结果文件），但重排序管道不在公开的基准测试脚本中。我们正在添加。
>
> **仍然真实且可复现的：**
>
> - **在原始模式下 LongMemEval R@5 为 96.6%**，500 个问题，零 API 调用——由 [@gizmax](https://github.com/milla-jovovich/mempalace/issues/39) 在 M2 Ultra 上不到 5 分钟内独立复现。
> - 本地、免费、无需订阅、无需云端、数据不离开你的机器。
> - 架构（翼、房间、壁橱、抽屉）是真实且有用的，即使它不是神奇的检索提升。
>
> **我们正在做的：**
>
> 1. 使用真实分词器计数重写 AAAK 示例，并使用 AAAK 实际展示压缩的场景
> 2. 在基准测试文档中清楚地添加 `mode raw / aaak / rooms`，使权衡可见
> 3. 将 `fact_checker.py` 接入知识图谱操作，使矛盾检测的声明变为现实
> 4. 将 ChromaDB 固定到经过测试的版本范围（Issue #100），修复 hooks 中的 shell 注入（#110），并解决 macOS ARM64 段错误（#74）
>
> **感谢每一个质疑的人。** 残酷而诚实的批评正是让开源运作的方式，也是我们所要求的。特别感谢 [@panuhorsmalahti](https://github.com/milla-jovovich/mempalace/issues/43)、[@lhl](https://github.com/milla-jovovich/mempalace/issues/27)、[@gizmax](https://github.com/milla-jovovich/mempalace/issues/39)，以及在前 48 小时内提交 issue 或 PR 的所有人。我们在倾听，我们在修复，我们宁愿正确也不愿华而不实。
>
> — *Milla Jovovich & Ben Sigman*

---

## 快速开始

```bash
pip install mempalace

# 设置你的世界——你和谁一起工作，你的项目是什么
mempalace init ~/projects/myapp

# 挖掘你的数据
mempalace mine ~/projects/myapp                    # 项目——代码、文档、笔记
mempalace mine ~/chats/ --mode convos              # 对话——Claude、ChatGPT、Slack 导出
mempalace mine ~/chats/ --mode convos --extract general  # 通用——分类为决策、里程碑、问题

# 搜索你曾经讨论过的任何内容
mempalace search "为什么我们切换到 GraphQL"

# 你的 AI 记住了
mempalace status
```

三种挖掘模式：**项目**（代码和文档）、**对话**（对话导出）和**通用**（自动分类为决策、偏好、里程碑、问题和情感上下文）。一切都在你的机器上。

---

## 你实际如何使用它

一次性设置（安装 → 初始化 → 挖掘）后，你不需要手动运行 MemPalace 命令。你的 AI 会替你使用它。有两种方式，取决于你使用哪个 AI。

### 使用 Claude、ChatGPT、Cursor、Gemini（兼容 MCP 的工具）

```bash
# 一次性连接 MemPalace
claude mcp add mempalace -- python -m mempalace.mcp_server
```

现在你的 AI 通过 MCP 有 19 个工具可用。问它任何问题：

> *"上个月我们对认证做了什么决定？"*

Claude 自动调用 `mempalace_search`，获取逐字结果，并回答你。你再也不用输入 `mempalace search`。AI 处理一切。

MemPalace 也原生支持 **Gemini CLI**（它自动处理服务器和保存钩子）——参见 [Gemini CLI 集成指南](https://github.com/milla-jovovich/mempalace/blob/main/examples/gemini_cli_setup.md)。

### 使用本地模型（Llama、Mistral 或任何离线 LLM）

本地模型通常还不支持 MCP。两种方法：

**1. 唤醒命令**——将你的世界加载到模型的上下文中：

```bash
mempalace wake-up > context.txt
# 将 context.txt 粘贴到本地模型的系统提示中
```

这在你提出任何问题之前，就给你的本地模型提供了约 170 个 token 的关键事实（如果你偏好可以使用 AAAK）。

**2. CLI 搜索**——按需查询，将结果输入你的提示：

```bash
mempalace search "认证决策" > results.txt
# 在你的提示中包含 results.txt
```

或使用 Python API：

```python
from mempalace.searcher import search_memories
results = search_memories("认证决策", palace_path="~/.mempalace/palace")
# 注入到本地模型的上下文中
```

无论哪种方式——你的整个记忆栈都离线运行。ChromaDB 在你的机器上，Llama 在你的机器上，AAAK 用于压缩，零云端调用。

---

## 问题所在

决策现在发生在对话中。不在文档里。不在 Jira 里。在与 Claude、ChatGPT、Copilot 的对话中。推理、权衡、"我们尝试了 X 但因为 Y 失败了"——全都困在会话结束就消失的聊天窗口中。

**六个月的日常 AI 使用 = 1950 万 token。** 那是每一个决策、每一次调试、每一场架构讨论。化为乌有。

| 方法 | 加载的 token | 年度成本 |
|----------|--------------|-------------|
| 粘贴全部 | 1950 万——放不进任何上下文窗口 | 不可能 |
| LLM 摘要 | ~65 万 | ~$507/年 |
| **MemPalace 唤醒** | **~170 token** | **~$0.70/年** |
| **MemPalace + 5 次搜索** | **~13,500 token** | **~$10/年** |

MemPalace 在唤醒时加载 170 个 token 的关键事实——你的团队、你的项目、你的偏好。然后只在需要时搜索。$10/年记住一切 vs $507/年获取丢失上下文的摘要。

---

## 工作原理

### 宫殿

布局相当简单，尽管花了很多时间才达到这个效果。

它从一个**翼（wing）**开始。你归档的每个项目、人物或主题在宫殿中都有自己的翼。

每个翼连接着**房间（room）**，信息在其中按与该翼相关的主题划分——所以每个房间是你项目包含内容的不同元素。项目想法可以是一个房间，员工可以是另一个，财务报表又是一个。可以有无限数量的房间将翼分成不同部分。MemPalace 安装时会自动检测这些，当然你也可以按自己觉得合适的方式个性化定制。

每个房间连接着一个**壁橱（closet）**，这就是有趣的地方。我们开发了一种名为 **AAAK** 的 AI 语言。别问——这本身就是一个完整的故事。你的智能体每次唤醒时都会学习 AAAK 简写。因为 AAAK 本质上是英语，但是一个非常截断的版本，你的智能体在几秒钟内就能理解如何使用它。它作为安装的一部分，内置在 MemPalace 代码中。在下一次更新中，我们将把 AAAK 直接添加到壁橱中，这将是一个真正的游戏规则改变者——壁橱中的信息量会大得多，但占用的空间和智能体的阅读时间会少得多。

壁橱里面有**抽屉（drawer）**，那些抽屉就是你原始文件的存放处。在第一个版本中，我们没有将 AAAK 用作壁橱工具，但即便如此，摘要在我们跨多个基准测试平台所做的所有基准测试中显示了 **96.6% 的召回率**。一旦壁橱使用 AAAK，搜索将更快，同时保持每个字精确不变。但即使现在，壁橱方法对于在狭小空间中存储多少信息也是一个巨大的助力——它用于轻松地将你的 AI 智能体指向原始文件所在的抽屉。你永远不会丢失任何东西，这一切在几秒钟内完成。

还有**厅（hall）**，它们连接同一翼内的房间，以及**隧道（tunnel）**，它们连接不同翼的房间相互连接。所以查找事物变得真正毫不费力——我们给了 AI 一种干净有序的方式来知道从哪里开始搜索，而不必翻遍巨大文件夹中的每个关键词。

你说出你在找什么，砰，它已经知道该去哪个翼了。仅仅是这一点就会产生很大的不同。而且这很美、很优雅、很有机，最重要的是，很高效。

```
  ┌─────────────────────────────────────────────────────────────┐
  │  翼：人物                                                    │
  │                                                            │
  │    ┌──────────┐  ──厅──  ┌──────────┐                    │
  │    │  房间 A  │            │  房间 B  │                    │
  │    └────┬─────┘            └──────────┘                    │
  │         │                                                  │
  │         ▼                                                  │
  │    ┌──────────┐      ┌──────────┐                          │
  │    │  壁橱    │ ───▶ │  抽屉    │                          │
  │    └──────────┘      └──────────┘                          │
  └─────────┼──────────────────────────────────────────────────┘
            │
          隧道
            │
  ┌─────────┼──────────────────────────────────────────────────┐
  │  翼：项目                                                    │
  │         │                                                  │
  │    ┌────┴─────┐  ──厅──  ┌──────────┐                    │
  │    │  房间 A  │            │  房间 C  │                    │
  │    └────┬─────┘            └──────────┘                    │
  │         │                                                  │
  │         ▼                                                  │
  │    ┌──────────┐      ┌──────────┐                          │
  │    │  壁橱    │ ───▶ │  抽屉    │                          │
  │    └──────────┘      └──────────┘                          │
  └─────────────────────────────────────────────────────────────┘
```

**翼（Wing）**——一个人或项目。需要多少有多少。
**房间（Room）**——一个翼内的特定主题。认证、计费、部署——无尽的房间。
**厅（Hall）**——*同一*翼内相关房间之间的连接。如果房间 A（认证）和房间 B（安全）相关，一个厅连接它们。
**隧道（Tunnel）**——翼*之间*的连接。当人物 A 和项目都有一个关于"认证"的房间时，隧道自动交叉引用它们。
**壁橱（Closet）**——指向原始内容的摘要。（在 v3.0.0 中这些是纯文本摘要；AAAK 编码的壁橱将在未来更新中推出——参见 [Task #30](https://github.com/milla-jovovich/mempalace/issues/30)。）
**抽屉（Drawer）**——原始逐字文件。精确的字词，绝不摘要。

**厅**是记忆类型——在每个翼中都相同，充当走廊：
- `hall_facts`——已做出的决策、已锁定的选择
- `hall_events`——会话、里程碑、调试
- `hall_discoveries`——突破、新洞察
- `hall_preferences`——习惯、喜好、观点
- `hall_advice`——建议和解决方案

**房间**是命名的想法——`auth-migration`、`graphql-switch`、`ci-pipeline`。当同一房间出现在不同翼中时，它创建了一条**隧道**——跨领域连接同一主题：

```
wing_kai       / hall_events / auth-migration  → "Kai 调试了 OAuth token 刷新"
wing_driftwood / hall_facts  / auth-migration  → "团队决定将认证迁移到 Clerk"
wing_priya     / hall_advice / auth-migration  → "Priya 批准了 Clerk 而不是 Auth0"
```

同一个房间。三个翼。隧道连接它们。

### 为什么结构很重要

在 22,000+ 真实对话记忆上测试：

```
搜索所有壁橱：          60.9%  R@10
在翼内搜索：            73.1%  (+12%)
搜索翼 + 厅：          84.8%  (+24%)
搜索翼 + 房间：          94.8%  (+34%)
```

翼和房间不是装饰性的。它们是 **34% 的检索改进**。宫殿结构就是产品本身。

### 记忆栈

| 层 | 内容 | 大小 | 时机 |
|-------|------|------|------|
| **L0** | 身份——这个 AI 是谁？ | ~50 token | 始终加载 |
| **L1** | 关键事实——团队、项目、偏好 | ~120 token (AAAK) | 始终加载 |
| **L2** | 房间回忆——近期会话、当前项目 | 按需 | 当话题出现时 |
| **L3** | 深度搜索——跨所有壁橱的语义查询 | 按需 | 当被明确询问时 |

你的 AI 以 L0 + L1（约 170 token）唤醒，了解你的世界。搜索只在需要时触发。

### AAAK 方言（实验性）

AAAK 是一种有损缩写系统——实体编码、结构标记和句子截断——旨在将重复实体和关系以更少的 token 打包到大规模场景中。它**任何能读取文本的 LLM 都能阅读**（Claude、GPT、Gemini、Llama、Mistral），无需解码器，因此本地模型可以不依赖任何云服务来使用它。

**真实状态（2026 年 4 月）：**

- **AAAK 是有损的，不是无损的。** 它使用基于正则表达式的缩写，不是可逆压缩。
- **它在小规模下不节省 token。** 短文本的分词已经很高效。AAAK 的开销（编码、分隔符）在少量句子上成本高于节省。
- **它在大规模下可以节省 token**——在有许多重复实体的场景中（一个被提及数百次的团队，同一项目跨数千个会话），实体编码会摊销成本。
- **AAAK 目前在 LongMemEval 上相比原始逐字检索有所回退**（84.2% R@5 vs 96.6%）。96.6% 的标题数字来自**原始模式**，不是 AAAK 模式。
- **MemPalace 的存储默认值是 ChromaDB 中的原始逐字文本**——那是基准测试获胜的来源。AAAK 是一个独立的压缩层，用于上下文加载，不是存储格式。

我们正在迭代方言规范，添加真实分词器用于统计，并探索更好的切换点来决定何时使用它。在 [Issue #43](https://github.com/milla-jovovich/mempalace/issues/43) 和 [#27](https://github.com/milla-jovovich/mempalace/issues/27) 中跟踪进度。

### 矛盾检测（实验性，尚未接入知识图谱）

一个独立的工具（`fact_checker.py`）可以对照实体事实检查断言。它目前不会被知识图谱操作自动调用——这正在修复中（在 [Issue #27](https://github.com/milla-jovovich/mempalace/issues/27) 中跟踪）。启用后，它可以捕捉以下情况：

```
输入：  "Soren 完成了认证迁移"
输出： 🔴 AUTH-MIGRATION：归因冲突——Maya 被分配了，不是 Soren

输入：  "Kai 已经在这里 2 年了"
输出： 🟡 KAI：错误任期——记录显示 3 年（2023-04 入职）

输入：  "冲刺在周五结束"
输出： 🟡 SPRINT：过时日期——当前冲刺在周四结束（2 天前更新）
```

事实对照知识图谱检查。年龄、日期和任期动态计算——不是硬编码的。

---

## 真实世界示例

### 跨多个项目的独立开发者

```bash
# 挖掘每个项目的对话
mempalace mine ~/chats/orion/  --mode convos --wing orion
mempalace mine ~/chats/nova/   --mode convos --wing nova
mempalace mine ~/chats/helios/ --mode convos --wing helios

# 六个月后："我为什么在这里使用 Postgres？"
mempalace search "数据库决策" --wing orion
# → "选择 Postgres 而不是 SQLite，因为 Orion 需要并发写入
#    且数据集将超过 10GB。决定于 2025-11-03。"

# 跨项目搜索
mempalace search "限流方案"
# → 找到你在 Orion 和 Nova 中的方案，显示差异
```

### 管理产品的团队负责人

```bash
# 挖掘 Slack 导出和 AI 对话
mempalace mine ~/exports/slack/ --mode convos --wing driftwood
mempalace mine ~/.claude/projects/ --mode convos

# "Soren 上个冲刺做了什么？"
mempalace search "Soren 冲刺" --wing driftwood
# → 14 个壁橱：OAuth 重构、暗黑模式、组件库迁移

# "谁决定使用 Clerk？"
mempalace search "Clerk 决策" --wing driftwood
# → "Kai 推荐 Clerk 而不是 Auth0——价格 + 开发者体验。
#    团队于 2026-01-15 同意。Maya 负责迁移。"
```

### 挖掘前：拆分大文件

一些对话导出将多个会话连接成一个巨大文件：

```bash
mempalace split ~/chats/                      # 拆分为按会话的文件
mempalace split ~/chats/ --dry-run            # 先预览
mempalace split ~/chats/ --min-sessions 3     # 只拆分有 3+ 个会话的文件
```

---

## 知识图谱

时序实体-关系三元组——类似 Zep 的 Graphiti，但使用 SQLite 而不是 Neo4j。本地且免费。

```python
from mempalace.knowledge_graph import KnowledgeGraph

kg = KnowledgeGraph()
kg.add_triple("Kai", "works_on", "Orion", valid_from="2025-06-01")
kg.add_triple("Maya", "assigned_to", "auth-migration", valid_from="2026-01-15")
kg.add_triple("Maya", "completed", "auth-migration", valid_from="2026-02-01")

# Kai 在做什么？
kg.query_entity("Kai")
# → [Kai → works_on → Orion (当前), Kai → recommended → Clerk (2026-01)]

# 一月份什么是真的？
kg.query_entity("Maya", as_of="2026-01-20")
# → [Maya → assigned_to → auth-migration (活跃)]

# 时间线
kg.timeline("Orion")
# → 项目的按时间顺序的故事
```

事实具有有效期。当某事不再为真时，使其失效：

```python
kg.invalidate("Kai", "works_on", "Orion", ended="2026-03-01")
```

现在查询 Kai 当前的工作不会返回 Orion。历史查询仍然会。

| 特性 | MemPalace | Zep (Graphiti) |
|---------|-----------|----------------|
| 存储 | SQLite（本地） | Neo4j（云端） |
| 成本 | 免费 | $25/月+ |
| 时序有效性 | 是 | 是 |
| 自托管 | 始终 | 仅企业版 |
| 隐私 | 一切本地 | SOC 2, HIPAA |

---

## 专家智能体

创建专注于特定领域的智能体。每个智能体在宫殿中获得自己的翼和日记——不在你的 CLAUDE.md 中。添加 50 个智能体，你的配置大小保持不变。

```
~/.mempalace/agents/
  ├── reviewer.json       # 代码质量、模式、bug
  ├── architect.json      # 设计决策、权衡
  └── ops.json            # 部署、事件、基础设施
```

你的 CLAUDE.md 只需要一行：

```
你有 MemPalace 智能体。运行 mempalace_list_agents 来查看它们。
```

AI 在运行时从宫殿中发现它的智能体。每个智能体：

- **有焦点**——它关注什么
- **记日记**——用 AAAK 编写，跨会话持久化
- **建立专长**——阅读自己的历史以在其领域保持敏锐

```
# 智能体在代码审查后写入日记
mempalace_diary_write("reviewer",
    "PR#42|auth.bypass.found|missing.middleware.check|pattern:3rd.time.this.quarter|★★★★")

# 智能体读回自己的历史
mempalace_diary_read("reviewer", last_n=10)
# → 最近 10 条发现，以 AAAK 压缩
```

每个智能体是你数据上的专业镜头。审查者记住它见过的每一个 bug 模式。架构师记住每一个设计决策。运维智能体记住每一个事件。它们不共享草稿本——它们各自维护自己的记忆。

Letta 对智能体管理记忆收取 $20–200/月。MemPalace 用一个翼就做到了。

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
| `mempalace_list_wings` | 翼及其计数 |
| `mempalace_list_rooms` | 一个翼内的房间 |
| `mempalace_get_taxonomy` | 完整的翼 → 房间 → 计数树 |
| `mempalace_search` | 带翼/房间过滤的语义搜索 |
| `mempalace_check_duplicate` | 归档前检查 |
| `mempalace_get_aaak_spec` | AAAK 方言参考 |

**宫殿（写入）**

| 工具 | 功能 |
|------|------|
| `mempalace_add_drawer` | 归档逐字内容 |
| `mempalace_delete_drawer` | 按 ID 删除 |

**知识图谱**

| 工具 | 功能 |
|------|------|
| `mempalace_kg_query` | 带时间过滤的实体关系 |
| `mempalace_kg_add` | 添加事实 |
| `mempalace_kg_invalidate` | 标记事实为已结束 |
| `mempalace_kg_timeline` | 按时间顺序的实体故事 |
| `mempalace_kg_stats` | 图谱概览 |

**导航**

| 工具 | 功能 |
|------|------|
| `mempalace_traverse` | 从一个房间跨翼遍历图谱 |
| `mempalace_find_tunnels` | 查找连接两个翼的房间 |
| `mempalace_graph_stats` | 图谱连通性概览 |

**智能体日记**

| 工具 | 功能 |
|------|------|
| `mempalace_diary_write` | 写入 AAAK 日记条目 |
| `mempalace_diary_read` | 读取最近的日记条目 |

AI 从 `mempalace_status` 响应中自动学习 AAAK 和记忆协议。无需手动配置。

---

## 自动保存钩子

两个 Claude Code 钩子，在工作中自动保存记忆：

**保存钩子**——每 15 条消息，触发一次结构化保存。主题、决策、引述、代码变更。同时重新生成关键事实层。

**PreCompact 钩子**——在上下文压缩之前触发。窗口缩小前的紧急保存。

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

在标准学术基准测试上测试——可复现、已发布的数据集。

| 基准测试 | 模式 | 得分 | API 调用 |
|-----------|------|-------|-----------|
| **LongMemEval R@5** | 原始（仅 ChromaDB） | **96.6%** | 零 |
| **LongMemEval R@5** | 混合 + Haiku 重排序 | **100%** (500/500) | ~500 |
| **LoCoMo R@10** | 原始，会话级别 | **60.3%** | 零 |
| **个人宫殿 R@10** | 启发式基准 | **85%** | 零 |
| **宫殿结构影响** | 翼+房间过滤 | **+34%** R@10 | 零 |

96.6% 的原始得分是已发布的最高 LongMemEval 结果，不需要 API 密钥、不需要云端、在任何阶段都不需要 LLM。

### 与已发布系统的对比

| 系统 | LongMemEval R@5 | 需要 API | 成本 |
|--------|----------------|--------------|------|
| **MemPalace（混合）** | **100%** | 可选 | 免费 |
| Supermemory ASMR | ~99% | 是 | — |
| **MemPalace（原始）** | **96.6%** | **否** | **免费** |
| Mastra | 94.87% | 是 (GPT) | API 成本 |
| Mem0 | ~85% | 是 | $19–249/月 |
| Zep | ~85% | 是 | $25/月+ |

---

## 所有命令

```bash
# 设置
mempalace init <dir>                              # 引导式入门 + AAAK 引导

# 挖掘
mempalace mine <dir>                              # 挖掘项目文件
mempalace mine <dir> --mode convos                # 挖掘对话导出
mempalace mine <dir> --mode convos --wing myapp   # 用翼名标记

# 拆分
mempalace split <dir>                             # 拆分连接的对话记录
mempalace split <dir> --dry-run                   # 预览

# 搜索
mempalace search "查询"                           # 搜索一切
mempalace search "查询" --wing myapp              # 在一个翼内
mempalace search "查询" --room auth-migration     # 在一个房间内

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

### 翼配置（`~/.mempalace/wing_config.json`）

由 `mempalace init` 生成。将你的人物和项目映射到翼：

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

纯文本。成为第 0 层——每次会话都加载。

---

## 文件参考

| 文件 | 功能 |
|------|------|
| `cli.py` | CLI 入口点 |
| `config.py` | 配置加载和默认值 |
| `normalize.py` | 将 5 种聊天格式转换为标准对话记录 |
| `mcp_server.py` | MCP 服务器——19 个工具，AAAK 自动教学，记忆协议 |
| `miner.py` | 项目文件导入 |
| `convo_miner.py` | 对话导入——按交换对分块 |
| `searcher.py` | 通过 ChromaDB 的语义搜索 |
| `layers.py` | 4 层记忆栈 |
| `dialect.py` | AAAK 压缩——30 倍无损 |
| `knowledge_graph.py` | 时序实体-关系图（SQLite） |
| `palace_graph.py` | 基于房间的导航图 |
| `onboarding.py` | 引导式设置——生成 AAAK 引导 + 翼配置 |
| `entity_registry.py` | 实体编码注册表 |
| `entity_detector.py` | 从内容自动检测人物和项目 |
| `split_mega_files.py` | 将连接的对话记录拆分为按会话的文件 |
| `hooks/mempal_save_hook.sh` | 每 N 条消息自动保存 |
| `hooks/mempal_precompact_hook.sh` | 压缩前的紧急保存 |

---

## 项目结构

```
mempalace/
├── README.md                  ← 你在这里
├── mempalace/                 ← 核心包（README）
│   ├── cli.py                 ← CLI 入口点
│   ├── mcp_server.py          ← MCP 服务器（19 个工具）
│   ├── knowledge_graph.py     ← 时序实体图
│   ├── palace_graph.py        ← 房间导航图
│   ├── dialect.py             ← AAAK 压缩
│   ├── miner.py               ← 项目文件导入
│   ├── convo_miner.py         ← 对话导入
│   ├── searcher.py            ← 语义搜索
│   ├── onboarding.py          ← 引导式设置
│   └── ...                    ← 参见 mempalace/README.md
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
├── tests/                     ← 测试套件（README）
├── assets/                    ← logo + 品牌资源
└── pyproject.toml             ← 包配置（v3.0.0）
```

---

## 系统要求

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

MIT——参见 [LICENSE](https://github.com/milla-jovovich/mempalace/blob/main/LICENSE)。

<!-- 链接定义 -->
[version-shield]: https://img.shields.io/badge/version-3.0.0-4dc9f6?style=flat-square&labelColor=0a0e14
[release-link]: https://github.com/milla-jovovich/mempalace/releases
[python-shield]: https://img.shields.io/badge/python-3.9+-7dd8f8?style=flat-square&labelColor=0a0e14&logo=python&logoColor=7dd8f8
[python-link]: https://www.python.org/
[license-shield]: https://img.shields.io/badge/license-MIT-b0e8ff?style=flat-square&labelColor=0a0e14
[license-link]: https://github.com/milla-jovovich/mempalace/blob/main/LICENSE
[discord-shield]: https://img.shields.io/badge/discord-join-5865F2?style=flat-square&labelColor=0a0e14&logo=discord&logoColor=5865F2
[discord-link]: https://discord.com/invite/ycTQQCu6kn