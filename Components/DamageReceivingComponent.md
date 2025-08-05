# DamageReceivingComponent API

## 概述
`DamageReceivingComponent` 是战斗系统的核心组件，用于接收来自`DamageComponent`的伤害。当此组件的Area2D"受伤盒"与攻击者的`DamageComponent`"攻击盒"碰撞时，根据阵营关系决定是否受到伤害。

**继承链**：`Component` → `DamageReceivingComponent`

**⚠️ 警告**：正确设置每个Area2D的`collision_layer`和`collision_mask`，否则战斗系统可能异常！

## 依赖要求
- **Area2D**：此组件必须是Area2D，代表"受伤盒"
- **HealthComponent**：实体必须有HealthComponent（或子类）来处理伤害

## 主要特性
- 🛡️ **伤害接收**：处理来自DamageComponent的伤害碰撞
- 🏴‍☠️ **阵营系统**：基于FactionComponent检查敌友关系
- 🎯 **闪避系统**：支持missChance闪避机制
- 📊 **小数伤害**：累积小数伤害转换为整数（实验性）
- 🔄 **碰撞追踪**：跟踪当前接触的伤害组件
- 💀 **自动清理**：无HealthComponent时可选择自动删除实体

## 导出属性

### 基础配置
- **shouldRemoveEntityIfNoHealthComponent** (`bool = true`)：无HealthComponent时是否删除实体
- **missChance** (`int = 0`, 0-100%)：闪避几率，与DamageComponent.hitChance相减
- **isEnabled** (`bool = true`)：是否启用伤害接收，同时影响Area2D的monitoring和monitorable

## 信号

### 伤害事件
- **didReceiveDamage**(damageComponent: DamageComponent, amount: int, attackerFactions: int)
  - 当阵营敌对或有友伤时发出，即使无HealthComponent也会发出
  - amount可能不是实际扣血量（取决于HealthComponent实现）
- **didCollideWithDamage**(damageComponent: DamageComponent)：与DamageComponent碰撞时总是发出
- **didAccumulateFractionalDamage**(damageComponent: DamageComponent, amount: float, attackerFactions: int)：累积小数伤害时发出（实验性）
- **willRemoveEntity**：无HealthComponent且shouldRemoveEntityIfNoHealthComponent时发出

## 状态属性
- **accumulatedFractionalDamage** (`float`)：累积的小数伤害值
- **damageComponentsInContact** (`Array[DamageComponent]`)：当前接触的伤害组件列表
- **area** (`Area2D`)：代表此组件的Area2D受伤盒

## 主要方法

### 碰撞处理
- **processCollision**(damageComponent: DamageComponent, attackerFactionComponent: FactionComponent) → `bool`
  - 由碰撞的DamageComponent调用
  - 返回是否有敌对阵营或友伤
- **getDamageComponent**(collidingArea: Area2D) → `DamageComponent`：从Area2D获取DamageComponent

### 伤害处理
- **handleDamage**(damageComponent: DamageComponent, damageAmount: int, attackerFactions: int, friendlyFire: bool) → `bool`
  - 处理伤害并传递给HealthComponent
  - 返回是否造成伤害
- **handleFractionalDamage**(damageComponent: DamageComponent, fractionalDamage: float, attackerFactions: int, friendlyFire: bool) → `void`：处理小数伤害（实验性）

### 阵营检查
- **checkFactions**(attackerFactions: int, friendlyFire: bool) → `bool`：检查是否应该受到伤害

## 使用示例

### 基础伤害接收
```gdscript
# 设置基本的伤害接收组件
@export var damage_receiving: DamageReceivingComponent
@export var health_component: HealthComponent

func _ready():
    damage_receiving.missChance = 10  # 10%闪避率
    damage_receiving.didReceiveDamage.connect(_on_damage_received)

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    print(f"受到 {amount} 点伤害")
    # 播放受伤音效
    GlobalSonic.playSound(hurt_sound)
```

### 高级伤害处理
```gdscript
# 复杂的伤害接收逻辑
@export var damage_receiving: DamageReceivingComponent
@export var shield_component: ShieldComponent

func _ready():
    damage_receiving.didReceiveDamage.connect(_on_damage_received)
    damage_receiving.didCollideWithDamage.connect(_on_damage_collision)

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    # 根据伤害类型播放不同效果
    match damage_component.damage_type:
        DamageComponent.Type.FIRE:
            apply_burn_effect()
        DamageComponent.Type.ICE:
            apply_freeze_effect()
        DamageComponent.Type.POISON:
            apply_poison_effect()

func _on_damage_collision(damage_component: DamageComponent):
    # 即使没有伤害也触发，可用于计数、音效等
    combat_stats.total_hits += 1
```

### 无生命值实体
```gdscript
# 可破坏环境对象（如箱子、岩石）
@export var damage_receiving: DamageReceivingComponent

func _ready():
    # 没有HealthComponent，受到伤害时直接删除
    damage_receiving.shouldRemoveEntityIfNoHealthComponent = true
    damage_receiving.willRemoveEntity.connect(_on_will_remove)

func _on_will_remove():
    # 删除前的清理工作
    spawn_loot()
    play_destruction_effect()
```

### 条件伤害接收
```gdscript
# 基于状态的伤害接收
@export var damage_receiving: DamageReceivingComponent
@export var status_component: StatusComponent

func _ready():
    damage_receiving.didReceiveDamage.connect(_on_damage_received)

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    # 根据状态调整反应
    if status_component.hasStatus("Invulnerable"):
        return  # 无敌状态不处理
    
    if status_component.hasStatus("Rage"):
        # 愤怒状态下受伤会增加攻击力
        var damage_component_self = findFirstComponentSubclass(DamageComponent)
        if damage_component_self:
            damage_component_self.damageModifier += 5
```

### 伤害统计系统
```gdscript
# 伤害统计和分析
class_name DamageTracker
extends Node

var damage_log: Array[DamageRecord] = []

func track_damage_receiving_component(component: DamageReceivingComponent):
    component.didReceiveDamage.connect(_on_damage_received)
    component.didCollideWithDamage.connect(_on_damage_collision)

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    var record = DamageRecord.new()
    record.timestamp = Time.get_ticks_msec()
    record.amount = amount
    record.attacker = damage_component.parentEntity
    record.damage_type = damage_component.damage_type
    damage_log.append(record)

func get_damage_statistics() -> Dictionary:
    return {
        "total_damage": damage_log.map(func(r): return r.amount).reduce(func(a, b): return a + b, 0),
        "hit_count": damage_log.size(),
        "average_damage": get_average_damage(),
        "damage_by_type": group_damage_by_type()
    }
```

### 闪避系统增强
```gdscript
# 动态闪避率系统
@export var damage_receiving: DamageReceivingComponent
@export var agility_stat: Stat

func _ready():
    # 根据敏捷属性动态计算闪避率
    update_miss_chance()
    agility_stat.valueChanged.connect(_on_agility_changed)

func _on_agility_changed():
    update_miss_chance()

func update_miss_chance():
    # 敏捷值转换为闪避率 (0-30%)
    damage_receiving.missChance = min(30, agility_stat.value / 10)
```

## 设计模式

### 伤害过滤器
```gdscript
# 伤害过滤和修改系统
class_name DamageFilter
extends Node

@export var damage_receiving: DamageReceivingComponent
@export var resistances: Dictionary = {}  # damage_type -> resistance%

func _ready():
    # 拦截伤害处理
    damage_receiving.didReceiveDamage.connect(_on_damage_received)

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    var resistance = resistances.get(damage_component.damage_type, 0)
    var reduced_amount = amount * (1.0 - resistance / 100.0)
    
    if reduced_amount < amount:
        print(f"抗性减少了 {amount - reduced_amount} 点伤害")
```

### 连击系统
```gdscript
# 连击检测和奖励
@export var damage_receiving: DamageReceivingComponent
var hit_combo: int = 0
var combo_timer: Timer

func _ready():
    damage_receiving.didReceiveDamage.connect(_on_damage_received)
    setup_combo_timer()

func _on_damage_received(damage_component: DamageComponent, amount: int, attacker_factions: int):
    hit_combo += 1
    combo_timer.start()
    
    if hit_combo >= 5:
        trigger_combo_breaker()

func trigger_combo_breaker():
    # 连击打断奖励
    GlobalUI.showFloatingText("连击打断!", Color.GOLD)
    hit_combo = 0
```

## 技术细节

### 碰撞检测机制
- 被动检测：由DamageComponent主动调用processCollision()
- 区域跟踪：维护damageComponentsInContact列表
- 自引用检查：防止实体攻击自己

### 阵营系统集成
- 与FactionComponent协作检查敌友关系
- 支持friendly fire覆盖阵营检查
- 无阵营组件时默认受到所有伤害

### 性能优化
- 避免频繁的类型转换和查找
- 缓存常用组件引用
- 条件检查顺序优化

## 注意事项

### 物理层配置
- 玩家实体：layer = "players", mask = "enemies"
- 敌方实体：layer = "enemies", mask = "players"
- 中性实体：根据需要配置

### 组件顺序
- DamageReceivingComponent是被动组件
- 确保在HealthComponent之后初始化
- 与其他战斗组件协调工作

### 实验性功能
- handleFractionalDamage仍在开发中
- 小数伤害累积机制可能变化
- 在生产环境中谨慎使用

## 相关组件
- [`DamageComponent`](../Combat/DamageComponent.md) - 伤害造成
- [`HealthComponent`](../Combat/HealthComponent.md) - 生命值管理
- [`FactionComponent`](../Combat/FactionComponent.md) - 阵营系统
- [`ShieldedHealthComponent`](../Combat/ShieldedHealthComponent.md) - 护盾生命值 