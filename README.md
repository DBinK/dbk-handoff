# dbk-handoff

[![skills.sh](https://skills.sh/b/DBinK/dbk-handoff)](https://skills.sh/DBinK/dbk-handoff)
[![Agent Skills](https://img.shields.io/badge/Agent_Skills-Standard-black)](https://agentskills.io)
[![Protocol](https://img.shields.io/badge/Handoff_Protocol-v1-blue)](ARCHITECTURE.md)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)

> **跨 AI Agent 的通用上下文交接与接力协议**

在 Claude Code、Cursor、Windsurf、OpenHands 等工具之间切换时，新 Agent 往往缺少上一个会话的目标、进度和踩坑记录，只能重新翻代码。`dbk-handoff` 用一份高密度 Markdown 快照解决这个问题：发出端负责总结，接收端负责接手，全程不依赖服务端或外部中转。

## 核心特性

- **纯文件协议**：只需要读写文件的能力，报告是普通 Markdown，无数据库、无 RPC。
- **渐进式加载**：优先消费摘要；原始会话日志仅在关键信息缺失时按需检索，避免上下文爆炸。
- **防重复考古**：报告显式列出核心工作集、已排除的死胡同方案和验证命令，新 Agent 不需要重新摸索。
- **可追溯**：每次交接生成带时间戳的历史快照，`latest.md` 始终保存最新副本。
- **人工确认环**：交接通过一段可复制的 Prompt 完成，用户可以在接手前审计和补充。

## 灵感与设计来源

本项目的结构化交接模板与设计哲学深受 [`can1357/oh-my-pi`](https://github.com/can1357/oh-my-pi)（omp）项目及其 `/handoff` 机制启发。特别吸纳了 omp 对**核心工作集（Key Files，8~15 个文件上下文）**、**已排除的死胡同方案（Dead Ends，防止接手 Agent 重蹈覆辙）**与**验证闭环（Verification Plan）**的严格定义，避免把时间耗在“代码考古（File Archaeology）”上。

## 组成

- **`dbk-handoff`（发出端）**：总结当前进度、写入 `.agent/handoff/`、生成接手指引。
- **`dbk-pickup`（接收端）**：读取交接快照，向用户确认理解后继续执行。

## 安装

```bash
npx skills add DBinK/dbk-handoff -g -y
```

支持 skills 生态的客户端可直接执行上述命令；其他客户端可由 Agent 根据自身环境完成 Skill 安装。

读取交接报告必须遵循 `dbk-pickup` 中的指引（路径解析、版本与 Git 状态校验、Transcript 按需检索），不要绕过 Skill 直接裸读 `latest.md`。

## 使用

1. **发出交接**：在当前 Agent 中调用 `dbk-handoff`。它会写入一份 `<YYMMDD>-<HHMM>-<主题>.md` 快照和 `latest.md` 副本，并输出一段引导 Prompt。
2. **接手任务**：把引导 Prompt 粘贴给新 Agent。它调用 `dbk-pickup`，读取报告、复述理解，然后继续执行。

报告元数据示例：

```yaml
---
handoff_version: 1
created: "2026-10-18T12:30:00+08:00"
snapshot: 261018-1230-auth-refactor.md
workspace: /abs/path/to/workspace
source_agent: claude-code
source_model: claude-sonnet-4
task: auth-refactor
git: { branch: main, head: 11670ce, dirty: true }
---
```

## 兼容性

任何具备文件读写能力的 Agent 都可以参与交接流程，但读取报告必须加载 `dbk-pickup` 的指引；支持 Skill 的客户端可一键调用，其他客户端可由 Agent 按该指引执行。快照目录自带 `.gitignore`，默认不会污染业务仓库。

## 文档

- [ARCHITECTURE.md](ARCHITECTURE.md) — 存储与命名规范、报告元数据、工作流程、Skill 设计细节。
- [AGENTS.md](AGENTS.md) — 面向 Agent 与贡献者的修改约定与校验命令。
- [CHANGELOG.md](CHANGELOG.md) — 项目显著变更记录。

## 许可

[MIT](LICENSE)
