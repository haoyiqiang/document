# ThrustControlComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > ThrustControlComponent  
> **控制类型**: 推进/制动控制

为实体的CharacterBody2D提供推进和摩擦控制的组件，支持太空船或坦克式的前进/后退控制。可与TurningControlComponent组合实现类似小行星的运动控制。

## ✨ 主要特性

- 🚀 基于方向的推进控制
- 🛑 摩擦制动系统
- 🎮 上下键控制推进力
- 🌌 浮动运动模式
- 🔄 可启用/禁用功能

## 📊 导出属性

### 推进设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `thrust` | `float` | `100.0` | 推进力大小 (0-1000) |
| `friction` | `float` | `50.0` | 摩擦力大小 (0-1000) |
| `isEnabled` | `bool` | `true` | 是否启用推进控制 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `InputComponent` | **之后** | 提供推进输入 |
| `CharacterBodyComponent` | **之前** | 处理移动和碰撞 |

### 组件属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `inputComponent` | `InputComponent` | 对输入组件的引用 |

## 🎮 输入控制

### 推进输入
- **前进**: `moveForward` / W / ↑
- **后退**: `moveBackward` / S / ↓
- **输入值**: InputComponent.thrustInput (-1.0 到 1.0)

## 🎯 使用示例

### 基础太空船控制

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── InputComponent
#     ├── TurningControlComponent  # 转向控制
#     └── ThrustControlComponent   # 推进控制
```

### 配合速度限制

```gdscript
# 添加VelocityClampComponent限制最大速度
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── InputComponent
#     ├── ThrustControlComponent
#     └── VelocityClampComponent (maxVelocity: 200)
```

### 动态推进力调整

```gdscript
# VariableThrustComponent.gd
extends ThrustControlComponent

@export var boostMultiplier: float = 2.0
@export var boostDuration: float = 3.0
@export var boostCooldown: float = 5.0

var isBoosting: bool = false
var boostTimer: float = 0.0
var cooldownTimer: float = 0.0
var baseThrustForce: float

func _ready():
    super._ready()
    baseThrustForce = thrust

func _input(event: InputEvent):
    if event.is_action_pressed("boost") and canBoost():
        activateBoost()

func _physics_process(delta: float):
    updateBoostState(delta)
    super._physics_process(delta)

func canBoost() -> bool:
    return not isBoosting and cooldownTimer <= 0.0

func activateBoost():
    isBoosting = true
    boostTimer = boostDuration
    thrust = baseThrustForce * boostMultiplier
    print("Boost activated!")

func updateBoostState(delta: float):
    if isBoosting:
        boostTimer -= delta
        if boostTimer <= 0.0:
            deactivateBoost()
    elif cooldownTimer > 0.0:
        cooldownTimer -= delta

func deactivateBoost():
    isBoosting = false
    cooldownTimer = boostCooldown
    thrust = baseThrustForce
    print("Boost deactivated, cooldown started")
```

### 燃料系统集成

```gdscript
# FuelThrustComponent.gd
extends ThrustControlComponent

@export var maxFuel: float = 100.0
@export var fuelConsumption: float = 10.0  # 每秒消耗
@export var fuelRegenRate: float = 5.0     # 每秒恢复

var currentFuel: float
var isThrustingLastFrame: bool = false

func _ready():
    super._ready()
    currentFuel = maxFuel

func _physics_process(delta: float):
    var thrustInput = inputComponent.thrustInput
    var isThrusting = not is_zero_approx(thrustInput)
    
    # 检查燃料
    if isThrusting and currentFuel <= 0.0:
        # 没有燃料时禁用推进
        return
    
    # 消耗或恢复燃料
    if isThrusting:
        currentFuel = max(0.0, currentFuel - fuelConsumption * delta)
        # 推进力根据燃料调整
        var fuelRatio = currentFuel / maxFuel
        var originalThrust = thrust
        thrust = thrust * (0.3 + 0.7 * fuelRatio)  # 最低30%推力
        
        super._physics_process(delta)
        
        thrust = originalThrust  # 恢复原始值
    else:
        # 燃料恢复
        currentFuel = min(maxFuel, currentFuel + fuelRegenRate * delta)
    
    # 燃料用尽警告
    if currentFuel <= 0.0 and not isThrustingLastFrame:
        showFuelEmptyWarning()
    
    isThrustingLastFrame = isThrusting

func showFuelEmptyWarning():
    print("Fuel empty!")
    # 显示UI警告或播放音效

func getFuelPercentage() -> float:
    return currentFuel / maxFuel
```

### 制动增强组件

```gdscript
# EnhancedThrustComponent.gd
extends ThrustControlComponent

@export var brakingMultiplier: float = 2.0
@export var reverseThrustRatio: float = 0.7
@export var emergencyBrakeForce: float = 300.0

func _input(event: InputEvent):
    if event.is_action_pressed("emergency_brake"):
        emergencyBrake()

func _physics_process(delta: float):
    var thrustInput = inputComponent.thrustInput
    
    if not is_zero_approx(thrustInput):
        applyThrust(thrustInput, delta)
    else:
        applyFriction(delta)
    
    characterBodyComponent.shouldMoveThisFrame = true

func applyThrust(input: float, delta: float):
    var direction = Vector2.from_angle(body.rotation)
    var velocityDirection = body.velocity.normalized()
    var movementDirection = direction * input
    
    # 检查是否在减速（反向推进）
    var isDecelerating = body.velocity.dot(movementDirection) < 0
    
    var effectiveThrust = thrust
    if input < 0:  # 后退推进
        effectiveThrust *= reverseThrustRatio
    elif isDecelerating:  # 减速推进
        effectiveThrust *= brakingMultiplier
    
    body.velocity += movementDirection * effectiveThrust * delta

func applyFriction(delta: float):
    # 增强的摩擦系统
    var currentSpeed = body.velocity.length()
    if currentSpeed > 0:
        var frictionForce = min(friction * delta, currentSpeed)
        body.velocity = body.velocity.normalized() * (currentSpeed - frictionForce)

func emergencyBrake():
    var currentSpeed = body.velocity.length()
    if currentSpeed > 0:
        var brakeAmount = min(emergencyBrakeForce, currentSpeed)
        body.velocity = body.velocity.normalized() * (currentSpeed - brakeAmount)
        
        # 视觉效果
        showBrakeEffect()

func showBrakeEffect():
    # 添加制动特效
    print("Emergency brake activated!")
```

## 🔧 技术细节

### 推进力计算
```gdscript
func _physics_process(delta: float) -> void:
    var input: float = inputComponent.thrustInput
    
    if not is_zero_approx(input):
        var direction: Vector2 = Vector2.from_angle(body.rotation)
        body.velocity += direction * thrust * delta
    else:
        body.velocity = body.velocity.move_toward(Vector2.ZERO, friction * delta)
    
    characterBodyComponent.shouldMoveThisFrame = true
```

### 运动模式设置
```gdscript
func _ready() -> void:
    if parentEntity.body:
        parentEntity.body.motion_mode = CharacterBody2D.MOTION_MODE_FLOATING
```

### 性能优化
- **条件处理**: 使用isEnabled setter优化_physics_process调用
- **向量计算**: 使用`Vector2.from_angle()`直接获取单位向量
- **摩擦应用**: 使用`move_toward()`平滑减速

## ⚠️ 注意事项

1. **运动模式**: 自动设置CharacterBody2D为FLOATING模式
2. **输入依赖**: 需要InputComponent提供thrustInput
3. **处理顺序**: 在InputComponent之后，CharacterBodyComponent之前
4. **组合使用**: 推荐与TurningControlComponent和VelocityClampComponent组合
5. **待开发功能**: 制动和碰撞速度重置功能还在开发中

## 🔗 相关组件

- [TurningControlComponent](TurningControlComponent.md) - 转向控制组件
- [AsteroidsControlComponent](AsteroidsControlComponent.md) - 小行星风格控制
- [VelocityClampComponent](./VelocityClampComponent.md) - 速度限制组件
- [CharacterBodyDependentComponentBase](./CharacterBodyDependentComponentBase.md) - 基类组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 