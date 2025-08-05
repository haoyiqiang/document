# PortalInteractionComponent API

> **继承关系**: Component > InteractionComponent > PortalInteractionComponent  
> **对象类型**: 传送门交互

专门用于传送门和传送器的交互组件。玩家与之交互时会被传送到指定的目标位置，适用于门、传送点、关卡切换等场景。

## ✨ 主要特性

- 🚪 瞬间传送功能
- 🎯 灵活的目标设置
- 🔄 继承交互系统
- 📍 全局位置传送
- 🎮 简单易用的接口

## 📊 导出属性

### 传送设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `destinationNode` | `Node2D` | `null` | 传送目标节点位置 |

## 🎯 使用示例

### 基础传送门

```gdscript
# Entity Scene Structure:
# └── Portal (Area2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── PortalInteractionComponent
#         destinationNode: [目标Node2D引用]
```

### 双向传送门系统

```gdscript
# BidirectionalPortal.gd
extends PortalInteractionComponent

@export var linkedPortal: PortalInteractionComponent
@export var portalCooldown: float = 1.0
@export var teleportEffect: PackedScene

var lastTeleportTime: float = 0.0

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    var currentTime = Time.get_time_dict_from_system()
    
    # 检查冷却时间
    if currentTime - lastTeleportTime < portalCooldown:
        print("Portal is on cooldown")
        return interactorEntity.global_position
    
    # 记录传送时间
    lastTeleportTime = currentTime
    
    # 创建传送特效
    if teleportEffect:
        createTeleportEffect(interactorEntity.global_position)
    
    # 执行传送
    var newPosition = super.performInteraction(interactorEntity, interactionControlComponent)
    
    # 目标位置特效
    if teleportEffect and destinationNode:
        createTeleportEffect(destinationNode.global_position)
    
    # 设置关联传送门的冷却
    if linkedPortal:
        linkedPortal.lastTeleportTime = currentTime
    
    print("Teleported from ", interactorEntity.global_position, " to ", newPosition)
    return newPosition

func createTeleportEffect(position: Vector2):
    if not teleportEffect:
        return
    
    var effect = teleportEffect.instantiate()
    get_tree().current_scene.add_child(effect)
    effect.global_position = position
    
    # 自动清理特效
    var timer = Timer.new()
    timer.wait_time = 2.0
    timer.one_shot = true
    timer.timeout.connect(effect.queue_free)
    effect.add_child(timer)
    timer.start()
```

### 条件传送门

```gdscript
# ConditionalPortal.gd
extends PortalInteractionComponent

@export var requiredKey: String = ""
@export var requiredLevel: int = 0
@export var consumeKey: bool = true
@export var lockedMessage: String = "This portal is locked!"

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    # 检查条件
    if not checkTeleportConditions(interactorEntity):
        showLockedMessage()
        return interactorEntity.global_position
    
    # 消耗钥匙（如果需要）
    if consumeKey and requiredKey != "":
        consumeRequiredKey(interactorEntity)
    
    print("Access granted! Teleporting...")
    return super.performInteraction(interactorEntity, interactionControlComponent)

func checkTeleportConditions(interactorEntity: Entity) -> bool:
    # 检查等级要求
    if requiredLevel > 0:
        var levelComponent = interactorEntity.findFirstComponentOfClass(LevelComponent)
        if not levelComponent or levelComponent.currentLevel < requiredLevel:
            print("Required level: ", requiredLevel)
            return false
    
    # 检查钥匙要求
    if requiredKey != "":
        var inventoryComponent = interactorEntity.findFirstComponentOfClass(InventoryComponent)
        if not inventoryComponent or not inventoryComponent.hasItem(requiredKey):
            print("Required key: ", requiredKey)
            return false
    
    return true

func consumeRequiredKey(interactorEntity: Entity):
    var inventoryComponent = interactorEntity.findFirstComponentOfClass(InventoryComponent)
    if inventoryComponent:
        inventoryComponent.removeItem(requiredKey, 1)
        print("Consumed key: ", requiredKey)

func showLockedMessage():
    print(lockedMessage)
    # 这里可以添加UI提示
    GlobalUI.showTemporaryMessage(lockedMessage, 2.0)
```

### 随机传送门

```gdscript
# RandomPortal.gd
extends PortalInteractionComponent

@export var destinationNodes: Array[Node2D] = []
@export var excludeLastDestination: bool = true
@export var teleportChance: float = 1.0

var lastDestinationIndex: int = -1

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    # 检查传送概率
    if randf() > teleportChance:
        print("Teleport failed due to chance")
        return interactorEntity.global_position
    
    # 选择随机目标
    var selectedDestination = selectRandomDestination()
    if not selectedDestination:
        print("No valid destination found")
        return interactorEntity.global_position
    
    # 临时设置目标
    var originalDestination = destinationNode
    destinationNode = selectedDestination
    
    # 执行传送
    var result = super.performInteraction(interactorEntity, interactionControlComponent)
    
    # 恢复原始设置
    destinationNode = originalDestination
    
    return result

func selectRandomDestination() -> Node2D:
    if destinationNodes.is_empty():
        return destinationNode  # 回退到默认目标
    
    var availableDestinations = destinationNodes.duplicate()
    
    # 排除上次的目标
    if excludeLastDestination and lastDestinationIndex >= 0 and lastDestinationIndex < availableDestinations.size():
        availableDestinations.remove_at(lastDestinationIndex)
    
    if availableDestinations.is_empty():
        return destinationNode
    
    # 随机选择
    var randomIndex = randi() % availableDestinations.size()
    var selectedDestination = availableDestinations[randomIndex]
    
    # 记录选择的索引（在原数组中）
    lastDestinationIndex = destinationNodes.find(selectedDestination)
    
    print("Random destination selected: ", selectedDestination.name)
    return selectedDestination

func addDestination(destination: Node2D):
    if destination and destination not in destinationNodes:
        destinationNodes.append(destination)

func removeDestination(destination: Node2D):
    destinationNodes.erase(destination)

func clearDestinations():
    destinationNodes.clear()
    lastDestinationIndex = -1
```

### 延迟传送门

```gdscript
# DelayedPortal.gd
extends PortalInteractionComponent

@export var teleportDelay: float = 2.0
@export var showCountdown: bool = true
@export var allowCancellation: bool = true
@export var channelEffect: PackedScene

var isTeleporting: bool = false
var teleportTimer: float = 0.0
var channelingEntity: Entity
var channelingEffect: Node

signal teleport_started(entity: Entity)
signal teleport_cancelled(entity: Entity)
signal teleport_completed(entity: Entity)

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    if isTeleporting:
        # 如果允许取消，则取消传送
        if allowCancellation:
            cancelTeleport()
        return interactorEntity.global_position
    
    # 开始延迟传送
    startDelayedTeleport(interactorEntity)
    return interactorEntity.global_position

func startDelayedTeleport(entity: Entity):
    isTeleporting = true
    channelingEntity = entity
    teleportTimer = teleportDelay
    
    # 创建引导特效
    if channelEffect:
        channelingEffect = channelEffect.instantiate()
        entity.add_child(channelingEffect)
    
    teleport_started.emit(entity)
    print("Teleport started, will complete in ", teleportDelay, " seconds")
    
    # 开始计时
    set_physics_process(true)

func _physics_process(delta: float):
    if not isTeleporting:
        set_physics_process(false)
        return
    
    teleportTimer -= delta
    
    # 显示倒计时
    if showCountdown:
        var remainingTime = max(0, teleportTimer)
        print("Teleporting in: ", "%.1f" % remainingTime)
    
    # 检查是否完成
    if teleportTimer <= 0.0:
        completeTeleport()

func completeTeleport():
    if not channelingEntity:
        cancelTeleport()
        return
    
    # 清理引导效果
    if channelingEffect:
        channelingEffect.queue_free()
        channelingEffect = null
    
    # 执行传送
    var originalDestination = destinationNode
    var newPosition = channelingEntity.global_position
    
    if destinationNode:
        channelingEntity.global_position = destinationNode.global_position
        newPosition = destinationNode.global_position
    
    teleport_completed.emit(channelingEntity)
    print("Teleport completed to: ", newPosition)
    
    # 重置状态
    resetTeleportState()

func cancelTeleport():
    if not isTeleporting:
        return
    
    # 清理引导效果
    if channelingEffect:
        channelingEffect.queue_free()
        channelingEffect = null
    
    teleport_cancelled.emit(channelingEntity)
    print("Teleport cancelled")
    
    # 重置状态
    resetTeleportState()

func resetTeleportState():
    isTeleporting = false
    teleportTimer = 0.0
    channelingEntity = null
    channelingEffect = null
    set_physics_process(false)

func _input(event: InputEvent):
    if isTeleporting and allowCancellation and event.is_action_pressed("cancel"):
        cancelTeleport()
```

### 网络传送门

```gdscript
# NetworkPortal.gd
extends PortalInteractionComponent

@export var portalNetwork: String = "default"
@export var portalID: String = ""
@export var excludeSamePortal: bool = true

static var portalNetworks: Dictionary = {}

func _ready():
    super._ready()
    registerPortal()

func registerPortal():
    if portalID == "":
        portalID = str(get_instance_id())
    
    if not portalNetworks.has(portalNetwork):
        portalNetworks[portalNetwork] = []
    
    var portalData = {
        "id": portalID,
        "portal": self,
        "position": global_position
    }
    
    portalNetworks[portalNetwork].append(portalData)
    print("Portal registered: ", portalID, " in network: ", portalNetwork)

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    var targetPortal = selectNetworkDestination()
    
    if not targetPortal:
        print("No network destination available")
        return interactorEntity.global_position
    
    # 传送到网络中的另一个传送门
    interactorEntity.global_position = targetPortal.global_position
    print("Network teleport to portal: ", targetPortal.portalID)
    
    return targetPortal.global_position

func selectNetworkDestination() -> PortalInteractionComponent:
    if not portalNetworks.has(portalNetwork):
        return null
    
    var networkPortals = portalNetworks[portalNetwork]
    var availablePortals = []
    
    for portalData in networkPortals:
        var portal = portalData.portal
        if portal and portal != self:  # 排除自己
            if not excludeSamePortal or portal.portalID != portalID:
                availablePortals.append(portal)
    
    if availablePortals.is_empty():
        return null
    
    # 随机选择一个
    var randomIndex = randi() % availablePortals.size()
    return availablePortals[randomIndex]

func _exit_tree():
    unregisterPortal()

func unregisterPortal():
    if not portalNetworks.has(portalNetwork):
        return
    
    var networkPortals = portalNetworks[portalNetwork]
    for i in range(networkPortals.size() - 1, -1, -1):
        if networkPortals[i].portal == self:
            networkPortals.remove_at(i)
            break
    
    print("Portal unregistered: ", portalID)

static func getNetworkPortals(networkName: String) -> Array:
    return portalNetworks.get(networkName, [])

static func clearNetwork(networkName: String):
    if portalNetworks.has(networkName):
        portalNetworks[networkName].clear()
```

## 🔧 技术细节

### 基础传送逻辑
```gdscript
func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Vector2:
    interactorEntity.global_position = destinationNode.global_position
    return interactorEntity.global_position
```

### 继承特性
- 继承自InteractionComponent的所有功能
- 自动处理交互检测和触发
- 支持交互条件和冷却

## ⚠️ 注意事项

1. **目标节点**: destinationNode必须有效，否则传送失败
2. **全局位置**: 使用global_position确保正确的世界坐标
3. **继承关系**: 完全兼容InteractionComponent的所有功能
4. **性能考虑**: 瞬间传送，无动画过渡
5. **多场景**: 目标节点必须在同一场景中

## 🔗 相关组件

- [InteractionComponent](InteractionComponent.md) - 基础交互组件
- [InteractionControlComponent](../Control/InteractionControlComponent.md) - 交互控制组件
- [CollectorComponent](CollectorComponent.md) - 收集器组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 