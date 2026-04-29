# 变更影响矩阵

遇到不确定"这次改动要同步哪些文件"时查这张表。

## 代码层变更 → 文档层变更

| 本次对话发生的事 | 要改的文件(按受众) |
|---|---|
| 新增 API / 路由 | 项目根 markdown 路由清单 · `docs/integration-guide.md` API 速查表 · `docs/architecture.md` Routes 小节 |
| 新增 / 改名 环境变量 | 项目根 markdown 环境变量表 · `docs/operator-runbook.md` 环境变量章节 · `docs/integration-guide.md`(如果下游要配) |
| 新增数据库表 / 列 | 项目根 markdown 数据库表 · `docs/architecture.md` Data Model |
| 新增 / 改动 用户流程 | 项目根 markdown 用户流程 · README 相关命令行示例 · `docs/handoff.md` What Exists Today |
| 新增大特性(能跨多文件) | 以上全部 + `docs/architecture.md` 新增章节 + `docs/handoff.md` 已完成清单 |
| 新增术语 / 改命名 | `docs/integration-guide.md` 术语表(如果有)+ 全局搜索旧术语替换 |
| 部署参数 / 基础设施变化 | `docs/operator-runbook.md` · 项目根 markdown 部署章节 |
| 下游项目接入方式变化 | 下游项目的 `docs/<integration>.md` · 上游项目的 `integration-guide.md` |
| 项目阶段完成 / 里程碑变化 | Project notes 当前状态 · 本次进展 · 下一步任务 · 工作日志 |
| 新决策 / 推翻旧决策 | 项目根 markdown(若影响 agent 执行) · `docs/architecture.md`(若影响系统设计) · Project notes 关键决策 |
| 风险 / 阻塞变化 | `docs/operator-runbook.md`(若影响运维) · Project notes 风险 / 阻塞 |

## 记忆层变更

| 情况 | 处理方式 |
|---|---|
| 过期事实 | 改记忆文件,同时更新索引(如 MEMORY.md)的 description |
| 相对时间("今天"、"最近") | 全部转成绝对日期(`2026-04-29` 而非"今天") |
| 重复记录(多条说同一件事) | 合并为一条,改索引 |
| 已完成的待办 | 删除——知识库不是历史档案 |
| 推翻的决策 | 删除旧条目,留新决策 |
| 跨会话只用一次的临时上下文 | 删除 |

## 跨项目影响检查

最容易漏改的场景:

- **上游 API 变了 → 下游 SDK 文档**:协议变化必须两边对齐
- **共享子域 / 路由 / 环境变量改了 → 所有 consumer 项目的 setup 文档**
- **认证中台变更 → 所有接入应用的 integration guide**
- **公共组件 / 基础设施 升级 → 各项目的 operator-runbook 提及版本号的地方**

判断方法:这次改的东西有没有 SDK、子域、共享配置、跨进程协议?有就要在所有依赖项目里搜一遍提到这件事的文档。

Project notes 也要按跨项目影响检查：如果发现的项目笔记目录下存在上游和下游两个项目文件夹/笔记，不能只更新当前代码仓库对应的那一个。上游 API、共享配置、交付状态、阻塞原因发生变化时，下游项目笔记里的当前状态、下一步任务和风险也要对齐。

## 文档结构通用约定

新增一个能力(API、flow、特性)的标准动作是**四处都补**:

1. **integration-guide / 外部视角文档**:怎么用(curl / SDK 示例 / 错误码)
2. **architecture**:怎么工作(数据流、状态机、设计取舍)
3. **runbook**:怎么运维(冒烟命令、故障排查、环境变量)
4. **handoff / CHANGELOG**:已完成
5. **Project notes**:项目状态、关键决策、本次进展、下一步任务、风险、工作日志

API 速查表、环境变量表、术语表是高频查询的结构化信息,**必须保持"所见即最新"**。
