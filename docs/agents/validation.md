# 验证合同

## 选择验证面

- 文档/治理：链接、模板、字段、模型映射、秘密与范围边界；不运行真实 Calendar/Reminders 写入或无关全量 suite。
- Swift 行为：相关 unit/handler/dispatch 测试及既有 CI，必要时扩大；沿用上游测试命名及 narrow source seam，不复制实现写无效测试。
- Stateless CLI：显式 binary 路径、版本、commit，前后独立进程读回。
- Stateful MCP：同一长驻会话证明 undo/redo 状态，再独立系统回读。CLI 每调用一进程不能验 in-memory history。

继承的 [上游验证经验](../../.claude/rules/mcp-binary-version-testing.md) 是历史与风险输入；其“手动 stdio 回读不可靠”不能静默泛化为所有新协议实现的永久事实。遇到同类问题先复现并标明测试面。

## 必须区分的证据

官方 API / README、源码判断、mock 测试、binary build、安装、长驻会话版本、TCC 授权、真实写入回读、同步与用户验收分别记录。commit/merge/release 不证明当前 host 已换新 binary；替换文件不影响正在运行的 process。

本机测试用独立且可识别的集合与对象，记创建 ID，写前/写后分别读回，清理仅限本轮对象。不要在真实个人日程上试批量删除/移动，也不把个人内容带进 public repo。覆盖同日窗口、未来 ID、跨时区/全天、重复实例、部分失败/重试、外部修改与 undo/恢复的合同；具体范围由地图决策确定。

Capability 矩阵每项记录 read/create/update/delete、公开 API、项目 schema、版本、验证面、限制与来源；“未调查”不等于“不支持”，备注模拟不等于原生支持。
