# InteractionControlComponent API

## 概述
`InteractionControlComponent` 是一个交互控制组件，允许玩家通过按钮输入与InteractionComponent进行交互。与自动触发的CollectibleComponent不同，此组件需要主动按键才能触发交互，支持指示器显示和冷却时间管理。

**继承链**：`CooldownComponent` → `InteractionControlComponent`

## 依赖要求
- **组件节点必须是Area2D**：用于检测范围内的InteractionComponent

## 主要特性
- 🎮 **按键交互**：需要主动按键才能触发交互
- 📊 **范围检测**：自动检测范围内的InteractionComponent
- 🔄 **批量交互**：支持同时与多个对象交互
- 💡 **视觉指示**：可配置交互指示器显示
- ⏰ **冷却管理**：继承冷却时间功能防止频繁交互
- 📡 **事件系统**：丰富的信号支持外部响应

## 导出属性

### 交互配置
- **maximumSimultaneousInteractions** (`int = 1`, 范围:1~100)：单次交互的最大对象数量
- **interactionIndicator** (`Node`)：交互指示器节点（Node2D或Control）
- **inputEventName** (`StringName = interact`)：交互输入事件名称

### 控制选项
- **isEnabled** (`bool = true`)：是否启用交互功能
  - 影响Area2D的monitoring和monitorable属性

## 状态属性

### 交互状态
- **interactionsInRange** (`Array[InteractionComponent]`)：范围内的交互组件数组
- **haveInteracionsInRange** (`bool`)：是否有交互对象在范围内
- **selfAsArea** (`Area2D`)：组件作为Area2D的引用

## 信号

### 区域事件
- **didEnterInteractionArea(entity: Entity, interactionComponent: InteractionComponent)**：进入交互区域
- **didExitInteractionArea(entity: Entity, interactionComponent: InteractionComponent)**：离开交互区域

### 交互事件
- **willPerformInteraction(entity: Entity, interactionComponent: InteractionComponent)**：即将执行交互
- **didPerformInteraction(result: Variant)**：完成交互（返回结果）

## 主要方法

### 交互控制
- **interact()** → `int`：执行交互，返回成功的交互次数
- **updateIndicator()** → `void`：更新指示器显示状态

### 区域检测
- **onArea_entered(area: Area2D)** → `void`：进入区域处理
- **onArea_exited(area: Area2D)** → `void`：离开区域处理

### 冷却重写
- **startCooldown(overrideTime: float)** → `void`：开始冷却并调整指示器透明度
- **finishCooldown()** → `void`：完成冷却并恢复指示器透明度

## 使用示例

### 基础交互设置
```gdscript
# 设置简单的交互控制
@export var interaction_control: InteractionControlComponent
@export var interaction_icon: Sprite2D

func _ready():
    interaction_control.interactionIndicator = interaction_icon
    interaction_control.maximumSimultaneousInteractions = 1
    interaction_control.inputEventName = "interact"
```

### 商店交互系统
```gdscript
# 商店NPC交互
@export var shop_interaction: InteractionControlComponent
@export var shop_indicator: Control

func _ready():
    shop_interaction.interactionIndicator = shop_indicator
    shop_interaction.didPerformInteraction.connect(on_shop_interaction)

func on_shop_interaction(result: Variant):
    if result:
        # 打开商店界面
        GlobalUI.showShopDialog()
    else:
        GlobalUI.showMessage("商店暂时关闭")
```

### 多对象交互
```gdscript
# 允许同时拾取多个物品
@export var multi_interaction: InteractionControlComponent

func setup_multi_collection():
    multi_interaction.maximumSimultaneousInteractions = 5  # 最多同时拾取5个
    multi_interaction.didPerformInteraction.connect(on_multi_collect)

func on_multi_collect(result: Variant):
    if typeof(result) == TYPE_INT and result > 0:
        GlobalUI.showMessage(str("拾取了", result, "个物品"))
```

### 条件交互控制
```gdscript
# 基于条件的交互可用性
@export var conditional_interaction: InteractionControlComponent
@export var player_stats: StatsComponent

func _ready():
    conditional_interaction.didEnterInteractionArea.connect(check_interaction_requirements)

func check_interaction_requirements(entity: Entity, interaction_component: InteractionComponent):
    # 检查玩家等级是否足够
    if player_stats.level.value < 5:
        conditional_interaction.isEnabled = false
        GlobalUI.showMessage("等级不足，无法使用此物品")
    else:
        conditional_interaction.isEnabled = true
```

### 智能指示器管理
```gdscript
# 动态指示器显示
@export var smart_interaction: InteractionControlComponent
@export var default_indicator: Sprite2D
@export var quest_indicator: Sprite2D
@export var shop_indicator: Sprite2D

func _ready():
    smart_interaction.didEnterInteractionArea.connect(update_indicator_type)

func update_indicator_type(entity: Entity, interaction_component: InteractionComponent):
    # 根据交互类型选择不同指示器
    var interaction_type = interaction_component.get_meta("type", "default")
    
    match interaction_type:
        "quest":
            smart_interaction.interactionIndicator = quest_indicator
        "shop":
            smart_interaction.interactionIndicator = shop_indicator
        _:
            smart_interaction.interactionIndicator = default_indicator
    
    smart_interaction.updateIndicator()
```

## 设计模式

### 交互管理器
```gdscript
# 统一管理多种交互类型
class_name InteractionManager
extends Node

@export var item_interaction: InteractionControlComponent
@export var npc_interaction: InteractionControlComponent
@export var environment_interaction: InteractionControlComponent

func _ready():
    setup_interaction_handlers()

func setup_interaction_handlers():
    # 物品交互
    item_interaction.inputEventName = "interact"
    item_interaction.didPerformInteraction.connect(handle_item_interaction)
    
    # NPC对话
    npc_interaction.inputEventName = "talk"
    npc_interaction.didPerformInteraction.connect(handle_npc_interaction)
    
    # 环境交互
    environment_interaction.inputEventName = "use"
    environment_interaction.didPerformInteraction.connect(handle_environment_interaction)

func handle_item_interaction(result: Variant):
    # 处理物品拾取结果
    pass

func handle_npc_interaction(result: Variant):
    # 处理NPC对话结果
    pass

func handle_environment_interaction(result: Variant):
    # 处理环境交互结果
    pass
```

### 优先级交互系统
```gdscript
# 根据距离和重要性排序交互对象
@export var priority_interaction: InteractionControlComponent

func _ready():
    # 重写交互逻辑实现优先级
    priority_interaction.didEnterInteractionArea.connect(sort_interactions_by_priority)

func sort_interactions_by_priority(_entity: Entity, _interaction: InteractionComponent):
    # 按距离和优先级排序
    priority_interaction.interactionsInRange.sort_custom(compare_interaction_priority)

func compare_interaction_priority(a: InteractionComponent, b: InteractionComponent) -> bool:
    # 先按优先级排序
    var priority_a = a.get_meta("priority", 0)
    var priority_b = b.get_meta("priority", 0)
    
    if priority_a != priority_b:
        return priority_a > priority_b
    
    # 再按距离排序
    var distance_a = global_position.distance_to(a.global_position)
    var distance_b = global_position.distance_to(b.global_position)
    return distance_a < distance_b
```

### 状态驱动交互
```gdscript
# 基于游戏状态的交互行为
@export var context_interaction: InteractionControlComponent
@export var game_state: GameState

func _ready():
    context_interaction.willPerformInteraction.connect(validate_interaction_context)

func validate_interaction_context(entity: Entity, interaction_component: InteractionComponent):
    # 根据游戏状态验证交互有效性
    match game_state.current_mode:
        GameState.Mode.COMBAT:
            # 战斗中只允许紧急物品交互
            if not interaction_component.has_meta("emergency"):
                GlobalUI.showMessage("战斗中无法进行此操作")
                return false
        
        GameState.Mode.STEALTH:
            # 潜行中禁用有声音的交互
            if interaction_component.has_meta("makes_noise"):
                GlobalUI.showMessage("会发出声音，暴露位置")
                return false
    
    return true
```

## 技术细节

### Area2D集成
- 组件必须是Area2D节点才能工作
- 使用`monitoring`和`monitorable`控制检测状态
- 通过`area_entered`和`area_exited`信号检测交互对象

### 冷却时间集成
- 继承CooldownComponent的冷却功能
- 成功交互后自动启动冷却
- 冷却期间指示器透明度降低为0.1

### 批量交互处理
- 支持`maximumSimultaneousInteractions`限制同时交互数量
- 按数组顺序处理交互对象
- 统计成功交互次数决定是否启动冷却

## 注意事项

### 组件要求
- **必须**是Area2D节点才能正常工作
- 需要正确设置碰撞形状和图层
- 确保与InteractionComponent的图层匹配

### 性能考虑
- 大范围检测可能影响性能
- 合理设置交互范围和数量限制
- 避免频繁的指示器更新

### 与CollectibleComponent的区别
- InteractionControlComponent：需要主动按键交互
- CollectibleComponent：碰撞自动触发
- 根据游戏设计需求选择合适的组件

## 相关组件
- [`InteractionComponent`](../Objects/InteractionComponent.md) - 交互对象组件
- [`CollectibleComponent`](../Objects/CollectibleComponent.md) - 可拾取物品组件
- [`CooldownComponent`](../Gameplay/CooldownComponent.md) - 冷却时间基础组件
- [`InputComponent`](../Control/InputComponent.md) - 输入控制组件 