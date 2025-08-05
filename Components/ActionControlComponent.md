# ActionControlComponent

## 概述
`ActionControlComponent` 是一个玩家行动控制组件，接收玩家输入并调用`ActionsComponent.performAction`来执行特殊行动。支持键盘快捷键、gamepad按钮和UI按钮触发行动，并处理需要目标选择的行动。

**继承关系：** `ActionControlComponent` → `Component` → `Node`

## 主要特性
- ⌨️ 键盘快捷键支持
- 🎮 手柄按钮支持
- 🖱️ UI按钮事件处理
- 🎯 自动目标选择系统
- 🔧 可配置目标选择组件
- 📊 与ActionsComponent无缝集成
- 🎭 支持合成输入事件

## 依赖组件
- **ActionsComponent** (必需) - 行动管理组件

## 导出属性

### `targetingComponentPath: String`
目标选择组件的场景路径。

**类型：** `String`  
**默认值：** `"res://Components/Control/ActionTargetingMouseComponent.tscn"`  
**用途：** 当行动需要目标时实例化的组件  
**限制：** 必须是ActionTargetingComponentBase的子类

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**setter特性：** 自动控制输入处理的启用状态

## 状态属性

### `actionsComponent: ActionsComponent` (只读)
关联的行动组件引用。

**类型：** `ActionsComponent`  
**获取：** `@onready var actionsComponent: ActionsComponent = coComponents.ActionsComponent`  
**用途：** 执行具体的行动逻辑

## 核心方法

### `_input(event: InputEvent) -> void`
处理合成输入事件。

**参数：** `event` - 输入事件，主要处理ActionButton等生成的InputEventAction  
**功能：**
1. 检查事件是否为InputEventAction
2. 验证是否为特殊行动（以specialActionPrefix开头）
3. 提取行动名称并执行

### `_unhandled_input(event: InputEvent) -> void`
处理未处理的输入事件。

**功能：**
1. 检查键盘快捷键和手柄按钮
2. 查找匹配的行动
3. 执行行动并标记输入已处理

### `onActionsComponent_didRequestTarget(action: Action, source: Entity) -> void`
行动组件请求目标时的处理。

**参数：**
- `action` - 需要目标的行动
- `source` - 发起行动的实体

**功能：** 为当前实体创建目标选择组件

### `createTargetingComponent(actionToPerform: Action) -> ActionTargetingComponentBase`
创建目标选择组件。

**参数：** `actionToPerform` - 要执行的行动  
**返回：** ActionTargetingComponentBase - 创建的目标选择组件  
**功能：**
1. 从路径加载目标选择组件场景
2. 实例化组件并设置行动
3. 添加到父实体

## 使用示例

### 基本设置
```gdscript
# 为玩家实体设置行动控制
func setupPlayerActionControl():
    var actionControl = $ActionControlComponent
    
    # 启用行动控制
    actionControl.isEnabled = true
    
    # 确保有ActionsComponent
    var actions = $ActionsComponent
    if not actions:
        print("需要ActionsComponent!")
        return
    
    # 添加一些行动
    actions.actions = [
        createFireballAction(),
        createHealAction(),
        createDashAction()
    ]

func createFireballAction() -> Action:
    var action = Action.new()
    action.name = "fireball"
    action.displayName = "火球术"
    action.manaCost = 20
    action.requiresTarget = true
    return action

func createHealAction() -> Action:
    var action = Action.new()
    action.name = "heal"
    action.displayName = "治疗术"
    action.manaCost = 15
    action.requiresTarget = false
    return action
```

### 键盘快捷键设置
```gdscript
# 在项目设置的Input Map中添加输入动作：
# - "specialAction_fireball" 映射到键盘F键
# - "specialAction_heal" 映射到键盘H键
# - "specialAction_dash" 映射到键盘Space键

# ActionControlComponent会自动处理这些快捷键
# 玩家按F键时会执行火球术
# 玩家按H键时会执行治疗术
# 玩家按空格键时会执行冲刺
```

### 自定义目标选择
```gdscript
# 创建自定义目标选择组件
func setupCustomTargeting():
    var actionControl = $ActionControlComponent
    
    # 使用自定义的目标选择组件
    actionControl.targetingComponentPath = "res://UI/CustomActionTargeting.tscn"
    
    # 连接目标选择事件
    var actions = $ActionsComponent
    actions.didRequestTarget.connect(_on_target_requested)

func _on_target_requested(action: Action, source: Entity):
    print("行动 ", action.displayName, " 需要选择目标")
    
    # 可以在这里显示自定义UI或给出提示
    show_targeting_hint(action)

func show_targeting_hint(action: Action):
    var hintUI = get_node("UI/TargetingHint")
    hintUI.text = "请选择 " + action.displayName + " 的目标"
    hintUI.visible = true
```

### 手柄支持
```gdscript
# 在Input Map中添加手柄按钮：
# - "specialAction_primary" 映射到手柄Y按钮
# - "specialAction_secondary" 映射到手柄X按钮
# - "specialAction_ultimate" 映射到手柄B按钮

func setupGamepadActions():
    var actions = $ActionsComponent
    
    # 添加手柄专用行动
    var primaryAction = Action.new()
    primaryAction.name = "primary"
    primaryAction.displayName = "主要攻击"
    
    var secondaryAction = Action.new()
    secondaryAction.name = "secondary"
    secondaryAction.displayName = "次要技能"
    
    var ultimateAction = Action.new()
    ultimateAction.name = "ultimate"
    ultimateAction.displayName = "终极技能"
    ultimateAction.requiresTarget = true
    
    actions.actions = [primaryAction, secondaryAction, ultimateAction]
```

### UI按钮集成
```gdscript
# 与ActionButtons UI组件集成
class_name SkillUI
extends Control

@onready var fireballButton: ActionButton = $VBoxContainer/FireballButton
@onready var healButton: ActionButton = $VBoxContainer/HealButton
@onready var dashButton: ActionButton = $VBoxContainer/DashButton

func _ready():
    setupActionButtons()

func setupActionButtons():
    # 设置按钮对应的行动名称
    fireballButton.actionName = "fireball"
    healButton.actionName = "heal"
    dashButton.actionName = "dash"
    
    # ActionButton会自动生成InputEventAction事件
    # ActionControlComponent会接收这些事件并执行相应行动

# ActionButton会生成类似这样的事件：
# var inputEvent = InputEventAction.new()
# inputEvent.action = "specialAction_fireball"
# inputEvent.pressed = true
# Input.parse_input_event(inputEvent)
```

### 条件行动执行
```gdscript
# 带条件检查的行动控制
class_name ConditionalActionControl
extends ActionControlComponent

@export var minLevelForAdvancedActions: int = 5

func _input(event: InputEvent) -> void:
    if not isEnabled or not event is InputEventAction: 
        return
    
    var eventAction: InputEventAction = event as InputEventAction
    var eventName: StringName = eventAction.action
    
    if not eventName.to_lower().begins_with(GlobalInput.Actions.specialActionPrefix.to_lower()): 
        return
    
    var actionName: StringName = eventName.trim_prefix(GlobalInput.Actions.specialActionPrefix)
    
    # 检查行动执行条件
    if canExecuteAction(actionName):
        if actionsComponent: 
            actionsComponent.performAction(actionName)
    else:
        showActionDeniedMessage(actionName)

func canExecuteAction(actionName: StringName) -> bool:
    var playerLevel = GameState.playerLevel
    
    # 高级行动需要等级限制
    var advancedActions = ["meteor", "timeStop", "resurrection"]
    if actionName in advancedActions and playerLevel < minLevelForAdvancedActions:
        return false
    
    # 检查法力值
    var action = actionsComponent.findAction(actionName)
    if action and action.manaCost > 0:
        var stats = get_parent().get_node("StatsComponent")
        if stats and stats.getMana() < action.manaCost:
            return false
    
    return true

func showActionDeniedMessage(actionName: StringName):
    var message = ""
    var playerLevel = GameState.playerLevel
    
    var advancedActions = ["meteor", "timeStop", "resurrection"]
    if actionName in advancedActions and playerLevel < minLevelForAdvancedActions:
        message = "需要等级 " + str(minLevelForAdvancedActions) + " 才能使用此技能"
    else:
        message = "法力值不足"
    
    # 显示消息
    GameUI.showMessage(message)
```

### 动态行动管理
```gdscript
# 根据游戏状态动态管理可用行动
func updateAvailableActions():
    var actions = $ActionsComponent
    var actionControl = $ActionControlComponent
    
    # 根据职业解锁不同行动
    var playerClass = GameState.playerClass
    var availableActions: Array[Action] = []
    
    match playerClass:
        "warrior":
            availableActions = [
                createAction("slash", "劈砍", 0, false),
                createAction("charge", "冲锋", 10, true),
                createAction("battleCry", "战吼", 15, false)
            ]
        "mage":
            availableActions = [
                createAction("magicMissile", "魔法飞弹", 5, true),
                createAction("fireball", "火球术", 20, true),
                createAction("teleport", "传送", 25, true)
            ]
        "rogue":
            availableActions = [
                createAction("backstab", "背刺", 0, true),
                createAction("stealth", "潜行", 15, false),
                createAction("poisonDart", "毒镖", 10, true)
            ]
    
    actions.actions = availableActions

func createAction(name: String, displayName: String, manaCost: int, requiresTarget: bool) -> Action:
    var action = Action.new()
    action.name = name
    action.displayName = displayName
    action.manaCost = manaCost
    action.requiresTarget = requiresTarget
    return action
```

### 多人游戏支持
```gdscript
# 支持多玩家的行动控制
class_name MultiplayerActionControl
extends ActionControlComponent

@export var playerID: int = 0

func _input(event: InputEvent) -> void:
    # 只处理当前玩家的输入
    if GameState.currentPlayerID != playerID:
        return
    
    super._input(event)

func _unhandled_input(event: InputEvent) -> void:
    # 只处理当前玩家的输入
    if GameState.currentPlayerID != playerID:
        return
    
    super._unhandled_input(event)

func onActionsComponent_didRequestTarget(action: Action, source: Entity) -> void:
    # 只为当前玩家创建目标选择
    if source == parentEntity and GameState.currentPlayerID == playerID:
        super.onActionsComponent_didRequestTarget(action, source)
```

### 行动队列系统
```gdscript
# 支持行动队列的控制组件
class_name QueuedActionControl
extends ActionControlComponent

@export var maxQueueSize: int = 3
var actionQueue: Array[String] = []
var isExecutingQueue: bool = false

func _input(event: InputEvent) -> void:
    if not isEnabled or not event is InputEventAction: 
        return
    
    var eventAction: InputEventAction = event as InputEventAction
    var eventName: StringName = eventAction.action
    
    if not eventName.to_lower().begins_with(GlobalInput.Actions.specialActionPrefix.to_lower()): 
        return
    
    var actionName: StringName = eventName.trim_prefix(GlobalInput.Actions.specialActionPrefix)
    
    # 添加到队列而不是立即执行
    queueAction(actionName)

func queueAction(actionName: String):
    if actionQueue.size() >= maxQueueSize:
        print("行动队列已满!")
        return
    
    actionQueue.append(actionName)
    print("行动 ", actionName, " 已加入队列")
    
    # 如果当前没有执行队列，开始执行
    if not isExecutingQueue:
        executeQueue()

func executeQueue():
    if actionQueue.is_empty():
        isExecutingQueue = false
        return
    
    isExecutingQueue = true
    var nextAction = actionQueue.pop_front()
    
    if actionsComponent:
        # 连接行动完成信号
        if not actionsComponent.didPerformAction.is_connected(_on_action_completed):
            actionsComponent.didPerformAction.connect(_on_action_completed)
        
        actionsComponent.performAction(nextAction)

func _on_action_completed(action: Action, result: bool):
    # 继续执行队列中的下一个行动
    await get_tree().create_timer(0.5).timeout  # 行动间隔
    executeQueue()
```

## 设计模式

### 命令模式
- **行动执行：** 将玩家输入转换为行动命令
- **解耦输入：** 输入处理与行动执行分离
- **队列支持：** 支持行动队列和延迟执行

### 观察者模式
- **目标请求：** 监听ActionsComponent的目标请求信号
- **事件驱动：** 基于输入事件和信号的响应
- **松耦合：** 组件间通过信号通信

### 工厂模式
- **组件创建：** 动态创建目标选择组件
- **场景加载：** 根据路径实例化不同的目标选择组件
- **扩展性：** 支持自定义目标选择实现

## 技术细节

### 输入事件处理
ActionControlComponent处理两种输入：
1. `_input()` - 处理UI生成的合成InputEventAction
2. `_unhandled_input()` - 处理真实的键盘/手柄输入

### 特殊行动前缀
```gdscript
var actionName: StringName = eventName.trim_prefix(GlobalInput.Actions.specialActionPrefix)
```

### 目标选择流程
```gdscript
ActionsComponent.performAction() 
→ 发现需要目标 
→ 发出didRequestTarget信号 
→ ActionControlComponent接收 
→ 创建目标选择组件
```

### 输入优先级
使用`set_input_as_handled()`确保输入不被重复处理。

## 注意事项

1. **组件依赖：** 必须有ActionsComponent才能工作
2. **输入映射：** 需要在项目设置中配置相应的输入动作
3. **目标选择：** 目标选择组件路径必须正确
4. **性能考虑：** isEnabled影响输入处理的启用状态
5. **多人游戏：** 注意区分不同玩家的输入

## 输入映射约定

在项目设置的Input Map中，行动快捷键应该遵循以下命名约定：
- `specialAction_` + 行动名称
- 例如：`specialAction_fireball`、`specialAction_heal`

## 相关组件
- `ActionsComponent` - 行动管理，执行具体行动逻辑
- `ActionTargetingComponentBase` - 目标选择基类
- `ActionButtons` - UI按钮组件，生成输入事件
- `InputComponent` - 基础输入处理组件 