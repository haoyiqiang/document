# PushRigidBodyComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > PushRigidBodyComponent  
> **物理类型**: 刚体推动

允许CharacterBody2D推动RigidBody2D的物理组件，为角色与环境中可移动物体的交互提供真实的物理反馈。

## ✨ 主要特性

- 🏋️ CharacterBody2D推动RigidBody2D
- ⚡ 实时碰撞检测和力计算
- 🎛️ 可调节的推动力度
- 🔄 可启用/禁用功能
- 📊 性能优化的处理顺序

## 📊 导出属性

### 物理设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `pushingForce` | `float` | `100.0` | 推动力大小 (10.0-100.0) |
| `isEnabled` | `bool` | `true` | 是否启用推动功能 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `CharacterBodyComponent` | **基类依赖** | 提供CharacterBody2D访问 |

### 处理顺序
> **重要**: 此组件应在所有调用`move_and_slide()`的组件之后处理

## 🎯 使用示例

### 基础推动设置

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── PlatformerPhysicsComponent  # 或其他移动组件
#     └── PushRigidBodyComponent     # 必须在最后
```

### 可推动箱子场景

```gdscript
# 创建可推动的箱子实体
# Box.tscn:
# └── Box (RigidBody2D)
#     ├── CollisionShape2D
#     ├── Sprite2D
#     └── RigidBodyBoxComponent (自定义组件)

# 玩家实体设置
# Player.tscn:  
# └── Player (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── PlatformerPhysicsComponent
#     ├── PlatformerControlComponent
#     └── PushRigidBodyComponent (pushingForce = 150.0)
```

### 动态推动力调整

```gdscript
# VariablePushComponent.gd
extends PushRigidBodyComponent

@export var strengthMultiplier: float = 1.0
@export var maxPushForce: float = 300.0
@export var minPushForce: float = 50.0

var playerStrength: float = 1.0

func _ready():
    super._ready()
    updatePushingForce()

func updatePushingForce():
    pushingForce = clamp(
        minPushForce * playerStrength * strengthMultiplier,
        minPushForce,
        maxPushForce
    )

func setPlayerStrength(newStrength: float):
    playerStrength = newStrength
    updatePushingForce()

func addStrength(amount: float):
    setPlayerStrength(playerStrength + amount)

# 使用示例
func _ready():
    super._ready()
    # 根据玩家属性调整推动力
    var statsComponent = parentEntity.findFirstComponentSubclass(StatsComponent)
    if statsComponent:
        var strength = statsComponent.getStat("strength")
        setPlayerStrength(strength.currentValue / 100.0)
```

### 条件推动组件

```gdscript
# ConditionalPushComponent.gd
extends PushRigidBodyComponent

@export var requiresActionKey: bool = false
@export var pushAction: String = "interact"
@export var maxPushMass: float = 50.0
@export var pushCooldown: float = 0.5

var lastPushTime: float = 0.0
var isActionPressed: bool = false

func _ready():
    super._ready()
    if requiresActionKey:
        # 连接输入组件
        var inputComponent = parentEntity.findFirstComponentSubclass(InputComponent)
        if inputComponent:
            Tools.connectSignal(inputComponent.didProcessInput, self.onInputProcessed)

func onInputProcessed(event: InputEvent):
    if event.is_action_pressed(pushAction):
        isActionPressed = true
    elif event.is_action_released(pushAction):
        isActionPressed = false

func _physics_process(delta: float):
    var currentTime = Time.get_time_dict_from_system()
    
    # 检查冷却时间
    if currentTime - lastPushTime < pushCooldown:
        return
    
    # 检查是否需要按键
    if requiresActionKey and not isActionPressed:
        return
    
    # 执行推动逻辑
    for collisionIndex in body.get_slide_collision_count():
        var collision: KinematicCollision2D = body.get_slide_collision(collisionIndex)
        var collider: RigidBody2D = collision.get_collider() as RigidBody2D
        
        if collider and canPushRigidBody(collider):
            pushRigidBody(collision, collider)
            lastPushTime = currentTime

func canPushRigidBody(rigidBody: RigidBody2D) -> bool:
    # 检查质量限制
    if rigidBody.mass > maxPushMass:
        return false
    
    # 检查是否可推动（自定义标签）
    if rigidBody.has_method("canBePushed"):
        return rigidBody.canBePushed()
    
    return true

func pushRigidBody(collision: KinematicCollision2D, rigidBody: RigidBody2D):
    var impulse = -collision.get_normal() * pushingForce
    rigidBody.apply_central_impulse(impulse)
    
    # 可选：添加视觉/音频效果
    showPushEffect(collision.get_position())

func showPushEffect(position: Vector2):
    # 添加推动特效
    print("Pushed object at ", position)
```

### 智能推动系统

```gdscript
# SmartPushComponent.gd
extends PushRigidBodyComponent

@export var pushEfficiency: float = 1.0
@export var enablePushChains: bool = true
@export var maxChainLength: int = 3

var pushedObjects: Array[RigidBody2D] = []
var pushChainDepth: int = 0

func _physics_process(delta: float):
    if not isEnabled:
        return
    
    pushedObjects.clear()
    pushChainDepth = 0
    
    for collisionIndex in body.get_slide_collision_count():
        var collision: KinematicCollision2D = body.get_slide_collision(collisionIndex)
        var collider: RigidBody2D = collision.get_collider() as RigidBody2D
        
        if collider and not collider in pushedObjects:
            processPush(collision, collider)

func processPush(collision: KinematicCollision2D, rigidBody: RigidBody2D):
    # 计算推动效率
    var effectiveForce = calculateEffectiveForce(rigidBody)
    var impulse = -collision.get_normal() * effectiveForce
    
    # 应用推力
    rigidBody.apply_central_impulse(impulse)
    pushedObjects.append(rigidBody)
    
    # 处理推动链
    if enablePushChains and pushChainDepth < maxChainLength:
        checkPushChain(rigidBody, impulse.normalized())

func calculateEffectiveForce(rigidBody: RigidBody2D) -> float:
    # 基于质量和效率计算有效推动力
    var massModifier = 1.0 / sqrt(rigidBody.mass)
    return pushingForce * pushEfficiency * massModifier

func checkPushChain(pushedBody: RigidBody2D, direction: Vector2):
    pushChainDepth += 1
    
    # 检查被推动物体是否会推动其他物体
    var spaceState = pushedBody.get_world_2d().direct_space_state
    var query = PhysicsRayQueryParameters2D.create(
        pushedBody.global_position,
        pushedBody.global_position + direction * 50.0
    )
    
    var result = spaceState.intersect_ray(query)
    if result and result.collider is RigidBody2D:
        var chainTarget = result.collider as RigidBody2D
        if not chainTarget in pushedObjects:
            # 传递减弱的推力
            var chainForce = pushingForce * 0.7 * pushEfficiency
            chainTarget.apply_central_impulse(direction * chainForce)
            pushedObjects.append(chainTarget)
            
            # 递归处理更深层的推动链
            if pushChainDepth < maxChainLength:
                checkPushChain(chainTarget, direction)
```

## 🔧 技术细节

### 推动力计算
```gdscript
func _physics_process(_delta: float) -> void:
    for collisionIndex in body.get_slide_collision_count():
        var collision: KinematicCollision2D = body.get_slide_collision(collisionIndex)
        var collider: RigidBody2D = collision.get_collider() as RigidBody2D
        if collider: 
            collider.apply_central_impulse(-collision.get_normal() * pushingForce)
```

### 性能优化
- **条件处理**: 使用isEnabled setter优化_physics_process调用
- **碰撞检测**: 仅处理实际发生的碰撞
- **类型检查**: 快速过滤非RigidBody2D对象

### 物理原理
- **冲量应用**: 使用`apply_central_impulse()`而非`apply_force()`
- **方向计算**: 基于碰撞法线的反方向
- **力度控制**: 通过pushingForce参数调节

## ⚠️ 注意事项

1. **处理顺序**: 必须在所有`move_and_slide()`调用之后处理
2. **性能考虑**: 每帧检查所有碰撞，大量RigidBody时需要优化
3. **物理设置**: 确保RigidBody2D具有合适的质量和阻尼
4. **碰撞层**: 正确设置碰撞层和碰撞掩码
5. **稳定性**: 过大的推动力可能导致物理不稳定

## 🔗 相关组件

- [CharacterBodyDependentComponentBase](CharacterBodyDependentComponentBase.md) - 基类组件
- [CharacterBodyComponent](CharacterBodyComponent.md) - 角色体组件
- [PlatformerPhysicsComponent](PlatformerPhysicsComponent.md) - 平台物理组件
- [Component](../Component.md) - 基础组件类

## 📚 参考资料

**致谢**: 基于 KidsCanCode@YouTube 的推动系统教程  
**链接**: https://www.youtube.com/watch?v=SJuScDavstM

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 