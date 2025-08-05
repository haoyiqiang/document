# ScrollerControlComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > ScrollerControlComponent  
> **控制类型**: 横版卷轴控制  
> **依赖组件**: CharacterBodyComponent（之前）

专为横版卷轴游戏设计的控制组件，应用恒定的水平/垂直推力并维持各轴的最小速度。适用于飞行射击游戏、跑酷游戏等需要持续移动的游戏类型。

## ✨ 主要特性

- 🚀 恒定推力系统
- 📏 分轴速度控制
- ⚡ 加速度和摩擦力支持
- 🎮 玩家输入响应
- 📹 可选摄像机弹簧效果
- 🔄 速度限制和维持

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `CharacterBodyComponent` | **之前** | 提供CharacterBody2D物理处理 |

### 可选组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `Camera2D` | **子节点** | 提供摄像机跟随功能 |

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `isEnabled` | `bool` | `true` | 是否启用组件 |
| `parameters` | `ScrollerMovementParameters` | `new()` | 运动参数配置 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `camera` | `Camera2D` | 摄像机引用 |
| `inputDirection` | `Vector2` | 当前输入方向 |
| `lastInputDirection` | `Vector2` | 上一帧输入方向 |
| `lastDirection` | `Vector2` | 标准化的移动方向 |

## ⚙️ ScrollerMovementParameters

### 速度控制
| 参数 | 类型 | 描述 |
|------|------|------|
| `horizontalSpeed` | `float` | 水平移动速度 |
| `verticalSpeed` | `float` | 垂直移动速度 |
| `horizontalVelocityDefault` | `float` | 默认水平速度（无输入时） |
| `verticalVelocityDefault` | `float` | 默认垂直速度（无输入时） |

### 速度限制
| 参数 | 类型 | 描述 |
|------|------|------|
| `horizontalVelocityMin` | `float` | 最小水平速度 |
| `horizontalVelocityMax` | `float` | 最大水平速度 |
| `verticalVelocityMin` | `float` | 最小垂直速度 |
| `verticalVelocityMax` | `float` | 最大垂直速度 |

### 物理属性
| 参数 | 类型 | 描述 |
|------|------|------|
| `shouldApplyAcceleration` | `bool` | 是否应用加速度 |
| `acceleration` | `float` | 加速度值 |
| `shouldApplyFriction` | `bool` | 是否应用摩擦力 |
| `friction` | `float` | 摩擦力值 |
| `shouldMaintainPreviousVelocity` | `bool` | 是否维持前一帧速度 |

## 🎯 使用示例

### 基础飞行射击游戏

```gdscript
# Entity Scene Structure:
# └── Spaceship (CharacterBody2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     ├── CharacterBodyComponent
#     └── ScrollerControlComponent
#         parameters: [ScrollerMovementParameters Resource]
#           horizontalSpeed: 300.0
#           verticalSpeed: 250.0
#           horizontalVelocityDefault: 100.0  # 持续向右移动
#           verticalVelocityDefault: 0.0
```

### 跑酷游戏配置

```gdscript
# Entity Scene Structure:
# └── Runner (CharacterBody2D)
#     ├── Sprite2D
#     ├── CollisionShape2D
#     ├── CharacterBodyComponent
#     └── ScrollerControlComponent
#         parameters: [ScrollerMovementParameters Resource]
#           horizontalSpeed: 400.0
#           verticalSpeed: 300.0
#           horizontalVelocityDefault: 200.0  # 持续跑步
#           shouldApplyAcceleration: true
#           acceleration: 800.0
#           shouldApplyFriction: true
#           friction: 600.0
```

### 智能卷轴控制组件

```gdscript
# SmartScrollerComponent.gd
extends ScrollerControlComponent

@export var speedBoostMultiplier: float = 1.5
@export var speedBoostDuration: float = 3.0
@export var autoAdjustSpeed: bool = true
@export var targetFrameRate: float = 60.0

var isSpeedBoosted: bool = false
var speedBoostTimer: float = 0.0
var baseParameters: ScrollerMovementParameters

func _ready():
    super._ready()
    baseParameters = parameters.duplicate()

func _physics_process(delta: float):
    # 自动调整速度以保持帧率
    if autoAdjustSpeed:
        adjustSpeedForPerformance(delta)
    
    # 处理速度提升
    processSpeedBoost(delta)
    
    super._physics_process(delta)

func adjustSpeedForPerformance(delta: float):
    var currentFPS = 1.0 / delta
    var fpsRatio = currentFPS / targetFrameRate
    
    if fpsRatio < 0.9:  # 帧率太低，降低速度
        parameters.horizontalSpeed = baseParameters.horizontalSpeed * 0.95
        parameters.verticalSpeed = baseParameters.verticalSpeed * 0.95
    elif fpsRatio > 1.1:  # 帧率充足，可以提高速度
        parameters.horizontalSpeed = min(
            baseParameters.horizontalSpeed * 1.05,
            baseParameters.horizontalSpeed * 1.2  # 最大120%
        )
        parameters.verticalSpeed = min(
            baseParameters.verticalSpeed * 1.05,
            baseParameters.verticalSpeed * 1.2
        )

func processSpeedBoost(delta: float):
    if isSpeedBoosted:
        speedBoostTimer -= delta
        if speedBoostTimer <= 0.0:
            endSpeedBoost()

func activateSpeedBoost():
    if isSpeedBoosted:
        return  # 已经在加速状态
    
    isSpeedBoosted = true
    speedBoostTimer = speedBoostDuration
    
    # 应用速度提升
    parameters.horizontalSpeed = baseParameters.horizontalSpeed * speedBoostMultiplier
    parameters.verticalSpeed = baseParameters.verticalSpeed * speedBoostMultiplier
    parameters.acceleration = baseParameters.acceleration * speedBoostMultiplier
    
    print("Speed boost activated!")

func endSpeedBoost():
    isSpeedBoosted = false
    
    # 恢复基础速度
    parameters.horizontalSpeed = baseParameters.horizontalSpeed
    parameters.verticalSpeed = baseParameters.verticalSpeed
    parameters.acceleration = baseParameters.acceleration
    
    print("Speed boost ended")

func _input(event: InputEvent):
    if event.is_action_pressed("speed_boost"):
        activateSpeedBoost()
```

### 自适应难度卷轴

```gdscript
# AdaptiveScrollerComponent.gd
extends ScrollerControlComponent

@export var difficultyLevel: int = 1
@export var maxDifficultyLevel: int = 10
@export var difficultyIncreaseRate: float = 0.1
@export var playerSkillThreshold: float = 0.8

var timeSurvived: float = 0.0
var damagesTaken: int = 0
var playerSkillScore: float = 1.0

func _ready():
    super._ready()
    adjustParametersForDifficulty()

func _physics_process(delta: float):
    super._physics_process(delta)
    
    timeSurvived += delta
    updateDifficulty(delta)

func updateDifficulty(delta: float):
    # 基于时间增加难度
    var newDifficultyLevel = int(timeSurvived * difficultyIncreaseRate) + 1
    newDifficultyLevel = min(newDifficultyLevel, maxDifficultyLevel)
    
    if newDifficultyLevel != difficultyLevel:
        difficultyLevel = newDifficultyLevel
        adjustParametersForDifficulty()
        print("Difficulty increased to level ", difficultyLevel)

func adjustParametersForDifficulty():
    # 根据难度调整速度
    var speedMultiplier = 1.0 + (difficultyLevel - 1) * 0.1
    
    parameters.horizontalVelocityDefault = baseParameters.horizontalVelocityDefault * speedMultiplier
    parameters.verticalSpeed = baseParameters.verticalSpeed * speedMultiplier
    
    # 根据玩家技能调整
    if playerSkillScore > playerSkillThreshold:
        speedMultiplier *= 1.2  # 技能好的玩家，增加挑战
    
    print("Speed adjusted for difficulty level ", difficultyLevel, ", multiplier: ", speedMultiplier)

func onPlayerTookDamage():
    damagesTaken += 1
    updatePlayerSkillScore()

func updatePlayerSkillScore():
    # 计算玩家技能分数（存活时间vs受伤次数）
    if timeSurvived > 0:
        playerSkillScore = max(0.1, 1.0 - (damagesTaken / (timeSurvived / 10.0)))
    
    print("Player skill score: ", playerSkillScore)
```

### 动态环境卷轴

```gdscript
# DynamicEnvironmentScroller.gd
extends ScrollerControlComponent

@export var windForce: Vector2 = Vector2.ZERO
@export var turbulenceStrength: float = 0.0
@export var environmentEffects: Array[String] = []

var turbulenceTimer: float = 0.0

func _physics_process(delta: float):
    # 应用环境效果
    applyEnvironmentalForces(delta)
    
    super._physics_process(delta)

func applyEnvironmentalForces(delta: float):
    # 应用风力
    if windForce.length() > 0:
        body.velocity += windForce * delta
    
    # 应用湍流
    if turbulenceStrength > 0:
        turbulenceTimer += delta
        var turbulence = Vector2(
            sin(turbulenceTimer * 10.0) * turbulenceStrength,
            cos(turbulenceTimer * 15.0) * turbulenceStrength
        )
        body.velocity += turbulence
    
    # 处理特殊环境效果
    for effect in environmentEffects:
        applyEnvironmentEffect(effect, delta)

func applyEnvironmentEffect(effect: String, delta: float):
    match effect:
        "underwater":
            # 水下阻力
            parameters.friction *= 1.5
            parameters.horizontalSpeed *= 0.8
            parameters.verticalSpeed *= 0.8
        "zero_gravity":
            # 零重力
            parameters.verticalVelocityDefault = 0.0
            parameters.shouldApplyFriction = false
        "storm":
            # 风暴效果
            var stormForce = Vector2(
                randf_range(-100, 100),
                randf_range(-50, 50)
            )
            body.velocity += stormForce * delta
        "speed_zone":
            # 加速区域
            parameters.horizontalVelocityDefault *= 1.3

func setWindForce(force: Vector2):
    windForce = force
    print("Wind force set to: ", windForce)

func setTurbulence(strength: float):
    turbulenceStrength = strength
    print("Turbulence strength set to: ", turbulenceStrength)

func addEnvironmentEffect(effect: String):
    if effect not in environmentEffects:
        environmentEffects.append(effect)
        print("Environment effect added: ", effect)

func removeEnvironmentEffect(effect: String):
    environmentEffects.erase(effect)
    print("Environment effect removed: ", effect)

func clearEnvironmentEffects():
    environmentEffects.clear()
    print("All environment effects cleared")
```

## 🔧 技术细节

### 物理设置
```gdscript
func _ready() -> void:
    if parentEntity.body:
        parentEntity.body.motion_mode = CharacterBody2D.MOTION_MODE_FLOATING
        parentEntity.body.wall_min_slide_angle = 0  # 允许侧向滑动
```

### 输入处理流程
```gdscript
func processInput(delta: float) -> void:
    inputDirection = Input.get_vector(
        GlobalInput.Actions.moveLeft,
        GlobalInput.Actions.moveRight,
        GlobalInput.Actions.moveUp,
        GlobalInput.Actions.moveDown
    )
    
    # 计算输入速度
    var inputVelocity = Vector2(
        inputDirection.x * parameters.horizontalSpeed,
        inputDirection.y * parameters.verticalSpeed
    )
    
    # 应用加速度或直接设置
    if parameters.shouldApplyAcceleration:
        body.velocity = body.velocity.move_toward(inputVelocity, parameters.acceleration * delta)
    else:
        body.velocity = inputVelocity
```

### 默认速度维持
```gdscript
# 无输入时维持默认速度
if is_zero_approx(inputDirection.x):
    body.velocity.x = parameters.horizontalVelocityDefault

if is_zero_approx(inputDirection.y):
    body.velocity.y = parameters.verticalVelocityDefault
```

## ⚠️ 注意事项

1. **依赖顺序**: 必须在CharacterBodyComponent之后添加
2. **浮动模式**: 自动设置CharacterBody2D为FLOATING模式
3. **滑动角度**: 设置wall_min_slide_angle为0，允许完全滑动
4. **性能优化**: 通过isEnabled属性控制physics_process
5. **摄像机**: 自动查找子节点中的Camera2D
6. **速度维持**: 可能与其他移动组件冲突

## 🔗 相关组件

- [CharacterBodyComponent](../Physics/CharacterBodyComponent.md) - 角色身体组件
- [PlatformerControlComponent](PlatformerControlComponent.md) - 平台控制组件
- [InputComponent](InputComponent.md) - 输入管理组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 