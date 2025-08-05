# AreaComponentBase API

> **继承关系**: Component > AreaComponentBase  
> **抽象类**: 必须由子类实现

Area2D基础组件的抽象基类，为依赖、监控或操作Area2D的组件提供统一的Area2D管理和选择机制。

## ✨ 主要特性

- 🎯 智能Area2D选择机制
- 🔄 支持Area2D重写和自动发现
- 📐 自动计算Area2D边界
- 🏗️ 为子类提供Area2D操作基础
- 🔧 支持全局和本地坐标系

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `areaOverride` | `Area2D` | `null` | 指定要使用的Area2D，如果未指定则自动选择 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `area` | `Area2D` | 当前使用的Area2D（自动选择或手动指定） |
| `selfAsArea` | `Area2D` | 组件本身作为Area2D的引用 |
| `selfAscollisionObject` | `CollisionObject2D` | 组件作为CollisionObject2D的引用 |

## 🔍 派生属性

### 边界计算
```gdscript
var areaBounds: Rect2
var areaBoundsGlobal: Rect2
```

| 属性 | 类型 | 描述 |
|------|------|------|
| `areaBounds` | `Rect2` | Area2D第一个CollisionShape2D的本地边界 |
| `areaBoundsGlobal` | `Rect2` | Area2D的全局边界（本地边界+全局位置） |

## 🛠️ 主要方法

### 边界更新
```gdscript
func updateAreaBounds() -> Rect2
```
**描述**: 更新并返回Area2D的边界矩形  
**返回**: Area2D的本地边界

## 🎯 Area2D选择逻辑

组件按以下优先级选择Area2D：

1. **areaOverride** - 手动指定的Area2D
2. **selfAsArea** - 组件本身（如果是Area2D）
3. **parentEntity.getArea()** - 父实体的Area2D
4. **null** - 无可用Area2D（会输出警告）

## 🎯 使用示例

### 创建Area组件子类

```gdscript
# CustomAreaComponent.gd
class_name CustomAreaComponent
extends AreaComponentBase

func _ready():
    super._ready()
    
    # 使用自动选择的Area2D
    if area:
        print("Using area: ", area.name)
        print("Area bounds: ", areaBounds)
        print("Global bounds: ", areaBoundsGlobal)
    else:
        print("No area available!")

func processAreaCollision(otherArea: Area2D):
    # 使用继承的area属性
    if area and area.overlaps_area(otherArea):
        print("Collision detected!")
```

### 边界检测组件

```gdscript
# AreaBoundsChecker.gd
class_name AreaBoundsChecker
extends AreaComponentBase

@export var targetPosition: Vector2

func _ready():
    super._ready()
    checkIfPositionInBounds()

func checkIfPositionInBounds():
    if not area:
        return false
    
    # 使用全局边界检查
    return areaBoundsGlobal.has_point(targetPosition)

func getClosestPointOnBounds(point: Vector2) -> Vector2:
    if not area:
        return Vector2.ZERO
    
    # 获取最近的边界点
    return areaBoundsGlobal.get_center().move_toward(point, areaBoundsGlobal.size.length() / 2)
```

### 动态Area2D管理

```gdscript
# DynamicAreaManager.gd
class_name DynamicAreaManager
extends AreaComponentBase

@export var alternativeAreas: Array[Area2D] = []
var currentAreaIndex: int = 0

func _ready():
    super._ready()
    setupAreaSwitching()

func setupAreaSwitching():
    # 添加原始area到列表
    if area:
        alternativeAreas.insert(0, area)

func switchToNextArea():
    if alternativeAreas.is_empty():
        return
    
    currentAreaIndex = (currentAreaIndex + 1) % alternativeAreas.size()
    
    # 手动设置新的area
    areaOverride = alternativeAreas[currentAreaIndex]
    area = areaOverride
    
    # 更新边界
    updateAreaBounds()
    
    print("Switched to area: ", area.name)
```

### 区域工具组件

```gdscript
# AreaUtilityComponent.gd
class_name AreaUtilityComponent
extends AreaComponentBase

func _ready():
    super._ready()

func getAreaInfo() -> Dictionary:
    if not area:
        return {}
    
    return {
        "name": area.name,
        "position": area.global_position,
        "bounds_local": areaBounds,
        "bounds_global": areaBoundsGlobal,
        "collision_layers": area.collision_layer,
        "collision_mask": area.collision_mask
    }

func isPointInArea(point: Vector2) -> bool:
    return areaBoundsGlobal.has_point(point)

func getRandomPointInArea() -> Vector2:
    if areaBounds.size == Vector2.ZERO:
        return area.global_position
    
    var randomLocal = Vector2(
        randf_range(areaBounds.position.x, areaBounds.position.x + areaBounds.size.x),
        randf_range(areaBounds.position.y, areaBounds.position.y + areaBounds.size.y)
    )
    
    return randomLocal + area.global_position

func getAreaCorners() -> Array[Vector2]:
    var corners: Array[Vector2] = []
    var bounds = areaBoundsGlobal
    
    corners.append(bounds.position)  # 左上
    corners.append(Vector2(bounds.end.x, bounds.position.y))  # 右上
    corners.append(bounds.end)  # 右下
    corners.append(Vector2(bounds.position.x, bounds.end.y))  # 左下
    
    return corners
```

## 🏛️ 设计模式

### 模板方法模式
- **基类**: AreaComponentBase提供Area2D管理框架
- **子类**: 实现具体的Area2D操作逻辑
- **通用功能**: 边界计算、Area选择等

### 策略模式
- **上下文**: Area2D选择逻辑
- **策略**: 不同的Area2D获取方式
- **回退**: 多级回退机制确保可用性

## 🔧 技术细节

### Area2D选择机制
```gdscript
func _enter_tree() -> void:
    # 1. 优先使用手动指定的areaOverride
    if self.areaOverride:
        self.area = self.areaOverride
    
    # 2. 尝试使用组件本身作为Area2D
    if not self.area:
        self.area = selfAsArea
    
    # 3. 使用父实体的Area2D
    if not self.area:
        self.area = parentEntity.getArea()
```

### 边界计算
```gdscript
func updateAreaBounds() -> Rect2:
    self.areaBounds = Tools.getShapeBoundsInNode(area)
    return areaBounds
```

### 延迟加载CollisionObject2D
```gdscript
var selfAscollisionObject: CollisionObject2D:
    get:
        if not selfAscollisionObject: 
            selfAscollisionObject = self.get_node(^".") as CollisionObject2D
        return selfAscollisionObject
```

## ⚠️ 注意事项

1. **抽象类**: 不能直接实例化，必须通过子类使用
2. **Area2D依赖**: 确保至少有一个可用的Area2D
3. **边界更新**: 如果Area2D形状在运行时改变，需要调用updateAreaBounds()
4. **坐标系**: areaBounds是本地坐标，areaBoundsGlobal是全局坐标
5. **性能考虑**: areaBounds使用延迟计算，避免不必要的重复计算

## 🔗 相关组件

- [AreaCollisionComponent](AreaCollisionComponent.md) - Area碰撞检测
- [AreaContactComponent](AreaContactComponent.md) - Area接触管理
- [ModifyOnCollisionComponent](ModifyOnCollisionComponent.md) - 碰撞修改
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21