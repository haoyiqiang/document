# OverheadPhysicsComponent API

## 概述
`OverheadPhysicsComponent` 处理俯视角（即"自上而下"）移动的摩擦力和其他物理效果。专为俯视角游戏设计，为父实体的`CharacterBodyComponent`提供物理处理。

**继承链**：`Component` → `CharacterBodyDependentComponentBase` → `OverheadPhysicsComponent`

**注意**：不处理玩家输入，输入由`InputComponent`和/或AI代理提供。

## 依赖要求
- **CharacterBodyComponent**：必需的CharacterBody2D管理组件
- **InputComponent**（可选）：提供移动方向输入

## 组件顺序要求
- **之前**：CharacterBodyComponent、InputComponent

## 主要特性
- 🎮 **俯视角移动**：专为自上而下游戏优化的物理处理
- 🏃 **加速度控制**：可选的渐进加速和即时移动
- 🛑 **摩擦系统**：分轴摩擦力处理，提供真实的停止感
- 📏 **最小速度**：维持最小移动速度防止抖动
- 🔄 **速度保持**：可选择保持前一帧的速度

## 导出属性

### 基础配置
- **parameters** (`OverheadMovementParameters`)：移动参数资源
- **isEnabled** (`bool = true`)：是否启用物理处理

## 状态属性
- **movementDirection** (`Vector2`)：当前移动方向（从InputComponent获取）

## OverheadMovementParameters 属性

### 速度配置
- **speed** (`float`)：最大移动速度
- **minimumSpeed** (`float`)：最小维持速度

### 加速度配置
- **shouldApplyAcceleration** (`bool`)：是否使用渐进加速
- **acceleration** (`float`)：加速度值

### 摩擦力配置
- **shouldApplyFriction** (`bool`)：是否应用摩擦力
- **friction** (`float`)：摩擦力强度

### 特殊行为
- **shouldMaintainPreviousVelocity** (`bool`)：无输入时是否保持前一帧速度
- **shouldMaintainMinimumVelocity** (`bool`)：是否维持最小速度

## 主要方法
- **processMovement**(delta: float) → `void`：处理移动和摩擦力

## 使用示例

### 基础俯视角移动
```gdscript
# 设置基本的俯视角物理
@export var overhead_physics: OverheadPhysicsComponent
@export var movement_params: OverheadMovementParameters

func _ready():
    overhead_physics.parameters = movement_params
    setup_movement_parameters()

func setup_movement_parameters():
    var params = overhead_physics.parameters
    params.speed = 200.0
    params.shouldApplyAcceleration = true
    params.acceleration = 800.0
    params.shouldApplyFriction = true
    params.friction = 600.0
```

### 即时响应移动
```gdscript
# 设置即时响应的移动（如街机游戏）
@export var overhead_physics: OverheadPhysicsComponent

func setup_arcade_movement():
    var params = overhead_physics.parameters
    params.speed = 150.0
    params.shouldApplyAcceleration = false  # 即时达到最大速度
    params.shouldApplyFriction = false      # 即时停止
```

### 滑冰效果
```gdscript
# 创建滑冰般的滑行效果
@export var overhead_physics: OverheadPhysicsComponent

func setup_ice_movement():
    var params = overhead_physics.parameters
    params.speed = 180.0
    params.shouldApplyAcceleration = true
    params.acceleration = 400.0    # 较慢的加速
    params.shouldApplyFriction = true
    params.friction = 100.0        # 很低的摩擦力
```

### 载具物理
```gdscript
# 载具式的移动感觉
@export var overhead_physics: OverheadPhysicsComponent

func setup_vehicle_movement():
    var params = overhead_physics.parameters
    params.speed = 300.0
    params.shouldApplyAcceleration = true
    params.acceleration = 500.0
    params.shouldApplyFriction = true
    params.friction = 800.0
    params.shouldMaintainMinimumVelocity = true
    params.minimumSpeed = 20.0     # 防止完全停止的抖动
```

### 动态物理调整
```gdscript
# 根据地形调整物理参数
@export var overhead_physics: OverheadPhysicsComponent
var base_params: OverheadMovementParameters

func _ready():
    base_params = overhead_physics.parameters.duplicate()

func enter_swamp():
    # 沼泽：降低速度，增加摩擦
    var params = overhead_physics.parameters
    params.speed = base_params.speed * 0.5
    params.friction = base_params.friction * 2.0

func enter_ice():
    # 冰面：正常速度，极低摩擦
    var params = overhead_physics.parameters
    params.speed = base_params.speed
    params.friction = base_params.friction * 0.1

func exit_special_terrain():
    # 恢复正常参数
    overhead_physics.parameters = base_params.duplicate()
```

### AI移动集成
```gdscript
# 与AI系统集成
@export var overhead_physics: OverheadPhysicsComponent
@export var ai_component: AIComponent

func _ready():
    # 连接AI输入到物理组件
    if ai_component:
        ai_component.movement_direction_changed.connect(_on_ai_direction_changed)

func _on_ai_direction_changed(direction: Vector2):
    # AI直接设置移动方向
    overhead_physics.movementDirection = direction
```

## 设计模式

### 物理状态切换
```gdscript
# 不同状态的物理模式
class_name PhysicsStateManager
extends Node

@export var overhead_physics: OverheadPhysicsComponent
var physics_presets: Dictionary = {}

func _ready():
    setup_presets()

func setup_presets():
    # 正常状态
    physics_presets["normal"] = create_params(200.0, 800.0, 600.0, true, true)
    # 潜行状态
    physics_presets["stealth"] = create_params(80.0, 400.0, 800.0, true, true)
    # 冲刺状态
    physics_presets["dash"] = create_params(400.0, 1200.0, 200.0, true, true)

func set_physics_state(state: String):
    if state in physics_presets:
        overhead_physics.parameters = physics_presets[state]
```

### 环境物理系统
```gdscript
# 基于环境的物理调整
class_name EnvironmentalPhysics
extends Area2D

@export var speed_multiplier: float = 1.0
@export var friction_multiplier: float = 1.0
@export var acceleration_multiplier: float = 1.0

func _on_body_entered(body: Node2D):
    var entity = body.get_parent() as Entity
    if entity:
        var physics = entity.findFirstComponentSubclass(OverheadPhysicsComponent)
        if physics:
            apply_environmental_effects(physics)

func apply_environmental_effects(physics: OverheadPhysicsComponent):
    var params = physics.parameters
    params.speed *= speed_multiplier
    params.friction *= friction_multiplier
    params.acceleration *= acceleration_multiplier
```

## 技术细节

### CharacterBody2D配置
- 自动设置motion_mode为MOTION_MODE_FLOATING
- 建议启用shouldResetVelocityIfZeroMotion
- 每帧设置shouldMoveThisFrame = true

### 物理处理顺序
1. 获取输入方向
2. 应用加速度或直接设置速度
3. 分轴处理摩擦力
4. 处理特殊速度维持逻辑
5. 清空移动方向

### 性能优化
- 输入方向每帧重置防止累积
- 条件检查优化减少不必要计算
- 使用move_toward()进行平滑过渡

## 注意事项

### 与其他组件协调
- 确保在CharacterBodyComponent之前初始化
- 与InputComponent协调获取输入
- 避免与其他物理组件冲突

### 参数调优建议
- acceleration应大于friction以获得响应感
- minimumSpeed防止小幅抖动
- speed影响整体游戏节奏

### 俯视角特殊考虑
- 没有重力概念，完全基于输入驱动
- 摩擦力分X/Y轴处理更真实
- 适合RPG、策略、射击等俯视角游戏

## 相关组件
- [`CharacterBodyComponent`](../Physics/CharacterBodyComponent.md) - CharacterBody2D管理
- [`InputComponent`](../Control/InputComponent.md) - 输入控制
- [`PlatformerPhysicsComponent`](../Physics/PlatformerPhysicsComponent.md) - 平台物理（侧视角） 