# Agent 记忆与配置路径速查

不同 agent 平台的记忆系统和项目配置文件位置不一样。执行第一步盘点时按你正在使用的平台查这张表。

## 平台无关：Obsidian 项目笔记

| 用途 | 路径 |
|---|---|
| Obsidian Vault | `/Users/mikawang/Documents/Obsidian/Mika Daily` |
| 项目管理目录 | `/Users/mikawang/Documents/Obsidian/Mika Daily/20 Projects` |
| Projects Dashboard | `/Users/mikawang/Documents/Obsidian/Mika Daily/20 Projects/Projects Dashboard.md` |
| Project Context Index | `/Users/mikawang/Documents/Obsidian/Mika Daily/20 Projects/Project Context Index.md` |

这是一层**跨 agent 共享的项目管理文档面**，不是 Claude / Codex / OpenCode / OpenClaw 的私有记忆。只要本机存在该 Vault，所有 agent 在执行 neat-freak 时都要按项目名、仓库名、产品名、别名匹配 `20 Projects` 下的项目文件夹或 `.md` 笔记，并把相关笔记纳入盘点与同步。

优先维护章节：`当前状态`、`关键决策`、`本次进展`、`下一步任务`、`风险 / 阻塞`、`工作日志`。新建项目笔记时，同时更新 `Projects Dashboard.md` 和 `Project Context Index.md`。

## Claude Code

| 用途 | 路径 |
|---|---|
| 跨会话记忆(全局) | `~/.claude/projects/<encoded-project-path>/memory/` |
| 记忆索引文件 | `~/.claude/projects/<...>/memory/MEMORY.md` |
| 全局指令 | `~/.claude/CLAUDE.md` |
| 项目级指令 | 项目根 `CLAUDE.md`(可层级嵌套) |
| Skills 目录 | `~/.claude/skills/<name>/SKILL.md` |

记忆文件用 YAML frontmatter:`name`、`description`、`type`(user / feedback / project / reference)。

## OpenAI Codex

| 用途 | 路径 |
|---|---|
| 跨会话指令(全局) | `~/.codex/AGENTS.md` 或 `$CODEX_HOME/AGENTS.md` |
| 项目级指令 | 项目根 `AGENTS.md`(可层级嵌套) |
| 项目级 override | `AGENTS.override.md`(若存在,覆盖同目录 AGENTS.md) |
| Skills 目录 | `~/.codex/skills/<name>/SKILL.md` 或项目内 `.codex/skills/<name>/` |

Codex 没有独立的"记忆文件 + 索引"机制,所有跨会话信息都直接写在 `AGENTS.md` 里。同步时把"项目事实"那部分内容统一放 AGENTS.md。

发现项目里有 `TEAM_GUIDE.md` 或 `.agents.md` 也要看——这是 Codex 的 fallback 文件名。

## OpenClaw

| 用途 | 路径 |
|---|---|
| 用户级 skills | `~/.openclaw/skills/<name>/SKILL.md`（首次运行自动创建） |
| 项目级 skills | `.openclaw/skills/<name>/SKILL.md`（仓库根目录下） |
| Workspace skills | 当前 workspace 的 `skills/` 目录 |

**加载优先级**：workspace > project-agent > personal-agent > managed/local > bundled > extra dirs。同名 skill 高优先级覆盖低优先级。

OpenClaw 没有独立的"记忆文件 + 索引"机制，跨会话信息可放在项目根的 markdown（CLAUDE.md / AGENTS.md / 等价文件）里，参照 Codex 的做法。frontmatter 支持 `metadata.openclaw` 字段做加载时的 gating（按 OS、环境变量、二进制依赖筛选），但不是 neat-freak 必需的。

## OpenCode

| 用途 | 路径 |
|---|---|
| 全局配置 | `~/.config/opencode/` |
| 项目配置 | `.opencode/` |
| Skills 目录(项目) | `.opencode/skills/`、`.claude/skills/`、`.codex/skills/` 都会被扫描 |
| Skills 目录(全局) | `~/.config/opencode/skills/`、`~/.claude/skills/`、`~/.codex/skills/` |

OpenCode 同时读取 Claude Code 和 Codex 的目录,所以同一个 skill 装在 `~/.claude/skills/` 下的话三家都能识别。OpenClaw 走自己的 `~/.openclaw/skills/`，需要单独装一份（或用符号链接）。

## 如果当前 agent 没有独立记忆系统

跳过"记忆"那一层,把功夫全花在:
- 项目根 markdown(CLAUDE.md / AGENTS.md / 本平台等价文件)
- README.md
- docs/
- Obsidian `20 Projects` 里的相关项目笔记(若本机存在)

仍然是有效的同步——记忆是锦上添花,docs 才是项目知识的最低保障。

## 跨平台共存策略

如果一个项目同时被 Claude Code 用户和 Codex 用户使用,推荐:

- **项目根同时放 `CLAUDE.md` 和 `AGENTS.md`**,内容可以互相 symlink 或在两边维护
- 或者一份内容主文件 + 另一份用一行 `See CLAUDE.md` 跳转
- docs/、README 和 Obsidian `20 Projects` 项目笔记是平台中立的,不需要按 agent 分多份
