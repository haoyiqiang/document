# FactionComponent

## 概述
`FactionComponent` 定义实体所属的游戏阵营，用于区分玩家、盟友、敌人等不同群体。被 `DamageComponent` 和 `DamageReceivingComponent` 使用，以及潜在的AI NPC交互系统。

**继承关系：** `FactionComponent` → `Component` → `Node`

## 主要特性
- 🏳️ 支持多阵营归属
- ⚔️ 智能的敌友识别系统
- 🎯 基于位标志的高效比较
- 🔄 灵活的阵营组合
- 🛡️ 自动伤害过滤机制

## 阵营常量

### `Factions` 枚举
```gdscript
enum Factions {
    neutral = 1,
    players = 2,
    playerAllies = 3,
    enemies = 4,
    hazards = 5
}
```

### `factionStrings` 常量
阵营名称字符串数组，用于导出属性显示：
```gdscript
const factionStrings: PackedStringArray = [
    "neutral", 
    "players", 
    "playerAllies", 
    "enemies",
    "hazards"
]
```

## 导出属性

### `factions: int`
实体所属的阵营标志位组合。

**类型：** `int` (位标志)  
**默认值：** `Factions.neutral`  
**显示方式：** `@export_flags` 多选框  
**特性：** 
- 支持多个阵营同时归属
- 使用位运算进行高效比较
- 共享任意阵营的实体为盟友关系

## 核心方法

### `checkAlliance(otherFactions: int) -> bool`
检查与其他阵营是否为盟友关系。

**参数：** `otherFactions` - 要比较的阵营标志位  
**返回：** `true` 表示有任意阵营匹配（盟友关系）  
**算法：** 使用位运算 `&` 检查是否有共同阵营

**示例：**
```gdscript
# [player, playerAllies] vs [playerAllies, enemies] → true
var isAlly = checkAlliance(otherFactionComponent.factions)
```

### `checkOpposition(otherFactions: int) -> bool`
检查与其他阵营是否为敌对关系。

**参数：** `otherFactions` - 要比较的阵营标志位  
**返回：** `true` 表示没有任何阵营匹配（敌对关系）  
**算法：** `checkAlliance()` 的逻辑反转

**示例：**
```gdscript
# [player, playerAllies] vs [enemies] → true
var isEnemy = checkOpposition(otherFactionComponent.factions)
```

## 使用示例

### 基本阵营设置
```gdscript
# 设置玩家阵营
var playerFaction = $FactionComponent
playerFaction.factions = FactionComponent.Factions.players

# 设置混合阵营（玩家+盟友）
var allyFaction = $AllyFactionComponent
allyFaction.factions = FactionComponent.Factions.players | FactionComponent.Factions.playerAllies
```

### 伤害系统集成
```gdscript
# 在DamageComponent中的使用
func shouldDealDamage(targetEntity: Entity) -> bool:
    var myFaction = parentEntity.findFirstComponent(FactionComponent)
    var targetFaction = targetEntity.findFirstComponent(FactionComponent)
    
    if myFaction and targetFaction:
        # 只有敌对阵营才造成伤害
        return myFaction.checkOpposition(targetFaction.factions)
    
    # 没有阵营信息时默认造成伤害
    return true
```

### AI行为决策
```gdscript
# AI决策系统示例
func evaluateTarget(targetEntity: Entity) -> String:
    var myFaction = $FactionComponent
    var targetFaction = targetEntity.findFirstComponent(FactionComponent)
    
    if not targetFaction:
        return "neutral"
    
    if myFaction.checkAlliance(targetFaction.factions):
        return "ally"
    elif myFaction.checkOpposition(targetFaction.factions):
        return "enemy"
    else:
        return "neutral"
```

### 动态阵营切换
```gdscript
# 角色背叛系统
func betrayAllies():
    var faction = $FactionComponent
    # 从玩家阵营切换到敌对阵营
    faction.factions = FactionComponent.Factions.enemies

# 招募敌人
func recruitEnemy():
    var faction = $FactionComponent
    # 添加玩家盟友阵营，保留原阵营
    faction.factions |= FactionComponent.Factions.playerAllies
```

### 复杂阵营关系
```gdscript
# 三方混战场景
func setupComplexFactions():
    # 玩家角色：纯玩家阵营
    $Player.get_component(FactionComponent).factions = FactionComponent.Factions.players
    
    # 守卫：玩家盟友
    $Guard.get_component(FactionComponent).factions = FactionComponent.Factions.playerAllies
    
    # 雇佣兵：玩家+盟友（不攻击玩家和守卫）
    $Mercenary.get_component(FactionComponent).factions = 
        FactionComponent.Factions.players | FactionComponent.Factions.playerAllies
    
    # 匪徒：纯敌对
    $Bandit.get_component(FactionComponent).factions = FactionComponent.Factions.enemies
    
    # 陷阱：危险物，攻击所有人
    $Trap.get_component(FactionComponent).factions = FactionComponent.Factions.hazards
```

## 设计模式

### 位标志优势
- **性能：** 位运算比字符串或数组比较更快
- **灵活性：** 单个实体可属于多个阵营
- **扩展性：** 容易添加新阵营类型

### 阵营继承关系
推荐的阵营层次结构：
```
neutral (1)     - 中立，不攻击任何人
├─ players (2)  - 玩家控制的角色
├─ playerAllies (3) - 玩家的NPC盟友
├─ enemies (4)  - 敌对实体
└─ hazards (5)  - 环境危险（攻击所有人）
```

### 与伤害系统的配合
```gdscript
# 标准伤害检查模式
if not sourceFaction or not targetFaction:
    # 没有阵营信息，默认造成伤害
    dealDamage()
elif sourceFaction.checkOpposition(targetFaction.factions):
    # 敌对阵营，造成伤害
    dealDamage()
else:
    # 盟友阵营，不造成伤害
    pass
```

## 实际应用场景

### 战术游戏
- **玩家军队：** `players | playerAllies`
- **敌方军队：** `enemies`
- **中立商人：** `neutral`
- **环境陷阱：** `hazards`

### RPG游戏
- **主角：** `players`
- **队友：** `players | playerAllies`
- **城镇守卫：** `playerAllies`
- **怪物：** `enemies`
- **野生动物：** `neutral` 或 `enemies`

### 生存游戏
- **玩家：** `players`
- **玩家建筑：** `players | playerAllies`
- **野生动物：** `neutral`
- **僵尸：** `enemies`
- **毒气云：** `hazards`

## 注意事项

1. **默认行为：** 没有阵营信息时通常默认造成伤害
2. **位标志限制：** 最多支持32个不同阵营（int大小限制）
3. **性能考虑：** 位运算比其他比较方式更高效
4. **设计一致性：** 确保项目中的阵营逻辑一致
5. **扩展规划：** 预留阵营ID，便于后续扩展

## 相关组件
- `DamageComponent` - 使用阵营信息决定是否造成伤害
- `DamageReceivingComponent` - 使用阵营信息决定是否接受伤害
- `HealthComponent` - 被伤害系统影响的生命值组件
- AI相关组件 - 可能使用阵营信息做行为决策