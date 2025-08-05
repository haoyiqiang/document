# ShapeDrawComponent API

## 概述
`ShapeDrawComponent` 绘制矢量形状的实验性组件。使用Godot的2D绘制API在画布上绘制线条等基本形状，适用于调试可视化、轨迹显示、UI装饰等场景。

**继承链**：`Component` → `ShapeDrawComponent`

**状态**：🧪 **实验性** - 功能可能会发生变化

## 主要特性
- 📐 **矢量绘制**：使用Godot 2D绘制API绘制形状
- 🎨 **多色支持**：每条线可以有不同的颜色
- 📏 **线宽控制**：支持像素精确的线宽设置
- ⚡ **性能优化**：使用PackedVector2Array高效存储点
- 🔧 **简单易用**：直接配置线条点和颜色

## 导出属性

### 线条配置
- **linePoints** (`PackedVector2Array`)：线条的起点和终点对
  - 每两个点组成一条不连接的线段
- **lineColors** (`PackedColorArray`)：每条线的颜色数组
  - 与linePoints中的线条对应
- **lineWidth** (`float = -1.0`)：线条宽度
  - 负值：保持1像素宽度，不受缩放影响
  - 正值：实际像素宽度，会随缩放变化

### 控制配置
- **isEnabled** (`bool = true`)：是否启用绘制

## 状态属性
- **selfAsCanvasItem** (`CanvasItem`)：用于绘制的画布项目

## 主要方法
- **_draw**() → `void`：Godot绘制回调，执行实际绘制

## 使用示例

### 基础线条绘制
```gdscript
# 绘制简单线条
@export var shape_drawer: ShapeDrawComponent

func _ready():
    # 绘制一条从(0,0)到(100,0)的红线
    shape_drawer.linePoints = PackedVector2Array([
        Vector2(0, 0), Vector2(100, 0)
    ])
    shape_drawer.lineColors = PackedColorArray([Color.RED])
    shape_drawer.lineWidth = 2.0
```

### 多线条系统
```gdscript
# 绘制多条不同颜色的线
@export var shape_drawer: ShapeDrawComponent

func draw_coordinate_system():
    var points = PackedVector2Array()
    var colors = PackedColorArray()
    
    # X轴 (红色)
    points.append(Vector2(-50, 0))
    points.append(Vector2(50, 0))
    colors.append(Color.RED)
    
    # Y轴 (绿色)
    points.append(Vector2(0, -50))
    points.append(Vector2(0, 50))
    colors.append(Color.GREEN)
    
    # 对角线 (蓝色)
    points.append(Vector2(-30, -30))
    points.append(Vector2(30, 30))
    colors.append(Color.BLUE)
    
    shape_drawer.linePoints = points
    shape_drawer.lineColors = colors
    shape_drawer.lineWidth = 1.0
    shape_drawer.queue_redraw()
```

### 动态轨迹绘制
```gdscript
# 实时轨迹绘制系统
@export var shape_drawer: ShapeDrawComponent
@export var traced_object: Node2D
var trail_points: Array[Vector2] = []
var max_trail_length: int = 50

func _ready():
    # 开始追踪
    var timer = Timer.new()
    timer.wait_time = 0.1
    timer.timeout.connect(update_trail)
    add_child(timer)
    timer.start()

func update_trail():
    if traced_object:
        trail_points.append(traced_object.global_position - global_position)
        
        # 限制轨迹长度
        if trail_points.size() > max_trail_length:
            trail_points.pop_front()
        
        draw_trail()

func draw_trail():
    var points = PackedVector2Array()
    var colors = PackedColorArray()
    
    # 连接轨迹点
    for i in range(trail_points.size() - 1):
        points.append(trail_points[i])
        points.append(trail_points[i + 1])
        
        # 渐变颜色，越新越亮
        var alpha = float(i) / float(trail_points.size())
        colors.append(Color(1, 1, 1, alpha))
    
    shape_drawer.linePoints = points
    shape_drawer.lineColors = colors
    shape_drawer.lineWidth = -1.0  # 像素精确
    shape_drawer.queue_redraw()
```

### 调试可视化
```gdscript
# 物理调试可视化
@export var shape_drawer: ShapeDrawComponent
@export var physics_body: RigidBody2D

func _ready():
    # 显示速度向量
    set_process(true)

func _process(_delta):
    if physics_body and shape_drawer.isEnabled:
        visualize_physics()

func visualize_physics():
    var points = PackedVector2Array()
    var colors = PackedColorArray()
    
    # 速度向量
    var velocity = physics_body.linear_velocity
    if velocity.length() > 0:
        points.append(Vector2.ZERO)
        points.append(velocity * 0.1)  # 缩放以便显示
        colors.append(Color.YELLOW)
    
    # 角速度指示
    var angular_vel = physics_body.angular_velocity
    if abs(angular_vel) > 0.1:
        var radius = 20
        var angle_step = PI / 8
        for i in range(8):
            var angle1 = i * angle_step
            var angle2 = (i + 1) * angle_step
            points.append(Vector2(cos(angle1), sin(angle1)) * radius)
            points.append(Vector2(cos(angle2), sin(angle2)) * radius)
            colors.append(Color.CYAN)
    
    shape_drawer.linePoints = points
    shape_drawer.lineColors = colors
    shape_drawer.queue_redraw()
```

### 网格绘制
```gdscript
# 绘制网格系统
@export var shape_drawer: ShapeDrawComponent
@export var grid_size: int = 32
@export var grid_extent: int = 10

func draw_grid():
    var points = PackedVector2Array()
    var colors = PackedColorArray()
    var grid_color = Color(0.3, 0.3, 0.3, 0.5)
    
    # 垂直线
    for x in range(-grid_extent, grid_extent + 1):
        var x_pos = x * grid_size
        points.append(Vector2(x_pos, -grid_extent * grid_size))
        points.append(Vector2(x_pos, grid_extent * grid_size))
        colors.append(grid_color)
    
    # 水平线
    for y in range(-grid_extent, grid_extent + 1):
        var y_pos = y * grid_size
        points.append(Vector2(-grid_extent * grid_size, y_pos))
        points.append(Vector2(grid_extent * grid_size, y_pos))
        colors.append(grid_color)
    
    shape_drawer.linePoints = points
    shape_drawer.lineColors = colors
    shape_drawer.lineWidth = -1.0
    shape_drawer.queue_redraw()
```

### 数据可视化
```gdscript
# 简单图表绘制
@export var shape_drawer: ShapeDrawComponent
var data_points: Array[float] = []

func draw_line_chart(data: Array[float]):
    data_points = data
    var points = PackedVector2Array()
    var colors = PackedColorArray()
    
    if data.size() < 2:
        return
    
    var chart_width = 200.0
    var chart_height = 100.0
    var max_value = data.max()
    var min_value = data.min()
    var value_range = max_value - min_value
    
    # 连接数据点
    for i in range(data.size() - 1):
        var x1 = (float(i) / float(data.size() - 1)) * chart_width
        var y1 = -((data[i] - min_value) / value_range) * chart_height
        var x2 = (float(i + 1) / float(data.size() - 1)) * chart_width
        var y2 = -((data[i + 1] - min_value) / value_range) * chart_height
        
        points.append(Vector2(x1, y1))
        points.append(Vector2(x2, y2))
        colors.append(Color.GREEN)
    
    shape_drawer.linePoints = points
    shape_drawer.lineColors = colors
    shape_drawer.lineWidth = 2.0
    shape_drawer.queue_redraw()
```

## 设计模式

### 形状绘制器管理
```gdscript
# 多形状绘制管理器
class_name ShapeDrawManager
extends Node

var drawers: Array[ShapeDrawComponent] = []

func register_drawer(drawer: ShapeDrawComponent):
    drawers.append(drawer)

func clear_all():
    for drawer in drawers:
        drawer.linePoints.clear()
        drawer.lineColors.clear()
        drawer.queue_redraw()

func set_all_enabled(enabled: bool):
    for drawer in drawers:
        drawer.isEnabled = enabled
        drawer.queue_redraw()
```

## 技术细节

### 绘制系统
- 使用draw_multiline_colors()高效绘制多条线
- 禁用抗锯齿以保持像素艺术风格
- 支持不连接的线段绘制

### 线宽处理
- 负值：保持"2点基元"，始终1像素宽
- 正值：实际像素宽度，受变换影响
- 适合不同的视觉需求

### 性能优化
- 使用PackedArray减少内存分配
- 避免每帧重复绘制
- 仅在需要时调用queue_redraw()

## 注意事项

### 实验性功能
- API可能在未来版本中更改
- 当前仅支持线条绘制
- 未来可能添加更多形状类型

### 性能考虑
- 大量线条可能影响性能
- 避免每帧更新大量几何体
- 考虑使用对象池优化

### 坐标系统
- 使用局部坐标系
- 相对于组件的transform
- 注意与父节点的变换关系

## 相关组件
- [`CanvasItem`](https://docs.godotengine.org/en/stable/classes/class_canvasitem.html) - Godot绘制基类
- [`Line2D`](https://docs.godotengine.org/en/stable/classes/class_line2d.html) - Godot专用线条节点 