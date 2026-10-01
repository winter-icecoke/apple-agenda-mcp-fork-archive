# Issue tracker: Linear

- Workspace：`icecoke`；Team：`Special Force` / `SPE`（`a2854931-4021-45fa-bfcc-7c1525ecd13c`）。
- Project：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)（`fe758034-a090-49bb-8759-4bf05c0b3d7e`）。
- 默认负责人：Leo Liu（`da144ef5-1d32-4842-bf20-793545fcc8c0`）。
- 所有列表/检索限定本 project；决策地图 child 未领取时 `assignee=null`，不能将默认负责人误作已领取。
- Linear 保存当前任务、状态、依赖和完成标准；GitHub 保存代码/PR；repo 保存合同、研究和证据。GitHub Issues 不作为活跃执行 tracker，不启用双向同步。

## 状态与字段

新票 Backlog + No priority，Type/Area 各一个，Gate 最多一个，cycle/estimate/due date 留空。真实开始时 In Progress 并移除 Agent Ready；停笔待 review/验证时 In Review；完成标准与证据满足后 Done。长期等待用 Backlog + 实际 Gate。引用词、分支与 PR 可能触发共享状态流转，不能把它当作验收结论。

修改前读当前值；状态变化写短评论和 evidence pointer。内部依赖用原生 `blockedBy`/`blocks`。创建超时先按 project、精确标题、创建时间查重。

## Wayfinding operations

- Canonical map：[日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)，label `wayfinder:map`。模板见 [map](templates/wayfinder-map.md)。当前只有 Day 1 skeleton，尚未 chart frontier。
- Chart 与 Work through 分开：首次 chart 建票、第二遍连依赖、启动 research 后停止，不手动解决非研究票。map 默认 planning-only；只有 Notes 明确覆盖时，执行才进入 map，不能把普通功能实现伪装成决策前置 task。
- Child：用 `parentId` 指向 map，Type=Research（决策调查）或实际 task 类型，附 `wayfinder:research/prototype/grilling/task` 之一；不要创建第二张同义 map。
- Claim：先读票/评论/relations，设 assignee 为实际主控用户并 In Progress；再开始工作。map 的 owner 不等于每个 child 的 claim。
- Blocking：所有 child 创建完后，第二遍设置 `blockedBy`/`blocks`。
- Frontier：查询 `project + parentId + 非终态 + assignee=null`，逐票读 relations，仅选择所有 blocker 都终态且已满足解除条件的 child。排序采用地图约定/依赖顺序，不靠标签猜测。
- Resolve：答案写 resolution comment，满足决策完成标准才关闭；map 的 Decisions so far 只追加带标题的链接与一句 gist。资产指向 repo；不在 map 重复完整推导。
- Fog 已能精确成问题时创建新 child；超出 destination 的票 Canceled，索引写 Out of scope，不写 Decisions so far。
- 每 session 最多解决一个非研究票；HITL 由 Leo 实际回答，不能让 agent 代答。只读/研究按 skill 的例外与派工规则执行。

## 名称与技能

对 Leo 的说明使用票的标题包住链接，不裸列编号。`publish to issue tracker` 指本 Linear project；fetch ticket 需正文、评论、parent/children、labels 和 relations。PR 是交付载体；外部 PR 不自动作为需求入口。
