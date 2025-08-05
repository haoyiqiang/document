# WaveMotionComponent API

> **继承关系**: Component > WaveMotionComponent  
> **运动类型**: 波浪运动

生成正弦/余弦波形运动的组件，可以应用到实体位置产生波浪式移动。支持单轴波形运动或双轴组合的圆周运动，适用于飞行器、浮动平台、装饰性动画等。

## ✨ 主要特性

- 🌊 正弦/余弦波形生成
- ⭕ 圆周运动支持
- 🎛️ 独立的XY轴控制
- 📍 直接位置修改（非物理）
- 🎨 可指定目标节点
- ⚡ 高性能增量更新

## 📊 导出属性

### 波形控制
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldUseCosineForX` | `bool` | `false` | X轴是否使用余弦波（用于圆周运动） |
| `xAmplitude` | `float` | `0.0` | X轴波形振幅（像素范围：-1000到+1000） |
| `xFrequency` | `float` | `0.0` | X轴波形频率（每秒周期数：-100到+100） |
| `yAmplitude` | `float` | `64.0` | Y轴波形振幅（像素范围：-1000到+1000） |
| `yFrequency` | `float` | `1.0` | Y轴波形频率（每秒周期数：-100到+100） |

### 目标设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `nodeToMove` | `Node2D` | `null` | 要移动的节点（为空则使用父实体） |
| `isEnabled` | `bool` | `true` | 是否启用波形运动 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `time` | `float` | 组件内部时间计数器 |
| `wavePosition` | `Vector2` | 当前波形位置 |
| `previousWavePosition` | `Vector2` | 上一帧波形位置 |
| `waveDelta` | `Vector2` | 波形位置变化量 |

## 🎯 使用示例

### 基础垂直浮动

```gdscript
# Entity Scene Structure:
# └── FloatingPlatform (Node2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     └── WaveMotionComponent
#         xAmplitude: 0.0
#         yAmplitude: 32.0
#         yFrequency: 0.5  # 2秒一个周期
```

### 圆周运动

```gdscript
# Entity Scene Structure:
# └── OrbitingDrone (Node2D)
#     ├── Sprite2D
#     └── WaveMotionComponent
#         shouldUseCosineForX: true  # 启用圆周运动
#         xAmplitude: 100.0
#         xFrequency: 1.0
#         yAmplitude: 100.0
#         yFrequency: 1.0
```

### 复杂波形运动

```gdscript
# ComplexWaveComponent.gd
extends WaveMotionComponent

@export var phaseOffset: float = 0.0
@export var amplitudeModulation: bool = false
@export var frequencyModulation: bool = false
@export var modulationSpeed: float = 0.5

var baseXAmplitude: float
var baseYAmplitude: float
var baseXFrequency: float
var baseYFrequency: float

func _ready():
    super._ready()
    # 保存基础值
    baseXAmplitude = xAmplitude
    baseYAmplitude = yAmplitude
    baseXFrequency = xFrequency
    baseYFrequency = yFrequency

func _physics_process(delta: float):
    # 应用调制
    if amplitudeModulation:
        applyAmplitudeModulation()
    
    if frequencyModulation:
        applyFrequencyModulation()
    
    super._physics_process(delta)

func applyAmplitudeModulation():
    var modulationFactor = sin(time * modulationSpeed) * 0.5 + 0.5  # 0-1范围
    xAmplitude = baseXAmplitude * (0.5 + modulationFactor * 0.5)
    yAmplitude = baseYAmplitude * (0.5 + modulationFactor * 0.5)

func applyFrequencyModulation():
    var modulationFactor = cos(time * modulationSpeed) * 0.5 + 0.5  # 0-1范围
    xFrequency = baseXFrequency * (0.5 + modulationFactor * 0.5)
    yFrequency = baseYFrequency * (0.5 + modulationFactor * 0.5)

func setPhaseOffset(offset: float):
    phaseOffset = offset
    # 调整时间以应用相位偏移
    time += offset / (PI * max(xFrequency, yFrequency))

func resetWave():
    time = 0.0
    wavePosition = Vector2.ZERO
    previousWavePosition = Vector2.ZERO
    waveDelta = Vector2.ZERO
```

### 编队波形运动

```gdscript
# FormationWaveComponent.gd
extends WaveMotionComponent

@export var formationOffset: Vector2 = Vector2.ZERO
@export var formationPhase: float = 0.0
@export var followLeader: bool = false
@export var leaderEntity: Entity

var originalPosition: Vector2

func _ready():
    super._ready()
    originalPosition = nodeToMove.position if nodeToMove else parentEntity.position

func _physics_process(delta: float):
    # 调整时间以包含编队相位
    var adjustedTime = time + formationPhase
    
    # 计算波形位置
    var formationWavePosition: Vector2
    
    if shouldUseCosineForX:
        formationWavePosition = Vector2(
            cos(adjustedTime * (PI * xFrequency)) * xAmplitude,
            sin(adjustedTime * (PI * yFrequency)) * yAmplitude
        )
    else:
        formationWavePosition = Vector2(
            sin(adjustedTime * (PI * xFrequency)) * xAmplitude,
            sin(adjustedTime * (PI * yFrequency)) * yAmplitude
        )
    
    # 应用编队偏移
    formationWavePosition += formationOffset
    
    # 跟随领队（如果设置）
    if followLeader and leaderEntity:
        var leaderDelta = leaderEntity.position - originalPosition
        formationWavePosition += leaderDelta
    
    # 计算增量并应用
    var currentDelta = formationWavePosition - previousWavePosition
    nodeToMove.position += currentDelta
    
    # 更新状态
    previousWavePosition = formationWavePosition
    time += delta

func setFormationPosition(offset: Vector2, phase: float):
    formationOffset = offset
    formationPhase = phase

func setLeader(leader: Entity):
    leaderEntity = leader
    followLeader = leader != null
```

### 响应式波形组件

```gdscript
# ResponsiveWaveComponent.gd
extends WaveMotionComponent

@export var respondToInput: bool = false
@export var inputSensitivity: float = 1.0
@export var environmentalFactors: bool = false

var inputInfluence: Vector2 = Vector2.ZERO

func _physics_process(delta: float):
    if respondToInput:
        updateInputInfluence()
    
    if environmentalFactors:
        applyEnvironmentalEffects()
    
    super._physics_process(delta)

func updateInputInfluence():
    var inputVector = Input.get_vector(
        "move_left", "move_right",
        "move_up", "move_down"
    )
    
    inputInfluence = inputVector * inputSensitivity
    
    # 修改频率和振幅
    xFrequency = baseXFrequency + inputInfluence.x * 0.1
    yFrequency = baseYFrequency + inputInfluence.y * 0.1

func applyEnvironmentalEffects():
    # 模拟风力影响
    var windStrength = GlobalWeather.getWindStrength() if GlobalWeather else 0.0
    var windDirection = GlobalWeather.getWindDirection() if GlobalWeather else Vector2.ZERO
    
    if windStrength > 0:
        xAmplitude = baseXAmplitude + windDirection.x * windStrength * 10
        yAmplitude = baseYAmplitude + windDirection.y * windStrength * 10
    
    # 模拟重力影响
    var gravityFactor = GlobalPhysics.getGravityStrength() if GlobalPhysics else 1.0
    yFrequency = baseYFrequency * gravityFactor

func _input(event: InputEvent):
    if respondToInput and event is InputEventKey:
        var keyEvent = event as InputEventKey
        if keyEvent.pressed:
            match keyEvent.keycode:
                KEY_SPACE:
                    # 空格键产生冲击波效果
                    createShockwave()

func createShockwave():
    var originalAmplitude = yAmplitude
    yAmplitude *= 2.0  # 瞬间增大振幅
    
    # 使用Tween恢复正常
    var tween = create_tween()
    tween.tween_property(self, "yAmplitude", originalAmplitude, 0.5)
```

### 物理影响波形

```gdscript
# PhysicsInfluencedWave.gd
extends WaveMotionComponent

@export var respondToCollisions: bool = true
@export var dampingFactor: float = 0.95
@export var impactAmplification: float = 2.0

var hasCollisionDetection: bool = false
var collisionBody: RigidBody2D

func _ready():
    super._ready()
    setupCollisionDetection()

func setupCollisionDetection():
    # 查找父实体中的刚体
    collisionBody = parentEntity.get_node("RigidBody2D") if parentEntity.has_node("RigidBody2D") else null
    
    if collisionBody:
        hasCollisionDetection = true
        collisionBody.body_entered.connect(_on_collision_detected)

func _physics_process(delta: float):
    # 应用阻尼
    if dampingFactor < 1.0:
        xAmplitude *= dampingFactor
        yAmplitude *= dampingFactor
    
    super._physics_process(delta)

func _on_collision_detected(body: Node):
    if not respondToCollisions:
        return
    
    # 碰撞产生冲击效果
    var impactStrength = 1.0
    if body.has_method("get_mass"):
        impactStrength = min(body.get_mass() / 10.0, 3.0)
    
    # 增加振幅模拟冲击
    xAmplitude = min(xAmplitude * impactAmplification * impactStrength, 200.0)
    yAmplitude = min(yAmplitude * impactAmplification * impactStrength, 200.0)
    
    print("Collision impact: strength = ", impactStrength)

func addExternalForce(force: Vector2):
    # 外部力量影响波形
    var forceInfluence = force * 0.1
    xAmplitude += abs(forceInfluence.x)
    yAmplitude += abs(forceInfluence.y)
    
    # 限制最大振幅
    xAmplitude = min(xAmplitude, 300.0)
    yAmplitude = min(yAmplitude, 300.0)

func dampWave(factor: float):
    xAmplitude *= factor
    yAmplitude *= factor

func amplifyWave(factor: float):
    xAmplitude *= factor
    yAmplitude *= factor
```

### 同步波形网络

```gdscript
# SynchronizedWaveComponent.gd
extends WaveMotionComponent

@export var waveGroup: String = "default"
@export var isMaster: bool = false
@export var syncTolerance: float = 0.1

static var waveGroups: Dictionary = {}

func _ready():
    super._ready()
    registerToWaveGroup()

func registerToWaveGroup():
    if not waveGroups.has(waveGroup):
        waveGroups[waveGroup] = {
            "master": null,
            "slaves": [],
            "syncTime": 0.0
        }
    
    var group = waveGroups[waveGroup]
    
    if isMaster:
        group.master = self
        print("Wave master registered for group: ", waveGroup)
    else:
        group.slaves.append(self)
        print("Wave slave registered for group: ", waveGroup)

func _physics_process(delta: float):
    if isMaster:
        # 主控制器更新同步时间
        waveGroups[waveGroup].syncTime = time
        super._physics_process(delta)
    else:
        # 从属控制器同步到主控制器
        synchronizeWithMaster(delta)

func synchronizeWithMaster(delta: float):
    var group = waveGroups[waveGroup]
    if not group.master:
        # 没有主控制器，正常运行
        super._physics_process(delta)
        return
    
    var masterTime = group.syncTime
    var timeDifference = abs(time - masterTime)
    
    # 如果时间差太大，进行同步
    if timeDifference > syncTolerance:
        time = masterTime
        print("Wave synchronized to master, difference was: ", timeDifference)
    
    super._physics_process(delta)

func _exit_tree():
    unregisterFromWaveGroup()

func unregisterFromWaveGroup():
    if not waveGroups.has(waveGroup):
        return
    
    var group = waveGroups[waveGroup]
    
    if isMaster and group.master == self:
        group.master = null
        # 选择新的主控制器
        if not group.slaves.is_empty():
            var newMaster = group.slaves[0]
            group.slaves.remove_at(0)
            newMaster.isMaster = true
            group.master = newMaster
            print("New wave master selected for group: ", waveGroup)
    else:
        group.slaves.erase(self)
    
    print("Wave component unregistered from group: ", waveGroup)

static func getGroupInfo(groupName: String) -> Dictionary:
    return waveGroups.get(groupName, {})

static func synchronizeGroup(groupName: String):
    if not waveGroups.has(groupName):
        return
    
    var group = waveGroups[groupName]
    if not group.master:
        return
    
    var masterTime = group.master.time
    for slave in group.slaves:
        if slave:
            slave.time = masterTime
    
    print("Group synchronized: ", groupName)
```

## 🔧 技术细节

### 波形计算
```gdscript
func _physics_process(delta: float) -> void:
    time += delta
    previousWavePosition = wavePosition
    
    # 生成波形位置
    if shouldUseCosineForX:
        wavePosition = Vector2(
            cos(time * (PI * xFrequency)) * xAmplitude,
            sin(time * (PI * yFrequency)) * yAmplitude
        )
    else:
        wavePosition = Vector2(
            sin(time * (PI * xFrequency)) * xAmplitude,
            sin(time * (PI * yFrequency)) * yAmplitude
        )
    
    # 计算增量并应用
    waveDelta = wavePosition - previousWavePosition
    nodeToMove.position += waveDelta
```

### 圆周运动设置
- **X轴**: 使用余弦波 (`shouldUseCosineForX = true`)
- **Y轴**: 使用正弦波
- **相位差**: 余弦和正弦的90度相位差产生圆周运动

## ⚠️ 注意事项

1. **直接位置修改**: 不使用物理velocity，直接修改position
2. **增量更新**: 使用waveDelta避免位置累积误差
3. **节点选择**: 如果不指定nodeToMove，默认移动父实体
4. **频率单位**: 频率表示每秒的周期数
5. **振幅限制**: 建议将振幅限制在合理范围内
6. **性能考虑**: 每帧进行三角函数计算

## 🔗 相关组件

- [LinearMotionComponent](LinearMotionComponent.md) - 直线运动组件
- [SpinComponent](SpinComponent.md) - 旋转运动组件
- [PathFollowComponent](PathFollowComponent.md) - 路径跟随组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21