# ChaseComponent

## 概述
`ChaseComponent` 是一个追踪移动组件，使父实体能够自动追向指定的目标节点。通过操控实体的`inputDirection`来实现追踪行为，速度和加速度由`OverheadPhysicsComponent`及其移动参数控制。

**继承关系：** `ChaseComponent` → `CharacterBodyDependentComponentBase` → `Component` → `Node`

## 主要特性
- 🎯 自动追踪目标实体
- 🤖 AI友好的追踪系统
- 🏃 自动玩家追踪选项
- ⚡ 高性能物理集成
- 🔧 简单配置界面
- 📊 调试信息支持

## 依赖组件
- **CharacterBodyComponent** (必需) - 角色身体控制
- **OverheadPhysicsComponent** (必需) - 俯视角物理系统

## 导出属性

### `nodeToChase: Node2D`
要追踪的目标节点。

**类型：** `Node2D`  
**默认值：** `null`  
**自动设置：** 如果为空且`shouldChasePlayerIfUnspecified`为true，会自动设为第一个玩家

### `shouldChasePlayerIfUnspecified: bool = true`
当目标节点未指定时是否自动追踪玩家。

**类型：** `bool`  
**默认值：** `true`  
**用途：** 简化AI敌人的配置

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`

## 状态属性

### `recentChaseDirection: Vector2`
最近计算的追踪方向向量。

**类型：** `Vector2`  
**用途：** 存储当前帧的追踪方向，用于调试显示

## 核心方法

### `_physics_process(delta: float) -> void`
物理帧更新，计算并应用追踪方向。

**参数：** `delta` - 物理帧时间间隔  
**功能：**
1. 检查启用状态和目标有效性
2. 计算从当前位置到目标的方向
3. 标准化方向向量
4. 设置物理组件的输入方向
5. 显示调试信息

### `showDebugInfo() -> void`
显示调试信息。

**功能：** 在调试监视列表中显示当前追踪方向

## 使用示例

### 基本追踪设置
```gdscript
# 简单的敌人追踪玩家
func setupEnemyChase():
    var chaseComponent = $ChaseComponent
    
    # 自动追踪第一个玩家
    chaseComponent.shouldChasePlayerIfUnspecified = true
    chaseComponent.nodeToChase = null  # 会自动设为玩家
    chaseComponent.isEnabled = true

# 追踪特定目标
func chaseSpecificTarget(target: Node2D):
    var chaseComponent = $ChaseComponent
    
    chaseComponent.nodeToChase = target
    chaseComponent.shouldChasePlayerIfUnspecified = false
    chaseComponent.isEnabled = true
```

### 宠物追踪系统
```gdscript
# 宠物跟随主人
class_name PetFollower
extends Entity

@export var owner: Entity
@export var followDistance: float = 50.0

func _ready():
    super._ready()
    setupPetChase()

func setupPetChase():
    var chaseComponent = $ChaseComponent
    
    # 设置追踪目标为主人
    chaseComponent.nodeToChase = owner
    chaseComponent.shouldChasePlayerIfUnspecified = false
    chaseComponent.isEnabled = true
    
    # 配置跟随物理参数
    var physicsComponent = $OverheadPhysicsComponent
    physicsComponent.parameters.maxSpeed = 120.0
    physicsComponent.parameters.acceleration = 600.0

func _physics_process(delta):
    super._physics_process(delta)
    
    # 检查距离，太近就停止追踪
    if owner and global_position.distance_to(owner.global_position) < followDistance:
        $ChaseComponent.isEnabled = false
    else:
        $ChaseComponent.isEnabled = true
```

### 巡逻与追踪结合
```gdscript
# 有巡逻和追踪两种模式的敌人
class_name PatrollingEnemy
extends Entity

@export var patrolPoints: Array[Vector2] = []
@export var detectionRange: float = 100.0
@export var chaseRange: float = 200.0

enum State { PATROL, CHASE, RETURN }
var currentState: State = State.PATROL
var currentPatrolIndex: int = 0
var originalPosition: Vector2

func _ready():
    super._ready()
    originalPosition = global_position
    setupPatrolAndChase()

func setupPatrolAndChase():
    var chaseComponent = $ChaseComponent
    chaseComponent.isEnabled = false  # 开始时禁用
    
    # 初始巡逻点
    if patrolPoints.is_empty():
        patrolPoints = [Vector2.ZERO, Vector2(100, 0), Vector2(100, 100), Vector2(0, 100)]

func _physics_process(delta):
    super._physics_process(delta)
    
    var player = GameState.getPlayer(0)
    if not player:
        return
    
    var distanceToPlayer = global_position.distance_to(player.global_position)
    
    match currentState:
        State.PATROL:
            handlePatrol()
            # 检测到玩家进入范围
            if distanceToPlayer <= detectionRange:
                switchToChase(player)
        
        State.CHASE:
            # 玩家逃出追踪范围
            if distanceToPlayer > chaseRange:
                switchToReturn()
        
        State.RETURN:
            handleReturn()

func handlePatrol():
    if patrolPoints.is_empty():
        return
    
    var targetPoint = originalPosition + patrolPoints[currentPatrolIndex]
    var distanceToTarget = global_position.distance_to(targetPoint)
    
    if distanceToTarget < 10.0:
        # 到达巡逻点，前往下一个
        currentPatrolIndex = (currentPatrolIndex + 1) % patrolPoints.size()
    
    # 手动设置移动方向
    var direction = global_position.direction_to(targetPoint)
    $OverheadPhysicsComponent.inputDirection = direction

func switchToChase(target: Entity):
    currentState = State.CHASE
    var chaseComponent = $ChaseComponent
    chaseComponent.nodeToChase = target
    chaseComponent.isEnabled = true
    
    # 提高追踪速度
    $OverheadPhysicsComponent.parameters.maxSpeed = 150.0
    print("开始追踪玩家!")

func switchToReturn():
    currentState = State.RETURN
    $ChaseComponent.isEnabled = false
    
    # 恢复正常速度
    $OverheadPhysicsComponent.parameters.maxSpeed = 80.0
    print("返回巡逻")

func handleReturn():
    var distanceToOriginal = global_position.distance_to(originalPosition)
    
    if distanceToOriginal < 10.0:
        # 回到原位，开始巡逻
        currentState = State.PATROL
        currentPatrolIndex = 0
    else:
        # 返回原始位置
        var direction = global_position.direction_to(originalPosition)
        $OverheadPhysicsComponent.inputDirection = direction
```

### 群体AI追踪
```gdscript
# 群体追踪行为
class_name SwarmChaser
extends Entity

@export var swarmLeader: Entity
@export var formationOffset: Vector2
@export var useFormation: bool = true

func _ready():
    super._ready()
    setupSwarmBehavior()

func setupSwarmBehavior():
    var chaseComponent = $ChaseComponent
    
    if swarmLeader:
        chaseComponent.nodeToChase = swarmLeader
        chaseComponent.shouldChasePlayerIfUnspecified = false
    else:
        # 没有领袖时追踪玩家
        chaseComponent.shouldChasePlayerIfUnspecified = true
    
    chaseComponent.isEnabled = true

func _physics_process(delta):
    super._physics_process(delta)
    
    if useFormation and swarmLeader:
        adjustFormationPosition()

func adjustFormationPosition():
    # 计算编队位置
    var targetPosition = swarmLeader.global_position + formationOffset
    var chaseComponent = $ChaseComponent
    
    # 如果太接近编队位置，减速
    var distance = global_position.distance_to(targetPosition)
    if distance < 30.0:
        $OverheadPhysicsComponent.parameters.maxSpeed = 50.0
    else:
        $OverheadPhysicsComponent.parameters.maxSpeed = 100.0

# 设置编队
func setFormationOffset(offset: Vector2):
    formationOffset = offset
```

### 智能追踪AI
```gdscript
# 具有预测和策略的智能追踪
class_name SmartChaser
extends Entity

@export var predictionTime: float = 0.5
@export var alternateTargets: Array[Entity] = []

var targetVelocity: Vector2
var lastTargetPosition: Vector2

func _ready():
    super._ready()
    setupSmartChasing()

func setupSmartChasing():
    var chaseComponent = $ChaseComponent
    chaseComponent.shouldChasePlayerIfUnspecified = true
    chaseComponent.isEnabled = true

func _physics_process(delta):
    super._physics_process(delta)
    
    var chaseComponent = $ChaseComponent
    var target = chaseComponent.nodeToChase
    
    if target:
        updateTargetVelocity(target, delta)
        
        # 使用预测位置
        if predictionTime > 0:
            var predictedPosition = target.global_position + targetVelocity * predictionTime
            chaseComponent.recentChaseDirection = global_position.direction_to(predictedPosition)
            $OverheadPhysicsComponent.inputDirection = chaseComponent.recentChaseDirection
        
        # 检查是否需要切换目标
        checkAlternateTargets()

func updateTargetVelocity(target: Entity, delta: float):
    if lastTargetPosition != Vector2.ZERO:
        targetVelocity = (target.global_position - lastTargetPosition) / delta
    lastTargetPosition = target.global_position

func checkAlternateTargets():
    var chaseComponent = $ChaseComponent
    var currentTarget = chaseComponent.nodeToChase
    
    if alternateTargets.is_empty() or not currentTarget:
        return
    
    var currentDistance = global_position.distance_to(currentTarget.global_position)
    
    # 找到更近的目标
    for target in alternateTargets:
        if target and target != currentTarget:
            var distance = global_position.distance_to(target.global_position)
            if distance < currentDistance * 0.8:  # 只有明显更近才切换
                chaseComponent.nodeToChase = target
                print("切换追踪目标到: ", target.name)
                break
```

### 条件追踪
```gdscript
# 根据条件启用/禁用追踪
class_name ConditionalChaser
extends Entity

@export var chaseOnlyWhenPlayerVisible: bool = true
@export var chaseOnlyInDaytime: bool = false
@export var healthThreshold: float = 0.5

func _ready():
    super._ready()
    setupConditionalChase()

func setupConditionalChase():
    var chaseComponent = $ChaseComponent
    chaseComponent.shouldChasePlayerIfUnspecified = true
    
    # 定期检查追踪条件
    var timer = Timer.new()
    timer.wait_time = 0.5  # 每0.5秒检查一次
    timer.timeout.connect(checkChaseConditions)
    timer.autostart = true
    add_child(timer)

func checkChaseConditions():
    var chaseComponent = $ChaseComponent
    var shouldChase = true
    
    # 检查可见性
    if chaseOnlyWhenPlayerVisible:
        shouldChase = shouldChase and isPlayerVisible()
    
    # 检查时间
    if chaseOnlyInDaytime:
        shouldChase = shouldChase and isDaytime()
    
    # 检查生命值
    var healthComponent = $HealthComponent
    if healthComponent:
        var healthRatio = healthComponent.currentHealth / healthComponent.maxHealth
        if healthRatio < healthThreshold:
            shouldChase = false  # 血量过低时撤退
    
    chaseComponent.isEnabled = shouldChase

func isPlayerVisible() -> bool:
    var player = GameState.getPlayer(0)
    if not player:
        return false
    
    # 使用射线检测可见性
    var space_state = get_world_2d().direct_space_state
    var query = PhysicsRayQueryParameters2D.create(global_position, player.global_position)
    var result = space_state.intersect_ray(query)
    
    # 如果射线直接击中玩家，说明可见
    return result and result.collider == player

func isDaytime() -> bool:
    # 简单的昼夜判断
    var time = Time.get_time_dict_from_system()
    var hour = time.hour
    return hour >= 6 and hour < 18
```

### 性能优化版本
```gdscript
# 为大量追踪单位优化的版本
class_name OptimizedChaser
extends Entity

@export var updateInterval: float = 0.1  # 降低更新频率
@export var maxChaseDistance: float = 300.0  # 超出范围停止追踪

var updateTimer: float = 0.0
var randomOffset: float

func _ready():
    super._ready()
    # 随机化更新时间避免同步
    randomOffset = randf() * updateInterval
    setupOptimizedChase()

func setupOptimizedChase():
    var chaseComponent = $ChaseComponent
    chaseComponent.shouldChasePlayerIfUnspecified = true
    chaseComponent.isEnabled = true
    
    # 禁用默认的物理处理
    set_physics_process(false)

func _process(delta):
    updateTimer += delta
    
    if updateTimer >= updateInterval + randomOffset:
        updateTimer = 0.0
        updateChaseTarget()

func updateChaseTarget():
    var chaseComponent = $ChaseComponent
    var target = chaseComponent.nodeToChase
    
    if not target:
        return
    
    var distance = global_position.distance_to(target.global_position)
    
    # 超出最大距离时停止追踪
    if distance > maxChaseDistance:
        chaseComponent.isEnabled = false
        return
    
    if not chaseComponent.isEnabled:
        chaseComponent.isEnabled = true
    
    # 手动计算方向
    var direction = global_position.direction_to(target.global_position)
    $OverheadPhysicsComponent.inputDirection = direction
```

## 设计模式

### 组合模式
- **物理集成：** 与OverheadPhysicsComponent协作
- **依赖注入：** 通过组件依赖实现功能
- **职责分离：** 只负责方向计算，速度由物理组件控制

### 策略模式
- **目标选择：** 可配置不同的目标选择策略
- **追踪行为：** 可通过继承实现不同追踪逻辑
- **条件检查：** 支持自定义追踪条件

### 观察者模式
- **调试信息：** 通过调试系统观察状态
- **事件响应：** 可响应目标状态变化
- **松耦合：** 与其他系统低耦合

## 技术细节

### 性能优化
- **方向标准化：** 确保方向向量长度为1
- **空值检查：** 避免无效目标导致的错误
- **简单计算：** 使用高效的方向计算

### 物理集成
- **输入驱动：** 通过设置inputDirection驱动移动
- **速度控制：** 依赖OverheadPhysicsComponent的参数
- **重置设置：** 建议禁用shouldResetVelocityIfZeroMotion

### 调试支持
- **可视化：** 显示当前追踪方向
- **监视列表：** 实时查看追踪状态
- **条件检查：** 调试追踪逻辑

## 注意事项

1. **组件顺序：** 必须在OverheadPhysicsComponent之前
2. **速度控制：** 建议设置shouldResetVelocityIfZeroMotion为false
3. **目标有效性：** 需要检查目标是否存在和有效
4. **性能考虑：** 大量单位时考虑降低更新频率
5. **路径查找：** 不支持复杂路径查找，考虑使用NavigationComponent

## 相关组件
- `OverheadPhysicsComponent` - 俯视角物理系统，控制实际移动
- `CharacterBodyComponent` - 角色身体控制
- `NavigationComponent` - 复杂路径查找系统
- `PatrolComponent` - 巡逻移动系统
- `AIAgentComponent` - AI代理系统 