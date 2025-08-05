# HealthComponent API 文档

## 概述
`HealthComponent` 存储实体的生命值并管理销毁等功能。

应配合 `DamageReceivingComponent` 来处理伤害应用和阵营处理。

## 继承
`HealthComponent` → `Component` → `Node`

## 主要特性
- 管理实体生命值
- 支持生命值变化事件
- 可配置死亡时行为
- 支持伤害限制机制
- 游戏结束触发器

## 导出属性

#### `health: Stat`
实体的生命值Stat资源。

**注意**：如果同时使用 `StatsComponent`，必须在两个组件之间共享相同的Stat资源。

#### `shouldGameOverOnZero: bool = false`
当生命值归零时是否触发全局游戏结束信号。
- 受 `isEnabled` 影响
- **警告**：不应在多玩家游戏中使用；应使用 `GameState.removePlayer()` 替代

#### `shouldRemoveEntityOnZero: bool = false`
当生命值归零时是否移除实体。
- 受 `isEnabled` 影响

#### `shouldClampHealthDamage: bool`
是否限制单次伤害的最大值。
- 用于需要固定次数攻击才能摧毁的对象/敌人
- 伤害值受 `maximumHealthDamagePerHit` 限制

#### `maximumHealthDamagePerHit: int`
单次攻击的最大伤害限制。
- 仅在 `shouldClampHealthDamage` 为true时生效
- **注意**：如果此值为负数，则只允许治疗

#### `isEnabled: bool = true`
是否启用此组件。

## 信号

### `healthDidDecrease(difference: int)`
生命值减少时发出。
- `difference` 是负数
- 可与 `healthDidIncrease` 连接到同一函数

### `healthDidIncrease(difference: int)`
生命值增加时发出。
- `difference` 总是正数
- 可与 `healthDidDecrease` 连接到同一函数

### `healthDidZero`
生命值降到零或以下时发出。
- 仅在生命值从 >0 降到 ≤0 时发出
- 不受 `isEnabled` 影响

### `healthDidMax`
生命值达到或超过最大值时发出。

### `willRemoveEntity`
即将移除实体时发出。

## 主要方法

### 伤害处理

#### `damage(damageAmount: int) -> int`
对实体造成伤害。

**参数**：
- `damageAmount: int` - 必须为正数，负值会增加生命值

**返回**：剩余生命值

**行为**：
- 检查 `isEnabled` 状态
- 如果启用了伤害限制，会应用 `maximumHealthDamagePerHit`
- 从生命值中扣除伤害值

```gdscript
# 造成伤害
var remainingHealth = healthComponent.damage(25)
print("剩余生命值: ", remainingHealth)
```

### 治疗处理

#### `heal(healAmount: int) -> int`
治疗实体。

**参数**：
- `healAmount: int` - 必须为正数，负值会减少生命值

**返回**：剩余生命值

**行为**：
- 检查 `isEnabled` 状态
- 增加生命值
- 如果达到最大值会发出 `healthDidMax` 信号

```gdscript
# 治疗
var remainingHealth = healthComponent.heal(10)
print("治疗后生命值: ", remainingHealth)
```

### 内部方法

#### `onHealthChanged() -> void`
生命值变化时的内部处理方法。
- 发出相应的信号
- 处理死亡逻辑
- 触发游戏结束等事件

## 使用示例

### 基本设置
```gdscript
# 设置生命值组件
var healthComponent = HealthComponent.new()
entity.add_child(healthComponent)

# 创建生命值Stat
var healthStat = Stat.new()
healthStat.max = 100
healthStat.value = 100
healthComponent.health = healthStat

# 配置死亡行为
healthComponent.shouldRemoveEntityOnZero = true
```

### 连接信号
```gdscript
# 监听生命值变化
healthComponent.healthDidDecrease.connect(_on_health_decreased)
healthComponent.healthDidIncrease.connect(_on_health_increased)
healthComponent.healthDidZero.connect(_on_death)

func _on_health_decreased(difference: int) -> void:
    print("受到伤害: ", abs(difference))

func _on_health_increased(difference: int) -> void:
    print("获得治疗: ", difference)

func _on_death() -> void:
    print("实体死亡")
```

### 伤害限制设置
```gdscript
# 设置伤害限制（每次最多受到20点伤害）
healthComponent.shouldClampHealthDamage = true
healthComponent.maximumHealthDamagePerHit = 20

# 即使攻击力是100，也只会受到20点伤害
healthComponent.damage(100) # 实际只扣除20点生命值
```

### Boss战设置
```gdscript
# 需要固定次数攻击的Boss
healthComponent.health.max = 100
healthComponent.health.value = 100
healthComponent.shouldClampHealthDamage = true
healthComponent.maximumHealthDamagePerHit = 10
# Boss需要10次攻击才能击败，无论攻击力多强
```

## 设计模式

### 与其他组件协作
```gdscript
# 配合DamageReceivingComponent使用
# DamageReceivingComponent接收伤害 → HealthComponent处理生命值变化

# 配合StatsComponent使用时
var sharedHealthStat = Stat.new()
healthComponent.health = sharedHealthStat
statsComponent.addStat("health", sharedHealthStat)
```

## 注意事项

1. **Stat资源共享**：与StatsComponent共用时必须使用相同的Stat资源
2. **死亡处理**：只在生命值从正数变为零或负数时触发死亡逻辑
3. **游戏结束**：谨慎使用 `shouldGameOverOnZero`，适合单玩家游戏
4. **伤害限制**：负数限制值表示只允许治疗
5. **信号时机**：某些信号不受 `isEnabled` 影响

## 相关组件
- `DamageReceivingComponent` - 伤害接收组件
- `DamageComponent` - 伤害造成组件
- `StatsComponent` - 数值管理组件
- `FactionComponent` - 阵营组件
- `ShieldedHealthComponent` - 护盾生命值组件 