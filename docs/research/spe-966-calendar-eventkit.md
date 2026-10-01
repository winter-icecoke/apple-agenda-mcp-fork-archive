# Calendar、来源、时间与重复日程的公开 EventKit 边界

> 研究交回快照：以下工作区、未 commit/push 等执行状态记录作者交回时的事实。主控于 2026-10-01 独立复核后集成；当前发布链接、状态与后续未知以研究票 resolution comment 为准。

调查日期：2026-10-01。研究票：[调查 Calendar 与来源、时间及重复日程的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-966)。父地图：[Apple Agenda MCP：日历与提醒全面覆盖决策地图](https://linear.app/icecoke/issue/SPE-961)。

本报告是研究输入。没有决定产品覆盖口径、时间合同或受限功能文案，没有执行产品实现、build、安装、MCP 切换或真实对象读写。报告留在本机 WIP；研究票由主控核对后再处理 resolution 与状态。

## 结论与适用范围

公开 EventKit 支持读取来源与日历、在允许的来源创建日历、修改日历名称/颜色、删除日历，以及日程 CRUD 和有限时间窗口的实例查询。来源本身的账户创建、登录、改名、删除、共享/委托授权管理没有公开 EventKit 写入入口。日历元数据的 `isImmutable` 与内容的 `allowsContentModifications` 是两个边界；账户类型不能替代它们或保存时的实际错误。[EKSource][EKCalendar][H-SOURCE][H-CAL][H-ERROR]

重复规则的公开表达能力明显多于当前 MCP：支持带序数的 weekday、负数 month day、指定月份、年内周/日和 set positions；当前 Calendar MCP 的输入及输出只覆盖一部分。原生 `EKSpan` 只有单次与当前及未来；当前 `span="all"` 是项目组合行为。原始发生时间 `occurrenceDate`、修改后开始时间 `startDate`、`isDetached` 必须分开。不能把一个 ID 或一个日期当作跨同步、移动、拆分后的稳定系列身份。[D-RECURRENCE][D-SPAN][D-OCCURRENCE][H-EVENT][H-ITEM][S-RECURRENCE][S-FORMAT]

全天、时区及委托来源存在重要的平台/版本差异。TN3130 说明 macOS 13 的新行为：全天 `endDate` 是末日 23:59:59；更改 `timeZone` 不再改变绝对时刻；`sources` 包含委托来源；按 ID 取重复首实例有例外。通用符号文档和 SDK 注释有些仍描述旧行为，本报告保留冲突，并以 macOS 专属技术说明约束结论，不能将它们当成本机实测。[D-MACOS13]

参与者列表、组织者和参与者状态可读；EventKit 明确不能新增参与者，组织者与参与者各属性只读。当前 MCP 可返回这些字段，没有公开原生邀请管理或 RSVP 写入能力。已有邀请的普通日程字段是否可改、保存/删除带来的服务端通知与不同来源限制仍需隔离测试；不能承诺不会通知其他人。[D-ATTENDEES][H-PARTICIPANT][H-EVENT][H-ERROR][S-PARTICIPANT]

地点、URL、notes、结构化地点与 alarms 详细矩阵属于 [调查 Reminders 原生结构、地点与提醒字段的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-967)。本报告只保留它们与复制、邀请或重复身份相关的交叉边界。

## 证据基线与状态含义

| 项目 | 本次记录 |
|---|---|
| 工作区 / branch | `/Users/leolau/Documents/Codex/worktrees/apple-agenda-chart/spe-966-calendar` / `research/spe-966-calendar` |
| 固定源码 | `3ad64088feb82104982b6b3e7181f6bb4ca8af94`；所有 S-* 链接固定在此 commit，不使用浮动 main |
| 源码声明版本 | `AppVersion.current = 1.18.0`，MCP identity `che-ical-mcp`；只是源码声明，非 release 或运行会话证据。[S-VERSION] |
| 部署下限 | `Package.swift` 为 macOS 14。[S-PACKAGE] |
| Host | `sw_vers`：macOS 27.0，build `26A428`；没有执行 EventKit 运行时探针 |
| 工具链 / SDK | `xcodebuild -version`：Xcode 26.6，build `17F113`；`xcrun --sdk macosx --show-sdk-path` 指向 macOS 26.5 SDK |
| binary / TCC / MCP 会话 | 本次均未检查或调用；不能将源码判断归给现用 MCP |
| 数据 | 未实例化事件 store 读取个人对象；未记录个人日历、日程、参与者或账户内容 |

状态栏区分：`API 支持但尚未实现`、`已实现但未验证`、`API 不支持（所查公开接口边界）`、`未知`。API/SDK 文档核对不等于运行时验证。表中验证层级 `A` 表示官方声明/公开 SDK，`S` 表示固定源码；本次没有 mock 执行、binary、CLI、长驻 MCP、独立系统回读、同步或用户验收证据。

MCP 错误约定简称 `E`：单调用错误经外层 catch 返回 `isError=true` 与文本 `Error: <code/message>`；`EKErrorDomain` 转为 `eventkit_error_<N>`，受信任的项目错误可保留作者定义的消息。不能把下面列出的 SDK 错误名字误当成当前已返回的机器可读错误类型。未调用的错误分支仅是源码路径。[S-DISPATCH][S-ERROR]

## Source / Calendar 能力矩阵

Calendar/event 侧的真实来源与日历读取受 Calendar full access 限制；Reminders 清单对应 Reminders 权限，由 [调查 Reminders 原生结构、地点与提醒字段的公开 EventKit 边界](https://linear.app/icecoke/issue/SPE-967) 覆盖。macOS 14 起 write-only 可保存新日程，但不能读取已有日程/真实日历列表、修改或删除已有日程；日历列表返回虚拟日历，读它的事件为空。没有单独 read-only 授权模式。当前项目对 Calendar 工具统一要求 full access，write-only 分支返回 `insufficientAccess`；这比“只创建新事件”的 API 最低权限要求更严格。[D-ACCESS][D-TN3153][D-WWDC23][H-TYPES][S-AUTH]

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具 / schema / 输出 / 错误 | 状态 | 验证层级与缺口 | 关联决策 |
|---|---|---|---|---|---|---|---|
| Source read：账户类型、名称、ID、委托标志 | `EKEventStore.sources`、`source(withIdentifier:)`；`EKSource.sourceIdentifier/sourceType/title/isDelegate`；[EKSource]、[D-SOURCES]；`EKSource.h:19–43`、`EKEventStore.h:98–118` | `isDelegate` macOS 13+；来源枚举类型不代表可写权限；不保证数组顺序 | `EventKitManager.swift:50–53,285–305`；只通过 Calendar.source 读 title/type；[S-STORE][S-CAL] | `list_calendars` 返回 `source` 名称、`source_type`；没有 Source 独立对象输出，未返回 `sourceIdentifier/isDelegate`；[S-CAL-HANDLER]；E | 名称/type 已实现但未验证；其余 API 支持但尚未实现 | A/S；未读取真实来源；同名账户的区分能力不足 | [覆盖口径][ID合同] |
| Source query：指定 source ID / 按 entity 取所属集合 | `source(withIdentifier:)`、`EKSource.calendars(for:)`；[EKSource]；`EKSource.h:32–37`、`EKEventStore.h:114–118` | 应使用 `calendars(for:)`；旧 `EKSource.calendars` 官方文档明确不适用于 macOS。[D-SOURCE-CALS] | 当前使用日历枚举再匹配 source title，不按 source ID 定位；[S-CAL] | `calendar_source` 是名称字符串，并且只有伴随 `calendar_name` 时参与过滤；不是独立 source filter；[S-LIST][S-SEARCH]；E | API 支持但尚未实现（ID/独立查询）；名称过滤已实现但未验证 | A/S；来源名称重复、重命名和空/默认来源未测 | [覆盖口径][ID合同] |
| Source create/update/delete：账户建立、登录、改名、移除 | Apple 明确 EKSource 由 store 提供，不能自行创建；公开 EKSource 属性只读，EKEventStore 无 save/remove source；[EKSource]；`EKSource.h:17–45`、[H-STORE] | 账户供应与账户级权限不在本票可用 API 中 | 无实现；当前 `create_calendar` 只选已有 source；[S-CAL] | 无工具/schema；现有工具缺失不是本结论依据 | API 不支持（所查公开 EventKit 的账户 CRUD） | A/S；未借用系统设置、AppleScript 或私有 API | [覆盖口径] |
| Source read/query：委托来源可见性 | `delegateSources`、`init(sources:)`、`isDelegate`；[D-DELEGATES]、[D-MACOS13]；`EKEventStore.h:75–80,102–118`、`EKSource.h:39–43` | 通用 delegateSources 文档说默认 sources 不含委托；TN3130 对 macOS 13 说明包含。macOS 14+ 项目不能套用前者断言遗漏；对象不能跨 store 使用 | 仅 `EKEventStore()`；不读 `isDelegate`，未显式构造委托 store；[S-STORE] | `list_calendars` 没有委托区分字段；E | API 支持；当前委托集合覆盖未知，标志 API 支持但尚未实现 | A/S；需按 macOS 版本、来源、权限实际枚举；文档冲突不等于实测失败 | [覆盖口径][ID合同] |
| Source update：授予/撤销共享与委托、改变服务器权限 | 所查 `EKSource/EKCalendar/EKEventStore` 无共享 ACL、邀请共享者或委托管理 setter；[EKSource][EKCalendar][H-SOURCE][H-CAL][H-STORE] | `isDelegate` 只是读取状态；权限由来源提供，不能据它授权写入 | 无实现 | 无工具/schema | API 不支持（所查公开 EventKit 的授权管理） | A/S；服务端管理 API 不在 Calendar + Reminders EventKit 研究范围 | [受限功能] |
| Calendar read/query：枚举与按 ID 读取集合 | `calendars(for:)`、`calendar(withIdentifier:)`；title、type、color、allowedEntityTypes、source、isSubscribed、isImmutable、allowsContentModifications、supportedEventAvailabilities；[EKCalendar]、[H-CAL]；`EKEventStore.h:136–160` | full access；API 可创建单 entity 集合，服务器可能返回混合 entity 集合；两个实体授权分别检查 | `listCalendars` 枚举 event/reminder；`findCalendar(s)` 名称/source 名称匹配；[S-CAL] | `list_calendars {type?:event/reminder}` 返回数组：id/title/type/allowsContentModifications/isSubscribed/source/source_type；没有 color/isImmutable/allowedEntityTypes/availability mask；[S-CAL-HANDLER]；E | 子集已实现但未验证；缺省字段 API 支持但尚未实现 | A/S；未省略 type 时会先要求 Calendar 与 Reminders 两项权限；混合集合可能重复出现，未测 | [覆盖口径] |
| Calendar create：指定已有来源的新集合 | `EKCalendar(for:eventStore:)`、设置 title/color/source、`saveCalendar(_:commit:)`；[EKCalendar][H-CAL][H-STORE] | source 只在创建时可设；一些来源不允许新增集合或不支持 event；可能抛 `calendarHasNoSource/sourceDoesNotAllowCalendarAddDelete/sourceDoesNotAllowEvents`；[H-ERROR] | `createCalendar:394–424`：按 title+entity 全局查重；event 分支使用 `defaultCalendarForNewEvents` 的 source，reminder 分支使用 `defaultCalendarForNewReminders` 的 source，各自默认 source 可能 nil；[S-CAL] | `create_calendar {title,type,color?}`；无 source ID/name 入参；返回 action created/skipped、id；[S-CAL-HANDLER]；E | 默认来源路径已实现但未验证；选择来源 API 支持但尚未实现 | A/S；同名但不同来源会被当重复；来源能力不能只按 type 推定 | [覆盖口径][ID合同] |
| Calendar update：title / color | 修改属性后 `saveCalendar`；`isImmutable` 控制元数据可修改/删除；[D-IMMUTABLE][H-CAL][H-STORE] | `isImmutable=true` 仍可能允许新增/改/删内容；`allowsContentModifications` 不等价于元数据可改；`calendarIsImmutable`；[H-ERROR] | `updateCalendar:427–452` 先要求两项实体权限，用 `allowsContentModifications` 作 guard，未判断 isImmutable；[S-CAL] | `update_calendar {id,title?,color?}` 至少一项；作者消息把 guard 失败写成 calendarNotFound/read-only；返回 action updated、id；[S-UPDATE-CAL]；E | 已实现但未验证；guard 与公开元数据边界不一致 | A/S；不能宣称全覆盖元数据可写性；需独立测试 immutable 与内容可写组合 | [覆盖口径] |
| Calendar update：移动集合到另一 Source / 改 type / 改订阅 URI | `source` 创建后实质只读；`type/isSubscribed` 只读，无订阅 URI setter；[H-CAL]；`calendarSourceCannotBeModified`；[H-ERROR] | 不能用迁移每个 event 冒充移动 Calendar；订阅/共享属性不在公开 setter 中 | 无实现；`update_calendar` 只 title/color | 无相应 schema | API 不支持（现有集合换来源、原生订阅建立/改 URI）；普通复制方案未决 | A/S；不以工具缺失作判断 | [覆盖口径][受限功能] |
| Calendar delete：集合及其中内容 | `removeCalendar(_:commit:)`；[EKCalendar][H-STORE] `EKEventStore.h:177–200` | isImmutable/来源限制；对混合 entity 集合，只有部分实体权限时可只删除已获权的实体并收缩 allowedEntityTypes；全获权时删集合及全部内容 | `deleteCalendar:455–464` 强制先要求 Calendar 与 Reminders 权限，然后 remove；[S-CAL] | `delete_calendar {id}` 返回 action deleted/id；没有 dry_run 或部分 entity 状态输出；[S-CAL-HANDLER]；E | 已实现但未验证；项目授权门槛更严格 | A/S；删除影响含集合所有日程，不能按工具名猜只删空集合；未写入验证 | [覆盖口径][状态合同] |

只读、订阅、生日和委托来源不能一律当成相同能力集。`EKCalendar.type` 取决于来源及订阅状态；CalDAV 订阅日历可仍是 `.calDAV` 且 `isSubscribed=true`。当前可读取 type/isSubscribed，但缺少 isImmutable、allowedEntityTypes 与可用 availability mask。服务器拒绝写入可能只在保存时暴露，公开 EKSource 没有通用的“允许新增日历”布尔量。[H-CAL][H-SOURCE][H-ERROR][S-CAL-HANDLER]

## Event CRUD / 查询矩阵

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具 / schema / 输出 / 错误 | 状态 | 验证层级与缺口 | 关联决策 |
|---|---|---|---|---|---|---|---|
| Event read：按 ID 取得日程 | `event(withIdentifier:)`；`calendarItem(withIdentifier:)`；[D-GET][D-RETRIEVE][H-STORE] | 通用文档称返回重复首实例；TN3130 对修改首实例等情况说明可能返回不同 occurrence；不能称为无条件“master” | `getEvent:1065–1071` 内部有调用；更新/删除也依赖 ID lookup；[S-GET][S-UPDATE] | 无独立 `get_event` 工具或 dispatch；`getEvent` 用于批量删除预览等内部路径；[S-DISPATCH] | API 支持但尚未独立暴露；内部已实现但未验证 | A/S；未来日期按 ID、修改首实例、detached ID 均未测 | [ID合同][时间合同] |
| Event query：时间窗口实例列表 | `predicateForEvents(withStart:end:calendars:)` + `events(matching:)` / `enumerateEvents(matching:using:)`；[D-PREDICATE][D-RETRIEVE]；`EKEventStore.h:286–331` | predicate 必须来自 store；同步调用；无顺序保证；单 predicate 超过 4 年被截为起点后的前 4 年；calendars=nil 表示可见集合 | `listEvents:493–508` 原样传窗口；Server 排序/filter/limit；[S-LIST][S-LIST-HANDLER] | `list_events` 必填 start_date/end_date；calendar_name/source 可选，filter/sort/limit/detail_level/fields/display_timezone；event_count 是过滤后、limit 前数量；E | 已实现但未验证；4 年以上完整查询尚未实现 | A/S；没有拆窗、cursor 或稳定 tie-break；窗口端点、跨界持续日程、同日空窗口未测 | [ID合同][时间合同] |
| Event query：关键词 / 快捷窗口 / 冲突 / 重复候选 | 先用上述公开窗口 predicate，再应用程序过滤；SDK 没有这些 MCP 特定高级查询；[D-PREDICATE][H-STORE] | 受窗口、可见来源和读取权限约束；不能用自建任意 NSPredicate 替换官方 predicate | `searchEvents:1084–1127` 搜 title/notes/location，默认 now±2年；冲突与重复也是枚举后逻辑；[S-SEARCH] | `search_events` 支持 keyword/keywords、any/all、窗口/集合；`list_events_quick/check_conflicts/find_duplicate_events`；[S-QUERY-SCHEMA]；E | 应用层实现但未验证；不是 API 原生去重/冲突服务 | A/S；候选启发式、顺序、分页与来源歧义需合同；不证明全历史完整性 | [ID合同][状态合同] |
| Event create：单次日程及基础属性 | `EKEvent(eventStore:)`、title/startDate/endDate/calendar/isAllDay/timeZone 等 setter；`save(_:span:)`；[D-CREATE][H-EVENT][H-ITEM][H-STORE] | 项目 full access；calendar 应支持 event 且内容可写；SDK `calendarReadOnly/eventNotMutable/noCalendar/noStartDate/noEndDate/datesInverted` 等；[H-ERROR] | `createEvent:528–670`；calendar_name 强制；title 与约±30秒窗口查重，返回已有对象；[S-CREATE] | `create_event {title,start_time,end_time,calendar_name,...}`；action created/skipped、id；未返回完整实际保存对象；[S-CREATE-HANDLER]；E | 已实现但未验证 | A/S；“skipped”只证明启发式匹配，不证明全字段相等；同步后的保存结果未回读 | [覆盖口径][状态合同] |
| Event update：普通字段 / 改所属 Calendar | 设置已有 EKEvent 属性、calendar 后 `save(_:span:)`；[D-CREATE][H-ITEM][H-STORE] | 内容可写不保证每个 invite 可移动；`invitesCannotBeMoved/eventNotMutable`；换日历时 eventIdentifier 可能变化；[H-ERROR][D-EVENT-ID] | `updateEvent:772–939`；仅 startDate 时按原绝对秒数保留 duration；calendar_name/source 定位目的集合；[S-UPDATE] | `update_event {event_id,...}`；返回传入的旧 event_id，不读保存后 event.eventIdentifier；[S-UPDATE-HANDLER]；E | 已实现但未验证；ID 回传完整性有源码缺口 | A/S；跨 source 移动、身份更新、只读来源/邀请行为未测 | [ID合同][时间合同][状态合同] |
| Event delete：非重复日程 | `remove(_:span:)`；[D-CREATE][H-STORE] | full access、event mutable；SDK 说明 remove 后除 eventIdentifier 外属性被清空；已有邀请的通知效果不能从这份 API 文档保证 | `deleteEvent:966–991` 非重复走 this/future 输入；[S-DELETE] | `delete_event {event_id,span?,occurrence_date?}` 返回 action deleted/id/span；[S-DELETE-HANDLER]；E | 已实现但未验证 | A/S；未证明不存在远端副作用、同步完成或 undo 恢复等价 | [状态合同][受限功能] |
| Event create/delete：复制 / 复制后移除原对象 | 公开 API 可新建 EKEvent 后 save/remove；没有“复制一切并保证身份/邀请/系列不变”的 API；[D-CREATE][H-ITEM][H-EVENT] | 组织者/参与者只读；源/目标分别受权限限制；失败可有目标成功而源未删的状态 | `copyEvent:1357–1408` 手工复制 title/start/end/notes/location/url/isAllDay/timeZone 和相对 alarm；未复制 recurrenceRules/participants/organizer；删除只 `.thisEvent`；[S-COPY] | `copy_event {event_id,target_calendar,target_calendar_source?,delete_original?}`；`move_events_batch` 逐项复制+删除；new_id 及每项结果；[S-COPY-HANDLER]；E | 已实现但未验证；不是系列原生移动 | A/S；会新建身份与独立发生，不能承诺参与者/例外/结构化地点无损；详细 alarm/地点边界看 SPE-967 | [ID合同][状态合同][受限功能] |
| Event read/update：availability / status / birthday identity | availability 可写且按 calendar mask 支持；status、birthdayContactIdentifier 只读；[D-AVAILABILITY][D-STATUS][H-EVENT][H-CAL] | availability `.notSupported` 表示来源不支持；status 只有 canceled 被官方描述为可操作可靠，其他为信息；birthday ID 限生日集合 | 当前 Calendar formatter 与输入无这些字段；[S-FORMAT][S-EVENT-SCHEMA] | 无对应 schema；不能把它们当 API 不支持 | API 支持但尚未实现（可读及 availability 可写子集）；status/birthday setter API 不支持 | A/S；来源差异未测；Cancel/RSVP 不可由 status setter 冒充 | [覆盖口径][受限功能] |

SDK 还说明，保存一个已被其他 store/process 删除的旧 EKEvent 可能重新创建日程；数据库变更通知到来时应将已有实例视为失效，重新查询或对正在编辑的对象 `refresh()`。当前 `refreshIfNeeded` 只在本项目写入标记后调用 `refreshSourcesIfNecessary`，所查 manager 未注册 `EKEventStoreChanged` 监听。后者只在必要时启动同步，不能证明远端完成；通知、失效对象与外部编辑/删除需要独立合同和测试。[H-STORE] `EKEventStore.h:230–243,438–475`、[H-EVENT] `EKEvent.h:154–170`、[S-REFRESH]

## 全天、时区与 DST 矩阵

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具 / schema / 输出 / 错误 | 状态 | 验证层级与缺口 | 关联决策 |
|---|---|---|---|---|---|---|---|
| Event read/create/update：全天标志和日期范围 | `isAllDay/startDate/endDate` 可读写；floating 开始值按默认时区返回；[D-START][H-EVENT]；macOS 13+ endDate 新表示见 [D-MACOS13] | API 层日期仍是 Date；不能由字段 setter 推出 MCP 应采用 inclusive/exclusive 日期合同 | 创建/更新直接解析 Date 后赋值；`validateTimeRange` 仅排除 timed 的零/倒序；全天豁免；[S-CREATE][S-UPDATE] | `all_day` JSON boolean；当前要求与 timezone 互斥；输出 ISO start/end、is_all_day；E | 已实现但未验证；多日/结束边界合同未知 | A/S；同日与跨多日、末日23:59:59/次日00:00 round-trip、timed↔all-day 未测 | [时间合同] |
| Event read/create/update：指定时区与 floating timed | `EKCalendarItem.timeZone` 可读写；nil 是 floating，start/end 应按系统时区设置；[D-TIMEZONE][H-ITEM]；TN3130 更改时区不再改绝对时刻 | nil 不等于“永久固定为当前系统时区”；来源/系统 round-trip 未测 | 创建/更新赋 timeZone，clear_timezone 赋 nil；[S-CREATE][S-UPDATE] | timezone 是 IANA string；clear_timezone 与 timezone 同时给拒绝；读回 `timezone` 却把 nil 替换为 TimeZone.current.identifier；[S-UPDATE-HANDLER][S-EVENT-OUTPUT]；E | 已实现但未验证；floating 状态 API 支持但尚未准确表达 | A/S；当前输出不能区分 explicit=current 与 nil/floating；不能声称 clear_timezone 是固定当前区 | [时间合同] |
| Event query/read：展示时区 | 官方日期对象与 formatter 是分层问题；floating `startDate/occurrenceDate` 在默认区返回；[D-TIMEZONE][D-OCCURRENCE][H-EVENT] | display timezone 不能改变源对象语义 | `formatEventDict` 只用 displayTimezone 改 `*_local`；`timezone` 保持源时区或系统 fallback；[S-EVENT-OUTPUT] | list/search/quick 有 display_timezone；ISO字段和 *_local 及 timezone 并存；[S-LIST-HANDLER] | 应用层已实现但未验证 | A/S；没有 floating flag、日期精度、offset 明确结构；display/source/system 不得混用 | [时间合同] |
| Event create/update：全天同时 timezone | 两个公开 setter 存在；所查官方来源未说明任意来源/系统组合都必然非法；[H-EVENT][H-ITEM] | 当前源码注释称会剥离全天标志，这是项目经验/注释，未在本次验证其普遍性 | 两层 guard 阻止 all_day+timezone；[S-CREATE][S-UPDATE] | 返回 allDayTimezoneConflict/invalidParameter；`clear_timezone` 可用于已有全天；[S-TIME-PARSER]；E | 当前项目不支持该组合；API 具体归一化行为未知 | A/S；应保留为合同选择或待测约束，不能标成 Apple API 不支持 | [时间合同] |
| Event create/update/query：DST 跳过/重复的本地时刻 | EventKit 接收 Date 和 TimeZone；WWDC23 要求 Calendar/DateComponents 作日期计算，以免 DST 错误；[D-WWDC23][H-EVENT][H-ITEM] | 所查公开 EventKit 接口没有 gap/fold policy 或实例展开 DST 策略参数；不能推定非法时刻会拒绝、偏移或选哪一次 | `parseFlexibleDate` 用 ISO8601DateFormatter、DateFormatter、Calendar.date；未显式 round-trip 校验或 fold/gap 决策；[S-TIME-PARSER] | explicit offset 优先；无 offset/date-only 用请求timezone或系统；time-only依“今天”；没有 DST policy 入参；E | Date/zone 路径已实现但未验证；DST 结果未知 | A/S；春季缺失时刻、秋季双重时刻、跨转换重复与全天窗口均待测 | [时间合同] |
| Event update：日期移动后的 duration | 开始/结束 setter 可分别改；API 本身不决定“保留墙上时钟时长还是绝对秒数”；[H-EVENT] | 跨 DST 绝对秒数与民用小时的含义可能不同 | update 仅给 start_time 时 `endDate = newStart + 原timeInterval`；[S-UPDATE] `824–835` | schema 说保留原 duration；输出不附计算语义；E | 绝对秒数实现但未验证；产品语义未决 | A/S；23/25小时全天、跨时区、改 timezone 是否保留 instant/clock 需决定 | [时间合同] |

源码风险输入：`update_event.occurrence_date` 用请求的 timezone 解析；若省略，则先按系统区解析，manager 再按已有事件区取那一天。`delete_event` 则先读取已有事件区再解析日期。系统区与事件区不同且传 date-only 时，两条路径可能落在不同民用日期。此处是固定源码推断，不是失败实测；合同需要规定 occurrence 日期所属时区、是否允许新时区影响目标定位。[S-UPDATE-HANDLER] `1390–1405`、[S-DELETE-HANDLER] `1463–1466`、[S-OCCURRENCE] `949–963`

## 重复规则、单次实例与系列矩阵

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具 / schema / 输出 / 错误 | 状态 | 验证层级与缺口 | 关联决策 |
|---|---|---|---|---|---|---|---|
| Recurrence read：完整公开规则 | `recurrenceRules/hasRecurrenceRules`；频率、interval、end、weekday+weekNumber、monthDays、months、weeksOfYear、daysOfYear、setPositions、firstDayOfWeek；[D-RECURRENCE][H-RULE][H-DAY] | nil/空数组不能照搬旧行为：TN3130 说明 macOS13 无规则返回空数组；source 能力与保存归一化未知 | EventFormattingSource.swift:73–75,109–133 只输出 frequency/interval/end/count/weekday rawValue/monthDays；[S-FORMAT] | list/search 标准结果 `recurrence_rules` / `event_recurrence_rules` 同一部分子集，未返回 ordinal、year字段、setPositions、firstDayOfWeek；E | 部分已实现但未验证；完整读取 API 支持但尚未实现 | A/S；现有复杂系列经 MCP 输出无法完整重建；勿以 Reminder 的完整 formatter 代推 Calendar | [覆盖口径][时间合同] |
| Recurrence create：daily/weekly/monthly/yearly 基础规则 | `EKRecurrenceRule(recurrenceWith:interval:end:)` / 复杂 initializer；`EKRecurrenceEnd(end:)` 或 occurrenceCount；设置 recurrenceRules 再 save；[D-RECURRENCE][H-RULE][H-END] | interval/count 必须正；某些不适用组合被忽略；来源可能拒绝持续时长超过间隔等；[H-ERROR] | `createRecurrenceRule:1875–1905` 构造复杂 rule，years/setPositions 等给 nil；[S-RECURRENCE] | create/update/batch 支持 frequency/interval/end_date/occurrence_count/days_of_week/days_of_month；[S-EVENT-SCHEMA][S-PARSER]；E | 子集已实现但未验证 | A/S；正数/组合输入验证不足见下文；日期端点/count含首实例与删除后计数未测 | [时间合同] |
| Recurrence create：每月第n weekday、负 month day、年内复杂规则 | `EKRecurrenceDayOfWeek(_:weekNumber:)`；复杂 initializer 的 months/weeks/days/setPositions；[D-RECURRENCE][H-RULE][H-DAY] | weekday ordinal 仅 monthly/yearly 有义；month day ±1…31；year fields 各有适用频率；SDK 会忽略不适用组合 | builder weekday 只有 weekNumber=0；monthDays parser 仅1…31；其他 fields 无解析；[S-RECURRENCE][S-PARSER] | 无 corresponding input；输入字典没有通用未知键拒绝，可被忽略，不能承诺传入高级 field 自动生效 | API 支持但尚未实现 | A/S；小时/分钟级重复不是公开 frequency enum；完整 RFC5545 文本导入/编辑也无公开入口 | [覆盖口径][时间合同] |
| Recurrence update：换规则 / 改截止 / 移除 | 官方类说明要求构造新规则再替换；`recurrenceRules` setter/add/remove；但 `recurrenceEnd` 本身在 SDK/符号文档是 get-set；[D-RULE][D-RULE-END][H-RULE][H-ITEM] | 文档“所有属性不可直接改”的概括与 recurrenceEnd setter 有差异；不应直接宣称 end 只读；保存 span 决定范围 | 当前替换数组或 clear_recurrence=nil；没有仅改 rule end 的专门工具；[S-UPDATE] `917–921` | update_event recurrence / clear_recurrence；与 exclusions 同传会拒绝 exclusions；[S-PARSER]；E | 替换/清除已实现但未验证；end setter 路径未实现 | A/S；修改现有复杂规则时可能降为可表达子集；已有 exceptions 的保留未知 | [时间合同][状态合同] |
| Occurrence read/query：detached 与原始发生时间 | `EKEvent.isDetached`、`occurrenceDate` 只读；后者在改 startDate 后仍保留原定时刻；[D-DETACHED][D-OCCURRENCE][H-EVENT] | detached 是属于重复且属性偏离默认；不是独立系列或“已删除实例”标志 | `findOccurrence` 按日期窗口枚举、找到第一个 eventIdentifier 相同者；没有比较原 occurrenceDate；[S-OCCURRENCE] | 读结果没有 is_detached/occurrence_date；调用却要求 occurrence_date；[S-FORMAT][S-EVENT-SCHEMA] | 原生读取 API 支持但尚未实现；现有定位实现但未验证 | A/S；跨日移动 detached、同日多实例、身份变化可能无法定位；缺系列/实例 identity envelope | [ID合同][时间合同] |
| Occurrence update/delete：单次 | `save/remove` 配 `.thisEvent`；[D-SPAN][D-CREATE][H-STORE] | 修改实例可形成 detached；具体新ID/例外保存、source同步行为无普遍保证 | manager 要 recurring this/future 提供 occurrence_date；枚举定位后 save/remove；[S-UPDATE][S-DELETE][S-OCCURRENCE] | update/delete_event span=this；缺 date 返回项目 invalidTimeRange；找不到返回 eventNotFound；schema日期可含时间但代码按整天搜索；E | 已实现但未验证 | A/S；时间精度不足、detached移动、原始/当前日期取舍未决 | [ID合同][时间合同] |
| Series update/delete：当前及未来 | 原生 `.futureEvents` 影响选定实例及之后实例；[D-SPAN][H-STORE] `16–29` | 不是“只未来、不含当前”；范围锚点必须是正确 occurrence；分裂/ID/异常传播未测 | 调用 span=future，目标来自 findOccurrence；[S-UPDATE][S-DELETE] | occurrence_date 对 recurring 必须；更新返回旧输入id；E | 已实现但未验证 | A/S；前半系列、后半系列、已detached实例、截止/count、ID映射未测 | [时间合同][ID合同][状态合同] |
| Series update/delete：整个系列 | 没有 `.allEvents`；可组合 ID查取起点与 futureEvents，但需先证明定位起点；[D-SPAN][D-GET][D-MACOS13][H-STORE] | 通用 first occurrence 文档有 macOS13 特例；不存在公开独立 master 对象接口保证 | update all=ID查取对象+.futureEvents；deleteSeries 相同；源码称 master，但官方契约更窄；[S-UPDATE][S-DELETE][S-UPDATE-HANDLER] | span=all，不要求 occurrence_date；E | 项目组合实现但未验证；native all span API 不支持 | A/S；不能无条件保证 detached首实例、历史拆分/同步后“整个系列”闭合 | [时间合同][ID合同] |
| Recurrence create/delete：排除指定实例 | 所查 SDK rule/item/event 无公开 EXDATE/RDATE 数组、setter 或 exception集合；原生 remove occurrence + thisEvent 可删除某次；[H-RULE][H-ITEM][H-EVENT][D-SPAN] | 没有 EXDATE 不等于不能排除实例；也不能用缺失实例反推完整排除列表或原因 | 创建后双阶段 resolve all → remove each thisEvent；失败补偿删新系列；[S-EXCLUDE][S-EXCLUSION-EXEC] | create_event/create_events_batch 的 recurrence.excluded_occurrence_dates max100；不能排首日、超窗、去掉全部count；返回请求归一化日期/count；update传该字段拒绝；E | 排除组合已实现但未验证；直接EXDATE读写 API 不支持 | A/S；结果是请求回显，不是公开EXDATE读取；补偿失败会留partial状态；重试不能发现额外既有排除 | [时间合同][状态合同] |
| Recurrence read：枚举所有exceptions或恢复被删单次 | 公共查询可取得现存 occurrence / detached 信息；没有删除例外清单或恢复指定 EXDATE 的公开 setter；[H-RULE][H-EVENT][H-STORE] | 空结果可能是规则不生成、窗口/权限/同步/删除；没有公有取消排除操作 | 无 exception inventory；现有 undo snapshot重建不是原身份例外恢复证明；[S-OCCURRENCE][S-EXCLUDE] | 无直接工具/schema；通用 undo 需要同一会话另验 | EXDATE inventory/直接恢复 API 不支持；等价组合方案未知 | A/S；不能重新创建独立日程后声称恢复原系列实例 | [状态合同][ID合同] |

补充两项声明边界：

- `EKRecurrenceRule` 公开 frequency 只有 daily/weekly/monthly/yearly；并非任意 RRULE 文本解析器。没有公开 hourly/minutely frequency、EXDATE/RDATE 参数或 setter。此结论来自实际公开类与 enum 的检查，不能扩大为 Calendar 原生存储永远不能表示这些信息。[H-TYPES] `56–68`、[H-RULE]、[H-ITEM]
- 当前 schema 文案说 end_date 与 occurrence_count 互斥，但 parser 可同时解析，builder 优先 endDate；所查 Calendar parser 没有 interval>0 或 count>0 检查。SDK 对零值初始化注明可能抛 Objective-C exception，不能假定落入普通 Swift throw/MCP E。此处只做源码风险记录，没有运行非法输入。[S-PARSER] `3066–3084,3147–3155`、[S-RECURRENCE] `1889–1894`、[H-RULE] `33–48`、[H-END] `22–26`

## 参与者与邀请矩阵

| 能力 / 对象与操作 | 公开 API 与来源 | 来源/权限限制 | 当前实现与源码 | MCP 工具 / schema / 输出 / 错误 | 状态 | 验证层级与缺口 | 关联决策 |
|---|---|---|---|---|---|---|---|
| Event read/query：attendees / organizer | `EKCalendarItem.attendees/hasAttendees`、`EKEvent.organizer`；[D-ATTENDEES][H-ITEM][H-EVENT] | full access；nil/空取决于日程与来源；不能据缺字段推导服务器不支持邀请 | EventFormattingSource:89–94 与 ParticipantFormatting:55–88；[S-FORMAT][S-PARTICIPANT] | list/search标准结果可返回 attendees、organizer；summary默认省略，fields可指定；E | 已实现但未验证 | A/S；真实各source返回、隐私脱敏要求、缺失/未知值未验 | [覆盖口径][受限功能] |
| Participant read：身份、role/type/status/current user | `EKParticipant.URL/name/participantRole/participantType/participantStatus/isCurrentUser` 只读；[H-PARTICIPANT][H-TYPES] | URL不一定mailto；联系信息查询涉及Contacts，范围不扩展 | helper从mailto提取email，其他URL原样放email；有is_current_user；organizer只返name/email/current_user；[S-PARTICIPANT] | 无独立 participant工具；read输出状态strings | 已实现但未验证（输出子集） | A/S；字段email可能实际是非mail URL；role/status不代表写入权限 | [受限功能][ID合同] |
| Event create/update/delete：新增/移除参与者、设置组织者 | attendees官方明确不可新增；organizer、EKParticipant属性只读，无公开邀请者构造/修改方法；[D-ATTENDEES][H-PARTICIPANT][H-EVENT] | full access 不会使readonly变可写 | 无实现；copy也没有能力复制参与者/组织者；[S-COPY] | 无输入/schema | API 不支持（直接邀请/参与者/组织者管理） | A/S；不通过notes、复制、URL或私有类冒充原生邀请 | [受限功能] |
| Participant update：接受/拒绝/暂定/委托回复 RSVP | participantStatus是readonly；EKEvent.status同样readonly；所查公有class无reply方法；[H-PARTICIPANT][D-STATUS][H-EVENT] | status可读不等于可设置回复；删事件不等于拒绝邀请 | 无实现 | 无工具/schema | API 不支持（直接 EventKit RSVP 写入） | A/S；现有邀请编辑/删除的远端效果另列未知 | [受限功能] |
| Event update/delete：已有邀请的普通字段/跨集合移动/取消 | 普通可写字段仍可尝试save/remove；SDK有 eventNotMutable、invitesCannotBeMoved；status canceled是可读信息，取消应remove而非赋status；[H-EVENT][H-ERROR][D-CREATE] | 来源、当前人角色、邀请权限具体矩阵未知；写入可同步到远端，并非私人本地patch保证 | update_event可改普通字段/calendar；copy/move通过新建再删；[S-UPDATE][S-COPY] | 现有普通工具，E只暴露SDK数码或作者错误；没有invite特定副作用描述 | 基础路径已实现但未验证；source/通知行为未知 | A/S；没有保证保留邀请、自动发通知/不发通知、参会方与组织方权限相同 | [受限功能][状态合同] |

## 身份变化与待验证状态表

以下是官方保证、源码处理与未知的分界，不是已决定的数据模型。公开接口中的“unique”不能解释成永久、跨设备、跨同步且全局唯一。[H-CAL][H-ITEM][H-EVENT]

| 操作/变化 | 官方/SDK 可确认的边界 | 当前处理 | 必须补的证据 |
|---|---|---|---|
| Calendar full sync | `calendarIdentifier` 不保证同步稳定；需要失效后的处理方案；`EKCalendar.h:59–66` | write用id，query多用title/source title | 失效ID、同名多来源、rename后重查；不可盲猜同名对象 |
| Event move/sync | `eventIdentifier` 换calendar很可能变化，sync也可能变化；`EKEvent.h:45–61` | update返回旧输入ID；copy回newID；无映射envelope | 保存后新ID、旧ID可读性、跨source远端回读、重试 |
| 外部ID | `calendarItemExternalIdentifier` 跨设备参考，重复/导入/共享/被邀请/委托/订阅可导致多个匹配；重复所有实例共用；Exchange有跨iOS/macOS差异；`EKCalendarItem.h:45–70` | Event输出无external ID、calendar/source ID | API是候选集，不是唯一键；多匹配和source范围验证 |
| Detached改startDate | occurrenceDate保留原始定时刻；isDetached只读；`EKEvent.h:130–152` | 只输出startDate，无original occurrenceDate/isDetached；lookup按当前日期窗+ID首匹配 | 跨日移位、首实例detach、重复同日候选、时区改变后的定位 |
| future split / all | `.futureEvents`是当前及之后；TN3130 ID lookup例外，SDK无allspan | 源码假定ID lookup所得为master/起点 | 拆分前后两个规则与ID、旧/新实例、已改/删例外是否保留；不能只验数量 |
| 删除后restore/undo | remove使对象失效；新建对象与新ID不是原对象复活保证；`EKEventStore.h:254–275` | snapshot/undo是项目组合行为，只有同一会话能验history | 重复规则+detached+排除+邀请的等价性、部分恢复、外部修改后的冲突；详交状态合同票 |
| store change / 权限变化 | 收到EKEventStoreChanged应重取/refresh；save旧且已被删除对象可能新建；`EKEventStore.h:230–243,461–475` | 仅本项目写入flag触发必要时sync；所查manager无通知监听 | 外部删/改、TCC撤销、远端延迟、缓存对象保存；不可把refresh调用当同步完成 |

## 未执行的验证与下一步研究问题

本次只到 A/S 层。下列验证须在主控另行授权后，用独立可识别测试集合、保存创建ID、逐项独立系统回读并清理本次对象；不使用真实个人日程当试验样本。没有替代用户的 HITL 决定。

| 隔离情景 | 要观察的具体事实 | 当前证据缺口 / 输入票 |
|---|---|---|
| macOS14+、当前host、普通/订阅/生日/委托source | sources与delegateSources的实际内容、isDelegate、calendar权限组合、可读/保存错误 | 解决通用docs与TN3130的适用差异；[覆盖口径] |
| 单天/多天全天，转timed，系统区改变 | 实际保存后的start/end/isAllDay/timeZone、末日和查询端点 | 当前guard豁免及date-only解析不足以决定民用日期合同；[时间合同] |
| America/New_York等春秋DST、跨时区重复 | gap/fold选择、offset、wall clock、absolute duration、跨转换query日界 | EventKit公开API没有策略参数；当前parser未经runtime检查；[时间合同] |
| recurring首实例修改/删除、移动detached到另一天 | ID lookup返回什么、occurrenceDate、startDate、isDetached、对象与series关系 | TN3130例外与现有日期定位冲突；[时间合同][ID合同] |
| this/future/all、规则替换/清空 | 前后规则、detached/排除实例、过去/未来影响、ID变化、服务端归一化 | 无原生allspan；不能从MCP action证明修改范围正确；[时间合同][状态合同] |
| exclusions成功、resolve失败、remove失败、补偿失败、重试 | 实际剩余系列/发生、已应用排除、new/oldID、错误partial state | 本票只读源码，非事务等价性证据；[状态合同] |
| Calendar metadata/content权限不同的组合 | isImmutable与allowsContentModifications分别效果、create/delete source限制 | 当前metadata guard用错层次的风险未实测；[覆盖口径] |
| 已有邀请：组织方/参会方，来源差异 | 哪些普通字段能改、移动错误、保存/删除的通知与远端状态 | 不新增真实参与者或主动发送消息；[受限功能][状态合同] |
| 查询>4年、同起点候选、跨窗口持续事件 | 拆窗覆盖、端点、排序、去重、截断元数据、source筛选 | 单predicate 4年硬边界；无Calendar cursor合同；[ID合同] |

not-run：`swift build/test`、产品binary执行、CLI/MCP调用、TCC查询/申请、系统日历/提醒对象访问、签名/安装/配置切换、同步与远端账户测试、Git commit/push、Linear写入。既有测试文件的存在或源码注释中的上游“verified”字样均未升级成本次验证证据。

仍未完成：本机/来源运行时行为、完整原生例外/系列身份模型、所有source的写入权限/同步副作用和MCP错误可处理合同。这些是明确的研究/实验缺口，不是已完成覆盖。报告交主控核对，不据此关闭依赖决策票。

## 命令与采集记录

| 实际执行 | 结果 / 用途 |
|---|---|
| `pwd`、`git status --short --branch`、`git rev-parse HEAD`、`git remote -v` | exit 0；确认指定工作区、research branch、基线及fork/upstream；起始工作区无改动 |
| `cat` 项目AGENTS、agent-execution、validation、CONTEXT、capability-matrix模板、Day1、PROJECT_BRIEF、progress-map、research SKILL | exit 0；只读取约定；research角色不递归派工 |
| `rg --files Sources Tests docs/research` | exit 2：基线没有docs/research目录；Sources/Tests列表成功，不代表研究中断 |
| `rg -n` manager/server/formatter/parser及相关tests，`nl -ba ... \| sed -n '起,止p'` | 所用证据读取exit 0；验证源码路径/行号；部分无匹配rg exit 1（如Validation.swift中的recurrence验证），按“无匹配”处理，不宣称运行测试 |
| `xcrun --sdk macosx --show-sdk-path`、`xcodebuild -version`、`sw_vers` | exit 0；仅工具链/系统版本，不是build或EventKit运行证据 |
| `nl -ba` SDK公开头文件；`shasum -a 256` 11个头文件 | exit 0；声明/注释/可用性与可复核hash见下表 |
| web官方搜索/open；Apple页面View Markdown链接 | 浏览返回文档/搜索内容；部分`.md`链接报`Unsupported content-type: text/markdown`，不作为无内容/不支持API依据 |
| `curl --fail --silent --show-error https://developer.apple.com/documentation/... .md`（精确链接见官方来源） | exit 0；读回retrieving/creating、delegateSources、isDelegate、TN3130、sources、endDate、recurrenceEnd；未生成临时文件 |
| `curl --head --fail --silent --show-error` 固定manager GitHub permalink | exit 0、HTTP 200；确认固定commit源码URL可达，不证明报告已发布 |
| `git diff --check` | exit 0；只检查tracked差异，不能代替新增报告检查 |
| `git diff --no-index --check /dev/null docs/research/spe-966-calendar-eventkit.md` | exit 1、无错误输出：新增文件与空文件不同；未发现whitespace问题，没有修改index |
| Python文档检查：引用定义、8列能力表、固定commit/源码行号范围 | exit 0；75个引用定义，无未定义证据引用，无表格或源码范围错误；没有产品测试 |
| Python并行HTTP读取全部官方documentation引用对应`.md` | exit 0；25/25官方文档HTTP 200并返回Markdown；仅验证可访问，未把文档声明提升为runtime证据 |

内存快速检索 `MEMORY.md` 的 apple-agenda/EventKit/SPE-966 无匹配（exit 1），未使用历史记忆作为结论来源。全部API证据来自本次读取。

## SDK 公开头文件

SDK根目录为 `/Applications/Xcode-26.6.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX26.5.sdk`；下表路径相对 `System/Library/Frameworks/EventKit.framework/Headers/`。引用H-*链接到此节，表内保留行号与sha256，避免以非官方SDK镜像替代本机头文件。

| 文件 / 本报告使用的范围 | SHA-256 |
|---|---|
| EKSource.h:17–45 | `ee8514241aef819a89403c231baeb73a858eeb9273aacb7ace5067935bcc313d` |
| EKCalendar.h:38–130 | `3eb9f759054a392b3c4985d4a65af84a62b399b0a201f0570e39ff4b9db8b510` |
| EKEvent.h:32–179 | `b2f71613e98f0f58e7c8d366cc9139ecf169512608738cf4a098da02f1ebc36b` |
| EKCalendarItem.h:28–125 | `e7a810b989480305fccf1fa2dde439d25042b8b49b580187bfed12ccf41fedf0` |
| EKEventStore.h:16–331,419–475 | `0a9c7c4f80a4bc19060360585c5edb903be680938c63aefcee6d1efb22699f43` |
| EKRecurrenceRule.h:19–189 | `751505020abaf956d52907fed8f92a299c4a7adf6d508696e5bb8cc55808ab44` |
| EKRecurrenceEnd.h:13–57 | `72af4ffa2bef64cd8927e8495bf5f0208716111837dbbdede41610352a50b43c` |
| EKRecurrenceDayOfWeek.h:14–67 | `4ef8600d7a3d5d41fca9c29b3c4f999715be7c53400624e24909b95162cc94aa` |
| EKParticipant.h:29–84 | `26d88ed13db37d39cc2fcc34e87cc1a69d4d7f2b5d401771c3cebb05b31bfc1a` |
| EKTypes.h:12–200 | `0d143c805633040ef21b861060676baaa0c82ee1dd250991d5a4bbf9dc44fda6` |
| EKError.h:19–104 | `b53bf9d8424b2f588487c91bbdb525b5300bba9262ff7ae616a955dd0d75c649` |

## 官方与固定源码引用

Apple符号/文章URL为在线可变文档，读取日如上；SDK为上述固定本机文件。macOS特定TN3130与通用符号描述冲突处已在正文列明。没有引用社区回复、第三方博客、Calendar私有数据库或非公开头文件。

[EKSource]: https://developer.apple.com/documentation/eventkit/eksource
[EKCalendar]: https://developer.apple.com/documentation/eventkit/ekcalendar
[D-SOURCES]: https://developer.apple.com/documentation/eventkit/ekeventstore/sources
[D-SOURCE-CALS]: https://developer.apple.com/documentation/eventkit/eksource/calendars?language=objc
[D-DELEGATES]: https://developer.apple.com/documentation/eventkit/ekeventstore/delegatesources
[D-IMMUTABLE]: https://developer.apple.com/documentation/eventkit/ekcalendar/isimmutable
[D-ACCESS]: https://developer.apple.com/documentation/eventkit/accessing-the-event-store
[D-TN3153]: https://developer.apple.com/documentation/technotes/tn3153-adopting-api-changes-for-eventkit-in-ios-macos-and-watchos
[D-MACOS13]: https://developer.apple.com/documentation/technotes/tn3130-changes-to-eventkit-in-macos13-ventura
[D-WWDC23]: https://developer.apple.com/videos/play/wwdc2023/10052/
[D-CREATE]: https://developer.apple.com/documentation/eventkit/creating-events-and-reminders
[D-RETRIEVE]: https://developer.apple.com/documentation/eventkit/retrieving-events-and-reminders
[D-PREDICATE]: https://developer.apple.com/documentation/eventkit/ekeventstore/predicateforevents(withstart:end:calendars:)
[D-GET]: https://developer.apple.com/documentation/eventkit/ekeventstore/event(withidentifier:)
[D-EVENT-ID]: https://developer.apple.com/documentation/eventkit/ekevent/eventidentifier
[D-START]: https://developer.apple.com/documentation/eventkit/ekevent/startdate
[D-TIMEZONE]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/timezone
[D-RECURRENCE]: https://developer.apple.com/documentation/eventkit/creating-a-recurring-event
[D-RULE]: https://developer.apple.com/documentation/eventkit/ekrecurrencerule
[D-RULE-END]: https://developer.apple.com/documentation/eventkit/ekrecurrencerule/recurrenceend
[D-SPAN]: https://developer.apple.com/documentation/eventkit/ekspan
[D-OCCURRENCE]: https://developer.apple.com/documentation/eventkit/ekevent/occurrencedate
[D-DETACHED]: https://developer.apple.com/documentation/eventkit/ekevent/isdetached
[D-ATTENDEES]: https://developer.apple.com/documentation/eventkit/ekcalendaritem/attendees
[D-AVAILABILITY]: https://developer.apple.com/documentation/eventkit/ekevent/availability
[D-STATUS]: https://developer.apple.com/documentation/eventkit/ekevent/status
[H-SOURCE]: #sdk-公开头文件
[H-CAL]: #sdk-公开头文件
[H-ITEM]: #sdk-公开头文件
[H-EVENT]: #sdk-公开头文件
[H-STORE]: #sdk-公开头文件
[H-RULE]: #sdk-公开头文件
[H-END]: #sdk-公开头文件
[H-DAY]: #sdk-公开头文件
[H-PARTICIPANT]: #sdk-公开头文件
[H-TYPES]: #sdk-公开头文件
[H-ERROR]: #sdk-公开头文件
[S-PACKAGE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Package.swift#L1-L29
[S-VERSION]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Version.swift#L3-L18
[S-STORE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L34-L54
[S-AUTH]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/AuthorizationStatusSource.swift#L37-L116
[S-CAL]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L285-L465
[S-CAL-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1177-L1227
[S-UPDATE-CAL]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1779-L1798
[S-LIST]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L493-L508
[S-LIST-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1231-L1312
[S-CREATE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L528-L670
[S-CREATE-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1314-L1383
[S-UPDATE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L772-L939
[S-UPDATE-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1385-L1456
[S-OCCURRENCE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L942-L964
[S-DELETE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L966-L1063
[S-DELETE-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1458-L1476
[S-GET]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L1065-L1071
[S-SEARCH]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L1084-L1128
[S-QUERY-SCHEMA]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L684-L776
[S-RECURRENCE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L1875-L1906
[S-PARSER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L3043-L3156
[S-TIME-PARSER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L2902-L3041
[S-FORMAT]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventFormattingSource.swift#L27-L145
[S-EVENT-OUTPUT]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L2834-L2885
[S-EVENT-SCHEMA]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L198-L435
[S-PARTICIPANT]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/ParticipantFormatting.swift#L6-L89
[S-COPY]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L1357-L1408
[S-COPY-HANDLER]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L2524-L2600
[S-EXCLUDE]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L553-L767
[S-EXCLUSION-EXEC]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/ExclusionExecutor.swift#L3-L63
[S-DISPATCH]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/Server.swift#L1050-L1173
[S-ERROR]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitErrorSanitizer.swift#L27-L50
[S-REFRESH]: https://github.com/winter-icecoke/apple-agenda-mcp/blob/3ad64088feb82104982b6b3e7181f6bb4ca8af94/Sources/CheICalMCP/EventKit/EventKitManager.swift#L261-L274
[覆盖口径]: https://linear.app/icecoke/issue/SPE-969
[时间合同]: https://linear.app/icecoke/issue/SPE-970
[ID合同]: https://linear.app/icecoke/issue/SPE-973
[状态合同]: https://linear.app/icecoke/issue/SPE-974
[受限功能]: https://linear.app/icecoke/issue/SPE-972
