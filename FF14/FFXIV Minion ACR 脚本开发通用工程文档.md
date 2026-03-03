**版本**：v2.1 (通用版 - 双阶段工作流)
**适用框架**：Minion / TensorCore / RikuACR
**支持职能**：白魔法师 (WHM), 贤者 (SGE)

---

## 1. 标准工作流 (Standard Workflow)

本项目严格遵循以下**双阶段**开发流程，以确保人工干预的准确性：

### 阶段一：时间轴结构化 (Timeline Structuring)
1.  **输入**：用户提供原始战斗日志或时间轴文本（通常包含时间、机制英文名）。
2.  **AI 任务**：清洗数据，生成一个待填写的**Markdown 表格**。
3.  **表格格式**：
    * `Timeline Index`：对应 Lua 脚本中的索引。
    * `Time`：机制触发的秒数。
    * `Raw Skill Name`：原始时间轴中的技能名（英文/日文）。
    * `CN Name`：**（通过用户提供的翻译进行精准翻译）** 机制的中文译名。
    * `Skill Instructions`：**（留空供用户填写）** 该时间点需要释放的技能指令（如“7秒法令”、“场中庇护所”、“对自己庇护所”）。

### 阶段二：代码生成 (Code Generation)
1.  **输入**：用户填写完成的表格。
2.  **AI 任务**：读取表格中用户填写的指令，根据**“技能数据库”**和**“逻辑规范”**，将自然语言转换为标准 Lua 代码结构。
3.  **输出**：完整的 `.lua` 脚本文件。

---

## 2. 阶段一输出模板：待填写表格 (Stage 1 Output)

当用户提供原始时间轴时，AI 应输出如下格式的 5 列表格供用户编辑：

| Timeline Index | Time | Raw Skill Name | CN Name (用户填写) | Skill Instructions (用户填写) |
| :--- | :--- | :--- | :--- | :--- |
| 1 | 5.4 | --sync-- | (无需填写) | (无需填写) |
| 2 | 14.6 | Hot Impact | **炽焰冲击** | 7秒法令 1秒前全大赦 |
| 3 | 30.3 | Flame Floater | **浪顶炽火** | 场中庇护所 |
| 4 | 45.0 | Unknown Mech | **未知机制** | 对自己庇护所 |
| ... | ... | ... | ... | ... |

---

## 3. 技能数据库 (Skill Database)

生成代码时，请严格查阅以下 ID 和类型。

### 3.1 白魔法师 (White Mage - WHM)

| 技能名称 (CN) | English | ID | 类型 | 参数要求 |
| :--- | :--- | :--- | :--- | :--- |
| **狂喜之心** | Afflatus Rapture | `16534` | GCD | - |
| **苦难之心** | Afflatus Misery | `16535` | GCD | `targetType="Current Target"` |
| **医济** | Medica III | `37010` | GCD | - |
| **法令** | Assize | `3571` | oGCD | `ignoreWeaveRules=true` |
| **全大赦** | Plenary Indulgence | `7433` | oGCD | `ignoreWeaveRules=true` |
| **庇护所** | Asylum | `3569` | oGCD | **需特殊逻辑** (见下文4.3) |
| **水流幕** | Aquaveil | `25861` | oGCD | `ignoreWeaveRules=true` |
| **神祝祷** | Divine Benison | `7432` | oGCD | `ignoreWeaveRules=true` |
| **神名** | Tetragrammaton | `3570` | oGCD | `ignoreWeaveRules=true` |
| **节制** | Temperance | `16536` | oGCD | `ignoreWeaveRules=true` |
| **神爱抚** | Divine Caress | `37011` | oGCD | 需在节制后使用 |
| **铃铛** | Liturgy of the Bell| `25862` | oGCD | `ignoreWeaveRules=true` |
| **天赐** | Benediction | `140` | oGCD | - |

### 3.2 贤者 (Sage - SGE)

| 技能名称 (CN) | English | ID | 类型 | 参数要求 |
| :--- | :--- | :--- | :--- | :--- |
| **预后** | Prognosis | `24286` | GCD | - |
| **均衡预后** | Eukrasian Prognosis | `24292` | GCD | (群盾) 7.0 可能为II |
| **发炎** | Phlegma III | `24313` | GCD | `targetType="Current Target"` |
| **魂灵风息** | Pneuma | `24318` | GCD | `targetType="Current Target"` |
| **心神风息** | Psyche | `37033` | oGCD | `ignoreWeaveRules=true` |
| **坚角清汁** | Kerachole | `24298` | oGCD | `ignoreWeaveRules=true` |
| **寄生清汁** | Ixochole | `24299` | oGCD | `ignoreWeaveRules=true` |
| **白牛清汁** | Taurochole | `24295` | oGCD | 单体减伤 |
| **灵依清汁** | Druochole | `24296` | oGCD | 单体治疗 |
| **泛输血** | Panhaima | `24311` | oGCD | `ignoreWeaveRules=true` |
| **输血** | Haima | `24305` | oGCD | 单体盾 |
| **整体论** | Holos | `24310` | oGCD | `ignoreWeaveRules=true` |
| **智慧之爱** | Philosophia | `37035` | oGCD | 7.0新技能 |
| **混合** | Zoe | `24306` | oGCD | 下个GCD治疗加倍 |
| **自生II** | Physis II | `24302` | oGCD | 群体HOT |
| **活化** | Krasis | `24317` | oGCD | 单体受疗加成 |
| **拯救** | Soteria | `24294` | oGCD | 强化心关治疗 |
| **根素** | Rhizomata | `24301` | oGCD | 回豆子 |

---

## 4. 代码生成逻辑规范 (Implementation Specs)

在**阶段二**中，AI 将根据用户表格中的指令生成 Lua 代码，遵循以下逻辑：

### 4.1 基础参数规则
* **oGCD 能力技**：必须设置 `atomicPriority = true` 和 `ignoreWeaveRules = true`，确保在机制处理期间强制插入。
* **GCD 魔法**：保持默认设置（`ignoreWeaveRules` 缺省为 false）。
* **UUID**：为每个 Action 和 Condition 生成唯一的 UUID 字符串。

### 4.2 时间偏移处理 (Timer Offset)
解析用户指令中的时间描述：
* “**X秒前** [技能]” -> `timerOffset = -X`
* “**X秒后** [技能]” -> `timerOffset = X`
* “**[技能]**” (无时间描述) -> `timerOffset = 0`

### 4.3 技能特殊逻辑

#### 庇护所 (Asylum) - WHM
根据用户指令的不同描述，设置不同的参数：
* **情况 A：放置在场中**（指令包含：“场中”、“场地中心”或仅写“庇护所”时默认）：
    * `castPosX = 100`
    * `castPosZ = 100`
    * `isAreaTarget = true`
    * `showPositionPreview = true`
* **情况 B：以目标为中心**（指令包含：“对自己”、“对MT”、“对目标”）：
    * **不要**设置 castPos 坐标。
    * 设置 `targetType`（如 "Self", "Main Tank", "Current Target"）。
    * 设置 `isAreaTarget = false`（或移除该字段，视ACR具体实现而定，通常跟随目标不需要坐标）。

#### 苦难之心 (WHM) / 发炎 (SGE) / 魂灵风息 (SGE)
* 始终添加 `targetType = "Current Target"`。

### 4.4 目标与条件逻辑
根据指令关键词自动添加参数：

* **指定目标**：
    * 指令：“[技能] **MT**” -> 添加 `targetType = "Main Tank"`
    * 指令：“[技能] **ST**” -> 添加 `targetType = "Off Tank"`
    * 指令：“[技能] **自己**” -> 添加 `targetType = "Self"`

* **血量检测 (Condition)**：
    * 指令：“**检测团血** [技能]” ->
        * 添加 `conditions` 块。
        * `category="Party"`, `comparator=2 (<)`, `hpValue=[默认90或用户指定]`, `partyTargetSubType="Lowest HP"`.
    * 指令：“**检测MT血量** [技能]” ->
        * 添加 `targetType = "Detection Target"`.
        * 添加 `Filter` (Filter Target: Main Tank).
        * 添加 `Condition` (HP% < X).

---

## 5. 脚本结构模板 (Lua Template)

```lua
local tbl = {
    -- [Timeline Index]
    [2] = {
        -- [Entry 1]
        {
            data = {
                actions = {
                    {
                        data = {
                            -- 技能参数区
                            actionID = 24298, -- 示例ID
                            atomicPriority = true,
                            ignoreWeaveRules = true,
                            gVar = "ACR_Riku[JOB]_CD", -- 根据职业自动替换 JOB 为 WHM 或 SGE
                            uuid = "Auto_Gen_UUID_1",
                            version = 2.1,
                        },
                    },
                },
                -- 条件参数区 (仅当指令要求检测时生成)
                conditions = {}, 
                mechanicTime = 14.6, -- 对应表格Time
                name = "机制中文名 (技能备注)",
                timelineIndex = 2,
                timerOffset = -1, -- 对应表格偏移
                uuid = "Auto_Gen_UUID_2",
                version = 2,
            },
        },
    },
    -- ... 更多机制
    inheritedProfiles = {
        "store\\anyone\\savage6\\m9s\\modules\\core",
        "store\\anyone\\savage6\\m9s\\modules\\optimization",
        "store\\anyone\\savage6\\m9s\\modules\\mitigation",
    },
    mapID = 1321, -- 需用户确认
    version = 2,
}
return tbl