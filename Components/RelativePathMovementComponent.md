# RelativePathMovementComponent API 参考

## 概述

`RelativePathMovementComponent` 应用相对于节点位置的一系列移动。该组件使用向量列表，每帧"消耗"这些向量来实现复杂的移动模式。适用于创建预定义的移动轨迹和动画路径。

**继承关系：**
`Component` → `RelativePathMovementComponent`

## 主要特性

- 📝 **相对移动** - 基于当前位置的相对移动向量
- 🔄 **逐帧消耗** - 每帧部分应用移动向量
- ⏰ **延迟控制** - 支持初始延迟和移动间延迟
- 🔁 **循环支持** - 可选的移动序列循环
- 🎯 **灵活目标** - 可指定移动的目标节点

## 导出属性

### 目标设置
```gdscript
@export var nodeOverride: Node2D
```
指定移动的目标节点。如果为 `null`，则使用 `parentEntity`。

### 移动参数
```gdscript
@export_range(0, 1000, 5) var speed: float = 50.0
```
移动速度（像素/秒）。

```gdscript
@export var vectors: Array[Vector2] = [Vector2(32, 0), Vector2(-32, 0)]
```
表示相对移动的向量列表，如"向左移动10像素"或"向西北移动45像素"。

### 时序控制
```gdscript
@export_range(0, 60, 0.1, "seconds") var initialDelay: float = 0.0
```
第一次移动前的延迟，不会在后续循环中重复。

```gdscript
@export_range(0, 60, 0.1, "seconds") var delayBetweenMoves: float = 0.0
```
移动之间的延迟时间。

### 行为设置
```gdscript
@export var shouldLoop: bool = true
```
是否循环执行移动序列。

```gdscript
@export var isEnabled: bool = true
```
控制组件是否启用。

## 状态属性

### 当前状态
```gdscript
var currentMoveIndex: int = 0
```
当前正在执行的移动向量的索引。

```gdscript
var currentVector: Vector2
```
当前剩余的相对移动量。

### 控制标志
```gdscript
var isInDelay: bool = false
```
当前是否处于延迟状态。

```gdscript
var hasNoMoreMoves: bool = false
```
如果 `shouldLoop` 为 `false` 或 `vectors` 数组为空时为 `true`。

## 信号

### willStartMove
```gdscript
signal willStartMove
```
在设置 `currentVector` 后发出，表示即将开始新的移动。

### didFinishMove
```gdscript
signal didFinishMove
```
完成当前移动向量时发出。

## 主要方法

### setNextMove()
```gdscript
func setNextMove() -> void
```
设置下一个移动：
- 处理移动间延迟
- 更新移动索引
- 处理循环逻辑

### setCurrentVector()
```gdscript
func setCurrentVector() -> Vector2
```
设置当前移动向量，如果索引无效则返回 `Vector2.ZERO`。

## 使用示例

### 基本往返移动
```gdscript
# 创建简单的左右往返移动
extends Entity

func _ready():
    var path_movement = $RelativePathMovementComponent
    
    # 设置往返移动：右32像素，然后左32像素
    path_movement.vectors = [Vector2(32, 0), Vector2(-32, 0)]
    path_movement.speed = 50.0
    path_movement.shouldLoop = true
    path_movement.delayBetweenMoves = 1.0
```

### 复杂巡逻路径
```gdscript
# 创建复杂的巡逻模式
extends GuardEntity

func setup_patrol_pattern():
    var movement = $RelativePathMovementComponent
    
    # 方形巡逻路径
    movement.vectors = [
        Vector2(64, 0),    # 向右
        Vector2(0, 64),    # 向下
        Vector2(-64, 0),   # 向左
        Vector2(0, -64)    # 向上
    ]
    movement.speed = 30.0
    movement.shouldLoop = true
    movement.delayBetweenMoves = 2.0  # 每个转角停留2秒
    movement.initialDelay = 3.0       # 开始前等待3秒
```

### 动态移动序列
```gdscript
# 运行时修改移动模式
extends EnemyEntity

@onready var movement = $RelativePathMovementComponent

func _ready():
    # 监听移动完成事件
    movement.didFinishMove.connect(_on_move_finished)
    setup_initial_pattern()

func setup_initial_pattern():
    movement.vectors = [Vector2(50, 0), Vector2(-50, 0)]
    movement.speed = 40.0
    movement.shouldLoop = true

func _on_move_finished():
    # 随机调整下一次移动
    if randf() < 0.3:  # 30%概率改变模式
        change_movement_pattern()

func change_movement_pattern():
    var new_patterns = [
        [Vector2(0, 40), Vector2(0, -40)],     # 垂直移动
        [Vector2(30, 30), Vector2(-30, -30)],  # 对角移动
        [Vector2(60, 0), Vector2(-30, 0), Vector2(-30, 0)]  # 不对称移动
    ]
    movement.vectors = new_patterns.pick_random()
```

### 与其他组件集成
```gdscript
# 结合伤害检测的移动敌人
extends Entity

@onready var movement = $RelativePathMovementComponent
@onready var damage_receiving = $DamageReceivingComponent

func _ready():
    setup_movement()
    damage_receiving.didReceiveDamage.connect(_on_damaged)

func setup_movement():
    movement.vectors = [Vector2(40, 0), Vector2(-40, 0)]
    movement.speed = 25.0
    movement.shouldLoop = true

func _on_damaged(damage_component: DamageComponent, amount: int, factions: int):
    # 受伤时暂时停止移动
    movement.isEnabled = false
    
    # 创建停顿效果
    await get_tree().create_timer(0.5).timeout
    movement.isEnabled = true
    
    # 受伤后加速移动
    movement.speed *= 1.2
```

### 触发式移动
```gdscript
# 基于事件触发的移动序列
extends InteractiveEntity

@onready var movement = $RelativePathMovementComponent

func _ready():
    # 初始时禁用移动
    movement.isEnabled = false
    movement.shouldLoop = false
    
    # 设置触发移动序列
    movement.vectors = [
        Vector2(0, -20),   # 向上移动
        Vector2(40, 0),    # 向右移动
        Vector2(0, 20),    # 回到原始高度
        Vector2(-40, 0)    # 回到原始位置
    ]

func activate_movement():
    movement.isEnabled = true
    # 监听移动完成
    if not movement.didFinishMove.is_connected(_on_sequence_complete):
        movement.didFinishMove.connect(_on_sequence_complete)

func _on_sequence_complete():
    # 检查是否完成了所有移动
    if movement.hasNoMoreMoves:
        on_movement_sequence_finished()

func on_movement_sequence_finished():
    # 移动序列完成后的处理
    print("移动序列完成！")
    # 可以触发其他事件、播放音效等
```

## 技术细节

### 移动算法
- 使用 `Vector2.move_toward()` 计算每帧的移动量
- 逐帧从当前向量中减去已应用的移动
- 当向量接近零时切换到下一个移动

### 延迟处理
- 使用 `get_tree().create_timer()` 实现异步延迟
- 延迟期间设置标志位防止移动更新
- 支持初始延迟和移动间延迟

### 坐标系统
- 所有移动都相对于目标节点的当前位置
- 支持指定不同的目标节点
- 直接修改节点的 `position` 属性

## 注意事项

⚠️ **组件交互：**
- 与其他操纵移动的组件一起使用时需要小心
- 直接修改位置可能与物理系统冲突
- 建议在使用前充分测试组件间交互

💡 **性能优化：**
- 避免过多的小幅移动向量
- 合理设置延迟时间避免频繁触发
- 在不需要时禁用组件

🔮 **使用技巧：**
- 结合信号系统创建复杂的移动逻辑
- 使用不同的移动速度创建变速效果
- 配合动画系统增强视觉效果

## 相关组件

- [`PathFollowComponent`](PathFollowComponent.md) - 基于Path2D的路径跟随
- [`LinearMotionComponent`](LinearMotionComponent.md) - 简单的直线运动
- [`WaveMotionComponent`](WaveMotionComponent.md) - 波形运动模式
- [`NavigationComponent`](NavigationComponent.md) - 智能寻路移动 