# Reminders 原生结构、地点与提醒字段 · 能力矩阵

> 研究交回快照：以下工作区、未 commit/push 等执行状态记录作者交回时的事实。主控于 2026-10-01 独立复核后集成；当前发布链接、状态与后续未知以研究票 resolution comment 为准。

调查日期：2026-10-01。研究票：[调查 Reminders 原生结构、地点与提醒字段的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-967)，parent：[Apple Agenda MCP：日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。这是研究输入，不解决 HITL 产品票，不代表能力验收。

源码固定于 `3ad64088feb82104982b6b3e7181f6bb4ca8af94`，分支 `research/spe-967-reminders`；源码声明版本 `1.18.0`、Swift package 最低 macOS 14。机器系统为 macOS 27.0 / `26A428`，所选工具链为 Xcode 26.6 / `17F113`，实际检查的公开 SDK 是 macOS 26.5。系统版本、SDK 版本和源码版本不能互换。实际 binary 路径/版本、MCP 会话、签名、TCC 状态均未测试；没有创建 EventKit store、读取个人对象或触发权限请求。

## 结论与证据口径

Reminders 清单是支持 `.reminder` 的 `EKCalendar`；提醒事项是 `EKReminder`，继承 `EKCalendarItem`。公开 API 提供清单/事项 CRUD、开始/到期日期组件、完成/重新打开、完成时间、优先级、单个重复规则、文字地点、URL、notes、时间和地理 alarm。可调用 API 不保证特定来源接受保存或实际发出通知；来源错误是合同的一部分。[D1][D1] [D2][D2] [D3][D3] [D13][D13]；H1–H4、H7。

Calendar 的 `EKEvent.structuredLocation` 是日程地点；Calendar 和 Reminders 都可用 `EKAlarm.structuredLocation + proximity` 表达地理触发。Reminders 没有独立的 `EKReminder.structuredLocation` 属性。`EKStructuredLocation` 可由标题或 `MKMapItem` 创建，公开字段是标题、`CLLocation` 和半径；保存 Maps URL 只建立链接，不能据此声称有原生结构化地点、POI 身份或地理提醒。[D8][D8] [D9][D9] [D10][D10]；H1、H2、H5、H6、H8。

Apple Reminders 应用确实有原生子任务、分区和图片/扫描等功能；这不代表 EventKit 暴露同样能力。当前官方类目录和 macOS 26.5 SDK 的 `EKReminder → EKCalendarItem → EKObject`、`EKCalendar`、`EKEventStore` 完整公开声明没有父子关系、分区或附件的类型/字段/CRUD。就本次检查的公开 EventKit 表面，可判为 API 不支持这些原生操作；没有调用私有 selector、KVC、AppleScript 或数据库。macOS 27 的未检查 SDK、未来公开 API 和更新时隐藏数据的保存行为仍不能由此推断。[D1][D1] [D3][D3] [D16][D16] [D17][D17] [D18][D18]；H1–H4、H10。

状态严格使用：`API 不支持`、`API 支持但尚未实现`、`已实现但未验证`、`未知`。本报告的“已实现”仅表示固定源码有路径；所有表中的运行时状态均未验证。API/SDK 声明、源码判断、历史注释和本轮实测分别标识。本轮没有 `已验证` 产品能力，未把仓库已有测试文件或旧版注释升级为实测证据。

## 权限、来源与 macOS 差异

以下约束适用于后面的矩阵。

| 代号 | 约束与一手证据 |
|---|---|
| P-R | Reminders 读/写使用 full access。macOS 14 引入 `requestFullAccessToReminders`；没有 Reminders write-only 或 read-only 请求 API。Calendar 有 full/write-only，读取仍需 full。Sandboxed macOS 的 Calendar 访问需要相应 entitlement；实际宿主、签名/TCC 另验。[D11][D11]；H4 L48–90。固定源码 `AuthorizationGate` 每次检查 full access，`Info.plist` 有 Calendar/Reminders 的新旧 usage keys，entitlements 有两种 personal-information keys；这是配置声明，不证明本机授权。[S5][S5] [S9][S9] [S10][S10] |
| P-C | 清单内容写入看 `allowsContentModifications`；清单自身重命名/改属性/删除看 `isImmutable`。两者不同：immutable 清单仍可能允许增删事项。来源可禁止增删清单或不支持 reminders；必须检查 entity mask 并处理保存错误。[D2][D2] [D13][D13]；H3 L82–100、L125–130，H7 L39–51。当前 `updateCalendar` 用内容可写性作属性修改 guard，并对清单 update/delete 同时要求 Calendar + Reminders 权限；这比只操作纯 reminder 清单的 API 条件更宽。[S1][S1] L427–464 |
| P-S | `EKSource` 是已配置账户的只读抽象，不由应用创建；公开操作是查询 sources、来源 ID/type/title、按 entity type 列清单。清单 source 只可在新建时设置，之后不能搬到另一 source；不能将创建清单等同于配置/创建账户。[D12][D12] [D19][D19]；H3 L49–57、H9 |
| P-G | 来源可能分别拒绝 structured location、reminder location 或 alarm proximity，对应 `structuredLocationsNotSupported`、`reminderLocationsNotSupported`、`alarmProximityNotSupported`。文档声明 macOS/iOS 都支持 geofence，并指出移动设备更有效；本轮未验证来源保存、同步、定位/通知设置或实际触发。[D8][D8] [D13][D13]；H7 L44–46 |
| P-A | 来源可能限制 alarm 数量，保存时会截断多余 alarms；写后必须回读，不能用输入数量证明保存数量。旧 procedure alarm 从 OS X 10.9 起不能新建/修改，现有 alarm URL 不可读取；修改对象其他字段且不碰旧 alarm 被允许。[D8][D8]；H2 L100–108、H5 L101–110 |
| P-D | 日期组件的 calendar 必须 Gregorian；nil time zone 是 floating，缺少 hour/minute/second 是仅日期/全天。iOS 有 due 时要求 start，macOS 无此要求。start components 的 zone 与 `EKCalendarItem.timeZone` 联动，due components 的 zone 独立于二者；修改组件后须赋回属性。[D4][D4] [D5][D5]；H1 L29–48 |
| P-I | 清单和 calendar-item 本机 ID 不保证 full sync 后保持；external ID 也不是全局唯一定位条件，Exchange reminders 还存在跨设备差异。涉及查询/重复/恢复的稳定身份合同仍待后续研究，不以名称替代稳定身份。[D3][D3]；H2 L35–70、H3 L59–66 |

来源类型包括 local、Exchange、CalDAV、旧名 MobileMe/iCloud、subscribed、birthdays，但不能据此推导每一类账户都能存 Reminders 或接受每一字段。Apple 用户指南说明完整 Reminders 应用能力面向 updated iCloud reminders，其他 provider 可能缺少能力；该说明用于应用功能背景，不是 EventKit 来源逐项兼容表。[D19][D19] [D16][D16] [D17][D17] [D18][D18]；H9、H11。来源兼容仍需获授权的独立测试集合验证。

## 来源、清单和事项 CRUD

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具/输入/输出/错误 | 状态 | 验证层级与证据 | 缺口 / 决策票 |
|---|---|---|---|---|---|---|---|
| 查询 Reminders 来源与来源下的清单 | `EKEventStore.sources/source(withIdentifier:)`、`EKSource.calendars(for:)`；[D12][D12] [D19][D19]；H4 L99–118、H9 | P-R、P-S；只出现本机已配置且可访问来源 | `[S1]` 中按 `.reminder` 查询 calendars，未直接列 sources | `list_calendars(type:"reminder")` 返回 `id/title/type/source/source_type/allowsContentModifications/isSubscribed`；无 source ID、delegate 或独立 sources tool。[S2][S2] L1177–1195 | 清单来源摘要已实现但未验证；独立 sources 查询 API 支持但尚未实现 | API + SDK + 源码；无 TCC/来源读回 | 覆盖口径由 [SPE-969](https://linear.app/icecoke/issue/SPE-969) 决定 |
| 创建/更新/删除账户来源 | `EKSource` 官方说明只能取回，不能创建；H9 仅只读属性；[D19][D19] | P-S | 无路径 | 无工具及明确 unsupported error | API 不支持公开 EventKit 账户配置 CRUD | 官方类说明 + SDK 声明；无运行时实验必要 | [SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Read / 查询提醒清单 | `calendars(for:.reminder)`、`calendar(withIdentifier:)`；[D12][D12]；H4 L136–160 | P-R、P-I；entity type 不等于 calendar/source type | `[S1]` L371–386 | `list_calendars(type:"reminder")`；输出未带 `allowedEntityTypes/isImmutable/color/sourceIdentifier`。[S2][S2] L1183–1195 | 已实现但未验证 | API + SDK + 源码 | 缺字段及 ID 合同由 [SPE-969](https://linear.app/icecoke/issue/SPE-969) 后续判断 |
| Create 提醒清单 | `EKCalendar(for:.reminder,eventStore:)`、title/source/color、`saveCalendar`；[D2][D2] [D12][D12]；H3、H4 L163–175 | P-R/P-C/P-S；必须选择允许 reminders 的来源 | `[S1]` L394–424：使用默认 reminder 清单的 source；同 title/type 即返回已有对象；不能传 source | `create_calendar(title,type:"reminder",color?)` → action `created` 或 `skipped/duplicate` + id。无 source 入参。[S2][S2] L135–155、L1198–1218 | 已实现但未验证；指定来源创建 API 支持但尚未实现 | API + SDK + 源码 | 默认来源为空、同名跨来源、来源拒绝的明确合同；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Update 清单 title/color | 改 `EKCalendar` 属性后 `saveCalendar`；[D2][D2] [D12][D12]；H3 L69–115 | P-C；`isImmutable` 与内容可写性分开 | `[S1]` L427–451：同时要求两种 full access，以 `allowsContentModifications` 作 guard | `update_calendar(id,title?,color?)` → action `updated`；read-only guard 报 `calendarNotFound` 文案；底层 immutable/source errors 经通用 sanitizer | 已实现但未验证 | API + SDK + 源码；未验 guard 与真实清单属性权限关系 | [SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Update 清单所属 source | 新建可指定，保存后不可修改；[D12][D12]；H3 L49–57，H7 `calendarSourceCannotBeModified` | P-S | 没有迁移清单路径 | 无参数；不能以复制清单冒充原生移动 | API 不支持已保存清单跨来源移动 | 官方明确声明 + SDK | [SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Delete 提醒清单及内容 | `removeCalendar(_:commit:)`；[D12][D12]；H4 L177–200 | P-C/P-R；混合 entity 清单只获一类权限时，会删除已授权那类内容并移除该 entity mask；全授权时删除清单与所有内容 | `[S1]` L455–464：按 ID 删除，要求两种权限 | `delete_calendar(id)` → action `deleted/id`；无 deleted-item 清单返回 | 已实现但未验证 | API + SDK + 源码；未验级联、恢复、同步 | 不能把清单删除视作低影响；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Read / 查询提醒事项及完成过滤 | `calendarItem(withIdentifier:)`、官方提醒 predicates + 异步 `fetchReminders`；[D12][D12]；H4 L205–224、L373–417 | P-R/P-I；callback 数组可为 nil；无 API 顺序承诺 | `[S1]` L1451–1486、L1696–1703；nil 数组转空。分页值源见 `[S3]` | `list_reminders` / `search_reminders`：完成/filter、清单名+来源、sort/limit；输出计数、metadata、id/title/完成/priority/due/notes/location_trigger。无独立 get tool。[S2][S2] L476–502、L1566–1646 | 已实现但未验证；公开按 ID 完整字段读取尚未通过 MCP 暴露 | API + SDK + 源码；既有 mock 文件未运行 | 无 start/location text/URL/完整 alarms；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Create 提醒事项 | `EKReminder(eventStore:)`，title + calendar 必设，`save(_:commit:)`；[D1][D1] [D20][D20]；H1、H4 L337–354 | P-R/P-C；对象须属同一 store | `[S1]` L1531–1605；先同清单 title(+due) 去重，填写 title/notes/priority/due/rule/geofence | `create_reminder(title,calendar_name,calendar_source?,notes?,due_date?,priority?,recurrence?,location_trigger?,tags?)` → created 或 skipped/duplicate + id/title；无 completed 初始值。[S2][S2] L503–572、L1649–1686 | 已实现但未验证 | API + SDK + 源码；无真实创建/回读 | 字段合同与重复返回不等于输入完全相同；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Update 提醒事项/移动到另一清单 | 修改 fetched `EKReminder.calendar/title/...` 后 save；[D1][D1] [D20][D20]；H2 L28–33 | P-R/P-C；来源移入兼容性/同步另验 | `[S1]` L1608–1693；清单名+来源解析后换 calendar | `update_reminder(reminder_id,title?,notes?,due_date?,clear_due_date?,priority?,calendar_name?,calendar_source?,location_trigger?,clear_location_trigger?,tags?,clear_tags?)` → action `updated/id/title`；清单重名报错 | 已实现但未验证 | API + SDK + 源码 | 跨 source 事项移动是否保持字段/ID、隐藏结构副作用未知；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Delete 提醒事项 | `remove(_:commit:)`；[D12][D12]；H4 L357–370 | P-R/P-C；对象须属同一 store；有原生子任务时应用行为可能级联，EventKit 行为未测 | `[S1]` L1706–1716 | `delete_reminder(reminder_id)` → action `deleted/id`；not found 明确；底层错误见错误合同 | 已实现但未验证 | API + SDK + 源码；删除/恢复均未实测 | 原生子任务级联不得凭应用说明推定 API 行为；[SPE-972](https://linear.app/icecoke/issue/SPE-972) |

## 日期、完成、优先级和重复

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具/输入/输出/错误 | 状态 | 验证层级与证据 | 缺口 / 决策票 |
|---|---|---|---|---|---|---|---|
| R/C/U/清除 start | `startDateComponents: DateComponents?`；[D4][D4]；H1 L29–36 | P-D/P-R/P-C；Gregorian；不以 Date 转换抹掉 floating/date-only | 当前创建/更新和 read snapshot 无 start 字段。[S1][S1] L1531–1693；[S3][S3] L30–83 | 无 start、clear_start 或原始 start components 入/出参 | API 支持但尚未实现 | 官方文档 + SDK + 源码 | [SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| R/C/U/清除 due | `dueDateComponents`；[D5][D5]；H1 L38–48 | P-D；macOS 不强制有 start，iOS 强制；Gregorian | `[S1]` L1561–1574、L1633–1645 从 Date 取 host 年月日时分，并写 host zone；clear 写 nil | create/update `due_date`，update `clear_due_date`；互斥时 invalidParameter。read 有 legacy due_date + 原始语义 `due`。[S2][S2] L1614–1628、L1700–1703；[S4][S4] L149–172 | Date 路径已实现但未验证；完整 DateComponents 写入 API 支持但尚未实现 | API + SDK + 源码 | 输入 zone 被转为 host zone；writer 未显式设置 Gregorian calendar；未观察保存失败，不能据此宣称运行时 bug；[SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| C/U 仅日期或 floating 时间；保留源时区 | components 不带 HMS 是仅日期；zone nil 是 floating；start/item zone 耦合、due zone 独立；[D4][D4] [D5][D5] | P-D；Date 是 instant，不能替代无 zone 的壁钟或仅日期 | 写入一律带 hour/minute + host zone；read `ReminderDueValue` 保留日期/时间/zone/calendar，floating 和仅日期不给绝对 `date_time`。[S1][S1] L1569–1574；[S4][S4] L63–162 | 没有 reminder all_day/floating/timezone/components 写入字段；read 的顶层 `timezone` 是 host，不能当 due zone。[S2][S2] L1604–1617 | R 已实现但未验证；原生 C/U 语义 API 支持但尚未实现 | 官方声明 + 源码；跨时区/DST/iCloud readback 未跑 | [SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| R/C/U priority | `priority` 0–9，0 无、1–4 high、5 medium、6–9 low；枚举 canonical 值 0/1/5/9；[D6][D6]；H1 L69–76、H11 L234–250 | P-R/P-C；越界保存失败 `priorityIsInvalid` | `[S1]` L1558、L1631；create handler 要 integer，但无 0–9 range guard；update 读 `.intValue`。[S2][S2] L1661、L1705 | create/update `priority`；read raw Int。schema 主要举 0/1/5/9，不代表 API 仅四值 | 已实现但未验证 | SDK 范围 + 源码；未测越界错误/显示归类 | API raw priority 与用户四档语义需区分；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| R/U 完成 / 重新打开 | `isCompleted` 可读写，true 设 completionDate 为当前时间，false 清空；异端已完成可能 date=nil；[D7][D7]；H1 L50–67 | P-R/P-C；完成重复事项可能推进，见下行 | `[S6]` L6–33、`ReminderCompletionWrite` 避免同状态重复改 stamp。[S7][S7] L199–223 | `complete_reminder(reminder_id,completed?)`，缺省/null=true，false reopen；非 boolean 写前拒绝。结果 `operation.type/status/target` + `observed`，legacy action 常为 completed。[S2][S2] L626–636、L1759–1767 | 已实现但未验证 | API + SDK + 源码；没有同 MCP 会话测试 | `operation.status` 与保存对象状态分开；[SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| R/U 指定 completionDate | 可写 Date；非 nil 令 completed=true，nil 令 false；[D7b][D7b]；H1 L50–67 | P-R/P-C；不能要求 completed=true 时 date 一定存在 | read `completion_date`；undo 内部写 recorded instant。[S2][S2] L1619–1621；[S7][S7] L212–222 | 无任意 completion_date 输入；complete 工具由源码使用 now | R 已实现但未验证；用户指定时间 API 支持但尚未实现 | 官方文档 + SDK + 源码 | 是否允许回填完成时间为产品决定；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| R creationDate / lastModifiedDate；C/U 元数据时间 | 继承的 `creationDate`、`lastModifiedDate` 是 readonly，区别于可写 completionDate；[D3][D3]；H2 L82–84 | P-R；外部同步/变更可改变元数据，不能作为唯一身份 | Reminder read snapshot/list 已输出 creationDate，未输出 lastModifiedDate。[S3][S3] L38、L80；[S2][S2] L1623–1625 | `creation_date/creation_date_local`；无 last_modified 或元数据时间输入 | creation R 已实现但未验证；lastModified R API 支持但尚未实现；C/U 元数据时间 API 不支持 | SDK + 源码；无真实 readback | [SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| R/C 单个提醒重复规则 | 继承 `recurrenceRules/addRecurrenceRule`；当前 EventKit 实现只支持单一规则；`EKRecurrenceRule` daily/weekly/monthly/yearly、interval、BY*、ordinal weekday、end/count；[D14][D14] [D15][D15]；H2 L118–125、H12 | P-R/P-C；`recurringReminderRequiresDueDate`；某些 BY* 组合被忽略，不能声称任意 RFC 5545 字符串支持 | create 设置一条；输入仅 frequency/interval/weekdays/monthdays/end/count，builder 将其他 BY* 设 nil；read 保留更多 native 字段。[S1][S1] L1585–1587、L1875–1905；[S4][S4] L6–59 | `create_reminder.recurrence`；read `has_recurrence/recurrence_rules/reminder_recurrence_rules`；没有 RRULE 文本/多规则参数 | 子集已实现但未验证；其他公开规则字段 C API 支持但尚未实现 | API + SDK + 源码；无 save/sync 证据 | [SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| U/清除提醒重复规则 | 替换新 `EKRecurrenceRule` 或 `recurrenceRules=nil` / removeRule，再 save；[D14][D14] [D15][D15]；H2、H12 | P-R/P-C；规则对象主体多为 readonly，不是逐属性编辑 | `updateReminder` 不收 recurrence 或 clear；`ReminderUpdateRequest` 不传它。[S1][S1] L1608–1619；[S8][S8] L13–23 | `update_reminder` 无 recurrence/clear_recurrence | API 支持但尚未实现 | API + SDK + 源码 | [SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| 单次重复提醒完成、后继识别和作用域修改 | save/remove reminder 没有 `EKSpan` 或 occurrenceDate 参数；公开 recurrence 和 completion 字段存在；[D12][D12] [D20][D20]；H4 L337–370 | ID full sync/重复推进不稳定；SDK 未给后继生成和身份保证 | `[S6]` 仅保存后观察一次，匹配 before/after 的同 ID/due/rules/list/source，不能确认则 unknown；注释引用旧 iCloud probe，非本轮实测 | complete 输出 `next_occurrence.status=confirmed/unknown`，写成功不能因 unknown 重试；没有 reminder this/future/all span | 当前 completion 观察已实现但未验证；来源后继/单次与系列 U/D 语义未知；公开 EventKit 无 reminder EKSpan 入口 | SDK + 源码；历史 probe 只作风险线索 | 不能借用 Calendar 实例语义；[SPE-970](https://linear.app/icecoke/issue/SPE-970) |

## 地点、URL、notes 和 alarms

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具/输入/输出/错误 | 状态 | 验证层级与证据 | 缺口 / 决策票 |
|---|---|---|---|---|---|---|---|
| Calendar R/C/U/清除地点文本 | 继承 `location:String?`；SDK 明确 event getter 返回 structured title，setter 等于新建仅标题的 structured location；[D21][D21] [D10][D10]；H8 L89–96 | P-R 的 Calendar full access / P-C；改文本可能替换坐标信息 | event create/update 赋 location；read 返回 location。[S1][S1] L603、L847；[S2][S2] L2861 | create/update_event `location`；无 explicit clear_location；空 string 是赋空文本，JSON null 被 stringValue 忽略 | R/C/U 已实现但未验证；native nil 清除尚未定义 | API + SDK + 源码 | 文字地点与结构地点不是两个互不影响的 event 槽；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Reminders R/C/U/清除地点文本 | 同一继承 `EKCalendarItem.location`；[D21][D21]；H1 继承、H2 L78 | P-R/P-C/P-G；provider 可以拒绝 reminder locations | reminder write/read 没有 location 字段。[S1][S1] L1531–1693；[S3][S3] | 没有 location 输入/输出；`location_trigger.title` 是另一对象的数据 | API 支持但尚未实现 | SDK + 官方字段 + 源码 | 文字位置不保证进入 Reminders UI 哪一原生槽/保存兼容；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Calendar R/C/U/清除结构化地点坐标/半径 | `EKEvent.structuredLocation` + `EKStructuredLocation.title/geoLocation/radius`；[D9][D9] [D10][D10]；H6、H8 | P-G；macOS 10.11 起 event structuredLocation；半径米，0=系统默认 | `[S1]` L646–655、L924–933 title initializer，lat/lon 均有才赋；radius 只在 >0 时赋；read 输出 >0 radius。[S11][S11] L78–87 | create/update_event `structured_location{title,latitude?,longitude?,radius?}`；schema 写 default100，实际未传 radius 时留 SDK 默认0；无 clear_structured_location | R/C/U 子集已实现但未验证；native nil 清除尚未定义 | SDK + 源码；无真实保留/Maps UI 证据 | 默认半径 schema/代码不一致、文本覆盖坐标、范围验证；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Reminders 独立 item structuredLocation | `EKReminder` / `EKCalendarItem` 无该属性；alarm 上有；[D1][D1] [D3][D3] [D9][D9]；H1、H2、H5 | 当前 SDK 范围 | 以 alarm 做 location_trigger | 无 reminder `structured_location` 字段 | API 不支持该独立属性；alarm 地点支持见下行 | 完整公开声明比对 | 不得将 `location_trigger` 冒充 item POI；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Calendar + Reminders 用 MKMapItem 建结构地点 | `EKStructuredLocation.init(mapItem:)`，macOS 10.11 起；[D9][D9] [D9b][D9b]；H6 L19–24 | 公开结果只有 title/CLLocation/radius；无公开 POI identifier/readback mapItem 属性 | 当前 title+coordinate initializer，未使用 mapItem。[S1][S1] L646–655、L1591–1598 | 无 mapItem/search/place-ID schema | API 支持 mapItem 初始化但尚未实现；POI 身份持久化/完整地图详情回读未知 | API + SDK + 源码；无 POI 写入/独立回读 | 不能因 initializer 存在承诺原生地点卡/POI身份；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Calendar + Reminders R/C/U/清除 URL / Maps URL | `EKCalendarItem.url:URL?`；Apple Maps URL 打开地图/搜索/路线，不自动建立 EventKit structuredLocation；[D22][D22] [D23][D23]；H2 L80 | P-R/P-C；URL 属性与 alarm.url 不同；API 未在字段文档限制仅 http(s) | Event create/update 接受 URL 并 read；reminder read/write 无 URL。[S1][S1] L607–609、L857–859；[S2][S2] L2864；[S12][S12] L20–38 | create_event schema 有 url；update_event handler 读 url 但 schema 缺 url。代码仅允许 http/https；无 clear_url，null 不清。Reminders 无 URL schema | Calendar C/R 已实现但未验证；U handler 已实现但 schema 未暴露；Reminder R/C/U 与双方 nil 清除 API 支持但尚未实现 | 官方 URL/Maps 说明 + SDK + 源码 | 链接和 POI 能力分开；update schema parity；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Calendar + Reminders R/C/U/清除 notes 文本 | `EKCalendarItem.notes:String?`；[D24][D24]；H2 L79 | P-R/P-C；公开字段为 String，没有 rich-text/attachment/checkbox-object API | 双方 notes read/write；reminder 有 hashtag 提取/合并，read 是 cleanNotes。[S2][S2] L1603–1613、L1657–1658、L1712–1753 | create/update notes；长度65535是项目限制。空 string 可替换为文本空，null 不代表 nil-clear；没有 native checklist schema | 文本 R/C/U 已实现但未验证；native nil 清除与格式完整保留未知/尚未定义；原生子任务不由 notes 支持 | API + SDK + 源码 | Markdown `- [ ]` 只可能是备注文本；不承诺应用富文本完整 round-trip；[SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| Calendar R/C/U/D 相对时间 alarms | `EKAlarm(relativeOffset:)`，相对 event start 秒数；absolute/relative 设置互相清除；[D8][D8]；H5 L26–58 | P-A；官方要求相对 alarm 在开始之前或开始时；来源最大数可能截断 | create/add，update 全删后重建，输入分钟转负秒。[S1][S1] L616–621、L872–884 | create/update_event `alarms_minutes_offsets`；[] 删除全部。read formatter 无 alarms 输出。[S11][S11] L27–48；[S2][S2] L2843–2884 | C/U/D 已实现但未验证；完整 R API 支持但尚未实现 | API + SDK + 源码；未测正负值、截断、触发 | 修改 alarms 会替换现有绝对/地理/动作 alarms；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Reminders R/C/U/D 相对时间 alarms | 继承 alarms/add/remove；[D1][D1] [D8][D8]；H2 L100–116、H5 | P-A/P-D；文档给相对 start，不足以认定 Reminders UI 的 early reminder 以 due 为锚；start/due 缺失和兼容要测 | manager 有 alarmOffsets 参数，但 `ReminderCreateRequest/UpdateRequest` 和 handlers 不传；read snapshot 只取第一地理 alarm。[S1][S1] L1577–1582、L1652–1663；[S8][S8] | create/update_reminder schema 无 alarms_minutes_offsets；无完整 alarms 输出 | 内部方法已实现但未验证；MCP R/C/U/D API 支持但尚未实现 | API + SDK + 源码；无 due/start 锚点实验 | 不能用 manager 参数证明 MCP 可调用；[SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| 双方 R/C/U/D 绝对时间 alarms | `EKAlarm(absoluteDate:)` / `absoluteDate`、item alarms；[D8][D8]；H5 L26–58 | P-A/P-R/P-C | 无直接路径；undo 快照只取 relativeOffset | 无 absolute-date input/output | API 支持但尚未实现 | 官方声明 + SDK + 源码 | 时间 alarm 完整合同；[SPE-970](https://linear.app/icecoke/issue/SPE-970) |
| Reminders R/C/U/D 地理 alarms | `EKAlarm.structuredLocation + proximity .enter/.leave/.none`；[D8][D8] [D9][D9]；H5 L60–73、H11 L204–214 | P-G/P-A；坐标、半径米，0=系统默认；实际定位/通知另验 | create 添加；update 替换已有地理 alarms 或 clear；非正 radius 改100；read 只第一条地点 alarm，radius0被省略。[S1][S1] L1590–1598、L1665–1686；[S3][S3] L61–83 | `location_trigger{title,latitude,longitude,proximity,radius?}` / `clear_location_trigger`；parser 缺 proximity 默认enter、未知字符串也变enter，坐标无范围 guard；schema 宣称 required/enum。[S2][S2] L553–567、L3177–3195 | 单地理触发 R/C/U/D 已实现但未验证；完整 alarms R/API 默认 radius0 尚未正确暴露 | API + SDK + 源码；无 source save/sync/fire 证据 | default0 与硬编码100、非法值、replace/clear 冲突、保留其他 alarms；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| Calendar R/C/U/D 地理 alarms | 同一 `EKAlarm` 公开 geofence，官方明确可用于 events；[D8][D8]；H5 | P-G/P-A；event 地点本身不会自动变 geofence | 无 event location-trigger write/read；event structuredLocation 只是 item 地点 | `structured_location` 不能替代 geo alarm；无 event location_trigger | API 支持但尚未实现 | 官方声明 + SDK + 源码 | geofence 事件覆盖是否纳入由 [SPE-969](https://linear.app/icecoke/issue/SPE-969) 决定 |
| macOS alarm 类型/声音/email 动作 | `EKAlarm.type` readonly，由 soundName/emailAddress 设置；macOS 专有，iOS unavailable；[D25][D25] [D26][D26]；H5 L75–99 | P-A；Reminders 有 `reminderAlarmContainsEmailOrUrl` 错误符号，但无说明文本，不能把所有 event 动作推广到 reminder。[D27][D27]；H7 L101 | 当前只生成普通 relative/geo alarms，无动作字段 | 无 type/sound/email 参数 | macOS 字段 API 支持但尚未实现；Reminders 动作可接受范围未知，有明确拒绝线索 | SDK + 官方字段/错误符号；无保存/实际动作测试 | 不因通用文章例子承诺提醒 email 可用；[SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| procedure alarm 打开 URL | `EKAlarm.url` 已在 macOS 10.9 deprecated；H5 L101–110，`procedureAlarmsNotMutable`；[D13][D13] | 现代 macOS 不可新建/修改，不可读取既有 URL；可改其他字段前提不碰旧 alarm | 无专用写；目前 alarm 全替换/undo 可碰旧 alarm | 无此 schema；item URL 仍可用 | API 不支持现代 macOS 新建/修改 procedure alarm；不是“不支持 item URL” | SDK 明确限制；未做破坏性测试 | [SPE-969](https://linear.app/icecoke/issue/SPE-969) |
| 上/下车或联系人消息触发 | Reminders 应用支持 car/message 提醒；公开 EKAlarm 只有日期、enter/leave/none geofence、动作字段；[D18][D18] [D8][D8]；H5、H11 | 当前 SDK 无 car/Bluetooth 或 messaging-contact trigger 字段 | 无路径 | 无参数/明确 unsupported error | API 不支持这些专用原生触发入口（本 SDK 范围） | 应用官方说明 + 完整公开声明比对；非由工具缺失推导 | [SPE-972](https://linear.app/icecoke/issue/SPE-972) |

## 原生组织结构与信息保留

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具/输入/输出/错误 | 状态 | 验证层级与证据 | 缺口 / 决策票 |
|---|---|---|---|---|---|---|---|
| R/C/U/D 原生父子层级、子任务排序、缩进 | 应用有子任务；官方 API 类目录及完整继承公开声明无 parent/children/subtasks/order API；[D1][D1] [D3][D3] [D16][D16]；H1/H2/H10 | updated iCloud 应用功能可用，不代表 EventKit API。父任务在应用中完成/删除/移动会连带子任务 | 无原生关系路径；notes/tags 不表达层级 | 无工具/schema/error；不能将 flat reminders 推定父子树 | API 不支持原生关系 CRUD（本 SDK 范围）；对子任务平面可见性和写入级联未知 | SDK + 官方应用说明；没有个人对象实验 | 明确 unsupported 口径、已有父任务写入副作用；[SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| R/C/U/D 分区、列、分区归属与顺序 | 应用有 sections；`EKCalendar`/`EKReminder`/store 无 section 类型或字段；[D2][D2] [D17][D17]；H1–H4 | 应用删除 section 会连带其中 reminders；不把该行为套到不存在的 EventKit section API | 无路径 | 无 schema；list 名不等于 section | API 不支持原生分区操作（本 SDK 范围）；现有 section 内事项 CRUD 保留归属未知 | SDK + 官方说明 | [SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| R/C/U/D 图片、扫描/素描、其他附件 | 应用可附 images/scan/sketch；EventKit reminder/item/calendar/store 无 attachments、二进制/文件引用集合或附件 CRUD；[D18][D18] [D1][D1] [D3][D3]；H1–H4/H10 | 更新后的 iCloud 应用完整能力，不等于 API；链接不等于附件 | 无路径；URL/notes 不是 attachment | 无 schema/error；不能用 notes 内 file URL 冒充原生附件 | API 不支持原生附件 CRUD（本 SDK 范围）；隐藏附件在更新/移动/恢复时是否保留未知 | SDK + 官方应用说明 | [SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| Calendar R/C/U/D 原生文件附件 | Calendar 应用可添加/预览/删除文件附件；`EKEvent` 及 item/store 公开声明无附件属性/CRUD。[D28][D28] [D3][D3] [D12][D12]；H8 完整成员、H2/H4 | 应用能力和 provider 同步不等于 EventKit 能力；URL 是不同字段 | 无 attachment 路径；EventSnapshot 也无附件字段。[S13][S13] L69–104 | 无工具/schema/明确 unsupported error；notes 或 URL 不等于附件 | API 不支持原生附件 CRUD（本 SDK 范围）；保存/复制/恢复时隐藏附件保留未知 | SDK + 官方应用说明；无复杂 event 实验 | [SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| 原生 tags/flag/Smart List/列表图标与 grocery 类型 | 应用有这些组织字段；`EKReminder`/`EKCalendar` 公开声明无对应 tags/flag/icon/SmartList/grocery 属性；[D18][D18] [D2][D2]；H1–H4 | 与 priority/color 区分；不能把应用指南当公共 API | MCP tags 被写成 notes 中 `#hashtag`；源码/schema 已注明文本语义。[S2][S2] L548–551、L611–618、L3203–3239 | `tags/clear_tags/list_reminder_tags` 是项目文本工具；没有原生 tag 身份、flag 或 smart-list rule | 原生 API 不支持这些入口（本 SDK 范围）；notes tags 已实现但未验证 | SDK + 源码 + 官方应用能力 | 命名/展示需要避免原生支持误解；[SPE-972](https://linear.app/icecoke/issue/SPE-972) |
| 编辑/删除/恢复已有复杂原生对象时保留信息 | API 声明 save 后更新为数据库最新字段；未说明每种隐藏子任务/section/attachment 的保留、级联、跨来源转换；[D20][D20]；H4 L337–345 | P-I/P-C；重复推进、外部修改、来源转换和隐藏数据有独立风险 | ReminderSnapshot 无 start/location/URL/recurrence/完整 alarm；restore 清空 alarms，仅按 relativeOffset 重建，清单只按 title 找。[S13][S13] L118–139；[S1][S1] L2139–2160 | undo/redo 不能据现有 snapshot 声称任意 reminder 原样恢复；现有 complete guard 只保护部分重复身份 | 完整信息保留未知；源码可确认 snapshot 表达缺口；已实现的恢复路径未验证 | SDK + 源码；未运行 undo/redo 或复杂对象真实回读 | 需同长驻 MCP + 独立系统回读；[SPE-970](https://linear.app/icecoke/issue/SPE-970) / [SPE-972](https://linear.app/icecoke/issue/SPE-972) |

## 当前 MCP 合同的证据与缺口

源码 schema 是 `Server.defineTools`，handler 是 `executeToolCall` 后各工具方法，不可只检查其中一面。入口没有通用“未知参数拒绝”代码，各 handler 读取自身字段；不能把传入未公开字段后无错误当成功，也不能把工具不存在当 API 不支持。[S2][S2] L117–1048、L1100–1169。

单写入通常返回 `actionResult` 包装的 action/id/title；读取通常是 JSON text，reminder list 有 `reminder_count/reminders/metadata`。所有工具外层失败为 MCP `isError:true` 和文本 `Error: <code>`；可信项目错误保留消息，EventKit NSError 变 `eventkit_error_<N>`。这保留了框架 code，但不是按来源/权限/字段建立的 typed capability error。[S2][S2] L1067–1088；[S14][S14] L27–50。缺少 native 子任务等 schema 并不会自动产生统一的 unsupported 响应。

需要交给后续合同决策的具体差异：

1. Reminder Date 写入丢失日期组件原始形态：仅日期、floating、输入 zone 不能可靠表达；start 完全未暴露，Gregorian calendar 未显式设置。legacy due_date 是转换输出，`due` 才保留源组件语义。源 start/item zone 和 due zone 必须分开。[S1][S1] L1561–1574、L1633–1645；[S4][S4] L149–172；[D4][D4] [D5][D5]。
2. Event `url` update handler 可接受但 schema 缺失；Reminder URL/文字地点、相对/绝对 alarms、重复 update/clear 未经 MCP 暴露。内部 manager 有参数不能视作当前调用能力。[S2][S2] L329–414、L1398、L503–623；[S8][S8]。
3. geofence radius=0 的原生默认被 reminder writer 改为100，read 省略0；event schema 宣称默认100，writer 未提供 radius 时却留0。地点 trigger 缺/非法 proximity 被 parser 转enter，lat/lon 未校验范围。此处为源码判断，没有保存/实际触发结果。[S1][S1] L652–654、L1594、L1682；[S2][S2] L3177–3195；[S3][S3] L63–75。
4. 清单属性 immutable 与内容可写 guard 混用；source 创建选择和稳定来源 ID 缺失；update/delete 要求两类权限。API 自身的纯 reminder 操作并不因此需要 Calendar full access。[S1][S1] L394–464；[D2][D2] [D11][D11] [D12][D12]。
5. 一条 location_trigger 输出不是完整 alarms 读取；alarm 全替换和仅 relativeOffset 的 undo snapshot 会丢失表达绝对/地理/声音/email alarm 的必要字段。SDK 明确限制 procedure alarm 修改，不能承诺“旧 alarm 原样保留/恢复”。这是快照信息不足的源码证据，尚未记录真实数据丢失事故。[S3][S3] L63–83；[S13][S13] L126–138；[S1][S1] L2152–2160；H5 L101–110。

批量 create/reminder cleanup 另有 row-level success/failure；`create_reminders_batch` 比单条工具少 recurrence/location_trigger，不能假设单条字段都适用。[S2][S2] L962–993、L1873–1941。批量部分失败、undo/redo、标识变化的完整状态合同属于后续合同/验证票，本轮不扩展实现。

## 需要授权实验的未知

下列均未执行，不能把 API 声明/已有 mock 当答案。后续执行前按项目验证合同先确认 binary 路径/版本/commit、同一 MCP 会话、签名与 TCC；只用本次可识别独立清单和对象，记录 ID 并清理本次对象。

| 需要实验的事实 | 最小验证面 / 判定证据 |
|---|---|
| 各来源的清单增删、提醒字段、structured location/geofence 支持 | 来源属性 + 新建独立对象 save 的 domain/code；新 store/独立系统读回；不得用来源名称推断兼容 |
| 日期组件 Gregorian/仅日期/floating/start+due 双 zone、DST、iCloud 显示 | 组件写前/写后、新 store readback、Reminders UI 对照；跨设备/远端同步另记，不与本机保存混同 |
| Relative reminder alarm 的实际 start/due 锚点、无日期 alarm、混合 absolute/geo、数量截断、实际通知触发 | 完整 alarm 数组回读；触发/系统通知为独立层。不能以保存成功证明通知出现 |
| 地点文字/结构位置/MKMapItem 保存后的 native UI、POI 信息保留 | 公共字段独立读回 + UI 观察；用 Maps 链接打开成功不能证明 native POI |
| 完成重复提醒后 ID/due/完成记录/后继行为及 reopen | 同长驻 MCP、保存前后快照、新 store/系统回读；next unknown 时不重试；不把单来源结果泛化 |
| 已有子任务/sections/attachments 的平面可见性、普通编辑/完成/删除/移动的级联与保留 | 由主控获授权创建独立复杂样本，UI 和公开读回并列；明确本轮对象，禁止在个人父任务试验 |
| 原始 notes 富文本、tags、绝对/地理/旧 procedure alarm 的编辑/恢复保留 | 同 MCP 会话 + before/after 全字段与 UI；只用可控样本。现有快照无字段意味着不能承诺恢复完整性 |

本报告不给这些实验先验结果，也不把保护未知数据的产品策略作为已定决定。

## 一手来源索引

所有 Apple API 链接在本轮查阅；普通文档页面有 JavaScript-only 响应时，通过官方 `.md` 版本核对正文。Web 搜索用于核对 niche facts，正文与 SDK 优先于检索摘要；来源均为 Apple 或固定项目源码。Apple 的 alarm 总览仍列 URL procedure 示例，但本机 SDK 的明确 deprecated/禁止修改说明优先，不能照例子承诺现代支持。

### 官方文档

| 代号 | 文档与本次用途 |
|---|---|
| D1 | [Creating events and reminders][D1]：reminder 继承、必填字段、CRUD、alarms/recurrence |
| D2 | [EKCalendar][D2]：清单字段、内容权限、immutable、entity/source |
| D3 | [EKCalendarItem][D3]：继承公共字段、完整公开目录 |
| D4 / D5 | [startDateComponents][D4] / [dueDateComponents][D5]：floating/date-only/Gregorian、双 zone 与平台差异 |
| D6 | [priority][D6]：字段；具体0–9范围和分档以 H1/H11 核对 |
| D7 / D7b | [isCompleted][D7] / [completionDate][D7b]：完成时间耦合、异端 nil 情形 |
| D8 | [Setting an alarm][D8]：双方 alarms、相对锚点、geofence、macOS 声明 |
| D9 / D9b / D10 | [EKStructuredLocation][D9] / [init(mapItem:)][D9b] / [EKEvent.structuredLocation][D10]：结构字段、mapItem 初始化与 event 属性 |
| D11 | [Accessing the event store][D11]：权限级别与 macOS sandbox；availability 由 H4 核对 |
| D12 | [EKEventStore][D12]：source/calendar/item/query/save/remove 完整方法目录 |
| D13 | [EKError][D13]：来源和保存限制；未说明的错误符号不扩成实测结论 |
| D14 / D15 | [recurrenceRules][D14] / [EKRecurrenceRule][D15]：单规则、规则字段与构造限制 |
| D16 | [Add and remove subtasks to reminders on Mac][D16]：应用存在原生层级及应用级联行为 |
| D17 | [Manage sections in reminder lists on Mac][D17]：应用存在分区及来源限制 |
| D18 | [Add or change reminders on Mac][D18]：应用图片/URL/notes/flag/car/message 功能；非 API 证据 |
| D19 | [EKSource][D19]：只取回已配置账户、不创建 source |
| D20 | [save(_:commit:)][D20]：reminder save、同 store 与 commit |
| D21 / D22 / D24 | [location][D21] / [url][D22] / [notes][D24]：双方继承字段 |
| D23 | [Map Links][D23]：Apple Maps URL 只是地图打开/查询/路线链接；归档说明不作当前 POI API 合同 |
| D25 / D26 / D27 | [emailAddress][D25] / [soundName][D26] / [reminderAlarmContainsEmailOrUrl][D27]：macOS 动作字段和 Reminder 保存拒绝线索 |
| D28 | [Add notes, a URL, or files to events in Calendar on Mac][D28]：Calendar 应用原生文件附件能力；非 EventKit API 证据 |

### SDK 证据

本机公开头文件共同根：`/Applications/Xcode-26.6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX26.5.sdk/System/Library/Frameworks/EventKit.framework/Headers/`。下列行号由 `nl -ba` 读取；没有复制整份 Apple 头文件进仓库。

| 代号 | 文件与行号 |
|---|---|
| H1 | `EKReminder.h` L20–78：完整 reminder 成员及日期/priority/完成说明 |
| H2 | `EKCalendarItem.h` L15–127：完整公共 item 成员；L35–70身份；L78–84 location/notes/URL/zone；L100–125 alarms/recurrence |
| H3 | `EKCalendar.h` L38–130：初始化/source/ID/权限/颜色/entity mask |
| H4 | `EKEventStore.h` L48–90权限；L99–200来源/清单；L205–224 item；L337–417提醒 CRUD/query |
| H5 | `EKAlarm.h` L22–110：完整 alarm 字段；L101–110 procedure 禁止说明 |
| H6 | `EKStructuredLocation.h` L16–26：完整 location 字段，macOS10.11 mapItem；radius0默认 |
| H7 | `EKError.h` L25–60、L64–104：来源/recurrence/priority/权限及 reminderEmailURL错误 |
| H8 | `EKEvent.h` L37–203完整成员，无 attachments；L89–96：text location setter 替换结构地点的声明 |
| H9 | `EKSource.h` L16–45：完整 source 字段/查询，delegate macOS13起 |
| H10 | `EKObject.h` L10–35：只有 changes/new/reset/rollback/refresh，无原生父子操作 |
| H11 | `EKTypes.h` L176–182来源枚举；L204–214 proximity；L234–250优先级枚举 |
| H12 | `EKRecurrenceRule.h` L20–118、L137–189：规则构造、正 interval、BY*、不可任意改规则主体 |

用于复现的 SHA-256：

```text
EKReminder.h           488450204d1c8c943e5ef681d9696cb8a5cfcd19911acc32818b793f0ef4c83a
EKCalendarItem.h       e7a810b989480305fccf1fa2dde439d25042b8b49b580187bfed12ccf41fedf0
EKCalendar.h           3eb9f759054a392b3c4985d4a65af84a62b399b0a201f0570e39ff4b9db8b510
EKAlarm.h              2f3e332d043e8216f0133f2a849e3cbe7e3d15fd6715308972e3712c40298ca7
EKStructuredLocation.h a829b7a278f30f7f02353b399e17269030c7ff752268c8e530fd25c589eb1886
EKEventStore.h         0a9c7c4f80a4bc19060360585c5edb903be680938c63aefcee6d1efb22699f43
```

### 固定源码

链接均锁定本研究基线 commit，不使用浮动 main。表中 `[S1]` 等表示以下文件和具体行号；目录中有 tests 只说明测试资产存在，本轮未跑。

| 代号 | 文件 |
|---|---|
| S1 | [EventKitManager.swift][S1]：清单/提醒 CRUD、字段、alarm 和恢复 |
| S2 | [Server.swift][S2]：工具 schema、handler、输出和错误外层 |
| S3 | [ReminderReadSource.swift][S3]：read snapshot 和第一条地点 alarm |
| S4 | [ReminderRecurrence.swift][S4]：原始 due/规则输出 |
| S5 | [AuthorizationStatusSource.swift][S5]：请求权限和每次 gate |
| S6 / S7 | [EventKitManager+ReminderCompletion.swift][S6] / [ReminderCompletion.swift][S7]：完成/后继观察和完成时间恢复 |
| S8 | [ReminderWriteSource.swift][S8]：MCP 写请求字段，未转发内部 alarmOffsets |
| S9 / S10 | [Info.plist][S9] / [Entitlements.plist][S10]：源码授权/签名配置声明 |
| S11 / S12 | [EventFormattingSource.swift][S11] / [Validation.swift][S12]：event 输出与项目限制 |
| S13 / S14 | [UndoManager.swift][S13] / [EventKitErrorSanitizer.swift][S14]：snapshot 与错误合同 |

## 执行记录和 not-run

工作区：`/Users/leolau/Documents/Codex/worktrees/apple-agenda-chart/spe-967-reminders`。只增加本 Markdown；没有修改共享入口/Linear，没有 commit/push。票维持主控领取后的 In Progress，主控核对后再决定 resolution/关闭。

实际命令与退出状态：

| 命令 / 操作 | 结果 |
|---|---|
| `pwd && git status --short && git branch --show-current && git rev-parse HEAD` | exit0；初始干净；分支/HEAD与派工一致 |
| `cat AGENTS.md docs/agents/agent-execution.md docs/agents/validation.md CONTEXT.md docs/agents/templates/capability-matrix.md`；`cat /Users/leolau/.agents/skills/research/SKILL.md`；`cat docs/agents/day-1.md` | exit0；按已派 background Research 角色执行，不递归派工 |
| `rg -n "apple-agenda\|EventKit\|Reminders\|SPE-967" /Users/leolau/.codex/memories/MEMORY.md` | exit1：无相关命中；没有使用历史 memory 结论 |
| `rg --files Sources Tests docs/research`（与 Day1/Package 读取同一命令） | exit2：初始 `docs/research` 不存在；Sources/Tests 列出；之后创建唯一指定报告文件 |
| `cat Package.swift`；多次 `rg -n` 与 `nl -ba … \| sed -n …` 读取上表固定源码 | exit0；无执行产品 |
| 一次尝试 `nl -ba Sources/CheICalMCP/EventKit/EKError.swift` | exit1：文件不存在；改查真实 `EventKitManager.swift` 的 `EventKitError` 与 SDK `EKError.h`，未伪造路径 |
| `xcode-select -p && xcrun --sdk macosx --show-sdk-path && xcrun --sdk macosx --show-sdk-version && sw_vers`；`xcodebuild -version` | exit0；SDK26.5 / Xcode26.6 / host27.0；`xcodebuild -version` 只取版本，未 build |
| `ls …/EventKit.framework/Headers`；`nl -ba` 上表 H1–H12；`rg -n 'parent\|children\|subtask\|section\|attachment\|tag\|flag\|smart\|order' …/Headers` | exit0；相关词仅命中普通注释/无关 conference ordering 等；核心继承/类完整声明无对应功能。关键词扫描仅辅助，结论依完整公开声明 |
| `shasum -a 256` H1/H2/H3/H4/H5/H6 | exit0；摘要见上 |
| Python3 `urllib.request.urlopen` 官方 `https://developer.apple.com/documentation/eventkit/<symbol>.md`，每 URL timeout15/20秒，仅打印正文/目录 | shell exit0；所引用 `.md` 返回200；尝试错误路径 `ekeventstore/removereminder(_:commit:).md` 返回404，后通过 `ekeventstore.md` 找到真实 `remove(_:commit:)`，不引用404路径 |
| Web 官方域 search/open/click | API查询成功；普通页有JS-only正文；click官方 `.md` 时 Web fetcher 报 unsupported content-type400，改用上述 urllib直接读取官方正文，未使用二手内容 |
| Linear read `get_issue(SPE-961/SPE-967, includeRelations:true)` | 工具 `isError:false`；核对本项目、parent、票范围/领取状态/原生 blockers；没有写操作 |
| `git diff --check`；`git diff --no-index --check -- /dev/null docs/research/spe-967-reminders-location-eventkit.md` | 前者 exit0；新文件的 no-index 比较 exit1（存在新增 diff），无 whitespace 诊断；结合下面直接文件检查，不将 untracked 文件漏出验证 |
| Python3 文档自检：引用键、`git cat-file -e <baseline>:<source-path>`、八列表格、尾部空白、最后换行、SDK SHA-256、secret pattern | exit0；42 个能力行，44 个引用定义，无缺引用；所有源码路径存在于基线；六个 SDK 摘要一致；无尾部空白或敏感 token/private-key 模式 |

文档检查：引用键均有定义，所有源码引用路径在固定基线存在，六个 SDK 摘要重新计算一致，42 个能力行保持模板的八列。检查仅针对文档，不是产品运行验证。最终 `git status --short --untracked-files=all` 仅列本报告，branch/HEAD 保持派工值。无临时下载/脚本文件落盘；报告作为本机 WIP 保留。

Not-run：Swift build / Swift tests / CLI / MCP session / Calendar or Reminders API 实例与个人数据读取 / 新建、修改或删除真实对象 / TCC prompt / 签名、安装、MCP 配置切换、发布 / 跨设备同步或通知实际触发 / 独立 reviewer。原因是本票仅授权一手来源研究和文档交付，且 docs-only 不需产品 suite。未完成的是上表需授权实验、独立 review、主控文档集成及后续 HITL 合同；不是研究失败或 API 不支持的同义词。

[D1]: https://developer.apple.com/documentation/eventkit/creating-events-and-reminders
[D2]: https://developer.apple.com/documentation/eventkit/ekcalendar
[D3]: https://developer.apple.com/documentation/eventkit/ekcalendaritem
[D4]: https://developer.apple.com/documentation/eventkit/ekreminder/startdatecomponents
[D5]: https://developer.apple.com/documentation/eventkit/ekreminder/duedatecomponents
[D6]: https://developer.apple.com/documentation/eventkit/ekreminder/priority
[D7]: https://developer.apple.com/documentation/eventkit/ekreminder/iscompleted
[D7b]: https://developer.apple.com/documentation/eventkit/ekreminder/completiondate
[D8]: https://developer.apple.com/documentation/eventkit/setting-an-alarm
[D9]: https://developer.apple.com/documentation/eventkit/ekstructuredlocation
[D9b]: https://developer.apple.com/documentation/eventkit/ekstructuredlocation/init(mapitem:)
[D10]: https://developer.apple.com/documentation/eventkit/ekevent/structuredlocation
[D11]: https://developer.apple.com/documentation/eventkit/accessing-the-event-store
[D12]: https://developer.apple.com/documentation/eventkit/ekeventstore
[D13]: https://developer.apple.com/documentation/eventkit/ekerror
[D14]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/recurrencerules
[D15]: https://developer.apple.com/documentation/eventkit/ekrecurrencerule
[D16]: https://support.apple.com/guide/reminders/add-subtasks-to-reminders-remn32a9622b/mac
[D17]: https://support.apple.com/guide/reminders/manage-sections-in-reminder-lists-remn14bf0e77/7.0/mac/27
[D18]: https://support.apple.com/en-qa/guide/reminders/remndc729e28/mac
[D19]: https://developer.apple.com/documentation/eventkit/eksource
[D20]: https://developer.apple.com/documentation/eventkit/ekeventstore/save(_:commit:)
[D21]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/location
[D22]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/url
[D23]: https://developer.apple.com/library/archive/featuredarticles/iPhoneURLScheme_Reference/MapLinks/MapLinks.html
[D24]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/notes
[D25]: https://developer.apple.com/documentation/eventkit/ekalarm/emailaddress
[D26]: https://developer.apple.com/documentation/eventkit/ekalarm/soundname
[D27]: https://developer.apple.com/documentation/eventkit/ekerror/reminderalarmcontainsemailorurl
[D28]: https://support.apple.com/en-afri/guide/calendar/icl58679aba2/mac
[S1]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift
[S2]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift
[S3]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/ReminderReadSource.swift
[S4]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/ReminderRecurrence.swift
[S5]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/AuthorizationStatusSource.swift
[S6]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager+ReminderCompletion.swift
[S7]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/ReminderCompletion.swift
[S8]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/ReminderWriteSource.swift
[S9]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Info.plist
[S10]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Entitlements.plist
[S11]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventFormattingSource.swift
[S12]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Validation.swift
[S13]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/UndoManager.swift
[S14]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitErrorSanitizer.swift
