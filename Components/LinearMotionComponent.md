# LinearMotionComponent API 文档

## 概述
`LinearMotionComponent` 使实体沿着实体 `rotation` 方向的直线移动。

**注意**：直接设置父实体的position；不使用CharacterBody2D.velocity等"物理"系统。适用于子弹和类似抛射物以提高性能。

## 继承
`LinearMotionComponent` → `Component` → `Node`

## 主要特性
- 高性能直线运动
- 支持加速度和摩擦力
- 最大距离限制
- 自动停止或删除实体
- 适合抛射物和子弹

## 导出属性

### 速度设置

#### `initialSpeed: float = 150`
速度的初始值（范围：-2500到2500）。

#### `maximumSpeed: float = 800`
加速时的最大速度限制（范围：-2500到2500）。

#### `minimumSpeed: float = 10`
减速时的最小速度限制（范围：-2500到2500）。

### 速度修正

#### `shouldApplyModifier: bool = true`
是否每帧对速度应用加速度/摩擦力修正器。

#### `modifier: float = 800`
每帧添加到速度的加速度（正值）或摩擦力（负值）（范围：-5000到5000）。

### 距离控制

#### `shouldStopAtMaximumDistance: bool = false`
是否在达到最大距离时停止移动。

#### `shouldDeleteParentAtMaximumDistance: bool = false`
是否在达到最大距离时删除父实体。

#### `maximumDistance: float = 200`
最大移动距离（范围：50到2000）。

#### `isEnabled: bool = true`
是否启用组件。

## 状态属性

#### `speed: float`
当前移动速度。在 `_ready` 时设置为 `initialSpeed`。

#### `distanceTraveled: float`
已移动的总距离。

#### `isMoving: bool`
是否正在移动。达到最大距离且 `shouldStopAtMaximumDistance` 为true时设为false。

## 信号

### `didReachMaximumDistance`
达到最大距离时发出。在更新标志后、删除实体前发出。

## 主要方法

无公共方法（运动在 `_physics_process` 中自动处理）。

## 使用示例

### 基本子弹
```gdscript
# 设置线性运动组件
var motionComponent = LinearMotionComponent.new()
bulletEntity.add_child(motionComponent)

# 配置基本移动
motionComponent.initialSpeed = 500
motionComponent.shouldDeleteParentAtMaximumDistance = true
motionComponent.maximumDistance = 1000

# 设置子弹方向
bulletEntity.rotation = deg_to_rad(45)  # 45度角
```

### 带加速度的火箭
```gdscript
# 火箭逐渐加速
motionComponent.initialSpeed = 100
motionComponent.maximumSpeed = 800
motionComponent.modifier = 400  # 正值=加速
motionComponent.shouldApplyModifier = true

# 无距离限制（飞到屏幕外）
motionComponent.shouldStopAtMaximumDistance = false
motionComponent.shouldDeleteParentAtMaximumDistance = false
```

### 减速箭矢
```gdscript
# 箭矢逐渐减速直到停下
motionComponent.initialSpeed = 400
motionComponent.minimumSpeed = 0
motionComponent.modifier = -200  # 负值=摩擦/减速
motionComponent.shouldApplyModifier = true

# 停在原地而不删除
motionComponent.shouldStopAtMaximumDistance = false
```

### 激光束（固定距离）
```gdscript
# 激光束移动固定距离后消失
motionComponent.initialSpeed = 1000
motionComponent.shouldApplyModifier = false  # 恒定速度
motionComponent.shouldDeleteParentAtMaximumDistance = true
motionComponent.maximumDistance = 800
```

### 信号处理
```gdscript
# 连接距离到达信号
motionComponent.didReachMaximumDistance.connect(_on_max_distance_reached)

func _on_max_distance_reached():
    print("抛射物到达最大距离")
    # 播放爆炸效果
    create_explosion_effect()
    # 造成区域伤害
    apply_area_damage()
```

### 动态控制
```gdscript
# 运行时修改参数
func boost_projectile():
    motionComponent.modifier = 1000  # 突然加速
    motionComponent.maximumSpeed = 1500

func apply_drag():
    motionComponent.modifier = -300  # 应用阻力

func stop_projectile():
    motionComponent.isMoving = false  # 立即停止
```

## 运动计算

### 方向计算
```gdscript
# 组件内部使用的方向计算
var direction: Vector2 = Vector2.RIGHT.rotated(parentEntity.rotation)
```

### 速度修正
```gdscript
# 每帧的速度更新（如果shouldApplyModifier为true）
speed += modifier * delta
speed = clampf(speed, minimumSpeed, maximumSpeed)
```

### 移动计算
```gdscript
# 每帧的位置更新
var offset: Vector2 = direction * (speed * delta)
parentEntity.position += offset
distanceTraveled += offset.length()
```

## 性能优化

### 条件处理
```gdscript
# 组件自动优化处理
# 停止移动时禁用_physics_process
self.set_process(isEnabled and isMoving)
```

### 批量抛射物
```gdscript
# 创建多个抛射物时的优化
func fire_shotgun():
    for i in range(5):
        var bullet = BulletScene.instantiate()
        bullet.rotation = base_rotation + (i - 2) * deg_to_rad(15)
        
        var motion = bullet.get_node("LinearMotionComponent")
        motion.initialSpeed = 400
        motion.maximumDistance = 600
```

## 物理考虑

### 与物理系统的关系
- **不使用**：CharacterBody2D.velocity
- **不使用**：RigidBody2D物理
- **直接修改**：Node2D.position
- **性能优势**：避免物理计算开销

### 碰撞检测
```gdscript
# 需要单独实现碰撞检测
# LinearMotionComponent只处理移动，不处理碰撞

# 配合AreaCollisionComponent使用
var areaComponent = AreaCollisionComponent.new()
bulletEntity.add_child(areaComponent)
areaComponent.collisionDetected.connect(_on_bullet_hit)

func _on_bullet_hit(other_entity):
    # 停止移动
    motionComponent.isMoving = false
    # 处理碰撞
    handle_impact(other_entity)
```

## 距离限制模式

### 模式1：停止移动
```gdscript
motionComponent.shouldStopAtMaximumDistance = true
motionComponent.shouldDeleteParentAtMaximumDistance = false
# 结果：抛射物停在原地
```

### 模式2：删除实体
```gdscript
motionComponent.shouldStopAtMaximumDistance = false
motionComponent.shouldDeleteParentAtMaximumDistance = true
# 结果：抛射物消失
```

### 模式3：无限制
```gdscript
motionComponent.shouldStopAtMaximumDistance = false
motionComponent.shouldDeleteParentAtMaximumDistance = false
# 结果：抛射物持续移动
```

### 模式4：停止并删除
```gdscript
motionComponent.shouldStopAtMaximumDistance = true
motionComponent.shouldDeleteParentAtMaximumDistance = true
# 结果：先停止移动，然后在信号处理中删除
```

## 注意事项

1. **方向设置**：移动方向基于实体的rotation属性
2. **性能考虑**：适合大量简单抛射物
3. **物理互动**：不与物理系统交互
4. **碰撞处理**：需要额外的碰撞检测组件
5. **帧率依赖**：使用delta时间确保帧率无关

## 适用场景

### 最佳使用场景
- 子弹和抛射物
- 激光束
- 简单的移动平台
- 特效粒子

### 不适用场景
- 需要物理碰撞的对象
- 复杂轨迹运动
- 重力影响的物体
- 玩家控制的角色

## 相关组件
- `DamageComponent` - 抛射物伤害
- `AreaCollisionComponent` - 碰撞检测
- `OffscreenRemovalComponent` - 屏幕外清理
- `WaveMotionComponent` - 波浪运动 