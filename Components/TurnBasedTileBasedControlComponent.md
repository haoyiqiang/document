# TurnBasedTileBasedControlComponent API 文档

## 概述
`TurnBasedTileBasedControlComponent` 是一个回合制瓦片移动控制组件，使用 `TileBasedPositionComponent` 根据玩家输入或随机逻辑在瓦片网格上移动实体。该组件是回合制游戏中基于瓦片的移动控制核心。

## 继承关系
```
Component
└── TurnBasedComponent
    └── TurnBasedTileBasedControlComponent
        └── TurnBasedTileBasedPlatformerControlComponent
```

## 主要功能
- **瓦片移动**: 基于瓦片网格的精确移动控制
- **回合制集成**: 完整的回合制游戏流程集成
- **输入处理**: 支持方向键输入和随机移动
- **移动验证**: 验证移动目标的有效性

## 导出属性

| 属性名 | 类型 | 默认值 | 描述 |
|--------|------|--------|------|
| `randomMovement` | `bool` | `false` | 是否启用随机移动（忽略玩家输入） |

## 公共属性

| 属性名 | 类型 | 描述 |
|--------|------|------|
| `recentInputVector` | `Vector2i` | 最近的输入向量 |

## 依赖组件

| 组件名 | 类型 | 必需 | 描述 |
|--------|------|------|------|
| `tileBasedPositionComponent` | `TileBasedPositionComponent` | 是 | 瓦片位置组件 |

## 必需组件
- `TurnBasedEntity` - 回合制实体
- `TileBasedPositionComponent` - 瓦片位置组件

## 主要方法

### validateMove() -> bool
验证移动操作的有效性。
- **返回**: 移动是否有效
- **功能**: 检查输入、回合状态和目标位置的有效性

### processTurnBegin() -> void
处理回合开始阶段。
- **功能**: 显示调试信息

### processTurnUpdate() -> void
处理回合更新阶段。
- **功能**: 执行移动逻辑，处理随机移动或玩家输入

### processTurnEnd() -> void
处理回合结束阶段。
- **功能**: 清理输入状态，显示调试信息

## 使用示例

### 基本设置
```gdscript
# 在回合制实体中添加TurnBasedTileBasedControlComponent
@onready var tile_control = $TurnBasedTileBasedControlComponent

func _ready():
    # 确保有必需的组件
    assert(has_component(TileBasedPositionComponent))
    assert(has_component(TurnBasedEntity))
    
    # 配置移动行为
    tile_control.randomMovement = false
    tile_control.isEnabled = true
```

### 随机移动AI
```gdscript
# 创建随机移动的AI实体
func setup_random_ai():
    var ai_entity = create_turn_based_entity()
    var tile_control = ai_entity.get_component(TurnBasedTileBasedControlComponent)
    
    # 启用随机移动
    tile_control.randomMovement = true
    tile_control.isEnabled = true
    
    print("AI实体将随机移动")
```

### 自定义移动验证
```gdscript
# 创建自定义移动验证的组件
class_name CustomTileControl
extends TurnBasedTileBasedControlComponent

@export var movement_cost: int = 1
@export var max_movement_per_turn: int = 1

var moves_this_turn: int = 0

func processTurnBegin() -> void:
    super.processTurnBegin()
    moves_this_turn = 0

func validateMove() -> bool:
    # 检查移动次数限制
    if moves_this_turn >= max_movement_per_turn:
        if debugMode:
            print("已达到本回合最大移动次数")
        return false
    
    # 检查移动成本
    var entity = parentEntity as TurnBasedEntity
    if entity.action_points < movement_cost:
        if debugMode:
            print("行动点不足")
        return false
    
    # 调用父类验证
    if super.validateMove():
        moves_this_turn += 1
        entity.action_points -= movement_cost
        return true
    
    return false
```

## 扩展示例

### 智能移动AI
```gdscript
# 创建智能移动AI
class_name SmartTileAI
extends TurnBasedTileBasedControlComponent

@export var target_entity: Entity
@export var preferred_distance: int = 3

func processTurnUpdate() -> void:
    if not target_entity:
        # 如果没有目标，使用随机移动
        self.recentInputVector = Vector2i([-1, 1].pick_random(), [-1, 1].pick_random())
    else:
        # 计算朝向目标的移动
        self.recentInputVector = calculate_smart_move()
    
    # 执行移动
    tileBasedPositionComponent.inputVector = Vector2i(self.recentInputVector)
    tileBasedPositionComponent.processMovementInput()
    showDebugInfo()

func calculate_smart_move() -> Vector2i:
    var current_pos = tileBasedPositionComponent.currentCellCoordinates
    var target_pos = target_entity.get_component(TileBasedPositionComponent).currentCellCoordinates
    
    var distance = current_pos.distance_to(target_pos)
    
    if distance > preferred_distance:
        # 靠近目标
        return (target_pos - current_pos).sign()
    elif distance < preferred_distance:
        # 远离目标
        return (current_pos - target_pos).sign()
    else:
        # 保持距离，可能移动到侧面
        return Vector2i([-1, 1].pick_random(), [-1, 1].pick_random())
```

### 移动模式切换
```gdscript
# 支持多种移动模式
class_name MultiModeMovement
extends TurnBasedTileBasedControlComponent

enum MovementMode {
    PLAYER_CONTROL,
    RANDOM_WALK,
    FOLLOW_TARGET,
    PATROL
}

@export var movement_mode: MovementMode = MovementMode.PLAYER_CONTROL
@export var patrol_points: Array[Vector2i] = []
@export var follow_target: Entity

var current_patrol_index: int = 0

func processTurnUpdate() -> void:
    match movement_mode:
        MovementMode.PLAYER_CONTROL:
            # 使用默认的玩家控制
            super.processTurnUpdate()
        MovementMode.RANDOM_WALK:
            randomMovement = true
            super.processTurnUpdate()
        MovementMode.FOLLOW_TARGET:
            move_toward_target()
        MovementMode.PATROL:
            move_on_patrol()

func move_toward_target():
    if not follow_target:
        return
    
    var target_pos = follow_target.get_component(TileBasedPositionComponent).currentCellCoordinates
    var current_pos = tileBasedPositionComponent.currentCellCoordinates
    
    self.recentInputVector = (target_pos - current_pos).sign()
    tileBasedPositionComponent.inputVector = self.recentInputVector
    tileBasedPositionComponent.processMovementInput()

func move_on_patrol():
    if patrol_points.is_empty():
        return
    
    var target_point = patrol_points[current_patrol_index]
    var current_pos = tileBasedPositionComponent.currentCellCoordinates
    
    if current_pos == target_point:
        # 到达巡逻点，切换到下一个
        current_patrol_index = (current_patrol_index + 1) % patrol_points.size()
        target_point = patrol_points[current_patrol_index]
    
    self.recentInputVector = (target_point - current_pos).sign()
    tileBasedPositionComponent.inputVector = self.recentInputVector
    tileBasedPositionComponent.processMovementInput()
```

### 移动限制和特殊能力
```gdscript
# 添加移动限制和特殊能力
class_name EnhancedTileControl
extends TurnBasedTileBasedControlComponent

@export var can_move_diagonally: bool = true
@export var can_pass_through_walls: bool = false
@export var teleport_distance: int = 0

func _input(event: InputEvent) -> void:
    if not isEnabled or randomMovement or not event.is_action_type():
        return
    
    # 处理特殊移动输入
    if event.is_action_pressed("teleport") and teleport_distance > 0:
        handle_teleport_input()
        return
    
    # 处理普通移动
    if event.is_action_pressed(GlobalInput.Actions.moveLeft) \
    or event.is_action_pressed(GlobalInput.Actions.moveRight) \
    or event.is_action_pressed(GlobalInput.Actions.moveUp) \
    or event.is_action_pressed(GlobalInput.Actions.moveDown):
        
        var input_vector = Input.get_vector(
            GlobalInput.Actions.moveLeft, 
            GlobalInput.Actions.moveRight, 
            GlobalInput.Actions.moveUp, 
            GlobalInput.Actions.moveDown
        )
        
        # 限制对角移动
        if not can_move_diagonally and abs(input_vector.x) > 0 and abs(input_vector.y) > 0:
            # 优先水平移动
            if abs(input_vector.x) > abs(input_vector.y):
                input_vector.y = 0
            else:
                input_vector.x = 0
        
        self.recentInputVector = input_vector
        validateMove()

func handle_teleport_input():
    var mouse_pos = get_global_mouse_position()
    var target_cell = tileBasedPositionComponent.worldToCell(mouse_pos)
    var current_cell = tileBasedPositionComponent.currentCellCoordinates
    
    if current_cell.distance_to(target_cell) <= teleport_distance:
        if can_pass_through_walls or tileBasedPositionComponent.validateCoordinates(target_cell):
            tileBasedPositionComponent.setPosition(target_cell)
            TurnBasedCoordinator.startTurnProcess()
```

## 技术细节

### 输入处理
- 监听移动相关的输入事件
- 支持四方向移动（上下左右）
- 可以通过设置 `randomMovement` 忽略玩家输入

### 回合制集成
- 实现完整的回合制处理流程
- 在回合开始时显示调试信息
- 在回合更新时执行移动逻辑
- 在回合结束时清理状态

### 移动验证
- 检查输入向量的有效性
- 验证 `TurnBasedCoordinator` 的准备状态
- 通过 `TileBasedPositionComponent` 验证目标位置

## 注意事项

1. **实验性功能**: 标记为实验性功能，API可能会发生变化
2. **依赖要求**: 必须有 `TurnBasedEntity` 和 `TileBasedPositionComponent`
3. **回合制约束**: 移动受回合制系统的约束
4. **碰撞检测**: 可能需要添加碰撞检测逻辑

## 相关组件
- `TurnBasedComponent` - 回合制基类
- `TurnBasedTileBasedPlatformerControlComponent` - 平台控制子类
- `TileBasedPositionComponent` - 瓦片位置组件
- `TurnBasedEntity` - 回合制实体
- `TurnBasedCoordinator` - 回合制协调器

## 版本信息
- **状态**: 实验性功能
- **兼容性**: 需要完整的回合制系统支持
- **依赖**: 需要 `TileBasedPositionComponent` 和回合制系统 