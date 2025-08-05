# PlatformerPhysicsComponent API

## 概述
`PlatformerPhysicsComponent` 是平台游戏物理系统的核心组件，为实体的CharacterBody2D处理重力和摩擦力。它允许玩家角色和怪物共享相同的移动逻辑，专门设计用于2D平台游戏。

**继承链**：`Component` → `CharacterBodyDependentComponentBase` → `PlatformerPhysicsComponent`

**⚠️ 警告**：不要与`GravityComponent`同时使用，因为两者都处理重力，会导致重力过强！

## 依赖要求
- **CharacterBodyComponent**：管理CharacterBody2D
- **InputComponent**（可选）：提供输入控制

## 组件顺序要求
- **之前**：CharacterBodyComponent、InputComponent
- **之后**：其他物理修改组件

## 主要特性
- 🌍 **重力系统**：完整的重力处理，支持重力缩放
- 🏃 **水平移动**：地面和空中的加速度控制
- 🛑 **摩擦系统**：地面和空中的摩擦力处理
- ⚡ **性能优化**：可临时跳过加速度和摩擦力计算
- 📊 **状态管理**：跟踪实体移动状态（待机、地面移动、空中移动）
- 🎛️ **灵活配置**：通过PlatformerMovementParameters资源配置

## 导出属性

### 物理配置
- **gravityScaleOverride** (`float = 1.0`, -10到10)：重力缩放覆盖值
- **isGravityEnabled** (`bool = true`)：是否启用重力（用于攀爬、飞行等）
- **isEnabled** (`bool = true`)：是否启用整个组件
- **parameters** (`PlatformerMovementParameters`)：移动参数资源

### 临时控制标志
- **shouldSkipAcceleration** (`bool`)：跳过当前帧的加速度计算
- **shouldSkipFriction** (`bool`)：跳过当前帧的摩擦力计算

## 状态枚举
```gdscript
enum State { idle, moveOnFloor, moveInAir }
```

## 状态属性
- **currentState** (`State`)：当前移动状态
- **isInputZero** (`bool`)：输入是否为零
- **horizontalInput** (`float`)：水平输入值（从InputComponent复制）
- **lastNonzeroHorizontalInput** (`float`)：最后一次非零的水平输入
- **gravity** (`float`)：当前重力值

## 主要方法

### 物理处理
- **processGravity**(delta: float) → `void`：处理重力加速度
- **processHorizontalMovement**(delta: float) → `void`：处理水平移动和加速度
- **processAllFriction**(delta: float) → `void`：处理地面和空中摩擦力

### 状态管理
- **updateStateBeforeMove**() → `void`：移动前更新状态

## PlatformerMovementParameters 属性

### 速度配置
- **speedOnFloor**：地面移动速度
- **speedInAir**：空中移动速度

### 加速度配置
- **shouldApplyAccelerationOnFloor**：地面是否渐进加速
- **shouldApplyAccelerationInAir**：空中是否渐进加速
- **accelerationOnFloor**：地面加速度
- **accelerationInAir**：空中加速度

### 摩擦力配置
- **shouldApplyFrictionOnFloor**：地面是否应用摩擦力
- **shouldApplyFrictionInAir**：空中是否应用摩擦力
- **frictionOnFloor**：地面摩擦力
- **frictionInAir**：空中摩擦力
- **shouldStopInstantlyOnFloor**：地面是否瞬间停止
- **shouldStopInstantlyInAir**：空中是否瞬间停止

### 其他配置
- **shouldAllowMovementInputInAir**：是否允许空中移动输入
- **gravityScale**：重力缩放系数

## 使用示例

### 基础平台物理
```gdscript
# 设置基本的平台物理组件
@export var physics_component: PlatformerPhysicsComponent
@export var movement_params: PlatformerMovementParameters

func _ready():
    physics_component.parameters = movement_params
    physics_component.gravityScaleOverride = 1.0
    physics_component.isGravityEnabled = true
```

### 自定义移动参数
```gdscript
# 创建自定义的移动参数
func setup_movement_parameters():
    var params = PlatformerMovementParameters.new()
    
    # 速度设置
    params.speedOnFloor = 200.0
    params.speedInAir = 150.0
    
    # 加速度设置
    params.shouldApplyAccelerationOnFloor = true
    params.accelerationOnFloor = 1000.0
    params.shouldApplyAccelerationInAir = true
    params.accelerationInAir = 500.0
    
    # 摩擦力设置
    params.shouldApplyFrictionOnFloor = true
    params.frictionOnFloor = 800.0
    params.shouldStopInstantlyOnFloor = false
    
    # 空中控制
    params.shouldAllowMovementInputInAir = true
    
    physics_component.parameters = params
```

### 特殊移动状态
```gdscript
# 实现特殊移动状态（如冲刺、滑行）
@export var physics_component: PlatformerPhysicsComponent

func dash():
    # 冲刺时跳过摩擦力
    physics_component.shouldSkipFriction = true
    # 添加冲刺力
    physics_component.body.velocity.x = dash_speed * get_facing_direction()

func start_sliding():
    # 滑行时跳过加速度
    physics_component.shouldSkipAcceleration = true
    # 减少摩擦力
    var original_friction = physics_component.parameters.frictionOnFloor
    physics_component.parameters.frictionOnFloor = original_friction * 0.1

func stop_sliding():
    # 恢复正常摩擦力
    physics_component.parameters.frictionOnFloor = normal_friction_value
```

### 环境特效
```gdscript
# 不同环境的物理效果
@export var physics_component: PlatformerPhysicsComponent

func enter_water():
    # 水中物理：降低重力和速度
    physics_component.gravityScaleOverride = 0.3
    physics_component.parameters.speedOnFloor *= 0.5
    physics_component.parameters.speedInAir *= 0.7

func enter_ice():
    # 冰面物理：减少摩擦力
    physics_component.parameters.frictionOnFloor *= 0.1
    physics_component.parameters.shouldStopInstantlyOnFloor = false

func enter_quicksand():
    # 流沙物理：增加摩擦，降低速度
    physics_component.parameters.frictionOnFloor *= 3.0
    physics_component.parameters.speedOnFloor *= 0.3
```

### 角色特性系统
```gdscript
# 基于角色属性的物理调整
@export var physics_component: PlatformerPhysicsComponent
@export var character_stats: CharacterStats

func apply_character_physics():
    var params = physics_component.parameters
    
    # 根据速度属性调整移动
    var speed_multiplier = 1.0 + (character_stats.speed - 10) * 0.1
    params.speedOnFloor *= speed_multiplier
    params.speedInAir *= speed_multiplier
    
    # 根据重量调整重力和加速度
    var weight_factor = character_stats.weight / 100.0
    physics_component.gravityScaleOverride = weight_factor
    params.accelerationOnFloor /= weight_factor
```

### 动画集成
```gdscript
# 与动画系统集成
@export var physics_component: PlatformerPhysicsComponent
@export var animation_component: PlatformerAnimationComponent

func _ready():
    # 监听状态变化
    pass

func _physics_process(_delta):
    # 根据物理状态更新动画
    match physics_component.currentState:
        PlatformerPhysicsComponent.State.idle:
            animation_component.play_idle()
        PlatformerPhysicsComponent.State.moveOnFloor:
            animation_component.play_walk()
        PlatformerPhysicsComponent.State.moveInAir:
            if physics_component.body.velocity.y < 0:
                animation_component.play_jump()
            else:
                animation_component.play_fall()
```

## 设计模式

### 物理状态机
```gdscript
# 复杂的物理状态管理
class_name PhysicsStateMachine
extends Node

@export var physics_component: PlatformerPhysicsComponent
var special_states: Dictionary = {}

func add_special_state(name: String, config: Dictionary):
    special_states[name] = config

func enter_special_state(name: String):
    if name in special_states:
        var config = special_states[name]
        apply_physics_config(config)

func apply_physics_config(config: Dictionary):
    if "gravity_scale" in config:
        physics_component.gravityScaleOverride = config.gravity_scale
    if "disable_friction" in config and config.disable_friction:
        physics_component.shouldSkipFriction = true
```

### 物理修改器
```gdscript
# 临时物理效果系统
class_name PhysicsModifier
extends Resource

@export var duration: float
@export var gravity_scale_modifier: float = 1.0
@export var speed_modifier: float = 1.0
@export var friction_modifier: float = 1.0

func apply_to(physics_component: PlatformerPhysicsComponent):
    var original_values = store_original_values(physics_component)
    modify_physics(physics_component)
    
    # 设置定时器恢复原始值
    var timer = Timer.new()
    timer.wait_time = duration
    timer.one_shot = true
    timer.timeout.connect(func(): restore_original_values(physics_component, original_values))
    physics_component.add_child(timer)
    timer.start()
```

## 技术细节

### 处理顺序
1. 更新输入状态
2. 更新移动状态
3. 处理重力
4. 处理水平移动
5. 处理摩擦力
6. 执行move_and_slide()

### 性能优化
- 缓存频繁访问的属性
- 使用标志位控制逐帧处理
- 避免重复的函数调用

### 状态管理
- 基于输入和速度的智能状态切换
- 状态变化时的平滑过渡
- 与CharacterBody2D状态同步

## 注意事项

### 与其他组件的兼容性
- 不要与GravityComponent同时使用
- 确保在CharacterBodyComponent之后初始化
- 与JumpComponent协调工作

### 临时控制标志
- shouldSkipAcceleration和shouldSkipFriction只影响一帧
- 其他组件使用这些标志时要注意时机
- 标志会在使用后自动重置

### 参数调优
- 重力和摩擦力需要根据游戏感觉调整
- 空中控制程度影响游戏难度
- 加速度影响角色的"重量感"

## 相关组件
- [`CharacterBodyComponent`](../Physics/CharacterBodyComponent.md) - CharacterBody2D管理
- [`InputComponent`](../Control/InputComponent.md) - 输入控制
- [`JumpComponent`](../Control/JumpComponent.md) - 跳跃机制
- [`GravityComponent`](../Physics/GravityComponent.md) - 简单重力（不要同时使用）