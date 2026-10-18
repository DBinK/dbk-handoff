# dbk-handoff

> **跨 AI Agent 的通用上下文交接与接力协议 (Universal AI Agent Context Handoff Protocol)**

`dbk-handoff` 是一套极轻量、免复杂配置、跨任何 AI 编程客户端（Antigravity, Claude Code, Cursor, Windsurf, OpenHands 等）通用的上下文交接工具方案。

它通过 **`dbk-handoff`（导出交接）** 与 **`dbk-pickup`（接手继续）** 两个配套 Skill，实现多 Agent 间的无缝切换与渐进式上下文传承。

---

## 核心设计理念

1. **普适无依赖**：纯 Markdown + 文件协议，任何具备读写文件能力的 Agent 均可使用，不需要复杂的 RPC 服务或外部中转服务器。
2. **渐进式加载（Token 友好）**：杜绝盲目倾倒数万 Token 的完整聊天记录；优先消费高密度的交接摘要（目标、已完成、决策坑点、下一步），必要时才按需回溯历史细节。
3. **人类确认环（Human-in-the-loop）**：通过一段格式规范、一键复制的 Prompt 完成交接，方便人类随时审计、补充或纠偏。

> **灵感与设计来源 (Inspiration & Credits)**：
> 本项目的结构化交接模板与设计哲学深受 **[`can1357/oh-my-pi`](https://github.com/can1357/oh-my-pi)** (omp) 项目及其 `/handoff` 机制启发。特别吸纳了 omp 中对 **核心工作集（Key Files / 8~15 文件上下文）**、**已排除的死胡同方案（Dead Ends 防重蹈覆辙）** 以及 **验证闭环（Verification Plan）** 的严格定义，从而彻底杜绝新 Agent 的“代码考古（File Archaeology）”消耗。

---

## 目录与文件规范

在工作区内自动维护如下结构：

```text
<workspace-root>/
└── .agent/
    └── handoff/
        ├── .gitignore                        # 自动生成：开箱即用，避免污染版本库
        ├── latest.md                         # 最新交接快照（支持免参接手）
        ├── 261018-1230-auth-refactor.md      # 历史快照 1
        └── 261018-1420-fix-token-expiry.md   # 历史快照 2
```

- **文件名规范**：`<YYMMDD>-<HHMM>-<简短主题>.md`（如 `261018-1230-auth-refactor.md`）
- **时间基准**：写笔记时的本地时间。
- **命名要求**：全小写英文，短横线连接，旧文件永不回溯改名。

---

## 快速使用

### 0. 安装 Skills (推荐)
如果你使用的是支持 skills 生态的工具，可通过以下命令一键全局安装：
```bash
npx skills add DBinK/dbk-handoff -g -y
```

### 1. 发出交接：在当前 Agent 中执行
调用 `dbk-handoff`（或直接提示 Agent 总结交接）：
- 自动提炼核心目标、进度、卡点和下一步计划。
- 自动写入 `.agent/handoff/<YYMMDD>-<HHMM>-<slug>.md` 并同步至 `latest.md`。
- 输出一段带有一键安装指引与报告路径的格式化 Prompt。

### 2. 接手任务：在新 Agent 中执行
复制前一个 Agent 吐出的引导提示词，在新 Agent 窗口粘贴并回车。
- 若新 Agent 尚未安装，提示词会引导运行 `npx skills add DBinK/dbk-handoff -g -y` 完成前置安装。
- 新 Agent 调用 `dbk-pickup` skill：
  - **有引导 Prompt 时**：精准读取提示词中指定的 `.agent/handoff/<YYMMDD>-<HHMM>-<简短主题>.md`；
  - **无引导直接敲 `/dbk-pickup` 时**：默认读取 `.agent/handoff/latest.md` 作为兜底；
- 读取后向你确认理解并开展下一步。

---

## 详细架构文档

更多规范与协议细节，请参阅 [ARCHITECTURE.md](file:///C:/it/dbk-handoff/ARCHITECTURE.md)。
