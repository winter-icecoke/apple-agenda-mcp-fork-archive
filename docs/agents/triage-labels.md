# 标签约定

沿用 Special Force 的现有单选组，不新增同义标签或改变其他项目标签：

- Type：Bug / Feature / Improvement / Tech Debt / Research / Chore，恰好一个。
- Area：Desktop（EventKit/TCC/原生能力）、Backend（MCP/领域服务）、Release（打包/签名/updater）、Docs（规则/证据）或确无单一主域时 Cross-cutting，恰好一个。
- Gate：Agent Ready / Human Input / External Blocker / Needs Info，零或一个。内部 Issue 依赖用关系。
- Wayfinder：map / research / prototype / grilling / task 分别使用同名 `wayfinder:*` 标签；它们补充工作方式，不替代 Type/Area/状态。

Bug 要有实际合同与复现；新能力用 Feature，既有能力改善用 Improvement，纯拆分/复杂度治理用 Tech Debt，答案未知用 Research，准备工作用 Chore。默认 No priority，不为占位票编造影响。
