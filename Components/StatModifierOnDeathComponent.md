# StatModifierOnDeathComponent API

> **继承关系**: Component > StatModifierOnDeathComponent  
> **依赖组件**: HealthComponent

死亡时统计数据修改组件，当实体的生命值归零时自动修改指定的Stat资源，适用于击杀奖励、死亡惩罚等机制。

## ✨ 主要特性

- 💀 监听实体死亡事件
- 📊 批量修改多个统计数据
- 💬 可视化统计变化气泡
- 🎨 支持彩色数值显示
- 🎯 适用于经验值、分数、货币等奖励系统
- 🔧 支持正负数值修改

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `statsToModify` | `Dictionary[Stat, int]` | `{}` | 要修改的统计数据字典，键为Stat资源，值为修改量 |
| `shouldEmitBubble` | `bool` | `true` | 是否显示统计变化的文字气泡 |
| `shouldColorBubble` | `bool` | `true` | 是否为气泡文字着色（正数绿色，负数橙色） |
| `isEnabled` | `bool` | `true` | 是否启用组件功能 |

## 🔗 依赖组件

### 必需组件
```gdscript
@onready var healthComponent: HealthComponent
func getRequiredComponents() -> Array[Script]:
    return [HealthComponent]
```

**支持的HealthComponent类型**:
- HealthComponent - 基础生命值组件
- ShieldedHealthComponent - 护盾生命值组件
- 其他HealthComponent子类

## 🛠️ 主要方法

### 死亡处理
```gdscript
func onHealthComponent_healthDidZero() -> void
```
**描述**: 生命值归零时的处理方法，触发统计数据修改

### 统计修改
```gdscript
func modifyStats() -> void
```
**描述**: 执行所有统计数据的修改，包括气泡显示

## 🎯 使用示例

### 击杀怪物奖励

```gdscript
# Monster.gd - 怪物实体
extends Entity

func _ready():
    setupKillRewards()

func setupKillRewards():
    var statModifier = $StatModifierOnDeathComponent
    
    # 击杀这个怪物获得的奖励
    var playerXP = preload("res://Resources/Stats/PlayerXP.tres")
    var playerGold = preload("res://Resources/Stats/PlayerGold.tres")
    var killCount = preload("res://Resources/Stats/KillCount.tres")
    
    statModifier.statsToModify = {
        playerXP: 50,      # +50经验值
        playerGold: 25,    # +25金币
        killCount: 1       # +1击杀数
    }
```

### 玩家死亡惩罚

```gdscript
# PlayerEntity.gd
extends Entity

func _ready():
    setupDeathPenalty()

func setupDeathPenalty():
    var statModifier = $StatModifierOnDeathComponent
    var playerStats = $StatsComponent
    
    # 死亡惩罚
    statModifier.statsToModify = {
        playerStats.findStat("Gold"): -50,     # 失去50金币
        playerStats.findStat("Lives"): -1,     # 失去1生命
        playerStats.findStat("Deaths"): 1      # 死亡次数+1
    }
```

### Boss击杀特殊奖励

```gdscript
# BossEntity.gd
extends Entity

@export var bossLevel: int = 1
@export var playerStats: StatsComponent

func _ready():
    setupBossRewards()

func setupBossRewards():
    var statModifier = $StatModifierOnDeathComponent
    
    # 基于Boss等级的动态奖励
    var baseXP = 200 * bossLevel
    var baseGold = 100 * bossLevel
    
    statModifier.statsToModify = {
        playerStats.findStat("XP"): baseXP,
        playerStats.findStat("Gold"): baseGold,
        playerStats.findStat("BossKills"): 1
    }
    
    # 特殊视觉效果
    statModifier.shouldEmitBubble = true
    statModifier.shouldColorBubble = true
```

## 🔧 技术细节

### 统计修改执行
```gdscript
func modifyStats() -> void:
    var bubbleOffsetY: float = 0
    for stat in statsToModify:
        stat.value += statsToModify[stat]
        
        if shouldEmitBubble:
            createStatBubble(stat, bubbleOffsetY)
            bubbleOffsetY -= 10 # 气泡垂直间距
```

### 气泡创建
```gdscript
var labelSettings: LabelSettings = TextBubble.create(
    str(stat.displayName, "%+d" % stat.previousChange),
    parentEntity.get_parent(),  # 在实体父节点创建，避免随实体消失
    Vector2(parentEntity.position.x, parentEntity.position.y + bubbleOffsetY)
).label.label_settings

if shouldColorBubble:
    if   stat.previousChange > 0: labelSettings.font_color = Color.GREEN
    elif stat.previousChange < 0: labelSettings.font_color = Color.ORANGE
```

## ⚠️ 注意事项

1. **依赖组件**: 必须有HealthComponent或其子类
2. **字典配置**: statsToModify必须正确配置Stat资源
3. **气泡位置**: 气泡在父节点创建，避免随实体消失
4. **实际变化**: 使用previousChange获取实际修改量
5. **性能考虑**: 大量统计修改时注意性能影响
6. **空值检查**: 确保Stat资源有效

## 🔗 相关组件

- [HealthComponent](../Combat/HealthComponent.md) - 生命值组件
- [ShieldedHealthComponent](../Combat/ShieldedHealthComponent.md) - 护盾生命值
- [StatsComponent](../Data/StatsComponent.md) - 统计数据管理
- [StatModifierComponent](../Data/StatModifierComponent.md) - 定时统计修改
- [UI/TextBubble](../../UI/TextBubble.md) - 文字气泡显示

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 