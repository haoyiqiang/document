# PositionControlComponent API

> **继承关系**: Component > PositionControlComponent  
> **控制类型**: 直接位置控制

通过玩家输入直接设置实体位置的组件，不涉及任何物理计算。支持主/副输入轴，可用于控制摄像机、瞄准光标等需要精确控制的元素。

## ✨ 主要特性

- 🎮 直接位置控制，无物理计算
- 🕹️ 双轴输入支持（主轴/副轴）
- ⚡ 可调节移动速度
- 🎯 精确的位置控制
- 🔄 可启用/禁用功能

## 📊 导出属性

### 控制设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `speed` | `float` | `200.0` | 移动速度 (0-1000) |
| `shouldUseSecondaryAxis` | `bool` | `false` | 是否使用副轴输入（右摇杆） |
| `isEnabled` | `bool` | `true` | 是否启用位置控制 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `lastInput` | `Vector2` | 上一帧的输入向量 |

## 🎮 输入控制

### 主轴输入（默认）
- **左**: `moveLeft` / A / ←
- **右**: `moveRight` / D / →  
- **上**: `moveUp` / W / ↑
- **下**: `moveDown` / S / ↓

### 副轴输入（右摇杆模式）
- **左**: `lookLeft` / 右摇杆左
- **右**: `lookRight` / 右摇杆右
- **上**: `lookUp` / 右摇杆上  
- **下**: `lookDown` / 右摇杆下

## 🎯 使用示例

### 基础光标控制

```gdscript
# Entity Scene Structure:
# └── Cursor (Node2D)
#     ├── Sprite2D
#     └── PositionControlComponent
#         speed: 300.0
#         shouldUseSecondaryAxis: false
```

### 摄像机控制

```gdscript
# Entity Scene Structure:
# └── CameraController (Node2D)
#     ├── Camera2D
#     └── PositionControlComponent
#         speed: 150.0
#         shouldUseSecondaryAxis: true  # 使用右摇杆
```

### 双重控制系统

```gdscript
# DualControlComponent.gd
extends PositionControlComponent

@export var primarySpeed: float = 200.0
@export var secondarySpeed: float = 100.0
@export var enablePrecisionMode: bool = true

var isPrecisionMode: bool = false

func _process(delta: float):
    if not isEnabled:
        return
    
    # 检查精确模式
    updatePrecisionMode()
    
    # 获取输入
    var inputVector = getInputVector()
    
    # 应用速度修饰符
    var currentSpeed = getPrecisionAdjustedSpeed()
    
    # 更新位置
    parentEntity.position += inputVector * currentSpeed * delta

func _input(event: InputEvent):
    if enablePrecisionMode and event.is_action_pressed("precision_mode"):
        isPrecisionMode = not isPrecisionMode
        print("Precision mode: ", isPrecisionMode)

func updatePrecisionMode():
    if enablePrecisionMode:
        isPrecisionMode = Input.is_action_pressed("precision_mode")

func getInputVector() -> Vector2:
    if shouldUseSecondaryAxis:
        return Input.get_vector(
            GlobalInput.Actions.lookLeft,
            GlobalInput.Actions.lookRight,
            GlobalInput.Actions.lookUp,
            GlobalInput.Actions.lookDown)
    else:
        return Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)

func getPrecisionAdjustedSpeed() -> float:
    var baseSpeed = secondarySpeed if shouldUseSecondaryAxis else primarySpeed
    return baseSpeed * (0.3 if isPrecisionMode else 1.0)
```

### 边界限制控制

```gdscript
# BoundedPositionComponent.gd
extends PositionControlComponent

@export var boundaryRect: Rect2 = Rect2()
@export var enableBoundaries: bool = false
@export var bounceOnBoundary: bool = false
@export var wrapAroundBoundary: bool = false

func _process(delta: float):
    if not isEnabled:
        return
    
    # 获取输入
    var inputVector = getInputVector()
    
    # 计算新位置
    var newPosition = parentEntity.position + inputVector * speed * delta
    
    # 应用边界检查
    if enableBoundaries:
        newPosition = applyBoundaryConstraints(newPosition, inputVector)
    
    parentEntity.position = newPosition

func getInputVector() -> Vector2:
    if shouldUseSecondaryAxis:
        return Input.get_vector(
            GlobalInput.Actions.lookLeft,
            GlobalInput.Actions.lookRight,
            GlobalInput.Actions.lookUp,
            GlobalInput.Actions.lookDown)
    else:
        return Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)

func applyBoundaryConstraints(newPosition: Vector2, inputVector: Vector2) -> Vector2:
    if wrapAroundBoundary:
        return wrapPosition(newPosition)
    elif bounceOnBoundary:
        return bouncePosition(newPosition, inputVector)
    else:
        return clampPosition(newPosition)

func clampPosition(position: Vector2) -> Vector2:
    return Vector2(
        clamp(position.x, boundaryRect.position.x, boundaryRect.position.x + boundaryRect.size.x),
        clamp(position.y, boundaryRect.position.y, boundaryRect.position.y + boundaryRect.size.y)
    )

func wrapPosition(position: Vector2) -> Vector2:
    var wrappedX = position.x
    var wrappedY = position.y
    
    if wrappedX < boundaryRect.position.x:
        wrappedX = boundaryRect.position.x + boundaryRect.size.x
    elif wrappedX > boundaryRect.position.x + boundaryRect.size.x:
        wrappedX = boundaryRect.position.x
    
    if wrappedY < boundaryRect.position.y:
        wrappedY = boundaryRect.position.y + boundaryRect.size.y
    elif wrappedY > boundaryRect.position.y + boundaryRect.size.y:
        wrappedY = boundaryRect.position.y
    
    return Vector2(wrappedX, wrappedY)

func bouncePosition(position: Vector2, inputVector: Vector2) -> Vector2:
    # 简单的反弹逻辑
    var clampedPosition = clampPosition(position)
    
    # 如果位置被限制，反转输入方向
    if clampedPosition != position:
        # 这里可以添加反弹效果
        print("Boundary hit!")
    
    return clampedPosition

func setBoundaryRect(rect: Rect2):
    boundaryRect = rect
```

### 惯性控制组件

```gdscript
# InertiaPositionComponent.gd
extends PositionControlComponent

@export var inertiaEnabled: bool = false
@export var acceleration: float = 500.0
@export var friction: float = 200.0
@export var maxVelocity: float = 300.0

var velocity: Vector2 = Vector2.ZERO

func _process(delta: float):
    if not isEnabled:
        return
    
    var inputVector = getInputVector()
    
    if inertiaEnabled:
        processInertialMovement(inputVector, delta)
    else:
        processDirectMovement(inputVector, delta)

func getInputVector() -> Vector2:
    if shouldUseSecondaryAxis:
        return Input.get_vector(
            GlobalInput.Actions.lookLeft,
            GlobalInput.Actions.lookRight,
            GlobalInput.Actions.lookUp,
            GlobalInput.Actions.lookDown)
    else:
        return Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)

func processDirectMovement(inputVector: Vector2, delta: float):
    # 标准的直接位置控制
    parentEntity.position += inputVector * speed * delta
    lastInput = inputVector

func processInertialMovement(inputVector: Vector2, delta: float):
    # 带惯性的控制
    if inputVector.length() > 0:
        # 有输入时加速
        velocity += inputVector * acceleration * delta
    else:
        # 无输入时减速
        velocity = velocity.move_toward(Vector2.ZERO, friction * delta)
    
    # 限制最大速度
    if velocity.length() > maxVelocity:
        velocity = velocity.normalized() * maxVelocity
    
    # 应用移动
    parentEntity.position += velocity * delta
    lastInput = inputVector

func stopImmediately():
    velocity = Vector2.ZERO

func setInertiaEnabled(enabled: bool):
    inertiaEnabled = enabled
    if not enabled:
        stopImmediately()
```

### 网格对齐控制

```gdscript
# GridAlignedPositionComponent.gd
extends PositionControlComponent

@export var gridSize: Vector2 = Vector2(32, 32)
@export var snapToGrid: bool = true
@export var smoothSnapping: bool = false
@export var snapSpeed: float = 500.0

var targetGridPosition: Vector2
var isSnapping: bool = false

func _process(delta: float):
    if not isEnabled:
        return
    
    if isSnapping and smoothSnapping:
        processGridSnapping(delta)
    else:
        processNormalMovement(delta)

func processNormalMovement(delta: float):
    var inputVector = getInputVector()
    
    if snapToGrid:
        # 网格移动
        processGridMovement(inputVector)
    else:
        # 自由移动
        parentEntity.position += inputVector * speed * delta
    
    lastInput = inputVector

func processGridMovement(inputVector: Vector2):
    if inputVector.length() == 0 or isSnapping:
        return
    
    # 确定移动方向
    var direction = Vector2.ZERO
    if abs(inputVector.x) > abs(inputVector.y):
        direction.x = sign(inputVector.x)
    else:
        direction.y = sign(inputVector.y)
    
    # 计算目标网格位置
    targetGridPosition = snapPositionToGrid(parentEntity.position + direction * gridSize)
    
    if smoothSnapping:
        isSnapping = true
    else:
        parentEntity.position = targetGridPosition

func processGridSnapping(delta: float):
    parentEntity.position = parentEntity.position.move_toward(targetGridPosition, snapSpeed * delta)
    
    if parentEntity.position.distance_to(targetGridPosition) < 1.0:
        parentEntity.position = targetGridPosition
        isSnapping = false

func snapPositionToGrid(position: Vector2) -> Vector2:
    var snappedX = round(position.x / gridSize.x) * gridSize.x
    var snappedY = round(position.y / gridSize.y) * gridSize.y
    return Vector2(snappedX, snappedY)

func getInputVector() -> Vector2:
    if shouldUseSecondaryAxis:
        return Input.get_vector(
            GlobalInput.Actions.lookLeft,
            GlobalInput.Actions.lookRight,
            GlobalInput.Actions.lookUp,
            GlobalInput.Actions.lookDown)
    else:
        return Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)

func setGridSize(newSize: Vector2):
    gridSize = newSize

func alignToGrid():
    parentEntity.position = snapPositionToGrid(parentEntity.position)
```

## 🔧 技术细节

### 输入处理
```gdscript
func _process(delta: float) -> void:
    if not isEnabled: return
    
    if shouldUseSecondaryAxis:
        lastInput = Input.get_vector(
            GlobalInput.Actions.lookLeft,
            GlobalInput.Actions.lookRight,
            GlobalInput.Actions.lookUp,
            GlobalInput.Actions.lookDown)
    else:
        lastInput = Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)

    parentEntity.position += lastInput * speed * delta
```

### 性能考虑
- 使用`_process()`而非`_physics_process()`，适用于UI元素
- 通过`isEnabled`快速禁用处理
- `lastInput`可供子类访问和使用

## ⚠️ 注意事项

1. **无物理**: 不涉及任何物理计算，直接修改position
2. **坐标系统**: 修改的是局部position，不是global_position
3. **输入轴选择**: 主轴用于移动，副轴用于摄像机/瞄准
4. **处理模式**: 使用`_process()`而非`_physics_process()`
5. **子类扩展**: 子类可以访问`lastInput`进行额外处理

## 🔗 相关组件

- [MouseTrackingComponent](MouseTrackingComponent.md) - 鼠标跟踪组件
- [InputComponent](InputComponent.md) - 输入管理组件
- [TileBasedMouseControlComponent](TileBasedMouseControlComponent.md) - 瓦片鼠标控制
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 