# DamageRayComponent API 参考

## 概述

`DamageRayComponent` 是 [`DamageComponent`](DamageComponent.md) 的子类，使用 `RayCast2D` 而非 `Area2D` 进行物理碰撞检测。该组件防止单个抛射物在同一物理帧内与多个重叠的 [`DamageReceivingComponent`](DamageReceivingComponent.md) 碰撞，更适用于箭矢和小型子弹等抛射物。

**继承关系：**
`Component` → `DamageComponent` → `DamageRayComponent`

## 主要特性

- 🎯 **精确碰撞** - 基于RayCast2D的射线检测
- 🚫 **单次命中** - 防止单帧多重碰撞
- 🔍 **智能搜索** - 可配置的伤害接收者搜索
- ⚠️ **实验性功能** - 性能影响待评估

## 导出属性

### 搜索设置
```gdscript
@export var shouldSearchForDamageReceiver: bool = true
```
如果为 `true` 且射线在同一帧碰撞多个物理对象，组件会继续搜索直到找到 `DamageReceivingComponent`。如果为 `false` 则只报告第一个碰撞对象。

⚠️ **警告：** 启用此选项可能影响性能。

## 状态属性

### selfAsRayCast
```gdscript
var selfAsRayCast: RayCast2D
```
对自身作为 `RayCast2D` 的引用。

### recentCollidingObject
```gdscript
var recentCollidingObject: Object
```
最近碰撞的对象，作为类属性存储以避免每次更新时重新创建。

## 主要方法

### _physics_process()
```gdscript
func _physics_process(_delta: float) -> void
```
实验性的物理处理方法：
- 持续扫描射线碰撞
- 搜索 `DamageReceivingComponent`
- 处理碰撞排除逻辑

## 使用示例

### 基本射线伤害设置
```gdscript
# 为子弹实体添加射线伤害检测
extends BulletEntity

func _ready():
    # 确保此实体是RayCast2D
    var ray_damage = $DamageRayComponent
    
    # 配置射线参数
    enabled = true
    target_position = Vector2(100, 0)  # 射线长度和方向
    
    # 配置伤害搜索
    ray_damage.shouldSearchForDamageReceiver = true
    ray_damage.damageOnCollision = 25
```

### 狙击步枪射击
```gdscript
# 创建长距离精确射击
extends SniperBullet

func _ready():
    var ray_damage = $DamageRayComponent
    
    # 长距离射线
    target_position = Vector2(2000, 0)
    
    # 高伤害，精确命中
    ray_damage.damageOnCollision = 100
    ray_damage.hitChance = 95
    ray_damage.shouldSearchForDamageReceiver = true
    
    # 配置碰撞层
    collision_mask = 1  # 只检测敌人层
```

## 技术细节

### 射线碰撞机制
- 使用 `RayCast2D.is_colliding()` 检测碰撞
- 通过 `get_collider()` 获取碰撞对象
- 支持 `add_exception()` 排除特定对象

### 搜索算法
- 遍历射线路径上的所有碰撞对象
- 查找第一个 `DamageReceivingComponent`
- 自动排除非目标对象

## 注意事项

⚠️ **性能警告：**
- 启用 `shouldSearchForDamageReceiver` 可能影响性能
- 复杂场景中的射线计算开销较大

💡 **使用建议：**
- 适用于需要精确命中的抛射物
- 避免在高频武器上使用搜索功能

## 相关组件

- [`DamageComponent`](DamageComponent.md) - 父类，提供基础伤害功能
- [`DamageReceivingComponent`](DamageReceivingComponent.md) - 目标组件
- [`BulletModifierComponent`](BulletModifierComponent.md) - 可用于修改射线伤害参数 