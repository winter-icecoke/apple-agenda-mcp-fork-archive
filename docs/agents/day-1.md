# Day 1：新 session 入口

准备日期：2026-10-01；Work through 更新：2026-10-01。Chart、三项首轮研究、覆盖口径与时间/重复合同已闭合；Leo 对五组业务场景实际回答“按建议”，合同与不变式见 [时间与重复合同](../contracts/spe-970-time-and-scope.md)。下轮按创建顺序先核对地点/提醒主题的可领取状态；原生结构和查询身份同样须实时核对。先复读 claim、评论与原生关系，不重复 Chart、不代答 HITL；同一主题中已能判断的问题集中成一轮，当前证据见 PROGRESS。

## 位置与已定边界

- 本机：`home-mac-mini-m4` / `Leos-Mac-mini-M4`；checkout：`/Users/leolau/Documents/Projects/apple-agenda-mcp`。换机器先重新确认路径。
- Fork：[winter-icecoke/apple-agenda-mcp](https://github.com/winter-icecoke/apple-agenda-mcp)。`origin` 指向 fork，`upstream` 指向 [PsychQuant/che-ical-mcp](https://github.com/PsychQuant/che-ical-mcp)。
- Linear：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)，Team Special Force / SPE，project UUID `fe758034-a090-49bb-8759-4bf05c0b3d7e`。
- 决策地图：[Apple Agenda MCP：日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。治理准备票：[准备 Apple Agenda MCP 的 Linear 联动、agent 分工和 Day 1 入口](https://linear.app/icecoke/issue/SPE-960)。
- 目标：公开 EventKit 的 Calendar + Reminders 能力尽量全面覆盖，MCP 易发现、易调用，限制明确返回。当前内部 binary / Swift target / MCP identity 仍沿用 upstream，改名是待决策项。
- 本机配置入口与安装包元数据确认 `mcp-server-apple-events@1.5.0`；活跃会话仍未验证。本 fork 尚未安装或接管现用 MCP；源码、安装、会话和测试证据以 [PROGRESS](../_meta/PROGRESS.md) 为准。

## 开工顺序

1. 核实机器、`git status`、branch / HEAD / remotes 和其他 session 占用。读取根 `AGENTS.md`、[PROJECT_BRIEF](../PROJECT_BRIEF.md)、[agent-execution](agent-execution.md)、[issue-tracker](issue-tracker.md)、[linear-github](linear-github.md)、[validation](validation.md)、[CONTEXT](../../CONTEXT.md) 与当前 PROGRESS。
2. 使用 `$wayfinder`，读取当前 skill 原文及其 `grilling`、`domain-modeling` 引用。读取 Linear map、comment 和本项目未终态子票。只有骨架、没有子票时进入 Chart 模式：建 frontier、连依赖、启动研究后结束本 session，不手动解决非研究票。已有 frontier 时，另一个 Work through session 才领取一张可推进的票；领取前核对原生依赖。
3. 把需求拆成具体的决策问题，识别 HITL / AFK、研究前置与原生 `blockedBy` / `blocks`。研究按 skill 委派；主控沿用 Leo 选择的模型，角色与模型映射按 agent-execution。
4. 用能力矩阵区分公开 API 支持、当前实现、MCP 合同与验证证据。用真实结论更新 map 索引、CONTEXT 术语和需要的 ADR；未知保留未知。
5. 按 [推进地图规则](progress-map.md) 更新 [HTML 地图](../progress/spe-961-agenda-coverage.html)。本 map 默认只做规划，路线清楚后交接实施计划。后续经明确指令进入实现/验证批次时，要有完成标准、执行者、review 与结果回读。Work through 每 session 最多一个非研究决策，不一次固定启动全队。

## 可直接复制给新 session 的 prompt

```text
在 /Users/leolau/Documents/Projects/apple-agenda-mcp 工作，继续 Apple Agenda MCP。

先读取 AGENTS.md、docs/agents/day-1.md、docs/PROJECT_BRIEF.md、docs/_meta/PROGRESS.md，以及 day-1 指向的 agent execution、Linear、验证和领域规则；核实当前机器、checkout、HEAD、remotes 和未提交变更。

按 $wayfinder 继续正式规划。Linear 项目是 Apple Agenda MCP（UUID fe758034-a090-49bb-8759-4bf05c0b3d7e，Team Special Force / SPE），canonical map 是「Apple Agenda MCP：日历与提醒全面覆盖决策地图」https://linear.app/icecoke/issue/SPE-961。先回读地图、评论和当前子票。2026-10-01 已完成 Chart 建票和原生连边；已有 children 时不要重复 Chart，先核对研究是否已满足完成标准、原生依赖是否解除。研究还未关闭时，继续核对已派研究的证据或精确缺口，不代答其下游 HITL 问题。只有实际 unblocked、unclaimed 的 frontier 才进入 Work through，领取一张可推进的非研究票并遵守每 session 上限。若实时回读确实只有骨架、没有 child，才做 Chart：确定 destination、广度优先拆问题、建 child、第二遍连原生依赖、按 skill 启动 research 后停止。

已定范围仅 Calendar + Reminders，全面调查公开 EventKit 能力；不使用 AppleScript、私有 API 或 Calendar/Reminders 私有数据库。无法实现的能力明确返回不支持。MCP 调用要容易发现、输入合同明确、错误能处理、结果能回读。

调查应覆盖日历/清单与来源、日程/提醒 CRUD、地点文字与原生 structured location、Apple Maps URL、URL/notes、全天与时区/DST、重复规则及单次/未来/整个系列修改、时间/地点提醒、优先级/完成状态、批量/部分失败、冲突/去重、undo/redo、查询窗口/分页/标识符可靠性、TCC/签名/会话版本，以及原生子任务、分区、附件、邀请参与者等公开 API 的真实边界。用能力矩阵逐项给 API 证据、当前实现、MCP 暴露方式、验证层级、限制与未知；备注清单不能当原生子任务，Maps URL 不能当原生 POI。

把模块拆分、兼容旧工具名称、内部产品改名、上游同步、self-update 的 fork 目标、release/签名/安装与真实 MCP 会话验证纳入决策；拆分须由代码和调用合同决定，不为形式预建多个仓库。先确认 upstream main 与 release、fork main 和实际安装 binary 的区别。

主控沿用我选定的模型；领域/EventKit、MCP 接口、研究、验证、机械整理与独立 reviewer 按 docs/agents/agent-execution.md 分工。共享文件单写者，Git/凭据/发布串行，作者与 reviewer 独立。高风险状态流先写 invariant，再按规定做独立 review。

决策票挂在 canonical map 下，用已有 Type / Area / Gate / wayfinder 标签和原生依赖；引用用票名加链接。完成结论写 comment，地图只维护索引与结论摘要。同步 docs/progress/spe-961-agenda-coverage.html，保留整体方案、进度、后续批次和更新触发四块；不要用准备工作完成比例冒充产品覆盖比例。

推进当前模式已获授权的工作；需要我决定的 HITL 问题集中提出，并继续不依赖答案的研究。每次说明已决问题、仍待决定、下一 frontier 和实际证据。此 map 默认 planning-only，路线清楚时交付实施计划；不要把决策票当功能实现票，或未经 Notes 明确覆盖就顺带开发。以后按具体实施指令提交时，使用 Conventional Commits + 完整模板 + Refs SPE-*，经独立 review 用 PR 集成。现用 apple-events MCP 尚未切换；未来切换前按已有指令确认具体目标与实际会话生效，真实个人数据不进入 public repo / Linear / PR。
```

## Day 1 已备模板

- [Wayfinder map](templates/wayfinder-map.md)、[决策票](templates/wayfinder-ticket.md)：问题、领取、依赖、结论。
- [Agent 派工](templates/agent-task.md)：模型、文件所有权、输入、完成与验证、返回证据。
- [能力矩阵](templates/capability-matrix.md)：真实 API、实现、合同、未知与限制。
- [推进地图 HTML](templates/progress-map.html)：自包含，可本地打开，跟随系统深浅色。
- [.gitmessage](../../.gitmessage.txt)、[PR template](../../.github/pull_request_template.md)：治理与实现共用交付入口。

规则参考 BagelHouse、Vault Core 和 Voifolio 的本机 canonical 文档；采用最近 Voifolio 的角色映射，推进地图结构参考 Vault Core。没有修改这些项目或 SPE team 的共享自动化。
