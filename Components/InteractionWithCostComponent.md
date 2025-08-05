# InteractionWithCostComponent API

## 概述
`InteractionWithCostComponent` 是`InteractionComponent`的子类，增加了冷却时间和统计数据消耗功能。适用于需要资源消耗的交互，如使用法力值开门、消耗体力砍树等。

**继承链**：`InteractionComponent` → `InteractionWithCostComponent`

## 主要特性
- 💰 **资源消耗**：支持基于Stat的交互成本
- ⏰ **冷却时间**：内置Timer管理交互冷却
- ✅ **条件检查**：在消耗前验证资源是否充足
- 🎨 **状态显示**：自动更新交互指示器状态
- 🔄 **自动扣除**：成功交互后自动扣除资源

## 导出属性
- **cost** (`StatDependentResourceBase`)：交互所需的统计数据成本

## 子节点
- **CooldownTimer** (`Timer`)：冷却时间计时器

## 主要方法

### 交互接口（重写）
- **requestToInteract**(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) → `bool`：检查成本和条件
- **performInteraction**(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) → `Variant`：执行交互并扣除成本

### 标签更新
- **updateLabel**() → `void`：更新交互指示器状态
- **onCooldownTimer_timeout**() → `void`：冷却完成时的回调

## 使用示例

### 基础有成本交互
```gdscript
# 需要法力值的魔法门
@export var magic_door: InteractionWithCostComponent

func _ready():
    # 设置法力值成本
    var mana_cost = StatDependentResourceBase.new()
    mana_cost.statName = "mana"
    mana_cost.amount = 10.0
    magic_door.cost = mana_cost
    
    # 设置交互载荷
    var door_payload = ScriptPayload.new()
    door_payload.script = preload("res://scripts/OpenMagicDoor.gd")
    magic_door.payload = door_payload
    
    magic_door.didDenyInteraction.connect(_on_interaction_denied)

func _on_interaction_denied(interactor: Entity):
    print("法力值不足，无法开启魔法门！")
```

### 体力消耗系统
```gdscript
# 需要体力的伐木交互
@export var tree_chopping: InteractionWithCostComponent
@export var stamina_cost: float = 15.0

func _ready():
    setup_stamina_cost()
    tree_chopping.didPerformInteraction.connect(_on_tree_chopped)

func setup_stamina_cost():
    var stamina_cost_resource = StatDependentResourceBase.new()
    stamina_cost_resource.statName = "stamina"
    stamina_cost_resource.amount = stamina_cost
    tree_chopping.cost = stamina_cost_resource
    
    # 设置冷却时间
    var cooldown_timer = tree_chopping.get_node("CooldownTimer") as Timer
    cooldown_timer.wait_time = 3.0  # 3秒冷却

func _on_tree_chopped(interactor: Entity, interaction_control: InteractionControlComponent):
    print("成功砍伐了树木！")
    
    # 产生木材
    var wood_item = preload("res://items/Wood.tres")
    var inventory = interactor.findFirstComponentSubclass(InventoryComponent)
    if inventory:
        inventory.addItem(wood_item, 1)
```

### 多资源消耗
```gdscript
# 需要多种资源的合成台
extends InteractionWithCostComponent

@export var iron_cost: int = 5
@export var coal_cost: int = 2
@export var gold_cost: int = 10

func requestToInteract(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> bool:
    if not isEnabled or not is_zero_approx(cooldownTimer.time_left):
        return false
    
    var stats_component = interactor_entity.getComponent(StatsComponent) as StatsComponent
    if not stats_component:
        return false
    
    # 检查所有资源
    var iron_stat = stats_component.getStat("iron")
    var coal_stat = stats_component.getStat("coal")  
    var gold_stat = stats_component.getStat("gold")
    
    var has_resources = (iron_stat.value >= iron_cost and 
                        coal_stat.value >= coal_cost and 
                        gold_stat.value >= gold_cost)
    
    if has_resources and checkInteractionConditions(interactor_entity, interaction_control_component):
        return true
    else:
        didDenyInteraction.emit(interactor_entity)
        return false

func performInteraction(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> Variant:
    if not isEnabled or not is_zero_approx(cooldownTimer.time_left):
        return false
    
    var stats_component = interactor_entity.getComponent(StatsComponent) as StatsComponent
    
    # 扣除所有资源
    stats_component.getStat("iron").value -= iron_cost
    stats_component.getStat("coal").value -= coal_cost
    stats_component.getStat("gold").value -= gold_cost
    
    # 执行合成
    var result = super.performInteraction(interactor_entity, interaction_control_component)
    if result:
        cooldownTimer.start()
        create_crafted_item(interactor_entity)
    
    updateLabel()
    return result

func create_crafted_item(interactor: Entity):
    var sword = preload("res://items/IronSword.tres")
    var inventory = interactor.findFirstComponentSubclass(InventoryComponent)
    if inventory:
        inventory.addItem(sword, 1)
```

### 动态成本系统
```gdscript
# 基于玩家等级的动态成本
extends InteractionWithCostComponent

@export var base_cost: float = 10.0
@export var level_multiplier: float = 1.5

func requestToInteract(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> bool:
    # 动态计算成本
    update_dynamic_cost(interactor_entity)
    return super.requestToInteract(interactor_entity, interaction_control_component)

func update_dynamic_cost(interactor: Entity):
    var stats = interactor.findFirstComponentSubclass(StatsComponent)
    if stats:
        var level = stats.getStat("level").value
        var dynamic_cost = base_cost * pow(level_multiplier, level - 1)
        
        if not cost:
            cost = StatDependentResourceBase.new()
            cost.statName = "gold"
        
        cost.amount = dynamic_cost

func updateLabel():
    super.updateLabel()
    
    # 显示当前成本
    if interactionIndicator is Label and cost:
        var base_text = label if not label.is_empty() else "交互"
        if is_zero_approx(cooldownTimer.time_left):
            interactionIndicator.text = f"{base_text} (花费: {int(cost.amount)})"
        else:
            interactionIndicator.text = "冷却中..."
```

### 条件成本系统
```gdscript
# 基于条件的成本系统
extends InteractionWithCostComponent

@export var weather_system: WeatherSystem
@export var time_system: TimeSystem

func checkInteractionConditions(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> bool:
    if not super.checkInteractionConditions(interactor_entity, interaction_control_component):
        return false
    
    # 夜晚时成本翻倍
    if time_system and time_system.is_night:
        cost.amount *= 2
    
    # 下雨时无法使用
    if weather_system and weather_system.current_weather == "rain":
        return false
    
    return true

func performInteraction(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> Variant:
    var result = super.performInteraction(interactor_entity, interaction_control_component)
    
    # 重置成本（如果被修改过）
    if time_system and time_system.is_night:
        cost.amount /= 2
    
    return result
```

### 技能升级系统
```gdscript
# 技能升级交互
extends InteractionWithCostComponent

@export var skill_name: String = "magic"
@export var cost_progression: Array[int] = [10, 25, 50, 100, 200]

func _ready():
    super._ready()
    update_skill_cost()

func update_skill_cost():
    var player = get_tree().get_first_node_in_group("player")
    if not player:
        return
    
    var stats = player.findFirstComponentSubclass(StatsComponent)
    if not stats:
        return
    
    var skill_level = int(stats.getStat(skill_name + "_level").value)
    
    if skill_level < cost_progression.size():
        if not cost:
            cost = StatDependentResourceBase.new()
            cost.statName = "experience"
        cost.amount = cost_progression[skill_level]
        
        # 更新标签
        updateLabel()
    else:
        # 已达到最高等级
        isEnabled = false

func performInteraction(interactor_entity: Entity, interaction_control_component: InteractionControlComponent) -> Variant:
    var result = super.performInteraction(interactor_entity, interaction_control_component)
    
    if result:
        # 提升技能等级
        var stats = interactor_entity.findFirstComponentSubclass(StatsComponent)
        if stats:
            var skill_level_stat = stats.getStat(skill_name + "_level")
            skill_level_stat.value += 1
            
            # 更新下次升级的成本
            update_skill_cost()
            
            # 播放升级效果
            show_level_up_effect()
    
    return result

func show_level_up_effect():
    var effect = preload("res://effects/SkillLevelUp.tscn").instantiate()
    get_tree().current_scene.add_child(effect)
    effect.global_position = global_position
```

## 设计模式

### 成本工厂
```gdscript
# 成本配置工厂
class_name CostFactory
extends RefCounted

static func create_mana_cost(amount: float) -> StatDependentResourceBase:
    var cost = StatDependentResourceBase.new()
    cost.statName = "mana"
    cost.amount = amount
    return cost

static func create_stamina_cost(amount: float) -> StatDependentResourceBase:
    var cost = StatDependentResourceBase.new()
    cost.statName = "stamina"
    cost.amount = amount
    return cost

static func create_gold_cost(amount: float) -> StatDependentResourceBase:
    var cost = StatDependentResourceBase.new()
    cost.statName = "gold"
    cost.amount = amount
    return cost

static func create_item_cost(item_type: String, amount: int) -> ItemCost:
    var cost = ItemCost.new()
    cost.itemType = item_type
    cost.amount = amount
    return cost
```

### 交互管理器
```gdscript
# 成本交互管理器
class_name CostInteractionManager
extends Node

var discount_multiplier: float = 1.0
var global_cooldown_modifier: float = 1.0

func apply_discount(discount: float, duration: float = 0.0):
    discount_multiplier = 1.0 - discount
    
    if duration > 0:
        var timer = Timer.new()
        timer.wait_time = duration
        timer.one_shot = true
        timer.timeout.connect(func(): discount_multiplier = 1.0)
        add_child(timer)
        timer.start()

func modify_all_costs(multiplier: float):
    var interactions = get_tree().get_nodes_in_group("cost_interactions")
    for interaction in interactions:
        if interaction is InteractionWithCostComponent:
            if interaction.cost:
                interaction.cost.amount *= multiplier

func get_effective_cost(original_cost: float) -> float:
    return original_cost * discount_multiplier
```

## 技术细节

### 成本检查流程
1. 验证组件启用状态和冷却时间
2. 获取交互者的StatsComponent
3. 验证cost资源是否充足
4. 调用checkInteractionConditions()
5. 返回检查结果

### 资源扣除
- 在performInteraction()中执行
- 使用StatDependentResourceBase.deductCostFromStat()
- 仅在成功时扣除资源
- 失败时不产生任何消耗

### 标签更新
- 冷却时显示"COOLDOWN"
- 正常时显示配置的标签文本
- 通过self_modulate调整透明度

## 注意事项

### 成本配置
- 确保cost资源正确配置
- 验证statName与StatsComponent匹配
- 考虑成本的合理性和平衡性

### 冷却时间
- 编辑器中启用"Editable Children"来配置Timer
- 冷却期间交互会被拒绝
- 考虑用户反馈和状态指示

### 性能优化
- 避免频繁的成本计算
- 缓存StatsComponent引用
- 优化标签更新频率

## 相关组件
- [`InteractionComponent`](./InteractionComponent.md) - 基础交互组件
- [`StatsComponent`](./StatsComponent.md) - 统计数据管理
- [`StatDependentResourceBase`](../../Resources/StatDependentResourceBase.md) - 统计依赖资源 