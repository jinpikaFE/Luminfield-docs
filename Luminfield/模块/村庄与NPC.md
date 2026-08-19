---
project: Luminfield
graph_namespace: luminfield
type: module
module: 村庄与NPC
source_branch: main
source_commit: bf5027625612606e065cc5407affaa867e40a234
last_verified: 2026-08-19
status: in-progress
tags:
  - luminfield/项目
  - luminfield/模块/村庄与NPC
---

# 村庄与 NPC

## 当前范围

阶段 B 已在现有 192×128 大世界内加入完整首批村庄阵容、六个室内、第一版星邮与六名角色的完整事件链：

- 村庄范围：`x=77..115`、`y=30..63`，通过晶辉大道与家园道路连通。
- 11 处外部地标：月辉档案馆、星织茶屋、月石工坊、星井广场、村门、路牌、双灯长椅、发光花车、暮光商铺、星光邮驿和星落巡守所。
- 8 名核心村民：莉欧拉 `liora`、塔维 `tavi`、奈米 `nemi`、塞菈 `sela`、艾洛温 `elowen`、维莎 `vessa`、奥林 `orin`、凯尔 `kael`。
- 普通日按时段目标移动；第 7 天灯憩日使用最高优先级独立路线，天气条件高于季节条件，季节条件高于基础路线。
- 同场景内每 10 游戏分钟移动一格；跨场景在时段边界交接到安全入口相邻格后继续寻路。
- 首次见面显示人物自我介绍，重复交谈使用当前时段对话。
- 月辉档案馆 08:00–20:00 开放；莉欧拉在普通日 09:00–13:00 进入馆内工作。
- 月石工坊 08:00–19:00 开放；塔维在普通日 09:00–13:00 进入室内工作，月纹工作台当前只提供检视说明。
- 星织茶屋 09:00–21:00 开放；维莎在普通日 09:00–13:00 进入室内工作，星织茶台当前只提供草本与茶香检视，不扩展经济契约。
- 暮光商铺仅在普通日 10:00–18:00 开放；旅行货单提供按绝对周次与季节确定性轮换的四项种子直购，灯憩日整日关闭；奥林在普通日 10:00–13:00 进入室内工作。
- 星光邮驿 07:00–19:00 开放；奈米在普通日 09:00–13:00 进入室内工作，路线分拣台只提供检视说明，不新增投递、收费、物品或经济契约。
- 星落巡守所 06:00–20:00 开放；凯尔在普通日 09:00–13:00 进入室内工作，封印巡路台只提供检视说明，不开放遗迹、任务、装备、收费、物品或经济契约。
- 八名村民支持每日首次交谈加成、每日一次赠礼与三档关系阶段。
- 奈米欢迎信和莉欧拉、塔维、奈米的“可信朋友”奖励在条件达成后的次日只投递一次。
- 莉欧拉在关系 25/60 点时拥有两阶段友情事件；第二阶段要求第一阶段在更早日期完成，每阶段最后一页关闭后才写入完成状态。
- 塔维在关系 25/60 点时拥有独立两阶段友情事件；资格与莉欧拉事件共同按事件定义中的人物、场景、阈值和前置 ID 数据驱动解析，两条链分别归一化。
- 奈米在普通日下午世界路线中按关系 25/60 点触发“无法投递的信”与“星图路线”两阶段友情事件；资格读取交互前状态，第二阶段要求前一阶段在更早日期完成。
- 凯尔在普通日下午世界路线中按关系 25/60 点触发“断裂的蓝色符纹”与“安全归来的路线”两阶段友情事件；试玩与资格判定均从 `CurrentNpcs(..., world, playerCell)` 读取真实投影。
- 塞菈在普通日下午世界路线中按关系 25/60 点触发“淬炼的星光”与“共同的锻造节奏”两阶段友情事件；试玩与资格判定从第 15/17 天 14:00 的 `CurrentNpcs(..., world, playerCell)` 读取广场真实投影。
- 奥林在普通日下午世界广场路线中按关系 25/60 点触发“未标价的货单”与“共享灯路”两阶段友情事件；事件额外要求当前日程对话键为 `village.npc.orin.plaza`，清晨、晚间和灯憩日的真实投影均不会误触发。

## 状态与数据

- `VillageCatalog` 保存稳定人物 ID、地标 ID、村庄边界、道路、碰撞与日程定义。
- `NpcScheduleEntry` 使用开始/结束分钟、稳定场景 ID、目标位置、朝向、本地化键、星期、天气、季节和显式优先级。
- `VillageSystem` 保存已认识人物与关系记录，并把同一个 `WeatherSystem.CurrentId` 交给 `NpcScheduleSystem`；后者按日期、分钟、条件与玩家当前格批量解析 8 人投影。
- `NpcNavigationMap` 保存世界和六个室内共享的可通行几何与安全入口；玩家碰撞和 NPC 寻路读取同一份静态规则。
- `PlayerLocationIds` 定义 `world`、`cottage`、`moonlit_archive`、`moonstone_workshop`、`starweaver_tea_house`、`twilight_emporium`、`starlight_post` 与 `starfall_watch`；`GameSession` 是切换场景与保存位置的唯一业务入口。
- `MailSystem` 保存稳定邮件 ID、送达日、已读与附件领取状态；投递只读取 `VillageSystem` 的相遇和关系条件。
- `CharacterEventSystem` 保存稳定事件 ID 与完成日期；莉欧拉、塔维、奈米、凯尔、塞菈与奥林六条事件链分别归一化，资格由交互前的相遇、关系、场景、工具、日程对话键和前置事件状态纯判定。
- `WorldVillageChunk` 只绘制当前世界区块内的地标与村民；`ArchiveView`、`WorkshopView`、`TeaHouseView`、`TwilightEmporiumView`、`StarlightPostView` 与 `StarfallWatchView` 只绘制对应室内日程中的人物。
- 场景节点不保存第二份人物日程、相遇状态或翻译文本。

## 交互契约

- `GameSession.PreviewSelectedTarget` 与 `InteractWithVillager` 共用 `VillageSystem.CheckInteraction`。
- 第 1 格“手”用于交谈；选中非工具物品时切换为赠礼预览，其他工具显示暖金色切换提示。
- 每名村民每天只接受一份礼物；失败或重复赠礼不会扣物。喜爱项使用稳定物品 ID，喜欢/不喜欢使用物品类别。
- 面向人物或与人物邻接都可以触发，但高亮与动作目标固定为人物真实格。
- 当前人物格参与移动碰撞；NPC 不进入玩家当前格，不与其他 NPC 同格或对穿，路径与安全锚点避开建筑碰撞和关键入口。
- 六个室内的外门、出口、星图研究台、月纹工作台、星织茶台、旅行货单、路线分拣台和封印巡路台都复用统一目标预览，并和实际动作共享手、开放时间与场景规则；大型设施完整轮廓按可执行、工具不匹配和阻塞状态切换颜色。
- 家园邮箱复用 `GameSession.PreviewSelectedTarget` 与 Hand/邻接规则；背包满时附件保持在邮件中，不发生部分领取。

## 视觉资产

- `assets/generated/village_landmarks_chroma.png` / `village_landmarks.png`：严格 4×2 外部地标。
- `assets/generated/village_npcs_chroma.png` / `village_npcs.png`：严格 4×3，三行人物、四列方向。
- `assets/generated/village_npcs_expansion_chroma.png` / `village_npcs_expansion.png`：严格 4×5，五名新增人物、四列方向。
- `assets/generated/moonlit_archive_interior.png`：1536×1024 原创档案馆室内，运行时裁切为 640×360。
- `assets/generated/moonstone_workshop_interior.png`：1536×1024 原创工坊室内，运行时稳定裁切为 640×360。
- `assets/generated/starweaver_tea_house_interior.png`：1536×1024 原创茶屋室内，运行时稳定裁切为 640×360；中央通道、底部出口和茶台热点按实机画面校准。
- `assets/generated/twilight_emporium_exterior_chroma.png` / `twilight_emporium_exterior.png`：暮光商铺原创色键源图与透明运行时外观，稳定裁切后按约 100 像素高最近邻绘制。
- `assets/generated/twilight_emporium_interior.png`：1536×1024 原创商铺室内，运行时稳定裁切为 640×360；底部出口、旅行货单真实范围、中央通道和奥林落点已按实机画面校准。
- `assets/generated/starlight_post_exterior_chroma.png` / `starlight_post_exterior.png`：星光邮驿原创色键源图与透明运行时外观，落在村庄西北支路。
- `assets/generated/starlight_post_interior.png`：1536×864 运行时裁切的原创邮驿室内；柜台、后墙邮件架、侧柜、出口和奈米落点均按实机画面校准并与共享碰撞对齐。
- `assets/generated/starfall_watch_exterior_chroma.png` / `starfall_watch_exterior.png`：星落巡守所原创色键源图与透明运行时外观，稳定裁切为 `(194,49,866,1119)` 后落在村庄西南支路。
- `assets/generated/starfall_watch_interior.png`：1536×864 运行时裁切的原创巡守所室内；封印巡路台、左右记录架、出口和凯尔落点均按实机画面校准并与共享碰撞对齐。
- `assets/generated/relationship_gifts_chroma.png` / `relationship_gifts.png`：严格 3×2，三档关系徽记与三类赠礼反馈。
- `assets/generated/starlight_mailbox_chroma.png` / `starlight_mailbox.png`：严格 2×2，邮箱关闭/未读、信封与关系回信图标。
- 色键资产使用品红原稿、本地去色键和稳定裁切；人物身份、碰撞、日程、邮件和对话仍由领域模型驱动。
- 村庄非道路空地复用已有原创发光灌木、蘑菇和花丛，仅作确定性非阻挡布景。

## 存档

`GameSaveV1.Player.LocationId`、`Village.MetNpcIds`、`Village.Relationships`、`Mail.Entries` 与 `CharacterEvents.Entries` 是 `schemaVersion: 1` 的加法字段：

- 旧存档缺失字段时恢复为空列表。
- 旧版 `InsideCottage` 会迁移为 `cottage`；未知场景回退到 `world`。
- 未知人物/邮件/事件 ID、重复 ID 在读取时过滤与去重；孤立或同日乱序事件被过滤，关系点数、互动日期、已读和领取状态安全归一化。
- 日期、体力、背包、农田、任务、探索和其他阶段 A 状态不受影响。

## 当前边界

- 当前已加入天气与季节条件日程；主线事件和关系阶段尚未参与日程条件。
- 当前采用每 10 游戏分钟一格的离散移动，没有补充人物逐帧步态动画；跨场景在稳定安全入口交接，不模拟门内外连续行走。
- 目前开放月辉档案馆、月石工坊、星织茶屋、暮光商铺、星光邮驿与星落巡守所 6 个室内，阶段 B 的建筑内部目标已完成。
- 第一版关系奖励、星邮、暮光商铺轮换库存，以及莉欧拉、塔维、奈米、凯尔、塞菈、奥林各自的两阶段友情事件已完成；更多角色事件和更完整的关系奖励仍是后续可选内容。

## 验证

- 当前主线完整测试 222/222 通过；中英本地化键集合为 737/737。村庄与 NPC 的既有回归仍包含在该快照中。
- macOS 已实际查看暮光商铺轮换库存与灯憩日关闭，以及奥林两阶段事件的普通日下午真实广场投影；截图 99–100、104–105 来自第 8 轮集成态。
- 1–56 日、三种显式天气、全天每 10 分钟解析 8 名跨场景互不重叠、对应区域可通行的村民位置；同场景相邻刻度最多移动一格且不发生对穿。
- 灯憩日路线与普通日不同。
- 预览/动作工具规则、每日交谈、赠礼限制、关系点数、六个室内入口/设施/出口、星邮投递/领取、六条角色事件链顺序、本地化键和旧存档迁移均有覆盖。
- macOS 已实际查看 8 名 NPC 村庄布局、月石工坊、星织茶屋、暮光商铺、塔维/维莎/奥林互动、莉欧拉与塔维两阶段事件，以及邮箱未读、星邮列表和奖励已领取状态。
- macOS 已实际查看移动中的多名 NPC、小雨塞菈工坊路线和雨幕季维莎路线；截图 75–77 来自第 4 轮集成态。
- macOS 已实际查看星光邮驿外门、分拣台、奈米工作位置、奈米两阶段事件、10 地标回归和错误工具暖金轮廓；截图 78–84 来自第 5 轮集成态。
- macOS 已实际查看星落巡守所外门、巡路台、凯尔工作位置、凯尔两阶段事件、11 地标分区回归和错误工具暖金轮廓；截图 85–91 来自第 6 轮集成态。
- macOS 已实际查看塞菈两阶段事件的真实广场投影、多页对话与关系状态；截图 96–97 来自第 7 轮集成态。

## 关联

- [[Luminfield/模块/架构总览|架构总览]]
- [[Luminfield/模块/状态所有权与依赖方向|状态所有权与依赖方向]]
- [[Luminfield/模块/世界、场景与建筑|世界、场景与建筑]]
- [[Luminfield/模块/交互预览与原子动作|交互预览与原子动作]]
- [[Luminfield/模块/游戏循环与视觉|游戏循环与视觉]]
- [[Luminfield/模块/玩法路线图|玩法路线图]]
- [[Luminfield/决策记录/2026-07-31-村民确定性时段日程|村民确定性时段日程]]
- [[Luminfield/决策记录/2026-08-04-NPC条件日程与可重建格级寻路|NPC 条件日程与可重建格级寻路]]
- [[Luminfield/决策记录/2026-07-31-角色事件按交互前资格与末页完成|角色事件按交互前资格与末页完成]]
- [[Luminfield/变更记录/2026-07-31|2026-07-31 变更记录]]
- [[Luminfield/变更记录/2026-08-03|2026-08-03 变更记录]]
- [[Luminfield/变更记录/2026-08-04|2026-08-04 变更记录]]
- [[Luminfield/变更记录/2026-08-12|2026-08-12 变更记录]]
- [[Luminfield/变更记录/2026-08-13|2026-08-13 变更记录]]
