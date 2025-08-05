# CollectibleComponent

## 概述
`CollectibleComponent` 是一个代表可被拾取物品的组件，支持与拥有`CollectorComponent`的角色实体进行交互。提供载荷系统来定义物品被拾取后的具体效果，如添加子节点、执行脚本或发出信号。

**继承关系：** `CollectibleComponent` → `Component` → `Node`

## 主要特性
- 📦 可拾取物品系统
- 🎯 载荷效果机制
- 🔄 条件检查系统
- 🗑️ 自动清理机制
- 🎮 可继承定制逻辑
- ⚡ 性能优化设计

## 设计理念
- **性能优先：** 默认禁用监听以节省性能
- **协作模式：** 由CollectorComponent主导交互流程
- **条件驱动：** 双方都可以设置收集条件
- **效果灵活：** 通过载荷系统支持各种收集效果

## 导出属性

### `payload: Payload`
物品的内容载荷，定义拾取后的实际效果。

**类型：** `Payload`  
**用途：** 执行收集后的游戏效果  
**参数：** 
- `source` - 此CollectibleComponent
- `target` - CollectorComponent的父实体

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`

## 状态属性

### `previousPayloadResult: bool = false`
存储载荷执行的最近结果。

**类型：** `bool`  
**用途：** 用于决定是否移除物品实体  
**说明：** 成功收集后用于触发清理机制

## 信号

### `didCollideCollector(collectorComponent: CollectorComponent)`
与收集器碰撞时发出（基类未使用，供子类使用）。

**参数：** `collectorComponent` - 碰撞的收集器组件  
**用途：** 子类可以监听此信号进行特殊处理

### `willBeCollected(collectorEntity: Entity)`
即将被收集时发出。

**参数：** `collectorEntity` - 执行收集的实体  
**时机：** 条件检查通过后，执行收集前

### `didDenyCollection(collectorEntity: Entity)`
拒绝收集时发出。

**参数：** `collectorEntity` - 尝试收集的实体  
**时机：** 条件检查失败时

### `willBeFreed`
即将被删除时发出。

**用途：** 通知其他系统物品即将消失

## 核心方法

### `requestToCollect(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool`
处理收集请求。

**参数：**
- `collectorEntity` - 请求收集的实体
- `collectorComponent` - 收集器组件

**返回：** 是否批准收集  
**流程：**
1. 检查组件是否启用
2. 调用收集条件检查
3. 发出相应信号
4. 返回检查结果

### `collect(collectorComponent: CollectorComponent) -> Variant`
执行实际的收集操作。

**参数：** `collectorComponent` - 执行收集的组件  
**返回：** 载荷执行结果  
**流程：**
1. 执行载荷效果
2. 检查移除条件
3. 必要时删除物品实体

## 虚拟方法（可重写）

### `checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool`
检查收集条件。

**参数：**
- `collectorEntity` - 收集实体
- `collectorComponent` - 收集器组件

**默认实现：** 返回`isEnabled`  
**用途：** 子类可重写以实现自定义收集条件

### `checkRemovalConditions() -> bool`
检查移除条件。

**返回：** 是否应该移除物品  
**默认实现：** 返回`previousPayloadResult`  
**用途：** 子类可重写以实现自定义移除逻辑

### `onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> Variant`
CallablePayload调用的收集回调。

**参数：**
- `collectibleComponent` - 当前收集品组件
- `collectorEntity` - 收集者实体

**必须重写：** 子类必须实现此方法  
**用途：** 当载荷类型为CallablePayload时被调用

## 使用示例

### 基本收集品设置
```gdscript
# 设置简单的金币收集品
func setupCoinCollectible():
    var collectibleComponent = $CollectibleComponent
    
    # 创建统计修改载荷
    var coinPayload = StatModifierPayload.new()
    coinPayload.statName = "gold"
    coinPayload.modificationValue = 10
    
    collectibleComponent.payload = coinPayload
    collectibleComponent.isEnabled = true

# 连接收集信号
func _ready():
    var collectibleComponent = $CollectibleComponent
    collectibleComponent.willBeCollected.connect(_on_will_be_collected)
    collectibleComponent.willBeFreed.connect(_on_will_be_freed)

func _on_will_be_collected(collectorEntity: Entity):
    print("金币即将被 ", collectorEntity.name, " 收集")
    # 播放收集音效
    $SFX/CoinPickup.play()

func _on_will_be_freed():
    print("金币消失")
    # 显示收集特效
    $VFX/CollectEffect.emit()
```

### 血瓶收集品
```gdscript
# 血瓶收集品，检查玩家血量
class_name HealthPotionCollectible
extends CollectibleComponent

@export var healAmount: int = 50

func _ready():
    super._ready()
    
    # 设置治疗载荷
    var healPayload = CallablePayload.new()
    healPayload.callable = onCollectible_didCollect
    self.payload = healPayload

# 只有受伤的角色才能拾取血瓶
func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查收集者是否有生命值组件且未满血
    var healthComponent = collectorEntity.get_node("HealthComponent")
    if not healthComponent:
        return false
    
    if healthComponent.currentHealth >= healthComponent.maxHealth:
        print("血量已满，无法使用血瓶")
        return false
    
    return true

# 执行治疗效果
func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> Variant:
    var healthComponent = collectorEntity.get_node("HealthComponent")
    if healthComponent:
        healthComponent.heal(healAmount)
        print("治疗了 ", healAmount, " 点生命值")
        return true
    return false
```

### 钥匙收集品
```gdscript
# 钥匙收集品，添加到物品容器
class_name KeyCollectible
extends CollectibleComponent

@export var keyType: String = "red_key"
@export var keyItem: InventoryItem

func _ready():
    super._ready()
    
    # 设置物品载荷
    var itemPayload = InventoryItemPayload.new()
    itemPayload.inventoryItem = keyItem
    itemPayload.quantity = 1
    
    self.payload = itemPayload

# 检查物品容器是否有空间
func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查物品容器空间
    var inventoryComponent = collectorEntity.get_node("InventoryComponent")
    if not inventoryComponent:
        print("收集者没有物品容器")
        return false
    
    if not inventoryComponent.canAddItem(keyItem, 1):
        print("物品容器已满")
        return false
    
    return true
```

### 武器收集品
```gdscript
# 武器收集品，替换当前武器
class_name WeaponCollectible
extends CollectibleComponent

@export var weaponScene: PackedScene
@export var weaponName: String = "Sword"

func _ready():
    super._ready()
    
    # 设置组件载荷
    var weaponPayload = ComponentPayload.new()
    weaponPayload.componentScene = weaponScene
    weaponPayload.shouldReplaceExisting = true
    
    self.payload = weaponPayload

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查是否可以装备武器
    var canEquip = checkWeaponCompatibility(collectorEntity)
    if not canEquip:
        print("无法装备此武器")
        return false
    
    return true

func checkWeaponCompatibility(entity: Entity) -> bool:
    # 检查职业、等级等条件
    var playerClass = entity.get("playerClass")
    if playerClass == "Mage" and weaponName.contains("Sword"):
        return false  # 法师不能使用剑
    
    return true
```

### 经验宝石
```gdscript
# 经验宝石，给予经验值
class_name ExperienceGem
extends CollectibleComponent

@export var experienceAmount: int = 100
@export var levelRange: Vector2i = Vector2i(1, 99)  # 适用等级范围

func _ready():
    super._ready()
    
    # 设置信号载荷
    var expPayload = SignalPayload.new()
    expPayload.signalName = "experience_gained"
    expPayload.signalArgs = [experienceAmount]
    
    self.payload = expPayload

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查等级范围
    var level = collectorEntity.get("level", 1)
    if level < levelRange.x or level > levelRange.y:
        print("等级不符合要求")
        return false
    
    return true

# 根据收集者等级调整经验值
func collect(collectorComponent: CollectorComponent) -> Variant:
    var collectorEntity = collectorComponent.parentEntity
    var level = collectorEntity.get("level", 1)
    
    # 高等级获得的经验减少
    var adjustedExp = experienceAmount * (1.0 - level * 0.01)
    adjustedExp = max(adjustedExp, experienceAmount * 0.1)  # 最少10%
    
    # 更新载荷参数
    if payload is SignalPayload:
        payload.signalArgs[0] = int(adjustedExp)
    
    return super.collect(collectorComponent)
```

### 带冷却的收集品
```gdscript
# 可重复收集的治疗泉水
class_name HealingFountain
extends CollectibleComponent

@export var cooldownTime: float = 5.0
@export var healAmount: int = 20

var lastCollectionTime: float = 0.0

func _ready():
    super._ready()
    
    var healPayload = CallablePayload.new()
    healPayload.callable = onCollectible_didCollect
    self.payload = healPayload

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查冷却时间
    var currentTime = Time.get_time_dict_from_system()["hour"] * 3600.0 + \
                      Time.get_time_dict_from_system()["minute"] * 60.0 + \
                      Time.get_time_dict_from_system()["second"]
    
    if currentTime - lastCollectionTime < cooldownTime:
        print("治疗泉水冷却中，剩余: ", cooldownTime - (currentTime - lastCollectionTime), "秒")
        return false
    
    # 检查是否需要治疗
    var healthComponent = collectorEntity.get_node("HealthComponent")
    if not healthComponent or healthComponent.currentHealth >= healthComponent.maxHealth:
        print("无需治疗")
        return false
    
    return true

# 治疗效果但不移除物品
func checkRemovalConditions() -> bool:
    return false  # 不移除，可重复使用

func onCollectible_didCollect(collectibleComponent: CollectibleComponent, collectorEntity: Entity) -> Variant:
    var healthComponent = collectorEntity.get_node("HealthComponent")
    if healthComponent:
        healthComponent.heal(healAmount)
        lastCollectionTime = Time.get_time_dict_from_system()["hour"] * 3600.0 + \
                            Time.get_time_dict_from_system()["minute"] * 60.0 + \
                            Time.get_time_dict_from_system()["second"]
        
        # 播放治疗特效
        $HealEffect.play()
        return true
    
    return false
```

### 条件门卡
```gdscript
# 需要特定物品才能收集的门卡
class_name KeyCardCollectible
extends CollectibleComponent

@export var requiredKeyItem: InventoryItem
@export var consumeKey: bool = true

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 检查是否拥有所需钥匙
    var inventoryComponent = collectorEntity.get_node("InventoryComponent")
    if not inventoryComponent:
        print("没有物品容器")
        return false
    
    if not inventoryComponent.hasItem(requiredKeyItem):
        print("需要 ", requiredKeyItem.name, " 才能收集此物品")
        return false
    
    return true

func collect(collectorComponent: CollectorComponent) -> Variant:
    # 消耗钥匙
    if consumeKey:
        var inventoryComponent = collectorComponent.parentEntity.get_node("InventoryComponent")
        if inventoryComponent:
            inventoryComponent.removeItem(requiredKeyItem, 1)
            print("消耗了 ", requiredKeyItem.name)
    
    return super.collect(collectorComponent)
```

### 随机奖励箱
```gdscript
# 随机奖励宝箱
class_name RandomRewardChest
extends CollectibleComponent

@export var possibleRewards: Array[Payload] = []
@export var rewardWeights: Array[float] = []

func _ready():
    super._ready()
    
    # 随机选择奖励
    if possibleRewards.size() > 0:
        var randomIndex = selectWeightedRandom()
        self.payload = possibleRewards[randomIndex]

func selectWeightedRandom() -> int:
    if rewardWeights.is_empty():
        return randi() % possibleRewards.size()
    
    var totalWeight = 0.0
    for weight in rewardWeights:
        totalWeight += weight
    
    var randomValue = randf() * totalWeight
    var currentWeight = 0.0
    
    for i in range(rewardWeights.size()):
        currentWeight += rewardWeights[i]
        if randomValue <= currentWeight:
            return i
    
    return 0

func checkCollectionConditions(collectorEntity: Entity, collectorComponent: CollectorComponent) -> bool:
    if not super.checkCollectionConditions(collectorEntity, collectorComponent):
        return false
    
    # 宝箱可能需要特殊开启条件
    if not checkChestOpenConditions(collectorEntity):
        return false
    
    return true

func checkChestOpenConditions(entity: Entity) -> bool:
    # 检查是否有开锁技能或钥匙
    var canOpen = entity.get("lockpickSkill", 0) > 0
    return canOpen
```

## 设计模式

### 责任链模式
- **条件检查：** 多层条件验证
- **可扩展：** 子类可添加额外条件
- **分离关注：** 收集和移除条件分离

### 策略模式
- **载荷系统：** 不同载荷实现不同效果
- **条件策略：** 可自定义收集和移除条件
- **效果策略：** 支持多种收集后效果

### 观察者模式
- **信号通知：** 收集过程的各个阶段发出信号
- **松耦合：** 其他系统可监听收集事件
- **扩展性：** 易于添加新的监听者

## 技术细节

### 性能优化
- **默认禁用监听：** Area2D.monitoring默认关闭
- **按需激活：** 仅在必要时启用碰撞检测
- **延迟删除：** 通过信号系统安全删除实体

### 协作流程
1. **CollectorComponent** 检测碰撞
2. **CollectorComponent** 检查自身收集条件
3. **CollectorComponent** 调用 `requestToCollect`
4. **CollectibleComponent** 检查自身条件
5. **CollectorComponent** 调用 `collect`
6. **CollectibleComponent** 执行载荷并可能自删除

### 载荷系统
- **StatModifierPayload** - 修改统计数据
- **ComponentPayload** - 添加/替换组件
- **InventoryItemPayload** - 物品容器操作
- **CallablePayload** - 执行自定义函数
- **SignalPayload** - 发出信号

## 注意事项

1. **性能考虑：** 默认禁用Area2D监听以节省性能
2. **删除时机：** 通过checkRemovalConditions控制物品删除
3. **条件验证：** 双方都应验证收集条件
4. **信号时序：** 注意信号发出的先后顺序
5. **继承友好：** 虚拟方法可安全重写

## 相关组件
- `CollectorComponent` - 物品收集器组件
- `InventoryComponent` - 物品容器管理
- `StatsComponent` - 统计数据管理
- `HealthComponent` - 生命值相关收集品
- `AreaCollisionComponent` - 区域碰撞检测 