# OverheadControlComponent API

> **继承关系**: Component > OverheadControlComponent  
> **游戏类型**: 俯视角游戏

为OverheadPhysicsComponent提供玩家控制输入的组件，处理自上而下游戏的移动控制，支持WASD或方向键输入。

## ✨ 主要特性

- 🎮 标准俯视角控制输入
- 🔧 可调节的输入修饰符
- ⚡ 实时物理处理
- 🎯 紧密集成OverheadPhysicsComponent
- 🔄 可启用/禁用控制

## 📊 导出属性

### 控制设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `modifierScale` | `Vector2` | `Vector2(1, 1)` | 输入修饰符，(1,1)为正常控制，负值反转轴向 |
| `isEnabled` | `bool` | `true` | 是否启用控制 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `OverheadPhysicsComponent` | **之后** | 提供物理移动处理 |
| `CharacterBodyComponent` | **之后** | 提供CharacterBody2D管理 |

### 组件属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `overheadPhysicsComponent` | `OverheadPhysicsComponent` | 对俯视角物理组件的引用 |

## 🎮 输入映射

### 支持的输入动作
| 动作 | 输入键 | 方向 |
|------|--------|------|
| `moveLeft` | A / ← | 向左移动 |
| `moveRight` | D / → | 向右移动 |
| `moveUp` | W / ↑ | 向上移动 |
| `moveDown` | S / ↓ | 向下移动 |

## 🎯 使用示例

### 基础俯视角控制

```gdscript
# 在实体中添加组件
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── OverheadPhysicsComponent
#     ├── CharacterBodyComponent  
#     └── OverheadControlComponent  # 必须在physics组件之前
```

### 反转Y轴控制

```gdscript
# 设置修饰符实现反向控制
@export var modifierScale: Vector2 = Vector2(1, -1)  # Y轴反转
```

### 动态控制设置

```gdscript
# CustomOverheadController.gd
extends OverheadControlComponent

@export var normalSpeed: Vector2 = Vector2(1, 1)
@export var fastSpeed: Vector2 = Vector2(2, 2)
@export var slowSpeed: Vector2 = Vector2(0.5, 0.5)

func _ready():
    super._ready()
    setupSpeedModes()

func setupSpeedModes():
    # 设置速度模式切换
    pass

func _physics_process(delta: float):
    if not isEnabled: 
        return
    
    # 检查特殊输入修饰符
    var currentModifier = modifierScale
    
    if Input.is_action_pressed("sprint"):
        currentModifier = fastSpeed
    elif Input.is_action_pressed("sneak"):
        currentModifier = slowSpeed
    
    # 应用修饰后的输入
    overheadPhysicsComponent.inputDirection = Input.get_vector(
        GlobalInput.Actions.moveLeft,
        GlobalInput.Actions.moveRight,
        GlobalInput.Actions.moveUp,
        GlobalInput.Actions.moveDown) * currentModifier
```

### 平滑输入过渡

```gdscript
# SmoothOverheadController.gd
extends OverheadControlComponent

@export var inputSmoothness: float = 10.0
@export var deadzone: float = 0.1

var smoothedInput: Vector2 = Vector2.ZERO

func _physics_process(delta: float):
    if not isEnabled:
        overheadPhysicsComponent.inputDirection = Vector2.ZERO
        return
    
    # 获取原始输入
    var rawInput = Input.get_vector(
        GlobalInput.Actions.moveLeft,
        GlobalInput.Actions.moveRight,
        GlobalInput.Actions.moveUp,
        GlobalInput.Actions.moveDown)
    
    # 应用死区
    if rawInput.length() < deadzone:
        rawInput = Vector2.ZERO
    
    # 平滑过渡
    smoothedInput = smoothedInput.lerp(rawInput * modifierScale, inputSmoothness * delta)
    
    # 设置最终输入
    overheadPhysicsComponent.inputDirection = smoothedInput
```

### 触摸设备支持

```gdscript
# TouchOverheadController.gd
extends OverheadControlComponent

@export var enableTouchControl: bool = true
@export var touchSensitivity: float = 1.0

var touchStartPosition: Vector2
var currentTouchPosition: Vector2
var isTouching: bool = false

func _ready():
    super._ready()
    if enableTouchControl:
        setupTouchInput()

func setupTouchInput():
    # 连接触摸输入信号
    pass

func _input(event: InputEvent):
    if not enableTouchControl or not isEnabled:
        return
    
    if event is InputEventScreenTouch:
        handleTouchInput(event as InputEventScreenTouch)
    elif event is InputEventScreenDrag:
        handleDragInput(event as InputEventScreenDrag)

func handleTouchInput(event: InputEventScreenTouch):
    if event.pressed:
        touchStartPosition = event.position
        currentTouchPosition = event.position
        isTouching = true
    else:
        isTouching = false

func handleDragInput(event: InputEventScreenDrag):
    if isTouching:
        currentTouchPosition = event.position

func _physics_process(delta: float):
    if not isEnabled:
        return
    
    var inputDirection: Vector2
    
    if enableTouchControl and isTouching:
        # 使用触摸输入
        var touchDelta = currentTouchPosition - touchStartPosition
        inputDirection = touchDelta.normalized() * touchSensitivity
    else:
        # 使用标准键盘输入
        inputDirection = Input.get_vector(
            GlobalInput.Actions.moveLeft,
            GlobalInput.Actions.moveRight,
            GlobalInput.Actions.moveUp,
            GlobalInput.Actions.moveDown)
    
    overheadPhysicsComponent.inputDirection = inputDirection * modifierScale
```

### AI控制集成

```gdscript
# AIOverheadController.gd
extends OverheadControlComponent

@export var aiControlled: bool = false
@export var targetNode: Node2D

var aiInputDirection: Vector2 = Vector2.ZERO

func _ready():
    super._ready()
    if aiControlled:
        setupAIControl()

func setupAIControl():
    # 禁用玩家输入，启用AI控制
    isEnabled = false
    set_physics_process(true)

func _physics_process(delta: float):
    if aiControlled:
        updateAIInput()
        overheadPhysicsComponent.inputDirection = aiInputDirection
    elif isEnabled:
        # 调用父类的标准输入处理
        super._physics_process(delta)

func updateAIInput():
    if targetNode:
        var direction = (targetNode.global_position - global_position).normalized()
        aiInputDirection = direction * modifierScale
    else:
        aiInputDirection = Vector2.ZERO

func setAITarget(target: Node2D):
    targetNode = target

func enablePlayerControl():
    aiControlled = false
    isEnabled = true

func enableAIControl(target: Node2D = null):
    aiControlled = true
    isEnabled = false
    if target:
        targetNode = target
```

## 🏛️ 设计模式

### 控制器模式
- **输入层**: OverheadControlComponent处理输入
- **逻辑层**: OverheadPhysicsComponent处理物理
- **数据层**: CharacterBodyComponent管理实体

### 组合模式
- **核心**: 通过组件组合实现完整功能
- **解耦**: 控制逻辑与物理逻辑分离
- **复用**: 可独立替换控制方式

## 🔧 技术细节

### 输入处理流程
```gdscript
func _physics_process(_delta: float) -> void:
    if not isEnabled: return

    overheadPhysicsComponent.inputDirection = Input.get_vector(
        GlobalInput.Actions.moveLeft,
        GlobalInput.Actions.moveRight,  
        GlobalInput.Actions.moveUp,
        GlobalInput.Actions.moveDown) * modifierScale
```

### 修饰符应用
- **正常**: `Vector2(1, 1)` - 标准控制
- **反转X**: `Vector2(-1, 1)` - X轴反向
- **反转Y**: `Vector2(1, -1)` - Y轴反向  
- **加速**: `Vector2(2, 2)` - 双倍速度
- **减速**: `Vector2(0.5, 0.5)` - 半速

## ⚠️ 注意事项

1. **处理顺序**: 必须在OverheadPhysicsComponent之前处理
2. **输入延迟**: 使用_physics_process确保稳定的输入响应
3. **性能优化**: 输入检查每帧执行，避免复杂计算
4. **坐标系**: 适用于2D俯视角游戏，Y轴向下为正
5. **输入映射**: 确保GlobalInput.Actions中定义了相应的输入动作

## 🔗 相关组件

- [OverheadPhysicsComponent](../Physics/OverheadPhysicsComponent.md) - 俯视角物理组件
- [CharacterBodyComponent](../Physics/CharacterBodyComponent.md) - 角色体组件
- [InputComponent](InputComponent.md) - 输入管理组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 