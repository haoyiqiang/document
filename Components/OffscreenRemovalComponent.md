# OffscreenRemovalComponent API

## 概述
`OffscreenRemovalComponent` 当父实体的`VisibleOnScreenNotifier2D`边界矩形离开屏幕时，自动调用`Entity.requestDeletion()`删除实体。支持可选的延迟删除，允许实体有时间返回屏幕。

**继承链**：`Component` → `OffscreenRemovalComponent`

**用途**：自动清理离开屏幕的实体，如子弹、特效、敌人等，防止内存泄漏。

## 依赖要求
- **VisibleOnScreenNotifier2D**：实体需要有VisibleOnScreenNotifier2D子节点

## 主要特性
- 🗑️ **自动清理**：离开屏幕时自动删除实体
- ⏱️ **延迟删除**：可配置延迟时间，允许暂时离屏
- 👁️ **可见性检查**：可忽略隐藏节点的离屏状态
- 🎯 **性能优化**：防止大量离屏实体占用内存
- 🔧 **灵活控制**：支持启用/禁用删除功能

## 导出属性

### 删除配置
- **removalDelay** (`float = 0`)：离屏后的删除延迟时间（秒）
  - 0表示立即删除
  - 大于0允许实体返回屏幕
- **ignoreIfHidden** (`bool = true`)：是否忽略隐藏节点
  - true：visible = false的节点不会被删除
  - 防止闪烁动画等导致的意外删除
- **isEnabled** (`bool = true`)：是否启用离屏删除功能

## 主要方法
- **onScreenExited**() → `void`：屏幕退出时的回调处理

## 使用示例

### 基础离屏清理
```gdscript
# 设置基本的离屏删除
@export var offscreen_removal: OffscreenRemovalComponent

func _ready():
    # 立即删除离屏实体
    offscreen_removal.removalDelay = 0.0
    offscreen_removal.isEnabled = true
    
    # 连接VisibleOnScreenNotifier2D信号
    var notifier = get_node("VisibleOnScreenNotifier2D")
    notifier.screen_exited.connect(offscreen_removal.onScreenExited)
```

### 延迟删除系统
```gdscript
# 给实体一些时间返回屏幕
@export var offscreen_removal: OffscreenRemovalComponent

func _ready():
    # 离屏2秒后删除
    offscreen_removal.removalDelay = 2.0
    offscreen_removal.ignoreIfHidden = true
```

### 条件性删除
```gdscript
# 基于实体状态的条件删除
@export var offscreen_removal: OffscreenRemovalComponent
@export var health_component: HealthComponent

func _ready():
    # 只有受伤的敌人才会被离屏删除
    var notifier = get_node("VisibleOnScreenNotifier2D")
    notifier.screen_exited.connect(_on_conditional_screen_exit)

func _on_conditional_screen_exit():
    # 检查删除条件
    if should_remove_when_offscreen():
        offscreen_removal.onScreenExited()

func should_remove_when_offscreen() -> bool:
    # 满血的敌人不删除，让其有机会回来
    return health_component.health < health_component.maxHealth
```

### 子弹管理系统
```gdscript
# 子弹的离屏管理
@export var offscreen_removal: OffscreenRemovalComponent

func _ready():
    # 子弹离屏立即删除
    offscreen_removal.removalDelay = 0.0
    offscreen_removal.ignoreIfHidden = false  # 隐藏的子弹也删除
    
    # 连接信号以统计
    var notifier = get_node("VisibleOnScreenNotifier2D")
    notifier.screen_exited.connect(_on_bullet_offscreen)

func _on_bullet_offscreen():
    # 记录子弹统计
    BulletManager.register_offscreen_bullet()
    
    # 执行删除
    offscreen_removal.onScreenExited()
```

### 特效清理
```gdscript
# 特效的智能清理
@export var offscreen_removal: OffscreenRemovalComponent
@export var effect_duration: float = 5.0

func _ready():
    # 特效有较长的删除延迟
    offscreen_removal.removalDelay = 1.0
    
    # 但不超过特效总时长
    var max_lifetime = Timer.new()
    max_lifetime.wait_time = effect_duration
    max_lifetime.one_shot = true
    max_lifetime.timeout.connect(force_cleanup)
    add_child(max_lifetime)
    max_lifetime.start()

func force_cleanup():
    # 强制清理，无论是否在屏幕上
    get_parent_entity().requestDeletion()
```

### 动态删除控制
```gdscript
# 基于游戏状态的动态控制
@export var offscreen_removal: OffscreenRemovalComponent

func _ready():
    # 监听游戏状态变化
    GameState.game_paused.connect(_on_game_paused)
    GameState.game_resumed.connect(_on_game_resumed)
    GameState.cutscene_started.connect(_on_cutscene_started)

func _on_game_paused():
    # 暂停时禁用删除
    offscreen_removal.isEnabled = false

func _on_game_resumed():
    # 恢复时启用删除
    offscreen_removal.isEnabled = true

func _on_cutscene_started():
    # 过场动画时延长删除时间
    offscreen_removal.removalDelay = 5.0
```

## 设计模式

### 离屏管理器
```gdscript
# 全局离屏实体管理
class_name OffscreenManager
extends Node

var tracked_entities: Array[Entity] = []
var cleanup_stats: Dictionary = {}

func register_entity(entity: Entity):
    tracked_entities.append(entity)
    
    # 连接删除信号
    entity.tree_exiting.connect(_on_entity_removed.bind(entity))

func _on_entity_removed(entity: Entity):
    tracked_entities.erase(entity)
    
    # 更新统计
    var entity_type = entity.get_script().get_global_name()
    cleanup_stats[entity_type] = cleanup_stats.get(entity_type, 0) + 1

func get_cleanup_stats() -> Dictionary:
    return cleanup_stats.duplicate()

func force_cleanup_all():
    for entity in tracked_entities:
        if is_instance_valid(entity):
            entity.requestDeletion()
    tracked_entities.clear()
```

### 智能清理系统
```gdscript
# 基于重要性的智能清理
class_name SmartCleanupSystem
extends Node

enum EntityPriority {
    LOW,      # 装饰物、特效
    MEDIUM,   # 敌人子弹、拾取物
    HIGH,     # 玩家子弹、重要道具
    CRITICAL  # 玩家、BOSS、关键NPC
}

func setup_cleanup(entity: Entity, priority: EntityPriority):
    var offscreen_comp = entity.findFirstComponentSubclass(OffscreenRemovalComponent)
    if not offscreen_comp:
        return
    
    # 根据优先级设置删除参数
    match priority:
        EntityPriority.LOW:
            offscreen_comp.removalDelay = 0.0
        EntityPriority.MEDIUM:
            offscreen_comp.removalDelay = 1.0
        EntityPriority.HIGH:
            offscreen_comp.removalDelay = 3.0
        EntityPriority.CRITICAL:
            offscreen_comp.isEnabled = false  # 永不删除
```

### 性能监控系统
```gdscript
# 离屏删除性能监控
class_name OffscreenPerformanceMonitor
extends Node

var entities_cleaned: int = 0
var memory_saved: int = 0
var cleanup_rate: float = 0.0

func _ready():
    # 每秒更新统计
    var timer = Timer.new()
    timer.wait_time = 1.0
    timer.timeout.connect(_update_stats)
    add_child(timer)
    timer.start()

func register_cleanup(entity: Entity):
    entities_cleaned += 1
    
    # 估算节省的内存（简化计算）
    var node_count = count_nodes_recursive(entity)
    memory_saved += node_count * 100  # 每个节点约100字节

func count_nodes_recursive(node: Node) -> int:
    var count = 1
    for child in node.get_children():
        count += count_nodes_recursive(child)
    return count

func _update_stats():
    cleanup_rate = entities_cleaned / Engine.get_process_frames() * 60.0
    
    # 输出性能报告
    if cleanup_rate > 10.0:  # 每秒清理超过10个实体
        print(f"高清理率警告: {cleanup_rate:.1f} entities/sec")
```

## 技术细节

### 屏幕检测机制
- 依赖VisibleOnScreenNotifier2D.screen_exited信号
- 检测实体的边界矩形是否完全离开屏幕
- 支持多个视口的屏幕检测

### 延迟删除实现
- 使用get_tree().create_timer()创建异步延迟
- 延迟期间可以被新的screen_entered事件打断
- await语法确保异步操作的正确性

### 可见性处理
- ignoreIfHidden检查parentEntity.visible状态
- 防止闪烁动画或临时隐藏导致误删
- 考虑节点层级的可见性继承

## 注意事项

### 性能影响
- 大量离屏实体可能影响性能
- 合理设置removalDelay避免过频繁创建/删除
- 监控离屏删除的频率和内存使用

### 边界情况处理
- 处理VisibleOnScreenNotifier2D未正确设置的情况
- 考虑实体在屏幕边缘的闪现问题
- 注意多视口环境下的行为

### 设计建议
- 根据实体类型设置不同的删除策略
- 考虑实现实体池来优化内存使用
- 提供删除前的清理回调机制

## 相关组件
- [`VisibleOnScreenNotifier2D`](https://docs.godotengine.org/en/stable/classes/class_visibleonscreennotifier2d.html) - Godot内置屏幕检测
- [`Entity`](../../Entities/Entity.md) - 实体基类
- [`SceneManager`](../../Managers/SceneManager.md) - 场景管理器 