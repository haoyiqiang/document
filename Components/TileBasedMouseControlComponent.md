# TileBasedMouseControlComponent API

> **继承关系**: Component > TileBasedMouseControlComponent  
> **控制类型**: 瓦片网格鼠标控制  
> **依赖组件**: TileBasedPositionComponent

将TileBasedPositionComponent的坐标吸附到鼠标指针位置的组件。主要用于显示选择光标、建筑预览、网格对齐等需要在瓦片地图上精确定位的功能。

## ✨ 主要特性

- 🗂️ 瓦片网格精确对齐
- 🖱️ 实时鼠标位置跟踪
- ⚡ 性能优化的更新策略
- 🎯 自动坐标转换
- 📱 响应式处理模式

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `TileBasedPositionComponent` | **同级** | 提供瓦片坐标管理功能 |

## 📊 导出属性

### 处理模式
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldProcessEveryFrame` | `bool` | `false` | 是否每帧更新位置 |
| `isEnabled` | `bool` | `true` | 是否启用鼠标控制 |

### 内部引用
| 属性 | 类型 | 描述 |
|------|------|------|
| `tileBasedPositionComponent` | `TileBasedPositionComponent` | 瓦片位置组件引用 |

## 🎯 使用示例

### 基础建筑光标

```gdscript
# Entity Scene Structure:
# └── BuildingCursor (Node2D)
#     ├── Sprite2D (建筑预览)
#     ├── TileBasedPositionComponent
#     │   tileMap: [TileMapLayer引用]
#     └── TileBasedMouseControlComponent
#         shouldProcessEveryFrame: false  # 性能优化
#         isEnabled: true
```

### 选择工具光标

```gdscript
# Entity Scene Structure:
# └── SelectionCursor (Node2D)
#     ├── Sprite2D (选择框)
#     ├── TileBasedPositionComponent
#     │   tileMap: [TileMapLayer引用]
#     └── TileBasedMouseControlComponent
#         shouldProcessEveryFrame: true   # 实时更新
#         isEnabled: true
```

### 智能建筑光标

```gdscript
# SmartBuildingCursor.gd
extends TileBasedMouseControlComponent

@export var buildingType: String = "wall"
@export var showValidityIndicator: bool = true
@export var snapToValidTiles: bool = true
@export var highlightValidTiles: bool = false

var validTilePositions: Array[Vector2i] = []
var currentValidity: bool = true

func _ready():
    super._ready()
    if highlightValidTiles:
        updateValidTileHighlights()

func updatePosition():
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    var mousePosition = tileMap.local_to_map(tileMap.get_local_mouse_position())
    
    # 检查是否需要吸附到有效瓦片
    if snapToValidTiles:
        mousePosition = findNearestValidTile(mousePosition)
    
    # 更新有效性状态
    currentValidity = isValidBuildingPosition(mousePosition)
    
    # 更新位置
    tileBasedPositionComponent.currentCellCoordinates = mousePosition
    tileBasedPositionComponent.snapEntityPositionToTile(mousePosition)
    
    # 更新视觉反馈
    updateVisualFeedback()

func findNearestValidTile(position: Vector2i) -> Vector2i:
    if not snapToValidTiles or validTilePositions.is_empty():
        return position
    
    var nearestTile = position
    var minDistance = INF
    
    for validPos in validTilePositions:
        var distance = position.distance_squared_to(validPos)
        if distance < minDistance:
            minDistance = distance
            nearestTile = validPos
    
    # 只有在合理距离内才吸附
    if sqrt(minDistance) <= 3.0:
        return nearestTile
    
    return position

func isValidBuildingPosition(position: Vector2i) -> bool:
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    
    # 检查是否在地图范围内
    if not isWithinMapBounds(position):
        return false
    
    # 检查瓦片是否被占用
    if isTileOccupied(position):
        return false
    
    # 检查建筑类型特定的规则
    return checkBuildingSpecificRules(position)

func isWithinMapBounds(position: Vector2i) -> bool:
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    var mapRect = tileMap.get_used_rect()
    return mapRect.has_point(position)

func isTileOccupied(position: Vector2i) -> bool:
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    var tileData = tileMap.get_cell_tile_data(position)
    
    # 检查是否有瓦片数据以及是否标记为可建造
    return tileData != null and tileData.get_custom_data("occupied")

func checkBuildingSpecificRules(position: Vector2i) -> bool:
    match buildingType:
        "wall":
            return checkWallPlacement(position)
        "tower":
            return checkTowerPlacement(position)
        "resource":
            return checkResourcePlacement(position)
    
    return true

func checkWallPlacement(position: Vector2i) -> bool:
    # 墙可以建在大部分地方，但不能阻塞关键路径
    return not isBlockingCriticalPath(position)

func checkTowerPlacement(position: Vector2i) -> bool:
    # 塔需要一定的间距
    return not hasNearbyTower(position, 3)

func checkResourcePlacement(position: Vector2i) -> bool:
    # 资源建筑需要在特定地形上
    return isResourceTerrain(position)

func isBlockingCriticalPath(position: Vector2i) -> bool:
    # 检查是否阻塞了重要路径
    # 这里需要实现路径检查逻辑
    return false

func hasNearbyTower(position: Vector2i, radius: int) -> bool:
    # 检查附近是否有塔
    for x in range(-radius, radius + 1):
        for y in range(-radius, radius + 1):
            var checkPos = position + Vector2i(x, y)
            if isTowerAt(checkPos):
                return true
    return false

func isTowerAt(position: Vector2i) -> bool:
    # 检查指定位置是否有塔
    # 这里需要实现建筑检查逻辑
    return false

func isResourceTerrain(position: Vector2i) -> bool:
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    var tileData = tileMap.get_cell_tile_data(position)
    
    if not tileData:
        return false
    
    return tileData.get_custom_data("terrain_type") == "resource"

func updateVisualFeedback():
    var sprite = parentEntity.get_node("Sprite2D") as Sprite2D
    if not sprite:
        return
    
    # 根据有效性改变颜色
    if currentValidity:
        sprite.modulate = Color.GREEN
    else:
        sprite.modulate = Color.RED

func updateValidTileHighlights():
    # 更新有效瓦片的高亮显示
    validTilePositions.clear()
    
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    var mapRect = tileMap.get_used_rect()
    
    for x in range(mapRect.position.x, mapRect.position.x + mapRect.size.x):
        for y in range(mapRect.position.y, mapRect.position.y + mapRect.size.y):
            var pos = Vector2i(x, y)
            if isValidBuildingPosition(pos):
                validTilePositions.append(pos)

func setBuildingType(newType: String):
    buildingType = newType
    if highlightValidTiles:
        updateValidTileHighlights()
```

### 区域选择组件

```gdscript
# AreaSelectionComponent.gd
extends TileBasedMouseControlComponent

@export var enableAreaSelection: bool = true
@export var maxSelectionSize: Vector2i = Vector2i(10, 10)
@export var minSelectionSize: Vector2i = Vector2i(1, 1)

var isSelecting: bool = false
var selectionStart: Vector2i
var selectionEnd: Vector2i
var selectedArea: Rect2i

signal area_selected(area: Rect2i)
signal selection_cancelled()

func _input(event: InputEvent):
    if not isEnabled:
        return
    
    if event is InputEventMouseButton:
        var mouseEvent = event as InputEventMouseButton
        
        if mouseEvent.button_index == MOUSE_BUTTON_LEFT:
            if mouseEvent.pressed:
                startAreaSelection()
            else:
                finishAreaSelection()
        elif mouseEvent.button_index == MOUSE_BUTTON_RIGHT and mouseEvent.pressed:
            cancelAreaSelection()

func startAreaSelection():
    if not enableAreaSelection:
        return
    
    isSelecting = true
    selectionStart = tileBasedPositionComponent.currentCellCoordinates
    selectionEnd = selectionStart
    updateSelectionArea()

func updatePosition():
    super.updatePosition()
    
    if isSelecting:
        selectionEnd = tileBasedPositionComponent.currentCellCoordinates
        updateSelectionArea()

func updateSelectionArea():
    if not isSelecting:
        return
    
    var minPos = Vector2i(
        min(selectionStart.x, selectionEnd.x),
        min(selectionStart.y, selectionEnd.y)
    )
    var maxPos = Vector2i(
        max(selectionStart.x, selectionEnd.x),
        max(selectionStart.y, selectionEnd.y)
    )
    
    var size = maxPos - minPos + Vector2i.ONE
    
    # 限制选择区域大小
    size.x = min(size.x, maxSelectionSize.x)
    size.y = min(size.y, maxSelectionSize.y)
    
    selectedArea = Rect2i(minPos, size)
    
    # 更新视觉反馈
    updateSelectionVisual()

func finishAreaSelection():
    if not isSelecting:
        return
    
    isSelecting = false
    
    # 检查选择区域是否有效
    if selectedArea.size.x >= minSelectionSize.x and selectedArea.size.y >= minSelectionSize.y:
        area_selected.emit(selectedArea)
    else:
        cancelAreaSelection()

func cancelAreaSelection():
    isSelecting = false
    selectedArea = Rect2i()
    selection_cancelled.emit()
    updateSelectionVisual()

func updateSelectionVisual():
    # 这里可以添加选择区域的视觉反馈
    print("Selection area: ", selectedArea)

func getSelectedTiles() -> Array[Vector2i]:
    var tiles: Array[Vector2i] = []
    
    for x in range(selectedArea.position.x, selectedArea.position.x + selectedArea.size.x):
        for y in range(selectedArea.position.y, selectedArea.position.y + selectedArea.size.y):
            tiles.append(Vector2i(x, y))
    
    return tiles

func isPointInSelection(point: Vector2i) -> bool:
    return selectedArea.has_point(point)
```

### 路径预览组件

```gdscript
# PathPreviewComponent.gd
extends TileBasedMouseControlComponent

@export var showPathPreview: bool = true
@export var pathStartPosition: Vector2i = Vector2i.ZERO
@export var maxPathLength: int = 50

var currentPath: Array[Vector2i] = []
var pathfinder: AStar2D

signal path_updated(path: Array[Vector2i])

func _ready():
    super._ready()
    setupPathfinder()

func setupPathfinder():
    pathfinder = AStar2D.new()
    # 这里需要根据瓦片地图设置寻路网格
    updatePathfinder()

func updatePosition():
    super.updatePosition()
    
    if showPathPreview:
        updatePathPreview()

func updatePathPreview():
    var targetPosition = tileBasedPositionComponent.currentCellCoordinates
    
    if pathStartPosition == targetPosition:
        currentPath.clear()
        path_updated.emit(currentPath)
        return
    
    # 计算路径
    var newPath = findPath(pathStartPosition, targetPosition)
    
    if newPath != currentPath:
        currentPath = newPath
        path_updated.emit(currentPath)

func findPath(start: Vector2i, end: Vector2i) -> Array[Vector2i]:
    if not pathfinder:
        return []
    
    var startId = getPointId(start)
    var endId = getPointId(end)
    
    if not pathfinder.has_point(startId) or not pathfinder.has_point(endId):
        return []
    
    var pathPoints = pathfinder.get_point_path(startId, endId)
    var pathTiles: Array[Vector2i] = []
    
    for point in pathPoints:
        pathTiles.append(Vector2i(int(point.x), int(point.y)))
    
    # 限制路径长度
    if pathTiles.size() > maxPathLength:
        pathTiles = pathTiles.slice(0, maxPathLength)
    
    return pathTiles

func updatePathfinder():
    if not tileBasedPositionComponent or not tileBasedPositionComponent.tileMap:
        return
    
    pathfinder.clear()
    var tileMap = tileBasedPositionComponent.tileMap
    var mapRect = tileMap.get_used_rect()
    
    # 添加所有可通行的瓦片点
    for x in range(mapRect.position.x, mapRect.position.x + mapRect.size.x):
        for y in range(mapRect.position.y, mapRect.position.y + mapRect.size.y):
            var pos = Vector2i(x, y)
            if isWalkable(pos):
                var pointId = getPointId(pos)
                pathfinder.add_point(pointId, Vector2(x, y))
    
    # 连接相邻的可通行点
    for x in range(mapRect.position.x, mapRect.position.x + mapRect.size.x):
        for y in range(mapRect.position.y, mapRect.position.y + mapRect.size.y):
            var pos = Vector2i(x, y)
            if isWalkable(pos):
                connectNeighbors(pos)

func isWalkable(position: Vector2i) -> bool:
    var tileMap = tileBasedPositionComponent.tileMap
    var tileData = tileMap.get_cell_tile_data(position)
    
    if not tileData:
        return false
    
    return not tileData.get_custom_data("blocked")

func connectNeighbors(position: Vector2i):
    var pointId = getPointId(position)
    var neighbors = [
        position + Vector2i(1, 0),
        position + Vector2i(-1, 0),
        position + Vector2i(0, 1),
        position + Vector2i(0, -1)
    ]
    
    for neighbor in neighbors:
        if isWalkable(neighbor):
            var neighborId = getPointId(neighbor)
            if pathfinder.has_point(neighborId):
                pathfinder.connect_points(pointId, neighborId)

func getPointId(position: Vector2i) -> int:
    # 将2D坐标转换为唯一的点ID
    return position.y * 1000 + position.x

func setPathStart(newStart: Vector2i):
    pathStartPosition = newStart
    if showPathPreview:
        updatePathPreview()

func getPathLength() -> int:
    return currentPath.size()

func isPathValid() -> bool:
    return not currentPath.is_empty()
```

## 🔧 技术细节

### 位置更新逻辑
```gdscript
func updatePosition() -> void:
    var tileMap: TileMapLayer = tileBasedPositionComponent.tileMap
    tileBasedPositionComponent.currentCellCoordinates = tileMap.local_to_map(tileMap.get_local_mouse_position())
    tileBasedPositionComponent.snapEntityPositionToTile(tileBasedPositionComponent.currentCellCoordinates)
```

### 性能优化策略
- **默认模式**: 仅在鼠标移动时更新（`shouldProcessEveryFrame = false`）
- **实时模式**: 每帧更新位置（`shouldProcessEveryFrame = true`）
- **动态启用**: 通过`isEnabled`控制处理

## ⚠️ 注意事项

1. **依赖要求**: 必须在同一实体上具有TileBasedPositionComponent
2. **TileMapLayer**: 确保TileBasedPositionComponent正确配置了瓦片地图
3. **性能考虑**: 根据需要选择合适的处理模式
4. **坐标转换**: 自动处理屏幕坐标到瓦片坐标的转换
5. **边界检查**: 不会自动限制在地图边界内

## 🔗 相关组件

- [TileBasedPositionComponent](./TileBasedPositionComponent.md) - 瓦片位置组件
- [MouseTrackingComponent](MouseTrackingComponent.md) - 鼠标跟踪组件
- [TileBasedControlComponent](TileBasedControlComponent.md) - 瓦片控制组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 