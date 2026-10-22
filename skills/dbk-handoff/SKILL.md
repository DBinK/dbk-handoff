---
name: dbk-handoff
description: 总结当前会话的上下文与开发进度，生成标准化交接快照写入 .agent/handoff/ 并输出接手引导 Prompt，用于跨 AI Agent（Claude Code、Cursor、Windsurf、Antigravity、OpenHands、pi、opencode 等）或跨会话无缝接力。当用户要求生成交接文档、导出当前上下文、准备切换客户端或新开会话继续开发、做阶段性交接归档，或提到“交接”“handoff”“总结一下给下一个 agent 接着做”“换个窗口继续”时使用。不适用于在新会话中读取已有交接报告接手任务（应使用 dbk-pickup），也不适用于常规的 git commit message 生成、PR 描述撰写或普通代码总结。
license: MIT
metadata:
  author: DBinK
  version: "0.3.0"
---

# 上下文交接导出（dbk-handoff）

在当前会话结束或切换 AI 客户端前，将任务目标、核心工作集、已完成进展、踩坑记录与下一步待办沉淀为高密度交接快照，并输出可一键复制的引导 Prompt。

## 工作流

### 1. 采集环境与仓库元数据

准备交接报告所需的 YAML frontmatter 字段：

- `handoff_version`：固定填写整数 `1`。
- `created`：当前本地时间，采用带时区偏移的 ISO 8601 字符串并加双引号（如 `"2026-10-18T12:30:00+08:00"`）。
- `task`：2~5 个小写英文单词组成的简短主题，单词间用连字符 `-` 连接（如 `auth-refactor`、`fix-token-expiry`）。
- `snapshot`：权威快照文件名，格式为 `<YYMMDD>-<HHMM>-<task>.md`（时间与 `created` 的本地时间一致，如 `261018-1230-auth-refactor.md`）。
- `workspace`：当前工作区根目录的绝对路径。
- `source_agent`：当前客户端标识（如 `claude-code`、`cursor`、`antigravity`、`pi`、`opencode`）。
- `source_model`（可选）：当前模型标识，未知时可省略。
- `spec`（可选）：用户提供的需求文档相对/绝对路径（如 `docs/rfcs/auth.md`）或在线需求单链接（如飞书文档、Meegle 单号）；无明确需求文档时省略。
- `git`（可选）：若当前工作区是 Git 仓库，用只读命令获取当前分支名（`branch`）、HEAD 短哈希（`head`）以及是否有未提交变更（`dirty: true|false`）。`dirty` 应在写入交接文件之前采集，并忽略 `.agent/` 路径，避免交接产物本身把工作区标记为脏。非 Git 仓库省略该字段。
- `transcript`（可选）：**仅当当前上下文环境已显式暴露本机对话日志文件的绝对路径时填写**。未暴露时直接省略该字段，**严禁在宿主目录盲目搜索或编造猜测路径**。

### 2. 提炼高密度交接内容

回顾当前对话与代码改动，提炼以下六个板块（力求精炼具体，拒绝空泛套话）：

1. **🎯 任务目标与交付基线 (Goal & Baseline - 自适应双轨制)**：
   - **普通敏捷开发（无需求文档）**：概述用户的原始需求、边界条件与最终验收标准。
   - **规范驱动开发（用户提供了 PRD / 技术方案 / 需求单等）**：在 frontmatter 记录 `spec` 引用，并将本板块扩展为**需求与交付基线矩阵**，切勿全量拷贝 PRD 正文，而应逐条对照当前工作区代码与测试，列出功能项状态：
     ```markdown
     ## 📋 需求与交付基线 (Spec & Baseline)
     - **需求基准**: `<spec 路径或链接>`
     - **对照进度**: [M/N] 已完成 M 项，1 项进行中，余下待开工

     | 需求条目 / 功能点 | 状态 | 对应代码 / 验证状态 |
     | :--- | :---: | :--- |
     | 1. <功能点 A> | ✅ 已完成 | `<实现路径>`（已测试跑通） |
     | 2. <功能点 B> | 🚧 进行中 | `<实现路径>`（**当前停顿点**：具体进度） |
     | 3. <功能点 C> | ⏳ 待开工 | 尚未触及 |
     ```
2. **📂 核心工作集 (Key Files & Working Set)**：列出 5~15 个关键文件路径（已修改文件、核心依赖、相关测试、配置文件），统一使用相对于工作区根目录的路径，并附一句话说明该文件的角色或修改状态（未提交改动请注明），避免接手 Agent 全局盲搜。
3. **✅ 已完成事项 (Completed)**：已经落地的代码改动与跑通的验证结果。
4. **💡 关键决策与避坑指南 (Decisions & Dead Ends)**：
   - **采用方案**：关键技术选型与设计理由。
   - **已踩坑/已排除**：**重点记录已经尝试过但失败的死胡同方案、报错原因或环境限制**，明确提醒接手 Agent 不要重复尝试。若当前会话接手自前序交接，保留并合并依然有效的历史避坑记录。
5. **⏳ 当前停顿点与下一步待办 (Next Steps)**：明确指出当前停在哪个具体文件、函数或待解问题上（若有需求矩阵，与矩阵中“🚧 进行中”的条目对齐），并按顺序列出下一步可直接执行的待办清单。
6. **🧪 验证与自测方案 (Verification)**：接手 Agent 完成修改后用于验证闭环的命令（如 `pytest`、`cargo test`、`npm test` 等）。

### 3. 写入交接快照与 `latest.md`

1. 检查 `<workspace>/.agent/handoff/` 目录是否存在，不存在则创建。
2. 若 `<workspace>/.agent/handoff/.gitignore` 不存在，创建并写入以下内容，防止交接快照污染业务版本库：
   ```gitignore
   # 忽略所有交接快照与本文件自身，不污染业务仓库
   *
   ```
3. 按下方报告模板写入 `<workspace>/.agent/handoff/<YYMMDD>-<HHMM>-<task>.md`。
4. 将**完全相同的内容**覆盖写入 `<workspace>/.agent/handoff/latest.md`（保留相同的 `snapshot` 字段，不使用符号链接）。
5. 目录中已有的旧快照文件保持原样，不回溯改名或删除。

#### 交接报告模板

```markdown
---
handoff_version: 1
created: "<YYYY-MM-DDTHH:MM:SS+08:00>"
snapshot: <YYMMDD>-<HHMM>-<task>.md
workspace: <工作区绝对路径>
source_agent: <agent 标识>
source_model: <model-id>
task: <task>
git: { branch: <branch>, head: <short-sha>, dirty: <true|false> }
transcript: <transcript 绝对路径，客户端未暴露时省略本行>
---
# Handoff: <task>

## 🎯 任务目标 (Goal)
<简述任务目标与交付标准>

## 📂 核心工作集 (Key Files & Working Set)
- `<相对路径>`: <该文件在任务中的角色或当前修改点>
- `<测试文件相对路径>`: <相关测试用例>

## ✅ 已完成事项 (Completed)
- <已完成修改与验证成果>

## 💡 关键决策与避坑指南 (Decisions & Dead Ends)
- **采用方案**: <为什么这样设计>
- **已踩坑/已排除**: <尝试过但失败的做法，提醒接手 Agent 不要重复该方案>

## ⏳ 当前停顿点与下一步待办 (Next Steps)
- 当前停顿点: <停留在哪个具体文件、哪行代码或哪个未解决的问题>
- [ ] 待办 1: ...
- [ ] 待办 2: ...

## 🧪 验证与自测方案 (Verification)
- `<自测执行命令，例如 pytest / npm test / cargo test 等>`
```

### 4. 输出接手引导 Prompt

文件写入完成后，在回复末尾用独立代码块输出以下引导 Prompt，供用户直接复制给下一个 Agent（若当前环境未暴露 `transcript` 路径，省略代码块中的“- **原会话记录**”及相关要求）：

````markdown
--- [请复制以下内容发送给接手 Agent] ---

我需要将以下开发任务交接给你继续推进：

- **任务目标**：<一句话简述在做什么>
- **当前进度**：<一句话简述做到了哪一步 / 停在哪里>
- **项目目录**：<workspace 绝对路径>
- **交接报告**：<workspace 绝对路径>/.agent/handoff/<YYMMDD>-<HHMM>-<task>.md
- **原会话记录**：<transcript 绝对路径，若客户端未暴露则省略本行>

**接手要求**：
1. 请先确认已安装 `dbk-pickup` skill，未安装时先运行 `npx skills add DBinK/dbk-handoff -g -y`。
2. 调用 `dbk-pickup` 读取交接报告，用简短几句话向我汇报你理解的现状与下一步计划。
3. 原会话记录仅作兜底，只在报告细节缺失时按需局部检索，切勿全量读取。
4. **汇报完毕后请直接暂停并等待我确认**，确认前禁止修改任何文件或执行非只读命令。

----------------------------------------
````

## 注意事项

- 只写入 `.agent/handoff/` 下的文件，不要对业务代码执行 `git add`、`git commit` 或 `git push`，除非用户明确要求。
- `latest.md` 与 `<YYMMDD>-<HHMM>-<task>.md` 的全文内容（含 frontmatter 的 `snapshot` 字段）必须完全一致。
- `created` 时间戳务必加上双引号，防止 YAML 解析器自动将其转换为非字符串对象。
