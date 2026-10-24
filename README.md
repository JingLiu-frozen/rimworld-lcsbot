# RimWorld LCS Bot Bridge

一个给《边缘世界 / RimWorld》用的**本地回环自动化桥**：把殖民地状态以 JSON 暴露在 `127.0.0.1`，并从同一个 socket 接收指令，让外部 agent 能在不碰 UI 的情况下观察和操作这个殖民地。

仓库里同时收纳了与它配套使用的另外两个 mod：

| mod | packageId | 作用 |
|---|---|---|
| **LCS Bot Bridge** | `dsh.lcsbot` | 驾驶仪本体。状态导出 + 指令执行 + 面板 UI + 内置知识库 |
| **DSH Bot Bridge** | `dsh.botbridge` | 上一代实现（1.6 起）。文件式桥：状态写 JSON 文件，指令从文本文件按行读 |
| **LCS CleanFix** | `lcs.cleanfix` | 1.7 下的兼容垫片，补两个缺失的 def 引用，消掉加载期交叉引用红字 |

三个都是**纯客户端 / 纯补丁**性质：不改数值、不加新 content、不写存档结构。删掉之后存档本身没有任何变化。

---

## 安装

把需要的目录整个丢进 `RimWorld/Mods/`，在 mod 列表里勾上。

- 只想要驾驶仪 → 装 `LCSBot`
- 1.6 老环境 → 装 `DSHBot`
- 1.7 日志里有一堆 `Could not resolve cross-reference` → 再加 `LCSCleanFix`

**依赖**：Harmony（`brrainz.harmony`）。

支持版本：`1.7`（LCSBot / LCSCleanFix）、`1.6` + `1.7`（DSHBot）。

## 工作原理

```
┌──────────────┐   TCP 127.0.0.1:4771   ┌────────────────┐
│  外部 agent   │ ◀────────────────────▶ │  LCS Bot Bridge │──▶ Verse / RimWorld API
└──────────────┘   JSON 状态 / 文本指令  └────────────────┘
```

- 只监听回环地址，不对外网开放，没有远程代码执行面
- 单机模式专用；多人 / 联机不适用
- 驾驶仪只读游戏内存里的对象图，写操作走游戏自己的 `Designation` / `Job` / `Bill` 接口，不绕过游戏规则
- **权限闸门**：写操作默认受拦。被拦时命令返回 `READ-ONLY: ...` —— 这不是故障，是设计。在游戏里按 `F8` 或点面板齿轮授权

## 命令面

`state` `vitals` `health` `alerts` `messages` `pawn` `needs` `mooddetail` `supplies` `daysOfFood` `wealth` `power` `net` `farm` `ore` `researchinfo` `rooms` `zones` `area` `designations` `designators` `buildables` `materials` `worklist` `workorder` `codex` `know` `guide` `presets` `dialog` `note` `q`

写入类：`build` `blueprint` `designate` `growzone` `roof` `combatzone` `work` `workset` `draft` `goto` `attack` `rescue` `preset` `askplayer`

完整语义不在 README 里，在 `LCSBot/Knowledge/` 和 `LCSBot/Guides/`：

```
Knowledge/CommonSense/   26 篇机制常识（食物、温度、电力、医疗、防御、心情、手术……）
Knowledge/LongTerm/      身份定位、生存优先级
Knowledge/ToolChains/    reading / building / work / combat 四条工具链
Guides/                  分阶段作业单：开局 → 过渡基地 → 武装扩张 → 中期能源 → 终局
```

`Guides/` 里每一步都写成 `TASK / PREREQ / STEPS / DONE / FAIL` 的形式，是给 agent 直接执行的作业单，不是给人看的攻略。

## 目录结构

```
LCSBot/
├── About/About.xml
├── Assemblies/LCSBot.dll
├── Defs/MainButtonDefs.xml          ← 面板主按钮（LCSBotFish）
├── Guides/*.md
└── Knowledge/{CommonSense,LongTerm,ToolChains}/*.md

DSHBot/
├── About/About.xml
└── Assemblies/DSHBot.dll

LCSCleanFix/
├── About/About.xml
└── Patches/CleanFix.xml             ← 补 DamageDef PurificationBurn、ThingDef Kiiro_TailorSet
```

运行时产物（`lcsbot.log`、`plan.txt`、`port.txt`、`autoload.txt`）不入库。

## 构建

仓库里是**二进制发布**，`Assemblies/*.dll` 是编译产物，源码不在这里。要改行为，改 `Knowledge/` 和 `Guides/`（纯文本，改完重启即可，不用重编译）。

## 已知边界

- `READ-ONLY:` 返回值是权限闸门，不是 bug
- 一条命令超过 **39 秒**没返回，说明游戏主线程被挡住了，重试没有意义 —— 先 `dialog {action:"list"}` 看有没有 `Dialog_*` 硬暂停窗口
- 游戏失焦时 tick 会停（`runInBackground=False` 时表现像卡死，且不产生弹窗），用 `focus {on:true}`
- 多阶段对话 / 仪式 / 飞船起飞这类必须人来点的环节，会走 `askplayer`，驾驶仪不会替你点

## 许可

MIT，见 [LICENSE](LICENSE)。
