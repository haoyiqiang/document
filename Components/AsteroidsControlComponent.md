# AsteroidsControlComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > AsteroidsControlComponent  
> **控制类型**: 小行星风格控制  
> **实验性功能**

提供类似小行星游戏的控制方式，支持旋转和推进的太空船控制。这是TurningControlComponent和ThrustControlComponent的高级独立替代方案，不依赖InputComponent。

## ✨ 主要特性

- 🚀 小行星风格的太空船控制
- 🎮 独立的输入处理（不依赖InputComponent）
- 🔄 左右旋转 + 前进推进
- 🌌 浮动运动模式
- 🎛️ 高级运动参数配置

## 📊 导出属性

### 控制设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `isEnabled` | `bool` | `true` | 是否启用控制 |
| `parameters` | `AsteroidsMovementParameters` | 新实例 | 运动参数配置 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `horizontalInput` | `float` | 转向输入值 |
| `verticalInput` | `float` | 推进输入值 |
| `inputDirection` | `Vector2` | 当前输入方向 |
| `lastInputDirection` | `Vector2` | 上一帧输入方向 |

## 🎮 输入映射

### 支持的控制输入
| 动作 | 输入键 | 功能 |
|------|--------|------|
| `turnLeft` | A / ← | 逆时针旋转 |
| `turnRight` | D / → | 顺时针旋转 |
| `moveForward` | W / ↑ | 向前推进 |
| `moveBackward` | S / ↓ | 向后推进 |

## 🎯 使用示例

### 基础小行星控制

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     └── AsteroidsControlComponent
```

### 自定义运动参数

```gdscript
# 配置AsteroidsMovementParameters
@export var customParameters: AsteroidsMovementParameters

func _ready():
    customParameters = AsteroidsMovementParameters.new()
    customParameters.turningSpeed = 3.0
    customParameters.thrust = 200.0
    customParameters.acceleration = 150.0
    customParameters.friction = 75.0
    
    $AsteroidsControlComponent.parameters = customParameters
```

### 增强控制组件

```gdscript
# EnhancedAsteroidsController.gd
extends AsteroidsControlComponent

@export var boostMultiplier: float = 2.0
@export var enableStrafing: bool = false

func _physics_process(delta: float):
    if Input.is_action_pressed("boost"):
        var originalThrust = parameters.thrust
        parameters.thrust *= boostMultiplier
        super._physics_process(delta)
        parameters.thrust = originalThrust
    else:
        super._physics_process(delta)
    
    if enableStrafing:
        handleStrafing(delta)

func handleStrafing(delta: float):
    var strafeInput = Input.get_axis("strafe_left", "strafe_right")
    if not is_zero_approx(strafeInput):
        var strafeDirection = Vector2.from_angle(body.rotation + PI/2)
        body.velocity += strafeDirection * strafeInput * parameters.thrust * 0.5 * delta
```

## 🔧 技术细节

### 运动处理流程
```gdscript
func processInput(delta: float):
    # 获取输入
    horizontalInput = Input.get_axis(turnLeft, turnRight)
    verticalInput = Input.get_axis(moveBackward, moveForward)
    
    # 旋转
    if horizontalInput:
        body.rotation += parameters.turningSpeed * horizontalInput * delta
    
    # 推进
    var bodyDirection = Vector2.from_angle(body.rotation)
    body.velocity += bodyDirection * verticalInput * parameters.thrust * delta
    
    # 摩擦
    if parameters.shouldApplyFriction and is_zero_approx(verticalInput):
        body.velocity = body.velocity.move_toward(Vector2.ZERO, parameters.friction * delta)
```

### 运动模式设置
```gdscript
func _ready():
    parentEntity.body.motion_mode = CharacterBody2D.MOTION_MODE_FLOATING
```

## ⚠️ 注意事项

1. **实验性功能**: 组件标记为实验性，API可能变化
2. **独立设计**: 不依赖InputComponent，有自己的输入处理
3. **运动模式**: 自动设置为FLOATING模式
4. **参数配置**: 使用AsteroidsMovementParameters资源配置运动行为
5. **性能优化**: 包含自动速度重置以避免"粘墙效应"

## 🔗 相关组件

- [ThrustControlComponent](ThrustControlComponent.md) - 推进控制组件
- [TurningControlComponent](TurningControlComponent.md) - 转向控制组件
- [CharacterBodyDependentComponentBase](./CharacterBodyDependentComponentBase.md) - 基类组件
- [VelocityClampComponent](./VelocityClampComponent.md) - 速度限制组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 