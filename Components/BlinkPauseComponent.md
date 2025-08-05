# BlinkPauseComponent API

## 概述
`BlinkPauseComponent` 是一个视觉效果组件，用于暂停实体并进行闪烁动画。它通过切换可见性来吸引玩家注意力，常用于生成新对象、死亡动画或状态转换的视觉提示，完成后自动移除。

**继承链**：`Component` → `BlinkPauseComponent`

## 主要特性
- ✨ **闪烁动画**：通过切换可见性创建闪烁效果
- ⏸️ **实体暂停**：动画期间暂停实体的所有处理
- 🔄 **自动移除**：完成动画后自动清理组件
- 📡 **事件通知**：提供完成信号供其他系统响应
- 🎯 **注意力引导**：有效引导玩家注意力到重要对象

## 导出属性

### 动画配置
- **timesToBlink** (`int = 5`)：闪烁次数
  - 控制完整的显示/隐藏循环次数

### 状态属性（存储）
- **count** (`int`)：当前已完成的闪烁次数
- **entityPreviousProcessMode** (`ProcessMode`)：实体之前的处理模式

## 信号

### 事件通知
- **didFinishBlinking**：完成闪烁动画时发出
  - 可用于死亡动画或其他系统的响应

## 主要方法

### 动画控制
- **startBlink()** → `void`：开始闪烁动画
- **onTimeout()** → `void`：定时器超时处理
- **finishBlink()** → `void`：完成闪烁并清理

## 使用示例

### 生成物品闪烁
```gdscript
# 为新生成的物品添加闪烁效果
func spawn_collectible(position: Vector2):
    var collectible = collectible_scene.instantiate()
    add_child(collectible)
    collectible.global_position = position
    
    # 添加闪烁效果吸引注意
    var blink_component = BlinkPauseComponent.new()
    blink_component.timesToBlink = 3
    collectible.add_child(blink_component)
```

### 死亡动画
```gdscript
# 敌人死亡时的闪烁效果
@export var blink_pause: BlinkPauseComponent
@export var health_component: HealthComponent

func _ready():
    health_component.didDie.connect(start_death_animation)

func start_death_animation():
    # 设置较多次数的闪烁表示死亡
    blink_pause.timesToBlink = 6
    blink_pause.didFinishBlinking.connect(remove_enemy)
    blink_pause.startBlink()

func remove_enemy():
    queue_free()
```

### 状态转换提示
```gdscript
# 角色升级时的闪烁提示
@export var level_up_blink: BlinkPauseComponent

func on_level_up():
    level_up_blink.timesToBlink = 4
    level_up_blink.didFinishBlinking.connect(show_level_up_effects)
    level_up_blink.startBlink()

func show_level_up_effects():
    # 显示升级特效
    GlobalUI.showFloatingText("LEVEL UP!", global_position)
    GlobalSonic.playSound(level_up_sound)
```

### 危险警告闪烁
```gdscript
# 危险区域的警告闪烁
class_name DangerZoneWarning
extends Area2D

@export var warning_blink: BlinkPauseComponent
@export var danger_countdown: float = 3.0

func activate_danger_warning():
    # 快速频繁闪烁警告危险
    warning_blink.timesToBlink = 8
    warning_blink.didFinishBlinking.connect(explode)
    warning_blink.startBlink()

func explode():
    # 执行爆炸或其他危险效果
    create_explosion()
```

### 道具激活闪烁
```gdscript
# 道具被激活时的视觉反馈
@export var activation_blink: BlinkPauseComponent

func on_item_activated():
    # 短暂闪烁确认激活
    activation_blink.timesToBlink = 2
    activation_blink.didFinishBlinking.connect(apply_item_effect)
    activation_blink.startBlink()

func apply_item_effect():
    # 应用道具效果
    player.apply_power_up(item_type)
```

## 设计模式

### 闪烁效果管理器
```gdscript
# 统一管理各种闪烁效果
class_name BlinkEffectManager
extends Node

func create_spawn_blink(target: Node2D, blink_count: int = 3):
    var blink_component = BlinkPauseComponent.new()
    blink_component.timesToBlink = blink_count
    target.add_child(blink_component)
    return blink_component

func create_death_blink(target: Node2D, on_complete: Callable = Callable()):
    var blink_component = create_spawn_blink(target, 5)
    if on_complete.is_valid():
        blink_component.didFinishBlinking.connect(on_complete)
    return blink_component
```

### 条件闪烁系统
```gdscript
# 基于条件的闪烁控制
@export var health_component: HealthComponent
@export var critical_health_blink: BlinkPauseComponent

func _ready():
    health_component.health.valueChanged.connect(check_critical_health)

func check_critical_health():
    if health_component.health.percentage < 15:
        # 生命值极低时持续闪烁警告
        if not critical_health_blink.get_parent():
            add_child(critical_health_blink)
        critical_health_blink.timesToBlink = 999  # 近似无限
        critical_health_blink.startBlink()
```

### 序列闪烁动画
```gdscript
# 多个对象的序列闪烁
class_name SequentialBlinker
extends Node

@export var targets: Array[Node2D]
@export var blink_delay: float = 0.5

func start_sequence_blink():
    for i in targets.size():
        await get_tree().create_timer(i * blink_delay).timeout
        create_blink_for_target(targets[i])

func create_blink_for_target(target: Node2D):
    var blink_component = BlinkPauseComponent.new()
    blink_component.timesToBlink = 3
    target.add_child(blink_component)
```

## 技术细节

### 处理模式管理
- 设置组件为`PROCESS_MODE_ALWAYS`确保动画不被暂停
- 保存并恢复实体原始的处理模式
- 动画期间实体设为`PROCESS_MODE_DISABLED`

### 自动启动机制
- `autostart = true`在`_ready()`时自动开始
- 无需外部调用即可开始动画
- 适合"一次性"效果组件

### 计数逻辑
- 每次变为可见时增加计数
- 一个完整"闪烁"包含隐藏→显示的完整周期
- 达到目标次数时自动结束

## 注意事项

### 使用场景
- 适合短暂的注意力引导效果
- 不适合长期的状态指示
- 主要用于"事件"而非"状态"

### 性能考虑
- 组件会自动移除，无内存泄漏
- 动画期间实体暂停，减少处理负担
- 频繁创建/销毁可能有性能开销

### 设计限制
- 动画过程中实体完全暂停
- 不适合需要继续逻辑处理的场景
- 闪烁频率固定，不可动态调整

## 相关组件
- [`HealthVisualComponent`](../Visual/HealthVisualComponent.md) - 生命值视觉效果
- [`DamageVisualComponent`](../Visual/DamageVisualComponent.md) - 伤害视觉效果
- [`TurnBasedAnimationComponent`](../TurnBased/TurnBasedAnimationComponent.md) - 回合制动画 