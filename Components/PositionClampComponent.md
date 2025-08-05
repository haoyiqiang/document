# PositionClampComponent API 参考

## 概述

`PositionClampComponent` 每帧限制父 [`Entity`](../Entity.md) 的全局位置在指定范围内。该组件用于防止实体移动到游戏区域边界之外。

**继承关系：**
`Component` → `PositionClampComponent`

## 主要特性

- 🔒 **位置限制** - 将实体位置限制在指定矩形区域内
- ⚡ **实时处理** - 每帧检查并调整位置
- 🎮 **简单配置** - 仅需设置最小和最大坐标
- 🎯 **轻量级** - 最小性能开销的位置管理

## 导出属性

### 边界设置
```gdscript
@export var minimum: Vector2 = Vector2.ZERO
```
允许的最小位置坐标（左上角）。

```gdscript
@export var maximum: Vector2 = Vector2(500, 500)
```
允许的最大位置坐标（右下角）。

## 使用示例

### 基本游戏区域限制
```gdscript
# 在玩家实体中限制移动范围
extends PlayerEntity

func _ready():
    var position_clamp = $PositionClampComponent
    
    # 设置游戏区域边界
    position_clamp.minimum = Vector2(0, 0)
    position_clamp.maximum = Vector2(1920, 1080)
```

### 动态边界调整
```gdscript
# 根据关卡动态调整边界
extends Entity

func setup_level_boundaries(level_data: Dictionary):
    var clamp_component = $PositionClampComponent
    
    clamp_component.minimum = level_data.get("min_bounds", Vector2.ZERO)
    clamp_component.maximum = level_data.get("max_bounds", Vector2(1000, 1000))

func enter_restricted_area(new_bounds: Rect2):
    var clamp_component = $PositionClampComponent
    clamp_component.minimum = new_bounds.position
    clamp_component.maximum = new_bounds.end
```

### 相机跟随边界
```gdscript
# 为相机设置跟随边界
extends CameraEntity

func setup_camera_bounds(level_size: Vector2):
    var clamp_component = $PositionClampComponent
    
    # 相机边界应考虑视口大小
    var viewport_size = get_viewport().size
    var half_viewport = viewport_size / 2
    
    clamp_component.minimum = half_viewport
    clamp_component.maximum = level_size - half_viewport
```

## 技术细节

### 处理机制
- 在 `_process()` 中每帧调用 `Vector2.clamp()`
- 直接修改实体的 `position` 属性
- 无碰撞检测，纯数学位置限制

### 性能特点
- 极轻量级，每帧仅一次数学运算
- 无内存分配
- 适合大量实体同时使用

## 注意事项

⚠️ **调试信息：** 当前版本包含调试输出 `"SCRIPT ONLY WORKING"`，这可能在生产环境中产生大量日志。

💡 **最佳实践：**
- 根据游戏类型调整边界
- 考虑与相机系统配合使用
- 可与其他移动组件组合使用

## 相关组件

- [`LinearMotionComponent`](LinearMotionComponent.md) - 可能需要位置限制的运动组件
- [`CameraComponent`](../Visual/CameraComponent.md) - 常与此组件配合的相机系统
- [`CharacterBodyComponent`](../Physics/CharacterBodyComponent.md) - 可能受位置限制影响的物理组件 