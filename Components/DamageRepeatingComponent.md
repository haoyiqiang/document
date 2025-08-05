# DamageRepeatingComponent API 参考

## 概述

`DamageRepeatingComponent` 是 [`DamageComponent`](DamageComponent.md) 的变体，只要敌方实体的 [`DamageReceivingComponent`](DamageReceivingComponent.md) "受伤区域"保持接触，就会重复应用其伤害。适用于酸池、毒气等持续危险区域。

**继承关系：**
`Component` → `DamageComponent` → `DamageRepeatingComponent`

## 主要特性

- ⏰ **重复伤害** - 定时对接触的所有敌方实体造成伤害
- 🎯 **接触检测** - 基于Area2D碰撞系统
- 🔄 **智能定时** - 仅在有目标时运行定时器
- ⚠️ **实验性功能** - 未来可能支持基于接触时间的伤害偏移

## 重要特性

💡 **同时伤害：** 伤害会同时应用到所有接触的 `DamageReceivingComponent`，无论它们何时开始接触。

## 子节点

### RepetitionTimer
- **类型：** `Timer`
- **用途：** 控制重复伤害的触发间隔
- **默认：** 每1秒触发一次
- **配置：** 启用"Editable Children"来修改持续时间

## 主要方法

### onDidCollideReceiver()
```gdscript
func onDidCollideReceiver(_damageReceivingComponent: DamageReceivingComponent) -> void
```
当第一个受伤组件进入接触时启动重复定时器。

### onDidLeaveReceiver()
```gdscript
func onDidLeaveReceiver(_damageReceivingComponent: DamageReceivingComponent) -> void
```
当最后一个受伤组件离开接触时停止重复定时器。

### onRepetitionTimer_timeout()
```gdscript
func onRepetitionTimer_timeout() -> void
```
定时器触发时对所有接触的接收者造成伤害。

## 使用示例

### 酸池危险区域
```gdscript
# 创建持续伤害的酸池
extends HazardEntity

func _ready():
    var damage_repeating = $DamageRepeatingComponent
    
    # 配置伤害参数
    damage_repeating.damageOnCollision = 5
    damage_repeating.attackerFactions = FactionComponent.NEUTRAL
    damage_repeating.friendlyFire = false
    
    # 设置伤害间隔（每0.5秒）
    damage_repeating.get_node("RepetitionTimer").wait_time = 0.5
```

### 毒气区域
```gdscript
# 创建毒气伤害区域
extends EnvironmentalHazard

func setup_poison_gas():
    var poison_damage = $DamageRepeatingComponent
    
    # 毒气伤害配置
    poison_damage.damageOnCollision = 2
    poison_damage.hitChance = 100  # 必中
    poison_damage.attackerFactions = FactionComponent.ENEMIES
    
    # 毒气效果每2秒触发
    poison_damage.get_node("RepetitionTimer").wait_time = 2.0
    
    # 添加视觉效果
    create_poison_particles()
```

### 可升级陷阱
```gdscript
# 创建可以升级的重复伤害陷阱
extends TrapEntity

@export var trap_level: int = 1

func _ready():
    setup_trap_damage()

func setup_trap_damage():
    var damage_component = $DamageRepeatingComponent
    
    # 根据等级调整伤害和频率
    damage_component.damageOnCollision = trap_level * 3
    var timer = damage_component.get_node("RepetitionTimer")
    timer.wait_time = max(0.5, 2.0 - (trap_level * 0.2))  # 等级越高频率越快

func upgrade_trap():
    trap_level += 1
    setup_trap_damage()
    
    # 升级视觉效果
    $VisualEffects.scale *= 1.1
    $VisualEffects.modulate = Color.RED.lerp(Color.YELLOW, trap_level / 10.0)
```

### 动态危险区域
```gdscript
# 创建会移动的危险区域
extends MovingHazard

@onready var damage_repeating = $DamageRepeatingComponent
@onready var movement_component = $LinearMotionComponent

func _ready():
    # 配置移动伤害区域
    damage_repeating.damageOnCollision = 8
    damage_repeating.get_node("RepetitionTimer").wait_time = 1.0
    
    # 监听移动状态
    movement_component.didStartMoving.connect(_on_start_moving)
    movement_component.didStopMoving.connect(_on_stop_moving)

func _on_start_moving():
    # 移动时降低伤害频率
    damage_repeating.get_node("RepetitionTimer").wait_time = 1.5

func _on_stop_moving():
    # 停止时增加伤害频率
    damage_repeating.get_node("RepetitionTimer").wait_time = 0.8
```

## 技术细节

### 定时器管理
- 定时器仅在有接触目标时运行
- 第一个目标进入时启动
- 最后一个目标离开时停止
- 避免不必要的CPU消耗

### 批量伤害处理
- `causeDamageToAllReceivers()` 同时处理所有接触的目标
- 高效的批量操作
- 统一的伤害计算和应用

### 信号集成
- 继承自 `DamageComponent` 的完整信号系统
- 支持命中率、暴击等所有伤害机制
- 与其他战斗组件无缝集成

## 注意事项

⚠️ **使用场景：**
- 适用于攻击者实体（如酸池、毒气）
- 对于受害者身上的持续效果，使用 [`DamageOverTimeComponent`](DamageOverTimeComponent.md)

💡 **性能优化：**
- 定时器智能启停减少性能开销
- 批量伤害处理提高效率
- 避免过于频繁的伤害触发

🔮 **计划功能：**
- 基于接触时间的伤害偏移
- 更灵活的定时机制
- 渐增伤害效果

## 相关组件

- [`DamageComponent`](DamageComponent.md) - 父类，提供基础伤害功能
- [`DamageOverTimeComponent`](DamageOverTimeComponent.md) - 对比选择，用于受害者的DoT效果
- [`DamageReceivingComponent`](DamageReceivingComponent.md) - 必需目标组件
- [`AreaCollisionComponent`](../Physics/AreaCollisionComponent.md) - 提供碰撞检测基础 