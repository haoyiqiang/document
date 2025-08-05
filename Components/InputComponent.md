# InputComponent API 文档

## 概述
`InputComponent` 是统一的控制输入源，供其他组件使用。输入源可以是玩家、AI代理脚本或演示录制等。

相比直接检查 `Node._input`，共享输入状态的主要优势是其他组件/脚本可以修改共享状态，例如 `PlatformerPatrolComponent` 向 `PlatformerPhysicsComponent` "注入"移动。

## 继承
`InputComponent` → `Component` → `Node`

## 主要特性
- 统一的输入状态管理
- 支持玩家和AI控制
- 灵活的输入处理模式
- 方向变化检测
- 输入动作监控

## 要求
应放置在依赖玩家/AI控制的所有组件之后（在场景树下方），因为输入事件从场景树节点列表的底部向上传播。

## 导出属性

### 基础控制

#### `isEnabled: bool = true`
禁用所有输入处理和更新。

#### `isPlayerControlled: bool = true`
是否处理系统输入事件。
- false时不处理系统输入，但组件仍可被手动修改供AI代理或演示脚本使用
- 可随时手动调用 `handleInput()` 和 `updateInputActionsPressed()`

#### `shouldProcessUnhandledInputOnly: bool = true`
是否只处理未被其他组件消费的输入事件。
- true（默认）：处理 `_unhandled_input`
- false：处理 `_input`，捕获所有事件

#### `shouldIgnoreEchoes: bool = true`
是否忽略按键重复"回音"事件。

#### `shouldSetEventsAsHandled: bool = false`
是否阻止InputEvent冒泡到场景树。
- 可提高性能
- **警告**：会阻止其他InputComponent接收事件

### 输入缩放

#### `movementDirectionScale: Vector2 = Vector2.ONE`
主要移动控制的轴缩放，包括左摇杆和方向键。
- 负值反转控制，如(-1, 1)翻转水平行走方向

#### `lookDirectionScale: Vector2 = Vector2.ONE`
观察方向轴缩放，通常由右摇杆提供。
- 负值反转摄像机控制，如(1, -1)翻转垂直摄像机轴

## 状态属性

### 移动输入

#### `movementDirection: Vector2`
来自水平+垂直轴组合的主要移动输入。包括左摇杆和方向键。

#### `horizontalInput: float`
主要X轴输入。包括左摇杆和方向键。

#### `verticalInput: float`
主要Y轴输入。包括左摇杆和方向键。

#### `previousMovementDirection: Vector2`
之前的移动方向，不计算"回音"事件的变化。

### 观察输入

#### `lookDirection: Vector2`
右摇杆输入。

#### `turnInput: float`
仅左摇杆的水平X轴（不包括方向键）。

#### `thrustInput: float`
仅左摇杆的垂直Y轴（不包括方向键）。可能是 `verticalInput` 的反向。

### 历史状态

#### `lastNonzeroHorizontalInput: float`
最后接收到的非零水平输入。可用于确定角色面向等。

#### `lastNonzeroVerticalInput: float`
最后接收到的非零垂直输入。

#### `lastInputEvent: InputEvent`
最近接收到的InputEvent。只包括 `InputEvent.is_action_type()` 的事件。

### 控制标志

#### `inputActionsPressed: PackedStringArray`
当前按下的所有输入动作列表。

#### `shouldSkipNextEvent: bool`
下一个输入是否跳过一次。可用于临时抑制玩家控制。

## 信号

### `didUpdateInputActionsList`
输入动作列表更新时发出。在 `didProcessInput` 之前。

### `didProcessInput(event: InputEvent)`
InputEvent完全处理并更新所有状态属性后发出。在 `didUpdateInputActionsList` 之后。

### `didChangeHorizontalDirection`
X轴方向符号改变时发出，表示左右方向变化。可用于精灵翻转等动画。

### `didChangeVerticalDirection`
Y轴方向符号改变时发出，表示上下方向变化。

## 主要方法

### 输入处理

#### `handleInput(event: InputEvent) -> void`
处理输入事件的核心方法。
- 受 `isEnabled` 影响，但不受 `isPlayerControlled` 影响
- 不受 `shouldSkipNextEvent` 影响，允许手动处理合成事件
- 更新所有输入状态属性

#### `updateInputActionsPressed(event: InputEvent = null) -> bool`
更新当前按下的输入动作列表。

#### `setProcess() -> void`
根据标志启用或禁用每帧和事件处理。

## 使用示例

### 基本玩家控制
```gdscript
# 设置输入组件
var inputComponent = InputComponent.new()
playerEntity.add_child(inputComponent)

# 连接信号
inputComponent.didProcessInput.connect(_on_input_processed)
inputComponent.didChangeHorizontalDirection.connect(_on_direction_changed)

func _on_input_processed(event: InputEvent):
    # 使用处理后的输入状态
    if inputComponent.horizontalInput != 0:
        move_horizontally(inputComponent.horizontalInput)
    
    if "jump" in inputComponent.inputActionsPressed:
        jump()

func _on_direction_changed():
    # 翻转精灵
    sprite.flip_h = inputComponent.horizontalInput < 0
```

### AI控制设置
```gdscript
# 禁用玩家控制，启用AI控制
inputComponent.isPlayerControlled = false

# AI可以手动设置输入状态
inputComponent.movementDirection = Vector2(1, 0)  # 向右移动
inputComponent.horizontalInput = 1.0

# 或者发送合成输入事件
var syntheticEvent = InputEventAction.new()
syntheticEvent.action = "jump"
syntheticEvent.pressed = true
inputComponent.handleInput(syntheticEvent)
```

### 输入记录/回放
```gdscript
# 记录输入用于演示模式
var recordedInputs: Array[InputEvent] = []

func _on_input_processed(event: InputEvent):
    recordedInputs.append(event)

# 回放输入
func replay_inputs():
    inputComponent.isPlayerControlled = false
    for event in recordedInputs:
        inputComponent.handleInput(event)
        await get_tree().process_frame
```

### 临时禁用输入
```gdscript
# 暂停期间禁用玩家控制
func pause_game():
    inputComponent.shouldSkipNextEvent = true
    # 或者完全禁用
    inputComponent.isEnabled = false

# 对话期间禁用移动但保留其他输入
func start_dialogue():
    inputComponent.movementDirectionScale = Vector2.ZERO
```

### 输入修正器
```gdscript
# 反转水平控制
inputComponent.movementDirectionScale.x = -1

# 减慢垂直移动
inputComponent.movementDirectionScale.y = 0.5

# 反转摄像机垂直轴
inputComponent.lookDirectionScale.y = -1
```

### 高级输入处理
```gdscript
# 自定义输入缓冲
var inputBuffer: Array[String] = []
const BUFFER_SIZE = 10

func _on_input_processed(event: InputEvent):
    # 添加按下的动作到缓冲
    for action in inputComponent.inputActionsPressed:
        if not action in inputBuffer:
            inputBuffer.append(action)
            if inputBuffer.size() > BUFFER_SIZE:
                inputBuffer.pop_front()
    
    # 检查组合技
    if check_combo_input(inputBuffer):
        execute_special_move()
```

## 配置建议

### 单玩家游戏
```gdscript
# 防止其他输入组件接收事件
inputComponent.shouldSetEventsAsHandled = true
```

### 多玩家游戏
```gdscript
# 允许多个输入组件
inputComponent.shouldSetEventsAsHandled = false
```

### 菜单/UI场景
```gdscript
# 只处理未被UI消费的输入
inputComponent.shouldProcessUnhandledInputOnly = true
```

## 性能优化

1. **条件处理**：根据需要启用/禁用处理
2. **输入过滤**：使用 `shouldIgnoreEchoes` 避免重复处理
3. **事件处理**：设置 `shouldSetEventsAsHandled` 阻止不必要的传播
4. **AI优化**：AI控制时禁用 `isPlayerControlled`

## 注意事项

1. **场景树位置**：确保在依赖组件之后
2. **事件传播**：谨慎使用 `shouldSetEventsAsHandled`
3. **输入冲突**：多个InputComponent可能产生冲突
4. **状态持久性**：某些状态可能需要手动重置
5. **跨帧一致性**：确保输入状态在帧间保持一致

## 相关组件
- `PlatformerPhysicsComponent` - 平台物理组件
- `TurningControlComponent` - 转向控制组件
- `ThrustControlComponent` - 推进控制组件
- `PlatformerPatrolComponent` - 平台巡逻组件 