# CollectibleInventoryComponent API

> **继承关系**: Component > CollectibleComponent > CollectibleInventoryComponent  
> **收集类型**: 背包物品收集

CollectibleComponent的特化版本，专门用于将InventoryItem添加到收集者实体的InventoryComponent中。提供完整的背包物品收集系统，支持重复检查和视觉指示器。

## ✨ 主要特性

- 🎒 自动添加到背包系统
- 🔄 重复物品检查机制
- 💬 可选的收集视觉指示器
- 🎯 完整的收集条件验证
- 📦 基于CallablePayload的载荷系统

## 📊 导出属性

### 背包设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `inventoryItem` | `InventoryItem` | `null` | 要添加到背包的物品 |
| `preventCollectionIfDuplicateItem` | `bool` | `true` | 如果背包中已有相同物品则拒绝收集 |
| `shouldDisplayIndicator` | `bool` | `true` | 是否显示收集指示器 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `CollectibleComponent` | **基类** | 提供基础收集功能 |

### 收集者要求
| 组件 | 关系 | 描述 |
|------|------|------|
| `InventoryComponent` | **收集者必需** | 收集者实体必须有背包组件 |
| `CollectorComponent` | **收集者必需** | 收集者实体必须有收集器组件 |

## 🎯 使用示例

### 基础背包物品

```gdscript
# Entity Scene Structure:
# └── Collectible (Area2D)
#     ├── CollisionShape2D
#     ├── Sprite2D
#     └── CollectibleInventoryComponent
#         inventoryItem: preload("res://Items/HealthPotion.tres")
```

### 稀有物品收集

```gdscript
# RareItemCollectible.gd
extends CollectibleInventoryComponent

@export var rarity: String = "common"
@export var playSpecialEffect: bool = false

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查稀有度要求
    if rarity == "legendary":
        var playerLevel = getPlayerLevel(collectorEntity)
        if playerLevel < 10:
            showMessage("You need to be level 10+ to collect legendary items!")
            return false
    
    return true

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> InventoryItem:
    var result = super.onCollectible_didCollect(collectibleComponent, collectorEntity)
    
    if result and playSpecialEffect:
        createSpecialEffect(collectorEntity)
    
    return result

func getPlayerLevel(entity: Entity) -> int:
    var statsComponent = entity.findFirstComponentSubclass(StatsComponent)
    if statsComponent:
        return statsComponent.getStat("level").currentValue
    return 1

func createSpecialEffect(entity: Entity):
    match rarity:
        "rare":
            TextBubble.create("RARE ITEM!", entity).label.label_settings.font_color = Color.BLUE
        "epic":
            TextBubble.create("EPIC ITEM!", entity).label.label_settings.font_color = Color.PURPLE
        "legendary":
            TextBubble.create("LEGENDARY!", entity).label.label_settings.font_color = Color.GOLD

func showMessage(text: String):
    print(text)
```

### 钥匙和门系统

```gdscript
# KeyCollectible.gd
extends CollectibleInventoryComponent

@export var keyType: String = ""
@export var doorId: String = ""

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> InventoryItem:
    var result = super.onCollectible_didCollect(collectibleComponent, collectorEntity)
    
    if result:
        # 记录钥匙获得状态
        GameState.setKeyObtained(keyType, true)
        
        # 解锁对应的门
        if doorId:
            unlockDoor(doorId)
        
        # 特殊视觉效果
        TextBubble.create("Key Obtained: " + keyType.capitalize(), collectorEntity).label.label_settings.font_color = Color.YELLOW
    
    return result

func unlockDoor(id: String):
    # 找到对应的门并解锁
    var doors = get_tree().get_nodes_in_group("doors")
    for door in doors:
        if door.has_method("getDoorId") and door.getDoorId() == id:
            door.unlock()
            print("Door unlocked: ", id)
```

### 货币收集系统

```gdscript
# CoinCollectible.gd
extends CollectibleInventoryComponent

@export var coinValue: int = 1
@export var coinType: String = "gold"

func _ready():
    super._ready()
    # 创建对应的货币物品
    if not inventoryItem:
        inventoryItem = createCoinItem()

func createCoinItem() -> InventoryItem:
    var coin = InventoryItem.new()
    coin.displayName = coinType.capitalize() + " Coin"
    coin.description = "Worth " + str(coinValue) + " " + coinType
    coin.stackable = true
    coin.maxStackSize = 999
    return coin

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> InventoryItem:
    # 检查是否已有相同货币
    var inventoryComponent = collectorEntity.findFirstComponentSubclass(InventoryComponent)
    var existingCoin = findExistingCoin(inventoryComponent)
    
    if existingCoin:
        # 增加现有货币的数量
        existingCoin.quantity += coinValue
        updateCoinDisplay(collectorEntity, coinValue)
        return existingCoin
    else:
        # 添加新的货币物品
        inventoryItem.quantity = coinValue
        var result = super.onCollectible_didCollect(collectibleComponent, collectorEntity)
        return result

func findExistingCoin(inventoryComponent: InventoryComponent) -> InventoryItem:
    for item in inventoryComponent.items:
        if item.displayName.contains(coinType.capitalize() + " Coin"):
            return item
    return null

func updateCoinDisplay(entity: Entity, amount: int):
    TextBubble.create("+" + str(amount) + " " + coinType, entity).label.label_settings.font_color = Color.GOLD
```

### 条件收集组件

```gdscript
# ConditionalCollectible.gd
extends CollectibleInventoryComponent

@export var requiredItems: Array[InventoryItem] = []
@export var consumeRequiredItems: bool = false
@export var questRequirement: String = ""

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查任务要求
    if questRequirement and not GameState.hasQuest(questRequirement):
        showMessage("You need to accept the quest: " + questRequirement)
        return false
    
    # 检查必需物品
    if not requiredItems.is_empty():
        var inventoryComponent = collectorEntity.findFirstComponentSubclass(InventoryComponent)
        for requiredItem in requiredItems:
            if not inventoryComponent.hasItem(requiredItem):
                showMessage("You need: " + requiredItem.displayName)
                return false
    
    return true

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> InventoryItem:
    # 消耗必需物品
    if consumeRequiredItems and not requiredItems.is_empty():
        var inventoryComponent = collectorEntity.findFirstComponentSubclass(InventoryComponent)
        for requiredItem in requiredItems:
            inventoryComponent.removeItem(requiredItem)
    
    var result = super.onCollectible_didCollect(collectibleComponent, collectorEntity)
    
    # 更新任务进度
    if questRequirement and result:
        GameState.updateQuestProgress(questRequirement, "collected_" + inventoryItem.displayName)
    
    return result

func showMessage(text: String):
    TextBubble.create(text, parentEntity).label.label_settings.font_color = Color.RED
```

## 🔧 技术细节

### 收集条件检查
```gdscript
func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    # 1. 调用基类检查
    if not super.checkCollectionConditions(collectorEntity, collectorComponent): 
        return false

    # 2. 检查InventoryComponent
    var inventoryComponent = collectorEntity.components.get(&"InventoryComponent")
    if not inventoryComponent:
        return false
    
    # 3. 检查重复物品
    if preventCollectionIfDuplicateItem and inventoryComponent.items.has(inventoryItem):
        return false
    
    return true
```

### 载荷系统覆盖
```gdscript
func _ready() -> void:
    self.payload = CallablePayload.new()
    (self.payload as CallablePayload).payloadCallable = self.onCollectible_didCollect
```

### 收集处理
```gdscript
func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> InventoryItem:
    var inventoryComponent = collectorEntity.components.get(&"InventoryComponent")
    var result = inventoryComponent.addItem(self.inventoryItem)
    
    if result and shouldDisplayIndicator:
        TextBubble.create("GET " + inventoryItem.displayName.capitalize(), collectorEntity)
    
    return inventoryItem if result else null
```

## ⚠️ 注意事项

1. **InventoryComponent依赖**: 收集者实体必须有InventoryComponent
2. **重复检查**: 可以在组件级别和InventoryComponent级别都进行重复检查
3. **载荷覆盖**: 组件自动覆盖payload为CallablePayload
4. **视觉指示器**: 默认显示"GET [物品名]"的文字气泡
5. **继承链**: 继承自CollectibleComponent，可使用所有基类功能

## 🔗 相关组件

- [CollectibleComponent](CollectibleComponent.md) - 基础收集组件
- [InventoryComponent](../Gameplay/InventoryComponent.md) - 背包管理组件
- [CollectorComponent](CollectorComponent.md) - 收集器组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 