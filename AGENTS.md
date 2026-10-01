# Apple Agenda MCP

本 fork 的主入口。面向用户默认简体中文。`CLAUDE.md` 保留 Claude 路由与上游技术约定；共同规则以本文件及 `docs/agents/` 为准。

## 产品边界

- 范围仅 Apple Calendar + Reminders，覆盖公开 EventKit 能力；不使用 AppleScript、私有 API 或 Calendar/Reminders 私有数据库。无法实现的能力明确返回不支持，不能以备注模拟冒充原生功能。
- 日程、重复系列、单次实例、提醒事项、原生子任务、备注清单和地点链接必须区分；术语见 [CONTEXT](CONTEXT.md)。
- 先调查能力矩阵与合同，再分批实施。保持上游兼容，具体能力与拆分方案尚未决；见 [项目范围](docs/PROJECT_BRIEF.md)。
- 个人日历/提醒内容只用于获授权的本机验证，不写入 public repo、Linear 或 PR。测试使用可识别的独立测试集合，记录创建对象 ID 并清理本次对象。

## 主控与执行者

- 主会话沿用 Leo 选择的模型与推理档位，负责范围、决策、拆解、派工、复核、串行集成和 Linear 收尾。
- 研究、领域/EventKit 实现、MCP 接口、验证与独立 review 按文件和依赖划分派给 subagent；不固定启动全队。可用能力与模型映射见 [agent-execution](docs/agents/agent-execution.md)。
- 执行者沿用主控分配的 Issue、基线和工作区，不递归派工，不自行 commit/push；共享文件的写入串行，必要时隔离 worktree。
- 凭据、Git 写入、发布、签名、配置切换和真实 Calendar/Reminders 写入由主控按已有授权串行处理。
- 作者与 reviewer 独立。此次治理引入后，合 main 前要求独立 review；docs-only 只核对治理、链接、模板和敏感信息，不为它运行产品写入或全量 Swift 测试。

## Linear 与交付

- 唯一活跃 tracker：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)，Team `Special Force` / `SPE`。查询必须限定本项目，不创建第二套 GitHub execution issues，不修改共享 team 自动化。
- 新 Issue 默认 Leo Liu、Backlog、No priority，恰好一个 Type/Area，最多一个 Gate。内部依赖使用原生 `blockedBy`/`blocks`，不以正文或 Gate 代替。
- 分支含 `spe-<编号>`；commit 使用 Conventional Commits、本机 `/Users/leolau/.gitmessage.txt`（便携副本 `.gitmessage.txt`）及 `Refs SPE-*`，不用 closing keywords。PR title 含 `[SPE-*]`。
- 本 fork 的代码/治理变更通过 branch + PR 交付，独立 review 后由主控按授权合并。PR、commit、merge、发布、本机安装与用户验收分别回读；满足本票完成标准和证据才 Done。
- tracker、状态与 GitHub 链路见 [issue-tracker](docs/agents/issue-tracker.md)、[联动](docs/agents/linear-github.md)、[标签](docs/agents/triage-labels.md)。

## Day 1 / Agent skills

- 新 session 先读 [Day 1](docs/agents/day-1.md)，再读取 Linear 的[全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。
- 大需求与 Wayfinder 开工前维护 [推进地图](docs/agents/progress-map.md)。Linear map 是决策索引；HTML 仅汇总可视化，状态/依赖以 Linear 为准。
- `$wayfinder` 使用 `grilling` + `domain-modeling`，按地图一次解决一个非研究决策；研究按 skill 派 subagent。当前 Day 1 只准备入口，产品 frontier 留给新 session。
- 领域布局为单 context：根 `CONTEXT.md` 只保存术语；ADR 按真实取舍惰性创建，规则见 [domain](docs/agents/domain.md)。
- 当前证据写 [PROGRESS](docs/_meta/PROGRESS.md)，长期决定写 ADR；不在地图、README、历史台账复制完整票据状态。
- 上游 Spectra/openspec 记录保留以便追溯。用户指定 Wayfinder 的新工作按上述流程，不因继承的 Spectra block 要求另起重复规划。

## 验证与授权

- 验证按变更影响选择；现有检查通过后，只有新变更、失败或疑点才扩大或重复验证。清理本次无用临时文件。
- 测试先确认二进制路径、版本、提交、MCP 会话与 TCC 权限；不把源码 build、旧会话调用或新文件替换当成新版已经生效。见 [validation](docs/agents/validation.md)。
- `--cli` 不证明跨调用的 undo/redo；状态性功能须同一长驻 MCP 会话及独立系统回读。
- GitHub/Linear token、webhook secret、签名证书留在本机安全存储；仓库只保存检索指针，不能轮换共享 webhook 凭据来绕过本机授权问题。
- 本次准备没有授权产品开发、覆盖现用 MCP 或在真实个人对象上做破坏性批量验证；后续范围按 Leo 新 session 指令执行。

---

<!-- SPECTRA:START v1.0.1 -->

# Spectra Instructions

This project uses Spectra for Spec-Driven Development(SDD). Specs live in `openspec/specs/`, change proposals in `openspec/changes/`.

## Use `$spectra-*` skills when:

- A discussion needs structure before coding → `$spectra-discuss`
- User wants to plan, propose, or design a change → `$spectra-propose`
- Tasks are ready to implement → `$spectra-apply`
- There's an in-progress change to continue → `$spectra-ingest`
- User asks about specs or how something works → `$spectra-ask`
- Implementation is done → `$spectra-archive`

## Workflow

discuss? → propose → apply ⇄ ingest → archive

- `discuss` is optional — skip if requirements are clear
- Requirements change mid-work? `ingest` → resume `apply`

## Parked Changes

Changes can be parked（暫存）— temporarily moved out of `openspec/changes/`. Parked changes won't appear in `spectra list` but can be found with `spectra list --parked`. To restore: `spectra unpark <name>`. The `$spectra-apply` and `$spectra-ingest` skills handle parked changes automatically.

<!-- SPECTRA:END -->
