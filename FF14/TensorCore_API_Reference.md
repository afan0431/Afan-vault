# TensorCore API Reference

> **完整的 API 技术文档** - 函数、变量、事件、返回值详细说明  
> 最后更新：2026-01-07

## ⚠️ 重要说明

### ACR API 通用别名

在所有 API 调用中，`TensorCore.API.TensorACR` 会自动解析为当前激活的 Tensor/Riku ACR。

**示例：**

    -- 以下两种写法等价（当使用 RikuAST 时）
    TensorCore.API.TensorACR.setLockFaceHeading(radian)
    TensorCore.API.RikuAST3.setLockFaceHeading(radian)
  
    -- 以下两种写法等价（当使用 TensorWeeb 时）
    TensorCore.API.TensorACR.holdActionUntil(16481, Now() + 7000)
    TensorCore.API.TensorWeeb2.holdActionUntil(16481, Now() + 7000)

**优势：**

- 代码更通用，切换职业无需修改
- 适合编写跨职业的通用 Reaction
- 推荐在不确定职业时使用

---

## 📚 目录

### [核心模块(#核心模块)

- [TensorCore 核心 API](#tensorcore-核心-api)
- [TensorReactions](#tensorreactions-api)
- [Argus 图形引擎](#argus-api)

### [职业模块(#职业特定-api)

- [TensorRuin - 召唤师](#tensorruin-召唤师)
- [TensorMagnum - 机工士](#tensormagnum-机工士)
- [TensorRequiem - 吟游诗人](#tensorrequiem-吟游诗人)
- [TensorReaper - 钐镰客](#tensorreaper-钐镰客)
- [TensorWeeb - 武士](#tensorweeb-武士)
- [RikuAST - 占星术士](#rikuast-占星术士)
- [RikuRDM - 赤魔法师](#rikurdm-赤魔法师)
- [RikuTanks - 坦克职业](#rikutanks-坦克)

### [通用系统(#通用-acr-api)

- [Hotbar/Healbar 控制](#hotbarhealbar-目标控制)
- [优先级管理](#优先级管理)
- [动作延迟系统](#动作延迟系统)
- [Lock Face](#lock-face-系统)

---

## 核心模块

### TensorCore 核心 API

#### 聊天消息着色

##### `TensorCore.sendParsedChatMessage(msg)`

**用途**：发送带颜色格式化的聊天消息

**格式标记**：

    {color:r,g,b}       -- 设置颜色
    {resetcolor}        -- 重置颜色
    {ability:id}        -- 自动翻译技能名

**示例**：

    -- 红色文本
    TensorCore.sendParsedChatMessage("/e {color:255,0,0}警告信息 {resetcolor}正常文本")

    -- 多色混合
    TensorCore.sendParsedChatMessage("/e {color:255,0,0}asdf{resetcolor} asdf {color:0,255,0}asdf{resetcolor}")

---

#### 避险系统

##### `TensorCore.Avoidance.inUnavoidableAOE(aoeList, entity, largeAOESize, extraConditions)`

**参数**：

- `aoeList` (table) - AOE 列表
- `entity` (entity) - 目标实体（注：已从 pos 改为 entity）
- `largeAOESize` (number) - 大型 AOE 尺寸阈值
- `extraConditions` (function, optional) - 额外条件函数

**返回值**：(bool) - 是否在不可避免的 AOE 中

---

##### `TensorCore.Avoidance.canSafeDash(target, checkReturn)`

**参数**：

- `target` (entity) - 目标实体
- `checkReturn` (bool, optional) - 是否检查返回位置

**返回值**：(bool) - 是否可以安全冲刺

**用途**：使用 Argus 检测冲刺到目标位置是否会进入 AOE

**示例**：

    local canDash = TensorCore.Avoidance.canSafeDash(Player:GetTarget())
    if canDash then
        -- 执行冲刺
    end

    -- 同时检查返回位置
    local canDashSafe = TensorCore.Avoidance.canSafeDash(Player:GetTarget(), true)

---

#### 治疗预测系统

##### `TensorCore.calcDirectHeal(potency, simCrit, forceCrit, isPet)`

**参数**：

- `potency` (number) - 治疗威力
- `simCrit` (bool) - 模拟暴击
- `forceCrit` (bool) - 强制暴击
- `isPet` (bool) - 是否为宠物技能

**返回值**：(number) - 实际治疗量（HP 值）

---

##### `TensorCore.calcHoT(potencyPerTick, numTicks, simCrit, forceCrit, isPet)`

**参数**：

- `potencyPerTick` (number) - 每跳威力
- `numTicks` (int) - 跳数（默认 1）
- 其他参数同上

**返回值**：(number) - HoT 总治疗量

---

##### `TensorCore.getPredictedDirectHealHP(ent, potency, simCrit, forceCrit, isPet, numTicks)`

**参数**：

- `ent` (entity) - 目标实体
- `potency` (number) - 威力
- `numTicks` (number, optional) - 计算未来多少跳的 HoT

**返回值**：(number) - 治疗后的 HP 百分比

**注意**：

- 会自动计算正在施放的治疗和现有 HoT
- 超过 100% 表示过量治疗
- 获取 numTicks：`math.floor(buff.duration / 3)`

---

#### 警报系统

##### `TensorCore.addAlertText(duration, text, scale, priority, tts)`

**参数**：

- `duration` (int) - 持续时间（毫秒）
- `text` (string) - 警报文本
- `scale` (float) - 缩放比例（默认 1.0）
- `priority` (int) - 优先级（1=绿色, 2=黄色, 3=红色）
- `tts` (bool) - 是否语音播报（默认 true）

**重载版本**：

    TensorCore.addAlertText(duration, text, scale, {r=255, g=255, b=255}, tts)

**使用前提**：需要在 TensorCore 菜单中解锁并设置警报窗口位置

---

#### 工具函数

##### `TensorCore.getEntitySpeed(entid)`

**参数**：

- `entid` (number) - 实体 ID

**返回值**：(number) - 实体移动速度

---

##### `TensorCore.getEntityByGroup(groupName)`

**参数**：

- `groupName` (string) - 组名（如 "Main Tank"）

**返回值**：(entity) - 实体对象或 nil

---

#### 绘图工具

##### `TensorCore.getCachedDrawer(colorStart, colorMid, colorEnd, colorOutline, outlineThickness)`

**返回值**：(Argus.ShapeDrawer) - 缓存的渐变色绘图器

---

##### `TensorCore.getMoogleDrawer()`

**返回值**：(Argus.ShapeDrawer) - 预设的 Moogle 配色绘图器

---

##### `TensorCore.getStaticDrawer(color, outlineThickness)`

**返回值**：(Argus.ShapeDrawer) - 单色绘图器

**示例**：

    local drawer = TensorCore.getMoogleDrawer()
    drawer:addTimedRectOnEnt(10000, Player, 0, 4, Player.targetid)

---

### TensorReactions API

#### 触发器管理

##### `TensorCore.API.TensorReactions.getCurrentGeneralTriggers()`

**返回值**：(table) - 当前通用触发器列表（可直接修改）

**用途**：获取并修改触发器列表

---

#### 配置文件管理

##### `TensorCore.API.TensorReactions.getTimelineReactionProfileName()`

**返回值**：(string | nil) - 当前加载的时间轴配置文件名

---

##### `TensorCore.API.TensorReactions.getGeneralReactionProfileName()`

**返回值**：(string | nil) - 当前通用配置文件名

---

##### `TensorCore.API.TensorReactions.reloadGeneralReactions()`

**用途**：重新加载通用反应配置

---

##### `TensorCore.API.TensorReactions.reloadTimelineReactions()`

**用途**：重新加载时间轴反应配置

---

#### 继承系统（2.0+）

##### `TensorCore.API.TensorReactions.getTimelineReactionProfileInheritedProfiles()`

**返回值**：(table) - 当前时间轴配置的继承配置列表

---

##### `TensorCore.API.TensorReactions.getGeneralReactionProfileInheritedProfiles()`

**返回值**：(table) - 通用配置的继承列表

---

### Argus API

> **📖 参考**：完整 API 文档见 [Argus Wiki](https://wiki.mmominion.com/doku.php?id=argusdocs)

#### DirectionalAOE（方向性 AOE）

**字段说明：**

- `x, y, z` (number) - AOE 位置坐标
- `aoeType` (int) - 动画/预兆类型
- `heading` (number) - AOE 朝向（弧度）
- `aoeLength` (int) - AOE 长度
- `aoeWidth` (int) - AOE 宽度（线形 AOE，圆形和扇形为 0）
- `aoeName` (string) - AOE 名称
- `aoeID` (number) - 技能/法术 ID
- `aoeCastType` (number) - 施法类型/形状（见 castType 枚举）
- `targetAttach` (int) - 附着的实体 ID，如果没有则为 nil
- `aoeAnimationInfo` (table) - 动画信息
- `aoeEffectInfo` (table) - 预兆信息
- `isAreaTarget` (bool) - 是否为自由选择目标的技能
- `entityID` (number) - 施法者 ID
- `friendly` (bool) - 是否为友方技能
- `contentID` (number) - 内容 ID

#### GroundAOE（地面 AOE）

**字段说明：**

- `x, y, z` (number) - AOE 位置坐标
- `aoeType` (int) - 动画/预兆类型
- `aoeLength` (int) - 长度/半径（码）
- `aoeWidth` (int) - 宽度（码，圆形和扇形为 0）
- `aoeName` (string) - AOE 名称
- `aoeID` (int) - 技能/法术 ID
- `aoeCastType` (int) - 施法类型/形状
- `targetAttach` (int) - 附着的实体 ID
- `aoeAnimationInfo` (table) - 动画信息
- `aoeEffectInfo` (table) - 预兆信息
- `isAreaTarget` (bool) - 是否为自由目标技能

---

#### AOE 检测函数

### `Argus.getCurrentGroundAOEs(inOrder)`

**参数：**

- `inOrder` (bool, optional) - 默认 false
  - `false`: 返回的表以 entityID 为键
  - `true`: 返回有序列表，按出现顺序

**返回值：**

- (table) - GroundAOE 表

**示例：**

    -- 获取所有地面 AOE（按实体 ID 索引）
    local groundAOEs = Argus.getCurrentGroundAOEs()
    for entityID, aoe in pairs(groundAOEs) do
        d("Entity " .. entityID .. " has ground AOE: " .. aoe.aoeName)
    end
    
    -- 获取有序列表
    local orderedAOEs = Argus.getCurrentGroundAOEs(true)
    for i, aoe in ipairs(orderedAOEs) do
        d("AOE #" .. i .. ": " .. aoe.aoeName)
    end

---

### `Argus.getCurrentDirectionalAOEs(inOrder)`

**参数：**

- `inOrder` (bool, optional) - 默认 false

**返回值：**

- (table) - DirectionalAOE 表

---

#### Tether（连线）检测

### `Argus.getCurrentTethers()`

**返回值：**

- (table) - 所有连线信息，格式：`[entityID] = {tether1, tether2, ...}`

**Tether 结构：**

- `type` (int) - 连线类型
- `targetid` (int) - 目标实体 ID
- `partnerid` (int) - 连线伙伴 ID

**示例：**

    -- 打印所有连接到玩家的连线
    local tethers = Argus.getCurrentTethers()
    for id, ts in pairs(tethers) do
        for t = 1, #ts do
            local tether = ts[t]
            if tether.targetid == Player.id then
                d("Entity " .. id .. " is tethered to player with type " .. tether.type)
            end
        end
    end

---

### `Argus.getTethersOnEnt(entityID)`

**参数：**

- `entityID` (int) - 实体 ID

**返回值：**

- (table) - 该实体的连线列表（数组形式，不是键值对）

**示例：**

    -- 在所有连接到玩家的实体上画圈
    local pTethers = Argus.getTethersOnEnt(Player.id)
    for i = 1, #pTethers do
        local tether = pTethers[i]
        local partner = TensorCore.mGetEntity(tether.partnerid)
        if partner ~= nil then
            Argus.addCircleFilled(
                partner.pos.x, partner.pos.y, partner.pos.z,
                3, 30,
                GUI:ColorConvertFloat4ToU32(1, 1, 0, 0.1),
                GUI:ColorConvertFloat4ToU32(1, 1, 0, 1),
                1.5
            )
        end
    end

---

#### Marker（标记）检测

### `Argus.getWaymarkPosition(markerID)`

**参数：**

- `markerID` (int) - 标记 ID（ActionList type 15 中的技能 ID）

**返回值：**

1. `x` (number) - X 坐标
2. `y` (number) - Y 坐标
3. `z` (number) - Z 坐标
4. `isActive` (bool) - 是否激活
5. `timeLastModify` (int) - 最后修改时间

---

#### 实体 Aura（光环）检测

### `Argus.getEntityAuras(ent)`

**参数：**

- `ent` (int | table) - 实体对象或实体 ID

**返回值：**

1. `persistentAura` (int) - 持久光环
2. `activeAura1` (int) - 激活光环 1
3. `activeAura2` (int) - 激活光环 2

---

#### ShapeDrawer 类

### 构造函数

#### `Argus2.ShapeDrawer:new(colorStart, colorMid, colorEnd, colorOutline, outlineThickness)`

**参数：**

- `colorStart` (u32color, optional) - 起始颜色（仅用于 timed 绘图）
- `colorMid` (u32color, optional) - 中间颜色（用于渐变）
- `colorEnd` (u32color) - 结束颜色（必需）
- `colorOutline` (u32color) - 轮廓颜色
- `outlineThickness` (number, optional) - 轮廓厚度，默认 1.5

**返回值：**

- (ShapeDrawer) - ShapeDrawer 实例

**ShapeDrawer 属性：**

- `colorStart` (u32color)
- `colorMid` (u32color)
- `colorEnd` (u32color)
- `colorOutline` (u32color)
- `outlineThickness` (number)
- `gradientIntensity` (int, optional)
- `gradientMinOpacity` (number, optional)
- `segments` (number) - 圆形绘图的段数，默认 50.0

---

### 绘图方法参数说明

#### `ShapeDrawer:addArrow(x, y, z, heading, baseLength, baseWidth, tipLength, tipWidth, oldDraw)`

**参数：**

- `x, y, z` (number) - 位置坐标
- `heading` (number) - 朝向（弧度）
- `baseLength` (number) - 基座长度
- `baseWidth` (number) - 基座宽度
- `tipLength` (number, optional) - 尖端长度，默认等于 tipWidth
- `tipWidth` (number, optional) - 尖端宽度，默认为 baseWidth 的 2倍
- `oldDraw` (bool, optional) - 是否使用旧绘图方式

---

#### `ShapeDrawer:addChevron(x, y, z, length, thickness, heading, oldDraw)`

**参数：**

- `x, y, z` (number) - 位置坐标
- `length` (number) - 长度
- `thickness` (number) - 宽度/厚度（箭头基座矩形的宽度）
- `heading` (number) - 朝向（弧度）
- `oldDraw` (bool, optional)

---

#### `ShapeDrawer:addCircle(x, y, z, radius, oldDraw)`

**参数：**

- `x, y, z` (number) - 圆心坐标
- `radius` (number) - 半径
- `oldDraw` (bool, optional)

---

#### `ShapeDrawer:addCone(x, y, z, radius, angle, heading, oldDraw)`

**参数：**

- `x, y, z` (number) - 圆锥顶点坐标
- `radius` (number) - 半径
- `angle` (number) - 角度（弧度）
- `heading` (number) - 朝向（弧度）
- `oldDraw` (bool, optional)

---

#### `ShapeDrawer:addCross(x, y, z, length, width, heading, oldDraw)`

**参数：**

- `x, y, z` (number) - 十字中心坐标
- `length` (number) - 长度
- `width` (number) - 宽度
- `heading` (number) - 朝向（弧度）
- `oldDraw` (bool, optional)

---

#### `ShapeDrawer:addDonut(x, y, z, radiusInner, radiusOuter, oldDraw)`

**参数：**

- `x, y, z` (number) - 圆环中心坐标
- `radiusInner` (number) - 内半径
- `radiusOuter` (number) - 外半径
- `oldDraw` (bool, optional)

---

#### `ShapeDrawer:addLine(x1, y1, z1, x2, y2, z2, thickness, endpointThickness)`

**参数：**

- `x1, y1, z1` (number) - 起点坐标
- `x2, y2, z2` (number) - 终点坐标
- `thickness` (number, optional) - 线条厚度
- `endpointThickness` (number, optional) - 端点厚度

---

#### `ShapeDrawer:addRect(x, y, z, length, width, heading, oldDraw)`

**参数：**

- `x, y, z` (number) - 矩形起点坐标（角落）
- `length` (number) - 长度
- `width` (number) - 宽度
- `heading` (number) - 朝向（弧度）
- `oldDraw` (bool, optional)

---

### Timed 绘图方法

所有 `addTimed*` 方法都有额外参数：

- `timeout` (number) - 持续时间（毫秒）
- `delay` (number, optional) - 延迟时间（毫秒）

**OnEnt 版本额外参数：**

- `entID` (int) - 实体 ID
- `targetID` (int, optional) - 目标实体 ID（用于朝向）
- `keepLength` (bool, optional) - 是否保持固定长度（不随距离变化）

**示例：**

    -- 绘制一个持续 5 秒的圆，延迟 1 秒后显示
    local drawer = TensorCore.getMoogleDrawer()
    drawer:addTimedCircle(5000, Player.pos.x, Player.pos.y, Player.pos.z, 10, 1000)
    
    -- 在实体上绘制矩形，朝向目标
    drawer:addTimedRectOnEnt(10000, Player.id, 10, 4, Player.targetid)

---

### `ShapeDrawer:setGradient(gradientIntensity, gradientMinOpacity)`

**用途**：设置渐变效果

**参数：**

- `gradientIntensity` (int) - 渐变强度
- `gradientMinOpacity` (number) - 最小不透明度

---

#### 事件注册函数

### `Argus.registerOnAOECreateFunc(callback)`

**回调签名：**

    function onAOECreate(aoeData, isDirectional)
        -- aoeData: DirectionalAOE 或 GroundAOE 结构
        -- isDirectional: true 为方向性 AOE，false 为地面 AOE
    end

**用途**：AOE 创建时触发

---

### `Argus.registerOnEntityCastFunc(callback)`

**回调签名：**

    function onEntityCast(entityID, castInfo)
        -- entityID: 施法者 ID
        -- castInfo: 施法信息
    end

**用途**：实体开始施法时触发

---

### `Argus.registerOnEntityChannelFunc(callback)`

**回调签名：**

    function onEntityChannel(entityID, channelInfo)
        -- entityID: 引导者 ID
        -- channelInfo: 引导信息
    end

**用途**：实体开始引导法术时触发

---

#### 颜色格式

### u32color

使用 ImGui 的颜色转换函数：

    -- RGBA (0-1 范围)
    local color = GUI:ColorConvertFloat4ToU32(r, g, b, a)
    
    -- 示例：半透明红色
    local red = GUI:ColorConvertFloat4ToU32(1, 0, 0, 0.5)

---

#### 完整示例

### 检测并绘制所有方向性 AOE

    local drawer = TensorCore.getMoogleDrawer()
    local aoes = Argus.getCurrentDirectionalAOEs(true)
    
    for i, aoe in ipairs(aoes) do
        if aoe.aoeCastType == 3 then  -- 假设 3 是圆形
            drawer:addTimedCircle(
                5000,  -- 5秒
                aoe.x, aoe.y, aoe.z,
                aoe.aoeLength,  -- 半径
                0  -- 无延迟
            )
        elseif aoe.aoeCastType == 4 then  -- 假设 4 是扇形
            drawer:addTimedCone(
                5000,
                aoe.x, aoe.y, aoe.z,
                aoe.aoeLength,
                math.rad(90),  -- 90度角
                aoe.heading,
                0
            )
        end
    end

---

### 检测连线并警告

    local myTethers = Argus.getTethersOnEnt(Player.id)
    
    if #myTethers > 0 then
        for i = 1, #myTethers do
            local tether = myTethers[i]
            TensorCore.addAlertText(
                3000,
                "Tethered to entity: " .. tether.partnerid,
                1.0,
                3,  -- 红色警告
                true  -- TTS
            )
        end
    end

---

## 职业特定 API

> **注意**：职业特定 API 必须使用完整的模块名（如 `TensorCore.API.TensorRuin`），不能用 `TensorACR` 替代。

### TensorRuin (召唤师)

#### DoT 黑名单管理

##### `TensorCore.API.TensorRuin.addBlacklistEntry(contentid)`

**参数**：

- `contentid` (number) - 内容 ID

**返回值**：(table) - 黑名单条目对象

    {
        enabled = true,
        allowIfTarget = false,
        allowSmartDoT = false,
        allowBane = false,      -- 仅 Ruin
        allowTrid = false,      -- 仅 Ruin
    }

---

##### `TensorCore.API.TensorRuin.deleteBlacklistEntry(contentid)`

**用途**：删除黑名单条目

---

##### `TensorCore.API.TensorRuin.getBlacklistEntry(contentid)`

**返回值**：(table | nil) - 黑名单条目或 nil

**示例**：

    -- 修改现有条目
    local blEntry = TensorCore.API.TensorRuin.getBlacklistEntry(9300)
    if blEntry ~= nil then
        blEntry.enabled = false
    end

    -- 添加新条目并允许当前目标
    local blEntry = TensorCore.API.TensorRuin.addBlacklistEntry(9300)
    blEntry.allowIfTarget = true

---

##### `TensorCore.API.TensorRuin.loadDotBlacklist()` / `saveDotBlacklist()`

**用途**：加载/保存黑名单文件（v2 中保持不变）

---

#### 状态查询

##### `TensorCore.API.TensorRuin.getCurrentPhase()`

**返回值**：(string) - 当前阶段

- `"filler1"` - 填充阶段 1
- `"filler2"` - 填充阶段 2
- `"dwt"` - 死翼龙
- `"prebahamut"` - 巴哈前
- `"bahamut"` - 巴哈姆特
- `"fbt"` - 凤凰

---

##### `TensorCore.API.TensorRuin.getDreadwyrmAetherStacks()`

**返回值**：(number) - 死翼龙以太层数（需要 2 层召唤巴哈）

**用途**：70 级判断哪个死翼龙是巴哈前的（会有 1 层进入 DWT）

---

#### 特殊控制

##### `TensorCore.API.TensorRuin.enableAetherpactHold(arenaType, arenaCenterPos, arenaLength, arenaWidth)`

**参数**：

- `arenaType` (string) - `"rectangle"` 或 `"circle"`
- `arenaCenterPos` (vector3) - 场地中心坐标
- `arenaLength` (number) - 长度
- `arenaWidth` (number, optional) - 宽度（矩形必填）

**用途**：延迟灵护使用以优化覆盖范围

---

### TensorMagnum (机工士)

#### 安全 Hotbar 变量

**变量列表**：

    ACR_TensorMagnum_Hotbar_Tactician_Safe = false
    ACR_TensorMagnum_Hotbar_ArmsLength_Safe = false
    ACR_TensorMagnum_Hotbar_SummonQueen_Safe = false
    ACR_TensorMagnum_Hotbar_Potion_Safe = false
    ACR_TensorMagnum_Hotbar_HeadGraze_Safe = false
    ACR_TensorMagnum_Hotbar_QueenOverdrive_Safe = false
    ACR_TensorMagnum_Hotbar_Detonate_Safe = false
    ACR_TensorMagnum_Hotbar_SecondWind_Safe = false
    ACR_TensorMagnum_Hotbar_Sprint_Safe = false
    ACR_TensorMagnum_Hotbar_LimitBreak_Safe = false
    ACR_TensorMagnum_Hotbar_Flamethrower_Safe = false

**用途**：设为 true 后，ACR 会等到超荷时才使用，不会打断精心安排的能力技穿插

---

### TensorRequiem (吟游诗人)

#### 歌曲优先级

##### `ACR_TensorRequiem2_SongPriority`

**格式**：

    ACR_TensorRequiem2_SongPriority = {"WM", "MB", "AP"}

**用途**：拖放式歌曲优先级设置

---

### TensorReaper (钐镰客)

#### `TensorCore.API.TensorReaper.addArcaneCrestBlacklistEntry(spellID)`

**用途**：添加奥秘纹黑名单（不自动使用的技能）

---

##### `TensorCore.API.TensorReaper.removeArcaneCrestBlacklistEntry(spellID)`

**用途**：移除黑名单条目

---

### TensorWeeb (武士)

#### `TensorCore.API.TensorWeeb2.addThirdEyeBlacklistEntry(spellid)`

**用途**：添加心眼黑名单

---

##### `TensorCore.API.TensorWeeb2.removeThirdEyeBlacklistEntry(spellid)`

**用途**：移除黑名单条目

---

### RikuAST (占星术士)

#### `TensorCore.API.RikuAST3.isLordOfCrownsDrawn()`

**返回值**：(bool) - 是否已抽到王冠之领主

---

##### `TensorCore.API.RikuAST3.isLadyOfCrownsDrawn()`

**返回值**：(bool) - 是否已抽到王冠之贵妇

---

### RikuRDM (赤魔法师)

#### `TensorCore.API.RikuRDM.inMeleeCombo(meleeOnly)`

**参数**：

- `meleeOnly` (bool) - 是否仅检查近战连击（不包括赤核爆/赤神圣/焦热）

**返回值**：(bool) - 是否在近战连击中

---

### RikuTanks (坦克)

#### `TensorCore.API.RikuWAR.toggleTankStance(stance)`

**参数**：

- `stance` (string, optional) - `"mt"` 或 `"ot"`，不指定则切换到相反姿态

**用途**：模拟左键点击切换坦克姿态

---

## 通用 ACR API

> **💡 通用别名**：以下示例使用具体 ACR 名称（如 `RikuAST`, `TensorWeeb2`），但都可以替换为 `TensorCore.API.TensorACR` 以实现职业无关的通用代码。

### Hotbar/Healbar 目标控制

**注意**：以下示例使用 `RikuAST`，可替换为任何支持的 ACR

#### Getter 函数

    -- 返回 entity 对象（不是 ID）
    TensorCore.API.RikuAST.getHotbarHoverTarget(string var)
    TensorCore.API.RikuAST.getTankbarHoverTarget(string var)
    TensorCore.API.RikuAST.getHealbarHoverTarget(string var)

#### Setter 函数

    -- entity 是实体对象（不是 ID）
    TensorCore.API.RikuAST.setHotbarHoverTarget(string var, entity target)
    TensorCore.API.RikuAST.setTankbarHoverTarget(string var, entity target)
    TensorCore.API.RikuAST.setHealbarHoverTarget(string var, entity target)

#### 世界坐标

    -- 返回 {x = 0, y = 0, z = 0}
    TensorCore.API.RikuAST.getHotbarMouseWorldPos(string var)
    TensorCore.API.RikuAST.getHealbarMouseWorldPos(string var)

    -- 设置世界坐标
    TensorCore.API.RikuAST.setHotbarMouseWorldPos(string var, number x, number y, number z)
    TensorCore.API.RikuAST.setHealbarMouseWorldPos(string var, number x, number y, number z)

#### 使用示例

    -- Essential Dignity 施放给主坦
    local mt = TensorCore.getEntityByGroup("Main Tank")
    if mt ~= nil then
        ACR_RikuAST_Healbar_EssentialDignity = true
        TensorCore.API.RikuAST.setHealbarHoverTarget("ACR_RikuAST_Healbar_EssentialDignity", mt)
    end

    -- 救援主坦
    local mt = TensorCore.getEntityByGroup("Main Tank")
    if mt ~= nil then
        ACR_RikuAST_Hotbar_Rescue = true
        TensorCore.API.RikuAST.setHotbarHoverTarget("ACR_RikuAST_Hotbar_Rescue", mt)
    end

---

### 优先级管理

#### 获取优先级列表

    TensorCore.API.RikuAST.getPlayerPriority()
    TensorCore.API.RikuAST.getBurstPriority()

**返回值格式**：

    {
        { job = 23, name = "Player One" },
        { job = 38, name = "Player Two" },
        { job = 27, name = "Player Three" },
    }

**注意**：直接修改返回的列表无效，必须调用 setter

---

#### 设置优先级

    TensorCore.API.RikuAST.setPlayerPriority(number currentPriority, number newPriority)
    TensorCore.API.RikuAST.setBurstPriority(number currentPriority, number newPriority)

**示例**：

    -- 将 Player Two 移到优先级第一位
    local prio = TensorCore.API.RikuAST.getPlayerPriority()
    for i = 1, #prio do
        if prio[i].name == "Player Two" then
            TensorCore.API.RikuAST.setPlayerPriority(i, 1)
            break
        end
    end

---

### 动作延迟系统

#### `TensorCore.API.[ACRName].holdActionUntil(id, time, numCharges)`

**参数**：

- `id` (int) - 技能 ID（846 = 爆发药）
- `time` (int) - 持有到的时间（毫秒，使用 `Now() + 延迟`）
- `numCharges` (int, optional) - 保留层数

**用途**：精确控制技能释放时机

---

##### `TensorCore.API.[ACRName].holdActionFor(id, seconds, numCharges)`

**参数**：

- `seconds` (float) - 持有秒数 + CD 时间

**区别于关闭选项**：告知 ACR 持有时长，避免消耗资源（如剑气、超荷等）

---

#### 使用场景

1. **武士 Balance Opener**：延迟闪影 7秒

    TensorCore.API.TensorWeeb2.holdActionUntil(16481, Now() + 7000)
    self.used = true

1. **诗人第 3 GCD Buff**：

    TensorCore.API.TensorRequiem2.holdActionUntil(118, Now() + 5000)    -- 战逸
    TensorCore.API.TensorRequiem2.holdActionUntil(25785, Now() + 5000)  -- 光明神
    TensorCore.API.TensorRequiem2.holdActionUntil(3562, Now() + 5000)   -- 侧风诱导箭
    self.used = true

1. **绝枪第 3 GCD**：

    TensorCore.API.RikuGNB2.holdActionUntil(16138, Now() + 5000)  -- 无情
    TensorCore.API.RikuGNB2.holdActionUntil(846, Now() + 2500)    -- 爆发药
    TensorCore.API.RikuGNB2.holdActionUntil(16164, Now() + 7000)  -- 血壤
    self.used = true

**注意**：

- 擦除后自动重置
- 使用时需在时间轴第一个机制设置反应（时间范围 -11 到 0）

---

### Lock Face 系统

#### `TensorCore.API.TensorACR.setLockFaceHeading(radian)`

**参数**：

- `radian` (number) - 弧度值

**用途**：设置锁定朝向

---

##### `TensorCore.API.TensorACR.toggleLockFace(bool)`

**用途**：切换锁定朝向开关

**注意**：`TensorCore.API.TensorACR` 会自动解析为当前激活的 ACR

---

## API 变更记录

### 废弃 API（v2.0）

 | 旧 API | 新 API |
 | -------- | -------- |
 | `getDotBlacklist()` | `getBlacklistEntry()` |
 | `getTimelineTriggerProfileName()` | `getTimelineReactionProfileName()` |
 | `getGeneralTriggerProfileName()` | `getGeneralReactionProfileName()` |
 | `reloadGeneralTriggers()` | `reloadGeneralReactions()` |
 | `reloadTimelineTriggers()` | `reloadTimelineReactions()` |

### 参数变更

- `TensorCore.Avoidance.inUnavoidableAOE()`: 第二个参数从 `pos` 改为 `entity`

---

## 相关资源

- **Argus Wiki**: <https://wiki.mmominion.com/doku.php?id=argusdocs>
- **TensorReactions 2.0 更新日志**: <https://gist.github.com/Rikudouu/429abc931f1ebd39445d90efb57df768>
- **中文更新日志**: <https://gist.github.com/Rikudouu/37f528f5530de33a21b30a440a963b2e>

---

**文档版本**：1.0  
**API 覆盖范围**：TensorCore + Argus + ACR 全系列  
**总 API 数量**：120+
