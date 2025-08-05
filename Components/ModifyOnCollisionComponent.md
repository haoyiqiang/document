# ModifyOnCollisionComponent API

> **继承关系**: Component > AreaCollisionComponent > ModifyOnCollisionComponent  
> **基于**: Area2D碰撞检测

碰撞修改组件，当Area2D与其他区域或物理体碰撞时自动执行节点和组件的添加、移除操作，适用于子弹击中移除、箭矢插入地面等场景。

## ✨ 主要特性

- 💥 基于碰撞的自动化修改系统
- 🎯 支持实体完全移除
- 🔧 灵活的节点和组件管理
- 📦 支持载荷(Payload)系统
- 🏹 专为抛射物和交互物设计
- 📡 丰富的事件信号系统

## 📊 导出属性

### 修改设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldRemoveEntity` | `bool` | `false` | 是否移除整个实体（会阻止其他修改操作） |
| `nodesToRemove` | `Array[Node]` | `[]` | 要移除的节点列表 |
| `componentsToRemove` | `Array[Script]` | `[]` | 要移除的组件类型列表 |
| `componentsToCreate` | `Array[Script]` | `[]` | 要创建的组件脚本列表 |
| `payload` | `Payload` | `null` | 可选的载荷对象，在最后执行 |

## 📡 信号

```gdscript
signal willRemoveEntity()
signal didAddComponents(components: Array[Component])
```

| 信号 | 触发时机 | 用途 |
|------|----------|------|
| `willRemoveEntity` | 实体即将被移除时 | 执行清理工作或保存状态 |
| `didAddComponents` | 新组件添加完成后 | 配置新添加的组件 |

## 🛠️ 主要方法

### 碰撞处理
```gdscript
func onCollide(collidingNode: Node2D) -> void
```
**描述**: 处理碰撞事件，按顺序执行所有修改操作

## 🔄 执行顺序

当碰撞发生时，按以下顺序执行：

1. **实体移除**: `shouldRemoveEntity` (如果启用，会跳过后续步骤)
2. **节点移除**: `nodesToRemove`
3. **组件移除**: `componentsToRemove`
4. **组件创建**: `componentsToCreate`
5. **载荷执行**: `payload.execute()`

## 🎯 使用示例

### 子弹碰撞移除

```gdscript
# BulletEntity.gd
extends Entity

func _ready():
    setupBulletCollision()

func setupBulletCollision():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 碰撞任何东西都移除子弹
    collisionModifier.shouldRemoveEntity = true
    
    # 可选：添加爆炸效果
    var explosionPayload = preload("res://Resources/Payloads/ExplosionPayload.tres")
    collisionModifier.payload = explosionPayload
```

### 箭矢插入系统

```gdscript
# ArrowEntity.gd
extends Entity

func _ready():
    setupArrowBehavior()

func setupArrowBehavior():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 箭矢击中后的行为
    collisionModifier.shouldRemoveEntity = false
    
    # 移除移动和伤害组件，让箭矢"插入"目标
    collisionModifier.componentsToRemove = [
        LinearMotionComponent,
        DamageComponent
    ]
    
    # 添加定时移除组件，10秒后清理箭矢
    collisionModifier.componentsToCreate = [
        preload("res://Components/Gameplay/ModifyOnTimerComponent.tscn")
    ]
    
    # 连接信号来配置新组件
    collisionModifier.didAddComponents.connect(onArrowStuck)

func onArrowStuck(newComponents: Array[Component]):
    # 配置定时移除组件
    for component in newComponents:
        if component is ModifyOnTimerComponent:
            var timerComponent = component as ModifyOnTimerComponent
            timerComponent.get_node("InternalTimer").wait_time = 10.0
            timerComponent.shouldRemoveEntity = true
```

### 收集品碰撞

```gdscript
# CollectibleEntity.gd
extends Entity

func _ready():
    setupCollectibleBehavior()

func setupCollectibleBehavior():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 被收集后移除实体
    collisionModifier.shouldRemoveEntity = true
    
    # 使用载荷给玩家添加物品
    var collectPayload = CollectiblePayload.new()
    collectPayload.itemToAdd = preload("res://Resources/Items/Coin.tres")
    collectPayload.quantity = 1
    collisionModifier.payload = collectPayload
    
    # 连接移除信号播放音效
    collisionModifier.willRemoveEntity.connect(onBeingCollected)

func onBeingCollected():
    # 播放收集音效
    AudioManager.playSound("collect_coin")
```

### 陷阱激活系统

```gdscript
# TrapEntity.gd
extends Entity

var hasBeenTriggered: bool = false

func _ready():
    setupTrapTrigger()

func setupTrapTrigger():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 不移除陷阱本身
    collisionModifier.shouldRemoveEntity = false
    
    # 移除触发器组件，防止重复触发
    collisionModifier.componentsToRemove = [
        ModifyOnCollisionComponent  # 移除自己
    ]
    
    # 添加伤害和视觉效果组件
    collisionModifier.componentsToCreate = [
        preload("res://Components/Combat/DamageComponent.tscn"),
        preload("res://Components/Visual/BlinkPauseComponent.tscn")
    ]
    
    collisionModifier.didAddComponents.connect(onTrapActivated)

func onTrapActivated(newComponents: Array[Component]):
    hasBeenTriggered = true
    
    # 配置新添加的组件
    for component in newComponents:
        if component is DamageComponent:
            var damageComp = component as DamageComponent
            damageComp.damage = 50
            damageComp.isEnabled = true
```

### 动态环境交互

```gdscript
# InteractiveWallEntity.gd
extends Entity

@export var wallHealth: int = 3

func _ready():
    setupWallDestruction()

func setupWallDestruction():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 不立即移除墙体
    collisionModifier.shouldRemoveEntity = false
    
    # 使用载荷处理墙体损坏
    var wallDamagePayload = WallDamagePayload.new()
    wallDamagePayload.damageAmount = 1
    collisionModifier.payload = wallDamagePayload

# WallDamagePayload.gd
class_name WallDamagePayload
extends Payload

@export var damageAmount: int = 1

func execute(source: Entity, target: Node) -> void:
    if source.has_method("takeDamage"):
        source.takeDamage(damageAmount)
    
    # 如果墙体被摧毁，添加破坏效果
    if source.wallHealth <= 0:
        source.add_child(preload("res://Effects/WallDestructionEffect.tscn").instantiate())
```

### 条件修改系统

```gdscript
# ConditionalModifierEntity.gd
extends Entity

@export var playerLevel: int = 1

func _ready():
    setupConditionalModification()

func setupConditionalModification():
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 重写默认碰撞处理
    collisionModifier.onCollide = customCollisionHandler

func customCollisionHandler(collidingNode: Node2D):
    # 只对玩家实体有效
    if not collidingNode.is_in_group("player"):
        return
    
    var collisionModifier = $ModifyOnCollisionComponent
    
    # 根据玩家等级决定修改内容
    if playerLevel >= 5:
        # 高级玩家获得特殊奖励
        collisionModifier.componentsToCreate = [
            preload("res://Components/Objects/CollectibleComponent.tscn")
        ]
    else:
        # 普通玩家只是移除自己
        collisionModifier.shouldRemoveEntity = true
    
    # 执行默认修改逻辑
    collisionModifier.performAllModifications()
```

## 🏛️ 设计模式

### 命令模式
- **命令对象**: ModifyOnCollisionComponent封装修改操作
- **触发器**: 碰撞事件作为命令执行触发器
- **接收者**: 实体和组件作为命令的接收者

### 责任链模式
- **处理链**: 实体移除 → 节点移除 → 组件移除 → 组件创建 → 载荷执行
- **条件处理**: shouldRemoveEntity会中断后续处理

## 🔧 技术细节

### 碰撞处理流程
```gdscript
func onCollide(collidingNode: Node2D) -> void:
    if not isEnabled: return
    
    super.onCollide(collidingNode)  # 调用基类处理
    
    if shouldRemoveEntity:
        willRemoveEntity.emit()
        requestDeletionOfParentEntity()
    else:
        # 执行修改序列
        performModifications(collidingNode)
```

### 安全的节点移除
```gdscript
for node in nodesToRemove:
    if is_instance_valid(node):
        if is_instance_valid(node.get_parent()): 
            node.get_parent().remove_child(node)
        node.queue_free()
```

### 载荷执行
```gdscript
if payload: 
    payload.execute(self.parentEntity, collidingNode)
```

## ⚠️ 注意事项

1. **继承关系**: 继承自AreaCollisionComponent，具备完整的碰撞检测功能
2. **执行顺序**: shouldRemoveEntity会阻止其他所有修改操作
3. **实体引用**: 在修改过程中保存parentEntity引用，防止组件自身被移除导致的访问问题
4. **有效性检查**: 对节点和实体进行有效性检查，避免运行时错误
5. **载荷参数**: 载荷的source是父实体，target是碰撞节点

## 🔗 相关组件

- [AreaCollisionComponent](AreaCollisionComponent.md) - 基础碰撞检测
- [ModifyOnTimerComponent](../Gameplay/ModifyOnTimerComponent.md) - 定时修改组件
- [NodeModifierComponentBase](../NodeModifierComponentBase.md) - 节点修改基类
- [DamageComponent](../Combat/DamageComponent.md) - 伤害组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 