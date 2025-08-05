# GunRecoilComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > GunRecoilComponent  
> **物理类型**: 武器后坐力

当GunComponent发射子弹时，对父实体的CharacterBody2D施加反向击退力的组件。模拟真实的武器后坐力效果，建议配合VelocityClampComponent防止过度反冲。

## ✨ 主要特性

- 🔫 自动连接GunComponent射击事件
- ⬅️ 基于子弹方向的反向后坐力
- 🎛️ 可调节的后坐力强度
- 🔄 可启用/禁用功能
- ⚡ 瞬时力施加系统

## 📊 导出属性

### 后坐力设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `knockbackForce` | `float` | `150.0` | 后坐力强度 (0-1000) |
| `isEnabled` | `bool` | `true` | 是否启用后坐力 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `CharacterBodyComponent` | **基类依赖** | 提供CharacterBody2D访问 |
| `GunComponent` | **必需** | 提供射击事件 |

### 组件属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `gunComponent` | `GunComponent` | 对武器组件的引用 |

## 🎯 使用示例

### 基础武器后坐力

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── GunComponent
#     ├── GunRecoilComponent (knockbackForce: 150)
#     └── VelocityClampComponent  # 推荐配合使用
```

### 可变后坐力组件

```gdscript
# VariableRecoilComponent.gd
extends GunRecoilComponent

@export var lightWeaponForce: float = 100.0
@export var heavyWeaponForce: float = 300.0
@export var weaponType: String = "light"

func _ready():
    super._ready()
    updateRecoilForce()

func updateRecoilForce():
    match weaponType:
        "light":
            knockbackForce = lightWeaponForce
        "heavy":
            knockbackForce = heavyWeaponForce
        "medium":
            knockbackForce = (lightWeaponForce + heavyWeaponForce) / 2

func onGunComponentDidFire(bullet: Entity) -> void:
    if not isEnabled: 
        return
    
    # 根据弹药类型调整后坐力
    var bulletType = getBulletType(bullet)
    var forceMultiplier = getBulletForceMultiplier(bulletType)
    
    var forceVector = Vector2.from_angle(bullet.global_rotation) * -1
    body.velocity += forceVector * knockbackForce * forceMultiplier
    characterBodyComponent.shouldMoveThisFrame = true
    
    # 添加后坐力效果
    addRecoilEffect()

func getBulletType(bullet: Entity) -> String:
    # 从子弹实体获取类型信息
    var bulletComponent = bullet.findFirstComponentSubclass(LinearMotionComponent)
    if bulletComponent and bulletComponent.has_meta("bullet_type"):
        return bulletComponent.get_meta("bullet_type")
    return "normal"

func getBulletForceMultiplier(bulletType: String) -> float:
    match bulletType:
        "explosive": return 2.0
        "piercing": return 0.8
        "rapid": return 0.5
        _: return 1.0

func addRecoilEffect():
    # 添加屏幕震动或音效
    print("Weapon recoil!")
```

### 智能后坐力控制

```gdscript
# SmartRecoilComponent.gd
extends GunRecoilComponent

@export var maxRecoilAccumulation: float = 500.0
@export var recoilDecayRate: float = 200.0
@export var reducedMobilityThreshold: float = 300.0

var currentRecoilAccumulation: float = 0.0
var isStabilized: bool = false

func _ready():
    super._ready()

func _physics_process(delta: float):
    # 后坐力衰减
    if currentRecoilAccumulation > 0:
        currentRecoilAccumulation -= recoilDecayRate * delta
        currentRecoilAccumulation = max(0, currentRecoilAccumulation)
    
    # 检查稳定状态
    updateStabilization()

func onGunComponentDidFire(bullet: Entity) -> void:
    if not isEnabled:
        return
    
    # 计算当前后坐力效果
    var recoilReduction = calculateRecoilReduction()
    var effectiveForce = knockbackForce * recoilReduction
    
    var forceVector = Vector2.from_angle(bullet.global_rotation) * -1
    body.velocity += forceVector * effectiveForce
    characterBodyComponent.shouldMoveThisFrame = true
    
    # 累积后坐力
    currentRecoilAccumulation += effectiveForce
    currentRecoilAccumulation = min(maxRecoilAccumulation, currentRecoilAccumulation)

func calculateRecoilReduction() -> float:
    # 检查是否有稳定技能或装备
    if isStabilized:
        return 0.7  # 30%减少
    
    # 检查是否蹲下或静止
    if isPlayerCrouching() or isPlayerStationary():
        return 0.8  # 20%减少
    
    return 1.0

func updateStabilization():
    # 检查后坐力累积是否影响移动
    if currentRecoilAccumulation > reducedMobilityThreshold:
        reducePlayerMobility()
    else:
        restorePlayerMobility()

func isPlayerCrouching() -> bool:
    # 检查玩家是否蹲下
    return Input.is_action_pressed("crouch")

func isPlayerStationary() -> bool:
    # 检查玩家是否静止
    return body.velocity.length() < 10.0

func reducePlayerMobility():
    # 降低玩家移动速度
    var platformerPhysics = parentEntity.findFirstComponentSubclass(PlatformerPhysicsComponent)
    if platformerPhysics:
        platformerPhysics.runSpeed *= 0.7

func restorePlayerMobility():
    # 恢复正常移动速度
    var platformerPhysics = parentEntity.findFirstComponentSubclass(PlatformerPhysicsComponent)
    if platformerPhysics:
        platformerPhysics.runSpeed /= 0.7
```

### 双重武器后坐力

```gdscript
# DualWeaponRecoilComponent.gd
extends GunRecoilComponent

@export var primaryGunForce: float = 150.0
@export var secondaryGunForce: float = 200.0

var primaryGun: GunComponent
var secondaryGun: GunComponent

func _ready():
    # 找到多个武器组件
    findMultipleGuns()
    connectGunSignals()

func findMultipleGuns():
    var guns = parentEntity.findComponentsSubclass(GunComponent)
    if guns.size() >= 1:
        primaryGun = guns[0]
    if guns.size() >= 2:
        secondaryGun = guns[1]

func connectGunSignals():
    if primaryGun:
        primaryGun.didFire.connect(_on_primary_gun_fired)
    if secondaryGun:
        secondaryGun.didFire.connect(_on_secondary_gun_fired)

func _on_primary_gun_fired(bullet: Entity):
    applyRecoil(bullet, primaryGunForce)

func _on_secondary_gun_fired(bullet: Entity):
    applyRecoil(bullet, secondaryGunForce)

func applyRecoil(bullet: Entity, force: float):
    if not isEnabled:
        return
    
    var forceVector = Vector2.from_angle(bullet.global_rotation) * -1
    body.velocity += forceVector * force
    characterBodyComponent.shouldMoveThisFrame = true
```

## 🔧 技术细节

### 后坐力计算
```gdscript
func onGunComponentDidFire(bullet: Entity) -> void:
    var forceVector: Vector2 = Vector2.from_angle(bullet.global_rotation)
    forceVector = forceVector * -1  # 反向
    body.velocity += forceVector * knockbackForce
    characterBodyComponent.shouldMoveThisFrame = true
```

### 自动连接系统
```gdscript
func _ready() -> void:
    if gunComponent: 
        gunComponent.didFire.connect(self.onGunComponentDidFire)
    else: 
        printWarning("No GunComponent found")
```

## ⚠️ 注意事项

1. **VelocityClampComponent推荐**: 防止快速射击时的"火箭效应"
2. **GunComponent依赖**: 需要父实体包含GunComponent
3. **物理处理**: 使用瞬时速度添加而非力
4. **动态检测**: 当前为静态检测，未来版本将支持动态添加移除
5. **子弹方向**: 基于子弹的global_rotation计算反向力

## 🔗 相关组件

- [GunComponent](../Combat/GunComponent.md) - 武器射击组件
- [VelocityClampComponent](VelocityClampComponent.md) - 速度限制组件
- [CharacterBodyDependentComponentBase](CharacterBodyDependentComponentBase.md) - 基类组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 