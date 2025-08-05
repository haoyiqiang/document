# SpinComponent API

> **继承关系**: Component > SpinComponent  
> **运动类型**: 旋转运动

让实体或指定节点每帧持续旋转的简单运动组件。使用物理帧更新确保与其他物理组件的一致性，支持可变旋转速度和启用/禁用控制。

## ✨ 主要特性

- 🔄 持续旋转运动
- ⚡ 物理帧同步
- 🎛️ 可变旋转速度
- 🎯 可指定旋转节点
- 🔧 运行时启用/禁用
- 🚀 零配置即用

## 📊 导出属性

### 旋转设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `nodeToRotate` | `Node2D` | `null` | 要旋转的节点（为空则旋转父实体） |
| `rotationPerFrame` | `float` | `1.0` | 每帧旋转角度（-20到+20，单位：弧度） |
| `isEnabled` | `bool` | `true` | 是否启用旋转 |

## 🎯 使用示例

### 基础旋转效果

```gdscript
# Entity Scene Structure:
# └── SpinningCoin (RigidBody2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── SpinComponent
#         rotationPerFrame: 2.0
#         isEnabled: true
```

### 多层旋转系统

```gdscript
# MultiLayerSpinComponent.gd
extends SpinComponent

@export var layers: Array[Node2D] = []
@export var layerSpeeds: Array[float] = []
@export var reverseAlternating: bool = true
@export var speedMultiplier: float = 1.0

var layerSpinComponents: Array[SpinComponent] = []

func _ready():
    super._ready()
    setupLayers()

func setupLayers():
    # 为每个层创建独立的SpinComponent
    for i in layers.size():
        var layer = layers[i]
        var spinComponent = SpinComponent.new()
        
        # 设置旋转速度
        var speed = layerSpeeds[i] if i < layerSpeeds.size() else 1.0
        if reverseAlternating and i % 2 == 1:
            speed *= -1  # 交替反向
        
        spinComponent.nodeToRotate = layer
        spinComponent.rotationPerFrame = speed * speedMultiplier
        spinComponent.isEnabled = isEnabled
        
        add_child(spinComponent)
        layerSpinComponents.append(spinComponent)
        
        print("Layer ", i, " spin speed: ", speed * speedMultiplier)

func setEnabled(enabled: bool):
    isEnabled = enabled
    for spinComponent in layerSpinComponents:
        spinComponent.isEnabled = enabled

func setSpeedMultiplier(multiplier: float):
    speedMultiplier = multiplier
    for i in layerSpinComponents.size():
        var baseSpeed = layerSpeeds[i] if i < layerSpeeds.size() else 1.0
        if reverseAlternating and i % 2 == 1:
            baseSpeed *= -1
        layerSpinComponents[i].rotationPerFrame = baseSpeed * multiplier

func addLayer(layer: Node2D, speed: float):
    layers.append(layer)
    layerSpeeds.append(speed)
    
    var spinComponent = SpinComponent.new()
    spinComponent.nodeToRotate = layer
    spinComponent.rotationPerFrame = speed * speedMultiplier
    spinComponent.isEnabled = isEnabled
    
    add_child(spinComponent)
    layerSpinComponents.append(spinComponent)
```

### 响应式旋转组件

```gdscript
# ResponsiveSpinComponent.gd
extends SpinComponent

@export var respondToInput: bool = false
@export var respondToHealth: bool = false
@export var respondToSpeed: bool = false
@export var inputSensitivity: float = 1.0

var baseRotationSpeed: float
var healthComponent: HealthComponent
var velocityComponent: Component

func _ready():
    super._ready()
    baseRotationSpeed = rotationPerFrame
    setupResponseComponents()

func setupResponseComponents():
    # 查找相关组件
    healthComponent = parentEntity.components.get("HealthComponent")
    velocityComponent = parentEntity.components.get("LinearMotionComponent")
    
    if not velocityComponent:
        velocityComponent = parentEntity.components.get("PlatformerPhysicsComponent")

func _physics_process(delta: float):
    updateRotationSpeed()
    super._physics_process(delta)

func updateRotationSpeed():
    var speedModifier = 1.0
    
    # 基于输入调整
    if respondToInput:
        var inputStrength = getInputStrength()
        speedModifier *= 1.0 + inputStrength * inputSensitivity
    
    # 基于生命值调整
    if respondToHealth and healthComponent:
        var healthPercent = healthComponent.currentHealth / healthComponent.maxHealth
        speedModifier *= 0.3 + healthPercent * 0.7  # 生命值越低转得越慢
    
    # 基于速度调整
    if respondToSpeed and velocityComponent:
        var velocity = getVelocityFromComponent()
        var speedPercent = velocity.length() / 300.0  # 假设最大速度300
        speedModifier *= 1.0 + speedPercent * 0.5
    
    rotationPerFrame = baseRotationSpeed * speedModifier

func getInputStrength() -> float:
    var inputVector = Input.get_vector("move_left", "move_right", "move_up", "move_down")
    return inputVector.length()

func getVelocityFromComponent() -> Vector2:
    if velocityComponent.has_method("getVelocity"):
        return velocityComponent.getVelocity()
    elif velocityComponent.has_method("velocity"):
        return velocityComponent.velocity
    return Vector2.ZERO
```

### 轨道旋转系统

```gdscript
# OrbitSpinComponent.gd
extends SpinComponent

@export var orbitRadius: float = 100.0
@export var orbitSpeed: float = 1.0
@export var orbitCenter: Node2D
@export var satelliteNodes: Array[Node2D] = []
@export var satelliteAngularOffsets: Array[float] = []

var orbitTime: float = 0.0

func _ready():
    super._ready()
    setupOrbitNodes()

func setupOrbitNodes():
    # 如果没有设置轨道中心，使用父实体
    if not orbitCenter:
        orbitCenter = parentEntity
    
    # 为每个卫星节点设置初始角度偏移
    if satelliteAngularOffsets.size() < satelliteNodes.size():
        for i in range(satelliteAngularOffsets.size(), satelliteNodes.size()):
            var offset = (i * TAU) / satelliteNodes.size()  # 均匀分布
            satelliteAngularOffsets.append(offset)

func _physics_process(delta: float):
    super._physics_process(delta)
    
    # 更新轨道运动
    orbitTime += delta * orbitSpeed
    updateOrbitPositions()

func updateOrbitPositions():
    if not orbitCenter:
        return
    
    for i in satelliteNodes.size():
        var satellite = satelliteNodes[i]
        var angleOffset = satelliteAngularOffsets[i] if i < satelliteAngularOffsets.size() else 0.0
        var angle = orbitTime + angleOffset
        
        var orbitPosition = Vector2(
            cos(angle) * orbitRadius,
            sin(angle) * orbitRadius
        )
        
        satellite.global_position = orbitCenter.global_position + orbitPosition

func addSatellite(satellite: Node2D, angleOffset: float = 0.0):
    satelliteNodes.append(satellite)
    satelliteAngularOffsets.append(angleOffset)
    
    print("Added satellite: ", satellite.name, " with offset: ", angleOffset)

func removeSatellite(satellite: Node2D):
    var index = satelliteNodes.find(satellite)
    if index != -1:
        satelliteNodes.remove_at(index)
        if index < satelliteAngularOffsets.size():
            satelliteAngularOffsets.remove_at(index)
        print("Removed satellite: ", satellite.name)

func setOrbitRadius(radius: float):
    orbitRadius = radius

func setOrbitSpeed(speed: float):
    orbitSpeed = speed
```

### 条件旋转组件

```gdscript
# ConditionalSpinComponent.gd
extends SpinComponent

@export var rotateWhenMoving: bool = false
@export var rotateWhenDamaged: bool = false
@export var rotateWhenPowered: bool = false
@export var damageSpinDuration: float = 1.0
@export var powerSpinMultiplier: float = 2.0

var isDamaged: bool = false
var isPowered: bool = false
var isMoving: bool = false
var damageTimer: float = 0.0

func _ready():
    super._ready()
    connectToComponents()

func connectToComponents():
    # 连接到相关组件的信号
    var healthComponent = parentEntity.components.get("HealthComponent")
    if healthComponent and healthComponent.has_signal("health_changed"):
        healthComponent.health_changed.connect(_on_damage_received)
    
    var damageComponent = parentEntity.components.get("DamageReceivingComponent")
    if damageComponent and damageComponent.has_signal("damage_received"):
        damageComponent.damage_received.connect(_on_damage_received)
    
    var powerComponent = parentEntity.components.get("PowerComponent")
    if powerComponent and powerComponent.has_signal("power_changed"):
        powerComponent.power_changed.connect(_on_power_changed)

func _physics_process(delta: float):
    updateConditions(delta)
    updateRotationState()
    
    if shouldRotate():
        super._physics_process(delta)

func updateConditions(delta: float):
    # 检查移动状态
    if rotateWhenMoving:
        isMoving = checkMovementState()
    
    # 更新伤害状态
    if isDamaged:
        damageTimer -= delta
        if damageTimer <= 0:
            isDamaged = false

func checkMovementState() -> bool:
    var velocityComponent = parentEntity.components.get("LinearMotionComponent")
    if not velocityComponent:
        velocityComponent = parentEntity.components.get("PlatformerPhysicsComponent")
    
    if velocityComponent and velocityComponent.has_method("getVelocity"):
        var velocity = velocityComponent.getVelocity()
        return velocity.length() > 10.0  # 速度阈值
    
    return false

func updateRotationState():
    var shouldSpin = false
    var speedMultiplier = 1.0
    
    # 检查各种条件
    if rotateWhenMoving and isMoving:
        shouldSpin = true
    
    if rotateWhenDamaged and isDamaged:
        shouldSpin = true
        speedMultiplier *= 3.0  # 受伤时快速旋转
    
    if rotateWhenPowered and isPowered:
        shouldSpin = true
        speedMultiplier *= powerSpinMultiplier
    
    # 应用设置
    isEnabled = shouldSpin
    if shouldSpin:
        rotationPerFrame = abs(rotationPerFrame) * speedMultiplier
        if isDamaged:
            rotationPerFrame *= -1  # 受伤时反向旋转

func shouldRotate() -> bool:
    return isEnabled and not is_zero_approx(rotationPerFrame)

func _on_damage_received(amount: int):
    if rotateWhenDamaged:
        isDamaged = true
        damageTimer = damageSpinDuration
        print("Damage spin activated for ", damageSpinDuration, " seconds")

func _on_power_changed(powered: bool):
    if rotateWhenPowered:
        isPowered = powered
        print("Power spin ", "activated" if powered else "deactivated")

func setRotateWhenMoving(enabled: bool):
    rotateWhenMoving = enabled

func setRotateWhenDamaged(enabled: bool):
    rotateWhenDamaged = enabled

func setRotateWhenPowered(enabled: bool):
    rotateWhenPowered = enabled
```

### 同步旋转网络

```gdscript
# SynchronizedSpinComponent.gd
extends SpinComponent

@export var syncGroup: String = "default"
@export var isMaster: bool = false
@export var syncTolerance: float = 0.1

static var spinGroups: Dictionary = {}

func _ready():
    super._ready()
    registerToSpinGroup()

func registerToSpinGroup():
    if not spinGroups.has(syncGroup):
        spinGroups[syncGroup] = {
            "master": null,
            "slaves": [],
            "syncRotation": 0.0
        }
    
    var group = spinGroups[syncGroup]
    
    if isMaster:
        group.master = self
        print("Spin master registered for group: ", syncGroup)
    else:
        group.slaves.append(self)
        print("Spin slave registered for group: ", syncGroup)

func _physics_process(delta: float):
    if isMaster:
        # 主控制器正常旋转并同步
        super._physics_process(delta)
        spinGroups[syncGroup].syncRotation = nodeToRotate.rotation
    else:
        # 从属控制器同步到主控制器
        synchronizeWithMaster()

func synchronizeWithMaster():
    var group = spinGroups[syncGroup]
    if not group.master:
        # 没有主控制器，正常旋转
        super._physics_process(get_physics_process_delta_time())
        return
    
    var masterRotation = group.syncRotation
    var currentRotation = nodeToRotate.rotation
    var rotationDifference = abs(masterRotation - currentRotation)
    
    # 如果差异太大，进行同步
    if rotationDifference > syncTolerance:
        nodeToRotate.rotation = masterRotation
        print("Spin synchronized to master, difference was: ", rotationDifference)

func _exit_tree():
    unregisterFromSpinGroup()

func unregisterFromSpinGroup():
    if not spinGroups.has(syncGroup):
        return
    
    var group = spinGroups[syncGroup]
    
    if isMaster and group.master == self:
        group.master = null
        # 选择新的主控制器
        if not group.slaves.is_empty():
            var newMaster = group.slaves[0]
            group.slaves.remove_at(0)
            newMaster.isMaster = true
            group.master = newMaster
            print("New spin master selected for group: ", syncGroup)
    else:
        group.slaves.erase(self)
    
    print("Spin component unregistered from group: ", syncGroup)

static func synchronizeGroup(groupName: String):
    if not spinGroups.has(groupName):
        return
    
    var group = spinGroups[groupName]
    if not group.master:
        return
    
    var masterRotation = group.master.nodeToRotate.rotation
    for slave in group.slaves:
        if slave and slave.nodeToRotate:
            slave.nodeToRotate.rotation = masterRotation
    
    print("Group synchronized: ", groupName)
```

## 🔧 技术细节

### 旋转更新
```gdscript
func _physics_process(delta: float) -> void:
    nodeToRotate.rotation += rotationPerFrame * delta
```

### 性能优化
```gdscript
# 设置器优化，避免每帧检查
set(newValue):
    rotationPerFrame = newValue
    self.set_physics_process(isEnabled and not is_zero_approx(rotationPerFrame))
```

### 节点初始化
```gdscript
func _ready() -> void:
    if not nodeToRotate:
        self.nodeToRotate = parentEntity
    self.set_physics_process(isEnabled and not is_zero_approx(rotationPerFrame))
```

## ⚠️ 注意事项

1. **物理帧更新**: 使用_physics_process而非_process，确保与物理系统同步
2. **性能优化**: 当rotationPerFrame为0或禁用时，自动停止物理处理
3. **节点默认**: nodeToRotate为空时自动使用父实体
4. **角度累积**: 直接修改rotation属性，注意角度值的累积
5. **单位**: rotationPerFrame单位为弧度每秒

## 🔗 相关组件

- [WaveMotionComponent](WaveMotionComponent.md) - 波浪运动组件
- [LinearMotionComponent](LinearMotionComponent.md) - 直线运动组件
- [AttachmentComponent](AttachmentComponent.md) - 附着组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 