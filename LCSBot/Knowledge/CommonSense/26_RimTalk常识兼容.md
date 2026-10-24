# RimTalk 常识格式兼容

我们的常识库按 **RimTalk - Expand Memory** 的条目模型索引，两边可以互相导入导出。

## 条目格式

```
[标签|重要性]内容
```

- **标签**：逗号分隔，如 `规则,建造`。标签里包含分类关键词就会自动归类。
- **重要性**：0~1，决定排序。写 `0.9` 或 `9` 都可以。
- 一行一条，以 `#` 或 `//` 开头的行会被忽略。

## 自动分类（与 RimTalk 一致）

| 优先级 | 分类 | 标签里包含 |
|---|---|---|
| 1 | 规则 | `规则` `instructions` `instruction` `rule` |
| 2 | 殖民者状态 | `殖民者状态` `pawnstatus` `colonist` `状态` |
| 3 | 历史 | `历史` `history` `past` `记录` |
| 4 | 世界观 | `世界观` `lore` `background` `背景` `设定` |
| 5 | 其他 | 以上都不含 |

## 我们怎么归类自己的文件

| 目录 | 分类标签 | 默认重要性 |
|---|---|---|
| `Knowledge/LongTerm/` | 规则 | 0.9 |
| `Knowledge/ToolChains/` | 规则 | 0.8 |
| `Knowledge/CommonSense/` | 世界观 | 0.7 |
| `Knowledge/RimTalk/` | 不加（你写的标签说了算） | 0.6 |

每个 `.md` 文件按 `## 小节` 拆成独立条目，标签是「分类,文件名,小节标题」。

## 检索

```
know {action:"find", q:"食物"}                 按标签匹配
know {action:"find", q:"外墙", content:true}   连内容一起搜
know {action:"find", q:"工作", mode:"all"}     All=所有标签都要命中
know {action:"read", id:"ck-xxxx"}             取正文
know {action:"read", tag:"生存优先级"}          按标签取最相关的一条
know {action:"stats"}                          分类统计
```

中文没有词边界，所以除了子串匹配还有**二元组模糊匹配**：查「生存优先级」能找到「生存与安全优先级」。标签没命中时会自动回落到内容检索。

## 双向搬运

**从 RimTalk 搬到我们这**：
1. 在 RimTalk 的常识库界面点「导出」，复制文本
2. `know {action:"import", text:"<粘贴的内容>"}`
3. 落到 `Knowledge/RimTalk/imported.txt`，立刻可检索

**从我们搬到 RimTalk**：
1. `know {action:"export", file:true}` → 生成 `Knowledge/RimTalk/export.txt`
2. 打开该文件，复制内容
3. 在 RimTalk 常识库点「导入」粘贴

也可以用「常识库」标签页直接编辑 `imported.txt`，格式一样。

## 重要性怎么定

| 值 | 用途 |
|---|---|
| 0.9–1.0 | 核心规则，必须遵守（身份、权限、优先级铁律） |
| 0.7–0.8 | 重要机制（建造、战斗、医疗判据） |
| 0.5–0.6 | 一般背景（数值表、参考） |
| 0.3–0.4 | 可选细节 |
| 0.1–0.2 | 低优先级提示 |

## 注意

RimTalk 与 Expand Memory 当前**没有启用**（不在 ModsConfig 里）。我们是按它的格式与检索语义做的纯格式兼容，不依赖它的程序集。等你启用它之后，导入导出这条路立刻可用；如果需要运行时直连它的 API（它提供了 `CommonKnowledgeAPI`），再单独做反射桥接。