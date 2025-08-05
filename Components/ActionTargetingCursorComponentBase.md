# ActionTargetingCursorComponentBase API 文档

## 概述
`ActionTargetingCursorComponentBase` 是一个抽象基类，用于显示光标和UI供玩家或其他实体选择行动目标。该组件扩展了基本的目标选择功能，添加了可视化光标和碰撞检测机制。

## 继承关系
```
Component
└── ActionTargetingComponentBase
    └── ActionTargetingCursorComponentBase
        └── ActionTargetingMouseComponent
```

## 主要功能
- **可视化光标**: 显示目标选择光标和行动名称
- **碰撞检测**: 检测光标与可目标化组件的碰撞
- **目标高亮**: 自动高亮碰撞范围内的可选择目标
- **UI集成**: 显示行动名称和相关信息

## 节点结构

| 节点名 | 类型 | 描述 |
|--------|------|------|
| `CursorSprite` | `Sprite2D` | 光标精灵显示 |
| `CursorArea` | `Area2D` | 光标碰撞区域 |
| `Label` | `Label` | 行动名称显示 |

## 公共属性

| 属性名 | 类型 | 描述 |
|--------|------|------|
| `cursorSprite` | `Sprite2D` | 光标精灵节点引用 |
| `cursorArea` | `Area2D` | 光标碰撞区域节点引用 |
| `actionTargetableComponentInContact` | `Array[ActionTargetableComponent]` | 当前接触的可目标化组件列表 |

## 主要方法

### connectSignals() -> void
连接信号处理器。
- **功能**: 连接光标区域的进入和退出信号

### onCursorArea_areaEntered(area: Area2D) -> void
处理光标区域进入事件。
- **参数**: `area` - 进入的区域节点
- **功能**: 添加可目标化组件到接触列表，启用目标高亮

### onCursorArea_areaExited(area: Area2D) -> void
处理光标区域退出事件。
- **参数**: `area` - 退出的区域节点
- **功能**: 从接触列表移除组件，禁用目标高亮

## 使用示例

### 基本设置
```gdscript
# 在场景中添加ActionTargetingCursorComponentBase
@onready var cursor_targeting = $ActionTargetingCursorComponentBase

func _ready():
    # 设置行动
    cursor_targeting.action = my_spell_action
    
    # 自定义光标外观
    cursor_targeting.cursorSprite.texture = my_cursor_texture
    cursor_targeting.cursorSprite.modulate = Color.YELLOW
```

### 自定义光标样式
```gdscript
# 为不同行动设置不同的光标样式
func set_cursor_style(action_type: String):
    match action_type:
        "attack":
            cursor_targeting.cursorSprite.texture = load("res://cursors/attack_cursor.png")
            cursor_targeting.cursorSprite.modulate = Color.RED
        "heal":
            cursor_targeting.cursorSprite.texture = load("res://cursors/heal_cursor.png")
            cursor_targeting.cursorSprite.modulate = Color.GREEN
        "examine":
            cursor_targeting.cursorSprite.texture = load("res://cursors/examine_cursor.png")
            cursor_targeting.cursorSprite.modulate = Color.BLUE
```

### 监听目标变化
```gdscript
# 监听目标进入和退出
func _ready():
    super._ready()
    cursor_targeting.cursorArea.area_entered.connect(_on_target_entered)
    cursor_targeting.cursorArea.area_exited.connect(_on_target_exited)

func _on_target_entered(area: Area2D):
    print("目标进入光标范围")
    # 播放音效或显示提示

func _on_target_exited(area: Area2D):
    print("目标离开光标范围")
    # 隐藏提示信息
```

## 扩展示例

### 自定义光标控制
```gdscript
# 创建自定义光标控制组件
class_name CustomCursorTargeting
extends ActionTargetingCursorComponentBase

@export var cursor_speed: float = 200.0
@export var snap_to_targets: bool = true

func _process(delta):
    if not isEnabled or not isChoosing:
        return
    
    # 自定义光标移动逻辑
    update_cursor_position(delta)
    
    # 可选：自动吸附到目标
    if snap_to_targets:
        snap_to_nearest_target()

func update_cursor_position(delta: float):
    # 实现自定义光标移动逻辑
    pass

func snap_to_nearest_target():
    if actionTargetableComponentInContact.is_empty():
        return
    
    var nearest_target = find_nearest_target()
    if nearest_target:
        global_position = nearest_target.global_position
```

### 范围显示
```gdscript
# 添加范围显示功能
class_name RangeTargetingCursor
extends ActionTargetingCursorComponentBase

@export var show_range: bool = true
@export var range_radius: float = 100.0

var range_circle: Node2D

func _ready():
    super._ready()
    if show_range:
        create_range_indicator()

func create_range_indicator():
    range_circle = preload("res://ui/RangeCircle.tscn").instantiate()
    add_child(range_circle)
    range_circle.radius = range_radius
    range_circle.modulate = Color(1, 1, 1, 0.3)
```

### 多目标选择
```gdscript
# 支持多目标选择的光标
class_name MultiTargetCursor
extends ActionTargetingCursorComponentBase

@export var max_targets: int = 3
var selected_targets: Array[ActionTargetableComponent] = []

func onCursorArea_areaEntered(area: Area2D) -> void:
    super.onCursorArea_areaEntered(area)
    update_target_indicators()

func onCursorArea_areaExited(area: Area2D) -> void:
    super.onCursorArea_areaExited(area)
    update_target_indicators()

func update_target_indicators():
    # 更新目标指示器显示
    for i in range(actionTargetableComponentInContact.size()):
        var target = actionTargetableComponentInContact[i]
        if i < max_targets:
            target.setHighlight(true)
        else:
            target.setHighlight(false)
```

## 技术细节

### 碰撞检测机制
- 使用 `Area2D` 节点检测光标与目标的碰撞
- 自动管理可目标化组件的进入/退出状态
- 支持多个目标同时在碰撞范围内

### 目标高亮系统
- 自动调用目标组件的 `setHighlight()` 方法
- 进入范围时启用高亮，退出范围时禁用高亮
- 支持视觉反馈和用户交互提示

### UI显示
- 自动显示行动的显示名称
- 支持自定义光标外观和标签样式

## 注意事项

1. **抽象基类**: 这是一个抽象基类，需要具体的子类实现
2. **节点依赖**: 需要场景中包含 `CursorSprite`、`CursorArea` 和 `Label` 节点
3. **碰撞层设置**: 需要正确设置碰撞层以检测目标组件
4. **实验性功能**: 标记为实验性功能，API可能会发生变化

## 相关组件
- `ActionTargetingComponentBase` - 目标选择基类
- `ActionTargetingMouseComponent` - 鼠标目标选择组件
- `ActionTargetableComponent` - 可选择目标组件
- `ActionsComponent` - 行动管理组件

## 版本信息
- **状态**: 实验性功能
- **兼容性**: 需要Godot 4.x版本
- **依赖**: 需要正确的节点结构和碰撞层设置 