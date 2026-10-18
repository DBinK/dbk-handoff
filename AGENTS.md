# AGENTS.md

面向在本仓库工作的 AI Agent 与贡献者的约定。

## 项目性质

`dbk-handoff` 是一个纯文件协议：`skills/dbk-handoff/SKILL.md`（发出端）写交接快照，`skills/dbk-pickup/SKILL.md`（接收端）读快照恢复上下文。没有构建步骤、没有运行时依赖，产物是可校验的 Markdown。

## 目录

| 路径 | 作用 |
| :--- | :--- |
| `ARCHITECTURE.md` | 协议权威规范（命名、元数据、工作流、Skill 设计） |
| `README.md` | 面向用户的项目介绍、安装与使用 |
| `CHANGELOG.md` | Keep a Changelog 格式变更记录 |
| `skills/dbk-handoff/SKILL.md` | 发出端技能定义 + 报告模板 + 引导 Prompt 模板 |
| `skills/dbk-pickup/SKILL.md` | 接收端技能定义 + 校验与加载约束 |
| `skills/*/evals/trigger-queries.json` | 触发评估查询集（正例 / 边界负例） |

## 修改规则

- **协议优先**：任何字段、命名、目录语义的变更先改 `ARCHITECTURE.md`，再同步两个 `SKILL.md`、`README.md` 与 `CHANGELOG.md`，三者不得各自漂移。
- **模板唯一**：交接报告模板与引导 Prompt 模板在 `ARCHITECTURE.md` 与 `dbk-handoff/SKILL.md` 各有一份，修改时必须同步。
- **版本语义**：`handoff_version` 为单整数，仅在破坏性不兼容变更时递增；Skill 自身的 `metadata.version` 使用 SemVer。
- **兼容性**：未知 frontmatter 字段必须被忽略；新增字段一律可选，且在 `SKILL.md` 中写清缺省行为。
- **不要提交快照**：`.agent/handoff/.gitignore` 忽略目录内全部内容，仅在确实需要演示时才例外。
- **触发集同步**：调整 `description` 中的触发措辞后，复查 `evals/trigger-queries.json` 的正例与负例是否仍覆盖该边界。

## 校验

```bash
uvx --from skills-ref agentskills validate ./skills/dbk-handoff
uvx --from skills-ref agentskills validate ./skills/dbk-pickup
```

注意 PyPI 上 `skills-ref` 包的可执行文件名是 `agentskills`。

本地安装自测：

```bash
npx skills add ./skills -g -y
```

## 提交约定

Conventional Commits，类型后接中文描述，例如 `docs: 明确 git.dirty 的采集时机`。
