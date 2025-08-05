# DebugComponent API 文档

## 概述
`DebugComponent` 显示关于实体和其他组件或节点的调试信息和图表。

## 继承
`DebugComponent` → `Component` → `Node`

## 主要特性
- 实时显示节点属性标签
- 创建属性变化图表窗口
- 支持鼠标悬停显示/隐藏
- 自定义颜色区分不同组件

## 导出属性

### 基础设置

#### `isEnabled: bool = true`
是否启用调试组件。禁用时停止处理以优化性能。

### 标签覆盖

#### `propertiesToLabel: Array[NodePath]`
要显示标签的节点及其属性的NodePath列表。
- 格式示例：`../HealthComponent:health:value`
- 使用 `/` 分隔节点，`:` 分隔属性

#### `shouldHideLabelsUntilHover: bool = false`
是否隐藏标签直到鼠标悬停。

### 图表窗口

#### `propertiesToChart: Array[NodePath]`
要创建图表窗口的节点及其属性的NodePath列表。
- 格式示例：`../CharacterBodyComponent:body:velocity:x`

#### `chartVerticalHeight: float = 100`
图表垂直高度（范围：100-200）。

#### `chartValueScale: float = 0.5`
图表值缩放比例（范围：0.1-2.0）。

## 主要方法

### 初始化方法

#### `convertPathsToAbsolute(relativePaths: Array[NodePath]) -> Array[NodePath]`
将相对路径转换为绝对路径表示。

#### `createCharts() -> void`
为指定的属性创建图表窗口。

### 标签管理

#### `updateLabelsVisibility() -> void`
更新标签的可见性状态。

#### `updatePropertiesLabel() -> void`
更新属性标签的文本内容。

### 事件处理

#### `onVisibilityToggleHotspot_mouseEntered() -> void`
鼠标进入时显示标签。

#### `onVisibilityToggleHotspot_mouseExited() -> void`
鼠标离开时淡出标签。

## 使用示例

```gdscript
# 在场景中添加DebugComponent
var debugComponent = DebugComponent.new()
entity.add_child(debugComponent)

# 设置要监控的属性
debugComponent.propertiesToLabel = [
    NodePath("../HealthComponent:currentHealth"),
    NodePath("../MovementComponent:velocity")
]

# 设置要绘制图表的属性
debugComponent.propertiesToChart = [
    NodePath("../HealthComponent:currentHealth"),
    NodePath("../PhysicsComponent:velocity:x")
]

# 配置显示选项
debugComponent.shouldHideLabelsUntilHover = true
debugComponent.chartVerticalHeight = 150
debugComponent.chartValueScale = 1.0
```

## 路径格式说明

### 节点路径
使用标准的NodePath语法：
- `../` - 父节点
- `./` - 当前节点
- `SomeChild` - 子节点

### 属性路径
使用 `:` 分隔属性层次：
- `propertyName` - 直接属性
- `object:property` - 对象的属性
- `object:property:subproperty` - 嵌套属性

### 完整示例
```gdscript
# 监控兄弟组件的属性
"../HealthComponent:currentHealth"

# 监控物理体的速度X分量
"../CharacterBodyComponent:velocity:x"

# 监控资源的值
"../StatsComponent:mana:current_value"
```

## 性能注意事项

1. **处理优化**：组件禁用时自动停止处理
2. **标签更新**：仅在有属性需要显示时进行更新
3. **图表创建**：图表在_ready时创建，运行时不重复创建

## 可视化特性

1. **随机颜色**：每个DebugComponent使用不同颜色区分
2. **图表颜色**：每个图表使用随机颜色
3. **淡入淡出**：支持鼠标悬停的平滑显示/隐藏效果
4. **Z层级**：悬停模式下标签使用更高的z层级

## 相关类
- `Component` - 基础组件类
- `Entity` - 实体容器
- `Chart` - 图表显示类
- `Tools` - 路径处理工具 