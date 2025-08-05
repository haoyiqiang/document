# DamageOverTimeComponent API 参考

## 概述

`DamageOverTimeComponent` 以固定间隔对父实体的 [`DamageReceivingComponent`](DamageReceivingComponent.md) 应用持续伤害。该组件不依赖"伤害区域/受伤区域"碰撞系统。适用于中毒、燃烧等持续效果。

**继承关系：**
`Component` → `DamageOverTimeComponent`

## 主要特性

- ⏰ **定时伤害** - 按固定间隔造成伤害
- 🎯 **直接应用** - 无需碰撞检测
- ⚠️ **实验性功能** - 可能在未来版本中改进
- 🔄 **自动移除** - 时间到期后自动删除组件

## 导出属性

```gdscript
@export_range(0, 100, 1) var damagePerTick: int = 1
```
每次定时器触发时造成的伤害值。

```gdscript
@export_flags("neutral", "players", "playerAllies", "enemies") var attackerFactions: int = 1
```
伤害源的阵营标志。

```gdscript
@export var friendlyFire: bool = false
```
是否允许友军伤害。

```gdscript
@export var isEnabled: bool = true
```
控制组件是否启用。

## 子节点定时器

### DamageTimer
控制伤害触发频率的定时器。

### RemovalTimer
控制组件存在时长的定时器，到期后组件自动删除。

## 使用示例

### 基本中毒效果
```gdscript
# 为目标添加中毒效果
func apply_poison(target: Entity, duration: float = 5.0):
    var poison = preload("res://Components/Combat/DamageOverTimeComponent.tscn").instantiate()
    target.add_child(poison)
    
    # 配置参数
    poison.damagePerTick = 2
    poison.attackerFactions = FactionComponent.ENEMIES
    
    # 设置持续时间
    poison.get_node("RemovalTimer").wait_time = duration
    poison.get_node("DamageTimer").wait_time = 1.0  # 每秒伤害
```

### 燃烧效果
```gdscript
# 创建可叠加的燃烧效果
func apply_burn_damage(target: Entity, burn_stacks: int):
    for i in burn_stacks:
        var burn = DamageOverTimeComponent.new()
        target.add_child(burn)
        burn.damagePerTick = 1
        burn.get_node("DamageTimer").wait_time = 0.5
        burn.get_node("RemovalTimer").wait_time = 3.0
```

## 重要限制

⚠️ **单组件限制：** 当前版本不支持同一实体上存在多个相同类型的组件，只有最新添加的组件会保持活跃。

💡 **解决方案：** 要实现可叠加效果，请使用不同的组件实例或修改组件来支持叠加计数。

## 相关组件

- [`DamageReceivingComponent`](DamageReceivingComponent.md) - 必需依赖
- [`DamageRepeatingComponent`](DamageRepeatingComponent.md) - 用于攻击者实体的持续伤害
- [`HealthComponent`](HealthComponent.md) - 常受DoT效果影响的组件 