# IndependentPathFollowComponent API 参考

## 概述

`IndependentPathFollowComponent` 在不使用 `PathFollow2D` 的情况下沿着 `Path2D` 移动父实体。这允许多个实体在同一路径上具有不同的位置（进度偏移）。该组件直接设置实体位置，不使用物理系统。

**继承关系：**
`Component` → `IndependentPathFollowComponent`

## 主要特性

- 🛤️ **独立路径跟随** - 无需PathFollow2D节点
- 🎯 **多实体支持** - 多个实体可共享同一路径
- 🔄 **灵活定位** - 支持路径吸附和偏移模式
- 📐 **旋转控制** - 可选的沿路径方向旋转
- ⚠️ **实验性功能** - 正在开发中的高级功能

## 导出属性

### 路径设置
```gdscript
@export var path: Path2D
```
要跟随的路径。如果省略，则使用此组件的 `Path2D` 子节点，或实体的父节点（如果是 `Path2D`）。

### 移动参数
```gdscript
@export_range(-2000, 2000, 10) var speed: float = 100
```
沿路径的移动速度。

### 行为设置
```gdscript
@export var shouldSnapEntityToPath: bool = true
```
如果为 `true`（默认），实体会吸附到路径上距离当前位置最近的点。如果为 `false`，则实体相对于起始位置沿路径移动。

```gdscript
@export var shouldRotate: bool = false
```
是否根据路径方向旋转实体。

```gdscript
@export var shouldCubicInterpolate: bool = false
```
是否使用三次插值。三次插值能更好地跟随曲线，但线性插值更快。

```gdscript
@export var isEnabled: bool = true
```
控制组件是否启用。

## 状态属性

### 路径状态
```gdscript
var curve: Curve2D
```
Path2D的曲线引用。

```gdscript
var progress: float
```
沿曲线的当前进度（以距离为单位）。

## 信号

### didCompletePath
```gdscript
signal didCompletePath
```
当沿 `Path2D` 的 `Curve2D` 完成一个回路时发出。

## 主要方法

### snapEntityToCurve()
```gdscript
func snapEntityToCurve() -> Vector2
```
将实体位置吸附到路径上最近的点：
- 设置相应的 `progress` 值
- 遵循 `shouldRotate` 和 `shouldCubicInterpolate` 标志
- 返回新的吸附位置

### setPosition()
```gdscript
func setPosition() -> Vector2
```
根据当前进度设置实体在路径上的位置，支持旋转和插值选项。

## 使用示例

### 基本路径跟随
```gdscript
# 让实体沿预定义路径移动
extends Entity

func _ready():
    var path_follow = $IndependentPathFollowComponent
    
    # 配置基本设置
    path_follow.speed = 80.0
    path_follow.shouldSnapEntityToPath = true
    path_follow.shouldRotate = true
    path_follow.shouldCubicInterpolate = true
```

### 多实体共享路径
```gdscript
# 让多个实体以不同偏移跟随同一路径
extends PathManager

@export var entity_template: PackedScene
@export var entity_count: int = 5

func _ready():
    var shared_path = $Path2D
    create_path_followers(shared_path)

func create_path_followers(path: Path2D):
    for i in entity_count:
        var entity = entity_template.instantiate()
        add_child(entity)
        
        var path_component = entity.get_node("IndependentPathFollowComponent")
        path_component.path = path
        path_component.speed = randf_range(50, 150)
        
        # 设置不同的起始进度
        var start_offset = (path.curve.get_baked_length() / entity_count) * i
        path_component.progress = start_offset
```

### 动态路径切换
```gdscript
# 在运行时切换路径
extends Vehicle

@onready var path_follow = $IndependentPathFollowComponent
var available_paths: Array[Path2D] = []

func _ready():
    # 收集可用路径
    available_paths = get_tree().get_nodes_in_group("vehicle_paths")
    setup_initial_path()

func setup_initial_path():
    if available_paths.size() > 0:
        switch_to_path(available_paths[0])

func switch_to_path(new_path: Path2D):
    # 记录当前位置
    var current_position = global_position
    
    # 切换路径
    path_follow.path = new_path
    path_follow.setDependencies()
    
    # 如果需要，吸附到新路径
    if path_follow.shouldSnapEntityToPath:
        path_follow.snapEntityToCurve()
    
    print("切换到新路径: ", new_path.name)

func _input(event):
    if event.is_action_pressed("switch_path"):
        var current_index = available_paths.find(path_follow.path)
        var next_index = (current_index + 1) % available_paths.size()
        switch_to_path(available_paths[next_index])
```

### 路径进度控制
```gdscript
# 精确控制路径进度
extends ControlledVehicle

@onready var path_follow = $IndependentPathFollowComponent

func _ready():
    # 禁用自动移动
    path_follow.speed = 0
    path_follow.shouldSnapEntityToPath = false

func _process(delta):
    # 手动控制进度
    if Input.is_action_pressed("move_forward"):
        path_follow.progress += 100 * delta
    elif Input.is_action_pressed("move_backward"):
        path_follow.progress -= 100 * delta
    
    # 确保进度在有效范围内
    if path_follow.curve:
        var max_length = path_follow.curve.get_baked_length()
        path_follow.progress = clamp(path_follow.progress, 0, max_length)
    
    # 更新位置
    path_follow.setPosition()

func get_progress_ratio() -> float:
    if path_follow.curve and path_follow.curve.get_baked_length() > 0:
        return path_follow.progress / path_follow.curve.get_baked_length()
    return 0.0
```

### 路径动画系统
```gdscript
# 创建复杂的路径动画
extends AnimatedEntity

@onready var path_follow = $IndependentPathFollowComponent
@onready var tween = create_tween()

func _ready():
    path_follow.didCompletePath.connect(_on_path_completed)
    start_path_animation()

func start_path_animation():
    if not path_follow.curve:
        return
    
    var path_length = path_follow.curve.get_baked_length()
    var duration = path_length / path_follow.speed
    
    # 使用Tween创建平滑的进度动画
    tween.tween_method(
        func(progress: float): path_follow.progress = progress,
        0.0,
        path_length,
        duration
    )
    tween.tween_callback(_on_animation_complete)

func _on_path_completed():
    # 路径完成后的处理
    create_completion_effect()

func _on_animation_complete():
    # 动画完成后重新开始或切换到下一个路径
    await get_tree().create_timer(1.0).timeout
    start_path_animation()

func create_completion_effect():
    # 创建完成特效
    var particles = preload("res://Effects/PathCompleteEffect.tscn").instantiate()
    get_parent().add_child(particles)
    particles.global_position = global_position
```

## 技术细节

### 路径计算
- 使用 `Curve2D.sample_baked()` 或 `sample_baked_with_rotation()` 获取位置
- 支持线性和三次插值
- 自动处理坐标空间转换

### 依赖检测
- 自动查找 Path2D 引用
- 验证 Curve2D 的有效性
- 处理空曲线和零长度曲线

### 位置同步
- 直接设置 `global_position`
- 不使用物理系统（无 `velocity`）
- 支持同时设置位置和旋转

## 注意事项

⚠️ **重要要求：**
- `Path2D` 必须有有效的 `Curve2D`
- 启用"Editable Children"来修改子路径节点
- 不与基于物理的移动组件混合使用

💡 **性能考虑：**
- 三次插值比线性插值更耗费性能
- 多个实体共享路径时优化路径访问
- 合理设置更新频率

🔮 **使用技巧：**
- 结合动画系统创建复杂效果
- 使用进度比例创建同步动画
- 配合粒子系统增强视觉效果

## 相关组件

- [`PathFollowComponent`](PathFollowComponent.md) - 基于PathFollow2D的路径跟随
- [`RelativePathMovementComponent`](RelativePathMovementComponent.md) - 相对移动向量
- [`LinearMotionComponent`](LinearMotionComponent.md) - 直线运动
- [`NavigationComponent`](NavigationComponent.md) - 智能导航系统 