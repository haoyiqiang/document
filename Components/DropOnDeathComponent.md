# DropOnDeathComponent API

## 概述
`DropOnDeathComponent` 在父实体"死亡"时创建节点，通过监听`HealthComponent`实现。生成的节点可以是可收集物品（如金币）或装饰性节点（如墓碑精灵）。

**继承链**：`Component` → `DropOnDeathComponent`

## 依赖要求
- **HealthComponent**：必需的生命值组件，监听死亡事件

## 主要特性
- 💀 **死亡检测**：自动监听HealthComponent的死亡事件
- 🎁 **物品生成**：从PackedScene创建掉落物品
- 📍 **位置控制**：支持位置偏移和自定义父节点
- 🎯 **灵活配置**：可生成任何类型的节点
- 📡 **事件通知**：掉落完成时发出信号

## 导出属性

### 生成配置
- **sceneToSpawnOnDeath** (`PackedScene`)：死亡时要生成的场景
- **positionOffset** (`Vector2 = Vector2.ZERO`)：相对实体位置的偏移
- **parentOverrideForSpawnedNode** (`Node2D`)：生成节点的父节点覆盖
  - 如果为null，使用父实体的父节点
- **isEnabled** (`bool = true`)：是否启用掉落功能

## 状态属性
- **parentForSpawnedNode** (`Node2D`)：实际用于生成节点的父节点

## 信号
- **didDrop**(node: Node2D)：成功生成掉落物后发出

## 主要方法
- **drop**() → `Node`：手动执行掉落，返回生成的节点
- **onHealthComponent_healthDidZero**() → `void`：健康值归零时的回调

## 使用示例

### 基础掉落系统
```gdscript
# 设置基本的死亡掉落
@export var drop_component: DropOnDeathComponent
@export var coin_scene: PackedScene

func _ready():
    drop_component.sceneToSpawnOnDeath = coin_scene
    drop_component.positionOffset = Vector2(0, -10)  # 稍微向上偏移
    drop_component.didDrop.connect(_on_item_dropped)

func _on_item_dropped(dropped_node: Node2D):
    print(f"掉落了物品: {dropped_node.name}")
    # 可以添加掉落音效或特效
```

### 多种掉落物系统
```gdscript
# 根据条件掉落不同物品
@export var drop_component: DropOnDeathComponent
@export var coin_scene: PackedScene
@export var gem_scene: PackedScene
@export var rare_item_scene: PackedScene

func _ready():
    # 连接健康组件事件，自定义掉落逻辑
    var health_comp = get_component(HealthComponent)
    health_comp.healthDidZero.connect(_on_custom_death)

func _on_custom_death():
    # 禁用默认掉落，使用自定义逻辑
    drop_component.isEnabled = false
    
    var drop_chance = randf()
    if drop_chance < 0.1:        # 10%几率掉落稀有物品
        drop_component.sceneToSpawnOnDeath = rare_item_scene
    elif drop_chance < 0.4:      # 30%几率掉落宝石
        drop_component.sceneToSpawnOnDeath = gem_scene
    else:                        # 60%几率掉落金币
        drop_component.sceneToSpawnOnDeath = coin_scene
    
    # 手动执行掉落
    drop_component.drop()
```

### 掉落位置管理
```gdscript
# 智能掉落位置系统
@export var drop_component: DropOnDeathComponent
@export var ground_detector: RayCast2D

func _ready():
    drop_component.didDrop.connect(_on_item_dropped)

func _on_item_dropped(dropped_node: Node2D):
    # 确保掉落物在地面上
    adjust_drop_position(dropped_node)

func adjust_drop_position(item: Node2D):
    # 检测地面位置
    ground_detector.global_position = item.global_position
    ground_detector.force_raycast_update()
    
    if ground_detector.is_colliding():
        var ground_point = ground_detector.get_collision_point()
        item.global_position = ground_point + Vector2(0, -5)  # 稍微高于地面
```

### 掉落效果增强
```gdscript
# 添加掉落特效和动画
@export var drop_component: DropOnDeathComponent
@export var drop_effect_scene: PackedScene

func _ready():
    drop_component.didDrop.connect(_on_item_dropped_with_effects)

func _on_item_dropped_with_effects(dropped_node: Node2D):
    # 添加掉落特效
    if drop_effect_scene:
        var effect = drop_effect_scene.instantiate()
        get_tree().current_scene.add_child(effect)
        effect.global_position = dropped_node.global_position
    
    # 添加掉落动画
    animate_drop(dropped_node)
    
    # 播放音效
    AudioManager.play_sound("item_drop")

func animate_drop(item: Node2D):
    # 创建弹跳动画
    var tween = create_tween()
    var start_pos = item.global_position
    var bounce_height = 20
    
    tween.tween_method(
        func(pos): item.global_position = pos,
        start_pos + Vector2(0, -bounce_height),
        start_pos,
        0.3
    ).set_ease(Tween.EASE_OUT).set_trans(Tween.TRANS_BOUNCE)
```

### 条件掉落系统
```gdscript
# 基于条件的掉落系统
@export var drop_component: DropOnDeathComponent
@export var player_stats: PlayerStats

func _ready():
    # 覆盖默认行为
    var health_comp = get_component(HealthComponent)
    health_comp.healthDidZero.disconnect(drop_component.onHealthComponent_healthDidZero)
    health_comp.healthDidZero.connect(_on_conditional_death)

func _on_conditional_death():
    # 检查掉落条件
    if should_drop_items():
        drop_component.drop()

func should_drop_items() -> bool:
    # 基于玩家属性决定是否掉落
    if player_stats.has_buff("increased_drops"):
        return randf() < 0.8  # 80%掉落率
    elif player_stats.has_debuff("reduced_drops"):
        return randf() < 0.2  # 20%掉落率
    else:
        return randf() < 0.5  # 50%掉落率
```

### 掉落池系统
```gdscript
# 加权掉落池系统
class_name DropPool
extends Resource

@export var drops: Array[DropItem] = []

class_name DropItem
extends Resource

@export var scene: PackedScene
@export var weight: float = 1.0
@export var min_level: int = 1

# 使用掉落池
@export var drop_component: DropOnDeathComponent
@export var drop_pool: DropPool
@export var entity_level: int = 1

func _ready():
    var health_comp = get_component(HealthComponent)
    health_comp.healthDidZero.connect(_on_death_with_pool)

func _on_death_with_pool():
    var selected_drop = select_from_pool()
    if selected_drop:
        drop_component.sceneToSpawnOnDeath = selected_drop.scene
        drop_component.drop()

func select_from_pool() -> DropItem:
    var available_drops = []
    var total_weight = 0.0
    
    # 筛选可用的掉落物
    for drop in drop_pool.drops:
        if entity_level >= drop.min_level:
            available_drops.append(drop)
            total_weight += drop.weight
    
    if available_drops.is_empty():
        return null
    
    # 加权随机选择
    var random_value = randf() * total_weight
    var current_weight = 0.0
    
    for drop in available_drops:
        current_weight += drop.weight
        if random_value <= current_weight:
            return drop
    
    return available_drops[-1]  # fallback
```

## 设计模式

### 掉落管理器
```gdscript
# 全局掉落管理系统
class_name DropManager
extends Node

var active_drops: Array[Node2D] = []
var max_drops_on_screen: int = 50

func register_drop(drop_node: Node2D):
    active_drops.append(drop_node)
    
    # 限制屏幕上的掉落物数量
    if active_drops.size() > max_drops_on_screen:
        var oldest_drop = active_drops.pop_front()
        if is_instance_valid(oldest_drop):
            oldest_drop.queue_free()
    
    # 连接清理信号
    if drop_node.has_signal("collected"):
        drop_node.collected.connect(_on_drop_collected.bind(drop_node))

func _on_drop_collected(drop_node: Node2D):
    active_drops.erase(drop_node)

func cleanup_all_drops():
    for drop in active_drops:
        if is_instance_valid(drop):
            drop.queue_free()
    active_drops.clear()
```

### 掉落工厂
```gdscript
# 掉落物创建工厂
class_name DropFactory
extends Node

var drop_scenes: Dictionary[StringName, PackedScene] = {}

func _ready():
    load_drop_scenes()

func load_drop_scenes():
    drop_scenes["coin"] = load("res://items/Coin.tscn")
    drop_scenes["gem"] = load("res://items/Gem.tscn")
    drop_scenes["health_potion"] = load("res://items/HealthPotion.tscn")

func create_drop(drop_type: StringName, position: Vector2, parent: Node2D) -> Node2D:
    if not drop_type in drop_scenes:
        push_error(f"Unknown drop type: {drop_type}")
        return null
    
    var scene = drop_scenes[drop_type]
    var instance = scene.instantiate()
    parent.add_child(instance)
    instance.global_position = position
    
    return instance

func create_random_drop(position: Vector2, parent: Node2D) -> Node2D:
    var drop_types = drop_scenes.keys()
    var random_type = drop_types[randi() % drop_types.size()]
    return create_drop(random_type, position, parent)
```

## 技术细节

### 死亡检测机制
- 监听HealthComponent.healthDidZero信号
- 仅在shouldRemoveEntityOnZero为true时触发
- 支持手动调用drop()方法

### 位置计算
- 使用to_local()转换坐标空间
- 应用positionOffset偏移
- 支持自定义父节点层级

### 节点生成
- 使用SceneManager.addSceneInstance()统一创建
- 自动处理场景实例化失败的情况
- 返回生成的节点引用用于后续操作

## 注意事项

### 性能考虑
- 避免同时生成大量掉落物
- 考虑掉落物的自动清理机制
- 使用对象池优化频繁的创建/销毁

### 内存管理
- 确保生成的节点正确添加到场景树
- 处理场景实例化失败的情况
- 注意掉落物的生命周期管理

### 设计建议
- 使用掉落池系统增加变化性
- 考虑掉落物的视觉和音效反馈
- 实现掉落物的自动收集或过期机制

## 相关组件
- [`HealthComponent`](../Combat/HealthComponent.md) - 生命值管理
- [`CollectibleComponent`](../Objects/CollectibleComponent.md) - 可收集物品
- [`SceneManager`](../../Managers/SceneManager.md) - 场景管理器 