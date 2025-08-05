# StatModifierComponent

## 概述
`StatModifierComponent` 是一个时间驱动的统计数据修改组件，在指定的时间间隔内修改 `Stat` 资源。该组件非常适合用于实现生命值再生、毒素持续伤害等功能。

**继承关系：** `StatModifierComponent` → `Component` → `Node`

## 主要特性
- 🔄 定时修改统计数据
- ⚙️ 支持多个统计数据同时修改
- 🎮 可启用/禁用功能
- ⏰ 基于Timer组件的时间管理
- 📊 支持正负数值修改

## 导出属性

### `statsToModify: Dictionary[Stat, int]`
统计数据修改字典，键为 `Stat` 资源，值为应用到对应统计数据的正负修改量。

**类型：** `Dictionary[Stat, int]`  
**用途：** 定义要修改的统计数据及其修改量

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**用途：** 控制组件的激活状态

## 主要方法

### `onTimerTimeout() -> void`
定时器超时回调函数。检查组件是否启用，如果启用则调用统计数据修改。

**用途：** Timer节点的timeout信号处理函数

### `modifyStats() -> void`
执行统计数据修改的核心方法。遍历 `statsToModify` 字典并应用所有修改。

**实现逻辑：**
```gdscript
for stat in statsToModify:
    stat.value += statsToModify[stat]
```

## 使用示例

### 基本生命值再生
```gdscript
# 在场景中设置StatModifierComponent
var statModifier = $StatModifierComponent
var healthStat = load("res://Resources/HealthStat.tres")

# 设置生命值再生（每次+5）
statModifier.statsToModify[healthStat] = 5

# 确保Timer设置为合适的间隔（如2秒）
$StatModifierComponent/Timer.wait_time = 2.0
$StatModifierComponent/Timer.start()
```

### 毒素持续伤害
```gdscript
# 创建毒素伤害效果
var poisonModifier = $PoisonDamageComponent
var healthStat = load("res://Resources/HealthStat.tres")

# 设置持续伤害（每次-3）
poisonModifier.statsToModify[healthStat] = -3

# 快速伤害间隔（如0.5秒）
$PoisonDamageComponent/Timer.wait_time = 0.5
```

### 多统计数据修改
```gdscript
# 同时修改多个统计数据
var multiModifier = $MultiStatModifier
var healthStat = load("res://Resources/HealthStat.tres")
var manaStat = load("res://Resources/ManaStat.tres")

# 同时再生生命值和法力值
multiModifier.statsToModify[healthStat] = 3
multiModifier.statsToModify[manaStat] = 2
```

### 动态控制
```gdscript
# 根据游戏状态启用/禁用
func enableRegeneration():
    $HealthRegenComponent.isEnabled = true

func disableRegeneration():
    $HealthRegenComponent.isEnabled = false

# 与信号系统配合使用
func _ready():
    var shieldedHealth = $ShieldedHealthComponent
    # 当护盾减少时开始再生
    shieldedHealth.shieldDidDecrease.connect($StatModifierComponent/Timer.start)
```

## 设计模式

### 命名约定
为了更好地管理功能，建议重命名组件节点以反映其用途：
- `ManaRegenerationComponent`
- `PoisonDamageComponent`
- `HealthRegenComponent`
- `BurnDamageComponent`

### 与其他组件的配合

**与Timer的关系：**
- 需要作为子节点添加Timer组件
- Timer的 `timeout` 信号应连接到 `onTimerTimeout()` 方法

**与ShieldedHealthComponent的配合：**
- 可以监听 `shieldDidDecrease` 信号来启动再生

**与StatsComponent的配合：**
- 目标Stat资源通常来自Entity的StatsComponent

## 注意事项

1. **Timer设置：** 确保有Timer子节点并正确配置间隔时间
2. **Stat资源：** 确保statsToModify中的Stat资源有效且已正确初始化
3. **性能考虑：** 对于大量实体，考虑Timer间隔的性能影响
4. **数值平衡：** 正值增加统计数据，负值减少统计数据

## 相关组件
- `StatsComponent` - 管理实体的统计数据
- `StatModifierOnDeathComponent` - 死亡时修改统计数据
- `ShieldedHealthComponent` - 护盾系统，可触发再生
- `HealthComponent` - 生命值管理 