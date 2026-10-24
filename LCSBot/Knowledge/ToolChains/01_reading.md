# 先读哪个 op

数据不够就下命令，是这类自动化最常见的死法。按问题选 op：

| 我想知道 | 用 |
|---|---|
| 现在最急的是什么 | `alerts` |
| 刚刚发生了什么 | `messages` |
| 殖民地总览（人/物资/研究/区域） | `state` detail=brief |
| 补给够不够吃 | `supplies` |
| 财富构成 | `wealth` |
| 附近有什么矿、多远 | `ore` |
| 田里情况 | `farm` |
| 电力盈亏 | `power` |
| 某个殖民者细节 | `pawn` / `vitals` / `needs` / `mooddetail` |
| 岗位优先级 | `schedules` 读，`work` 写 |
| 有哪些工作类型可用 | `worklist`（含 Mod 新增） |
| 有哪些设计器可用 | `designators` |
| 蓝图为什么不动 | `bpinfo` / `designations` |
| 研究能选什么 | `researchinfo` |
| 现在能建什么 | `buildables`（只列已研究可放置） |
| 有哪些房间预设 | `presets` / `presetread <名字>` |
| 工作栏的槽位顺序 | `workorder`（灭火/就医恒在最前） |
| 战斗区范围 | `combatzone`（征召后不得离区） |
| 之前记下的待办 | `note` action=list |

## 常识怎么查
- `know {action:"find", q:"关键词"}` —— 按 RimTalk 标签模型检索，返回 id/标签/重要性
- `know {action:"read", id:"ck-xxxx"}` —— 取那条的正文
- 比 `REF` 更省 token：`REF` 读整篇，`know` 只取命中那一节

## 常识在哪儿
`Knowledge/CommonSense/` 里有分主题的机制说明：食物与营养、温度与房间、需求与心情、电力、种植与储存、医疗与疾病、防御。
需要细节时用 `REF Knowledge/CommonSense/04_温度与房间.md` 之类按需读，不要凭记忆猜机制。

规则：
- ID、坐标、DefName 必须来自实际返回值，不许猜。
- 一次拿够信息，不要每轮都全量重读；`state` detail=brief 已经包含人、物资、研究、信件。
- 不确定某个 op 的参数就先调一次不带参数，看返回结构。