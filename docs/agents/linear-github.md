# Linear 与 GitHub 联动

核对日期：2026-10-01。参考 Bagel House、Vault Core、Voifolio 当前文件；本仓库执行不依赖它们存在。

## 项目变量

- GitHub：`winter-icecoke/apple-agenda-mcp`，default branch `main`。
- Upstream：`PsychQuant/che-ical-mcp`；origin/upstream 分开。
- Linear：[Apple Agenda MCP](https://linear.app/icecoke/project/apple-agenda-mcp-066ab728e33d)，workspace `icecoke`，team `Special Force/SPE`，project UUID `fe758034-a090-49bb-8759-4bf05c0b3d7e`。
- 关联准备：[Day 1 入口](https://linear.app/icecoke/issue/SPE-960)；产品地图：[全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。

## 关联合同

- commit footer `Refs SPE-<n>`；不用 Fixes/Closes 等 closing keywords。
- 非 main 分支含 `spe-<n>`；PR title `[SPE-<n>] ...`，正文 `Refs SPE-<n>`。
- GitHub autolink：`SPE-` → `https://linear.app/icecoke/issue/SPE-<num>`，数字编号。
- PR linking 由现有 Linear Code GitHub App 处理；是否覆盖本 fork 需本仓库真实 PR attachment 证明，不能仅沿用旧项目记录。
- commit linking 用本仓库的 push-only webhook：active、JSON、SSL verification。不会继承其他个人账号 repo 的 webhook。
- 凭据仅复用 1Password `Linear GitHub Commit Webhook — icecoke`（item `quhfkrwpnbl27i3xygfbihr2jy`）；不把 password/endpoint 写入 repo/Issue/PR/日志。
- 不改共享 team 自动化，不关闭再开启 workspace commit linking；该操作会轮换共享凭据并影响其他项目。
- GitHub Issues sync 保持不启用；autolink/手动链接不证明自动 linkage 成功。

## 验收

1. 本仓库 hook ping delivery 2xx。
2. 一次真实 `Refs` commit push delivery 2xx，并在正确 Linear Issue 自动出现 commit attachment。
3. 真实 draft PR 自动关联、ready/merge 状态回读；`Refs` 不等同于 Done。手工补 attachment 不冒充自动关联。
4. 合 main 后核对 commit、PR、清洁工作区与 repo/Linear 项目资源。产品部署、安装、真实 EventKit 验收另行证明。

准备与验证结果写 [PROGRESS](../_meta/PROGRESS.md) 和准备票，缺项明确保持待验证。

来源：[Linear 官方 GitHub 文档](https://linear.app/docs/github)。
