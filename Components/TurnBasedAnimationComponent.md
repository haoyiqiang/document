# TurnBasedAnimationComponent API

## 概述
`TurnBasedAnimationComponent` 是回合制系统的动画控制组件，负责在回合制状态变化时播放相应的动画。它支持AnimationPlayer和AnimatedSprite2D两种动画节点，提供完整的回合生命周期动画支持。

**继承链**：`TurnBasedComponent` → `TurnBasedAnimationComponent`

## 依赖要求
- **TurnBasedEntity**：回合制实体
- **AnimationPlayer** 或 **AnimatedSprite2D**：动画播放节点

## 主要特性
- 🎬 **完整生命周期**：支持回合开始、更新、结束的所有动画阶段
- 🎭 **双重支持**：兼容AnimationPlayer和AnimatedSprite2D
- ⏳ **异步等待**：可选择等待动画完成再继续回合
- 🔍 **自动检测**：自动查找父实体的动画节点
- 🎯 **精确控制**：使用组件信号而非实体信号确保执行顺序

## 导出属性

### 动画节点配置
- **animationNode** (`Node`)：动画播放节点（AnimationPlayer或AnimatedSprite2D）
  - 未指定时自动查找父实体的动画节点

### 动画名称配置
- **defaultAnimation** (`StringName = "RESET"`)：默认动画
- **animationForTurnWillBegin** (`StringName = "turnBegin"`)：回合即将开始动画
- **animationForTurnDidBegin** (`StringName = "turnBegin"`)：回合已开始动画
- **animationForTurnWillUpdate** (`StringName = "turnUpdate"`)：回合即将更新动画
- **animationForTurnDidUpdate** (`StringName = "turnUpdate"`)：回合已更新动画
- **animationForTurnWillEnd** (`StringName = "turnEnd"`)：回合即将结束动画
- **animationForTurnDidEnd** (`StringName = "turnEnd"`)：回合已结束动画

### 控制选项
- **shouldWaitForAnimation** (`bool = true`)：是否等待动画完成
  - 重要：可能根据动画时长延迟回合状态循环

## 主要方法

### 动画控制
- **playAnimation(animationName: StringName)** → `void`：播放指定动画
- **findAnimationNode()** → `void`：自动查找动画节点

### 信号处理器（可重写）
- **onWillBeginTurn()** → `void`：处理回合即将开始
- **onDidBeginTurn()** → `void`：处理回合已开始
- **onWillUpdateTurn()** → `void`：处理回合即将更新
- **onDidUpdateTurn()** → `void`：处理回合已更新
- **onWillEndTurn()** → `void`：处理回合即将结束
- **onDidEndTurn()** → `void`：处理回合已结束

### 回合处理（异步）
- **processTurnBegin()** → `void`：处理回合开始阶段
- **processTurnUpdate()** → `void`：处理回合更新阶段
- **processTurnEnd()** → `void`：处理回合结束阶段

## 使用示例

### 基础回合动画
```gdscript
# 设置简单的回合动画
@export var turn_animation: TurnBasedAnimationComponent
@export var animation_player: AnimationPlayer

func _ready():
    turn_animation.animationNode = animation_player
    turn_animation.animationForTurnWillBegin = "attack_prepare"
    turn_animation.animationForTurnDidBegin = "attack_execute"
    turn_animation.animationForTurnDidEnd = "return_idle"
```

### 角色战斗动画
```gdscript
# 为战斗角色设置完整的动画序列
@export var combat_animation: TurnBasedAnimationComponent

func setup_combat_animations():
    combat_animation.defaultAnimation = "idle"
    combat_animation.animationForTurnWillBegin = "prepare"
    combat_animation.animationForTurnDidBegin = "attack"
    combat_animation.animationForTurnWillUpdate = "skill_charge"
    combat_animation.animationForTurnDidUpdate = "skill_cast"
    combat_animation.animationForTurnWillEnd = "recover"
    combat_animation.animationForTurnDidEnd = "idle"
    
    # 等待动画完成确保视觉完整性
    combat_animation.shouldWaitForAnimation = true
```

### 动态动画选择
```gdscript
# 根据行动类型选择不同动画
class_name DynamicTurnAnimation
extends TurnBasedAnimationComponent

@export var attack_animations: Array[StringName] = ["sword_attack", "magic_attack", "bow_attack"]
@export var defense_animations: Array[StringName] = ["shield_block", "dodge_roll", "counter_stance"]

var current_action_type: ActionType

func onDidBeginTurn():
    var animation_name: StringName
    
    match current_action_type:
        ActionType.ATTACK:
            animation_name = attack_animations[randi() % attack_animations.size()]
        ActionType.DEFENSE:
            animation_name = defense_animations[randi() % defense_animations.size()]
        _:
            animation_name = animationForTurnDidBegin
    
    playAnimation(animation_name)
```

### 连击系统动画
```gdscript
# 支持连击的动画系统
@export var combo_animation: TurnBasedAnimationComponent
var combo_count: int = 0

func onDidBeginTurn():
    combo_count += 1
    
    var combo_animation_name = "combo_" + str(min(combo_count, 3))
    playAnimation(combo_animation_name)

func onDidEndTurn():
    # 重置连击或继续保持
    if should_reset_combo():
        combo_count = 0
        playAnimation("combo_end")
    else:
        playAnimation("combo_continue")
```

### 状态驱动动画
```gdscript
# 基于角色状态的动画选择
@export var state_animation: TurnBasedAnimationComponent
@export var character_stats: StatsComponent

func onWillBeginTurn():
    var health_percent = character_stats.health.percentage
    
    if health_percent < 25:
        playAnimation("low_health_prepare")
    elif character_stats.has_status_effect("poisoned"):
        playAnimation("poisoned_prepare")
    else:
        playAnimation(animationForTurnWillBegin)
```

## 设计模式

### 动画状态机
```gdscript
# 复杂的动画状态管理
class_name AnimationStateMachine
extends TurnBasedAnimationComponent

enum AnimationState { IDLE, PREPARING, ATTACKING, RECOVERING, STUNNED }
var current_state: AnimationState = AnimationState.IDLE

func change_animation_state(new_state: AnimationState):
    match new_state:
        AnimationState.IDLE:
            playAnimation("idle")
        AnimationState.PREPARING:
            playAnimation("prepare")
        AnimationState.ATTACKING:
            playAnimation("attack")
        AnimationState.RECOVERING:
            playAnimation("recover")
        AnimationState.STUNNED:
            playAnimation("stunned")
    
    current_state = new_state
```

### 动画链式执行
```gdscript
# 复杂的动画序列
@export var chain_animation: TurnBasedAnimationComponent
var animation_queue: Array[StringName] = []

func queue_animation_sequence(animations: Array[StringName]):
    animation_queue = animations
    play_next_animation()

func play_next_animation():
    if animation_queue.is_empty():
        return
    
    var next_animation = animation_queue.pop_front()
    playAnimation(next_animation)
    
    # 等待动画完成再播放下一个
    await animationNode.animation_finished
    play_next_animation()
```

### 并行动画系统
```gdscript
# 同时控制多个动画节点
class_name MultiAnimationController
extends TurnBasedAnimationComponent

@export var body_animator: AnimationPlayer
@export var weapon_animator: AnimationPlayer
@export var effect_animator: AnimationPlayer

func onDidBeginTurn():
    # 同时播放多个动画
    body_animator.play("attack_body")
    weapon_animator.play("sword_swing")
    effect_animator.play("attack_effect")
    
    # 等待所有动画完成
    if shouldWaitForAnimation:
        await body_animator.animation_finished
        await weapon_animator.animation_finished
        await effect_animator.animation_finished
```

## 技术细节

### 自动节点检测
1. 首先检查父实体是否为AnimationPlayer或AnimatedSprite2D
2. 然后搜索第一个匹配类型的子节点
3. 优先选择AnimationPlayer over AnimatedSprite2D

### 信号连接机制
- 连接到**组件自身**的信号，不是实体的信号
- 确保遵循场景树的节点顺序
- 提供更精确的执行控制

### 异步处理
- `processTurnBegin/Update/End()`方法支持异步等待
- 使用`await animationNode.animation_finished`暂停回合进程
- 确保动画完整播放

## 注意事项

### 性能考虑
- 动画等待可能显著延长回合时间
- 大量实体同时播放动画可能影响性能
- 考虑为快速模式禁用动画等待

### 设计选择
- 空动画名称会跳过播放，便于灵活配置
- 支持运行时更改动画节点
- 可以继承重写信号处理器实现自定义逻辑

### 兼容性说明
- AnimationPlayer和AnimatedSprite2D的API差异已被抽象化
- `is_playing()`和`animation_finished`信号在两种节点中一致
- 自动适配不同类型的动画节点

## 相关组件
- [`TurnBasedComponent`](../TurnBased/TurnBasedComponent.md) - 回合制基础组件
- [`TurnBasedStateUIComponent`](../TurnBased/TurnBasedStateUIComponent.md) - 回合制UI状态
- [`PlatformerAnimationComponent`](../Visual/PlatformerAnimationComponent.md) - 平台游戏动画
- [`HealthVisualComponent`](../Visual/HealthVisualComponent.md) - 生命值视觉效果 