# TileBasedRandomMovementComponent API 参考

## 概述

`TileBasedRandomMovementComponent` 继承自 [`TileBasedControlComponent`](./TileBasedControlComponent.md)，专门用于实现随机移动。该组件会根据定时器在预设的移动选项中随机选择方向，适用于AI敌人的巡逻行为。

**继承关系：**
`Component` → `TileBasedControlComponent` → `TileBasedRandomMovementComponent`

## 主要特性

- 🎲 **随机移动** - 从预设方向集合中随机选择
- ⏰ **定时控制** - 基于Timer的移动触发
- 🔄 **智能重试** - 可配置的有效移动重试机制
- 🚫 **输入抑制** - 自动禁用玩家输入处理

## 导出属性

### 移动配置
```gdscript
@export var horizontalMovesSet: Array[int] = [-1, 0, 1]
```
水平方向的移动步长数组，每次随机从中选择一个值。

```gdscript
@export var verticalMovesSet: Array[int] = [-1, 0, 1]
```
垂直方向的移动步长数组，每次随机从中选择一个值。

### 行为设置
```gdscript
@export var shouldKeepTryingUntilValidMove: bool = true
```
如果为 `true`，会持续重试直到找到有效的移动位置。

```gdscript
const maximumTries: int = 10
```
最大重试次数，防止无限循环。

## 子节点

### StepTimer
- **类型：** `Timer`
- **用途：** 控制随机移动的触发间隔
- **配置：** 在场景中设置 `wait_time` 和 `autostart`

## 主要方法

### moveRandomly()
```gdscript
func moveRandomly() -> void
```
执行随机移动逻辑：
- 生成随机方向向量
- 如果启用智能重试，验证目标位置有效性
- 调用移动方法

### getRandomVector()
```gdscript
func getRandomVector() -> Vector2i
```
从水平和垂直移动集合中随机生成移动向量。

## 使用示例

### 基本AI巡逻
```gdscript
# 为敌人设置简单的随机巡逻
extends EnemyEntity

func _ready():
    var random_movement = $TileBasedRandomMovementComponent
    
    # 配置移动选项（仅四个基本方向）
    random_movement.horizontalMovesSet = [-1, 0, 1]
    random_movement.verticalMovesSet = [-1, 0, 1]
    
    # 设置移动间隔
    random_movement.get_node("StepTimer").wait_time = 2.0
    random_movement.shouldKeepTryingUntilValidMove = true
```

### 限制性随机移动
```gdscript
# 创建只能左右移动的AI
extends Entity

func setup_horizontal_patrol():
    var movement = $TileBasedRandomMovementComponent
    
    # 仅水平移动
    movement.horizontalMovesSet = [-1, 0, 1]
    movement.verticalMovesSet = [0]  # 不允许垂直移动
    
    movement.get_node("StepTimer").wait_time = 1.5
```

### 快速移动AI
```gdscript
# 创建可以跳跃多格的快速AI
extends Entity

func setup_fast_random_movement():
    var movement = $TileBasedRandomMovementComponent
    
    # 允许更大的移动距离
    movement.horizontalMovesSet = [-2, -1, 0, 1, 2]
    movement.verticalMovesSet = [-2, -1, 0, 1, 2]
    
    # 更快的移动频率
    movement.get_node("StepTimer").wait_time = 0.8
    movement.shouldKeepTryingUntilValidMove = true
```

### 复杂AI行为
```gdscript
# 结合其他组件创建更智能的AI
extends EnemyEntity

@onready var random_movement = $TileBasedRandomMovementComponent
@onready var chase_component = $ChaseComponent
@onready var faction_component = $FactionComponent

var is_chasing_player = false

func _ready():
    # 配置随机巡逻
    random_movement.horizontalMovesSet = [-1, 0, 1]
    random_movement.verticalMovesSet = [-1, 0, 1]
    random_movement.get_node("StepTimer").wait_time = 3.0
    
    # 监听玩家发现事件
    chase_component.didStartChasing.connect(_on_start_chasing)
    chase_component.didStopChasing.connect(_on_stop_chasing)

func _on_start_chasing(_target):
    # 停止随机移动，开始追逐
    random_movement.isEnabled = false
    is_chasing_player = true

func _on_stop_chasing():
    # 恢复随机巡逻
    random_movement.isEnabled = true
    is_chasing_player = false
```

## 技术细节

### 输入抑制机制
- 重写 `_input()`、`_unhandled_input()` 和 `_physics_process()` 为空
- 禁用物理处理、常规输入和未处理输入
- 确保不会响应玩家输入

### 移动验证
- 使用 `tileBasedPositionComponent.validateCoordinates()` 验证目标位置
- 支持重试机制避免无效移动
- 最大重试次数防止性能问题

### 时机控制
- 基于Timer的异步移动触发
- 在到达新单元格时可选择重启定时器
- 与瓦片位置组件的信号系统集成

## 注意事项

⚠️ **依赖要求：**
- 需要与 [`TileBasedControlComponent`](./TileBasedControlComponent.md) 相同的依赖
- 必须有有效的 `TileBasedPositionComponent`

💡 **性能考虑：**
- 重试机制可能在复杂地图中影响性能
- 合理设置 `maximumTries` 避免过度计算
- 定时器间隔应根据游戏节奏调整

🔮 **使用技巧：**
- 结合 `ChaseComponent` 创建智能AI
- 配合 `FactionComponent` 实现阵营巡逻
- 与 `AreaCollisionComponent` 结合检测玩家接近

## 相关组件

- [`TileBasedControlComponent`](./TileBasedControlComponent.md) - 父类，提供基础瓦片控制
- [`TileBasedPositionComponent`](TileBasedPositionComponent.md) - 必需依赖，管理瓦片位置
- [`ChaseComponent`](ChaseComponent.md) - 常用搭配，实现追逐行为
- [`FactionComponent`](./FactionComponent.md) - 可选搭配，阵营系统 