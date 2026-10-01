# 推进地图

参考 Vault Core 的四块地图结构，适配本项目。HTML 是汇总视图；Linear map/children 是状态与依赖的唯一来源，repo evidence 保存过程事实。

## 载体与入口

- 可复用源：[progress-map.html](templates/progress-map.html)，零依赖、无外部请求、浅深色及窄屏。
- 当前：[全面覆盖推进地图](../progress/spe-961-agenda-coverage.html)。主票首行保存 GitHub 源文件链接；有本地/Artifact 预览时提供同一内容。
- 版本 `v<N> · 更新 <日期>`；同一轮多个触发合并为一次更新，静默期不刷版本。
- 模板占位符必须在实例中全部替换；未确定阶段工作量写“尚未定”，不编造百分比。

## 四块与触发

1. 整体方案：目标、决策→合同与拆分→分期实现/验收，已有/待补能力。
2. 进度：决策标题链接与 gist、执行者、基线/PR、CI、review、待验项和需要 Leo 的事项。
3. 后续：真实原生依赖、批次验收与集成顺序；只在未授权或 Leo 明确要求等待时标暂停点。
4. 更新节点：派工、交回、集成/验收、阻断、批次边界、方案变化。

统计按“完成标准已满足且有证据的项目数 / 当前阶段总项目数”，范围变化同步分母。独立表达本地/commit/push/merge/build/签名/release/安装/MCP 会话/真实回读/用户验收；不用 merge 替代验收。

状态类 done/run/wait/you/alert，文字写具体事实。对 Leo 用票标题包链接，不裸列编号。决策地图只索引闭合决策；HTML 可列当前 open tickets，不把开放列表搬到 Linear map body。
