# Apple Agenda MCP · 当前证据

更新：2026-10-01。当前轮范围：Wayfinder Work through 的研究核对、首个 HITL 对谈与地图同步；不涉及产品实现、签名发布或现用 MCP 切换。下方 Chart / Day 1 保留各轮历史边界，实时票状态以 Linear 为准。

## Work through · 2026-10-01

- 三份研究经主控与独立 `gpt-6.1-sol / xhigh` reviewer 核对，票内 resolution 后 Done：[Calendar 公开边界](https://linear.app/icecoke/issue/SPE-966)、[Reminders 原生结构与地点边界](https://linear.app/icecoke/issue/SPE-967)、[MCP 合同与运行发布基线](https://linear.app/icecoke/issue/SPE-968)。报告分别为 [Calendar](../research/spe-966-calendar-eventkit.md)、[Reminders](../research/spe-967-reminders-location-eventkit.md)、[MCP/运行](../research/spe-968-mcp-runtime-baseline.md)；每项保留 API / 源码 / MCP / 验证 / 未知分层。
- 报告集成并 push：`97770a1`、`441e081`、`b67f0d0`；[Draft PR](https://github.com/winter-icecoke/apple-agenda-mcp/pull/2) 保持开放，未 merge。独立 review 两处 Calendar 与一处重复实例限定建议已修正；不把研究完成写作产品验收。
- 固定研究基线仍为 `3ad6408`；upstream main `4c4d396` 比正式 release `v1.18.0 / 7371f99` 多 5 个提交，版本字符串同为 1.18.0。当前工具数 29，结果/分页、copy+delete、部分失败、内存 history50 和不完整 redo 均为源码事实；未复现运行行为。
- 本机只读特定配置路径及包元数据确认 `mcp-server-apple-events@1.5.0`；09:18:24 过滤进程快照无匹配。实际活跃会话、binary commit、TCC、同步、签名与恢复仍未知；本 fork 未安装或接管现用配置。
- 领取前复读当前关系/评论：覆盖口径只决定能力纳入与限制分类，两份原生 API 报告已包含当前 MCP 字段；独立 reviewer 同意移除其对运行基线研究的非必要 blocker。原生依赖由 23 调整为 22，原因留在 [覆盖口径票](https://linear.app/icecoke/issue/SPE-969) 评论，其他下游依赖保留。
- 本次唯一非研究决策 [决定公开能力的覆盖口径与受限能力表达](https://linear.app/icecoke/issue/SPE-969) 已获 Leo 实际回答“按这个办”。先以预约、共享日历、邀请、商品链接、到店通知和原生子任务场景解释影响，再记录票内 resolution 并 Done、移除 Human Input；CONTEXT 只同步已确认术语。无产品实现、无新 ADR 或新增 child；时间/重复与受限原生功能留给下轮。
- HTML v5 同步三项研究与一项已确认决策、22 条直接依赖及下一 frontier；Linear map 只保留闭合索引，结论详情在各票 resolution。本次收口不领取第二张 HITL；当前同步提交和检查证据在 map 评论。
- v5 的四文件 diff 经独立 `gpt-6.1-sol / xhigh` review 无阻塞；四块/14 行、标题嵌套、26 个本地链接出现、敏感模式与 whitespace 检查通过，CSS 未改。当前只确认产品承诺，不把源码/保存结果推为实际效果验收。
- v4 同步 diff 独立 `gpt-6.1-sol / xhigh` review 无阻塞；HTML 四块/14 行、标题层级/嵌套、26 个本地链接出现、22 条依赖/领取快照、敏感信息与 whitespace 检查通过。CSS 与已查 Chart 版本一致；浏览器显示检查保留 Chart 轮事实，不冒充本轮产品或运行验收。
- docs-only：未运行 Swift suite、build、真实对象、权限弹窗/reset、签名/release/install 或 MCP 切换。规划阶段已关闭 4/14（3 研究 + 1 决策），产品覆盖分母仍未定。

## Chart · 2026-10-01（收口时快照）

- 实时开工基线：`Leos-Mac-mini-M4.local`，本 checkout `main@3ad64088feb82104982b6b3e7181f6bb4ca8af94`，开工前工作区干净；未见其他活跃 session 占用本仓库。fork main 同 SHA，upstream main `4c4d39632b2fae998bb9f25209b067b19ae64329`；上游正式 release [v1.18.0](https://github.com/PsychQuant/che-ical-mcp/releases/tag/v1.18.0)，2026-09-08 发布。正式 tag 不当作 main 或本机版本。
- 地图原为骨架、无评论和 child。沿用已确认 Destination，使用 Wayfinder + grilling + domain-modeling 广度拆分：创建 3 张 AFK research、11 张 HITL grilling，共 14 张 children；第二遍连接 23 条原生依赖，逐票回读 project/parent/labels/assignee/relations 并核对无环。未解决或关闭产品决策，阶段已关闭 0/14，不是产品覆盖比例。
- 研究主控 claim 为 Leo Liu、In Progress、移除 Agent Ready；未领取 HITL 为 Backlog、No priority、assignee=null。全部恰好一个 Type/Area；Gate 不替代内部依赖。
- 三路 Research 使用 `gpt-6.1-sol / xhigh`，各自独立 worktree 与唯一 Markdown 文件；实际 branch/workspace/资产 pointer 留在票评论：[调查 Calendar 与来源、时间及重复日程的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-966/调查-calendar-与来源时间及重复日程的公开-eventkit-边界)、[调查 Reminders 原生结构、地点与提醒字段的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-967/调查-reminders-原生结构地点与提醒字段的公开-eventkit-边界)、[调查当前 MCP 合同、状态行为与 fork 发布运行基线](https://linear.app/icecoke/issue/SPE-968/调查当前-mcp-合同状态行为与-fork-发布运行基线)。三路均已派出。研究资产当前本机 WIP，未核对或发布；执行者不递归派工、不 commit/push。
- 后续研究满足标准并关闭后，首层可推进问题为：[决定公开能力的覆盖口径与受限能力表达](https://linear.app/icecoke/issue/SPE-969/决定公开能力的覆盖口径与受限能力表达)、[决定全天、时区与重复系列的时间和修改范围合同](https://linear.app/icecoke/issue/SPE-970/决定全天时区与重复系列的时间和修改范围合同)、[决定原生子任务、分区、附件与邀请等受限功能的产品边界](https://linear.app/icecoke/issue/SPE-972/决定原生子任务分区附件与邀请等受限功能的产品边界)；下一 session 按原生关系与创建顺序领取一张。当前不存在 unblocked、unclaimed 的 HITL frontier。
- 独立规划与文档 diff 复核 `gpt-6.1-sol / xhigh` 均完成，无阻塞；提醒 CRUD/重新打开/时间字段与时间提醒依赖缺口已纳入。21 个本地链接、HTML 四块/14 票/结构/无占位、敏感信息与 whitespace 检查通过；浏览器显示核对通过。设计 hook 的低对比问题已修正，浅/深色正文最低对比度 4.71/6.13，未忽略或遗留检查项。
- HTML v3、Day 1 和 tracker 入口同步，map 开放清单只在 children；Decisions so far 仍为空；Fog 仅留研究后才能明确的新情景和 prototype。
- 本地 branch `docs/spe-961-chart`；本轮 commit/push/PR 的实际回读以 canonical map 的 Chart 收口评论为准，此快照不证明 merge。docs-only 不运行 Swift suite、产品 build、签名/release/install、新 MCP 会话或真实写入；本机现用 binary/会话本轮尚未核对。
- 停止点：按 Leo 要求完成 Chart 并启动研究后结束，不 claim/解决非研究票，不进入产品实现。研究报告交回后由主控核对，再在票内 resolution；不能把建票/派工冒充研究或产品验收。

## 仓库与范围

- Fork：[winter-icecoke/apple-agenda-mcp](https://github.com/winter-icecoke/apple-agenda-mcp)，upstream：[PsychQuant/che-ical-mcp](https://github.com/PsychQuant/che-ical-mcp)。fork / local 目录已改名；内部 Swift target / binary / MCP identity 保持上游。
- 开始基线：upstream / fork main `4c4d39632b2fae998bb9f25209b067b19ae64329`。这是 main 源码基线，不能当成 release 或本机正在运行的版本。
- 机器：`home-mac-mini-m4` / `Leos-Mac-mini-M4`。现用 MCP 是独立安装的 `mcp-server-apple-events@1.5.0`，本 fork 未接管其配置。
- 已定：Calendar + Reminders，公开 EventKit 覆盖与明确限制。产品能力矩阵、接口合同与模块拆分待新 session 决策。

## Linear 与交付

- 项目：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)，UUID `fe758034-a090-49bb-8759-4bf05c0b3d7e`，Team Special Force / SPE。
- 准备票：[准备 Apple Agenda MCP 的 Linear 联动、agent 分工和 Day 1 入口](https://linear.app/icecoke/issue/SPE-960)。状态和完成证据回读 Linear。
- 决策地图：[Apple Agenda MCP：日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。Day 1 准备轮仅有骨架；当前 Chart 与下一 frontier 见上方证据及 Linear 实时 children。
- GitHub autolink 已创建。push-only webhook active / JSON / SSL verification；ping 和真实 push delivery 均 200。凭据复用现有安全存储，仅检索指针进入治理文档。
- 实际 [准备提交](https://github.com/winter-icecoke/apple-agenda-mcp/commit/13596da9641197ff4f393134274404054a910350) 和 [Day 1 准备 PR](https://github.com/winter-icecoke/apple-agenda-mcp/pull/1) 已自动附在正确准备票，未手工补附件。
- 观测状态：初始 commit push → In Review；draft PR → In Progress。这是共享自动化的真实结果，不等于完成或产品验收；后续 ready / merge / 手动收尾的回读留在准备票评论。

## 本轮验证

| 项目 | 实际结果 |
|---|---|
| 文档入口、角色和模板 | 42 个本地链接有效；实际 HTML 无未替换占位符或外部请求，浏览器显示核对通过；staged diff whitespace 和凭据模式检查通过 |
| GitHub push webhook | ping 200；真实 Refs commit push 200，Linear 自动 commit attachment 已回读 |
| 实际 PR 自动关联 | PR #1 自动 attachment 已回读；证明现有 GitHub App 覆盖本 fork |
| 独立 review | 独立 gpt-6.1-sol / xhigh 完成；Chart / Work through 混用已修正，修复后无阻塞发现 |
| commit / push / merge | 初始 commit 13596da 已推送；其后修复与证据随 PR #1 集成。最终合并 SHA / main 回读见准备票收尾评论与 PR 实时状态 |
| GitHub CI | PR 初始 statusCheckRollup 为空、fork workflow API 未列出 workflow；未将此视为通过。docs-only 本轮不运行产品 Swift build/test |
| 产品 build / release / install / 新 MCP 会话 | 此轮未执行 |
| Calendar / Reminders 真实写入和用户验收 | 此轮未执行 |

## 续接

使用 [Day 1 prompt](../agents/day-1.md)，读取 Linear 实时 frontier 和 [HTML 地图](../progress/spe-961-agenda-coverage.html)。现用 apple-events 的同日查询窗口与未来事件查找问题，以及 che-ical move 的 copy+delete 行为，作为调查输入，不能未经复验推广成 fork 当前行为。
