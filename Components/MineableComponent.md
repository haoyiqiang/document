# MineableComponent

## 概述
`MineableComponent` 是一个可挖掘资源组件，继承自InteractionComponent，包含一个可被"消耗"的Stat，限制交互次数。适用于树木、岩石等可被玩家"挖掘"的对象，通过payload生成包含CollectibleStatComponent的实体，为玩家提供游戏资源。

**继承关系：** `MineableComponent` → `InteractionComponent` → `AreaCollisionComponent` → `AreaComponentBase` → `Component` → `Node`

## 主要特性
- ⛏️ 有限次数的挖掘交互
- 📊 基于Stat的资源含量管理
- 🎲 随机化的资源产出量
- 💎 通过payload系统生成收集品
- 🏗️ 资源耗尽时自动移除实体
- 🔧 可配置的成本和产出范围
- 🎯 与CollectibleStatComponent无缝集成

## 导出属性

### `contents: Stat`
可挖掘的资源数量统计。

**类型：** `Stat`  
**用途：** 表示可被挖掘的资源总量，如"木材"、"石头"等  
**机制：** 每次交互时value会减少随机成本，耗尽时不允许交互

### `minimumContentDeduction: int = 1`
每次挖掘的最小成本。

**类型：** `int`  
**默认值：** `1`  
**范围：** `0-100+`  
**影响：** 决定collectibleValue的下限

### `maximumContentDeduction: int = 1`
每次挖掘的最大成本。

**类型：** `int`  
**默认值：** `1`  
**范围：** `1-100+`  
**影响：** 决定collectibleValue的上限

### `allowCostHigherThanContents: bool = true`
是否允许成本高于剩余资源。

**类型：** `bool`  
**默认值：** `true`  
**true时：** 成本高于剩余资源时仍可挖掘，collectibleValue等于剩余资源  
**false时：** 成本高于剩余资源时拒绝交互

### `shouldRemoveEntityOnDepletion: bool = true`
资源耗尽时是否移除实体。

**类型：** `bool`  
**默认值：** `true`  
**功能：** true时资源耗尽后自动调用requestDeletionOfParentEntity

## 状态属性

### `collectibleValue: int`
当前挖掘产出的收集品价值。

**类型：** `int`  
**计算：** 等于从contents中扣除的实际成本  
**用途：** 设置生成实体的CollectibleStatComponent值

## 信号

### `willRemoveEntity`
实体即将被移除时发射。

**触发条件：** shouldRemoveEntityOnDepletion为true且资源耗尽时

### 继承信号
从InteractionComponent继承：
- `didPerformInteraction(result: Variant)` - 交互执行完成

## 核心方法

### `checkInteractionConditions(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool`
检查交互条件。

**参数：**
- `interactorEntity` - 执行交互的实体
- `interactionControlComponent` - 交互控制组件

**返回：** bool - 是否允许交互  
**功能：** 调用deductCost()检查并扣除资源成本

### `deductCost() -> bool`
扣除挖掘成本。

**返回：** bool - 是否成功扣除成本  
**功能：**
1. 生成minimumContentDeduction到maximumContentDeduction之间的随机成本
2. 检查contents.value是否足够支付成本
3. 扣除成本并设置collectibleValue
4. 处理allowCostHigherThanContents逻辑

### `onDidPerformInteraction(result: Variant) -> void`
交互完成后的处理。

**参数：** `result` - 交互结果，通常是生成的实体  
**功能：**
1. 如果result是Entity，查找其CollectibleStatComponent
2. 设置collectibleValue到组件的statModifierMinimum和statModifierMaximum
3. 检查是否需要移除父实体

### `createMineablePayload() -> NodePayload` (实验性)
创建可挖掘的payload。

**返回：** NodePayload - 新创建的节点载荷  
**状态：** 标记为@experimental，待实现

## 使用示例

### 基本资源节点
```gdscript
# 创建基本的树木资源节点
func setupTreeNode():
    var tree = preload("res://Entities/TreeEntity.tscn").instantiate()
    var mineable = tree.get_node("MineableComponent")
    
    # 设置木材资源
    var woodStat = Stat.new()
    woodStat.name = "Wood"
    woodStat.maxValue = 50
    woodStat.value = 50
    mineable.contents = woodStat
    
    # 配置挖掘参数
    mineable.minimumContentDeduction = 3
    mineable.maximumContentDeduction = 8
    mineable.allowCostHigherThanContents = true
    mineable.shouldRemoveEntityOnDepletion = true
    
    # 设置掉落物品payload
    var woodDropPayload = NodePayload.new()
    woodDropPayload.scenePath = "res://Entities/WoodCollectible.tscn"
    mineable.payload = woodDropPayload
    
    add_child(tree)
```

### 高级矿物节点
```gdscript
# 创建复杂的矿物节点
class_name AdvancedMineNode
extends Entity

@onready var mineable: MineableComponent = $MineableComponent

enum ResourceType {
    COMMON_ORE,
    RARE_ORE,
    PRECIOUS_GEM
}

@export var resourceType: ResourceType = ResourceType.COMMON_ORE

func _ready():
    setupMineableResource()
    mineable.willRemoveEntity.connect(_on_will_remove_entity)

func setupMineableResource():
    match resourceType:
        ResourceType.COMMON_ORE:
            setupCommonOre()
        ResourceType.RARE_ORE:
            setupRareOre()
        ResourceType.PRECIOUS_GEM:
            setupPreciousGem()

func setupCommonOre():
    var oreStat = Stat.new()
    oreStat.name = "Iron Ore"
    oreStat.maxValue = 30
    oreStat.value = 30
    mineable.contents = oreStat
    
    mineable.minimumContentDeduction = 2
    mineable.maximumContentDeduction = 5
    
    var orePayload = NodePayload.new()
    orePayload.scenePath = "res://Entities/IronOreCollectible.tscn"
    mineable.payload = orePayload

func setupRareOre():
    var rareStat = Stat.new()
    rareStat.name = "Gold Ore"
    rareStat.maxValue = 15
    rareStat.value = 15
    mineable.contents = rareStat
    
    mineable.minimumContentDeduction = 1
    mineable.maximumContentDeduction = 3
    
    var goldPayload = NodePayload.new()
    goldPayload.scenePath = "res://Entities/GoldOreCollectible.tscn"
    mineable.payload = goldPayload

func setupPreciousGem():
    var gemStat = Stat.new()
    gemStat.name = "Diamond"
    gemStat.maxValue = 5
    gemStat.value = 5
    mineable.contents = gemStat
    
    mineable.minimumContentDeduction = 1
    mineable.maximumContentDeduction = 1
    mineable.allowCostHigherThanContents = false  # 钻石必须精确挖掘
    
    var diamondPayload = NodePayload.new()
    diamondPayload.scenePath = "res://Entities/DiamondCollectible.tscn"
    mineable.payload = diamondPayload

func _on_will_remove_entity():
    print("矿物节点即将被移除: ", resourceType)
    
    # 播放枯竭效果
    $VFX/DepletionEffect.emit()
    $SFX/DepletionSound.play()
    
    # 延迟移除以显示效果
    await get_tree().create_timer(1.0).timeout
```

### 工具需求系统
```gdscript
# 需要特定工具才能挖掘的资源
class_name ToolRequiredMineNode
extends Entity

@onready var mineable: MineableComponent = $MineableComponent

enum RequiredTool {
    NONE,
    PICKAXE,
    AXE,
    SHOVEL
}

@export var requiredTool: RequiredTool = RequiredTool.NONE
@export var toolDurabilityDamage: int = 1

func _ready():
    # 重写交互条件检查
    mineable.checkInteractionConditions = checkToolRequirement

func checkToolRequirement(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool:
    # 检查工具需求
    if requiredTool != RequiredTool.NONE:
        var inventory = interactorEntity.get_node("InventoryComponent")
        if not inventory:
            showMessage("需要工具才能挖掘这个资源!")
            return false
        
        var toolName = getRequiredToolName()
        var tool = inventory.findItemByName(toolName)
        if not tool:
            showMessage("需要 " + toolName + " 才能挖掘!")
            return false
        
        # 检查工具耐久度
        if tool.durability <= 0:
            showMessage(toolName + " 已损坏!")
            return false
    
    # 调用原始检查逻辑
    return mineable.deductCost()

func getRequiredToolName() -> String:
    match requiredTool:
        RequiredTool.PICKAXE: return "Pickaxe"
        RequiredTool.AXE: return "Axe"
        RequiredTool.SHOVEL: return "Shovel"
        _: return ""

func onDidPerformInteraction(result: Variant):
    # 调用父类逻辑
    mineable.onDidPerformInteraction(result)
    
    # 损坏工具
    if requiredTool != RequiredTool.NONE:
        damagePlayerTool()

func damagePlayerTool():
    var player = get_tree().get_first_node_in_group("player")
    if not player:
        return
    
    var inventory = player.get_node("InventoryComponent")
    if not inventory:
        return
    
    var toolName = getRequiredToolName()
    var tool = inventory.findItemByName(toolName)
    if tool:
        tool.durability -= toolDurabilityDamage
        if tool.durability <= 0:
            showMessage(toolName + " 已损坏!")

func showMessage(text: String):
    # 显示消息给玩家
    var ui = get_node("/UI/MessageDisplay")
    if ui:
        ui.showMessage(text)
```

### 重生资源系统
```gdscript
# 可重生的资源节点
class_name RegeneratingMineNode
extends Entity

@onready var mineable: MineableComponent = $MineableComponent

@export var regenerationTime: float = 300.0  # 5分钟重生
@export var regenerationAmount: int = 10
@export var maxRegenerations: int = 3

var regenerationCount: int = 0
var regenerationTimer: Timer

func _ready():
    mineable.shouldRemoveEntityOnDepletion = false  # 不自动移除
    mineable.willRemoveEntity.connect(_on_would_remove_entity)
    
    setupRegenerationTimer()

func setupRegenerationTimer():
    regenerationTimer = Timer.new()
    regenerationTimer.wait_time = regenerationTime
    regenerationTimer.timeout.connect(_on_regeneration_timeout)
    add_child(regenerationTimer)

func _on_would_remove_entity():
    # 资源耗尽时开始重生计时
    if regenerationCount < maxRegenerations:
        print("资源开始重生, 剩余次数: ", maxRegenerations - regenerationCount)
        mineable.isEnabled = false  # 禁用挖掘
        showDepletedState()
        regenerationTimer.start()
    else:
        print("资源彻底耗尽")
        requestDeletionOfParentEntity()

func _on_regeneration_timeout():
    regenerationCount += 1
    
    # 恢复部分资源
    mineable.contents.value = min(mineable.contents.maxValue, regenerationAmount)
    mineable.isEnabled = true
    
    showRegeneratedState()
    print("资源重生完成, 当前含量: ", mineable.contents.value)

func showDepletedState():
    # 显示资源耗尽状态
    $Sprite2D.modulate = Color.GRAY
    $VFX/DepletedParticles.emit()

func showRegeneratedState():
    # 显示资源重生状态
    $Sprite2D.modulate = Color.WHITE
    $VFX/RegenerationParticles.emit()
    $SFX/RegenerationSound.play()
```

### 团队挖掘系统
```gdscript
# 支持多人协作挖掘的资源
class_name TeamMineNode
extends Entity

@onready var mineable: MineableComponent = $MineableComponent

@export var requiresMultipleMiners: bool = false
@export var requiredMinerCount: int = 2
@export var collaborationBonus: float = 1.5

var currentMiners: Array[Entity] = []
var miningInProgress: bool = false

func _ready():
    # 重写交互检查
    mineable.checkInteractionConditions = checkTeamMining

func checkTeamMining(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool:
    if not requiresMultipleMiners:
        return mineable.deductCost()
    
    # 添加矿工到当前列表
    if interactorEntity not in currentMiners:
        currentMiners.append(interactorEntity)
        print("矿工加入: ", interactorEntity.name, ", 当前人数: ", currentMiners.size())
    
    # 检查是否达到所需人数
    if currentMiners.size() >= requiredMinerCount:
        if not miningInProgress:
            startTeamMining()
        return true
    else:
        showMessage("需要 " + str(requiredMinerCount) + " 名矿工协作挖掘")
        return false

func startTeamMining():
    miningInProgress = true
    print("开始团队挖掘!")
    
    # 应用协作奖励
    var originalMin = mineable.minimumContentDeduction
    var originalMax = mineable.maximumContentDeduction
    
    mineable.minimumContentDeduction = int(originalMin * collaborationBonus)
    mineable.maximumContentDeduction = int(originalMax * collaborationBonus)
    
    # 执行挖掘
    var success = mineable.deductCost()
    
    # 恢复原始值
    mineable.minimumContentDeduction = originalMin
    mineable.maximumContentDeduction = originalMax
    
    # 分发奖励给所有参与者
    if success:
        distributeRewards()
    
    # 重置状态
    currentMiners.clear()
    miningInProgress = false

func distributeRewards():
    var rewardPerMiner = mineable.collectibleValue / currentMiners.size()
    
    for miner in currentMiners:
        # 为每个矿工创建奖励
        var reward = createRewardForMiner(miner, rewardPerMiner)
        print("分发奖励给 ", miner.name, ": ", rewardPerMiner)

func createRewardForMiner(miner: Entity, amount: int) -> Entity:
    # 创建个人奖励实体
    var collectible = mineable.payload.createInstance()
    if collectible and collectible is Entity:
        var collectibleStat = collectible.get_node("CollectibleStatComponent")
        if collectibleStat:
            collectibleStat.statModifierMinimum = amount
            collectibleStat.statModifierMaximum = amount
        
        # 在矿工位置生成奖励
        get_parent().add_child(collectible)
        collectible.global_position = miner.global_position + Vector2(randf_range(-20, 20), randf_range(-20, 20))
    
    return collectible

func showMessage(text: String):
    # 显示协作消息
    TextBubble.create(text, self)
```

## 设计模式

### 模板方法模式
- **交互流程：** 继承InteractionComponent的标准交互流程
- **扩展点：** 重写checkInteractionConditions提供挖掘特定逻辑
- **hook方法：** onDidPerformInteraction处理挖掘后逻辑

### 策略模式
- **成本策略：** 支持不同的成本扣除策略
- **payload策略：** 通过payload系统支持不同的掉落物品
- **耗尽策略：** 可配置的实体移除策略

### 观察者模式
- **耗尽通知：** willRemoveEntity信号通知相关系统
- **交互反馈：** 通过继承的信号系统提供反馈
- **状态同步：** CollectibleStatComponent自动同步产出值

## 技术细节

### 随机成本计算
```gdscript
var randomCost: int = randi_range(minimumContentDeduction, maximumContentDeduction)
```

### 成本检查逻辑
```gdscript
if contents.value >= randomCost:
    # 正常扣除
elif allowCostHigherThanContents:
    # 使用剩余值作为产出
else:
    # 拒绝交互
```

### CollectibleValue传递
通过onDidPerformInteraction自动设置生成实体的CollectibleStatComponent值。

### 实体移除机制
使用requestDeletionOfParentEntity()而不是直接queue_free()。

## 注意事项

1. **Stat配置：** 确保contents Stat正确初始化
2. **payload设置：** 确保payload指向有效的收集品实体
3. **CollectibleStatComponent：** 生成的实体必须包含此组件
4. **成本范围：** maximumContentDeduction必须 >= minimumContentDeduction
5. **性能考虑：** 大量可挖掘对象时注意内存管理

## 相关组件
- `InteractionComponent` - 基础交互功能
- `CollectibleStatComponent` - 收集品价值组件
- `InventoryComponent` - 物品收集系统
- `Stat` - 资源数量统计
- `NodePayload` - 实体生成载荷系统