# Agent 执行配置

参考 Voifolio 与 Vault Core 的共同分工，按本项目的 EventKit/MCP 范围调整。共同责任见 [AGENTS](../../AGENTS.md)。只描述派工偏好，不修改全局 model 配置。

## 主控与角色

主控沿用 Leo 在当前 session 选择的模型/effort，负责用户决定、Linear、基线、任务与文件所有权、独立审查、串行集成、凭据和发布。不固定启动全部角色。

| 执行角色 | 默认 Codex 模型 / effort | 交付物与边界 |
| --- | --- | --- |
| Research | `gpt-6.1-sol` / `xhigh` | 官方 API、源码、能力/限制矩阵，区分声明与实测；不写真实个人数据。 |
| EventKit / Domain | `gpt-6.1-sol` / `xhigh` | 领域合同、身份、重复系列、权限、原生数据行为；按已决接口实现。 |
| MCP / Interface | `gpt-6.1-sol` / `xhigh` | tools/schema、验证、错误与结果 envelope、版本兼容；不自行扩大语义。 |
| Verification | `gpt-6.1-sol` / `xhigh` | 风险与测试策略、集成/会话/真实回读及恢复证据；写真实对象由主控执行。 |
| Mechanical | `gpt-6-luna` / `max` | 条件明确的检索、固定验证与整理；小任务主控可直接做。 |
| Independent reviewer | `gpt-6.1-sol` / `xhigh` | 与作者独立，检查合同、数据/副作用、回归和复杂度；治理 diff 检查规则与链接。 |

Claude/Fable 主控保持原生运行时；技术研究/Swift 执行可用当前 Opus，MCP/领域和 review 使用真实可调用的 Codex 映射。不能把两套运行时设置混作全局配置。

真实派工传 `model`、`reasoning_effort`；指定型号用 `fork_turns=none` 或有限上下文，不使用 all。不可用时说明该票未执行；先询问替代方案，不静默降级或装框架绕过。

## 高风险状态流

重复系列/单次实例、批量写入、undo/redo、移动/恢复、异步身份变化与 TCC/签名/updater 的行为修改，先提交一页不变式与状态表，再实现。主控确认合同；独立 `gpt-6.1-sol / ultra` 审合同和完整行为链。其余角色仍用表中默认档位，不能因为一处复杂度给全队升档。

交错测试按风险覆盖在途时发生第二事件、对象已被用户修改/删除、部分写入失败、重试与恢复。若第二轮仍因回修出现新阻塞，返回合同层收敛，不持续加补丁。文档/模板修改不属于此产品高风险路径。

## 派工与文件所有权

使用 [派工模板](templates/agent-task.md)：目标/票、基线、文件范围、依赖、完成标准、验证、授权。共享工作区同文件只有一个 writer；按需 worktree，不强制一票一 worktree。subagent 不递归派工、不自行 commit/push，不用 stash/reset/checkout 覆盖他人改动。

返回实际改动、命令/退出状态、证据和未完成项。主控验证后集成，不能把自报成功视作验收。研究可并行；集成/发布队列每次一票；独立 review 与作者分开，跨文件/共享合同必须一起审。

## 授权与验证

本轮只准备工程治理与外部关联。后续真实写入、签名、安装和 MCP 切换按新 session 指令决定；不用其他项目的 production 权限推导本项目授权。参考 [validation](validation.md)。
