# 建造与区域

- 单个建筑：`build {def,x,z,stuff}`。蓝图已存在时会返回 ok+already，不算失败。
- 成片结构：`blueprint {def,rect,mode:"fill"|"perimeter",stuff}`。
- 地形标记：`designate {designator,rect}`。砍树必须用 `Designator_FullyGrownTreesChop`，`Designator_PlantsCut` 只割草不出木材。
- 全图砍伐会把殖民者派到地图边缘送死，必须配 `area` 活动区。
- 种植区：`growzone {x1,z1,x2,z2,plant}`，优先选相邻最高肥力地块。
- 仓库：`zones` 看现有，`area` 划活动区。

硬约束：
- 家具用木材，围墙用石砖（花岗岩优先）。钢铁可燃且前期不可再生，留给武器、医药、机械。
- 屋顶靠墙体围合自动生成：墙 + 门 = 房间闭合。不要用 `roof` 硬塞，支撑超过 6 格会坍塌压死小人。
- 建墙时注意别让炉灶之类占掉墙位，会出现看不见的缺口。
- 蓝图迟迟不建，查三件事：材料够不够、`designations` 有没有 SkilledBuilder 标记、蓝图品质锁是否开着。
## 下蓝图之前先查材料（硬规则）

**永远不要先点蓝图再等。** 材料不够的蓝图不是"在建设中"，是没人能建的死图纸：工人会一直走过去看、一直失败，你还以为在进度中。

每次 `build` / `blueprint` 之前：

1. `codex {topic:"cost", def:"<建筑>"}` 或看 `build` 返回的 `materials missing` —— 拿到单品材料需求
2. `supplies` 查现有量（木材/钢铁/石砖/零件分别多少）
3. 算一下：**现有量 ÷ 单品需求 = 最多能建几个**
4. 只建得起几个就下几个蓝图的量，别一次铺满整片

现在 `build` 和 `blueprint` 都带预检：

- 材料不够一个 → 直接报错 `materials missing: 木材 12/50`，不落蓝图
- 只够一部分 → 只落得起的那几个，返回 `limited:true` 和 `limitReason`
- 真要硬放（明知在等材料）→ 显式传 `force:true`

**特殊情况**：

- `SleepingSpot` 这类 `WorkToBuild = 0` 的建筑不需要材料，直接生成，不受此限
- 沼泽/水域等软地形上墙和地基放不了，会报「需要支持的地形：重型」——这不是材料问题，是选址问题，换地方
- 钢铁可燃且前期不可再生：家具用木材，承重墙用石砖，钢铁留给武器、医药、机械