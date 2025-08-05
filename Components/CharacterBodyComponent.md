# CharacterBodyComponent API 文档

## 概述
`CharacterBodyComponent` 管理CharacterBody2D的更新。确保每帧只调用一次 `CharacterBody2D.move_and_slide()`（防止过度移动）并更新相关标志。

**重要**：修改 `CharacterBody2D.velocity` 等属性后，必须将 `shouldMoveThisFrame` 设置为true。

## 继承
`CharacterBodyComponent` → `Component` → `Node`

## 要求
此组件必须放在所有移动物体的其他组件（如JumpComponent）之后。

## 主要特性
- 统一的CharacterBody2D管理
- 防止每帧多次移动
- 碰撞状态跟踪
- 速度历史记录
- 零运动时的"粘性"修复

## 导出属性

#### `body: CharacterBody2D`
要管理的CharacterBody2D。
- 如果为null，会在 `_enter_tree()` 时从父Entity获取
- 访问时如果为null会显示警告

#### `shouldResetVelocityIfZeroMotion: bool = false`
净运动为零时是否移除剩余"幽灵"速度。
- 启用以避免角色粘在墙上的"胶水效应"
- 在 `move_and_slide()` 后应用

## 状态属性

### 碰撞状态

#### `isOnFloor: bool`
`move_and_slide()` 后身体是否碰撞地面（可能从上一帧缓存）。

#### `wasOnFloor: bool`
上次 `move_and_slide()` 前身体是否在地面上。

#### `wasOnWall: bool`
上次 `move_and_slide()` 前身体是否在墙上。

#### `wasOnCeiling: bool`
上次 `move_and_slide()` 前身体是否在天花板上。

### 速度状态

#### `previousVelocity: Vector2`
上一帧的速度值。设置时会记录变化日志。

#### `previousWallNormal: Vector2`
与之接触的墙壁方向。

### 运动控制

#### `shouldMoveThisFrame: bool`
当前帧是否调用 `move_and_slide()`。
- 其他组件修改速度时应设置此标志
- 移动后自动重置为false
- 确保多个物理修改组件不会导致过度移动

#### `lastMotionCached: Vector2`
仅在 `shouldResetVelocityIfZeroMotion` 为true时使用和更新的缓存运动。

### 高级属性

#### `collisionShape: Shape2D` **（实验性）**
获取身体的碰撞形状。

## 信号

### `didMove(delta: float)`
`move_and_slide()` 后发出。需要在CharacterBody2D移动后处理更新的组件必须连接此信号。

## 主要方法

### 状态更新

#### `updateStateBeforeMove(delta: float) -> void`
移动前更新状态。
- 缓存碰撞状态（wasOnFloor等）
- 保存前一帧速度
- 记录墙壁法线

**注意**：如果子类重写此方法，必须调用super。

#### `updateStateAfterMove(delta: float) -> void`
移动后更新状态。
- 更新当前碰撞状态
- 应用零运动速度重置（如果启用）
- 记录速度变化

**注意**：如果子类重写此方法，必须调用super。

### 调试

#### `showDebugInfo() -> void`
显示调试信息，包括速度、运动、碰撞状态等。

## 使用示例

### 基本设置
```gdscript
# 设置CharacterBody组件
var bodyComponent = CharacterBodyComponent.new()
playerEntity.add_child(bodyComponent)

# 连接移动信号
bodyComponent.didMove.connect(_on_body_moved)

func _on_body_moved(delta: float):
    print("角色移动完成，速度：", bodyComponent.body.velocity)
```

### 移动控制模式
```gdscript
# 物理组件示例
class_name PlayerPhysicsComponent
extends Component

var bodyComponent: CharacterBodyComponent

func _ready():
    bodyComponent = coComponents.get(&"CharacterBodyComponent")

func _physics_process(delta: float):
    # 修改速度
    bodyComponent.body.velocity.x = Input.get_axis("move_left", "move_right") * 200
    
    # 重要：设置移动标志
    bodyComponent.shouldMoveThisFrame = true
```

### 跳跃组件集成
```gdscript
class_name JumpComponent
extends Component

var bodyComponent: CharacterBodyComponent
@export var jumpPower: float = 400

func _ready():
    bodyComponent = coComponents.get(&"CharacterBodyComponent")

func jump():
    if bodyComponent.isOnFloor:
        bodyComponent.body.velocity.y = -jumpPower
        bodyComponent.shouldMoveThisFrame = true
```

### 防止粘墙设置
```gdscript
# 启用零运动重置以防止粘墙
bodyComponent.shouldResetVelocityIfZeroMotion = true

# 这会在移动后自动执行：
# if is_zero_approx(lastMotion.x): body.velocity.x = 0
# if is_zero_approx(lastMotion.y): body.velocity.y = 0
```

### 状态检查
```gdscript
func _physics_process(delta: float):
    # 检查着陆
    if not bodyComponent.wasOnFloor and bodyComponent.isOnFloor:
        print("角色着陆")
        play_landing_sound()
    
    # 检查离开地面
    if bodyComponent.wasOnFloor and not bodyComponent.isOnFloor:
        print("角色离开地面")
    
    # 检查撞墙
    if not bodyComponent.wasOnWall and bodyComponent.body.is_on_wall():
        print("撞到墙壁，法线：", bodyComponent.body.get_wall_normal())
```

### 速度变化监控
```gdscript
func _on_body_moved(delta: float):
    # 检查速度变化
    var velocityDelta = bodyComponent.body.velocity - bodyComponent.previousVelocity
    
    if not velocityDelta.is_zero_approx():
        print("速度变化：", velocityDelta)
    
    # 检查突然停止
    if bodyComponent.previousVelocity.length() > 100 and bodyComponent.body.velocity.length() < 10:
        print("突然停止")
        apply_stop_particles()
```

## 性能优化

### 条件移动
```gdscript
# 只在需要时移动
func apply_movement(new_velocity: Vector2):
    if not new_velocity.is_equal_approx(bodyComponent.body.velocity):
        bodyComponent.body.velocity = new_velocity
        bodyComponent.shouldMoveThisFrame = true
```

### 批量速度修改
```gdscript
# 多个组件修改速度时的最佳实践
func _physics_process(delta: float):
    var needsMovement = false
    
    # 重力
    if not bodyComponent.isOnFloor:
        bodyComponent.body.velocity.y += gravity * delta
        needsMovement = true
    
    # 输入
    var horizontalInput = Input.get_axis("left", "right")
    if horizontalInput != 0:
        bodyComponent.body.velocity.x = horizontalInput * speed
        needsMovement = true
    
    # 一次性设置移动标志
    if needsMovement:
        bodyComponent.shouldMoveThisFrame = true
```

## 错误处理

### 缺失CharacterBody2D
```gdscript
func _ready():
    if not bodyComponent.body:
        push_error("实体缺少CharacterBody2D节点")
        return
```

### 多重移动检测
```gdscript
# 在组件中检查是否已经设置了移动
func apply_physics():
    if bodyComponent.shouldMoveThisFrame:
        print("警告：本帧已经请求移动")
    
    bodyComponent.body.velocity = calculate_velocity()
    bodyComponent.shouldMoveThisFrame = true
```

## 调试技巧

### 可视化调试
```gdscript
func _draw():
    if bodyComponent.debugMode:
        # 绘制速度向量
        var velocityEnd = bodyComponent.body.velocity * 0.1
        draw_line(Vector2.ZERO, velocityEnd, Color.RED, 2)
        
        # 绘制碰撞状态
        if bodyComponent.isOnFloor:
            draw_circle(Vector2(0, 20), 5, Color.GREEN)
```

### 状态日志
```gdscript
func _on_body_moved(delta: float):
    if bodyComponent.debugMode:
        print("移动状态：")
        print("  速度：", bodyComponent.body.velocity)
        print("  在地面：", bodyComponent.isOnFloor)
        print("  在墙上：", bodyComponent.body.is_on_wall())
        print("  最后运动：", bodyComponent.body.get_last_motion())
```

## 注意事项

1. **移动标志**：修改速度后始终设置 `shouldMoveThisFrame`
2. **组件顺序**：确保在修改速度的组件之后
3. **帧内多次修改**：同一帧内多次修改速度是安全的
4. **状态缓存**：某些状态从上一帧缓存，注意时序
5. **子类重写**：重写更新方法时必须调用super

## 相关组件
- `PlatformerPhysicsComponent` - 平台物理组件
- `JumpComponent` - 跳跃组件
- `GravityComponent` - 重力组件
- `InputComponent` - 输入控制组件 