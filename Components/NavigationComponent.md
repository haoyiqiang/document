# NavigationComponent API

## 概述
`NavigationComponent` 是一个基于NavigationAgent2D的寻路组件，用于指导实体朝向指定目标移动，同时避开墙壁等障碍物。它使用内部定时器定期更新目标位置，提供智能的pathfinding功能。

**继承链**：`Component` → `NavigationComponent`

**实验性功能**：此组件为实验性质，API可能在后续版本中变更。

## 主要特性
- 🧭 **智能寻路**：基于NavigationAgent2D的障碍物避让
- 🎯 **目标追踪**：自动朝向指定节点移动
- ⏰ **性能优化**：定时器更新目标而非每帧计算
- 🎮 **玩家追踪**：支持自动追踪玩家选项
- 🔧 **可扩展**：支持重写移动方法实现不同物理行为
- 📊 **调试支持**：详细的调试信息显示

## 导出属性

### 目标配置
- **destinationNode** (`Node2D`)：目标节点
  - 设置时自动更新目标位置并启动定时器
- **shouldChasePlayerIfUnspecified** (`bool = true`)：未指定目标时是否追踪玩家
  - 自动选择`GameState.players`中的第一个玩家

### 控制选项
- **isEnabled** (`bool = true`)：是否启用导航功能
  - 影响定时器和移动处理的启用状态

## 状态属性

### 导航状态
- **selfAsAgent** (`NavigationAgent2D`)：组件作为NavigationAgent2D的引用
- **recentDirection** (`Vector2`)：最近的移动方向向量

## 主要方法

### 导航控制
- **updateTargetPosition()** → `void`：更新NavigationAgent2D的目标位置
- **moveTowardsDestination()** → `void`：朝目标移动（可重写）
- **onDestinationUpdateTimer_timeout()** → `void`：定时器超时处理

### 调试功能
- **showDebugInfo()** → `void`：显示导航调试信息

## 使用示例

### 基础寻路设置
```gdscript
# 设置简单的AI寻路
@export var navigation_component: NavigationComponent
@export var target_player: Node2D

func _ready():
    navigation_component.destinationNode = target_player
    navigation_component.isEnabled = true
```

### AI敌人追踪玩家
```gdscript
# 敌人自动追踪玩家
@export var enemy_navigation: NavigationComponent

func _ready():
    # 未指定目标时自动追踪玩家
    enemy_navigation.shouldChasePlayerIfUnspecified = true
    enemy_navigation.isEnabled = true
    
    # 设置NavigationAgent2D属性
    var agent = enemy_navigation.get_node(".") as NavigationAgent2D
    agent.max_speed = 150.0
    agent.path_desired_distance = 10.0
    agent.target_desired_distance = 10.0
```

### 巡逻AI系统
```gdscript
# 在多个巡逻点之间移动
@export var patrol_navigation: NavigationComponent
@export var patrol_points: Array[Node2D]

var current_patrol_index: int = 0

func _ready():
    patrol_navigation.isEnabled = true
    if patrol_points.size() > 0:
        set_next_patrol_point()

func set_next_patrol_point():
    if patrol_points.size() == 0:
        return
    
    patrol_navigation.destinationNode = patrol_points[current_patrol_index]
    current_patrol_index = (current_patrol_index + 1) % patrol_points.size()

func _on_patrol_area_reached():
    # 到达巡逻点后等待一段时间再前往下一个
    await get_tree().create_timer(2.0).timeout
    set_next_patrol_point()
```

### 条件导航控制
```gdscript
# 基于游戏状态的导航行为
@export var navigation_component: NavigationComponent
@export var detection_area: Area2D

var player_detected: bool = false

func _ready():
    detection_area.body_entered.connect(on_player_detected)
    detection_area.body_exited.connect(on_player_lost)

func on_player_detected(body: Node2D):
    if body.is_in_group("player"):
        player_detected = true
        navigation_component.destinationNode = body
        navigation_component.isEnabled = true

func on_player_lost(body: Node2D):
    if body.is_in_group("player"):
        player_detected = false
        navigation_component.isEnabled = false
```

### 自定义移动行为
```gdscript
# 重写移动方法使用CharacterBody2D物理
class_name PhysicsNavigationComponent
extends NavigationComponent

@export var character_body: CharacterBody2D
@export var move_speed: float = 200.0

func moveTowardsDestination() -> void:
    if not character_body:
        super.moveTowardsDestination()
        return
    
    # 使用CharacterBody2D速度而非直接设置位置
    character_body.velocity = recentDirection * move_speed
    character_body.move_and_slide()
```

## 设计模式

### 群体导航系统
```gdscript
# 管理多个AI单位的导航
class_name FlockNavigationManager
extends Node

@export var navigation_components: Array[NavigationComponent]
@export var formation_spacing: float = 50.0

var leader_target: Node2D

func set_group_target(target: Node2D):
    leader_target = target
    update_formation_targets()

func update_formation_targets():
    for i in navigation_components.size():
        var nav_component = navigation_components[i]
        
        if i == 0:
            # 队长直接追踪目标
            nav_component.destinationNode = leader_target
        else:
            # 其他单位形成编队
            var formation_offset = Vector2(
                (i % 3 - 1) * formation_spacing,
                (i / 3) * formation_spacing
            )
            create_formation_target(nav_component, formation_offset)

func create_formation_target(nav_component: NavigationComponent, offset: Vector2):
    var formation_target = Node2D.new()
    add_child(formation_target)
    formation_target.global_position = leader_target.global_position + offset
    nav_component.destinationNode = formation_target
```

### 动态导航切换
```gdscript
# 根据情况切换不同的导航目标
@export var navigation_component: NavigationComponent
@export var home_position: Node2D
@export var alert_position: Node2D

enum AIState { IDLE, PATROL, CHASE, RETREAT }
var current_state: AIState = AIState.IDLE

func change_ai_state(new_state: AIState):
    current_state = new_state
    
    match new_state:
        AIState.IDLE:
            navigation_component.isEnabled = false
        AIState.PATROL:
            navigation_component.destinationNode = home_position
            navigation_component.isEnabled = true
        AIState.CHASE:
            navigation_component.shouldChasePlayerIfUnspecified = true
            navigation_component.destinationNode = null  # 触发玩家追踪
            navigation_component.isEnabled = true
        AIState.RETREAT:
            navigation_component.destinationNode = home_position
            navigation_component.isEnabled = true
```

### 智能导航调度器
```gdscript
# 优化大量AI单位的导航计算
class_name NavigationScheduler
extends Node

var navigation_components: Array[NavigationComponent] = []
var update_index: int = 0
var updates_per_frame: int = 3

func register_navigation(nav_component: NavigationComponent):
    navigation_components.append(nav_component)

func _physics_process(_delta):
    # 分帧更新不同的导航组件
    for i in updates_per_frame:
        if update_index >= navigation_components.size():
            update_index = 0
            break
        
        var nav_component = navigation_components[update_index]
        if is_instance_valid(nav_component):
            nav_component.updateTargetPosition()
        
        update_index += 1
```

## 技术细节

### NavigationAgent2D集成
- 组件本身就是NavigationAgent2D节点
- 使用`selfAsAgent`引用访问导航功能
- 支持所有NavigationAgent2D的配置选项

### 性能优化策略
- 使用DestinationUpdateTimer定期更新目标
- 移动方向每帧计算，目标位置定时更新
- 避免每帧的昂贵路径计算

### 移动方法设计
- 默认直接设置position，无物理交互
- 可重写`moveTowardsDestination()`实现不同移动方式
- 支持CharacterBody2D、RigidBody2D等物理方式

## 注意事项

### 实验性提醒
- 此组件为实验性质，API可能变更
- 建议在正式项目中谨慎使用
- 未来版本可能有重大改动

### 性能考虑
- 大量导航组件可能影响性能
- 合理设置更新频率和导航精度
- 考虑使用导航调度器分散计算负担

### 与ChaseComponent的区别
- NavigationComponent：复杂寻路，避开障碍
- ChaseComponent：简单直线追踪，无障碍避让
- 根据需求选择合适的组件

## 相关组件
- [`ChaseComponent`](./ChaseComponent.md) - 简单追踪移动
- [`LinearMotionComponent`](./LinearMotionComponent.md) - 直线运动
- [`PathFollowComponent`](./PathFollowComponent.md) - 路径跟随
- [`CharacterBodyComponent`](./CharacterBodyComponent.md) - 角色物理体 