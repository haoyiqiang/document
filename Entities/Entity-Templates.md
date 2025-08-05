# Entity Templates 实体模板

## 概述
实体模板是 Comedot 框架中预定义的实体配置，用于快速创建常见的游戏对象。这些模板提供了标准化的组件组合和配置，简化了实体创建过程。

## 🎯 主要特性
- 📋 预定义的组件组合
- ⚡ 快速实体创建
- 🔧 标准化配置
- 🎮 常见游戏对象支持
- 📚 可扩展的模板系统

## 📁 模板位置
实体模板位于 `/Templates/Entities/` 目录下，包含各种常见游戏对象的模板。

## 🏗️ 模板类型

### 基础实体模板

#### AreaEntityTemplate.tscn
区域实体模板，用于创建具有碰撞检测的实体。

**包含组件：**
- AreaCollisionComponent
- FactionComponent
- DamageComponent

**使用场景：**
- 陷阱
- 传送门
- 收集品
- 触发器

#### BulletEntityTemplate.tscn
子弹实体模板，用于创建投射物。

**包含组件：**
- LinearMotionComponent
- DamageComponent
- OffscreenRemovalComponent
- FactionComponent

**使用场景：**
- 子弹
- 箭矢
- 魔法弹
- 飞镖

#### CharacterBodyEntityTemplate.tscn
角色体实体模板，用于创建具有物理碰撞的实体。

**包含组件：**
- CharacterBodyComponent
- PlatformerPhysicsComponent
- HealthComponent
- FactionComponent

**使用场景：**
- 玩家角色
- NPC
- 敌人
- 动物

### 特殊实体模板

#### HUDTemplate.tscn
HUD模板，用于创建用户界面元素。

**包含组件：**
- LabelComponent
- HealthVisualComponent
- StatsVisualComponent

**使用场景：**
- 生命值显示
- 状态栏
- 信息面板
- 提示框

#### RectangleAreaTemplate.tscn
矩形区域模板，用于创建矩形碰撞区域。

**包含组件：**
- AreaCollisionComponent
- ShapeDrawComponent

**使用场景：**
- 检测区域
- 安全区域
- 限制区域
- 交互区域

### 场景模板

#### OverheadSceneTemplate.tscn
俯视角场景模板，适用于俯视角游戏。

**包含组件：**
- CameraComponent
- OverheadControlComponent
- OverheadPhysicsComponent

**使用场景：**
- 俯视角RPG
- 策略游戏
- 模拟游戏
- 探索游戏

#### PlatformerSceneTemplate.tscn
平台游戏场景模板，适用于平台跳跃游戏。

**包含组件：**
- CameraComponent
- PlatformerControlComponent
- PlatformerPhysicsComponent

**使用场景：**
- 平台跳跃
- 动作冒险
- 解谜游戏
- 跑酷游戏

## 🎮 使用模式

### 直接使用模板
```gdscript
# 加载模板
var bullet_template = preload("res://Templates/Entities/BulletEntityTemplate.tscn")
var bullet = bullet_template.instantiate()

# 配置实体
bullet.global_position = player.global_position
bullet.velocity = Vector2(100, 0)

# 添加到场景
add_child(bullet)
```

### 模板工厂模式
```gdscript
# 创建实体工厂
class_name EntityFactory

static func createBullet(position: Vector2, velocity: Vector2) -> Entity:
    var template = preload("res://Templates/Entities/BulletEntityTemplate.tscn")
    var bullet = template.instantiate()
    bullet.global_position = position
    bullet.velocity = velocity
    return bullet

static func createPlayer(position: Vector2) -> Entity:
    var template = preload("res://Templates/Entities/CharacterBodyEntityTemplate.tscn")
    var player = template.instantiate()
    player.global_position = position
    return player
```

### 模板继承
```gdscript
# 基于模板创建自定义实体
class_name CustomBulletEntity
extends Entity

func _ready():
    super._ready()
    
    # 添加自定义组件
    addComponent(CustomTrailComponent.new())
    addComponent(CustomSoundComponent.new())
    
    # 配置自定义属性
    var trail_component = getComponent(CustomTrailComponent)
    trail_component.trail_color = Color.RED
```

## 🔧 模板配置

### 组件配置
每个模板都包含预配置的组件，可以通过以下方式修改：

```gdscript
# 获取并配置组件
var health_component = entity.getComponent(HealthComponent)
if health_component:
    health_component.max_health = 100
    health_component.current_health = 100

var movement_component = entity.getComponent(MovementComponent)
if movement_component:
    movement_component.speed = 200
    movement_component.acceleration = 500
```

### 导出属性配置
模板中的导出属性可以在编辑器中直接配置：

```gdscript
# 在编辑器中配置
@export var bullet_speed: float = 300.0
@export var bullet_damage: int = 25
@export var bullet_lifetime: float = 5.0
```

### 运行时配置
```gdscript
# 运行时动态配置
func configureBullet(bullet: Entity, damage: int, speed: float):
    var damage_component = bullet.getComponent(DamageComponent)
    if damage_component:
        damage_component.damage = damage
    
    var motion_component = bullet.getComponent(LinearMotionComponent)
    if motion_component:
        motion_component.speed = speed
```

## 📚 创建自定义模板

### 步骤1：设计组件组合
```gdscript
# 确定需要的组件
var required_components = [
    HealthComponent,
    MovementComponent,
    InputComponent,
    CameraComponent
]
```

### 步骤2：创建场景文件
1. 创建新的场景文件
2. 添加 Entity 根节点
3. 添加所需的组件节点
4. 配置组件属性

### 步骤3：配置导出属性
```gdscript
# 在模板脚本中定义导出属性
@export var template_name: String = "CustomTemplate"
@export var default_health: int = 100
@export var default_speed: float = 200.0
```

### 步骤4：创建工厂方法
```gdscript
# 创建模板工厂方法
static func createCustomEntity(config: Dictionary) -> Entity:
    var template = preload("res://Templates/Entities/CustomTemplate.tscn")
    var entity = template.instantiate()
    
    # 应用配置
    var health_component = entity.getComponent(HealthComponent)
    if health_component:
        health_component.max_health = config.get("health", 100)
    
    return entity
```

## 🎯 最佳实践

### 模板设计原则
1. **单一职责** - 每个模板专注于特定类型的实体
2. **可配置性** - 提供足够的导出属性供配置
3. **可扩展性** - 允许添加额外的组件
4. **性能优化** - 只包含必要的组件

### 使用建议
1. **选择合适的模板** - 根据实体类型选择最合适的模板
2. **配置而非修改** - 优先使用配置而非修改模板
3. **工厂模式** - 使用工厂方法统一创建实体
4. **文档化** - 为自定义模板编写文档

### 性能考虑
1. **组件数量** - 控制模板中的组件数量
2. **懒加载** - 使用懒加载减少初始化时间
3. **对象池** - 对频繁创建的实体使用对象池
4. **资源管理** - 及时释放不需要的资源

## 🔗 相关系统

- **Component System** - 组件系统
- **Scene Management** - 场景管理
- **Entity Factory** - 实体工厂
- **Object Pool** - 对象池系统 