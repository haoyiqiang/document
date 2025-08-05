# TileCollisionComponent API

> **继承关系**: Component > AreaComponentBase > TileCollisionComponent  
> **物理类型**: 瓦片碰撞检测

监控Area2D与TileMapLayer的碰撞，提供精确的瓦片格坐标检测。专门用于处理实体与瓦片地图的交互，支持格子级别的碰撞事件。

## ✨ 主要特性

- 🎯 精确的瓦片格坐标检测
- 🚪 进入/离开瓦片事件
- 🔧 可启用/禁用监控
- 📡 信号和虚拟方法双重接口
- ⚠️ 特殊处理被破坏的瓦片

## 📊 导出属性

### 监控设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `isEnabled` | `bool` | `true` | 是否启用碰撞监控，影响Area2D的monitoring和monitorable |

## 🔔 信号事件

### 瓦片碰撞信号
```gdscript
signal didEnterTileCell(map: TileMapLayer, cellCoordinates: Vector2i)
signal didExitTileCell(map: TileMapLayer, cellCoordinates: Vector2i)
```

| 信号 | 参数 | 描述 |
|------|------|------|
| `didEnterTileCell` | `map`, `cellCoordinates` | 进入瓦片格时发射 |
| `didExitTileCell` | `map`, `cellCoordinates` | 离开瓦片格时发射 |

## 🎯 虚拟方法

### 可重写方法
```gdscript
func onCollideCell(map: TileMapLayer, cellCoordinates: Vector2i) -> void
func onExitCell(map: TileMapLayer, cellCoordinates: Vector2i) -> void
```

| 方法 | 调用时机 | 描述 |
|------|----------|------|
| `onCollideCell` | 进入瓦片前 | 在信号发射前调用 |
| `onExitCell` | 离开瓦片前 | 在信号发射前调用，被破坏瓦片坐标为(-1,-1) |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `Area2D` | **基类依赖** | 提供碰撞检测区域 |

### TileMapLayer设置要求
- **monitoring**: 必须启用
- **monitorable**: 必须启用  
- **physics_quadrant_size**: 必须设置为1 (Godot 4.5.dev1)

## 🎯 使用示例

### 基础瓦片检测

```gdscript
# Entity Scene Structure:
# └── Entity (Area2D)
#     ├── CollisionShape2D
#     └── TileCollisionComponent

# 连接信号
func _ready():
    var tileCollision = $TileCollisionComponent
    tileCollision.didEnterTileCell.connect(_on_enter_tile)
    tileCollision.didExitTileCell.connect(_on_exit_tile)

func _on_enter_tile(map: TileMapLayer, coords: Vector2i):
    print("Entered tile at: ", coords)

func _on_exit_tile(map: TileMapLayer, coords: Vector2i):
    if coords == Vector2i(-1, -1):
        print("Exited destroyed tile")
    else:
        print("Exited tile at: ", coords)
```

### 自定义瓦片响应组件

```gdscript
# CustomTileResponder.gd
extends TileCollisionComponent

@export var damagePerTile: float = 10.0
@export var collectibleTileTypes: Array[int] = []

func onCollideCell(map: TileMapLayer, cellCoordinates: Vector2i):
    super.onCollideCell(map, cellCoordinates)
    
    var tileData = map.get_cell_tile_data(cellCoordinates)
    if tileData:
        handleTileInteraction(map, cellCoordinates, tileData)

func handleTileInteraction(map: TileMapLayer, coords: Vector2i, tileData: TileData):
    # 检查瓦片类型
    var tileType = tileData.get_custom_data("type")
    
    match tileType:
        "damage":
            applyDamage()
        "collectible":
            collectTile(map, coords)
        "teleporter":
            handleTeleport(tileData)

func applyDamage():
    var healthComponent = parentEntity.findFirstComponentSubclass(HealthComponent)
    if healthComponent:
        healthComponent.takeDamage(damagePerTile)

func collectTile(map: TileMapLayer, coords: Vector2i):
    # 收集瓦片并移除
    map.set_cell(coords, -1)  # 移除瓦片
    
    var inventoryComponent = parentEntity.findFirstComponentSubclass(InventoryComponent)
    if inventoryComponent:
        # 添加到背包逻辑
        pass

func handleTeleport(tileData: TileData):
    var teleportTarget = tileData.get_custom_data("teleport_target")
    if teleportTarget:
        parentEntity.global_position = teleportTarget
```

### 瓦片状态跟踪组件

```gdscript
# TileStateTracker.gd
extends TileCollisionComponent

var occupiedTiles: Dictionary = {}  # Vector2i -> TileMapLayer
var tileHistory: Array[Vector2i] = []

func onCollideCell(map: TileMapLayer, cellCoordinates: Vector2i):
    super.onCollideCell(map, cellCoordinates)
    
    # 记录占用的瓦片
    occupiedTiles[cellCoordinates] = map
    tileHistory.append(cellCoordinates)
    
    # 限制历史记录长度
    if tileHistory.size() > 50:
        tileHistory = tileHistory.slice(-50)
    
    # 更新瓦片状态
    updateTileState(map, cellCoordinates, true)

func onExitCell(map: TileMapLayer, cellCoordinates: Vector2i):
    super.onExitCell(map, cellCoordinates)
    
    if cellCoordinates != Vector2i(-1, -1):  # 非被破坏瓦片
        occupiedTiles.erase(cellCoordinates)
        updateTileState(map, cellCoordinates, false)

func updateTileState(map: TileMapLayer, coords: Vector2i, occupied: bool):
    # 更新瓦片的占用状态（如果瓦片支持）
    var tileData = map.get_cell_tile_data(coords)
    if tileData and tileData.has_custom_data("occupied"):
        # 注意：TileData是只读的，这里只是示例
        print("Tile ", coords, " occupied: ", occupied)

func getCurrentTiles() -> Array[Vector2i]:
    return occupiedTiles.keys()

func isOccupyingTile(coords: Vector2i) -> bool:
    return coords in occupiedTiles

func getLastTile() -> Vector2i:
    return tileHistory[-1] if tileHistory.size() > 0 else Vector2i(-1, -1)
```

### 瓦片触发器组件

```gdscript
# TileTriggerComponent.gd
extends TileCollisionComponent

@export var triggerTileTypes: Array[String] = ["switch", "button", "pressure_plate"]
@export var oneTimeOnly: bool = false

var triggeredTiles: Array[Vector2i] = []

func onCollideCell(map: TileMapLayer, cellCoordinates: Vector2i):
    super.onCollideCell(map, cellCoordinates)
    
    if oneTimeOnly and cellCoordinates in triggeredTiles:
        return
    
    var tileData = map.get_cell_tile_data(cellCoordinates)
    if tileData:
        var tileType = tileData.get_custom_data("type")
        
        if tileType in triggerTileTypes:
            activateTrigger(map, cellCoordinates, tileData)
            
            if oneTimeOnly:
                triggeredTiles.append(cellCoordinates)

func activateTrigger(map: TileMapLayer, coords: Vector2i, tileData: TileData):
    var triggerID = tileData.get_custom_data("trigger_id")
    if triggerID:
        # 发送触发信号
        GameState.triggerActivated.emit(triggerID, coords)
        
        # 视觉反馈
        showTriggerEffect(coords)

func showTriggerEffect(coords: Vector2i):
    # 添加触发特效
    var worldPos = map.map_to_local(coords)
    # 创建粒子效果或音效
    print("Triggered tile at: ", coords)
```

## 🔧 技术细节

### 碰撞检测流程
```gdscript
func onBodyShapeEntered(bodyRID: RID, bodyEntered: Node2D, bodyShapeIndex: int, localShapeIndex: int):
    if bodyEntered is TileMapLayer:
        var cellCoordinates = bodyEntered.get_coords_for_body_rid(bodyRID)
        onCollideCell(bodyEntered, cellCoordinates)  # 先调用虚拟方法
        didEnterTileCell.emit(bodyEntered, cellCoordinates)  # 后发射信号
```

### 被破坏瓦片处理
- 当瓦片被销毁时，`get_coords_for_body_rid()`返回`Vector2i(-1, -1)`
- 退出信号依然会发射，但坐标为特殊值
- 可通过检查坐标是否为(-1, -1)来判断瓦片是否被破坏

### Godot版本兼容性
- **Godot 4.5.dev1**: 需要设置`physics_quadrant_size = 1`
- **Area2D设置**: 必须启用monitoring和monitorable

## ⚠️ 注意事项

1. **TileMapLayer设置**: 确保physics_quadrant_size设置为1
2. **Area2D标志**: monitoring和monitorable必须都启用
3. **坐标系统**: 使用瓦片坐标而非世界坐标
4. **性能考虑**: 大量瓦片交互时注意性能影响
5. **事件顺序**: 虚拟方法在信号之前调用

## 🔗 相关组件

- [AreaComponentBase](AreaComponentBase.md) - Area组件基类
- [TileBasedPositionComponent](../Movement/TileBasedPositionComponent.md) - 瓦片位置组件
- [ModifyOnCollisionComponent](ModifyOnCollisionComponent.md) - 碰撞修改组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 