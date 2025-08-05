# InputDependentComponentBase API

> **继承关系**: Component > InputDependentComponentBase  
> **抽象类**: 必须由子类实现

依赖输入的组件的抽象基类，为需要玩家输入或AI控制的组件提供统一的InputComponent依赖管理和事件处理框架。

## ✨ 主要特性

- 🎯 统一的InputComponent依赖管理
- 🔄 自动连接输入事件回调
- 📋 抽象方法模板，强制子类实现
- 🏗️ 简化输入相关组件的开发
- ⚡ 优化的事件传播顺序

## 🎯 设计模式

### 模板方法模式
- **基类**: 提供InputComponent依赖管理框架
- **子类**: 实现具体的输入处理逻辑
- **抽象方法**: 强制子类实现必要的事件处理

### 依赖注入模式
- **自动发现**: 在父实体中查找InputComponent
- **事件连接**: 自动连接到InputComponent的信号
- **错误处理**: 缺失依赖时输出警告

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `InputComponent` | **之前** | 输入组件必须在依赖组件之前处理 |

### 属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `inputComponent` | `InputComponent` | 对父实体InputComponent的引用 |

## 🎯 抽象方法

### 必须实现
```gdscript
abstract func oninputComponent_didProcessInput(event: InputEvent) -> void
```
**描述**: 处理输入事件的抽象方法  
**参数**: `event` - 输入事件对象  
**要求**: 所有子类必须实现此方法

### 可选实现
```gdscript
func oninputComponent_didUpdateInputActionsList() -> void
```
**描述**: 当输入动作列表更新时调用的可选方法  
**默认**: 空实现，子类可选择性重写

## 🎯 使用示例

### 创建基础输入组件

```gdscript
# CustomInputComponent.gd
class_name CustomInputComponent
extends InputDependentComponentBase

@export var sensitivity: float = 1.0

# 必须实现的抽象方法
func oninputComponent_didProcessInput(event: InputEvent) -> void:
    if event.is_action_pressed("jump"):
        handleJumpInput()
    elif event.is_action_pressed("shoot"):
        handleShootInput()

# 可选的输入列表更新处理
func oninputComponent_didUpdateInputActionsList() -> void:
    print("Input actions list updated!")
    updateInputHints()

func handleJumpInput():
    print("Jump input detected!")
    # 执行跳跃逻辑

func handleShootInput():
    print("Shoot input detected!")
    # 执行射击逻辑

func updateInputHints():
    # 更新UI中的输入提示
    pass
```

### 高级输入处理组件

```gdscript
# AdvancedInputHandler.gd
class_name AdvancedInputHandler
extends InputDependentComponentBase

@export var inputDelay: float = 0.1
@export var enableComboInputs: bool = true

var lastInputTime: float = 0.0
var inputQueue: Array[String] = []

func oninputComponent_didProcessInput(event: InputEvent) -> void:
    var currentTime = Time.get_time_dict_from_system()
    
    # 检查输入延迟
    if currentTime - lastInputTime < inputDelay:
        return
    
    lastInputTime = currentTime
    
    # 处理各种输入类型
    if event is InputEventKey:
        handleKeyInput(event as InputEventKey)
    elif event is InputEventMouseButton:
        handleMouseInput(event as InputEventMouseButton)
    elif event is InputEventJoypadButton:
        handleControllerInput(event as InputEventJoypadButton)

func handleKeyInput(event: InputEventKey):
    if enableComboInputs:
        inputQueue.append(event.get_action_name())
        checkForCombos()
    
    # 单独按键处理
    match event.get_action_name():
        "move_left":
            processMovement(Vector2.LEFT)
        "move_right":
            processMovement(Vector2.RIGHT)
        "action":
            executeAction()

func handleMouseInput(event: InputEventMouseButton):
    if event.pressed:
        var targetPosition = get_global_mouse_position()
        processTargetedAction(targetPosition)

func handleControllerInput(event: InputEventJoypadButton):
    # 手柄输入特殊处理
    print("Controller button pressed: ", event.button_index)

func checkForCombos():
    # 检查连招输入
    if inputQueue.size() >= 3:
        var combo = str(inputQueue[-3], inputQueue[-2], inputQueue[-1])
        if isValidCombo(combo):
            executeCombo(combo)
        
        # 保持队列长度
        inputQueue = inputQueue.slice(-5)

func isValidCombo(combo: String) -> bool:
    # 定义有效的连招
    var validCombos = ["move_leftmove_rightaction", "actionactionaction"]
    return combo in validCombos

func executeCombo(combo: String):
    print("Combo executed: ", combo)
    # 执行连招逻辑

func processMovement(direction: Vector2):
    # 处理移动输入
    pass

func executeAction():
    # 执行动作
    pass

func processTargetedAction(target: Vector2):
    # 处理目标动作
    pass
```

### 输入缓冲组件

```gdscript
# InputBufferComponent.gd
class_name InputBufferComponent
extends InputDependentComponentBase

@export var bufferWindow: float = 0.2  # 缓冲窗口时间
@export var maxBufferSize: int = 5

var inputBuffer: Array[Dictionary] = []

func oninputComponent_didProcessInput(event: InputEvent) -> void:
    # 将输入添加到缓冲区
    var inputData = {
        "event": event,
        "timestamp": Time.get_time_dict_from_system(),
        "action": event.get_action_name() if event.is_action_type() else ""
    }
    
    inputBuffer.append(inputData)
    
    # 清理过期的输入
    cleanExpiredInputs()
    
    # 限制缓冲区大小
    if inputBuffer.size() > maxBufferSize:
        inputBuffer = inputBuffer.slice(-maxBufferSize)
    
    # 处理缓冲的输入
    processBufferedInputs()

func cleanExpiredInputs():
    var currentTime = Time.get_time_dict_from_system()
    var validInputs: Array[Dictionary] = []
    
    for input in inputBuffer:
        if currentTime - input.timestamp <= bufferWindow:
            validInputs.append(input)
    
    inputBuffer = validInputs

func processBufferedInputs():
    # 查找特定的输入模式
    if hasInputPattern(["jump", "shoot"]):
        executeJumpShoot()
    elif hasInputPattern(["move_left", "move_left"]):
        executeDash(Vector2.LEFT)

func hasInputPattern(pattern: Array[String]) -> bool:
    if inputBuffer.size() < pattern.size():
        return false
    
    var recentInputs = inputBuffer.slice(-pattern.size())
    for i in range(pattern.size()):
        if recentInputs[i].action != pattern[i]:
            return false
    
    return true

func executeJumpShoot():
    print("Jump-shoot combo executed!")

func executeDash(direction: Vector2):
    print("Dash executed in direction: ", direction)

func getRecentInputs(count: int = 3) -> Array[Dictionary]:
    return inputBuffer.slice(-count) if inputBuffer.size() >= count else inputBuffer

func clearBuffer():
    inputBuffer.clear()
```

## 🏛️ 架构设计

### 事件传播顺序
```
Scene Tree Node Order (Bottom to Top):
1. InputComponent/Subclasses
2. InputDependentComponentBase Subclasses
3. Other Components
```

### 依赖解析流程
```gdscript
func _enter_tree() -> void:
    # 1. 调用父类初始化
    super._enter_tree()
    
    # 2. 查找InputComponent
    inputComponent = parentEntity.findFirstComponentSubclass(InputComponent)
    
    # 3. 连接信号
    if inputComponent:
        Tools.connectSignal(inputComponent.didProcessInput, self.oninputComponent_didProcessInput)
        Tools.connectSignal(inputComponent.didUpdateInputActionsList, self.oninputComponent_didUpdateInputActionsList)
    
    # 4. 错误处理
    else:
        printWarning("Missing InputComponent in " + str(parentEntity))
```

## ⚠️ 注意事项

1. **抽象类**: 不能直接实例化，必须通过子类使用
2. **节点顺序**: 必须放在InputComponent或其子类之前
3. **事件传播**: 输入事件从场景树底部向上传播
4. **依赖检查**: 确保父实体包含InputComponent
5. **信号连接**: 自动处理信号连接，子类无需手动连接
6. **性能考虑**: 避免在输入处理中执行耗时操作

## 🔗 相关组件

- [InputComponent](InputComponent.md) - 输入管理组件
- [ActionControlComponent](ActionControlComponent.md) - 行动控制组件  
- [PlatformerControlComponent](PlatformerControlComponent.md) - 平台控制组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21