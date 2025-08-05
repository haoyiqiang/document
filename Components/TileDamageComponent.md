# TileDamageComponent API 参考

## 概述

`TileDamageComponent` 是 [`DamageComponent`](DamageComponent.md) 的子类，专门用于对破坏性 `TileMapLayer` 单元格造成伤害。该组件通过调用 `Tools.damageTileMapCell` 方法来改变瓦片地图单元格的瓦片，具体行为取决于瓦片的自定义数据层设置。

**继承关系：**
`Component` → `TileCollisionComponent` → `TileDamageComponent`

## 主要特性

- 🧱 **瓦片破坏系统** - 基于瓦片自定义数据的破坏机制
- 🎯 **精确碰撞检测** - 获取准确的瓦片坐标进行处理
- ⚠️ **实验性功能** - 可能在未来版本中发生变化
- 🔧 **自动清理** - 支持瓦片删除和替换

## 导出属性

该组件继承父类的所有属性，无额外导出属性。

## 重要要求

⚠️ **重要限制：** 
- `TileMapLayer.physics_quadrant_size` 必须设置为 1 才能正确获取单元格坐标
- 瓦片集必须设置以下自定义数据层：
  - `isDestructible` - 标记瓦片是否可破坏
  - `nextTileOnDamage` - 指定破坏后的替换瓦片

## 主要方法

### onBodyShapeEntered()
```gdscript
func onBodyShapeEntered(bodyRID: RID, bodyEntered: Node2D, bodyShapeIndex: int, localShapeIndex: int) -> void
```
处理与 `TileMapLayer` 的碰撞，获取单元格坐标并应用破坏效果。

### onBodyShapeExited()
```gdscript
func onBodyShapeExited(bodyRID: RID, bodyExited: Node2D, bodyShapeIndex: int, localShapeIndex: int) -> void
```
重写父类方法，忽略离开瓦片单元格的事件。

## 工作机制

1. **碰撞检测** - 检测与 `TileMapLayer` 的物理碰撞
2. **坐标获取** - 使用 `get_coords_for_body_rid()` 获取精确单元格坐标
3. **瓦片处理** - 调用 `Tools.damageTileMapCell()` 处理瓦片破坏
4. **信号发射** - 发出 `didEnterTileCell` 信号通知碰撞事件

## 使用示例

### 基本设置
```gdscript
# 在弹药实体上添加瓦片破坏功能
extends BulletEntity

func _ready():
    var tileDamage = TileDamageComponent.new()
    add_child(tileDamage)
    
    # 确保TileMapLayer配置正确
    var tileMapLayer = get_node("../TileMapLayer")
    tileMapLayer.physics_quadrant_size = 1
```

### 瓦片集配置
```gdscript
# 在TileSet资源中设置自定义数据层
# 1. 添加布尔型数据层"isDestructible"
# 2. 添加Vector2i型数据层"nextTileOnDamage"
# 3. 为每个瓦片设置相应的属性值

# 示例：可破坏砖块
# isDestructible = true
# nextTileOnDamage = Vector2i(-1, -1)  # 表示删除瓦片
```

### 连锁破坏效果
```gdscript
# 监听瓦片破坏事件
func _ready():
    var tileDamage = $TileDamageComponent
    tileDamage.didEnterTileCell.connect(_on_tile_damaged)

func _on_tile_damaged(tileMapLayer: TileMapLayer, cellCoords: Vector2i):
    # 检查周围瓦片并触发连锁反应
    for offset in [Vector2i.UP, Vector2i.DOWN, Vector2i.LEFT, Vector2i.RIGHT]:
        var neighborCoords = cellCoords + offset
        if should_chain_damage(tileMapLayer, neighborCoords):
            Tools.damageTileMapCell(tileMapLayer, neighborCoords)
```

### 特殊破坏效果
```gdscript
# 创建破坏特效
func _on_tile_damaged(tileMapLayer: TileMapLayer, cellCoords: Vector2i):
    # 获取世界坐标
    var worldPos = tileMapLayer.to_global(tileMapLayer.map_to_local(cellCoords))
    
    # 创建粒子效果
    var particles = preload("res://Effects/TileDestroyParticles.tscn").instantiate()
    get_tree().current_scene.add_child(particles)
    particles.global_position = worldPos
    particles.emitting = true
    
    # 播放破坏音效
    AudioManager.play_sound("tile_break", worldPos)
```

## 技术细节

### 坐标系统
- 使用 `get_coords_for_body_rid()` 获取准确的瓦片坐标
- 坐标基于 `TileMapLayer` 的瓦片网格系统
- 支持与物理系统的精确集成

### 性能优化
- 仅在碰撞发生时进行瓦片处理
- 自动跳过非目标层的碰撞
- 高效的坐标计算算法

### 调试功能
- 启用 `debugMode` 时显示瓦片坐标
- 使用黄色文字气泡标记碰撞位置
- 输出详细的碰撞信息

## 注意事项

⚠️ **重要限制：**
- 当前版本仅支持固定伤害值
- 每个瓦片只有固定的生命值
- 需要 Godot 4.5.dev1 及以上版本
- `physics_quadrant_size` 必须设置为 1

🔮 **计划功能：**
- 可变伤害系统
- 瓦片生命值系统
- 接触数组管理
- 更灵活的破坏规则

## 相关组件

- [`DamageComponent`](DamageComponent.md) - 父类，提供基础伤害功能
- [`TileCollisionComponent`](./TileCollisionComponent.md) - 祖父类，提供瓦片碰撞检测
- [`AreaCollisionComponent`](./AreaCollisionComponent.md) - Area2D碰撞基类
- [`BulletModifierComponent`](BulletModifierComponent.md) - 可用于修改带有此组件的弹药

## 故障排除

**问题：无法获取瓦片坐标**
- 检查 `TileMapLayer.physics_quadrant_size` 是否为 1
- 确保瓦片图层有正确的物理碰撞形状

**问题：瓦片不会被破坏**
- 验证瓦片的 `isDestructible` 自定义数据为 true
- 检查 `nextTileOnDamage` 是否设置正确

**问题：性能问题**
- 避免在大量瓦片上同时触发破坏
- 考虑使用对象池管理破坏效果 