# TurningControlComponent API

> **继承关系**: Component > InputDependentComponentBase > TurningControlComponent  
> **控制类型**: 旋转控制

允许玩家使用左右输入动作旋转节点的控制组件，如武器或载具。可与ThrustControlComponent组合提供类似小行星的坦克式控制。

## ✨ 主要特性

- 🎮 左右输入旋转控制
- 🔫 支持指定节点旋转（如武器）
- 🕹️ 双摇杆支持（左摇杆/右摇杆）
- ⚡ 可调节旋转速度
- 🔄 可启用/禁用功能

## 📊 导出属性

### 旋转设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `rotationSpeed` | `float` | `5.0` | 旋转速度 (0.1-20.0) |
| `nodeToRotate` | `Node2D` | `null` | 要旋转的节点，默认为父实体 |
| `useLookDirectionInsteadOfTurnInput` | `bool` | `false` | 使用右摇杆朝向而非左摇杆转向 |
| `isEnabled` | `bool` | `true` | 是否启用旋转控制 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `rotationDirection` | `float` | 当前旋转方向 (-1.0 到 1.0) |

## 🎮 输入控制

### 旋转输入
- **向左转**: `turnLeft` / A / ←
- **向右转**: `turnRight` / D / →
- **或右摇杆**: 启用`useLookDirectionInsteadOfTurnInput`时使用

## 🎯 使用示例

### 基础坦克控制

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── InputComponent
#     ├── TurningControlComponent    # 转向
#     └── ThrustControlComponent     # 推进
```

### 武器旋转控制

```gdscript
# Entity Scene Structure:
# └── Player (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── InputComponent
#     ├── Gun (Node2D)
#     │   └── GunComponent
#     └── TurningControlComponent
#         nodeToRotate: Gun
#         useLookDirectionInsteadOfTurnInput: true
```

### 双摇杆控制

```gdscript
# DualStickController.gd
extends Node

@export var bodyTurning: TurningControlComponent
@export var weaponTurning: TurningControlComponent

func _ready():
    # 身体用左摇杆转向
    bodyTurning.useLookDirectionInsteadOfTurnInput = false
    
    # 武器用右摇杆瞄准
    weaponTurning.useLookDirectionInsteadOfTurnInput = true
    
    # 设置不同的旋转速度
    bodyTurning.rotationSpeed = 3.0
    weaponTurning.rotationSpeed = 8.0
```

### 条件旋转组件

```gdscript
# ConditionalTurningComponent.gd
extends TurningControlComponent

@export var requiresEnergy: bool = false
@export var energyCostPerSecond: float = 10.0
@export var minimumEnergyRequired: float = 5.0

var energyComponent: StatsComponent

func _ready():
    super._ready()
    energyComponent = parentEntity.findFirstComponentSubclass(StatsComponent)

func oninputComponent_didProcessInput(event: InputEvent) -> void:
    if requiresEnergy and not hasEnoughEnergy():
        rotationDirection = 0.0
        return
    
    super.oninputComponent_didProcessInput(event)

func _physics_process(delta: float):
    if requiresEnergy and not is_zero_approx(rotationDirection):
        consumeEnergy(delta)
    
    super._physics_process(delta)

func hasEnoughEnergy() -> bool:
    if not energyComponent:
        return true
    
    var energy = energyComponent.getStat("energy")
    return energy.currentValue >= minimumEnergyRequired

func consumeEnergy(delta: float):
    if energyComponent:
        var energy = energyComponent.getStat("energy")
        energy.currentValue -= energyCostPerSecond * delta
```

### 智能旋转组件

```gdscript
# SmartTurningComponent.gd
extends TurningControlComponent

@export var autoFaceTarget: bool = false
@export var targetTag: String = "enemy"
@export var autoFaceRange: float = 100.0
@export var autoFaceSpeed: float = 2.0

var currentTarget: Node2D

func _physics_process(delta: float):
    if autoFaceTarget and not inputComponent.hasInput():
        updateAutoFacing(delta)
    else:
        super._physics_process(delta)

func updateAutoFacing(delta: float):
    findNearestTarget()
    
    if currentTarget:
        var targetDirection = (currentTarget.global_position - nodeToRotate.global_position).normalized()
        var targetAngle = targetDirection.angle()
        var currentAngle = nodeToRotate.rotation
        
        var angleDifference = angle_difference(currentAngle, targetAngle)
        
        if abs(angleDifference) > 0.1:  # 避免抖动
            var rotationAmount = sign(angleDifference) * autoFaceSpeed * delta
            nodeToRotate.rotation += rotationAmount

func findNearestTarget():
    var targets = get_tree().get_nodes_in_group(targetTag)
    var nearestTarget: Node2D = null
    var nearestDistance: float = autoFaceRange
    
    for target in targets:
        if target is Node2D:
            var distance = nodeToRotate.global_position.distance_to(target.global_position)
            if distance < nearestDistance:
                nearestDistance = distance
                nearestTarget = target
    
    currentTarget = nearestTarget
```

### 限制旋转组件

```gdscript
# LimitedTurningComponent.gd
extends TurningControlComponent

@export var enableAngleLimits: bool = false
@export var minAngle: float = -90.0  # 度
@export var maxAngle: float = 90.0   # 度

func _physics_process(delta: float):
    if enableAngleLimits:
        var targetRotation = nodeToRotate.rotation + (rotationSpeed * rotationDirection) * delta
        var targetDegrees = rad_to_deg(targetRotation)
        
        # 限制角度范围
        targetDegrees = clamp(targetDegrees, minAngle, maxAngle)
        nodeToRotate.rotation = deg_to_rad(targetDegrees)
    else:
        super._physics_process(delta)
```

## 🔧 技术细节

### 旋转处理流程
```gdscript
func oninputComponent_didProcessInput(_event: InputEvent) -> void:
    rotationDirection = inputComponent.lookDirection.x if useLookDirectionInsteadOfTurnInput else inputComponent.turnInput

func _physics_process(delta: float) -> void:
    nodeToRotate.rotation += (rotationSpeed * rotationDirection) * delta
```

### 性能优化
- **条件处理**: 仅在有旋转输入时启用_physics_process
- **属性setter**: rotationDirection setter自动管理物理处理状态
- **节点引用**: 在_ready()中缓存nodeToRotate引用

### 双摇杆支持
- **左摇杆**: `inputComponent.turnInput` (默认)
- **右摇杆**: `inputComponent.lookDirection.x` (瞄准模式)

## ⚠️ 注意事项

1. **互斥组件**: 与MouseRotationComponent互斥，不要同时使用
2. **输入依赖**: 继承自InputDependentComponentBase，需要InputComponent
3. **节点顺序**: 必须在InputComponent之前，因为输入事件从下往上传播
4. **默认节点**: 如果未指定nodeToRotate，默认旋转父实体
5. **角度累积**: 旋转是累积的，不会自动重置角度

## 🔗 相关组件

- [ThrustControlComponent](ThrustControlComponent.md) - 推进控制组件
- [MouseRotationComponent](MouseRotationComponent.md) - 鼠标旋转组件（互斥）
- [AsteroidsControlComponent](AsteroidsControlComponent.md) - 小行星风格控制
- [InputDependentComponentBase](InputDependentComponentBase.md) - 基类组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 