# Game Objects 游戏对象实体

## 概述
游戏对象实体是 Comedot 框架中用于表示各种游戏对象的实体类型，包括子弹、激光、爆炸效果、闪电等。这些实体通常用于战斗系统、特效系统等。

## 🎯 主要特性
- 🎮 多样化的游戏对象类型
- ⚡ 高性能的渲染和更新
- 🎨 丰富的视觉效果
- 🔧 灵活的配置系统
- 🧹 自动生命周期管理

## 📁 对象类型

### BulletEntity - 子弹实体
**文件：** `Entities/Objects/BulletEntity.gd`

基础子弹实体，用于射击系统。

**主要功能：**
- 直线运动
- 碰撞检测
- 伤害处理
- 自动销毁

**使用示例：**
```gdscript
# 创建子弹实体
var bullet = BulletEntity.new()
bullet.global_position = player.global_position
bullet.velocity = Vector2(100, 0)
add_child(bullet)
```

### LaserEntity - 激光实体
**文件：** `Entities/Objects/LaserEntity.gd`

激光攻击实体，提供直线攻击效果。

**主要功能：**
- 直线激光效果
- 持续伤害
- 穿透攻击
- 视觉效果

**使用示例：**
```gdscript
# 创建激光实体
var laser = LaserEntity.new()
laser.start_position = player.global_position
laser.end_position = target.global_position
laser.damage = 50
add_child(laser)
```

### ExplosionEntity - 爆炸实体
**文件：** `Entities/Objects/ExplosionEntity.gd`

爆炸效果实体，提供爆炸动画和伤害。

**主要功能：**
- 爆炸动画
- 范围伤害
- 冲击波效果
- 自动销毁

**使用示例：**
```gdscript
# 创建爆炸实体
var explosion = ExplosionEntity.new()
explosion.global_position = target.global_position
explosion.radius = 100
explosion.damage = 30
add_child(explosion)
```

### LightningEntity - 闪电实体
**文件：** `Entities/Objects/LightningEntity.gd`

闪电攻击实体，提供链式攻击效果。

**主要功能：**
- 链式攻击
- 多目标伤害
- 闪电视觉效果
- 自动目标选择

**使用示例：**
```gdscript
# 创建闪电实体
var lightning = LightningEntity.new()
lightning.start_position = caster.global_position
lightning.targets = [enemy1, enemy2, enemy3]
lightning.damage = 25
lightning.chain_count = 3
add_child(lightning)
```

### ParabolaFireBallEntity - 抛物线火球实体
**文件：** `Entities/Objects/ParabolaFireBallEntity.gd`

抛物线火球实体，提供弧线攻击效果。

**主要功能：**
- 抛物线运动
- 重力影响
- 落地爆炸
- 轨迹预测

**使用示例：**
```gdscript
# 创建火球实体
var fireball = ParabolaFireBallEntity.new()
fireball.start_position = caster.global_position
fireball.target_position = target.global_position
fireball.speed = 200
fireball.gravity = 500
add_child(fireball)
```

### PortalEntity - 传送门实体
**文件：** `Entities/Objects/PortalEntity.tscn`

传送门实体，提供传送功能。

**主要功能：**
- 传送功能
- 视觉效果
- 触发检测
- 目标设置

**使用示例：**
```gdscript
# 创建传送门
var portal = preload("res://Entities/Objects/PortalEntity.tscn").instantiate()
portal.global_position = Vector2(100, 100)
portal.target_position = Vector2(500, 500)
add_child(portal)
```

## 🔧 通用功能

### 生命周期管理
所有游戏对象实体都遵循标准的生命周期：
1. **创建** - 实例化实体
2. **初始化** - 设置初始状态
3. **更新** - 处理逻辑和渲染
4. **销毁** - 清理资源并移除

### 碰撞检测
- 自动碰撞检测
- 伤害处理
- 效果触发
- 状态更新

### 视觉效果
- 精灵动画
- 粒子效果
- 屏幕特效
- 音效播放

### 性能优化
- 对象池管理
- 自动销毁
- 视距剔除
- 批量更新

## 🎮 使用模式

### 创建模式
```gdscript
# 直接创建
var bullet = BulletEntity.new()

# 从场景创建
var laser = preload("res://Entities/Objects/LaserEntity.tscn").instantiate()

# 从工厂创建
var explosion = EntityFactory.createExplosion(position, damage)
```

### 配置模式
```gdscript
# 基础配置
bullet.damage = 25
bullet.speed = 300
bullet.lifetime = 5.0

# 高级配置
laser.penetration = true
laser.max_targets = 5
laser.chain_damage = 0.8
```

### 管理模式
```gdscript
# 批量管理
var active_bullets = []
for bullet in active_bullets:
    if bullet.should_destroy():
        bullet.queue_free()
        active_bullets.erase(bullet)

# 对象池
var bullet_pool = ObjectPool.new(BulletEntity, 20)
var bullet = bullet_pool.get_object()
```

## 📊 性能考虑

### 内存管理
- 使用对象池减少创建/销毁开销
- 及时清理不需要的对象
- 避免内存泄漏

### 渲染优化
- 使用精灵图集
- 实现视距剔除
- 批量渲染

### 更新优化
- 只在必要时更新
- 使用帧率控制
- 实现LOD系统

## 🛠️ 扩展开发

### 创建新的游戏对象
1. 继承 `Entity` 类
2. 实现必要的生命周期方法
3. 添加特定的功能组件
4. 创建对应的场景文件

### 自定义组件
```gdscript
class_name CustomProjectileComponent
extends Component

@export var speed: float = 200.0
@export var damage: int = 10

func _physics_process(delta: float):
    # 自定义移动逻辑
    parentEntity.global_position += direction * speed * delta
```

## 🔗 相关系统

- **Combat System** - 战斗系统
- **Effect System** - 特效系统
- **Physics System** - 物理系统
- **Audio System** - 音频系统
- **Particle System** - 粒子系统 