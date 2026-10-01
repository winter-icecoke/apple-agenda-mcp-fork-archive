# Apple Agenda MCP · 当前证据

更新：2026-10-01。此轮范围：Day 1 治理与交接，不涉及产品实现、签名发布或现用 MCP 切换。

## 仓库与范围

- Fork：[winter-icecoke/apple-agenda-mcp](https://github.com/winter-icecoke/apple-agenda-mcp)，upstream：[PsychQuant/che-ical-mcp](https://github.com/PsychQuant/che-ical-mcp)。fork / local 目录已改名；内部 Swift target / binary / MCP identity 保持上游。
- 开始基线：upstream / fork main `4c4d39632b2fae998bb9f25209b067b19ae64329`。这是 main 源码基线，不能当成 release 或本机正在运行的版本。
- 机器：`home-mac-mini-m4` / `Leos-Mac-mini-M4`。现用 MCP 是独立安装的 `mcp-server-apple-events@1.5.0`，本 fork 未接管其配置。
- 已定：Calendar + Reminders，公开 EventKit 覆盖与明确限制。产品能力矩阵、接口合同与模块拆分待新 session 决策。

## Linear 与交付

- 项目：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)，UUID `fe758034-a090-49bb-8759-4bf05c0b3d7e`，Team Special Force / SPE。
- 准备票：[准备 Apple Agenda MCP 的 Linear 联动、agent 分工和 Day 1 入口](https://linear.app/icecoke/issue/SPE-960)。状态和完成证据回读 Linear。
- 决策地图：[Apple Agenda MCP：日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。当前只有骨架，无产品子票或已决产品问题；新 session 首先 chart frontier。
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
