# KnockbackOnHitComponent API 参考

## 概述

`KnockbackOnHitComponent` 在实体的 [`DamageReceivingComponent`](DamageReceivingComponent.md) 受到伤害时将实体推开。该组件为战斗系统提供触觉反馈，让受击感更加明显。

**继承关系：**
`Component` → `CharacterBodyDependentComponentBase` → `KnockbackOnHitComponent`

## 主要特性

- 💥 **击退效果** - 根据伤害方向推开实体
- 🎯 **方向计算** - 自动计算击退方向
- ⚖️ **力度控制** - 可配置击退强度和额外向量
- 🚀 **速度管理** - 可选择清零当前速度确保效果明显
- 🔧 **组件顺序** - 支持与其他物理组件的正确交互

## 导出属性

### 击退设置
```gdscript
@export_range(0, 1000, 5) var knockbackForce: float = 150.0
```
击退的力度大小，作为伤害方向向量的标量乘数。

```gdscript
@export var damageDirectionScale: Vector2 = Vector2(1, 1)
```
应用到伤害源方向的缩放向量，可用于调整水平/垂直击退比例。

```gdscript
@export var additionalVector: Vector2 = Vector2.ZERO
```
额外应用的固定向量，例如在平台游戏中受伤时的轻微跳跃。

```gdscript
@export var shouldZeroCurrentVelocity: bool = true
```
如果为 `true`，在应用击退前将实体当前速度设为0，确保击退效果总是明显。

```gdscript
@export var isEnabled: bool = true
```
控制组件是否启用。

## 状态属性

### shouldApplyKnockback
```gdscript
var shouldApplyKnockback: bool
```
如果为 `true`，在 `_physics_process` 中调用 `knockback()` 应用最近的伤害方向。

### recentDamageDirection
```gdscript
var recentDamageDirection: Vector2
```
最后一次伤害源的方向，如果 `shouldApplyKnockback` 为真则在 `_physics_process` 中应用。

## 依赖组件

该组件需要以下组件才能正常工作：
- `CharacterBodyComponent`
- `DamageReceivingComponent`

## 主要方法

### knockback()
```gdscript
func knockback(damageDirection: Vector2) -> Vector2
```
应用并返回来自伤害源的击退力。
- **参数：** `damageDirection` - 伤害方向的单位向量
- **返回：** 应用的总击退力向量

## 重要警告

⚠️ **组件顺序重要性：**
如果 `PlatformerMovementParameters` 的 `shouldStopInstantlyOnFloor` 或 `shouldStopInstantlyInAir` 为 `true`，击退可能不会被应用。为解决此问题，此组件应该在实体场景树中位于其他此类组件**之下**。

💡 **推荐配置：**
- 在 `CharacterBodyComponent` 之前
- 在 `DamageReceivingComponent` 之前  
- 在 `PlatformerPhysicsComponent` 之后

## 使用示例

### 基本击退设置
```gdscript
# 在玩家实体场景中配置击退
extends PlayerEntity

func _ready():
    var knockback = $KnockbackOnHitComponent
    
    # 配置击退参数
    knockback.knockbackForce = 200.0
    knockback.damageDirectionScale = Vector2(1.2, 0.8)  # 水平击退更强
    knockback.additionalVector = Vector2(0, -50)  # 轻微向上推
    knockback.shouldZeroCurrentVelocity = true
```

### 平台游戏击退
```gdscript
# 为平台游戏角色设置合适的击退
extends PlatformerEntity

func setup_platformer_knockback():
    var knockback = $KnockbackOnHitComponent
    
    # 平台游戏特化参数
    knockback.knockbackForce = 180.0
    knockback.damageDirectionScale = Vector2(1.0, 0.6)  # 减少垂直击退
    knockback.additionalVector = Vector2(0, -100)  # 较强的向上跳跃
    
    # 确保与跳跃系统配合
    var jump_component = $JumpComponent
    if jump_component:
        # 击退时重置跳跃计数，允许空中恢复
        knockback.damageReceivingComponent.didReceiveDamage.connect(
            func(_damage, _amount, _factions): jump_component.reset_jump_count()
        )
```

### 变量击退系统
```gdscript
# 根据伤害源类型调整击退
extends Component

@export var knockback_profiles = {
    "light": { "force": 100.0, "scale": Vector2(0.8, 0.8) },
    "heavy": { "force": 300.0, "scale": Vector2(1.5, 1.0) },
    "explosion": { "force": 400.0, "scale": Vector2(1.2, 1.2) }
}

func _ready():
    var knockback = $KnockbackOnHitComponent
    var damage_receiving = $DamageReceivingComponent
    
    damage_receiving.didReceiveDamage.connect(_adjust_knockback)

func _adjust_knockback(damage_component: DamageComponent, amount: int, _factions: int):
    var knockback = $KnockbackOnHitComponent
    
    # 根据伤害源调整击退
    var damage_type = damage_component.get_meta("damage_type", "light")
    var profile = knockback_profiles.get(damage_type, knockback_profiles["light"])
    
    knockback.knockbackForce = profile["force"]
    knockback.damageDirectionScale = profile["scale"]
    
    # 根据伤害量调整
    var damage_multiplier = clamp(amount / 10.0, 0.5, 2.0)
    knockback.knockbackForce *= damage_multiplier
```

### 击退防护系统
```gdscript
# 使用VelocityClampComponent防止过度击退
extends Entity

func _ready():
    # 添加速度限制组件
    var velocity_clamp = VelocityClampComponent.new()
    velocity_clamp.maxSpeed = 500.0  # 防止"火箭"效果
    add_child(velocity_clamp)
    
    # 配置击退
    var knockback = $KnockbackOnHitComponent
    knockback.knockbackForce = 250.0
    
    # 在击退后应用速度钳制
    knockback.damageReceivingComponent.didReceiveDamage.connect(
        func(_damage, _amount, _factions):
            await get_tree().process_frame  # 等待击退应用
            velocity_clamp.apply_clamp()
    )
```

### 多方向击退效果
```gdscript
# 创建更复杂的击退效果
extends KnockbackOnHitComponent

func knockback(damageDirection: Vector2) -> Vector2:
    # 调用基础击退
    var base_force = super.knockback(damageDirection)
    
    # 添加随机分散效果
    var scatter = Vector2(
        randf_range(-30, 30),
        randf_range(-20, 20)
    )
    body.velocity += scatter
    
    # 添加旋转效果（如果实体有旋转组件）
    var spin_component = coComponents.get("SpinComponent")
    if spin_component:
        spin_component.add_angular_impulse(randf_range(-180, 180))
    
    return base_force + scatter
```

### 击退连击系统
```gdscript
# 连续击中时增强击退效果
extends Entity

var hit_combo = 0
var combo_timer = 0.0
const COMBO_DECAY_TIME = 2.0

func _ready():
    var knockback = $KnockbackOnHitComponent
    var damage_receiving = $DamageReceivingComponent
    
    damage_receiving.didReceiveDamage.connect(_on_hit_received)

func _process(delta):
    # 连击衰减
    combo_timer -= delta
    if combo_timer <= 0:
        hit_combo = 0

func _on_hit_received(damage_component: DamageComponent, amount: int, factions: int):
    hit_combo += 1
    combo_timer = COMBO_DECAY_TIME
    
    # 根据连击数增强击退
    var knockback = $KnockbackOnHitComponent
    var base_force = knockback.knockbackForce
    knockback.knockbackForce = base_force * (1.0 + hit_combo * 0.2)
    
    # 在下一帧重置
    await get_tree().process_frame
    knockback.knockbackForce = base_force
```

## 技术细节

### 物理处理时机
- 击退在 `_physics_process` 中应用，确保与物理系统同步
- 使用延迟应用机制避免与其他组件的冲突
- 自动设置 `characterBodyComponent.shouldMoveThisFrame = true`

### 方向计算
- 使用实体全局位置计算伤害方向
- 支持方向缩放以适应不同游戏类型
- 击退方向为伤害方向的反向

### 性能优化
- 仅在受到实际伤害时计算击退
- 使用标志位避免不必要的物理计算
- 高效的向量数学运算

## 注意事项

⚠️ **重要限制：**
- 击退可能被其他物理组件抵消，特别是即时停止组件
- 需要正确的组件顺序才能确保效果
- 当前使用实体位置而非碰撞点计算方向

💡 **最佳实践：**
- 配合速度钳制组件防止过度击退
- 为不同游戏类型调整方向缩放
- 测试与其他物理组件的交互

🔮 **计划改进：**
- 支持基于碰撞点的精确方向计算
- 可配置的击退衰减曲线
- 与动画系统的更好集成

## 相关组件

- [`DamageReceivingComponent`](DamageReceivingComponent.md) - 必需依赖，提供伤害接收功能
- [`CharacterBodyComponent`](../Physics/CharacterBodyComponent.md) - 必需依赖，提供物理体
- [`PlatformerPhysicsComponent`](../Physics/PlatformerPhysicsComponent.md) - 常用搭配，可能需要调整顺序
- [`VelocityClampComponent`](../Movement/VelocityClampComponent.md) - 推荐搭配，防止过度击退
- [`InvulnerabilityOnHitComponent`](InvulnerabilityOnHitComponent.md) - 常用搭配的受击效果

## 故障排除

**问题：击退效果不明显**
- 增加 `knockbackForce` 值
- 启用 `shouldZeroCurrentVelocity`
- 检查其他组件是否抵消了击退

**问题：实体被"射向太空"**
- 降低 `knockbackForce` 值
- 添加 `VelocityClampComponent`
- 调整 `damageDirectionScale`

**问题：击退方向错误**
- 检查伤害源和接收者的位置
- 验证 `damageDirectionScale` 设置
- 考虑使用 `additionalVector` 进行补偿 