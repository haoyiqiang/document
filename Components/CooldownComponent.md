# CooldownComponent

## 概述
`CooldownComponent` 是一个冷却时间管理组件，用于控制玩家不能过于频繁执行的动作，如射击、挖掘资源等。提供基础的冷却机制，通常作为其他组件（如`GunComponent`、`InteractionControlComponent`）的基类。

**继承关系：** `CooldownComponent` → `Component` → `Node`

## 主要特性
- ⏱️ 精确的冷却时间控制
- 🔄 可配置的冷却时长
- 📊 统计数据修改器支持
- 🎯 信号驱动的状态通知
- 🛡️ 防零值错误保护
- 🎮 作为基类易于扩展

## 依赖节点
- **CooldownTimer: Timer** (子节点) - 内部计时器节点

## 导出属性

### `cooldown: float = 3`
基础冷却时间（秒）。

**类型：** `float`  
**范围：** `0.05` 到 `120` 秒  
**步进：** `0.05`  
**默认值：** `3.0`  
**setter保护：** 自动处理零值和负值  
**特性：** 自动更新内部计时器的等待时间

### `cooldownMillisecondsModifier: Stat`
可选的冷却时间修改器统计数据。

**类型：** `Stat`  
**默认值：** `null`  
**单位：** 毫秒（1000 = 1秒）  
**用途：** 允许升级或debuff影响冷却时间  
**实验特性：** 标记为@experimental

## 状态属性

### `cooldownTimer: Timer`
内部计时器节点引用。

**类型：** `Timer`  
**获取：** `@onready var cooldownTimer: Timer = $CooldownTimer`  
**用途：** 管理实际的计时逻辑

### `cooldownWithModifier: float` (只读)
包含修改器的总冷却时间。

**类型：** `float`  
**计算：** `cooldown + (cooldownMillisecondsModifier.value / 1000.0)`  
**返回：** 基础冷却时间加上统计修改器（如果存在）

### `hasCooldownCompleted: bool`
冷却是否已完成的标志。

**类型：** `bool`  
**默认值：** `true` (开始时冷却已完成)  
**用途：** 快速检查是否可以执行动作

## 常量

### `minimumTimerWaitTime: float = 0.05`
计时器的最小等待时间。

**用途：** 避免Godot的"时间应该大于零"错误

## 信号

### `didStartCooldown`
冷却开始时发出。

**时机：** 调用`startCooldown()`时

### `didFinishCooldown`
冷却完成时发出。

**时机：** 计时器超时或调用`finishCooldown()`时

## 核心方法

### `startCooldown(overrideTime: float = self.cooldownWithModifier) -> void`
开始冷却计时。

**参数：** `overrideTime` - 可选的自定义冷却时间，默认使用`cooldownWithModifier`  
**功能：**
1. 设置`hasCooldownCompleted = false`
2. 配置并启动计时器
3. 发出`didStartCooldown`信号
4. 处理边界情况（零值保护）

### `finishCooldown() -> void`
完成冷却。

**功能：**
1. 停止计时器
2. 设置`hasCooldownCompleted = true`
3. 发出`didFinishCooldown`信号

## 使用示例

### 基本冷却设置
```gdscript
# 简单的冷却组件使用
func setupBasicCooldown():
    var cooldownComponent = $CooldownComponent
    
    # 设置2秒冷却
    cooldownComponent.cooldown = 2.0
    
    # 连接信号
    cooldownComponent.didStartCooldown.connect(_on_cooldown_started)
    cooldownComponent.didFinishCooldown.connect(_on_cooldown_finished)

func _on_cooldown_started():
    print("冷却开始")
    # 禁用相关UI按钮
    $UI/ActionButton.disabled = true

func _on_cooldown_finished():
    print("冷却完成")
    # 启用相关UI按钮
    $UI/ActionButton.disabled = false
```

### 射击武器冷却
```gdscript
# 扩展冷却组件用于武器系统
class_name WeaponComponent
extends CooldownComponent

@export var damage: int = 10
@export var ammo: int = 30

func _ready():
    super._ready()
    
    # 设置武器射击间隔
    cooldown = 0.5  # 0.5秒射击间隔

func tryFire() -> bool:
    if not hasCooldownCompleted:
        print("武器冷却中...")
        return false
    
    if ammo <= 0:
        print("弹药不足!")
        return false
    
    # 执行射击
    fire()
    
    # 开始冷却
    startCooldown()
    ammo -= 1
    
    return true

func fire():
    print("开火! 剩余弹药: ", ammo)
    # 生成子弹，播放音效等
```

### 带统计修改器的冷却
```gdscript
# 使用统计数据修改冷却时间
func setupModifiableCooldown():
    var cooldownComponent = $CooldownComponent
    
    # 创建攻击速度统计
    var attackSpeed = Stat.new()
    attackSpeed.name = "AttackSpeed"
    attackSpeed.value = 0  # 初始无修改
    
    # 设置为冷却修改器
    cooldownComponent.cooldownMillisecondsModifier = attackSpeed
    cooldownComponent.cooldown = 1.0  # 基础1秒冷却

# 升级影响冷却时间
func upgradeAttackSpeed():
    var cooldownComponent = $CooldownComponent
    var modifier = cooldownComponent.cooldownMillisecondsModifier
    
    if modifier:
        # 减少300毫秒冷却时间
        modifier.value -= 300
        print("攻击速度提升! 当前冷却: ", cooldownComponent.cooldownWithModifier, "秒")

# 应用减速debuff
func applySlowDebuff(duration: float):
    var cooldownComponent = $CooldownComponent
    var modifier = cooldownComponent.cooldownMillisecondsModifier
    
    if modifier:
        var originalValue = modifier.value
        
        # 增加500毫秒冷却时间
        modifier.value += 500
        print("减速效果! 冷却增加到: ", cooldownComponent.cooldownWithModifier, "秒")
        
        # 定时恢复
        await get_tree().create_timer(duration).timeout
        modifier.value = originalValue
        print("减速效果结束")
```

### 技能冷却系统
```gdscript
# 多技能冷却管理
class_name SkillManager
extends Node

var skills: Dictionary = {}

func _ready():
    setupSkills()

func setupSkills():
    # 火球术 - 3秒冷却
    var fireball = CooldownComponent.new()
    fireball.cooldown = 3.0
    fireball.name = "FireballCooldown"
    add_child(fireball)
    skills["fireball"] = fireball
    
    # 治疗术 - 5秒冷却
    var heal = CooldownComponent.new()
    heal.cooldown = 5.0
    heal.name = "HealCooldown"
    add_child(heal)
    skills["heal"] = heal
    
    # 护盾术 - 10秒冷却
    var shield = CooldownComponent.new()
    shield.cooldown = 10.0
    shield.name = "ShieldCooldown"
    add_child(shield)
    skills["shield"] = shield

func castSkill(skillName: String) -> bool:
    if not skills.has(skillName):
        print("未知技能: ", skillName)
        return false
    
    var cooldown: CooldownComponent = skills[skillName]
    
    if not cooldown.hasCooldownCompleted:
        var remaining = cooldown.cooldownTimer.time_left
        print(skillName, " 冷却中，剩余时间: ", remaining, "秒")
        return false
    
    # 执行技能效果
    executeSkillEffect(skillName)
    
    # 开始冷却
    cooldown.startCooldown()
    return true

func executeSkillEffect(skillName: String):
    match skillName:
        "fireball":
            print("释放火球术!")
            # 造成伤害逻辑
        "heal":
            print("释放治疗术!")
            # 治疗逻辑
        "shield":
            print("释放护盾术!")
            # 护盾逻辑

# 检查技能冷却状态
func getSkillCooldownInfo(skillName: String) -> Dictionary:
    if not skills.has(skillName):
        return {}
    
    var cooldown: CooldownComponent = skills[skillName]
    return {
        "ready": cooldown.hasCooldownCompleted,
        "totalCooldown": cooldown.cooldownWithModifier,
        "remaining": cooldown.cooldownTimer.time_left if not cooldown.hasCooldownCompleted else 0.0
    }
```

### 采集系统冷却
```gdscript
# 资源采集组件
class_name HarvestingComponent
extends CooldownComponent

@export var resourceType: String = "wood"
@export var harvestAmount: int = 1
@export var requiredTool: String = "axe"

func _ready():
    super._ready()
    
    # 不同资源有不同采集间隔
    match resourceType:
        "wood":
            cooldown = 2.0
        "stone":
            cooldown = 3.0
        "metal":
            cooldown = 4.0

func tryHarvest(player: Entity) -> bool:
    if not hasCooldownCompleted:
        print("采集冷却中...")
        return false
    
    if not checkToolRequirement(player):
        print("需要工具: ", requiredTool)
        return false
    
    # 执行采集
    harvest(player)
    
    # 开始冷却
    startCooldown()
    return true

func checkToolRequirement(player: Entity) -> bool:
    var inventory = player.get_node("InventoryComponent")
    if not inventory:
        return false
    
    return inventory.hasItemType(requiredTool)

func harvest(player: Entity):
    print("采集了 ", harvestAmount, " 个 ", resourceType)
    
    # 给玩家添加资源
    var inventory = player.get_node("InventoryComponent")
    if inventory:
        inventory.addResource(resourceType, harvestAmount)
    
    # 播放采集音效和特效
    playHarvestEffect()

func playHarvestEffect():
    # 播放采集音效
    $SFX/HarvestSound.play()
    
    # 显示采集特效
    $VFX/HarvestParticles.emit()
```

### 动态冷却调整
```gdscript
# 根据游戏状态动态调整冷却
func adjustCooldownByGameState():
    var cooldownComponent = $CooldownComponent
    
    # 根据玩家等级调整冷却
    var playerLevel = GameState.playerLevel
    var levelBonus = playerLevel * 0.1  # 每级减少10%冷却
    var adjustedCooldown = cooldownComponent.cooldown * (1.0 - levelBonus)
    
    # 应用调整
    cooldownComponent.startCooldown(adjustedCooldown)

# 装备影响冷却时间
func applyEquipmentBonus(equipmentBonus: float):
    var cooldownComponent = $CooldownComponent
    
    # 装备减少冷却时间
    var bonusCooldown = cooldownComponent.cooldown * (1.0 - equipmentBonus)
    cooldownComponent.startCooldown(bonusCooldown)

# 环境效果影响冷却
func applyEnvironmentalEffect(environmentType: String):
    var cooldownComponent = $CooldownComponent
    var multiplier = 1.0
    
    match environmentType:
        "hot":
            multiplier = 1.2  # 炎热环境增加冷却
        "cold":
            multiplier = 0.8  # 寒冷环境减少冷却
        "magic_field":
            multiplier = 0.5  # 魔法场域大幅减少冷却
    
    var adjustedCooldown = cooldownComponent.cooldown * multiplier
    cooldownComponent.startCooldown(adjustedCooldown)
```

### UI集成示例
```gdscript
# 冷却UI显示
class_name CooldownUI
extends Control

@export var cooldownComponent: CooldownComponent
@onready var progressBar: ProgressBar = $ProgressBar
@onready var label: Label = $Label

func _ready():
    if cooldownComponent:
        cooldownComponent.didStartCooldown.connect(_on_cooldown_started)
        cooldownComponent.didFinishCooldown.connect(_on_cooldown_finished)

func _on_cooldown_started():
    progressBar.visible = true
    progressBar.max_value = cooldownComponent.cooldownWithModifier
    
    # 开始更新进度条
    set_process(true)

func _on_cooldown_finished():
    progressBar.visible = false
    label.text = "就绪"
    set_process(false)

func _process(delta):
    if cooldownComponent and not cooldownComponent.hasCooldownCompleted:
        var remaining = cooldownComponent.cooldownTimer.time_left
        progressBar.value = cooldownComponent.cooldownWithModifier - remaining
        label.text = str(ceil(remaining)) + "s"
```

## 设计模式

### 模板方法模式
- **基类框架：** 提供冷却管理的基础框架
- **扩展点：** 子类可重写具体的动作执行逻辑
- **通用流程：** 统一的冷却开始-等待-完成流程

### 观察者模式
- **信号通知：** 冷却状态变化时通知其他系统
- **UI更新：** UI组件监听冷却信号更新显示
- **游戏逻辑：** 其他系统响应冷却状态变化

### 策略模式
- **修改器系统：** 通过Stat实现不同的冷却修改策略
- **时间策略：** 支持固定时间和动态计算时间
- **继承策略：** 不同子类实现不同的冷却应用策略

## 技术细节

### 零值保护
```gdscript
if overrideTime > 0 and not is_zero_approx(overrideTime):
    cooldownTimer.start(overrideTime)
else:
    finishCooldown()  # 立即完成
```

### 修改器计算
```gdscript
var cooldownWithModifier: float:
    get:
        if cooldownMillisecondsModifier: 
            return cooldown + (float(cooldownMillisecondsModifier.value) / 1000.0)
        else: 
            return cooldown
```

### 信号自动连接
```gdscript
func _ready():
    Tools.connectSignal(cooldownTimer.timeout, self.finishCooldown)
```

### setter保护
自动处理边界值并更新计时器配置。

## 注意事项

1. **最小时间：** 冷却时间不应小于0.05秒
2. **统计单位：** 修改器统计使用毫秒为单位
3. **信号连接：** 确保计时器信号正确连接
4. **子类扩展：** 作为基类时注意调用super._ready()
5. **节点结构：** 需要CooldownTimer子节点

## 常见用途
- 武器射击间隔控制
- 技能释放冷却管理
- 资源采集速度限制
- 交互动作防刷机制
- 特殊能力使用间隔

## 相关组件
- `GunComponent` - 武器组件，常继承此组件
- `InteractionControlComponent` - 交互控制，可使用冷却机制
- `ActionsComponent` - 行动组件，与冷却系统协作
- `StatsComponent` - 统计系统，提供修改器支持 