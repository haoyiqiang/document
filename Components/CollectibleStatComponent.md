# CollectibleStatComponent API

> **继承关系**: Component > CollectibleComponent > CollectibleStatComponent  
> **对象类型**: 统计数据收集品

专门用于收集统计数据（Stat）的收集品组件。收集时可增减指定Stat的值，支持随机数值范围、防止过量收集、视觉反馈等高级功能。

## ✨ 主要特性

- 📊 修改Stat统计数据值
- 🎲 支持随机数值范围
- 🚫 防止过量收集（可选）
- 💬 自动视觉反馈气泡
- 🔄 智能重新检查机制
- 🎨 可自定义颜色和文本

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `stat` | `Stat` | `null` | 要修改的统计数据资源 |
| `statModifierMinimum` | `int` | `1` | 最小修改值（包含） |
| `statModifierMaximum` | `int` | `1` | 最大修改值（包含） |

### 收集控制
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `preventCollectionIfStatIsMax` | `bool` | `true` | 当Stat达到最大值时阻止收集 |

### 视觉效果
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldEmitBubble` | `bool` | `true` | 是否显示文字气泡 |
| `shouldColorBubble` | `bool` | `true` | 是否为气泡着色 |
| `shouldAppendStatName` | `bool` | `true` | 是否在气泡中显示Stat名称 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `previouslyDeniedCollector` | `CollectorComponent` | 之前被拒绝的收集器（用于重新检查） |

## 🎯 使用示例

### 基础生命值收集品

```gdscript
# Entity Scene Structure:
# └── HealthPotion (Area2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── CollectibleStatComponent
#         stat: [HealthStat Resource]
#         statModifierMinimum: 20
#         statModifierMaximum: 30
#         preventCollectionIfStatIsMax: true
```

### 金币收集品

```gdscript
# Entity Scene Structure:
# └── GoldCoin (Area2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── CollectibleStatComponent
#         stat: [GoldStat Resource]
#         statModifierMinimum: 1
#         statModifierMaximum: 5
#         preventCollectionIfStatIsMax: false  # 金币通常无上限
```

### 智能药剂组件

```gdscript
# SmartPotionComponent.gd
extends CollectibleStatComponent

@export var healingType: HealingType = HealingType.INSTANT
@export var effectDuration: float = 0.0
@export var effectStrength: float = 1.0

enum HealingType {
    INSTANT,
    OVER_TIME,
    PERCENTAGE
}

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> int:
    var modifier = calculateSmartModifier(collectorEntity)
    
    if debugMode:
        printLog(str("Smart healing: ", modifier, " (", healingType, ")"))
    
    applyHealingEffect(collectorEntity, modifier)
    
    # 创建增强的视觉效果
    if shouldEmitBubble:
        createEnhancedBubble(modifier)
    
    return modifier

func calculateSmartModifier(collectorEntity: Entity) -> int:
    var baseModifier = getRandomModifier()
    
    match healingType:
        HealingType.INSTANT:
            return baseModifier
        HealingType.OVER_TIME:
            # 分散到多个时间点
            return baseModifier
        HealingType.PERCENTAGE:
            # 基于当前生命值百分比
            var currentPercent = float(stat.value) / float(stat.max)
            return int(baseModifier * (1.0 - currentPercent))
    
    return baseModifier

func applyHealingEffect(collectorEntity: Entity, modifier: int):
    match healingType:
        HealingType.INSTANT:
            stat.value += modifier
        HealingType.OVER_TIME:
            startHealingOverTime(modifier)
        HealingType.PERCENTAGE:
            stat.value += modifier

func startHealingOverTime(totalHealing: int):
    var healingPerSecond = float(totalHealing) / effectDuration
    var timer = Timer.new()
    timer.wait_time = 0.1  # 每0.1秒治疗一次
    timer.timeout.connect(_on_healing_tick.bind(healingPerSecond * 0.1))
    add_child(timer)
    timer.start()
    
    # 设置总持续时间
    var durationTimer = Timer.new()
    durationTimer.wait_time = effectDuration
    durationTimer.one_shot = true
    durationTimer.timeout.connect(timer.queue_free)
    add_child(durationTimer)
    durationTimer.start()

func _on_healing_tick(healAmount: float):
    stat.value += int(healAmount)

func createEnhancedBubble(modifier: int):
    var bubbleText = getBubbleText(modifier)
    var bubbleColor = getBubbleColor()
    
    GameplayResourceBubble.createCustom(
        bubbleText,
        parentEntity.get_parent(),
        Vector2(parentEntity.position.x, parentEntity.position.y - 20),
        bubbleColor
    ).z_index = 100

func getBubbleText(modifier: int) -> String:
    var baseText = "+" + str(modifier)
    
    if shouldAppendStatName:
        baseText += " " + stat.name
    
    match healingType:
        HealingType.OVER_TIME:
            baseText += " (持续)"
        HealingType.PERCENTAGE:
            baseText += " (%)"
    
    return baseText

func getBubbleColor() -> Color:
    if not shouldColorBubble:
        return Color.WHITE
    
    match healingType:
        HealingType.INSTANT:
            return Color.GREEN
        HealingType.OVER_TIME:
            return Color.YELLOW
        HealingType.PERCENTAGE:
            return Color.CYAN
    
    return Color.WHITE
```

### 条件收集组件

```gdscript
# ConditionalStatComponent.gd
extends CollectibleStatComponent

@export var requiredLevel: int = 1
@export var requiredItems: Array[String] = []
@export var consumeRequiredItems: bool = false
@export var showRequirementHint: bool = true

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查等级要求
    if not checkLevelRequirement(collectorEntity):
        return false
    
    # 检查物品要求
    if not checkItemRequirements(collectorEntity):
        return false
    
    return true

func checkLevelRequirement(collectorEntity: Entity) -> bool:
    var levelComponent = collectorEntity.findFirstComponentOfClass(LevelComponent)
    if not levelComponent:
        return requiredLevel <= 1
    
    if levelComponent.currentLevel < requiredLevel:
        if showRequirementHint:
            showRequirementMessage("需要等级 " + str(requiredLevel))
        return false
    
    return true

func checkItemRequirements(collectorEntity: Entity) -> bool:
    if requiredItems.is_empty():
        return true
    
    var inventoryComponent = collectorEntity.findFirstComponentOfClass(InventoryComponent)
    if not inventoryComponent:
        if showRequirementHint:
            showRequirementMessage("需要物品: " + str(requiredItems))
        return false
    
    for itemName in requiredItems:
        if not inventoryComponent.hasItem(itemName):
            if showRequirementHint:
                showRequirementMessage("缺少物品: " + itemName)
            return false
    
    return true

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> int:
    # 消耗必需物品
    if consumeRequiredItems:
        consumeItems(collectorEntity)
    
    # 调用父类方法
    return super.onCollectible_didCollect(collectibleComponent, collectorEntity)

func consumeItems(collectorEntity: Entity):
    var inventoryComponent = collectorEntity.findFirstComponentOfClass(InventoryComponent)
    if not inventoryComponent:
        return
    
    for itemName in requiredItems:
        inventoryComponent.removeItem(itemName, 1)

func showRequirementMessage(message: String):
    print("Requirement not met: ", message)
    # 这里可以添加UI提示
```

### 稀有度系统组件

```gdscript
# RarityStatComponent.gd
extends CollectibleStatComponent

@export var rarity: Rarity = Rarity.COMMON
@export var rarityMultiplier: Dictionary = {
    Rarity.COMMON: 1.0,
    Rarity.UNCOMMON: 1.5,
    Rarity.RARE: 2.0,
    Rarity.EPIC: 3.0,
    Rarity.LEGENDARY: 5.0
}

enum Rarity {
    COMMON,
    UNCOMMON,
    RARE,
    EPIC,
    LEGENDARY
}

func getRandomModifier() -> int:
    var baseModifier = super.getRandomModifier()
    var multiplier = rarityMultiplier.get(rarity, 1.0)
    return int(baseModifier * multiplier)

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> int:
    var modifier = getRandomModifier()
    
    # 应用稀有度效果
    applyRarityEffects(collectorEntity, modifier)
    
    stat.value += modifier
    
    # 创建稀有度主题的气泡
    if shouldEmitBubble:
        createRarityBubble(modifier)
    
    return modifier

func applyRarityEffects(collectorEntity: Entity, modifier: int):
    match rarity:
        Rarity.UNCOMMON:
            # 额外的小效果
            addBonusEffect(collectorEntity, "speed_boost", 5.0)
        Rarity.RARE:
            # 中等额外效果
            addBonusEffect(collectorEntity, "damage_boost", 10.0)
        Rarity.EPIC:
            # 强大的额外效果
            addBonusEffect(collectorEntity, "shield", 15.0)
        Rarity.LEGENDARY:
            # 传奇效果
            addBonusEffect(collectorEntity, "invincibility", 3.0)

func addBonusEffect(collectorEntity: Entity, effectType: String, duration: float):
    print("Bonus effect applied: ", effectType, " for ", duration, " seconds")
    # 这里可以添加实际的效果逻辑

func createRarityBubble(modifier: int):
    var bubbleColor = getRarityColor()
    var bubbleText = "+" + str(modifier) + " " + stat.name
    
    if rarity != Rarity.COMMON:
        bubbleText += " (" + getRarityName() + ")"
    
    GameplayResourceBubble.createCustom(
        bubbleText,
        parentEntity.get_parent(),
        Vector2(parentEntity.position.x, parentEntity.position.y - 16),
        bubbleColor
    ).z_index = 100

func getRarityColor() -> Color:
    match rarity:
        Rarity.COMMON:
            return Color.WHITE
        Rarity.UNCOMMON:
            return Color.GREEN
        Rarity.RARE:
            return Color.BLUE
        Rarity.EPIC:
            return Color.PURPLE
        Rarity.LEGENDARY:
            return Color.GOLD
    return Color.WHITE

func getRarityName() -> String:
    match rarity:
        Rarity.COMMON:
            return "普通"
        Rarity.UNCOMMON:
            return "非凡"
        Rarity.RARE:
            return "稀有"
        Rarity.EPIC:
            return "史诗"
        Rarity.LEGENDARY:
            return "传奇"
    return "未知"
```

## 🔧 技术细节

### 收集处理流程
```gdscript
func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> int:
    var randomizedModifier: int = getRandomModifier()
    
    stat.value += randomizedModifier
    
    # 创建视觉指示器
    if shouldEmitBubble:
        GameplayResourceBubble.createForStat(
            stat, 
            parentEntity.get_parent(), 
            Vector2(parentEntity.position.x, parentEntity.position.y - 16), 
            shouldAppendStatName, 
            shouldColorBubble
        ).z_index = 100
    
    return randomizedModifier
```

### 智能重新检查机制
```gdscript
func onstat_changed() -> void:
    # 当Stat从最大值减少时，允许之前被拒绝的收集器重新收集
    if preventCollectionIfStatIsMax and previouslyDeniedCollector and stat.value < stat.max:
        previouslyDeniedCollector.handleCollection(self)

func onAreaExited(area: Area2D) -> void:
    # 收集器离开时清除拒绝记录
    var collectorComponent: CollectorComponent = area.get_node(^".") as CollectorComponent
    if previouslyDeniedCollector == collectorComponent: 
        previouslyDeniedCollector = null
```

## ⚠️ 注意事项

1. **继承关系**: 继承自CollectibleComponent，具有所有基础收集功能
2. **Stat资源**: 必须分配有效的Stat资源才能正常工作
3. **碰撞设置**: 需要正确设置CollisionObject2D的layer和mask
4. **视觉反馈**: 气泡在收集品的父节点生成，而非收集器
5. **重新检查**: 依赖正确的Area2D信号连接
6. **性能考虑**: 频繁的重新检查可能影响性能

## 🔗 相关组件

- [CollectibleComponent](CollectibleComponent.md) - 基础收集品组件
- [CollectorComponent](./CollectorComponent.md) - 收集器组件
- [StatsComponent](./StatsComponent.md) - 统计数据管理
- [InventoryComponent](./InventoryComponent.md) - 物品容器组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 