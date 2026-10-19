# Changelog

本项目所有显著变更记录于此文件。

格式基于 [Keep a Changelog](https://keepachangelog.com/zh-CN/1.1.0/)，条目按日期组织（个人项目，暂不采用版本号）。

## 2026-10-19

### Changed

- 收紧接手引导 Prompt 与 `dbk-pickup` 的行动边界（两个 skill 的 `metadata.version` 升至 `0.2.0`）：接手 Agent 读取报告并汇报现状与下一步计划后，默认必须暂停并等待用户确认，确认前禁止在同一轮直接修改文件或执行待办命令。

## 2026-10-18

### Added

- 初始版本：定义跨 AI Agent 上下文交接协议（`ARCHITECTURE.md`、`README.md`），采用 `.agent/handoff/` 纯文件存储、自闭环 `.gitignore` 与 `latest.md` 全量副本机制。
- 定义交接报告 YAML frontmatter 元数据规范（`handoff_version`、`created`、`snapshot`、`workspace`、`source_agent`、`source_model`、`task`、`git` 与可选的 `transcript`）。
- 新增发出端技能 `dbk-handoff`：采集环境与 Git 状态元数据，提炼任务目标、核心工作集、已完成进展、关键决策与死胡同避坑指南、停顿点与待办、验证方案，双写时间戳快照与 `latest.md` 并输出接手引导 Prompt。
- 新增接收端技能 `dbk-pickup`：按显式路径优先、`latest.md` 兜底规则读取交接快照，校验协议版本与 Git 工作区状态，执行渐进式加载与 Transcript 按需检索约束，向用户复述对齐后继续开发。
- 为 `dbk-handoff` 与 `dbk-pickup` 新增触发评估查询集 `evals/trigger-queries.json`（各 8 条正例、8 条边界负例）。
- 新增 `LICENSE`（MIT）与 `AGENTS.md`（仓库修改约定与校验命令）。
- 明确 `git.dirty` 的采集时机与口径：写入交接文件之前采集，并忽略 `.agent/` 路径，避免交接产物自身把工作区标记为脏。
