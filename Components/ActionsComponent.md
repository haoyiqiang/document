# ActionsComponent

## 概述
`ActionsComponent` 是一个特殊行动管理组件，存储和管理实体可以执行的特殊技能、法术、或命令列表。行动可能消耗统计资源，需要选择目标，并具有冷却时间机制。

**继承关系：** `ActionsComponent` → `Component` → `Node`

## 主要特性
- 🎯 管理特殊行动/技能列表
- 💰 支持资源消耗系统
- 🎯 目标选择机制
- ⏰ 冷却时间管理
- ⌨️ 键盘快捷键支持
- 🤖 AI友好的API

## 依赖组件
- **StatsComponent** (可选) - 用于消耗资源的行动

## 导出属性

### `actions: Array[Action]`
实体可执行的行动列表。

**类型：** `Array[Action]`  
**描述：** 包含所有可用的特殊行动、技能或命令  
**示例：** 火球术、冲刺、检查、治疗等

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`

## 状态属性

### `actionsOnCooldown: Array[Action]`
当前处于冷却状态的行动列表。

**类型：** `Array[Action]`  
**存储特性：** `@export_storage`  
**用途：** 每帧更新冷却时间

## 信号

### `didRequestTarget(action: Action, source: Entity)`
当行动需要目标但未提供时发出。

**参数：**
- `action` - 需要目标的行动
- `source` - 执行行动的实体

**处理方：** `ActionControlComponent`

### `willPerformAction(action: Action)`
即将执行行动时发出。

**参数：** `action` - 将要执行的行动

### `didPerformAction(action: Action, result: Variant)`
行动执行完成后发出。

**参数：**
- `action` - 已执行的行动
- `result` - 行动执行结果

## 核心方法

### `findAction(nameToSearch: StringName) -> Action`
查找指定名称的行动。

**参数：** `nameToSearch` - 要查找的行动名称  
**返回：** 找到的Action对象或null  
**用途：** 通过名称获取行动引用

### `findActionForInputEvent(inputEvent: InputEvent) -> Action`
查找匹配输入事件的行动。

**参数：** `inputEvent` - 输入事件  
**返回：** 匹配的Action对象或null  
**用途：** 处理键盘快捷键  
**命名规则：** `specialAction_` + action.name

### `performAction(actionName: StringName, target: Entity = null) -> Variant`
执行指定的行动。

**参数：**
- `actionName` - 行动名称
- `target` - 目标实体（可选）

**返回：** 行动执行结果或false  
**功能：**
- 检查目标需求
- 验证资源消耗
- 执行行动载荷
- 发出相关信号

## 冷却系统方法

### `createCooldownsList() -> void`
创建并更新冷却行动列表。

**功能：**
- 清空当前冷却列表
- 检查所有行动的冷却状态
- 启动或停止每帧处理

### `connectSignals() -> void`
连接所有行动的冷却信号。

**功能：** 监听行动的冷却开始和结束事件

### `onAction_didStartCooldown(action: Action) -> void`
行动开始冷却时的回调。

**参数：** `action` - 开始冷却的行动  
**功能：** 添加到冷却列表并启动更新

### `onAction_didFinishCooldown(action: Action) -> void`
行动结束冷却时的回调。

**参数：** `action` - 结束冷却的行动  
**功能：** 从冷却列表移除

### `_process(delta: float) -> void`
每帧更新冷却时间。

**参数：** `delta` - 帧时间间隔  
**优化：** 仅在有冷却行动时执行

## 使用示例

### 基本行动设置
```gdscript
# 获取行动组件
var actionsComponent = $ActionsComponent

# 创建基础行动
var dashAction = preload("res://Resources/Actions/Dash.tres")
var healAction = preload("res://Resources/Actions/Heal.tres")

# 添加到行动列表
actionsComponent.actions = [dashAction, healAction]

# 连接信号
actionsComponent.didPerformAction.connect(_on_action_performed)
```

### 执行行动
```gdscript
# 执行无目标行动（如冲刺）
func performDash():
    var actionsComponent = $ActionsComponent
    var result = actionsComponent.performAction("dash")
    
    if result:
        print("冲刺成功!")
    else:
        print("冲刺失败或冷却中")

# 执行有目标行动（如治疗）
func healTarget(target: Entity):
    var actionsComponent = $ActionsComponent
    var result = actionsComponent.performAction("heal", target)
    
    if result:
        print("治疗了 ", target.name)
```

### 键盘快捷键
```gdscript
# 在项目设置的输入映射中添加：
# "specialAction_fireball" -> F1键
# "specialAction_heal" -> F2键
# "specialAction_dash" -> F3键

func _ready():
    var actionsComponent = $ActionsComponent
    
    # ActionControlComponent会自动处理这些快捷键
    # 或者手动处理：
    pass

func _input(event):
    if event is InputEventKey and event.pressed:
        var actionsComponent = $ActionsComponent
        var action = actionsComponent.findActionForInputEvent(event)
        
        if action:
            actionsComponent.performAction(action.name)
```

### 法师角色示例
```gdscript
# 设置法师的法术列表
func setupMageActions():
    var actionsComponent = $ActionsComponent
    
    # 加载法术资源
    var fireball = preload("res://Spells/Fireball.tres")
    var heal = preload("res://Spells/Heal.tres")
    var shield = preload("res://Spells/Shield.tres")
    var teleport = preload("res://Spells/Teleport.tres")
    
    # 设置法术列表
    actionsComponent.actions = [fireball, heal, shield, teleport]
    
    # 连接法术事件
    actionsComponent.willPerformAction.connect(_on_spell_start)
    actionsComponent.didPerformAction.connect(_on_spell_complete)

func _on_spell_start(action: Action):
    print("开始施法: ", action.name)
    # 播放施法动画
    $AnimationPlayer.play("cast_" + action.name)

func _on_spell_complete(action: Action, result):
    print("法术完成: ", action.name, " 结果: ", result)
    # 显示法术效果
    showSpellEffect(action.name)
```

### 战士角色示例
```gdscript
# 设置战士的技能列表
func setupWarriorActions():
    var actionsComponent = $ActionsComponent
    
    # 加载技能资源
    var slash = preload("res://Skills/PowerSlash.tres")
    var charge = preload("res://Skills/Charge.tres")
    var block = preload("res://Skills/Block.tres")
    var rage = preload("res://Skills/Rage.tres")
    
    actionsComponent.actions = [slash, charge, block, rage]

# 连击系统
func executeCombo():
    var actionsComponent = $ActionsComponent
    
    # 连续执行技能组合
    actionsComponent.performAction("power_slash")
    await get_tree().create_timer(0.5).timeout
    actionsComponent.performAction("charge")
```

### 动态行动管理
```gdscript
# 根据等级解锁技能
func unlockSkillsByLevel(level: int):
    var actionsComponent = $ActionsComponent
    var availableActions = []
    
    # 基础技能
    availableActions.append(preload("res://Skills/BasicAttack.tres"))
    
    if level >= 5:
        availableActions.append(preload("res://Skills/SpecialAttack.tres"))
    
    if level >= 10:
        availableActions.append(preload("res://Skills/UltimateSkill.tres"))
    
    actionsComponent.actions = availableActions

# 临时获得技能（如道具效果）
func grantTemporarySkill(skillResource: Action, duration: float):
    var actionsComponent = $ActionsComponent
    
    # 添加临时技能
    actionsComponent.actions.append(skillResource)
    
    # 定时移除
    await get_tree().create_timer(duration).timeout
    actionsComponent.actions.erase(skillResource)
```

### 目标选择系统
```gdscript
# 处理需要目标的行动
func _ready():
    var actionsComponent = $ActionsComponent
    actionsComponent.didRequestTarget.connect(_on_target_requested)

func _on_target_requested(action: Action, source: Entity):
    print("技能 ", action.name, " 需要选择目标")
    
    # 显示目标选择UI
    showTargetSelector(action)

func showTargetSelector(action: Action):
    # 获取可选目标
    var validTargets = getValidTargets(action)
    
    # 显示目标选择UI
    $TargetSelectorUI.show()
    $TargetSelectorUI.setTargets(validTargets)
    
    # 等待用户选择
    var selectedTarget = await $TargetSelectorUI.target_selected
    
    # 执行带目标的行动
    var actionsComponent = $ActionsComponent
    actionsComponent.performAction(action.name, selectedTarget)

func getValidTargets(action: Action) -> Array[Entity]:
    # 根据行动类型返回有效目标
    var targets: Array[Entity] = []
    
    if action.name == "heal":
        # 治疗技能 - 选择受伤的友军
        targets = GameState.getAllies().filter(func(ally): return ally.health < ally.maxHealth)
    elif action.name == "fireball":
        # 攻击技能 - 选择敌人
        targets = GameState.getEnemies()
    
    return targets
```

### 冷却管理
```gdscript
# 监控技能冷却
func monitorCooldowns():
    var actionsComponent = $ActionsComponent
    
    # 检查特定技能是否可用
    var dashAction = actionsComponent.findAction("dash")
    if dashAction and not dashAction.isInCooldown:
        print("冲刺技能可用")
    
    # 显示所有冷却中的技能
    for action in actionsComponent.actionsOnCooldown:
        print(action.name, " 剩余冷却: ", action.cooldownRemaining, "秒")

# 重置所有冷却
func resetAllCooldowns():
    var actionsComponent = $ActionsComponent
    
    for action in actionsComponent.actions:
        action.cooldownRemaining = 0.0
        action.isInCooldown = false
    
    # 清空冷却列表
    actionsComponent.actionsOnCooldown.clear()
    actionsComponent.set_process(false)
```

### AI使用示例
```gdscript
# AI选择和执行技能
func aiChooseAction() -> bool:
    var actionsComponent = $ActionsComponent
    
    # AI决策：选择最佳行动
    var bestAction = chooseBestAction()
    if not bestAction:
        return false
    
    # 检查是否需要目标
    var target = null
    if bestAction.requiresTarget:
        target = chooseTarget(bestAction)
        if not target:
            return false
    
    # 执行行动
    var result = actionsComponent.performAction(bestAction.name, target)
    return result != false

func chooseBestAction() -> Action:
    var actionsComponent = $ActionsComponent
    
    # 简单AI：选择第一个可用的攻击技能
    for action in actionsComponent.actions:
        if not action.isInCooldown and action.name.contains("attack"):
            return action
    
    return null

func chooseTarget(action: Action) -> Entity:
    # AI选择目标
    if action.name.contains("heal"):
        # 选择血量最低的友军
        return findLowestHealthAlly()
    else:
        # 选择最近的敌人
        return findNearestEnemy()
```

## 设计模式

### 命令模式
- **行动封装：** 每个Action都是一个独立的命令对象
- **参数化：** 支持不同的目标和载荷
- **撤销支持：** 可通过载荷实现撤销机制

### 观察者模式
- **信号系统：** 行动执行前后发出信号
- **UI更新：** UI组件监听行动状态变化
- **效果系统：** 特效和音频响应行动事件

### 策略模式
- **载荷系统：** 不同类型的载荷实现不同策略
- **目标选择：** 可自定义目标选择策略
- **冷却策略：** 支持不同的冷却机制

## 技术细节

### 性能优化
- **按需处理：** 仅在有冷却行动时启动_process
- **信号连接：** 动态连接行动信号避免内存泄漏
- **缓存查找：** 考虑使用字典缓存行动查找

### 快捷键系统
- **命名约定：** `specialAction_` + 行动名称
- **自动映射：** 自动检测匹配的输入映射
- **AI兼容：** 支持合成InputEvent

### 扩展性
- **载荷系统：** 通过Payload接口支持各种行动类型
- **继承友好：** 可轻松继承实现特定游戏逻辑
- **模块化：** 与其他组件松耦合

## 注意事项

1. **资源消耗：** 需要StatsComponent才能处理资源消耗
2. **目标验证：** 必须验证目标的有效性
3. **信号顺序：** 注意信号发出的时机和顺序
4. **内存管理：** 正确处理行动资源的加载和释放
5. **冷却同步：** 确保冷却时间在保存/加载时正确同步

## 相关组件
- `ActionControlComponent` - 处理玩家控制和目标选择
- `StatsComponent` - 提供资源消耗支持
- `ActionButtons` / `ActionButtonsList` - UI显示组件
- `CooldownComponent` - 额外的冷却管理功能 