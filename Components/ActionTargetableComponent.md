# ActionTargetableComponent API

## 概述
`ActionTargetableComponent` 允许实体被玩家或其他角色的`ActionTargetingComponentBase`选中作为目标，用于特殊技能如"火球术"或简单命令如"检查"等需要选择目标的行动。

**继承链**：`AreaComponentBase` → `ActionTargetableComponent`

**状态**：🧪 **实验性** - API可能会发生变化

## 依赖要求
- **Node2D**：组件必须是Node2D以接收鼠标事件

## 主要特性
- 🎯 **目标选择**：可被ActionTargetingComponent选中
- 🖱️ **鼠标交互**：支持鼠标悬停高亮
- ✅ **条件检查**：可自定义选择条件
- 🏷️ **分组管理**：自动加入targetables组
- 🎨 **视觉反馈**：内置高亮效果

## 导出属性

### 视觉配置
- **shouldAlwaysHighlightOnMouseHover** (`bool = false`)：不被瞄准时是否也高亮
  - 设置时自动更新鼠标悬停状态

### 控制配置
- **isEnabled** (`bool = true`)：是否启用目标选择

## 信号
- **wasChosen**：被选中时发出

## 主要方法

### 目标选择接口
- **requestToChoose**() → `bool`：请求选择此目标
- **checkConditions**() → `bool`：检查选择条件（可重写）

### 高亮控制
- **setHighlight**(highlight: bool = true) → `void`：设置高亮状态
- **updateMouseHover**() → `void`：更新鼠标悬停设置

## 使用示例

### 基础目标选择
```gdscript
# 简单的可选择目标
@export var targetable: ActionTargetableComponent

func _ready():
    targetable.wasChosen.connect(_on_target_chosen)
    targetable.isEnabled = true

func _on_target_chosen():
    print("这个实体被选中了！")
    # 执行被选中时的逻辑
    show_selection_effect()
```

### 条件目标选择
```gdscript
# 带条件的目标选择
extends ActionTargetableComponent

@export var health_component: HealthComponent
@export var faction_component: FactionComponent

func checkConditions() -> bool:
    # 只有活着的敌人才能被选中
    if not super.checkConditions():
        return false
    
    if health_component and health_component.currentHealth <= 0:
        return false
    
    if faction_component and faction_component.faction == "ally":
        return false
    
    return true
```

### 治疗目标系统
```gdscript
# 治疗法术的目标选择
extends ActionTargetableComponent

@export var health_component: HealthComponent
@export var faction_component: FactionComponent

func checkConditions() -> bool:
    if not super.checkConditions():
        return false
    
    # 只能对友军使用治疗
    if faction_component.faction != "ally":
        return false
    
    # 只能对受伤的单位使用
    if health_component.currentHealth >= health_component.maxHealth:
        return false
    
    return true

func _ready():
    super._ready()
    wasChosen.connect(_on_heal_target_chosen)

func _on_heal_target_chosen():
    # 播放治疗准备效果
    show_heal_indicator()
```

### 交互目标系统
```gdscript
# 检查/交互目标
extends ActionTargetableComponent

@export var interaction_component: InteractionComponent
@export var inventory_component: InventoryComponent

func checkConditions() -> bool:
    if not super.checkConditions():
        return false
    
    # 检查是否在交互范围内
    var player = get_tree().get_first_node_in_group("player")
    if player:
        var distance = global_position.distance_to(player.global_position)
        return distance <= 100.0  # 交互距离
    
    return false

func _ready():
    super._ready()
    wasChosen.connect(_on_examine_chosen)

func _on_examine_chosen():
    # 显示检查信息
    show_examine_info()

func show_examine_info():
    var info = "这是一个神秘的物品。"
    
    if inventory_component:
        var items = inventory_component.getAllItems()
        info += f"\n包含 {items.size()} 个物品。"
    
    # 显示信息UI
    GlobalUI.show_message(info)
```

### 自定义高亮效果
```gdscript
# 自定义高亮视觉效果
extends ActionTargetableComponent

@export var highlight_color: Color = Color.YELLOW
@export var highlight_intensity: float = 1.5

func setHighlight(highlight: bool = true):
    # 不使用默认的modulate效果
    if highlight:
        show_custom_highlight()
    else:
        hide_custom_highlight()

func show_custom_highlight():
    # 创建高亮轮廓
    var outline_shader = preload("res://shaders/Outline.gdshader")
    var sprite = parentEntity.get_node("Sprite2D") as Sprite2D
    if sprite:
        sprite.material = ShaderMaterial.new()
        sprite.material.shader = outline_shader
        sprite.material.set_shader_parameter("outline_color", highlight_color)
        sprite.material.set_shader_parameter("outline_width", 2.0)

func hide_custom_highlight():
    var sprite = parentEntity.get_node("Sprite2D") as Sprite2D
    if sprite:
        sprite.material = null
```

### 多目标选择器
```gdscript
# 管理多个可选目标
class_name MultiTargetSelector
extends Node

@export var target_action: Action
var available_targets: Array[ActionTargetableComponent] = []
var selected_targets: Array[ActionTargetableComponent] = []
var max_targets: int = 3

func _ready():
    # 收集所有可选目标
    refresh_available_targets()

func refresh_available_targets():
    available_targets.clear()
    var all_targetables = get_tree().get_nodes_in_group(Global.Groups.targetables)
    
    for entity in all_targetables:
        var targetable = entity.findFirstComponentSubclass(ActionTargetableComponent)
        if targetable and targetable.checkConditions():
            available_targets.append(targetable)

func select_target(targetable: ActionTargetableComponent):
    if targetable in available_targets and targetable not in selected_targets:
        if selected_targets.size() < max_targets:
            selected_targets.append(targetable)
            targetable.setHighlight(true)
            targetable.wasChosen.emit()

func deselect_target(targetable: ActionTargetableComponent):
    if targetable in selected_targets:
        selected_targets.erase(targetable)
        targetable.setHighlight(false)

func execute_action():
    if selected_targets.size() > 0:
        for target in selected_targets:
            target_action.execute_on_target(target.parentEntity)
        clear_selection()

func clear_selection():
    for target in selected_targets:
        target.setHighlight(false)
    selected_targets.clear()
```

## 设计模式

### 目标管理器
```gdscript
# 全局目标管理系统
class_name TargetManager
extends Node

var registered_targets: Array[ActionTargetableComponent] = []

func register_target(target: ActionTargetableComponent):
    registered_targets.append(target)
    target.tree_exited.connect(unregister_target.bind(target))

func unregister_target(target: ActionTargetableComponent):
    registered_targets.erase(target)

func get_targets_by_condition(condition: Callable) -> Array[ActionTargetableComponent]:
    var filtered_targets: Array[ActionTargetableComponent] = []
    for target in registered_targets:
        if target.isEnabled and condition.call(target):
            filtered_targets.append(target)
    return filtered_targets

func get_nearest_target(position: Vector2) -> ActionTargetableComponent:
    var nearest: ActionTargetableComponent = null
    var nearest_distance: float = INF
    
    for target in registered_targets:
        if target.isEnabled and target.checkConditions():
            var distance = position.distance_to(target.global_position)
            if distance < nearest_distance:
                nearest_distance = distance
                nearest = target
    
    return nearest
```

## 技术细节

### 分组管理
- 自动加入Global.Groups.targetables组
- 组件移除时自动退出分组
- 实体也会加入分组以便查找

### 鼠标事件处理
- 要求组件是Node2D类型
- 使用input_pickable控制鼠标交互
- 自动连接/断开鼠标信号

### Area2D配置
- monitoring设为false（不需要监控其他区域）
- 仅需要被其他区域监控（monitorable）

## 注意事项

### 实验性功能
- API可能在未来版本中更改
- 某些功能可能不稳定
- 建议在使用前测试兼容性

### 性能考虑
- 大量目标时优化条件检查
- 避免每帧调用checkConditions()
- 考虑使用对象池管理目标

### 条件设计
- checkConditions()应快速执行
- 避免复杂的计算和查找
- 考虑缓存条件结果

## 相关组件
- [`ActionTargetingComponentBase`](./ActionTargetingComponentBase.md) - 目标瞄准基类
- [`InteractionComponent`](./InteractionComponent.md) - 交互系统
- [`AreaComponentBase`](./AreaComponentBase.md) - Area2D基类 