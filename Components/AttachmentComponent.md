# AttachmentComponent API

## 概述
`AttachmentComponent` 是一个专门用于将节点附着到组件位置的运动组件。它会在每帧将指定的节点位置设置为组件的位置，可用于实现悬浮物品、附着装饰等效果。

**继承链**：`Component` → `AttachmentComponent`

## 主要特性
- 🔗 **节点附着**：将任意CanvasItem节点附着到组件位置
- 📍 **偏移支持**：支持自定义位置偏移
- ⏸️ **可控启用**：支持运行时启用/禁用附着
- 🎯 **精确定位**：使用组件位置而非实体位置，提供更精确的控制
- ⚡ **性能优化**：对CollisionObject2D自动重置物理插值

## 导出属性

### 核心配置
- **nodeToAttach** (`CanvasItem`)：要附着的目标节点
- **offset** (`Vector2`)：相对于组件位置的偏移量
- **isEnabled** (`bool = true`)：是否启用附着功能

## 使用示例

### 基础附着
```gdscript
# 设置基本的节点附着
@export var attachment_component: AttachmentComponent
@export var floating_sprite: Sprite2D

func _ready():
    attachment_component.nodeToAttach = floating_sprite
    attachment_component.offset = Vector2(0, -32)  # 悬浮在上方32像素
```

### 动态附着管理
```gdscript
# 动态管理附着状态
func attach_decoration(decoration: Node2D, offset: Vector2 = Vector2.ZERO):
    attachment_component.nodeToAttach = decoration
    attachment_component.offset = offset
    attachment_component.isEnabled = true

func detach_decoration():
    attachment_component.isEnabled = false
    attachment_component.nodeToAttach = null
```

### 多个附着点
```gdscript
# 使用多个AttachmentComponent创建附着系统
@export var left_attachment: AttachmentComponent
@export var right_attachment: AttachmentComponent
@export var top_attachment: AttachmentComponent

func setup_attachments():
    # 左侧装饰
    left_attachment.nodeToAttach = left_decoration
    left_attachment.offset = Vector2(-40, 0)
    
    # 右侧装饰
    right_attachment.nodeToAttach = right_decoration
    right_attachment.offset = Vector2(40, 0)
    
    # 顶部装饰
    top_attachment.nodeToAttach = top_decoration
    top_attachment.offset = Vector2(0, -50)
```

### 条件附着
```gdscript
# 基于条件的附着控制
@export var attachment_component: AttachmentComponent

func update_attachment_based_on_health():
    var health_percentage = health_component.health.percentage
    
    if health_percentage < 30:
        # 血量低时显示警告标识
        attachment_component.nodeToAttach = warning_icon
        attachment_component.isEnabled = true
    else:
        # 血量正常时隐藏
        attachment_component.isEnabled = false
```

### 动画附着偏移
```gdscript
# 创建动态偏移动画
@export var attachment_component: AttachmentComponent
var base_offset: Vector2 = Vector2(0, -32)

func animate_attachment():
    var tween = create_tween()
    tween.set_loops()
    tween.tween_method(update_attachment_offset, 0.0, TAU, 2.0)

func update_attachment_offset(angle: float):
    var wave_offset = Vector2(sin(angle) * 5, cos(angle) * 3)
    attachment_component.offset = base_offset + wave_offset
```

## 设计模式

### 装饰模式
```gdscript
# 为实体添加可选装饰
class_name DecorationManager
extends Node

@export var decorations: Array[PackedScene]
@export var attachment_components: Array[AttachmentComponent]

func add_decoration(decoration_index: int, attachment_index: int):
    if decoration_index >= decorations.size() or attachment_index >= attachment_components.size():
        return
    
    var decoration = decorations[decoration_index].instantiate()
    get_tree().current_scene.add_child(decoration)
    
    attachment_components[attachment_index].nodeToAttach = decoration
    attachment_components[attachment_index].isEnabled = true
```

### 状态指示器
```gdscript
# 使用附着组件显示状态
@export var status_attachment: AttachmentComponent
@export var status_icons: Array[Texture2D]

func update_status_display(status: GameState.Status):
    if not status_sprite:
        status_sprite = Sprite2D.new()
        get_tree().current_scene.add_child(status_sprite)
        status_attachment.nodeToAttach = status_sprite
    
    status_sprite.texture = status_icons[status]
    status_attachment.isEnabled = (status != GameState.Status.NORMAL)
```

## 技术细节

### 位置更新机制
- 在`_process()`中每帧更新位置
- 使用`global_position`确保世界坐标正确性
- 使用组件位置而非实体位置，提供额外的控制精度

### 物理优化
- 对`CollisionObject2D`类型节点自动调用`reset_physics_interpolation()`
- 确保附着节点的物理表现正确

### 错误处理
- 在`_ready()`时检查`nodeToAttach`是否设置
- 运行时检查避免空引用错误

## 注意事项

### 性能考虑
- 附着功能在`_process()`中运行，每帧都会更新位置
- 大量附着组件可能影响性能
- 建议在不需要时禁用组件

### 与其他系统的区别
- 与`RideableComponent`不同，这是单向的位置跟随
- 不涉及复杂的载具逻辑或交互
- 适用于装饰和视觉效果

### 使用建议
- 对于静态装饰，考虑直接作为子节点
- 对于需要动态控制的附着，使用此组件
- 可以与其他运动组件组合使用

## 相关组件
- [`RideableComponent`](./RideableComponent.md) - 载具系统
- [`WaveMotionComponent`](./WaveMotionComponent.md) - 波形运动
- [`LinearMotionComponent`](./LinearMotionComponent.md) - 直线运动