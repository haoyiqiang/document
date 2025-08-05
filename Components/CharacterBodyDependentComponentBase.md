# CharacterBodyDependentComponentBase API

> **继承关系**: Component > CharacterBodyDependentComponentBase  
> **抽象类**: 必须由子类实现  
> **依赖组件**: CharacterBodyComponent

CharacterBody2D依赖组件的抽象基类，为需要在CharacterBody2D移动前后执行操作的组件提供统一的访问接口。

## ✨ 主要特性

- 🎯 自动发现CharacterBodyComponent依赖
- 🔄 访问CharacterBody2D节点
- 📡 支持move_and_slide后的操作
- 🏗️ 为物理相关组件提供基础架构
- ⚡ 高效的组件引用缓存

## 🔗 依赖组件

### 必需组件
```gdscript
func getRequiredComponents() -> Array[Script]:
    return [CharacterBodyComponent]
```

**支持的CharacterBodyComponent类型**:
- CharacterBodyComponent - 基础角色物理体组件
- CharacterBodyComponent的所有子类

## 🎯 状态属性

### 组件引用
| 属性 | 类型 | 描述 |
|------|------|------|
| `characterBodyComponent` | `CharacterBodyComponent` | 自动发现的CharacterBodyComponent引用 |
| `body` | `CharacterBody2D` | CharacterBody2D节点的直接引用 |

## 🛠️ 主要方法

### 依赖获取
```gdscript
func getRequiredComponents() -> Array[Script]
```
**描述**: 返回必需的组件列表，确保CharacterBodyComponent存在

## 🎯 使用示例

### 创建物理依赖组件

```gdscript
# CustomPhysicsComponent.gd
class_name CustomPhysicsComponent
extends CharacterBodyDependentComponentBase

@export var appliedForce: Vector2 = Vector2.ZERO

func _ready():
    # 组件会自动查找CharacterBodyComponent
    if characterBodyComponent:
        print("Found CharacterBodyComponent: ", characterBodyComponent.name)
        # 连接到移动完成信号
        characterBodyComponent.didMove.connect(onCharacterMoved)
    else:
        print("Missing CharacterBodyComponent!")

func _physics_process(delta):
    # 在move_and_slide之前修改velocity
    if body:
        body.velocity += appliedForce * delta

func onCharacterMoved():
    # 在move_and_slide之后执行的操作
    if body.is_on_floor():
        print("Character landed on floor")
```

### 速度监控组件

```gdscript
# VelocityMonitorComponent.gd
class_name VelocityMonitorComponent
extends CharacterBodyDependentComponentBase

@export var maxSpeed: float = 500.0
@export var enableSpeedLimit: bool = true

signal speedLimitExceeded(currentSpeed: float)

func _ready():
    super._ready()
    if characterBodyComponent:
        characterBodyComponent.didMove.connect(onAfterMovement)

func _physics_process(_delta):
    # 移动前检查和限制速度
    if body and enableSpeedLimit:
        var currentSpeed = body.velocity.length()
        
        if currentSpeed > maxSpeed:
            speedLimitExceeded.emit(currentSpeed)
            # 限制速度
            body.velocity = body.velocity.normalized() * maxSpeed

func onAfterMovement():
    # 移动后的速度分析
    var speed = body.velocity.length()
    Debug.addWatchValue("Current Speed", speed)
    Debug.addWatchValue("Is On Floor", body.is_on_floor())
```

### 摩擦力组件

```gdscript
# FrictionComponent.gd
class_name FrictionComponent
extends CharacterBodyDependentComponentBase

@export var groundFriction: float = 0.8
@export var airFriction: float = 0.95
@export var isEnabled: bool = true

func _physics_process(delta):
    if not isEnabled or not body:
        return
    
    # 在移动前应用摩擦力
    var friction = airFriction
    if body.is_on_floor():
        friction = groundFriction
    
    body.velocity.x *= friction
```

### 碰撞检测组件

```gdscript
# CollisionDetectorComponent.gd
class_name CollisionDetectorComponent
extends CharacterBodyDependentComponentBase

signal hitWall(normal: Vector2)
signal hitCeiling()
signal landedOnFloor()

var wasOnFloor: bool = false
var wasOnWall: bool = false

func _ready():
    super._ready()
    if characterBodyComponent:
        characterBodyComponent.didMove.connect(onAfterMovement)

func onAfterMovement():
    if not body:
        return
    
    # 检测新的碰撞状态
    checkFloorCollision()
    checkWallCollision()
    checkCeilingCollision()
    
    # 更新状态
    wasOnFloor = body.is_on_floor()
    wasOnWall = body.is_on_wall()

func checkFloorCollision():
    if body.is_on_floor() and not wasOnFloor:
        landedOnFloor.emit()

func checkWallCollision():
    if body.is_on_wall() and not wasOnWall:
        hitWall.emit(body.get_wall_normal())

func checkCeilingCollision():
    if body.is_on_ceiling():
        hitCeiling.emit()
```

### 平台跳跃组件

```gdscript
# PlatformJumpComponent.gd
class_name PlatformJumpComponent
extends CharacterBodyDependentComponentBase

@export var jumpStrength: float = 400.0
@export var coyoteTime: float = 0.1

var coyoteTimer: float = 0.0
var wasOnFloor: bool = false

func _ready():
    super._ready()
    if characterBodyComponent:
        characterBodyComponent.didMove.connect(onAfterMovement)

func _physics_process(delta):
    # 更新土狼时间
    if not body.is_on_floor():
        coyoteTimer += delta
    else:
        coyoteTimer = 0.0

func onAfterMovement():
    # 检测是否刚离开地面
    if wasOnFloor and not body.is_on_floor():
        # 开始土狼时间计时
        coyoteTimer = 0.0
    
    wasOnFloor = body.is_on_floor()

func canJump() -> bool:
    return body.is_on_floor() or coyoteTimer <= coyoteTime

func performJump():
    if canJump():
        body.velocity.y = -jumpStrength
        coyoteTimer = coyoteTime + 0.1  # 防止重复跳跃
```

## 🏛️ 设计模式

### 依赖注入模式
- **依赖**: CharacterBodyComponent提供CharacterBody2D访问
- **注入**: 自动发现和缓存组件引用
- **解耦**: 通过基类统一依赖管理

### 观察者模式
- **主体**: CharacterBodyComponent的didMove信号
- **观察者**: 依赖组件监听移动完成事件
- **响应**: 在移动后执行特定逻辑

## 🔧 技术细节

### 自动组件发现
```gdscript
var characterBodyComponent: CharacterBodyComponent:
    get:
        if not characterBodyComponent:
            characterBodyComponent = parentEntity.findFirstComponentSubclass(CharacterBodyComponent)
            if not characterBodyComponent:
                printError("Missing CharacterBody2D in parent Entity")
        return characterBodyComponent
```

### CharacterBody2D访问
```gdscript
var body: CharacterBody2D:
    get:
        if not body: body = characterBodyComponent.body
        return body
```

### 移动周期集成
```
_physics_process() -> 移动前操作
    ↓
CharacterBody2D.move_and_slide()
    ↓
CharacterBodyComponent.didMove信号 -> 移动后操作
```

## ⚠️ 注意事项

1. **抽象类**: 不能直接实例化，必须通过子类使用
2. **依赖检查**: 确保父实体包含CharacterBodyComponent
3. **移动时机**: 在_physics_process中进行移动前操作，在didMove信号中进行移动后操作
4. **性能优化**: 组件引用使用延迟加载和缓存机制
5. **错误处理**: 缺少依赖组件时会输出错误信息

## 🔗 相关组件

- [CharacterBodyComponent](CharacterBodyComponent.md) - 角色物理体组件
- [PlatformerPhysicsComponent](PlatformerPhysicsComponent.md) - 平台游戏物理
- [GravityComponent](GravityComponent.md) - 重力组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 