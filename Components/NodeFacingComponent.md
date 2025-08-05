# NodeFacingComponent API

> **继承关系**: Component > NodeFacingComponent  
> **视觉类型**: 节点朝向

让实体或指定节点旋转朝向另一个目标节点的组件。支持即时旋转和渐进旋转，可用于瞄准系统、跟踪敌人、炮塔控制等需要动态朝向的场景。

## ✨ 主要特性

- 🎯 精确目标朝向
- ⚡ 即时或渐进旋转
- 🎮 与控制组件互斥
- 📐 偏移和随机偏差
- 🎛️ 自动重新启用
- 🎛️ 灵活的节点选择

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `nodeToRotate` | `Node2D` | `null` | 要旋转的节点（为空则旋转父实体） |
| `targetToFace` | `Node2D` | `null` | 要朝向的目标节点 |
| `rotationSpeed` | `float` | `5.0` | 旋转速度（0.1-20） |
| `shouldRotateInstantly` | `bool` | `false` | 是否瞬间旋转到目标 |

### 控制互斥
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldDisableOnTurningInput` | `bool` | `true` | 收到转向输入时是否禁用 |
| `isEnabled` | `bool` | `true` | 是否启用朝向功能 |

### 偏差设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `offset` | `Vector2` | `Vector2.ZERO` | 目标位置的恒定偏移 |
| `shouldApplyRandomDeviation` | `bool` | `false` | 是否应用随机偏差 |
| `randomDeviationMin` | `Vector2` | `Vector2.ZERO` | 随机偏差最小值 |
| `randomDeviationMax` | `Vector2` | `Vector2.ZERO` | 随机偏差最大值 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `previousRotation` | `float` | 上一帧的旋转角度 |
| `didRotateThisFrame` | `bool` | 本帧是否发生了旋转 |
| `reenablingTimer` | `Timer` | 重新启用计时器 |

## 🎯 使用示例

### 基础炮塔瞄准

```gdscript
# Entity Scene Structure:
# └── Turret (Node2D)
#     ├── TurretBase (Sprite2D)
#     ├── TurretBarrel (Node2D)  # 要旋转的部分
#     │   └── Sprite2D
#     └── NodeFacingComponent
#         nodeToRotate: TurretBarrel
#         targetToFace: [Player引用]
#         rotationSpeed: 3.0
#         shouldRotateInstantly: false
```

### 敌人追踪视线

```gdscript
# Entity Scene Structure:
# └── Enemy (CharacterBody2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── NodeFacingComponent
#         targetToFace: [Player引用]
#         rotationSpeed: 2.0
#         shouldDisableOnTurningInput: false
```

### 智能瞄准系统

```gdscript
# SmartAimingComponent.gd
extends NodeFacingComponent

@export var predictTarget: bool = true
@export var leadTime: float = 0.5
@export var maxPredictionDistance: float = 200.0
@export var aimingAccuracy: float = 0.9

var targetVelocity: Vector2 = Vector2.ZERO
var lastTargetPosition: Vector2 = Vector2.ZERO

func _physics_process(delta: float):
    if not isEnabled or not targetToFace:
        return
    
    # 计算目标速度
    updateTargetVelocity(delta)
    
    # 计算预测位置
    var aimPosition = calculateAimPosition()
    
    # 临时修改目标位置
    var originalPosition = targetToFace.global_position
    targetToFace.global_position = aimPosition
    
    # 调用父类朝向逻辑
    super._physics_process(delta)
    
    # 恢复原始位置
    targetToFace.global_position = originalPosition

func updateTargetVelocity(delta: float):
    if targetToFace:
        var currentPosition = targetToFace.global_position
        targetVelocity = (currentPosition - lastTargetPosition) / delta
        lastTargetPosition = currentPosition

func calculateAimPosition() -> Vector2:
    if not predictTarget or targetVelocity.length() < 10.0:
        return targetToFace.global_position + offset
    
    # 计算预测位置
    var distance = nodeToRotate.global_position.distance_to(targetToFace.global_position)
    var timeToTarget = distance / getProjectileSpeed()
    var predictedPosition = targetToFace.global_position + targetVelocity * timeToTarget * leadTime
    
    # 限制预测距离
    var predictionOffset = predictedPosition - targetToFace.global_position
    if predictionOffset.length() > maxPredictionDistance:
        predictionOffset = predictionOffset.normalized() * maxPredictionDistance
        predictedPosition = targetToFace.global_position + predictionOffset
    
    # 应用精度偏差
    if aimingAccuracy < 1.0:
        var accuracyError = (1.0 - aimingAccuracy) * 50.0
        var errorOffset = Vector2(
            randf_range(-accuracyError, accuracyError),
            randf_range(-accuracyError, accuracyError)
        )
        predictedPosition += errorOffset
    
    return predictedPosition + offset

func getProjectileSpeed() -> float:
    # 获取弹药速度（可以从GunComponent或其他组件获取）
    var gunComponent = parentEntity.findFirstComponentOfClass(GunComponent)
    if gunComponent and gunComponent.currentAmmoParameters:
        return gunComponent.currentAmmoParameters.speed
    return 500.0  # 默认弹速

func setAimingAccuracy(accuracy: float):
    aimingAccuracy = clamp(accuracy, 0.0, 1.0)

func setPredictionSettings(predict: bool, leadTime: float, maxDistance: float):
    predictTarget = predict
    self.leadTime = leadTime
    maxPredictionDistance = maxDistance
```

### 动态目标选择器

```gdscript
# DynamicTargetSelector.gd
extends NodeFacingComponent

@export var autoTargetSelection: bool = true
@export var targetDetectionRange: float = 300.0
@export var targetPriority: TargetPriority = TargetPriority.CLOSEST
@export var targetTags: Array[String] = ["enemy"]

enum TargetPriority {
    CLOSEST,
    FARTHEST,
    LOWEST_HEALTH,
    HIGHEST_THREAT
}

var potentialTargets: Array[Entity] = []
var lastTargetScanTime: float = 0.0
var targetScanInterval: float = 0.5

func _physics_process(delta: float):
    # 定期扫描目标
    if autoTargetSelection:
        scanForTargets(delta)
    
    super._physics_process(delta)

func scanForTargets(delta: float):
    lastTargetScanTime += delta
    if lastTargetScanTime < targetScanInterval:
        return
    
    lastTargetScanTime = 0.0
    updatePotentialTargets()
    
    if potentialTargets.is_empty():
        targetToFace = null
        return
    
    # 选择最佳目标
    var bestTarget = selectBestTarget()
    if bestTarget != targetToFace:
        setTarget(bestTarget)

func updatePotentialTargets():
    potentialTargets.clear()
    
    # 获取范围内的所有实体
    var nearbyEntities = getNearbyEntities(targetDetectionRange)
    
    for entity in nearbyEntities:
        if isValidTarget(entity):
            potentialTargets.append(entity)

func getNearbyEntities(range: float) -> Array[Entity]:
    var entities: Array[Entity] = []
    var space_state = parentEntity.get_world_2d().direct_space_state
    
    # 使用区域查询或遍历场景中的实体
    var query = PhysicsShapeQueryParameters2D.new()
    var circle = CircleShape2D.new()
    circle.radius = range
    query.shape = circle
    query.transform.origin = nodeToRotate.global_position
    
    var results = space_state.intersect_shape(query)
    
    for result in results:
        var entity = result.collider.get_node(".") as Entity
        if entity:
            entities.append(entity)
    
    return entities

func isValidTarget(entity: Entity) -> bool:
    if not entity or entity == parentEntity:
        return false
    
    # 检查标签
    for tag in targetTags:
        if entity.has_method("hasTag") and entity.hasTag(tag):
            return true
    
    return false

func selectBestTarget() -> Entity:
    if potentialTargets.is_empty():
        return null
    
    match targetPriority:
        TargetPriority.CLOSEST:
            return getClosestTarget()
        TargetPriority.FARTHEST:
            return getFarthestTarget()
        TargetPriority.LOWEST_HEALTH:
            return getLowestHealthTarget()
        TargetPriority.HIGHEST_THREAT:
            return getHighestThreatTarget()
    
    return potentialTargets[0]

func getClosestTarget() -> Entity:
    var closest: Entity = null
    var minDistance = INF
    
    for target in potentialTargets:
        var distance = nodeToRotate.global_position.distance_to(target.global_position)
        if distance < minDistance:
            minDistance = distance
            closest = target
    
    return closest

func getFarthestTarget() -> Entity:
    var farthest: Entity = null
    var maxDistance = 0.0
    
    for target in potentialTargets:
        var distance = nodeToRotate.global_position.distance_to(target.global_position)
        if distance > maxDistance:
            maxDistance = distance
            farthest = target
    
    return farthest

func getLowestHealthTarget() -> Entity:
    var lowestHealth: Entity = null
    var minHealth = INF
    
    for target in potentialTargets:
        var healthComponent = target.findFirstComponentOfClass(HealthComponent)
        if healthComponent:
            var healthRatio = float(healthComponent.currentHealth) / float(healthComponent.maximumHealth)
            if healthRatio < minHealth:
                minHealth = healthRatio
                lowestHealth = target
    
    return lowestHealth

func getHighestThreatTarget() -> Entity:
    var highestThreat: Entity = null
    var maxThreat = 0.0
    
    for target in potentialTargets:
        var threatLevel = calculateThreatLevel(target)
        if threatLevel > maxThreat:
            maxThreat = threatLevel
            highestThreat = target
    
    return highestThreat

func calculateThreatLevel(target: Entity) -> float:
    var threat = 1.0
    
    # 基于距离的威胁（越近威胁越大）
    var distance = nodeToRotate.global_position.distance_to(target.global_position)
    threat += (targetDetectionRange - distance) / targetDetectionRange
    
    # 基于武器的威胁
    var gunComponent = target.findFirstComponentOfClass(GunComponent)
    if gunComponent:
        threat += 2.0
    
    # 基于生命值的威胁
    var healthComponent = target.findFirstComponentOfClass(HealthComponent)
    if healthComponent:
        var healthRatio = float(healthComponent.currentHealth) / float(healthComponent.maximumHealth)
        threat += healthRatio
    
    return threat

func setTarget(newTarget: Entity):
    targetToFace = newTarget
    print("Target changed to: ", newTarget.name if newTarget else "none")

func addTargetTag(tag: String):
    if tag not in targetTags:
        targetTags.append(tag)

func removeTargetTag(tag: String):
    targetTags.erase(tag)

func setTargetPriority(priority: TargetPriority):
    targetPriority = priority
```

### 群体协调朝向

```gdscript
# CoordinatedFacingComponent.gd
extends NodeFacingComponent

@export var formationGroup: String = "default"
@export var isFormationLeader: bool = false
@export var followLeaderRotation: bool = true
@export var rotationOffset: float = 0.0

static var formationGroups: Dictionary = {}

func _ready():
    super._ready()
    registerToFormation()

func registerToFormation():
    if not formationGroups.has(formationGroup):
        formationGroups[formationGroup] = {
            "leader": null,
            "members": []
        }
    
    var formation = formationGroups[formationGroup]
    
    if isFormationLeader:
        formation.leader = self
        print("Formation leader registered: ", formationGroup)
    else:
        formation.members.append(self)
        print("Formation member registered: ", formationGroup)

func _physics_process(delta: float):
    if followLeaderRotation and not isFormationLeader:
        synchronizeWithLeader(delta)
    else:
        super._physics_process(delta)

func synchronizeWithLeader(delta: float):
    var formation = formationGroups[formationGroup]
    if not formation.leader:
        # 没有领队，正常朝向
        super._physics_process(delta)
        return
    
    var leaderRotation = formation.leader.nodeToRotate.global_rotation
    var targetRotation = leaderRotation + rotationOffset
    
    if shouldRotateInstantly:
        nodeToRotate.global_rotation = targetRotation
    else:
        nodeToRotate.global_rotation = rotate_toward(
            nodeToRotate.global_rotation,
            targetRotation,
            rotationSpeed * delta
        )
    
    # 更新状态
    didRotateThisFrame = not is_equal_approx(nodeToRotate.global_rotation, previousRotation)
    previousRotation = nodeToRotate.global_rotation

func _exit_tree():
    unregisterFromFormation()

func unregisterFromFormation():
    if not formationGroups.has(formationGroup):
        return
    
    var formation = formationGroups[formationGroup]
    
    if isFormationLeader and formation.leader == self:
        formation.leader = null
        # 选择新领队
        if not formation.members.is_empty():
            var newLeader = formation.members[0]
            formation.members.remove_at(0)
            newLeader.isFormationLeader = true
            formation.leader = newLeader
            print("New formation leader selected: ", formationGroup)
    else:
        formation.members.erase(self)
    
    print("Unregistered from formation: ", formationGroup)

func setFormationOffset(offset: float):
    rotationOffset = offset

static func getFormationInfo(groupName: String) -> Dictionary:
    return formationGroups.get(groupName, {})

static func synchronizeFormation(groupName: String):
    if not formationGroups.has(groupName):
        return
    
    var formation = formationGroups[groupName]
    if not formation.leader:
        return
    
    var leaderRotation = formation.leader.nodeToRotate.global_rotation
    
    for member in formation.members:
        if member and member.followLeaderRotation:
            var targetRotation = leaderRotation + member.rotationOffset
            member.nodeToRotate.global_rotation = targetRotation
    
    print("Formation synchronized: ", groupName)
```

## 🔧 技术细节

### 旋转计算
```gdscript
func _physics_process(delta: float) -> void:
    var targetPosition = targetToFace.global_position + offset
    
    if shouldApplyRandomDeviation:
        targetPosition += Vector2(
            randf_range(randomDeviationMin.x, randomDeviationMax.x),
            randf_range(randomDeviationMin.y, randomDeviationMax.y)
        )
    
    if shouldRotateInstantly:
        nodeToRotate.look_at(targetPosition)
    else:
        var rotateTo = nodeToRotate.global_position.angle_to_point(targetPosition)
        nodeToRotate.global_rotation = rotate_toward(
            nodeToRotate.global_rotation, 
            rotateTo, 
            rotationSpeed * delta
        )
```

### 控制互斥机制
- 自动检测TurningControlComponent的存在
- 接收到转向输入时临时禁用
- 使用Timer自动重新启用

## ⚠️ 注意事项

1. **目标有效性**: targetToFace必须有效才能正常工作
2. **控制冲突**: 与TurningControlComponent互斥使用
3. **旋转速度**: 建议保持在合理范围内（0.1-20）
4. **性能考虑**: 每帧进行角度计算和插值
5. **随机偏差**: 每帧生成新的随机值
6. **节点选择**: 确保nodeToRotate是可旋转的Node2D

## 🔗 相关组件

- [TurningControlComponent](./TurningControlComponent.md) - 转向控制组件
- [GunComponent](./GunComponent.md) - 武器组件
- [MouseRotationComponent](./MouseRotationComponent.md) - 鼠标旋转组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 