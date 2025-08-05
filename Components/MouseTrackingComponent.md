# MouseTrackingComponent API

> **继承关系**: Component > MouseTrackingComponent  
> **控制类型**: 鼠标跟踪

将实体移动到鼠标位置的组件，支持即时传送或渐进移动。适用于光标、准星、UI元素等需要跟随鼠标的游戏对象。

## ✨ 主要特性

- 🖱️ 实时鼠标位置跟踪
- ⚡ 即时传送或平滑移动
- 🎛️ 可调节移动速度
- 🔄 可启用/禁用功能
- 📍 精确的位置同步

## 📊 导出属性

### 跟踪设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldRepositionImmediately` | `bool` | `true` | 是否立即移动到鼠标位置 |
| `speed` | `float` | `100.0` | 渐进移动速度 (0-1000) |
| `isEnabled` | `bool` | `true` | 是否启用鼠标跟踪 |

## 🎯 使用示例

### 基础光标跟踪

```gdscript
# Entity Scene Structure:
# └── Cursor (Node2D)
#     ├── Sprite2D
#     └── MouseTrackingComponent
#         shouldRepositionImmediately: true
```

### 平滑跟随光标

```gdscript
# Entity Scene Structure:
# └── FollowCursor (Node2D)
#     ├── Sprite2D
#     └── MouseTrackingComponent
#         shouldRepositionImmediately: false
#         speed: 200.0
```

### 智能光标组件

```gdscript
# SmartCursorComponent.gd
extends MouseTrackingComponent

@export var snapToGrid: bool = false
@export var gridSize: Vector2 = Vector2(32, 32)
@export var followOnlyInArea: bool = false
@export var allowedArea: Rect2 = Rect2()

func _physics_process(delta: float) -> void:
    if not isEnabled: 
        return
    
    var targetPosition = getAdjustedMousePosition()
    
    if shouldRepositionImmediately:
        parentEntity.global_position = targetPosition
    else:
        parentEntity.global_position = parentEntity.global_position.move_toward(targetPosition, speed * delta)

func getAdjustedMousePosition() -> Vector2:
    var mousePos = parentEntity.get_global_mouse_position()
    
    # 网格吸附
    if snapToGrid:
        mousePos = snapToGridPosition(mousePos)
    
    # 区域限制
    if followOnlyInArea:
        mousePos = clampToArea(mousePos)
    
    return mousePos

func snapToGridPosition(position: Vector2) -> Vector2:
    var snappedX = round(position.x / gridSize.x) * gridSize.x
    var snappedY = round(position.y / gridSize.y) * gridSize.y
    return Vector2(snappedX, snappedY)

func clampToArea(position: Vector2) -> Vector2:
    return Vector2(
        clamp(position.x, allowedArea.position.x, allowedArea.position.x + allowedArea.size.x),
        clamp(position.y, allowedArea.position.y, allowedArea.position.y + allowedArea.size.y)
    )
```

### 延迟跟踪组件

```gdscript
# DelayedTrackingComponent.gd
extends MouseTrackingComponent

@export var trackingDelay: float = 0.1
@export var positionHistory: Array[Vector2] = []
@export var maxHistorySize: int = 10

var lastUpdateTime: float = 0.0

func _physics_process(delta: float) -> void:
    if not isEnabled:
        return
    
    var currentTime = Time.get_time_dict_from_system()
    
    # 记录鼠标位置历史
    if currentTime - lastUpdateTime >= trackingDelay / maxHistorySize:
        recordMousePosition()
        lastUpdateTime = currentTime
    
    # 使用延迟的位置
    var targetPosition = getDelayedPosition()
    
    if shouldRepositionImmediately:
        parentEntity.global_position = targetPosition
    else:
        parentEntity.global_position = parentEntity.global_position.move_toward(targetPosition, speed * delta)

func recordMousePosition():
    var mousePos = parentEntity.get_global_mouse_position()
    positionHistory.append(mousePos)
    
    # 限制历史记录大小
    if positionHistory.size() > maxHistorySize:
        positionHistory = positionHistory.slice(-maxHistorySize)

func getDelayedPosition() -> Vector2:
    if positionHistory.is_empty():
        return parentEntity.get_global_mouse_position()
    
    # 使用历史记录中的位置
    var delayIndex = max(0, positionHistory.size() - 1)
    return positionHistory[delayIndex]

func clearHistory():
    positionHistory.clear()

func setTrackingDelay(newDelay: float):
    trackingDelay = max(0.0, newDelay)
```

### 条件跟踪组件

```gdscript
# ConditionalTrackingComponent.gd
extends MouseTrackingComponent

@export var onlyTrackWhenPressed: bool = false
@export var requiredMouseButton: int = MOUSE_BUTTON_LEFT
@export var trackingRange: float = 0.0  # 0为无限制
@export var centerPosition: Vector2 = Vector2.ZERO

var isMousePressed: bool = false

func _ready():
    super._ready()
    if centerPosition == Vector2.ZERO:
        centerPosition = parentEntity.global_position

func _input(event: InputEvent):
    if onlyTrackWhenPressed and event is InputEventMouseButton:
        var mouseEvent = event as InputEventMouseButton
        if mouseEvent.button_index == requiredMouseButton:
            isMousePressed = mouseEvent.pressed

func _physics_process(delta: float) -> void:
    if not isEnabled:
        return
    
    # 检查跟踪条件
    if onlyTrackWhenPressed and not isMousePressed:
        return
    
    var mousePos = parentEntity.get_global_mouse_position()
    
    # 检查跟踪范围
    if trackingRange > 0.0:
        var distance = centerPosition.distance_to(mousePos)
        if distance > trackingRange:
            # 限制在范围内
            var direction = (mousePos - centerPosition).normalized()
            mousePos = centerPosition + direction * trackingRange
    
    if shouldRepositionImmediately:
        parentEntity.global_position = mousePos
    else:
        parentEntity.global_position = parentEntity.global_position.move_toward(mousePos, speed * delta)

func setCenterPosition(newCenter: Vector2):
    centerPosition = newCenter

func isWithinTrackingRange() -> bool:
    if trackingRange <= 0.0:
        return true
    
    var mousePos = parentEntity.get_global_mouse_position()
    return centerPosition.distance_to(mousePos) <= trackingRange
```

### 多层级跟踪

```gdscript
# LayeredTrackingComponent.gd
extends MouseTrackingComponent

@export var enableParallax: bool = false
@export var parallaxFactor: Vector2 = Vector2(0.5, 0.5)
@export var basePosition: Vector2 = Vector2.ZERO
@export var trackingLayers: Array[Node2D] = []

func _ready():
    super._ready()
    if basePosition == Vector2.ZERO:
        basePosition = parentEntity.global_position

func _physics_process(delta: float) -> void:
    if not isEnabled:
        return
    
    var mousePos = parentEntity.get_global_mouse_position()
    var targetPosition: Vector2
    
    if enableParallax:
        # 视差效果
        var offset = (mousePos - basePosition) * parallaxFactor
        targetPosition = basePosition + offset
    else:
        targetPosition = mousePos
    
    # 更新主实体
    if shouldRepositionImmediately:
        parentEntity.global_position = targetPosition
    else:
        parentEntity.global_position = parentEntity.global_position.move_toward(targetPosition, speed * delta)
    
    # 更新其他层级
    updateTrackingLayers(targetPosition, delta)

func updateTrackingLayers(mainPosition: Vector2, delta: float):
    for i in range(trackingLayers.size()):
        var layer = trackingLayers[i]
        if not layer:
            continue
        
        # 不同层级使用不同的跟踪速度
        var layerSpeed = speed * (1.0 - i * 0.2)  # 每层递减20%
        var layerTarget = mainPosition + Vector2(i * 5, i * 5)  # 轻微偏移
        
        if shouldRepositionImmediately:
            layer.global_position = layerTarget
        else:
            layer.global_position = layer.global_position.move_toward(layerTarget, layerSpeed * delta)

func addTrackingLayer(layer: Node2D):
    if layer and layer not in trackingLayers:
        trackingLayers.append(layer)

func removeTrackingLayer(layer: Node2D):
    trackingLayers.erase(layer)

func clearTrackingLayers():
    trackingLayers.clear()
```

## 🔧 技术细节

### 跟踪逻辑
```gdscript
func _physics_process(delta: float) -> void:
    if not isEnabled: return
    
    if shouldRepositionImmediately:
        parentEntity.global_position = parentEntity.get_global_mouse_position()
    else:
        parentEntity.global_position = parentEntity.global_position.move_toward(
            parentEntity.get_global_mouse_position(), 
            speed * delta
        )
```

### 性能考虑
- 使用`_physics_process()`而非`_input()`以获得delta时间
- 可通过`isEnabled`快速禁用处理
- 平滑移动使用`move_toward()`方法

## ⚠️ 注意事项

1. **性能影响**: 每帧都会更新位置，对性能有一定影响
2. **鼠标坐标**: 使用全局鼠标坐标，考虑摄像机变换
3. **物理处理**: 使用`_physics_process()`确保帧率一致性
4. **即时模式**: `shouldRepositionImmediately`为true时忽略speed参数
5. **坐标系统**: 确保实体和鼠标使用相同的坐标系统

## 🔗 相关组件

- [MouseRotationComponent](MouseRotationComponent.md) - 鼠标旋转组件
- [PositionControlComponent](PositionControlComponent.md) - 位置控制组件
- [TileBasedMouseControlComponent](TileBasedMouseControlComponent.md) - 瓦片鼠标控制
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 