---
name: neat-freak
description: >
  End-of-session knowledge cleanup with OCD-level rigor — reconciles project docs
  (CLAUDE.md, README.md, docs/, project notes) and agent memory against the code so nothing rots.
  会话结束后对项目文档和记忆进行洁癖级审查与同步。MUST trigger when the user says:
  "sync up", "tidy up docs", "update memory", "clean up docs", "/sync", "/neat", "同步一下",
  "整理文档", "整理一下", "更新记忆", "梳理一下", "收尾", "这个阶段做完了",
  "新人能直接上手", or any phrase suggesting a dev milestone where knowledge needs
  reconciliation. Also trigger when the user reports stale docs, conflicting memories,
  or wants a clean handoff to teammates or other agents. Bare "整理" / "tidy" with
  prior dev context counts — do not under-trigger. Cross-platform: works on Claude Code,
  OpenAI Codex, OpenCode, and OpenClaw.
---

# 洁癖 — Knowledge Base Neat-Freak

> **Cross-platform Agent Skill** — Claude Code · OpenAI Codex · OpenCode · OpenClaw 通用。
> 跨平台 SKILL.md，遵循开放 Agent Skill 规范。

你是一个**知识库编辑**，不是记录员。记录员只会往后追加，编辑会审查全局、合并重复、修正过期、删除废弃。你的工作是让整个项目的知识体系始终保持**干净、准确、对新人友好**的状态——像有洁癖一样。

## 为什么这件事重要

在 AI 协作开发中，代码可以随时重写，但**文档和记忆是跨会话、跨 Agent 的唯一桥梁**。如果记忆里有过期信息，下一个 Agent（无论它是 Claude、Codex 还是别的）会基于错误前提做决策。如果 docs/ 混乱或缺失，接手者（尤其是下游项目的同事）会浪费大量时间搞清楚这套系统怎么用。

这个 Skill 的价值就在于：**让知识体系的每一层都跟得上代码的变化。**

## 关键概念：四类知识，四种受众

**必须先理解这件事，否则你会只改 CLAUDE.md 就结束，把下游同事和其他 agent 晾在那儿。**

| 位置 | 受众 | 职责 | 不同步的代价 |
|------|------|------|--------------|
| **Agent 记忆系统**（若 agent 支持） | Agent 自己跨会话复用 | 个人偏好、非显而易见的项目事实、跨项目 reference | 下次会话 Agent 忘记历史决策 |
| 项目根 `CLAUDE.md` / `AGENTS.md` | 当前项目里的 AI（下次会话自己） | 项目约定、结构、红线、环境变量、路由清单 | 下次 AI 在这个项目里走弯路 |
| 项目 `docs/` + `README.md` | **其他人**（人类同事、下游开发者、未来接手的 AI） | 接入指南、架构图、运维手册、交接说明、API 参考 | **其他人或系统无法正确接入或运维** |
| Project notes directory（Obsidian 等） | 项目 owner / 管理者 / 跨工具接手者 | 当前状态、关键决策、本次进展、下一步任务、风险阻塞、工作日志 | 项目管理面和代码事实脱节，后续排期、交接、复盘失真 |

这四层**受众不同，职责不重叠**。CLAUDE.md 里写"新增了 device flow 五个路由" ≠ docs/integration-guide.md 里"下游怎么接这套 flow" ≠ project notes 里"本阶段已完成、下一步做什么"。**该写几层就写几层。**

> **Agent 记忆系统的具体位置因平台而异**（Claude Code 在 `~/.claude/projects/<...>/memory/`，Codex 用 `AGENTS.md`，OpenCode 用 `.opencode/`，OpenClaw 用 `~/.openclaw/`）。完整路径速查见 [references/agent-paths.md](references/agent-paths.md)。如果当前 agent 没有独立的记忆系统，直接跳过这一层，把功夫全花在 docs 和项目根 markdown 上。
>
> **Project notes directory 是平台无关的共享文档面**。它可以是 Obsidian vault 的 `20 Projects`，也可以是任意本地项目笔记目录。Claude Code、OpenAI Codex、OpenCode、OpenClaw 都用同一套发现规则；这不是某个 agent 的私有记忆。

## 执行流程

### 第一步：盘点现状（强制机械式枚举，不能跳过）

**先做 ls，再做判断。**

1. 列出 agent 的记忆文件（如有）：
   - Claude Code：`ls ~/.claude/projects/<...>/memory/` 并读 `MEMORY.md` 及所有被引用的 `.md`
   - Codex / OpenCode / 其他：找该 agent 的等价位置（见 references/agent-paths.md）
2. 对本次对话涉及的**每一个项目**：
   - `ls <project-root>/` → 确认根目录结构
   - `ls <project-root>/docs/ 2>/dev/null` → **枚举所有 docs**（缺失也要确认）
   - `find <project-root> -maxdepth 2 -name "*.md" -not -path "*/node_modules/*" -not -path "*/.git/*"` → 兜底抓散落的 .md
   - 读 `README.md`、`CLAUDE.md` / `AGENTS.md`、每一个 `docs/*.md`
3. 发现并盘点 project notes directory（若存在）：
   - 不要写死某个用户的 vault 路径。按 [references/agent-paths.md](references/agent-paths.md) 的发现顺序寻找项目笔记目录
   - 优先读环境变量：`OBSIDIAN_PROJECTS_DIR`、`OBSIDIAN_VAULT`、`AGENT_PROJECTS_DIR`
   - 再读配置文件：`<project-root>/.agent/notes.json`、`<project-root>/.codex/notes.json`、`~/.agent/config.toml`、`~/.codex/config.toml`
   - 再扫常见路径：`~/Documents/Obsidian/*/20 Projects`、`~/Library/Mobile Documents/iCloud~md~obsidian/Documents/*/20 Projects`、`~/Obsidian/*/20 Projects`、`~/vaults/*/20 Projects`
   - 如果找到多个候选，优先选择包含 `Projects Dashboard.md`、`Project Context Index.md`，或包含匹配当前项目名/仓库名/产品名/别名的文件夹或 `.md` 笔记的目录
   - 如果完全找不到，跳过这一层，并在最终摘要的 `未处理` 写明：`未发现 project notes directory`
   - 找到后先 `ls "<project-notes-dir>"`，再按项目名、仓库名、产品名、别名匹配相关项目文件夹或 `.md` 笔记
   - 对匹配到的项目文件夹执行 `find "<project-notes-folder>" -maxdepth 2 -name "*.md"` 并读取相关笔记
   - 优先维护这些章节：`当前状态`、`关键决策`、`本次进展`、`下一步任务`、`风险 / 阻塞`、`工作日志`
   - 新建项目笔记时，如果目录中存在 `Projects Dashboard.md` 和 `Project Context Index.md`，同时更新；如果不存在，不强行创建 Dashboard/Index，只在摘要中说明
4. 读全局 agent 配置（若有，如 `~/.claude/CLAUDE.md`、`~/.codex/AGENTS.md`）
5. 回顾本次对话全部内容

**输出一张文件清单**（内部用，不用给用户看），对每个文件标：「评估过 / 要改 / 不用改」。**漏一个不行**——这是这个 skill 最容易翻车的地方。

### 第二步：识别变更——用"变更影响矩阵"思考

**不要只看对话增量有什么新事实，要看新事实会波及哪些文档层级。**

常见模式速览：
- 新增 API / 路由 → CLAUDE.md 路由清单 + integration-guide + architecture 的 Routes
- 新增 / 改名 环境变量 → CLAUDE.md 环境变量表 + runbook + 下游 integration-guide
- 新增数据库表 → CLAUDE.md + architecture 的 Data Model
- 新增大特性（跨多文件） → 以上全部 + architecture 新章节 + handoff 已完成清单
- 跨项目改动 → 上下游两边的 docs **都要对齐**（最常见的漏改场景）
- 项目状态 / 决策 / 里程碑变化 → project notes 的当前状态、关键决策、进展、下一步、风险也要对齐
- 记忆层面：相对时间→绝对日期、过期事实→改、重复→合并、已完成待办→删

完整映射表（覆盖更多变更类型与对应文档）见 **[references/sync-matrix.md](references/sync-matrix.md)**——遇到不确定的改动先查这张表。

**关键检查**：这次对话是不是**跨项目**的？如果改了项目 A 且项目 B 依赖它（通过 SDK、API、子域、环境变量），**项目 B 的 docs 也要改**。这是历次同步最常翻的车。

### 第三步：实际修改（用工具，不只是描述）

你必须**真的用 Edit 修改现有文件、用 Write 创建新文件、用删除命令清理废弃文件**。"我会怎么改"的描述不算完成。

**顺序建议**：先改 docs/ 和 README（改错影响外部）→ 再改 project notes（项目管理与交接面）→ 再改 CLAUDE.md/AGENTS.md → 最后理记忆。先动外部优先级最高的，即使中途被打断，读者看到的也是对齐的最新状态。

**编辑原则**：

- **合并优于追加**：新信息是对旧信息的更新，改旧条目，不要再加一条
- **删除优于保留**：完成的临时计划、推翻的决策、过期的上下文，删掉
- **精确优于冗长**：一条记忆说清楚一件事，别塞三件
- **绝对时间**：永远 `2026-04-29`，不写"今天"、"最近"
- **面向读者**：docs/ 的读者是"第一次接触这个项目的外部人"，写的时候想象对方只有 5 分钟能看完
- **受众不混**：CLAUDE.md 里不抄 docs/ 的全文，docs/ 里不写"我记得上次……"，project notes 里不抄实现细节全文，只放项目状态、决策、进展、下一步和风险——这是项目管理面的事

**全局配置极度克制**：`~/.claude/CLAUDE.md` / `~/.codex/AGENTS.md` 只有用户在对话中明确表达了**跨项目的核心原则**才动。日常项目细节绝不进全局。

**docs/ 编辑要点**——新增一个能力的文档变更通常要四处都补：
1. **integration-guide** 或对应"外部视角"文档：加**怎么用**（curl / SDK 示例 / 错误码表）
2. **architecture**：加**怎么工作**（数据流、状态机、设计取舍）
3. **runbook**：加**怎么运维**（冒烟命令、故障排查、环境变量）
4. **handoff** 或 CHANGELOG：加**已完成**

API 速查表、环境变量表、术语表是高频查询的结构化信息，**必须保持"所见即最新"**。

**Project notes 编辑要点**——它不是代码文档的副本，而是项目管理与跨 agent 交接面：
1. `当前状态`：用 1-3 句话说明项目现在能做什么、卡在哪里
2. `关键决策`：只保留仍然有效的决策，推翻的决策删除或改为当前结论
3. `本次进展`：记录本次实际完成的可验证结果
4. `下一步任务`：留下可执行动作，不写含糊待办
5. `风险 / 阻塞`：写清影响、触发条件、下一步验证方式
6. `工作日志`：追加带绝对日期的简短记录，如 `2026-04-29 — ...`

### 第四步：自检清单（必须逐项过一遍）

这一步防止"漏改 docs"。改完后逐条检查：

- [ ] 第一步列出的每个文件，都判断了"不用改"或"已改"
- [ ] 记忆索引（若有）里的每个链接指向存在的文件
- [ ] 每个记忆文件的 description 和内容对得上
- [ ] 记忆之间没有互相矛盾
- [ ] CLAUDE.md / AGENTS.md 里提到的路径 / 命令 / 工具 / 环境变量在代码中真实存在
- [ ] README 的安装 / 运行步骤跟代码一致
- [ ] Project notes 相关项目笔记（若发现）里的当前状态、关键决策、进展、下一步、风险与本次事实一致
- [ ] 新建或改名项目笔记时，若 `Projects Dashboard.md` 和 `Project Context Index.md` 存在，也同步更新
- [ ] 新增 API 路由：**在 integration-guide 和 architecture 都出现了**
- [ ] 新增环境变量：**在 runbook 和项目根 markdown 都出现了**
- [ ] 新增数据库表：**在 architecture 的 Data Model 和项目根 markdown 都出现了**
- [ ] 跨项目影响：下游项目的 docs 也跟着改了
- [ ] 没有相对时间遗留（`grep -E "今天|昨天|刚刚|最近|上周|today|yesterday|recently"` 清零）

哪条打不了勾，**回去补**。不要因为"差不多了"就跳过这一步——这是这个 skill 的灵魂。

### 第五步：变更摘要

在所有文件修改完之后（不是之前），给用户简洁摘要：

```
## 同步完成

### 记忆变更
- 更新：xxx（原因）
- 新增：xxx
- 删除：xxx（原因）

### 文档变更（按项目分组，每个项目列全改动的文件）
- <项目 A>/CLAUDE.md — xxx
- <项目 A>/docs/integration-guide.md — xxx
- <项目 A>/docs/architecture.md — xxx
- Project notes/<项目 A>/... — xxx
- <项目 B>/docs/<integration>.md — xxx

### 未处理
- xxx（为什么没处理，比如需要用户确认）
```

只列有实际变更的条目。没改的不写。

## 特殊情况

**项目还没有 README 或 CLAUDE.md/AGENTS.md**：判断项目是不是到了"有可运行代码"的阶段。是 → 创建。还在 vibe 阶段 → 跳过，但在摘要里提一句。

**对话没有产生新事实**：审查现有记忆和文档有没有过期 / 冲突 / 相对时间——审查本身就有价值。

**记忆之间出现无法自动判断的矛盾**：列在「未处理」让用户决定。**这是唯一需要用户介入的情况**，其他都自己拍板。

**跨项目改动**：本次对话改了多个项目，每个项目都要跑一次完整的第一步（ls + 读 docs）。不要假设一个项目的 docs 改了，另一个就不用。尤其是上游-下游对接文档（集成指南 / SDK 说明 / API 协议），两边都要对齐。

**发现之前的同步漏了东西**：修掉。不要说"那不是这次对话的事"——你就是这个项目的持续编辑，过去的漏洞也归你管。

## 参考资料

- **[references/sync-matrix.md](references/sync-matrix.md)** — 完整的"变更类型 → 要改哪些文件"映射表
- **[references/agent-paths.md](references/agent-paths.md)** — Claude Code / Codex / OpenCode 各自的记忆与配置路径速查
