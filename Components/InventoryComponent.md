# InventoryComponent

## 概述
`InventoryComponent` 是一个物品容器管理组件，用于存储和管理 `InventoryItem` 对象。支持容量限制、重量限制、重复检查等功能。

**继承关系：** `InventoryComponent` → `Component` → `Node`

## 主要特性
- 📦 灵活的物品存储系统
- ⚖️ 重量限制管理
- 🔢 物品数量限制
- 🚫 可选的重复防护
- 🔄 动态启用/禁用
- 📡 物品变化信号通知

## 导出属性

### `items: Array[InventoryItem]`
存储的物品数组。

**类型：** `Array[InventoryItem]`  
**重要：** 不要直接修改！使用提供的方法确保限制检查  
**用途：** 包含所有当前存储的物品

### `maximumItems: int = 100`
最大物品数量限制。

**类型：** `int`  
**范围：** `1 - 1000`  
**默认值：** `100`  
**注意：** 减少此值不会自动移除超出的物品

### `maximumWeight: float = 100`
最大重量限制。

**类型：** `float`  
**范围：** `1 - 1000`  
**默认值：** `100`  
**注意：** 减少此值不会自动移除超重物品

### `shouldPreventDuplicates: bool = false`
是否阻止重复物品添加。

**类型：** `bool`  
**默认值：** `false`  
**行为：** `true` 时拒绝添加已存在的物品

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**影响：** 禁用时无法添加物品

## 状态属性

### `totalWeight: float`
当前总重量。

**类型：** `float`  
**维护：** 通过 `recalculateWeight()` 自动更新  
**用途：** 检查重量限制

## 信号

### `didAddItem(item: InventoryItem)`
成功添加物品时触发。

**参数：** `item` - 被添加的物品  
**用途：** 通知UI更新或触发音效

### `didRemovetem(item: InventoryItem)`
成功移除物品时触发。

**参数：** `item` - 被移除的物品  
**注意：** 信号名称有拼写错误（应为didRemoveItem）

## 核心方法

### `addItems(newItems: Array[InventoryItem]) -> Array[InventoryItem]`
批量添加物品。

**参数：** `newItems` - 要添加的物品数组  
**返回：** 成功添加的物品数组  
**特性：** 遇到限制时部分添加，返回成功部分

### `addItem(newItem: InventoryItem) -> bool`
添加单个物品。

**参数：** `newItem` - 要添加的物品  
**返回：** `true` 表示添加成功  
**检查顺序：**
1. 组件是否启用
2. 数量是否超限
3. 重量是否超限
4. 是否重复（如果启用检查）

### `removeItems(itemsToRemove: Array[InventoryItem]) -> Array[InventoryItem]`
批量移除物品。

**参数：** `itemsToRemove` - 要移除的物品数组  
**返回：** 成功移除的物品数组  
**注意：** 移除是否受 `isEnabled` 影响待定（TBD）

### `removeItem(itemToRemove: InventoryItem) -> bool`
移除单个物品。

**参数：** `itemToRemove` - 要移除的物品  
**返回：** `true` 表示物品存在并已移除  
**行为：** 自动更新总重量

### `recalculateWeight() -> float`
重新计算总重量。

**返回：** 更新后的总重量  
**用途：** 手动校正重量计算或调试

## 使用示例

### 基本物品管理
```gdscript
# 获取背包组件
var inventory = $InventoryComponent

# 创建物品
var sword = load("res://Items/IronSword.tres")
var potion = load("res://Items/HealthPotion.tres")

# 添加物品
if inventory.addItem(sword):
    print("剑已添加到背包")
else:
    print("无法添加剑：背包已满或其他限制")

# 移除物品
if inventory.removeItem(potion):
    print("药水已使用")
```

### 批量操作
```gdscript
# 拾取多个物品
func pickupLoot(lootItems: Array[InventoryItem]):
    var inventory = $InventoryComponent
    var addedItems = inventory.addItems(lootItems)
    
    print("添加了 ", addedItems.size(), " / ", lootItems.size(), " 个物品")
    
    # 显示无法添加的物品
    var failedItems = []
    for item in lootItems:
        if not addedItems.has(item):
            failedItems.append(item)
    
    if not failedItems.is_empty():
        print("无法添加: ", failedItems)
```

### 容量管理
```gdscript
# 检查背包状态
func checkInventoryStatus():
    var inventory = $InventoryComponent
    
    print("物品数量: ", inventory.items.size(), "/", inventory.maximumItems)
    print("重量: ", inventory.totalWeight, "/", inventory.maximumWeight)
    
    # 计算剩余容量
    var spaceLeft = inventory.maximumItems - inventory.items.size()
    var weightLeft = inventory.maximumWeight - inventory.totalWeight
    
    print("剩余空间: ", spaceLeft, " 个物品, ", weightLeft, " 重量")

# 自动整理背包
func organizeInventory():
    var inventory = $InventoryComponent
    
    # 按重量排序（轻的在前）
    inventory.items.sort_custom(func(a, b): return a.weight < b.weight)
    
    # 或按价值排序
    # inventory.items.sort_custom(func(a, b): return a.value > b.value)
```

### 物品类型管理
```gdscript
# 查找特定类型物品
func findItemsByType(itemType: String) -> Array[InventoryItem]:
    var inventory = $InventoryComponent
    var foundItems: Array[InventoryItem] = []
    
    for item in inventory.items:
        if item.type == itemType:
            foundItems.append(item)
    
    return foundItems

# 检查是否有特定物品
func hasItem(itemName: String) -> bool:
    var inventory = $InventoryComponent
    
    for item in inventory.items:
        if item.name == itemName:
            return true
    
    return false

# 使用第一个可用的恢复物品
func useFirstHealingItem():
    var inventory = $InventoryComponent
    var healingItems = findItemsByType("healing")
    
    if not healingItems.is_empty():
        var item = healingItems[0]
        inventory.removeItem(item)
        # 应用恢复效果
        applyHealingEffect(item)
```

### 信号处理
```gdscript
func _ready():
    var inventory = $InventoryComponent
    
    # 连接信号
    inventory.didAddItem.connect(_on_item_added)
    inventory.didRemovetem.connect(_on_item_removed)

func _on_item_added(item: InventoryItem):
    print("获得物品: ", item.name)
    # 播放获得音效
    $SFX/ItemPickup.play()
    # 更新UI
    updateInventoryUI()
    # 显示通知
    showItemNotification("获得: " + item.name)

func _on_item_removed(item: InventoryItem):
    print("失去物品: ", item.name)
    # 播放使用音效
    $SFX/ItemUse.play()
    # 更新UI
    updateInventoryUI()
```

### 高级功能
```gdscript
# 物品交换系统
func tradeItems(targetInventory: InventoryComponent, 
               giveItems: Array[InventoryItem], 
               receiveItems: Array[InventoryItem]) -> bool:
    
    # 检查目标是否能接收我们的物品
    var canGive = targetInventory.addItems(giveItems)
    if canGive.size() != giveItems.size():
        return false  # 无法完成交易
    
    # 检查我们是否能接收对方的物品
    var canReceive = addItems(receiveItems)
    if canReceive.size() != receiveItems.size():
        # 回滚已添加到目标的物品
        targetInventory.removeItems(canGive)
        return false
    
    # 移除我们给出的物品
    removeItems(giveItems)
    return true

# 自动丢弃低价值物品
func autoDropLowValueItems():
    var inventory = $InventoryComponent
    var itemsToRemove: Array[InventoryItem] = []
    
    # 如果背包已满，丢弃价值最低的物品
    if inventory.items.size() >= inventory.maximumItems:
        inventory.items.sort_custom(func(a, b): return a.value < b.value)
        itemsToRemove.append(inventory.items[0])  # 最低价值的物品
    
    # 移除选中的物品
    for item in itemsToRemove:
        inventory.removeItem(item)
        print("自动丢弃: ", item.name)
```

## 设计模式

### 容量管理
- **双重限制：** 同时检查数量和重量限制
- **渐进添加：** 批量操作时尽可能多地添加物品
- **自动维护：** 重量自动计算和更新

### 信号驱动
- **即时通知：** 物品变化立即发出信号
- **UI集成：** 信号直接驱动界面更新
- **音效触发：** 通过信号播放合适的音效

### 灵活配置
- **可选限制：** 可以禁用重复检查
- **动态控制：** 运行时启用/禁用
- **参数调整：** 容量限制可以调整

## 注意事项

1. **直接修改：** 避免直接操作items数组
2. **信号拼写：** didRemovetem信号名有拼写错误
3. **重量一致性：** 使用recalculateWeight()确保准确性
4. **限制变更：** 减少限制不会自动移除超出物品
5. **禁用状态：** 移除操作是否受isEnabled影响尚未确定

## 相关组件
- `CollectibleComponent` - 可拾取物品组件
- `StatsComponent` - 可能包含重量相关统计
- `InteractionComponent` - 物品使用交互
- `DropOnDeathComponent` - 死亡时掉落物品
- UI相关组件 - 背包界面显示 