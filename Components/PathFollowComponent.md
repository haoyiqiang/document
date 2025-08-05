# PathFollowComponent

## 概述
`PathFollowComponent` 是一个路径跟随移动组件，每帧递增PathFollow2D.progress来实现沿路径移动。直接设置实体位置，不使用物理引擎，适合需要精确路径控制的场景如巡逻路线、轨道移动等。

**继承关系：** `PathFollowComponent` → `IndependentPathFollowComponent` → `Component` → `Node`

**要求：** 父实体必须是PathFollow2D的子节点，PathFollow2D必须是Path2D的子节点

## 主要特性
- 🛤️ 沿Path2D曲线精确移动
- 🔄 支持循环和往返路径
- 🎯 直接位置设置，无物理干扰
- 📐 遵循PathFollow2D的旋转和插值设置
- ⚡ 自动路径完成检测
- 🔧 可配置的进度重置
- 📊 基于IndependentPathFollowComponent扩展

## 注意事项
- **警告：** 如果多个实体共享同一个PathFollow2D，它们会重叠在相同位置
- **建议：** 为每个实体创建独立的PathFollow2D，或使用IndependentPathFollowComponent

## 导出属性

### `shouldResetProgress: bool = true`
是否在组件就绪和注销时重置进度。

**类型：** `bool`  
**默认值：** `true`  
**警告：** 此属性会覆盖shouldSnapEntityToPath  
**功能：** true时在_ready和unregisterEntity时将progress设为0

## 状态属性

### `pathFollower: PathFollow2D` (只读)
关联的PathFollow2D节点引用。

**类型：** `PathFollow2D`  
**获取：** 自动从父实体的父节点获取  
**用途：** 控制路径跟随的进度和设置

## 继承属性

从`IndependentPathFollowComponent`继承的属性：
- `speed: float` - 移动速度
- `path: Path2D` - 路径引用
- `curve: Curve2D` - 曲线引用
- `isEnabled: bool` - 是否启用

## 核心方法

### `setDependencies() -> bool`
设置路径依赖关系。

**返回：** bool - 是否成功设置所有依赖  
**功能：**
1. 获取父实体的PathFollow2D父节点
2. 获取PathFollow2D的Path2D父节点
3. 调用父类setDependencies获取Curve2D

### `snapEntityToCurve() -> Vector2`
将实体位置吸附到最近的路径点。

**返回：** Vector2 - 新的吸附位置  
**功能：**
1. 计算实体在路径空间中的位置
2. 找到曲线上最近的点
3. 应用旋转（如果启用）
4. 更新PathFollow2D的progress
5. 将实体移动到新位置

### `_physics_process(delta: float) -> void`
物理进程中的路径跟随更新。

**参数：** `delta` - 帧时间增量  
**功能：**
1. 根据速度更新progress
2. 检测路径完成
3. 处理循环逻辑

### `unregisterEntity() -> void`
实体注销时的清理。

**功能：** 如果shouldResetProgress为true，重置progress为0

## 信号

### `didCompletePath`
路径完成时发射。

**触发条件：**
- progress_ratio达到1.0或超过1.0
- 或者从0.9跳到0.1（环绕检测）

## 使用示例

### 基本路径跟随
```gdscript
# 设置基本的路径跟随实体
func setupPathFollower():
    # 创建路径结构
    var path2D = Path2D.new()
    var curve = Curve2D.new()
    
    # 添加路径点
    curve.add_point(Vector2(0, 0))
    curve.add_point(Vector2(100, 0))
    curve.add_point(Vector2(100, 100))
    curve.add_point(Vector2(0, 100))
    
    path2D.curve = curve
    add_child(path2D)
    
    # 创建PathFollow2D
    var pathFollow = PathFollow2D.new()
    pathFollow.loop = true  # 启用循环
    path2D.add_child(pathFollow)
    
    # 创建实体
    var entity = preload("res://Entities/PatrolEntity.tscn").instantiate()
    pathFollow.add_child(entity)
    
    # 配置PathFollowComponent
    var pathComponent = entity.get_node("PathFollowComponent")
    pathComponent.speed = 50  # 每秒50像素
```

### 巡逻AI设置
```gdscript
# 创建巡逻敌人
class_name PatrolEnemy
extends Entity

@onready var pathFollow: PathFollowComponent = $PathFollowComponent
@onready var detection: AreaCollisionComponent = $DetectionArea

var isPatrolling: bool = true
var originalSpeed: float

func _ready():
    originalSpeed = pathFollow.speed
    
    # 监听路径完成
    pathFollow.didCompletePath.connect(_on_path_completed)
    
    # 监听玩家检测
    detection.didEnterBody.connect(_on_player_detected)
    detection.didExitBody.connect(_on_player_lost)

func _on_path_completed():
    print("巡逻路径完成")
    # 可以在这里切换路径或改变行为

func _on_player_detected(body: Node2D):
    if body.is_in_group("player"):
        print("发现玩家，停止巡逻")
        isPatrolling = false
        pathFollow.isEnabled = false
        # 切换到追击模式
        startChaseMode(body)

func _on_player_lost(body: Node2D):
    if body.is_in_group("player"):
        print("失去玩家，恢复巡逻")
        await get_tree().create_timer(2.0).timeout  # 等待2秒
        resumePatrol()

func startChaseMode(target: Node2D):
    var chaseComponent = get_node("ChaseComponent")
    if chaseComponent:
        chaseComponent.target = target
        chaseComponent.isEnabled = true

func resumePatrol():
    isPatrolling = true
    pathFollow.isEnabled = true
    # 恢复到最近的路径点
    pathFollow.snapEntityToCurve()
```

### 动态路径切换
```gdscript
# 支持多条路径的切换
class_name MultiPathFollower
extends Entity

@onready var pathFollow: PathFollowComponent = $PathFollowComponent

var availablePaths: Array[Path2D] = []
var currentPathIndex: int = 0

func _ready():
    # 预加载多条路径
    availablePaths = [
        preload("res://Paths/PatrolPath1.tscn").instantiate(),
        preload("res://Paths/PatrolPath2.tscn").instantiate(),
        preload("res://Paths/EscapePath.tscn").instantiate()
    ]
    
    # 添加路径到场景
    for path in availablePaths:
        get_parent().add_child(path)
    
    pathFollow.didCompletePath.connect(_on_path_completed)
    switchToPath(0)

func switchToPath(pathIndex: int):
    if pathIndex < 0 or pathIndex >= availablePaths.size():
        return
    
    currentPathIndex = pathIndex
    var newPath = availablePaths[pathIndex]
    
    # 重新设置PathFollow2D的父节点
    var currentPathFollow = get_parent()
    if currentPathFollow is PathFollow2D:
        currentPathFollow.reparent(newPath)
    
    # 重置进度
    pathFollow.shouldResetProgress = true
    pathFollow.unregisterEntity()
    pathFollow._ready()

func _on_path_completed():
    # 循环切换路径
    var nextIndex = (currentPathIndex + 1) % availablePaths.size()
    switchToPath(nextIndex)
```

### 带速度变化的路径跟随
```gdscript
# 根据路径段调整速度
class_name VariableSpeedPathFollower
extends Entity

@onready var pathFollow: PathFollowComponent = $PathFollowComponent

# 路径段速度配置
var speedSegments: Array[Dictionary] = [
    {"start": 0.0, "end": 0.3, "speed": 30},    # 慢速段
    {"start": 0.3, "end": 0.7, "speed": 80},    # 快速段
    {"start": 0.7, "end": 1.0, "speed": 50}     # 中速段
]

func _ready():
    pathFollow.isEnabled = true

func _process(_delta):
    if not pathFollow.pathFollower:
        return
    
    var currentProgress = pathFollow.pathFollower.progress_ratio
    updateSpeedBasedOnProgress(currentProgress)

func updateSpeedBasedOnProgress(progress: float):
    for segment in speedSegments:
        if progress >= segment.start and progress <= segment.end:
            pathFollow.speed = segment.speed
            break
```

### 路径动画同步
```gdscript
# 将路径移动与动画同步
class_name AnimatedPathFollower
extends Entity

@onready var pathFollow: PathFollowComponent = $PathFollowComponent
@onready var sprite: AnimatedSprite2D = $AnimatedSprite2D
@onready var pathFollowNode: PathFollow2D = get_parent()

var lastPosition: Vector2
var isMoving: bool = false

func _ready():
    lastPosition = global_position
    pathFollow.isEnabled = true

func _process(_delta):
    var currentPosition = global_position
    var velocity = currentPosition - lastPosition
    
    # 检测是否在移动
    var wasMoving = isMoving
    isMoving = velocity.length() > 1.0
    
    # 更新动画
    if isMoving and not wasMoving:
        sprite.play("walk")
    elif not isMoving and wasMoving:
        sprite.play("idle")
    
    # 根据移动方向翻转精灵
    if velocity.x < 0:
        sprite.flip_h = true
    elif velocity.x > 0:
        sprite.flip_h = false
    
    lastPosition = currentPosition
```

### 平滑停止和启动
```gdscript
# 支持平滑加速和减速的路径跟随
class_name SmoothPathFollower
extends Entity

@onready var pathFollow: PathFollowComponent = $PathFollowComponent

@export var acceleration: float = 100.0
@export var deceleration: float = 150.0
@export var maxSpeed: float = 80.0

var targetSpeed: float = 0.0
var currentSpeed: float = 0.0
var isAccelerating: bool = false

func _ready():
    pathFollow.speed = 0  # 从静止开始
    currentSpeed = 0

func startMoving():
    targetSpeed = maxSpeed
    isAccelerating = true

func stopMoving():
    targetSpeed = 0
    isAccelerating = false

func _process(delta):
    # 平滑调整速度
    if currentSpeed < targetSpeed:
        currentSpeed = min(targetSpeed, currentSpeed + acceleration * delta)
    elif currentSpeed > targetSpeed:
        currentSpeed = max(targetSpeed, currentSpeed - deceleration * delta)
    
    pathFollow.speed = currentSpeed
    
    # 如果完全停止，禁用路径跟随
    pathFollow.isEnabled = currentSpeed > 0.1

# 外部控制接口
func pauseMovement():
    stopMoving()

func resumeMovement():
    startMoving()

func setTargetSpeed(speed: float):
    targetSpeed = clamp(speed, 0, maxSpeed)
```

## 设计模式

### 继承模式
- **扩展基类：** 基于IndependentPathFollowComponent扩展
- **特化功能：** 专门处理PathFollow2D节点结构
- **代码复用：** 继承基础路径跟随逻辑

### 依赖注入模式
- **自动依赖：** 自动检测和设置PathFollow2D和Path2D
- **结构验证：** 确保正确的节点层次结构
- **错误处理：** 依赖缺失时的优雅处理

### 观察者模式
- **路径完成：** 通过信号通知路径完成事件
- **事件驱动：** 基于进度变化的事件触发
- **松耦合：** 移动逻辑与响应逻辑分离

## 技术细节

### 节点结构要求
```
Path2D
└── PathFollow2D
    └── Entity (with PathFollowComponent)
```

### 进度计算
```gdscript
pathFollower.progress += self.speed * delta
```

### 环绕检测
```gdscript
(previousProgressRatio >= 0.9 and currentProgressRatio <= 0.1)
```

### 位置变换
使用Path2D.to_local()和to_global()进行坐标空间转换。

## 性能考虑

1. **直接位置设置：** 避免物理计算开销
2. **进度重置：** 可选的自动重置减少手动管理
3. **旋转计算：** 只在需要时计算旋转
4. **环绕检测：** 简单的阈值检测避免复杂计算

## 注意事项

1. **共享PathFollow2D：** 避免多实体共享同一PathFollow2D
2. **节点结构：** 确保正确的父子关系
3. **进度重置：** shouldResetProgress会覆盖shouldSnapEntityToPath
4. **性能影响：** 大量实体时考虑使用IndependentPathFollowComponent
5. **坐标变换：** 注意不同坐标空间的转换

## 相关组件
- `IndependentPathFollowComponent` - 独立路径跟随基类
- `ChaseComponent` - 追踪移动，可与路径跟随切换
- `LinearMotionComponent` - 直线运动组件
- `CharacterBodyComponent` - 物理体移动组件