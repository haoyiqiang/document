# PlatformerControlComponent API

## 概述
`PlatformerControlComponent` 为`PlatformerPhysicsComponent`提供玩家控制输入。专注于水平移动控制，跳跃功能由`JumpControlComponent`单独处理。

**继承链**：`Component` → `PlatformerControlComponent`

**注意**：跳跃由`JumpControlComponent`提供，此组件仅处理水平移动。

## 依赖要求
- **PlatformerPhysicsComponent**：必需的平台物理组件

## 组件顺序要求
- **之前**：PlatformerPhysicsComponent、CharacterBodyComponent

## 主要特性
- ⬅️➡️ **水平控制**：处理左右移动输入
- 🔄 **方向反转**：支持控制轴反转（如迷惑效果）
- 🤖 **AI覆盖**：支持AI代理覆盖玩家输入
- 📊 **状态跟踪**：跟踪输入方向和变化
- ⚡ **高性能**：优化的输入处理和状态管理

## 导出属性

### 控制配置
- **shouldInvertXAxis** (`bool = false`)：是否反转X轴控制
  - 用于镜像游戏或"迷惑"效果等情况
- **isEnabled** (`bool = true`)：是否启用输入控制

## 状态属性
- **inputDirectionOverride** (`float = 0`)：输入方向覆盖值
  - 允许AI代理控制，每帧重置为0
- **inputDirection** (`float`)：当前输入方向
- **lastInputDirection** (`float`)：最后一次输入方向
- **isInputZero** (`bool = true`)：输入是否为零

## 主要方法

### 输入处理
- **processInput**() → `void`：处理玩家输入，受isEnabled影响
- **copyInputToPhysicsComponent**() → `void`：将输入状态复制到物理组件

## 使用示例

### 基础平台控制
```gdscript
# 设置基本的平台游戏控制
@export var platformer_control: PlatformerControlComponent
@export var physics_component: PlatformerPhysicsComponent

func _ready():
    platformer_control.isEnabled = true
    platformer_control.shouldInvertXAxis = false
```

### 迷惑状态效果
```gdscript
# 实现控制反转的状态效果
@export var platformer_control: PlatformerControlComponent

func apply_confusion_effect(duration: float):
    platformer_control.shouldInvertXAxis = true
    
    # 设置定时器恢复正常
    var timer = Timer.new()
    timer.wait_time = duration
    timer.one_shot = true
    timer.timeout.connect(func():
        platformer_control.shouldInvertXAxis = false
        timer.queue_free()
    )
    add_child(timer)
    timer.start()
```

### AI控制集成
```gdscript
# AI代理控制玩家移动
@export var platformer_control: PlatformerControlComponent
@export var ai_agent: AIAgent

func _physics_process(_delta):
    if ai_agent and ai_agent.is_controlling:
        # AI覆盖玩家输入
        var ai_direction = ai_agent.get_desired_direction()
        platformer_control.inputDirectionOverride = ai_direction
```

### 控制状态监控
```gdscript
# 监控和响应控制状态变化
@export var platformer_control: PlatformerControlComponent
@export var animation_player: AnimationPlayer

func _physics_process(_delta):
    # 根据输入状态播放动画
    if platformer_control.isInputZero:
        animation_player.play("idle")
    elif platformer_control.inputDirection > 0:
        animation_player.play("walk_right")
    elif platformer_control.inputDirection < 0:
        animation_player.play("walk_left")
```

### 输入缓冲系统
```gdscript
# 实现输入缓冲提升手感
@export var platformer_control: PlatformerControlComponent
var input_buffer: Array[float] = []
var buffer_size: int = 5

func _physics_process(_delta):
    # 记录输入历史
    input_buffer.append(platformer_control.inputDirection)
    if input_buffer.size() > buffer_size:
        input_buffer.pop_front()
    
    # 可用于实现延迟容错等功能
    analyze_input_patterns()

func analyze_input_patterns():
    # 分析输入模式，如快速双击检测等
    pass
```

### 动态控制权限
```gdscript
# 基于游戏状态的控制权限管理
@export var platformer_control: PlatformerControlComponent

func update_control_permissions():
    # 对话中禁用控制
    if GameState.is_in_dialogue:
        platformer_control.isEnabled = false
        return
    
    # 菜单中禁用控制
    if GameState.is_menu_open:
        platformer_control.isEnabled = false
        return
    
    # 正常游戏状态启用控制
    platformer_control.isEnabled = true
```

## 设计模式

### 控制状态机
```gdscript
# 复杂的控制状态管理
class_name ControlStateMachine
extends Node

@export var platformer_control: PlatformerControlComponent
var current_state: ControlState = ControlState.NORMAL

enum ControlState {
    NORMAL,
    INVERTED,
    DISABLED,
    AI_CONTROLLED
}

func set_control_state(new_state: ControlState):
    match new_state:
        ControlState.NORMAL:
            platformer_control.isEnabled = true
            platformer_control.shouldInvertXAxis = false
        ControlState.INVERTED:
            platformer_control.isEnabled = true
            platformer_control.shouldInvertXAxis = true
        ControlState.DISABLED:
            platformer_control.isEnabled = false
        ControlState.AI_CONTROLLED:
            # 保持启用但AI会覆盖输入
            platformer_control.isEnabled = true
    
    current_state = new_state
```

### 输入装饰器
```gdscript
# 输入修饰和增强系统
class_name InputDecorator
extends Node

@export var platformer_control: PlatformerControlComponent
var modifiers: Array[InputModifier] = []

func add_modifier(modifier: InputModifier):
    modifiers.append(modifier)

func _physics_process(_delta):
    # 应用所有修饰器
    var final_input = platformer_control.inputDirection
    for modifier in modifiers:
        final_input = modifier.modify_input(final_input)
    
    # 覆盖最终输入
    platformer_control.inputDirectionOverride = final_input
```

## 技术细节

### 输入处理流程
1. 获取Input.get_axis()的原始输入
2. 应用shouldInvertXAxis反转
3. 处理inputDirectionOverride覆盖
4. 更新状态标志（isInputZero等）
5. 复制状态到PlatformerPhysicsComponent

### 性能优化
- 缓存频繁访问的属性避免重复计算
- 条件检查优化减少不必要的处理
- 每帧重置覆盖值防止状态累积

### 输入响应性
- 在_input()和_physics_process()中都处理
- 确保输入的即时响应性
- 与物理更新同步

## 注意事项

### 组件协调
- 与PlatformerPhysicsComponent紧密配合
- 不处理跳跃，避免与JumpControlComponent冲突
- 确保正确的初始化顺序

### 输入覆盖机制
- inputDirectionOverride每帧自动重置
- AI或其他系统需要每帧设置覆盖值
- 覆盖值优先于玩家输入

### 轴反转注意事项
- 仅影响水平移动，不影响其他输入
- 可用于特殊游戏机制或状态效果
- 注意与玩家预期的一致性

## 相关组件
- [`PlatformerPhysicsComponent`](./PlatformerPhysicsComponent.md) - 平台物理
- [`JumpControlComponent`](./JumpComponent.md) - 跳跃控制
- [`InputComponent`](./InputComponent.md) - 通用输入
- [`CharacterBodyComponent`](./CharacterBodyComponent.md) - CharacterBody2D管理 