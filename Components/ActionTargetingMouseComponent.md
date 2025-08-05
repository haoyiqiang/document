# ActionTargetingMouseComponent API 文档

## 概述
`ActionTargetingMouseComponent` 是一个具体的鼠标控制目标选择组件，提供完整的鼠标交互功能。玩家可以通过鼠标移动光标，点击选择目标来执行行动，如魔法技能、对话或检查等操作。

## 继承关系
```
Component
└── ActionTargetingComponentBase
    └── ActionTargetingCursorComponentBase
        └── ActionTargetingMouseComponent
```

## 主要功能
- **鼠标控制**: 光标跟随鼠标位置移动
- **点击选择**: 左键点击选择光标下的目标
- **实时反馈**: 实时显示可选择的目标
- **自动清理**: 选择完成后自动删除组件

## 使用示例

### 基本设置
```gdscript
# 在场景中添加ActionTargetingMouseComponent
@onready var mouse_targeting = $ActionTargetingMouseComponent

func _ready():
    # 设置要执行的行动
    mouse_targeting.action = my_fireball_spell
    
    # 连接信号
    mouse_targeting.didChooseTarget.connect(_on_target_chosen)
    mouse_targeting.didCancel.connect(_on_target_cancelled)

func _on_target_chosen(entity: Entity):
    print("对 ", entity.name, " 施放了 ", mouse_targeting.action.displayName)

func _on_target_cancelled():
    print("取消了技能施放")
```

### 动态创建目标选择
```gdscript
# 动态创建鼠标目标选择组件
func start_spell_targeting(spell_action: Action):
    var targeting_scene = preload("res://components/ActionTargetingMouseComponent.tscn")
    var targeting_component = targeting_scene.instantiate()
    
    # 添加到实体
    add_child(targeting_component)
    
    # 配置组件
    targeting_component.action = spell_action
    targeting_component.didChooseTarget.connect(_on_spell_target_chosen)
    targeting_component.didCancel.connect(_on_spell_cancelled)

func _on_spell_target_chosen(target_entity: Entity):
    print("魔法目标: ", target_entity.name)
    # 处理魔法施放逻辑

func _on_spell_cancelled():
    print("取消了魔法施放")
    # 恢复玩家控制状态
```

### 条件性目标选择
```gdscript
# 根据条件启用/禁用目标选择
func try_cast_spell(spell: Action):
    if not can_cast_spell(spell):
        show_error_message("无法施放该魔法")
        return
    
    # 创建目标选择组件
    var targeting = create_targeting_component(spell)
    
    # 设置特定条件
    if spell.requires_line_of_sight:
        targeting.connect("target_validation", _validate_line_of_sight)

func can_cast_spell(spell: Action) -> bool:
    return player.mana >= spell.mana_cost and not player.is_silenced

func _validate_line_of_sight(target: Entity) -> bool:
    # 检查视线是否被阻挡
    return has_line_of_sight_to(target)
```

## 扩展示例

### 自定义鼠标行为
```gdscript
# 创建自定义鼠标目标选择组件
class_name CustomMouseTargeting
extends ActionTargetingMouseComponent

@export var right_click_cancels: bool = true
@export var double_click_required: bool = false

var click_timer: Timer
var click_count: int = 0

func _ready():
    super._ready()
    
    if double_click_required:
        click_timer = Timer.new()
        add_child(click_timer)
        click_timer.wait_time = 0.3
        click_timer.timeout.connect(_on_click_timeout)

func _input(event: InputEvent) -> void:
    if not isEnabled or not isChoosing:
        return
    
    if event is InputEventMouseButton:
        if event.button_index == MOUSE_BUTTON_LEFT and event.pressed:
            if double_click_required:
                handle_double_click()
            else:
                chooseTargetsUnderCursor()
        elif event.button_index == MOUSE_BUTTON_RIGHT and event.pressed and right_click_cancels:
            cancelTargetSelection()

func handle_double_click():
    click_count += 1
    if click_count == 1:
        click_timer.start()
    elif click_count == 2:
        click_timer.stop()
        click_count = 0
        chooseTargetsUnderCursor()

func _on_click_timeout():
    click_count = 0
```

### 范围限制
```gdscript
# 添加目标选择范围限制
class_name RangedMouseTargeting
extends ActionTargetingMouseComponent

@export var max_range: float = 200.0
@export var show_range_indicator: bool = true

var range_circle: Node2D

func _ready():
    super._ready()
    if show_range_indicator:
        create_range_indicator()

func create_range_indicator():
    range_circle = preload("res://ui/RangeCircle.tscn").instantiate()
    parentEntity.add_child(range_circle)
    range_circle.radius = max_range
    range_circle.modulate = Color(0, 1, 0, 0.3)

func _process(delta: float) -> void:
    super._process(delta)
    
    # 检查距离并更新光标颜色
    var distance = global_position.distance_to(parentEntity.global_position)
    if distance > max_range:
        cursorSprite.modulate = Color.RED
    else:
        cursorSprite.modulate = Color.WHITE

func chooseTargetsUnderCursor() -> Array[ActionTargetableComponent]:
    # 检查范围限制
    var distance = global_position.distance_to(parentEntity.global_position)
    if distance > max_range:
        show_error_message("目标超出范围")
        return []
    
    return super.chooseTargetsUnderCursor()
```

### 多目标选择
```gdscript
# 支持多目标选择的鼠标组件
class_name MultiTargetMouseComponent
extends ActionTargetingMouseComponent

@export var max_targets: int = 3
@export var require_confirmation: bool = true

var selected_targets: Array[ActionTargetableComponent] = []

func _input(event: InputEvent) -> void:
    if not isEnabled or not isChoosing:
        return
    
    if event is InputEventMouseButton and event.pressed:
        if event.button_index == MOUSE_BUTTON_LEFT:
            add_target_under_cursor()
        elif event.button_index == MOUSE_BUTTON_RIGHT:
            if require_confirmation and not selected_targets.is_empty():
                confirm_selection()
            else:
                cancelTargetSelection()

func add_target_under_cursor():
    if selected_targets.size() >= max_targets:
        show_error_message("已达到最大目标数量")
        return
    
    for target in actionTargetableComponentInContact:
        if target not in selected_targets:
            selected_targets.append(target)
            target.setHighlight(true)
            break

func confirm_selection():
    for target in selected_targets:
        chooseTarget(target)
    requestDeletion()
```

## 技术细节

### 鼠标位置跟踪
- 在 `_ready()` 时设置初始位置为鼠标位置
- 在 `_process()` 中持续跟踪鼠标位置
- 使用 `parentEntity.get_global_mouse_position()` 获取鼠标位置

### 输入处理
- 监听鼠标按钮事件
- 左键点击触发目标选择
- 支持扩展其他鼠标按钮功能

### 自动管理
- 选择完成后自动请求删除组件
- 如果没有可选择的目标，自动取消选择

## 注意事项

1. **实验性功能**: 标记为实验性功能，API可能会发生变化
2. **场景要求**: 需要正确的场景结构和节点配置
3. **输入优先级**: 可能需要管理输入事件的优先级
4. **性能考虑**: 频繁的位置更新可能影响性能

## 相关组件
- `ActionTargetingComponentBase` - 目标选择基类
- `ActionTargetingCursorComponentBase` - 光标目标选择基类
- `ActionTargetableComponent` - 可选择目标组件
- `ActionsComponent` - 行动管理组件
- `InputComponent` - 输入处理组件

## 版本信息
- **状态**: 实验性功能
- **兼容性**: 需要Godot 4.x版本
- **依赖**: 需要鼠标输入支持和正确的场景结构 