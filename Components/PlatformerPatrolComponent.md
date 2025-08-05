# PlatformerPatrolComponent API

## 概述
`PlatformerPatrolComponent` 修改`InputComponent`使实体在"地面"平台上水平来回移动。当检测到平台边缘或墙壁时自动转向，适用于AI敌人的巡逻行为。

**继承链**：`Component` → `CharacterBodyDependentComponentBase` → `PlatformerPatrolComponent`

## 依赖要求
- **CornerCollisionComponent**：检测平台边缘和墙壁
- **PlatformerPhysicsComponent**：处理移动物理
- **InputComponent**：接收修改后的输入

## 组件顺序要求
- **之前**：PlatformerPhysicsComponent、InputComponent
- **之后**：CornerCollisionComponent

## 主要特性
- 🚶 **自动巡逻**：在平台上自动来回移动
- 🔄 **智能转向**：检测边缘和墙壁自动转向
- 🎲 **随机方向**：可随机化初始移动方向
- ⏱️ **转向延迟**：可设置转向时的停顿时间
- 👁️ **边缘检测**：精确的平台边缘和墙壁检测

## 导出属性

### 检测配置
- **turningDelay** (`float = 0`, 0-10秒)：转向延迟时间（未实现）
- **detectionGap** (`float = 0`, 0-16像素)：精灵边缘检测距离

### 方向配置
- **initialDirection** (`int = +1`)：初始移动方向
  - `-1` = 左
  - `+1` = 右
- **randomizeInitialDirection** (`bool = false`)：是否随机化初始方向
  - 覆盖initialDirection设置

### 基础配置
- **isEnabled** (`bool = true`)：是否启用巡逻功能

## 状态属性

### 碰撞检测状态
- **isFloorOnLeft** (`bool`)：左侧是否有地面
- **isFloorOnRight** (`bool`)：右侧是否有地面
- **isWallOnLeft** (`bool`)：左侧是否有墙壁
- **isWallOnRight** (`bool`)：右侧是否有墙壁

### 移动状态
- **patrolDirection** (`float`)：当前巡逻方向
- **previousPatrolDirection** (`float`)：前一次巡逻方向

## 信号
- **didTurn**()：实体在平台末端转向时发出

## 主要方法

### 状态更新
- **updateCollisionFlags**() → `void`：更新碰撞检测标志
- **updatePatrolDirection**() → `void`：根据检测结果更新巡逻方向
- **getNextHorizontalInput**(delta: float) → `float`：获取下一帧的水平输入

### 初始化
- **setInitialDirection**() → `void`：设置初始移动方向

## 使用示例

### 基础巡逻敌人
```gdscript
# 设置基本的巡逻AI
@export var patrol_component: PlatformerPatrolComponent
@export var corner_collision: CornerCollisionComponent

func _ready():
    patrol_component.randomizeInitialDirection = true
    patrol_component.isEnabled = true
    patrol_component.didTurn.connect(_on_patrol_turn)

func _on_patrol_turn():
    print("敌人改变了巡逻方向")
    # 可添加转向音效或动画
```

### 守卫巡逻系统
```gdscript
# 复杂的守卫巡逻行为
@export var patrol_component: PlatformerPatrolComponent
@export var detection_area: Area2D
var is_chasing_player: bool = false

func _ready():
    patrol_component.initialDirection = 1  # 向右开始
    detection_area.body_entered.connect(_on_player_detected)
    detection_area.body_exited.connect(_on_player_lost)

func _on_player_detected(body):
    if body.is_in_group("player"):
        is_chasing_player = true
        patrol_component.isEnabled = false  # 停止巡逻，开始追击

func _on_player_lost(body):
    if body.is_in_group("player"):
        is_chasing_player = false
        patrol_component.isEnabled = true   # 恢复巡逻
```

### 条件巡逻
```gdscript
# 基于条件的巡逻控制
@export var patrol_component: PlatformerPatrolComponent
@export var health_component: HealthComponent

func _ready():
    health_component.didTakeDamage.connect(_on_take_damage)

func _on_take_damage():
    # 受伤时随机改变方向
    patrol_component.patrolDirection *= -1
    
    # 短暂停止巡逻
    patrol_component.isEnabled = false
    await get_tree().create_timer(1.0).timeout
    patrol_component.isEnabled = true
```

### 巡逻区域限制
```gdscript
# 限制巡逻范围的系统
@export var patrol_component: PlatformerPatrolComponent
@export var patrol_bounds: Array[Vector2] = []  # 巡逻边界点
var start_position: Vector2

func _ready():
    start_position = global_position
    patrol_component.didTurn.connect(_check_patrol_bounds)

func _check_patrol_bounds():
    # 检查是否超出巡逻范围
    var current_pos = global_position
    var distance_from_start = abs(current_pos.x - start_position.x)
    
    if distance_from_start > patrol_bounds[0].x:
        # 强制返回起始区域
        patrol_component.patrolDirection = sign(start_position.x - current_pos.x)
```

### 巡逻状态可视化
```gdscript
# 可视化巡逻状态
@export var patrol_component: PlatformerPatrolComponent
@export var debug_label: Label

func _physics_process(_delta):
    if debug_label:
        var status = "巡逻状态:\n"
        status += f"方向: {patrol_component.patrolDirection}\n"
        status += f"左侧地面: {patrol_component.isFloorOnLeft}\n"
        status += f"右侧地面: {patrol_component.isFloorOnRight}\n"
        status += f"左侧墙壁: {patrol_component.isWallOnLeft}\n"
        status += f"右侧墙壁: {patrol_component.isWallOnRight}"
        debug_label.text = status
```

## 设计模式

### 巡逻状态机
```gdscript
# 复杂的巡逻行为状态机
class_name PatrolStateMachine
extends Node

@export var patrol_component: PlatformerPatrolComponent
var current_state: PatrolState = PatrolState.PATROLLING

enum PatrolState {
    PATROLLING,
    TURNING,
    WAITING,
    CHASING
}

func _ready():
    patrol_component.didTurn.connect(_on_patrol_turn)

func _on_patrol_turn():
    if current_state == PatrolState.PATROLLING:
        change_state(PatrolState.TURNING)

func change_state(new_state: PatrolState):
    match new_state:
        PatrolState.TURNING:
            patrol_component.isEnabled = false
            await get_tree().create_timer(0.5).timeout
            change_state(PatrolState.PATROLLING)
        PatrolState.PATROLLING:
            patrol_component.isEnabled = true
    
    current_state = new_state
```

### 智能巡逻路径
```gdscript
# 多点巡逻路径系统
class_name SmartPatrol
extends Node

@export var patrol_component: PlatformerPatrolComponent
@export var waypoints: Array[Vector2] = []
var current_target_index: int = 0

func _ready():
    if waypoints.size() > 0:
        setup_waypoint_patrol()

func setup_waypoint_patrol():
    # 禁用默认巡逻，使用自定义逻辑
    patrol_component.isEnabled = false

func _physics_process(_delta):
    if waypoints.size() == 0:
        return
    
    # 检查是否到达当前目标点
    var current_target = waypoints[current_target_index]
    var distance = abs(global_position.x - current_target.x)
    
    if distance < 10.0:  # 到达阈值
        current_target_index = (current_target_index + 1) % waypoints.size()
        current_target = waypoints[current_target_index]
    
    # 设置移动方向
    var direction = sign(current_target.x - global_position.x)
    patrol_component.inputDirectionOverride = direction
```

## 技术细节

### 碰撞检测机制
- 依赖CornerCollisionComponent的SW/SE区域检测地面
- 使用NW/NE区域检测墙壁碰撞
- 每帧更新碰撞标志确保准确性

### 方向切换逻辑
1. 检查当前方向是否被阻挡
2. 如果阻挡，切换到相反方向
3. 如果两个方向都阻挡，停止移动
4. 如果没有输入，尝试恢复前一个方向

### 输入覆盖
- 设置InputComponent.shouldSkipNextEvent = true
- 直接设置InputComponent.horizontalInput
- 确保AI输入覆盖玩家输入

## 注意事项

### 组件依赖性
- 必须有CornerCollisionComponent进行边缘检测
- 需要正确配置碰撞检测区域
- 确保物理组件初始化顺序正确

### 性能考虑
- 每帧更新碰撞检测可能影响性能
- 考虑降低检测频率或优化检测逻辑
- 大量巡逻敌人时注意性能开销

### 设计建议
- 转向延迟功能待实现，可增加行为真实感
- 结合动画系统提供视觉反馈
- 考虑添加巡逻范围限制

## 相关组件
- [`CornerCollisionComponent`](./CornerCollisionComponent.md) - 边缘碰撞检测
- [`PlatformerPhysicsComponent`](./PlatformerPhysicsComponent.md) - 平台物理
- [`InputComponent`](./InputComponent.md) - 输入控制
- [`CharacterBodyComponent`](./CharacterBodyComponent.md) - CharacterBody2D管理 