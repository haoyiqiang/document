# TileBasedPositionComponent API

## 概述
`TileBasedPositionComponent` 是一个复杂的瓦片位置管理组件，将实体位置设置为关联TileMapLayer中瓦片的位置。它不处理玩家输入或寻路，专注于瓦片位置的精确管理、边界检查和瓦片占用状态。

**继承链**：`Component` → `TileBasedPositionComponent`

## 依赖要求
- **TileMapLayerWithCellData** 或 **TileMapLayer + TileMapCellData**

## 主要特性
- 🗺️ **瓦片精确定位**：将实体位置锁定到瓦片网格
- 📍 **占用状态管理**：标记和管理瓦片占用状态
- 🔄 **平滑移动**：支持瞬间移动或动画过渡
- 🛡️ **边界检查**：确保移动在瓦片地图范围内
- ⚡ **性能优化**：按需启用物理处理
- 🎯 **视觉指示器**：可选的目标位置指示

## 导出属性

### 瓦片地图配置
- **tileMap** (`TileMapLayer`)：关联的瓦片地图层
- **tileMapData** (`TileMapCellData`)：瓦片数据管理器
- **shouldSearchForTileMap** (`bool = false`)：自动搜索场景中的瓦片地图

### 初始位置
- **setInitialCoordinatesFromEntityPosition** (`bool = false`)：从实体位置设置初始坐标
- **initialDestinationCoordinates** (`Vector2i`)：初始目标坐标
- **shouldSnapToInitialDestination** (`bool = true`)：是否立即移动到初始位置

### 移动控制
- **speed** (`float = 200.0`, 范围:10~1000)：瓦片间移动速度
- **shouldMoveInstantly** (`bool = false`)：是否瞬间移动
- **shouldClampToBounds** (`bool = true`)：限制在地图边界内
- **shouldOccupyCell** (`bool = true`)：是否标记瓦片为占用状态
- **shouldSnapPositionEveryFrame** (`bool = false`)：每帧对齐位置
- **visualIndicator** (`Node2D`)：目标位置视觉指示器

### 控制选项
- **isEnabled** (`bool = true`)：是否启用组件功能

## 状态属性

### 坐标状态
- **currentCellCoordinates** (`Vector2i`)：当前瓦片坐标
- **destinationCellCoordinates** (`Vector2i`)：目标瓦片坐标
- **inputVector** (`Vector2i`)：输入移动向量
- **isMovingToNewCell** (`bool`)：是否正在移动到新瓦片

## 信号

### 移动事件
- **willStartMovingToNewCell(newDestination: Vector2i)**：开始移动到新瓦片
- **didArriveAtNewCell(newDestination: Vector2i)**：到达新瓦片

### 地图事件
- **willSetNewMap(...)**：即将设置新地图
- **didSetNewMap(...)**：已设置新地图

## 主要方法

### 位置管理
- **validateTileMap()** → `bool`：验证瓦片地图配置
- **snapEntityPositionToTile()** → `void`：对齐实体到瓦片位置
- **vacateCurrentCell()** → `void`：清空当前瓦片占用状态

## 使用示例

### 基础瓦片定位
```gdscript
# 设置简单的瓦片位置管理
@export var tile_position: TileBasedPositionComponent
@export var tile_map: TileMapLayer

func _ready():
    tile_position.tileMap = tile_map
    tile_position.shouldMoveInstantly = false
    tile_position.speed = 150.0
```

### 回合制角色移动
```gdscript
# 回合制游戏的角色定位
@export var character_tile_pos: TileBasedPositionComponent

func move_character_to_tile(target_coords: Vector2i):
    if character_tile_pos.validateTileMap():
        character_tile_pos.destinationCellCoordinates = target_coords
```

### 网格建造系统
```gdscript
# 建筑物的网格放置系统
@export var building_position: TileBasedPositionComponent
@export var cursor_indicator: Sprite2D

func setup_building_placement():
    building_position.visualIndicator = cursor_indicator
    building_position.shouldOccupyCell = true  # 建筑占用瓦片
    building_position.shouldSnapPositionEveryFrame = true  # 实时对齐
```

## 技术细节

### 瓦片占用管理
- 使用`TileMapCellData`标记瓦片为已占用
- 支持动态添加/移除占用状态
- 移动时自动更新占用信息

### 性能优化策略
- 按需启用`_physics_process()`
- 仅在移动或需要实时对齐时处理
- 避免不必要的瓦片位置计算

## 注意事项

### 复杂性警告
- 此组件功能复杂，需要仔细配置
- 建议先理解TileMapLayer和TileMapCellData
- 适合需要精确瓦片控制的游戏

### 性能考虑
- `shouldSnapPositionEveryFrame`仅在必要时启用
- 大量瓦片实体可能影响性能
- 合理设置移动速度避免卡顿

## 相关组件
- [`TileBasedControlComponent`](../Control/TileBasedControlComponent.md) - 瓦片输入控制
- [`NavigationComponent`](../Movement/NavigationComponent.md) - 导航寻路
- [`LinearMotionComponent`](../Movement/LinearMotionComponent.md) - 直线运动 