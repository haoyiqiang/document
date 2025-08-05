# AreaCollisionComponent

## 概述
`AreaCollisionComponent` 是一个Area2D碰撞监控组件，监控Area2D并在与其他Area2D、PhysicsBody2D或TileMapLayer发生碰撞时发射信号。作为任何需要响应物理碰撞的组件的合适基类，提供了灵活的碰撞检测和事件处理机制。

**继承关系：** `AreaCollisionComponent` → `AreaComponentBase` → `Component` → `Node`

## 主要特性
- 🎯 监控Area2D、PhysicsBody2D和TileMapLayer碰撞
- 🔧 基于collision_layer和collision_mask的精确碰撞检测
- 📊 分别控制Area和Body的监控
- ⚡ 性能优化的信号连接控制
- 🏗️ 作为其他碰撞组件的基类
- 🎮 支持进入和退出事件
- 🛡️ 自动过滤父实体碰撞

## 导出属性

### `isEnabled: bool = true`
碰撞监控是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**setter特性：** 影响Area2D.monitorable但不影响monitoring  
**注意：** 不影响EXIT信号或已接触节点的移除

### `shouldMonitorAreas: bool = true`
是否监控Area2D节点。

**类型：** `bool`  
**默认值：** `true`  
**功能：** false时不监控Area2D的进入或退出

### `shouldMonitorBodies: bool = true`
是否监控PhysicsBody2D和TileMapLayer。

**类型：** `bool`  
**默认值：** `true`  
**功能：** false时不监控物理体的进入或退出

### `shouldConnectSignalsOnReady: bool = false`
是否在_ready时自动连接信号。

**类型：** `bool`  
**默认值：** `false`  
**性能提示：** 只在需要时启用物理监控，或在子类中连接信号

## 信号

### `didEnterArea(area: Area2D)`
Area2D进入碰撞区域时发射。

**参数：** `area` - 进入的Area2D节点  
**触发：** 在onCollide方法之后发射  
**特性：** 也为组件就绪时已存在的Area2D发射

### `didExitArea(area: Area2D)`
Area2D离开碰撞区域时发射。

**参数：** `area` - 离开的Area2D节点  
**触发：** 在onExit方法之后发射

### `didEnterBody(body: Node2D)`
PhysicsBody2D或TileMapLayer进入碰撞区域时发射。

**参数：** `body` - 进入的物理体或瓦片地图层  
**触发：** 在onCollide方法之后发射  
**特性：** 也为组件就绪时已存在的物理体发射

### `didExitBody(body: Node2D)`
PhysicsBody2D或TileMapLayer离开碰撞区域时发射。

**参数：** `body` - 离开的物理体或瓦片地图层  
**触发：** 在onExit方法之后发射

## 核心方法

### `connectSignals() -> void`
连接碰撞信号。

**功能：** 连接Area2D的area_entered、area_exited、body_entered、body_exited信号  
**注意：** 默认实现不自动调用，需要手动调用或在子类中调用

### `onAreaEntered(areaEntered: Area2D) -> void`
Area2D进入事件处理。

**参数：** `areaEntered` - 进入的Area2D  
**功能：** 调用onCollide并发射didEnterArea信号  
**过滤：** 忽略父实体及其子节点的碰撞

### `onBodyEntered(bodyEntered: Node2D) -> void`
物理体进入事件处理。

**参数：** `bodyEntered` - 进入的物理体或瓦片地图层  
**功能：** 调用onCollide并发射didEnterBody信号  
**过滤：** 忽略父实体及其子节点的碰撞

### `onAreaExited(areaExited: Area2D) -> void`
Area2D退出事件处理。

**参数：** `areaExited` - 退出的Area2D  
**功能：** 调用onExit并发射didExitArea信号  
**注意：** 不受isEnabled影响，但受shouldMonitorAreas影响

### `onBodyExited(bodyExited: Node2D) -> void`
物理体退出事件处理。

**参数：** `bodyExited` - 退出的物理体或瓦片地图层  
**功能：** 调用onExit并发射didExitBody信号  
**注意：** 不受isEnabled影响，但受shouldMonitorBodies影响

## 抽象方法

### `onCollide(collidingNode: Node2D) -> void`
碰撞发生时调用的抽象方法。

**参数：** `collidingNode` - 发生碰撞的节点  
**触发时机：** 在信号发射之前  
**用途：** 子类可重写此方法处理碰撞逻辑

### `onExit(exitingNode: Node2D) -> void`
节点离开时调用的抽象方法。

**参数：** `exitingNode` - 离开的节点  
**触发时机：** 在信号发射之前  
**用途：** 子类可重写此方法处理离开逻辑

## 使用示例

### 基本碰撞检测
```gdscript
# 为实体添加基本碰撞检测
func setupBasicCollision():
    var collision = $AreaCollisionComponent
    
    # 启用自动信号连接
    collision.shouldConnectSignalsOnReady = true
    
    # 连接信号处理
    collision.didEnterArea.connect(_on_area_entered)
    collision.didEnterBody.connect(_on_body_entered)
    collision.didExitArea.connect(_on_area_exited)
    collision.didExitBody.connect(_on_body_exited)

func _on_area_entered(area: Area2D):
    print("Area进入: ", area.name)

func _on_body_entered(body: Node2D):
    print("Body进入: ", body.name)

func _on_area_exited(area: Area2D):
    print("Area离开: ", area.name)

func _on_body_exited(body: Node2D):
    print("Body离开: ", body.name)
```

### 自定义碰撞处理
```gdscript
# 继承AreaCollisionComponent创建自定义碰撞组件
class_name CustomCollisionComponent
extends AreaCollisionComponent

var collisionCount: int = 0
var collidingBodies: Array[Node2D] = []

func _ready():
    super._ready()
    connectSignals()  # 手动连接信号

func onCollide(collidingNode: Node2D):
    collisionCount += 1
    if collidingNode is PhysicsBody2D:
        collidingBodies.append(collidingNode)
    
    print("碰撞发生，总计: ", collisionCount)
    
    # 自定义碰撞响应
    if collidingNode.has_method("takeDamage"):
        collidingNode.takeDamage(10)

func onExit(exitingNode: Node2D):
    if exitingNode in collidingBodies:
        collidingBodies.erase(exitingNode)
    
    print("节点离开，剩余碰撞: ", collidingBodies.size())
```

### 选择性监控
```gdscript
# 只监控特定类型的碰撞
func setupSelectiveMonitoring():
    var collision = $AreaCollisionComponent
    
    # 只监控物理体，不监控Area
    collision.shouldMonitorAreas = false
    collision.shouldMonitorBodies = true
    
    # 手动连接需要的信号
    collision.connectSignals()
    collision.didEnterBody.connect(_on_enemy_entered)

func _on_enemy_entered(body: Node2D):
    if body.is_in_group("enemies"):
        print("敌人进入攻击范围: ", body.name)
        startCombat(body)

func startCombat(enemy: Node2D):
    # 开始战斗逻辑
    pass
```

### 动态启用/禁用
```gdscript
# 动态控制碰撞检测
class_name ToggleableCollision
extends AreaCollisionComponent

@export var enableOnPlayerApproach: bool = true
@export var activationDistance: float = 100.0

var player: Node2D

func _ready():
    super._ready()
    player = get_tree().get_first_node_in_group("player")
    
    # 初始状态禁用
    isEnabled = false

func _process(_delta):
    if not enableOnPlayerApproach or not player:
        return
    
    var distance = global_position.distance_to(player.global_position)
    var shouldEnable = distance <= activationDistance
    
    if shouldEnable != isEnabled:
        isEnabled = shouldEnable
        if isEnabled:
            connectSignals()
        print("碰撞检测", "启用" if isEnabled else "禁用")
```

### 碰撞过滤
```gdscript
# 带过滤条件的碰撞组件
class_name FilteredCollisionComponent
extends AreaCollisionComponent

@export var allowedGroups: Array[String] = ["projectiles", "pickups"]
@export var blockedGroups: Array[String] = ["walls", "barriers"]

func onCollide(collidingNode: Node2D):
    # 检查节点是否在允许的组中
    var isAllowed = false
    for group in allowedGroups:
        if collidingNode.is_in_group(group):
            isAllowed = true
            break
    
    # 检查节点是否在阻止的组中
    var isBlocked = false
    for group in blockedGroups:
        if collidingNode.is_in_group(group):
            isBlocked = true
            break
    
    if isAllowed and not isBlocked:
        handleAllowedCollision(collidingNode)
    else:
        handleBlockedCollision(collidingNode)

func handleAllowedCollision(node: Node2D):
    print("允许的碰撞: ", node.name)
    # 处理允许的碰撞

func handleBlockedCollision(node: Node2D):
    print("阻止的碰撞: ", node.name)
    # 处理阻止的碰撞
```

### 碰撞计数器
```gdscript
# 统计碰撞信息的组件
class_name CollisionCounter
extends AreaCollisionComponent

var enterCount: int = 0
var exitCount: int = 0
var currentCollisions: int = 0

signal collisionStatsChanged(enter: int, exit: int, current: int)

func _ready():
    super._ready()
    connectSignals()

func onCollide(collidingNode: Node2D):
    enterCount += 1
    currentCollisions += 1
    updateStats()

func onExit(exitingNode: Node2D):
    exitCount += 1
    currentCollisions = max(0, currentCollisions - 1)
    updateStats()

func updateStats():
    collisionStatsChanged.emit(enterCount, exitCount, currentCollisions)
    print("碰撞统计 - 进入: %d, 离开: %d, 当前: %d" % [enterCount, exitCount, currentCollisions])

func resetStats():
    enterCount = 0
    exitCount = 0
    currentCollisions = 0
    updateStats()
```

### 性能优化版本
```gdscript
# 性能优化的碰撞组件
class_name OptimizedCollisionComponent
extends AreaCollisionComponent

@export var maxCollisionsToTrack: int = 10
@export var processInterval: float = 0.1

var trackedCollisions: Array[Node2D] = []
var lastProcessTime: float = 0.0

func _ready():
    super._ready()
    # 延迟连接信号以优化启动性能
    call_deferred("connectSignals")

func onCollide(collidingNode: Node2D):
    # 限制追踪的碰撞数量
    if trackedCollisions.size() >= maxCollisionsToTrack:
        var oldestCollision = trackedCollisions.pop_front()
        print("移除最旧碰撞: ", oldestCollision.name)
    
    trackedCollisions.append(collidingNode)
    
    # 限制处理频率
    var currentTime = Time.get_time_dict_from_system()["second"]
    if currentTime - lastProcessTime >= processInterval:
        processCollision(collidingNode)
        lastProcessTime = currentTime

func onExit(exitingNode: Node2D):
    trackedCollisions.erase(exitingNode)

func processCollision(node: Node2D):
    # 实际的碰撞处理逻辑
    print("处理碰撞: ", node.name)
```

## 设计模式

### 模板方法模式
- **抽象方法：** onCollide和onExit提供扩展点
- **公共流程：** 统一的碰撞检测和信号发射流程
- **可重写性：** 子类可自定义具体的碰撞处理逻辑

### 观察者模式
- **信号发射：** 碰撞事件通过信号通知
- **事件驱动：** 基于Godot的信号系统
- **松耦合：** 碰撞检测与处理逻辑分离

### 策略模式
- **监控策略：** 可选择监控Area、Body或两者
- **信号连接策略：** 可控制信号连接时机
- **过滤策略：** 支持自定义碰撞过滤逻辑

## 技术细节

### 碰撞检测机制
```gdscript
# 基于collision_layer和collision_mask
# 只有匹配的层级组合才会触发碰撞
```

### 防止自碰撞
```gdscript
if collidingNode == self.parentEntity or collidingNode.owner == self.parentEntity: 
    return
```

### 延迟属性设置
```gdscript
# 避免在信号处理中直接设置属性
selfAsArea.set_deferred("monitorable", newValue)
```

### 性能考虑
- shouldConnectSignalsOnReady默认false避免不必要的性能开销
- 退出事件不受isEnabled影响确保状态一致性
- 可选的监控类型减少不必要的信号处理

## 注意事项

1. **信号连接：** 默认不自动连接信号，需要手动调用connectSignals()
2. **退出事件：** 退出事件不受isEnabled影响，确保状态一致性
3. **父实体过滤：** 自动忽略父实体及其子节点的碰撞
4. **性能影响：** 大量碰撞时考虑限制监控范围和处理频率
5. **Layer配置：** 确保collision_layer和collision_mask正确配置

## 相关组件
- `AreaComponentBase` - 基础Area组件类
- `AreaContactComponent` - 维护接触节点列表的组件
- `DamageComponent` - 基于此组件的伤害处理组件
- `CollectibleComponent` - 拾取物品的碰撞检测 