# CollectorComponent API

## 概述
`CollectorComponent` 当与`CollectibleComponent`碰撞时执行收集逻辑。负责检查收集条件、处理收集流程，并执行收集物品的载荷效果，如添加组件、修改统计数据或执行自定义脚本。

**继承链**：`Component` → `CollectorComponent`

## 主要特性
- 📦 **自动收集**：与CollectibleComponent碰撞时自动触发
- ✅ **条件检查**：支持自定义收集条件
- 🎯 **载荷执行**：执行收集物品的各种效果
- 📡 **信号通信**：提供丰富的收集事件信号
- 🔧 **可扩展**：支持子类自定义收集逻辑

## 信号
- **didCollideCollectible**(collectibleComponent: CollectibleComponent)：与收集物碰撞时发出
- **didCollect**(collectibleComponent: CollectibleComponent, payload: Variant, result: Variant)：成功收集时发出

## 主要方法

### 收集处理
- **handleCollection**(collectibleComponent: CollectibleComponent) → `Variant`：处理收集流程
- **collect**(collectibleComponent: CollectibleComponent) → `Variant`：执行收集操作
- **checkCollectionConditions**(collectibleComponent: CollectibleComponent) → `bool`：检查收集条件（可重写）

### 碰撞处理
- **onAreaEntered**(area: Area2D) → `void`：Area2D进入时的处理

## 使用示例

### 基础收集器
```gdscript
# 简单的收集器设置
@export var collector: CollectorComponent

func _ready():
    # 连接收集事件
    collector.didCollideCollectible.connect(_on_collide_collectible)
    collector.didCollect.connect(_on_collect_item)

func _on_collide_collectible(collectible: CollectibleComponent):
    print(f"检测到收集物: {collectible.name}")

func _on_collect_item(collectible: CollectibleComponent, payload: Variant, result: Variant):
    print(f"成功收集: {collectible.name}")
    # 播放收集音效
    audio_player.play()
```

### 条件收集器
```gdscript
# 带收集条件的收集器
extends CollectorComponent

@export var inventory_component: InventoryComponent
@export var stats_component: StatsComponent

func checkCollectionConditions(collectible_component: CollectibleComponent) -> bool:
    # 首先调用父类检查
    if not super.checkCollectionConditions(collectible_component):
        return false
    
    # 检查背包空间
    if inventory_component:
        if not inventory_component.hasSpace():
            print("背包已满！")
            return false
    
    # 检查特定物品类型的条件
    if collectible_component.has_meta("item_type"):
        var item_type = collectible_component.get_meta("item_type")
        
        match item_type:
            "health_potion":
                # 生命值满时不收集生命药水
                var health = stats_component.getStat("health")
                var max_health = stats_component.getStat("max_health")
                return health.value < max_health.value
            
            "key":
                # 已有钥匙时不重复收集
                return not inventory_component.hasItemOfType("key")
            
            "ammo":
                # 弹药满时不收集
                var ammo = stats_component.getStat("ammo")
                var max_ammo = stats_component.getStat("max_ammo")
                return ammo.value < max_ammo.value
    
    return true
```

### 智能收集器
```gdscript
# 智能收集优先级系统
extends CollectorComponent

@export var collection_priorities: Dictionary = {
    "health_potion": 10,
    "key": 9,
    "weapon": 8,
    "ammo": 7,
    "coin": 5,
    "common_item": 1
}

var pending_collections: Array[CollectibleComponent] = []
var collection_cooldown: float = 0.5
var last_collection_time: float = 0.0

func handleCollection(collectible_component: CollectibleComponent) -> Variant:
    # 添加到待收集队列
    if collectible_component not in pending_collections:
        pending_collections.append(collectible_component)
    
    # 处理收集队列
    process_collection_queue()
    return true

func process_collection_queue():
    if Time.get_time() - last_collection_time < collection_cooldown:
        return
    
    if pending_collections.is_empty():
        return
    
    # 按优先级排序
    pending_collections.sort_custom(func(a, b): 
        return get_item_priority(a) > get_item_priority(b)
    )
    
    # 收集最高优先级的物品
    var highest_priority = pending_collections[0]
    pending_collections.erase(highest_priority)
    
    if is_instance_valid(highest_priority):
        super.handleCollection(highest_priority)
        last_collection_time = Time.get_time()

func get_item_priority(collectible: CollectibleComponent) -> int:
    if collectible.has_meta("item_type"):
        var item_type = collectible.get_meta("item_type")
        return collection_priorities.get(item_type, 0)
    return 0
```

### 专门收集器
```gdscript
# 金币专门收集器
extends CollectorComponent

@export var coin_value_multiplier: float = 1.0
@export var collection_range: float = 64.0

func checkCollectionConditions(collectible_component: CollectibleComponent) -> bool:
    if not super.checkCollectionConditions(collectible_component):
        return false
    
    # 只收集金币
    if not collectible_component.has_meta("item_type"):
        return false
    
    return collectible_component.get_meta("item_type") == "coin"

func collect(collectible_component: CollectibleComponent) -> Variant:
    # 应用金币倍数
    if collectible_component.payload and collectible_component.payload is StatPayload:
        var stat_payload = collectible_component.payload as StatPayload
        if stat_payload.statName == "gold":
            stat_payload.amount *= coin_value_multiplier
    
    return super.collect(collectible_component)

func _ready():
    super._ready()
    # 扩大收集范围
    var collision_shape = get_node("CollisionShape2D") as CollisionShape2D
    if collision_shape and collision_shape.shape is CircleShape2D:
        var circle = collision_shape.shape as CircleShape2D
        circle.radius = collection_range
```

### 收集特效管理
```gdscript
# 带视觉特效的收集器
extends CollectorComponent

@export var collection_effects: Dictionary = {}
@export var collection_sounds: Dictionary = {}

func _ready():
    super._ready()
    setup_effects()

func setup_effects():
    # 配置不同类型物品的特效
    collection_effects["health_potion"] = preload("res://effects/HealthCollectEffect.tscn")
    collection_effects["coin"] = preload("res://effects/CoinCollectEffect.tscn")
    collection_effects["weapon"] = preload("res://effects/WeaponCollectEffect.tscn")
    
    # 配置音效
    collection_sounds["health_potion"] = preload("res://audio/health_pickup.ogg")
    collection_sounds["coin"] = preload("res://audio/coin_pickup.ogg")
    collection_sounds["weapon"] = preload("res://audio/weapon_pickup.ogg")

func collect(collectible_component: CollectibleComponent) -> Variant:
    var result = super.collect(collectible_component)
    
    if result:
        play_collection_effects(collectible_component)
    
    return result

func play_collection_effects(collectible_component: CollectibleComponent):
    var item_type = collectible_component.get_meta("item_type", "default")
    
    # 播放视觉特效
    if item_type in collection_effects:
        var effect_scene = collection_effects[item_type]
        var effect = effect_scene.instantiate()
        get_tree().current_scene.add_child(effect)
        effect.global_position = collectible_component.global_position
    
    # 播放音效
    if item_type in collection_sounds:
        var audio_player = AudioStreamPlayer2D.new()
        audio_player.stream = collection_sounds[item_type]
        add_child(audio_player)
        audio_player.play()
        
        # 音效播放完后删除
        audio_player.finished.connect(audio_player.queue_free)
```

## 设计模式

### 收集器管理器
```gdscript
# 全局收集器管理系统
class_name CollectionManager
extends Node

var active_collectors: Array[CollectorComponent] = []
var collection_statistics: Dictionary = {}

func register_collector(collector: CollectorComponent):
    active_collectors.append(collector)
    collector.didCollect.connect(_on_item_collected)

func _on_item_collected(collectible: CollectibleComponent, payload: Variant, result: Variant):
    # 统计收集数据
    var item_type = collectible.get_meta("item_type", "unknown")
    collection_statistics[item_type] = collection_statistics.get(item_type, 0) + 1

func get_collection_stats() -> Dictionary:
    return collection_statistics.duplicate()

func pause_all_collection():
    for collector in active_collectors:
        collector.isEnabled = false

func resume_all_collection():
    for collector in active_collectors:
        collector.isEnabled = true
```

### 收集条件系统
```gdscript
# 可配置的收集条件系统
class_name CollectionConditionSystem
extends Node

var global_conditions: Array[Callable] = []
var item_specific_conditions: Dictionary = {}

func add_global_condition(condition: Callable):
    global_conditions.append(condition)

func add_item_condition(item_type: String, condition: Callable):
    if item_type not in item_specific_conditions:
        item_specific_conditions[item_type] = []
    item_specific_conditions[item_type].append(condition)

func check_conditions(collector: CollectorComponent, collectible: CollectibleComponent) -> bool:
    # 检查全局条件
    for condition in global_conditions:
        if not condition.call(collector, collectible):
            return false
    
    # 检查物品特定条件
    var item_type = collectible.get_meta("item_type", "")
    if item_type in item_specific_conditions:
        for condition in item_specific_conditions[item_type]:
            if not condition.call(collector, collectible):
                return false
    
    return true
```

## 技术细节

### 收集流程
1. onAreaEntered()检测碰撞
2. handleCollection()处理收集
3. checkCollectionConditions()验证条件
4. collect()执行载荷
5. 发出相应信号

### 错误处理
- 检查CollectibleComponent有效性
- 处理缺失payload的情况
- 验证载荷执行结果

### 性能优化
- 仅在有效碰撞时处理
- 避免重复检查相同物品
- 缓存条件检查结果

## 注意事项

### 碰撞检测
- 确保CollectorComponent有正确的Area2D配置
- 避免与非CollectibleComponent的无效碰撞
- 处理快速移动时的碰撞丢失

### 条件设计
- 保持checkCollectionConditions()轻量级
- 避免复杂的异步操作
- 考虑条件的优先级和依赖关系

### 载荷执行
- 验证载荷的有效性
- 处理载荷执行失败的情况
- 确保载荷效果的原子性

## 相关组件
- [`CollectibleComponent`](../Objects/CollectibleComponent.md) - 可收集物品
- [`InventoryComponent`](../Gameplay/InventoryComponent.md) - 物品容器
- [`StatsComponent`](../Data/StatsComponent.md) - 统计数据管理 