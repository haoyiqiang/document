# BulletModifierComponent API

## 概述
`BulletModifierComponent` 用于修改由`GunComponent`发射的"子弹"（或其他抛射物）实体。可用作增强伤害的buff或添加额外的视觉效果等。

**继承链**：`Component` → `BulletModifierComponent`

## 依赖要求
- **GunComponent**：必须在GunComponent之后初始化

## 主要特性
- 💥 **伤害修改**：增加或减少子弹伤害
- 🔧 **组件管理**：动态移除和添加组件到子弹
- 🎯 **自动触发**：监听GunComponent的发射事件
- 📡 **事件通知**：发出子弹修改完成信号

## 导出属性

### 修改配置
- **damageModifier** (`int = 0`, -1000到1000)：伤害修改值，添加到子弹的基础伤害
- **componentsToRemove** (`Array[Script]`)：要从子弹上移除的组件类型列表
- **componentsToCreate** (`Array[Script]`)：要添加到子弹上的组件类型列表
- **isEnabled** (`bool = true`)：是否启用子弹修改功能

## 信号
- **didModifyBullet**(bullet: Entity)：当子弹被修改后发出

## 主要方法
- **onGunComponent_didFire**(bullet: Entity) → `void`：处理Gun组件发射事件的回调

## 使用示例

### 基础伤害提升
```gdscript
# 设置基本的伤害提升
@export var bullet_modifier: BulletModifierComponent

func _ready():
    bullet_modifier.damageModifier = 10  # 增加10点伤害
    bullet_modifier.didModifyBullet.connect(_on_bullet_modified)

func _on_bullet_modified(bullet: Entity):
    print(f"子弹已强化，额外伤害: {bullet_modifier.damageModifier}")
```

### 子弹特效增强
```gdscript
# 为子弹添加特殊效果组件
@export var bullet_modifier: BulletModifierComponent

func setup_explosive_bullets():
    bullet_modifier.damageModifier = 5
    bullet_modifier.componentsToCreate = [
        ExplosionOnImpactComponent,
        AreaDamageComponent,
        ExplosionVisualComponent
    ]
```

### 条件性子弹修改
```gdscript
# 基于玩家状态的动态修改
@export var bullet_modifier: BulletModifierComponent
@export var player_stats: PlayerStats

func update_bullet_modification():
    # 根据玩家等级调整伤害
    bullet_modifier.damageModifier = player_stats.level * 2
    
    # 高级玩家的子弹有特殊效果
    if player_stats.level >= 10:
        bullet_modifier.componentsToCreate = [FireTrailComponent]
    else:
        bullet_modifier.componentsToCreate = []
```

### 武器升级系统
```gdscript
# 武器升级影响子弹属性
class_name WeaponUpgradeSystem
extends Node

@export var bullet_modifier: BulletModifierComponent
var current_upgrades: Array[WeaponUpgrade] = []

func apply_upgrade(upgrade: WeaponUpgrade):
    current_upgrades.append(upgrade)
    recalculate_bullet_modifications()

func recalculate_bullet_modifications():
    var total_damage_bonus = 0
    var components_to_add = []
    
    for upgrade in current_upgrades:
        total_damage_bonus += upgrade.damage_bonus
        components_to_add.append_array(upgrade.additional_components)
    
    bullet_modifier.damageModifier = total_damage_bonus
    bullet_modifier.componentsToCreate = components_to_add
```

### 临时增益效果
```gdscript
# 临时的子弹增强效果
@export var bullet_modifier: BulletModifierComponent

func apply_power_up(duration: float, damage_bonus: int, special_components: Array[Script] = []):
    # 保存原始值
    var original_damage = bullet_modifier.damageModifier
    var original_components = bullet_modifier.componentsToCreate.duplicate()
    
    # 应用增益
    bullet_modifier.damageModifier += damage_bonus
    bullet_modifier.componentsToCreate.append_array(special_components)
    
    # 设置恢复定时器
    var timer = Timer.new()
    timer.wait_time = duration
    timer.one_shot = true
    timer.timeout.connect(func():
        bullet_modifier.damageModifier = original_damage
        bullet_modifier.componentsToCreate = original_components
        timer.queue_free()
    )
    add_child(timer)
    timer.start()
```

## 设计模式

### 修改器栈
```gdscript
# 多个修改器的组合系统
class_name BulletModifierStack
extends Node

var modifiers: Array[BulletModifierComponent] = []

func add_modifier(modifier: BulletModifierComponent):
    modifiers.append(modifier)
    recalculate_total_modifications()

func recalculate_total_modifications():
    var total_damage = 0
    var all_components_to_create = []
    var all_components_to_remove = []
    
    for modifier in modifiers:
        if modifier.isEnabled:
            total_damage += modifier.damageModifier
            all_components_to_create.append_array(modifier.componentsToCreate)
            all_components_to_remove.append_array(modifier.componentsToRemove)
    
    # 应用到主修改器
    var main_modifier = modifiers[0] if modifiers.size() > 0 else null
    if main_modifier:
        main_modifier.damageModifier = total_damage
        main_modifier.componentsToCreate = all_components_to_create
        main_modifier.componentsToRemove = all_components_to_remove
```

## 技术细节

### 处理顺序
1. 监听GunComponent.didFire信号
2. 修改子弹的DamageComponent.damageOnCollision
3. 移除指定组件（componentsToRemove）
4. 创建新组件（componentsToCreate）
5. 发出didModifyBullet信号

### 组件操作
- 使用Entity.removeComponents()移除组件
- 使用Entity.createNewComponents()创建组件
- 操作按顺序执行：先移除，后创建

## 注意事项

### 组件依赖
- 必须在GunComponent之后初始化
- 确保GunComponent存在并可访问
- 监听正确的信号连接

### 性能考虑
- 每次发射都会触发修改
- 大量组件创建/移除可能影响性能
- 考虑使用对象池优化

### 安全检查
- 验证子弹实体的有效性
- 确保DamageComponent存在
- 处理组件创建失败的情况

## 相关组件
- [`GunComponent`](./GunComponent.md) - 武器射击
- [`DamageComponent`](./DamageComponent.md) - 伤害系统
- [`Entity`](../../Entities/Entity.md) - 实体系统 