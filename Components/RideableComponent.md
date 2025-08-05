# RideableComponent API

## 概述
`RideableComponent` 是一个专门用于创建载具、坐骑或可骑乘实体的运动组件。它允许一个实体"安装"和"骑乘"另一个实体，如玩家驾驶载具或骑马等。使用`RemoteTransform2D`子节点来附着骑乘者。

**继承链**：`Component` → `RideableComponent`

## 依赖要求
- **RemoteTransform2D**：必须有一个名为"RiderPlaceholder"的子节点

## 主要特性
- 🏇 **载具骑乘**：支持实体之间的骑乘关系
- 🎮 **控制转移**：可配置组件启用/禁用，实现控制权转移
- 🔄 **精灵同步**：自动同步载具和骑乘者的sprite方向
- ⌨️ **手动下马**：支持输入事件触发下马
- 📍 **位置偏移**：可配置骑乘者相对载具的位置
- 🔗 **组件切换**：自动管理载具和骑乘者之间的组件状态

## 导出属性

### 核心配置
- **rider** (`Entity`)：当前的骑乘者实体
- **riderPositionOffset** (`Vector2 = Vector2(0, -16)`)：骑乘者相对于组件的位置偏移

### 控制管理
- **componentTypesToToggle** (`Array[Script]`)：上马/下马时要切换的组件类型
  - 默认包含：`PlatformerControlComponent`, `JumpControlComponent`, `ActionsComponent`, `ActionControlComponent`
  - 载具：上马时启用，下马时禁用
  - 骑乘者：上马时禁用，下马时重新启用
- **shouldTogglePause** (`bool = false`)：切换组件时是否同时暂停
- **dismountInputEventName** (`StringName`)：手动下马的输入事件名称
- **isEnabled** (`bool = true`)：是否启用载具功能

## 只读属性
- **isMounted** (`bool`)：是否有骑乘者（基于rider是否有效）

## 信号

### 骑乘事件
- **didMount**(newRider: Entity)：当新骑乘者上马时发出
- **didDismount**(previousRider: Entity)：当骑乘者下马时发出

## 主要方法

### 骑乘控制
- **mount**(newRider: Entity) → `bool`：安装新骑乘者，成功返回true
- **dismount**() → `bool`：移除当前骑乘者，成功返回true
- **syncSpriteFlip**() → `bool`：同步载具和骑乘者的sprite翻转，返回载具sprite的flip_h状态

## 使用示例

### 基础载具系统
```gdscript
# 设置基本的载具组件
@export var rideable_component: RideableComponent
@export var player: Entity

func _ready():
    rideable_component.riderPositionOffset = Vector2(0, -20)
    rideable_component.dismountInputEventName = "ui_cancel"
    
    # 玩家上马
    if rideable_component.mount(player):
        print("玩家成功上马!")
```

### 载具控制转移
```gdscript
# 配置载具的控制组件切换
@export var rideable_component: RideableComponent

func setup_vehicle_control():
    # 定义要切换的组件类型
    rideable_component.componentTypesToToggle = [
        InputComponent,
        PlatformerControlComponent,
        ActionsComponent
    ]
    rideable_component.shouldTogglePause = true
```

### 多人载具系统
```gdscript
# 支持多个骑乘位置的载具
class_name MultiRiderVehicle
extends Entity

@export var rider_positions: Array[RideableComponent]
@export var max_riders: int = 4

func try_mount_player(player: Entity) -> bool:
    for rideable in rider_positions:
        if not rideable.isMounted:
            return rideable.mount(player)
    return false

func dismount_all():
    for rideable in rider_positions:
        if rideable.isMounted:
            rideable.dismount()
```

### 条件骑乘系统
```gdscript
# 基于条件的骑乘控制
@export var rideable_component: RideableComponent
@export var required_item: InventoryItem

func check_mount_permission(rider: Entity) -> bool:
    var inventory = rider.findFirstComponentSubclass(InventoryComponent)
    if not inventory:
        return false
    
    return inventory.hasItem(required_item)

func conditional_mount(rider: Entity) -> bool:
    if check_mount_permission(rider):
        return rideable_component.mount(rider)
    else:
        Global.showMessage("需要钥匙才能使用这个载具!")
        return false
```

### 载具状态监控
```gdscript
# 监控载具状态并响应事件
@export var rideable_component: RideableComponent
@export var engine_sound: AudioStreamPlayer2D
@export var idle_animation: AnimationPlayer

func _ready():
    rideable_component.didMount.connect(_on_rider_mounted)
    rideable_component.didDismount.connect(_on_rider_dismounted)

func _on_rider_mounted(rider: Entity):
    engine_sound.play()
    idle_animation.stop()
    print(f"{rider.name} 上了车")

func _on_rider_dismounted(rider: Entity):
    engine_sound.stop()
    idle_animation.play("idle")
    print(f"{rider.name} 下了车")
```

### 载具特效同步
```gdscript
# 同步载具和骑乘者的视觉效果
@export var rideable_component: RideableComponent
@export var vehicle_particles: GPUParticles2D

func _ready():
    rideable_component.didMount.connect(_on_mount)
    rideable_component.didDismount.connect(_on_dismount)

func _on_mount(rider: Entity):
    # 启动载具特效
    vehicle_particles.emitting = true
    
    # 隐藏骑乘者的粒子效果
    var rider_particles = rider.get_node_or_null("Particles")
    if rider_particles:
        rider_particles.emitting = false

func _on_dismount(rider: Entity):
    # 停止载具特效
    vehicle_particles.emitting = false
    
    # 恢复骑乘者的粒子效果
    var rider_particles = rider.get_node_or_null("Particles")
    if rider_particles:
        rider_particles.emitting = true
```

## 设计模式

### 载具工厂
```gdscript
# 载具管理和生成系统
class_name VehicleManager
extends Node

@export var vehicle_scenes: Array[PackedScene]
@export var spawn_points: Array[Node2D]

func spawn_vehicle(type: int, spawn_index: int) -> RideableComponent:
    if type >= vehicle_scenes.size() or spawn_index >= spawn_points.size():
        return null
    
    var vehicle = vehicle_scenes[type].instantiate()
    spawn_points[spawn_index].add_child(vehicle)
    
    return vehicle.findFirstComponentSubclass(RideableComponent)

func setup_vehicle_for_player(vehicle: RideableComponent, player: Entity):
    # 配置载具以适应玩家
    vehicle.componentTypesToToggle = [InputComponent, JumpComponent]
    vehicle.dismountInputEventName = "ui_cancel"
    vehicle.mount(player)
```

### 载具升级系统
```gdscript
# 载具升级和自定义
@export var rideable_component: RideableComponent
@export var base_components: Array[Script] = [InputComponent, ActionsComponent]

func apply_vehicle_upgrade(upgrade: VehicleUpgrade):
    match upgrade.type:
        VehicleUpgrade.Type.ENHANCED_CONTROL:
            rideable_component.componentTypesToToggle.append(JumpComponent)
        VehicleUpgrade.Type.QUICK_DISMOUNT:
            rideable_component.dismountInputEventName = "ui_select"
        VehicleUpgrade.Type.PASSENGER_SEAT:
            rideable_component.shouldTogglePause = false
```

## 技术细节

### 位置同步机制
- 使用`RemoteTransform2D`实现骑乘者位置跟随
- 组件位置而非实体位置，提供额外控制精度
- 自动处理位置偏移和旋转同步

### 组件管理
- 通过`Entity.toggleComponents()`管理组件状态
- 支持组件暂停而非仅禁用
- 智能处理组件之间的依赖关系

### 精灵同步
- 自动检测和缓存载具及骑乘者的sprite
- 响应控制组件的方向变化事件
- 处理AnimatedSprite2D和Sprite2D

## 注意事项

### RemoteTransform2D配置
- 必须有名为"RiderPlaceholder"的RemoteTransform2D子节点
- 在编辑器中启用"Editable Children"调整骑乘者偏移
- 确保RemoteTransform2D的update_position和update_rotation正确设置

### 组件切换协调
- 确保componentTypesToToggle中的组件类型存在
- 注意组件之间的依赖关系
- 测试不同组件组合的兼容性

### 生命周期管理
- 实体删除时自动处理下马
- 避免悬空引用和内存泄漏
- 正确处理信号连接和断开

### 与其他系统的区别
- 与`AttachmentComponent`不同，这是双向的载具关系
- 涉及复杂的控制权转移和状态管理
- 专门为载具/骑乘场景设计

## 相关组件
- [`AttachmentComponent`](../Movement/AttachmentComponent.md) - 简单节点附着
- [`InputComponent`](../Control/InputComponent.md) - 输入控制
- [`PlatformerControlComponent`](../Control/PlatformerControlComponent.md) - 平台游戏控制
- [`ActionsComponent`](../Gameplay/ActionsComponent.md) - 行动系统