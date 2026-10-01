# Apple Agenda MCP

确认日期：2026-10-01。

## 目标与已确认范围

基于 `PsychQuant/che-ical-mcp` 维护 Calendar + Reminders MCP，全面覆盖公开 EventKit 可提供的能力，接口易于 agent 调用，并保持可持续的上游同步与发布。

Leo 已确认仅这两个应用；限制必须明确，不用 AppleScript 或私有 API/数据库补洞。重点调查地点/Maps 链接、重复规则、提醒、原生子任务与备注清单、分区/附件/邀请支持范围、批量、冲突/重复检测、撤销/恢复及可靠性。列入调查不代表已承诺公开 API 可以实现。

## 当前基线

- Fork：`winter-icecoke/apple-agenda-mcp`；上游：`PsychQuant/che-ical-mcp`。
- Fork 起点：`4c4d39632b2fae998bb9f25209b067b19ae64329`，上游 main，包含正式版之后的改动；不能把 main 与 release 混用。
- GitHub 项目和本地目录已更名；内部 Swift target、可执行文件和 MCP identity 仍沿用上游名字，变更方案待决定。
- 本机仍使用单独安装的 `mcp-server-apple-events@1.5.0`；此 fork 尚未替换现用 MCP，尚未验证其真实个人 Calendar/Reminders 写入。
- 既有对比发现 apple-events 的同日窗口与未来事件按 ID 查询缺陷；这只是新项目需回归的输入，不证明 fork 同样有缺陷。
- 上游已知批量移动通过复制再删除，ID/重复系列/参与者保留存在限制；研究、合同与实测需共同确定后续处理。

## 未决与完成标准

完整能力矩阵、MCP 工具分组/JSON 合同、拆分顺序、身份与 updater、签名/版本/回滚以及隔离数据验收均未决。用 [Linear 决策地图](https://linear.app/icecoke/issue/SPE-961) 决定，不能从本 brief 猜为实现要求。

产品验收应同时证明：能力对应公开 API 与明确限制；MCP 调用可判断结果；写入和重试不丢数据；二进制与长驻会话版本可确认；本机真实回读和本次测试对象清理有证据。具体合同在决策票中收敛。
