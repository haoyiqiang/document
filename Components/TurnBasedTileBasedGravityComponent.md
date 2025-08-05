# TurnBasedTileBasedGravityComponent API

> **继承关系**: Component > TurnBasedComponent > TurnBasedTileBasedGravityComponent  
> **游戏类型**: 回合制瓦片游戏  
> **实验性功能**

在回合制游戏中模拟基于瓦片的伪重力系统。使用Timer自动推进回合，当实体悬空时自动下落到下方的瓦片位置。

## ✨ 主要特性

- 🎯 回合制瓦片重力模拟
- ⏰ 自动回合推进系统
- 📍 精确的瓦片坐标验证
- 🔄 可配置的处理阶段
- 📡 下落事件信号

## 📊 导出属性

### 重力设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `phaseToProcessIn` | `TurnBasedCoordinator.TurnBasedState` | `turnBegin` | 处理重力的回合阶段 |

## 🔔 信号事件

### 重力信号
```gdscript
signal didFall
```

| 信号 | 描述 |
|------|------|
| `didFall` | 实体开始下落时发射 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `TurnBasedEntity` | **实体类型** | 回合制实体 |
| `TileBasedPositionComponent` | **必需** | 瓦片位置管理 |

### 子节点
| 节点 | 类型 | 描述 |
|------|------|------|
| `GravityTimer` | `Timer` | 重力检测计时器 |

## 🎯 使用示例

### 基础重力设置

```gdscript
# Entity Scene Structure:
# └── TurnBasedEntity
#     ├── TileBasedPositionComponent
#     └── TurnBasedTileBasedGravityComponent
#         └── GravityTimer
```

### 自定义重力组件

```gdscript
# CustomGravityComponent.gd
extends TurnBasedTileBasedGravityComponent

@export var fallSpeed: int = 1
@export var enableCoyoteTime: bool = true

var framesInAir: int = 0

func checkForFall() -> bool:
    var shouldFall = super.checkForFall()
    
    if shouldFall:
        framesInAir += 1
        if enableCoyoteTime and framesInAir <= 2:
            return false
    else:
        framesInAir = 0
    
    return shouldFall

func fall():
    var coordinates = tileBasedPositionComponent.destinationCellCoordinates
    coordinates.y += fallSpeed
    tileBasedPositionComponent.setDestinationCellCoordinates(coordinates)
    didFall.emit()
```

### 重力监听器

```gdscript
# GravityListener.gd
extends Node

func _ready():
    var gravityComponent = get_parent().findFirstComponentSubclass(TurnBasedTileBasedGravityComponent)
    if gravityComponent:
        gravityComponent.didFall.connect(_on_entity_fall)

func _on_entity_fall():
    print("Entity is falling!")
    # 播放下落音效或动画
    playFallAnimation()

func playFallAnimation():
    # 添加下落动画逻辑
    pass
```

## 🔧 技术细节

### 重力检测逻辑
```gdscript
func checkForFall() -> bool:
    var currentPosition = tileBasedPositionComponent.currentCellCoordinates
    var cellBelow = Vector2i(currentPosition.x, currentPosition.y + 1)
    return tileBasedPositionComponent.validateCoordinates(cellBelow)
```

### 回合阶段处理
```gdscript
func processTurnBegin():
    if phaseToProcessIn == TurnBasedCoordinator.TurnBasedState.turnBegin and checkForFall():
        fall()

func processTurnEnd():
    if phaseToProcessIn == TurnBasedCoordinator.TurnBasedState.turnEnd and checkForFall():
        fall()
    startTimer()
```

### 下落实现
```gdscript
func fall():
    var coordinates = tileBasedPositionComponent.destinationCellCoordinates
    coordinates.y += 1
    tileBasedPositionComponent.setDestinationCellCoordinates(coordinates)
    didFall.emit()
```

## ⚠️ 注意事项

1. **实验性功能**: 组件标记为实验性，API可能变化
2. **回合制依赖**: 需要TurnBasedCoordinator和TurnBasedEntity
3. **瓦片验证**: 依赖TileBasedPositionComponent的坐标验证
4. **跳跃冲突**: 当前不支持跳跃时的重力暂停
5. **处理阶段**: 确保选择合适的回合处理阶段

## 🔗 相关组件

- [TurnBasedComponent](TurnBasedComponent.md) - 回合制基础组件
- [TileBasedPositionComponent](../Movement/TileBasedPositionComponent.md) - 瓦片位置组件
- [TurnBasedTileBasedControlComponent](TurnBasedTileBasedControlComponent.md) - 回合制瓦片控制
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 