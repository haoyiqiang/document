# TileBasedAnimationComponent API

## 概述
`TileBasedAnimationComponent` 基于`TileBasedPositionComponent`的移动状态来控制实体的`AnimatedSprite2D`动画。专为瓦片网格移动的角色提供待机和行走动画。

**继承链**：`Component` → `TileBasedAnimationComponent`

## 依赖要求
- **TileBasedPositionComponent**：必需的瓦片位置组件
- **AnimatedSprite2D**：实体必须有AnimatedSprite2D子节点

## 主要特性
- 🎬 **自动动画**：根据移动状态自动切换动画
- 🔄 **方向翻转**：支持左移时自动翻转精灵
- 🎯 **精确同步**：与瓦片移动完美同步
- ⚡ **性能优化**：仅在移动状态变化时更新动画
- 🔧 **灵活配置**：可自定义动画名称和翻转行为

## 导出属性

### 动画配置
- **idleAnimation** (`StringName = "idle"`)：待机动画名称
- **walkAnimation** (`StringName = "walk"`)：行走动画名称

### 视觉配置
- **flipWhenWalkingLeft** (`bool = true`)：左移时是否翻转精灵
- **isEnabled** (`bool = true`)：是否启用动画控制

## 状态属性
- **animatedSprite** (`AnimatedSprite2D`)：自动查找的动画精灵

## 主要方法
- **onTileBasedPositionComponent_willStartMovingToNewCell**(newDestination: Vector2i) → `void`：开始移动时的回调
- **onTileBasedPositionComponent_didArriveAtNewCell**(newDestination: Vector2i) → `void`：到达新格子时的回调

## 使用示例

### 基础瓦片动画
```gdscript
# 设置基本的瓦片动画
@export var tile_animation: TileBasedAnimationComponent

func _ready():
    tile_animation.idleAnimation = "idle"
    tile_animation.walkAnimation = "walk"
    tile_animation.flipWhenWalkingLeft = true
```

### 自定义动画名称
```gdscript
# 使用自定义动画名称
@export var tile_animation: TileBasedAnimationComponent

func setup_character_animations():
    # 根据角色类型设置不同动画
    match character_type:
        "warrior":
            tile_animation.idleAnimation = "warrior_idle"
            tile_animation.walkAnimation = "warrior_walk"
        "mage":
            tile_animation.idleAnimation = "mage_idle"
            tile_animation.walkAnimation = "mage_cast"
        "rogue":
            tile_animation.idleAnimation = "rogue_crouch"
            tile_animation.walkAnimation = "rogue_sneak"
```

### 条件动画控制
```gdscript
# 基于状态的动画控制
@export var tile_animation: TileBasedAnimationComponent
@export var health_component: HealthComponent

func _ready():
    health_component.didTakeDamage.connect(_on_damage_taken)
    setup_default_animations()

func setup_default_animations():
    tile_animation.idleAnimation = "idle"
    tile_animation.walkAnimation = "walk"

func _on_damage_taken():
    # 受伤时切换到受伤动画
    tile_animation.idleAnimation = "hurt_idle"
    tile_animation.walkAnimation = "hurt_walk"
    
    # 3秒后恢复正常动画
    await get_tree().create_timer(3.0).timeout
    setup_default_animations()
```

### 多状态动画系统
```gdscript
# 复杂的多状态动画系统
@export var tile_animation: TileBasedAnimationComponent
@export var stats_component: StatsComponent

enum AnimationState {
    NORMAL,
    TIRED,
    ENERGIZED,
    INJURED
}

var current_animation_state: AnimationState = AnimationState.NORMAL

func _ready():
    stats_component.statDidChange.connect(_on_stat_changed)
    update_animation_state()

func _on_stat_changed(stat_name: StringName, old_value: float, new_value: float):
    update_animation_state()

func update_animation_state():
    var health_ratio = get_health_ratio()
    var energy_ratio = get_energy_ratio()
    
    var new_state: AnimationState
    
    if health_ratio < 0.3:
        new_state = AnimationState.INJURED
    elif energy_ratio < 0.2:
        new_state = AnimationState.TIRED
    elif energy_ratio > 0.8:
        new_state = AnimationState.ENERGIZED
    else:
        new_state = AnimationState.NORMAL
    
    if new_state != current_animation_state:
        current_animation_state = new_state
        apply_animation_state()

func apply_animation_state():
    match current_animation_state:
        AnimationState.NORMAL:
            tile_animation.idleAnimation = "idle"
            tile_animation.walkAnimation = "walk"
        AnimationState.TIRED:
            tile_animation.idleAnimation = "tired_idle"
            tile_animation.walkAnimation = "tired_walk"
        AnimationState.ENERGIZED:
            tile_animation.idleAnimation = "energized_idle"
            tile_animation.walkAnimation = "energized_run"
        AnimationState.INJURED:
            tile_animation.idleAnimation = "injured_idle"
            tile_animation.walkAnimation = "injured_limp"
```

### 方向感知动画
```gdscript
# 增强的方向感知系统
@export var tile_animation: TileBasedAnimationComponent
@export var tile_position: TileBasedPositionComponent

func _ready():
    tile_position.willStartMovingToNewCell.connect(_on_movement_started)
    tile_position.didArriveAtNewCell.connect(_on_movement_ended)

func _on_movement_started(destination: Vector2i):
    var direction = tile_position.inputVector
    
    # 根据移动方向选择不同的行走动画
    if direction.y < 0:        # 向上
        tile_animation.walkAnimation = "walk_up"
    elif direction.y > 0:      # 向下
        tile_animation.walkAnimation = "walk_down"
    elif direction.x != 0:     # 左右
        tile_animation.walkAnimation = "walk_side"
        tile_animation.flipWhenWalkingLeft = true
    
    # 对角线移动
    if direction.x != 0 and direction.y != 0:
        if direction.y < 0:
            tile_animation.walkAnimation = "walk_diagonal_up"
        else:
            tile_animation.walkAnimation = "walk_diagonal_down"

func _on_movement_ended(destination: Vector2i):
    # 根据最后的朝向选择待机动画
    var sprite = tile_animation.animatedSprite
    if sprite.flip_h:
        tile_animation.idleAnimation = "idle_left"
    else:
        tile_animation.idleAnimation = "idle_right"
```

### 动画事件处理
```gdscript
# 动画事件和音效集成
@export var tile_animation: TileBasedAnimationComponent
@export var audio_player: AudioStreamPlayer2D

func _ready():
    # 连接动画精灵的信号
    var sprite = tile_animation.animatedSprite
    sprite.animation_finished.connect(_on_animation_finished)
    sprite.frame_changed.connect(_on_frame_changed)

func _on_animation_finished():
    # 某些动画完成时的处理
    if tile_animation.animatedSprite.animation == "special_move":
        tile_animation.idleAnimation = "idle"
        tile_animation.walkAnimation = "walk"

func _on_frame_changed():
    var sprite = tile_animation.animatedSprite
    
    # 在特定帧播放脚步声
    if sprite.animation == "walk" and sprite.frame in [1, 3]:
        audio_player.stream = preload("res://audio/footstep.ogg")
        audio_player.play()
    
    # 在特定帧播放特效
    if sprite.animation == "special_attack" and sprite.frame == 2:
        spawn_attack_effect()
```

## 设计模式

### 动画状态机
```gdscript
# 动画状态机系统
class_name TileAnimationStateMachine
extends Node

@export var tile_animation: TileBasedAnimationComponent
var animation_states: Dictionary = {}
var current_state: String = "normal"

func _ready():
    setup_animation_states()

func setup_animation_states():
    animation_states["normal"] = {
        "idle": "idle",
        "walk": "walk"
    }
    animation_states["combat"] = {
        "idle": "combat_idle",
        "walk": "combat_walk"
    }
    animation_states["stealth"] = {
        "idle": "crouch_idle",
        "walk": "crouch_walk"
    }

func change_state(new_state: String):
    if new_state in animation_states:
        current_state = new_state
        var state_anims = animation_states[current_state]
        tile_animation.idleAnimation = state_anims["idle"]
        tile_animation.walkAnimation = state_anims["walk"]
```

### 动画管理器
```gdscript
# 全局动画管理系统
class_name TileAnimationManager
extends Node

var registered_components: Array[TileBasedAnimationComponent] = []

func register_component(component: TileBasedAnimationComponent):
    registered_components.append(component)

func set_global_animation_speed(speed: float):
    for component in registered_components:
        if is_instance_valid(component):
            component.animatedSprite.speed_scale = speed

func pause_all_animations():
    for component in registered_components:
        if is_instance_valid(component):
            component.animatedSprite.pause()

func resume_all_animations():
    for component in registered_components:
        if is_instance_valid(component):
            component.animatedSprite.play()
```

## 技术细节

### 信号连接
- 监听willStartMovingToNewCell开始行走动画
- 监听didArriveAtNewCell切换回待机动画
- 自动处理移动状态变化

### 翻转逻辑
- 仅在X轴移动时处理翻转
- 垂直移动时保持当前翻转状态
- 支持对角线移动的翻转处理

### 性能优化
- 延迟查找AnimatedSprite2D节点
- 仅在移动状态变化时更新动画
- 缓存精灵引用避免重复查找

## 注意事项

### 动画资源
- 确保AnimatedSprite2D包含所需动画
- 动画名称必须与配置的名称匹配
- 考虑动画循环设置的正确性

### 同步问题
- 动画切换应与瓦片移动同步
- 避免动画状态与实际移动状态不一致
- 处理快速连续移动的情况

### 性能考虑
- 大量瓦片角色时优化动画更新
- 避免每帧检查动画状态
- 考虑使用动画池优化内存使用

## 相关组件
- [`TileBasedPositionComponent`](../Movement/TileBasedPositionComponent.md) - 瓦片位置管理
- [`PlatformerAnimationComponent`](../Visual/PlatformerAnimationComponent.md) - 平台动画（连续移动）
- [`AnimatedSprite2D`](https://docs.godotengine.org/en/stable/classes/class_animatedsprite2d.html) - Godot动画精灵 