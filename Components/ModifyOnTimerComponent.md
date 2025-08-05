# ModifyOnTimerComponent API

> **继承关系**: Component > NodeModifierComponentBase > ModifyOnTimerComponent

定时修改组件，在Timer超时后按顺序执行节点和组件的添加、移除操作，适用于定时清理、自毁、状态变化等场景。

## ✨ 主要特性

- ⏰ 基于Timer的自动化修改系统
- 🔄 顺序执行多种修改操作
- 🏗️ 支持组件和节点的添加/移除
- 💥 可实现实体自毁功能
- 🎯 与碰撞系统配合使用
- 🔧 灵活的Timer配置选项

## 📊 导出属性

继承自 [NodeModifierComponentBase](../NodeModifierComponentBase.md) 的所有属性：

| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldRemoveEntity` | `bool` | `false` | 是否移除整个实体 |
| `nodesToRemove` | `Array[Node]` | `[]` | 要移除的节点列表 |
| `componentsToRemove` | `Array[Script]` | `[]` | 要移除的组件类型列表 |
| `componentsToCreate` | `Array[PackedScene]` | `[]` | 要创建的组件场景列表 |

## 🛠️ 主要方法

### Timer超时处理
```gdscript
func onInternalTimer_timeout() -> void
```
**描述**: 内部Timer超时时的处理方法，执行所有修改操作

## 🔧 内置Timer配置

默认的 `$InternalTimer` 子节点：
- **默认等待时间**: 3秒
- **执行顺序**: 
  1. `removeEntity()` (如果 `shouldRemoveEntity` 为 true)
  2. `removeNodes()`
  3. `removeComponents()`
  4. `createComponents()`

## 🎯 使用示例

### 基础定时自毁

```gdscript
# TimedExplosive.gd
extends Entity

func _ready():
    setupTimedDestruction()

func setupTimedDestruction():
    var modifyTimer = $ModifyOnTimerComponent
    
    # 5秒后移除实体
    modifyTimer.get_node("InternalTimer").wait_time = 5.0
    modifyTimer.shouldRemoveEntity = true
```

### 定时移除特定组件

```gdscript
# TimedEffectRemoval.gd
extends Entity

func _ready():
    setupTimedEffectRemoval()

func setupTimedEffectRemoval():
    var modifyTimer = $ModifyOnTimerComponent
    
    # 10秒后移除增益效果组件
    modifyTimer.get_node("InternalTimer").wait_time = 10.0
    modifyTimer.componentsToRemove = [
        DamageComponent,
        SpeedBoostComponent
    ]
```

### 复杂的定时修改序列

```gdscript
# ComplexTimedModification.gd
extends Entity

func _ready():
    setupComplexModification()

func setupComplexModification():
    var modifyTimer = $ModifyOnTimerComponent
    
    # 配置Timer
    modifyTimer.get_node("InternalTimer").wait_time = 8.0
    
    # 移除特定节点
    modifyTimer.nodesToRemove = [
        $VisualEffects,
        $AudioSource
    ]
    
    # 移除组件
    modifyTimer.componentsToRemove = [
        HealthComponent,
        CollisionComponent
    ]
    
    # 添加新组件
    modifyTimer.componentsToCreate = [
        preload("res://Components/Visual/FadeComponent.tscn"),
        preload("res://Components/Objects/CollectibleComponent.tscn")
    ]
```

### 与碰撞系统配合

```gdscript
# ArrowStickSystem.gd - 箭矢系统示例
extends Entity

func _ready():
    setupArrowBehavior()

func setupArrowBehavior():
    # 碰撞检测组件
    var collisionComponent = $ModifyOnCollisionComponent
    collisionComponent.didEnterBody.connect(onArrowHit)
    
    # 定时移除组件
    var timerComponent = $ModifyOnTimerComponent
    timerComponent.get_node("InternalTimer").wait_time = 10.0
    timerComponent.shouldRemoveEntity = true

func onArrowHit(body: Node2D):
    # 箭矢击中时停止移动并启动定时器
    var movementComponent = findFirstComponentSubclass(LinearMotionComponent)
    if movementComponent:
        movementComponent.queue_free()
    
    # 启动定时移除
    var timerComponent = $ModifyOnTimerComponent
    timerComponent.get_node("InternalTimer").start()
```

### 多阶段定时器

```gdscript
# MultiStageTimer.gd
extends Entity

func _ready():
    setupMultiStageTimers()

func setupMultiStageTimers():
    # 禁用内置Timer，使用自定义Timer
    var modifyTimer = $ModifyOnTimerComponent
    modifyTimer.get_node("InternalTimer").stop()
    
    # 创建多个Timer用于不同阶段
    createCustomTimers(modifyTimer)

func createCustomTimers(modifyComponent: ModifyOnTimerComponent):
    # 阶段1: 5秒后移除视觉效果
    var timer1 = Timer.new()
    add_child(timer1)
    timer1.wait_time = 5.0
    timer1.one_shot = true
    timer1.timeout.connect(modifyComponent.removeNodes)
    timer1.start()
    
    # 阶段2: 10秒后移除组件
    var timer2 = Timer.new()
    add_child(timer2)
    timer2.wait_time = 10.0
    timer2.one_shot = true
    timer2.timeout.connect(modifyComponent.removeComponents)
    timer2.start()
    
    # 阶段3: 15秒后自毁
    var timer3 = Timer.new()
    add_child(timer3)
    timer3.wait_time = 15.0
    timer3.one_shot = true
    timer3.timeout.connect(modifyComponent.removeEntity)
    timer3.start()
```

### 条件定时器

```gdscript
# ConditionalTimer.gd
extends Entity

@export var playerInRange: bool = false

func _ready():
    setupConditionalTimer()

func setupConditionalTimer():
    var modifyTimer = $ModifyOnTimerComponent
    
    # 重写默认的超时处理
    modifyTimer.get_node("InternalTimer").timeout.disconnect(modifyTimer.onInternalTimer_timeout)
    modifyTimer.get_node("InternalTimer").timeout.connect(onCustomTimeout)

func onCustomTimeout():
    # 只有在满足条件时才执行修改
    if playerInRange:
        var modifyTimer = $ModifyOnTimerComponent
        modifyTimer.performAllModifications()
    else:
        # 重新启动Timer
        $ModifyOnTimerComponent/InternalTimer.start()
```

## 🏛️ 设计模式

### 命令模式
- **命令对象**: ModifyOnTimerComponent封装修改操作
- **调用者**: Timer触发命令执行
- **接收者**: 实体及其组件接收修改

### 模板方法模式
- **基类**: NodeModifierComponentBase定义修改流程
- **具体类**: ModifyOnTimerComponent添加Timer触发机制

## 🔧 技术细节

### 执行顺序
```gdscript
func onInternalTimer_timeout() -> void:
    super.performAllModifications()
```
调用父类的 `performAllModifications()` 方法，按以下顺序执行：
1. `removeEntity()` - 如果启用
2. `removeNodes()` - 移除指定节点
3. `removeComponents()` - 移除指定组件
4. `createComponents()` - 创建新组件

### Timer信号解绑
当使用自定义Timer时，记得解绑默认信号：
```gdscript
modifyTimer.get_node("InternalTimer").timeout.disconnect(modifyTimer.onInternalTimer_timeout)
```

## ⚠️ 注意事项

1. **Timer配置**: 默认3秒等待时间，根据需要调整
2. **执行顺序**: 按固定顺序执行，注意依赖关系
3. **信号连接**: 使用自定义Timer时要正确处理信号连接
4. **内存管理**: 移除实体时确保正确清理资源
5. **碰撞配合**: 与ModifyOnCollisionComponent配合时注意信号参数

## 🔗 相关组件

- [NodeModifierComponentBase](../NodeModifierComponentBase.md) - 基础修改组件
- [ModifyOnCollisionComponent](../Physics/ModifyOnCollisionComponent.md) - 碰撞修改组件
- [TimerComponentBase](TimerComponentBase.md) - 计时器基类
- [CooldownComponent](CooldownComponent.md) - 冷却时间组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 