# TimerComponentBase API

> **继承关系**: Component > TimerComponentBase  
> **抽象类**: 必须由子类实现

基于Timer的抽象基础组件，为需要计时器功能的组件提供统一的架构模式。

## ✨ 主要特性

- 🔧 抽象基类架构，提供计时器的标准化使用模式
- ⏰ 内置Timer节点管理
- 🎛️ 可配置的启用/禁用状态
- 🏗️ 为子类提供超时事件处理框架

## 📊 导出属性

### 基础设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `isEnabled` | `bool` | `true` | 是否启用组件功能 |

## 📡 信号

此组件没有定义信号，超时处理由抽象方法实现。

## 🛠️ 主要方法

### 抽象方法
```gdscript
abstract func onTimeout() -> void
```
**描述**: 抽象方法，子类必须实现此方法来处理Timer超时事件

## 🎯 使用示例

### 创建自定义计时器组件

```gdscript
# CustomTimerComponent.gd
class_name CustomTimerComponent
extends TimerComponentBase

@export var damage: int = 10
@export var interval: float = 1.0

func _ready() -> void:
    selfAsTimer.wait_time = interval
    selfAsTimer.autostart = true
    selfAsTimer.connect("timeout", onTimeout)

func onTimeout() -> void:
    if not isEnabled:
        return
    
    # 自定义超时处理逻辑
    var healthComponent = parentEntity.findFirstComponentSubclass(HealthComponent)
    if healthComponent:
        healthComponent.takeDamage(damage)
```

### 定时移除组件

```gdscript
# TimedRemovalComponent.gd
class_name TimedRemovalComponent
extends TimerComponentBase

@export var removalDelay: float = 5.0

func _ready() -> void:
    selfAsTimer.wait_time = removalDelay
    selfAsTimer.one_shot = true
    selfAsTimer.start()
    selfAsTimer.connect("timeout", onTimeout)

func onTimeout() -> void:
    if isEnabled:
        queue_free()
```

### 循环效果组件

```gdscript
# PeriodicEffectComponent.gd
class_name PeriodicEffectComponent
extends TimerComponentBase

@export var effectStrength: float = 1.0
@export var effectInterval: float = 2.0

func _ready() -> void:
    setupTimer()

func setupTimer() -> void:
    selfAsTimer.wait_time = effectInterval
    selfAsTimer.autostart = true
    selfAsTimer.connect("timeout", onTimeout)

func onTimeout() -> void:
    if not isEnabled:
        return
    
    # 应用周期性效果
    applyPeriodicEffect()

func applyPeriodicEffect() -> void:
    # 子类实现具体效果
    pass
```

## 🏛️ 设计模式

### 模板方法模式
- **抽象基类**: 定义计时器处理的标准流程
- **抽象方法**: `onTimeout()` 由子类实现具体逻辑
- **通用状态**: 提供启用状态和Timer节点管理

### 组合模式
- **Timer节点**: 作为组件的核心功能节点
- **自引用**: `selfAsTimer` 提供直接的Timer访问

## 🔧 技术细节

### Timer节点管理
```gdscript
@onready var selfAsTimer: Timer = self.get_node(^".") as Timer
```
- 组件本身就是Timer节点
- 通过自引用获取Timer功能

### 启用状态检查
```gdscript
if not isEnabled:
    return
```
建议在 `onTimeout()` 实现中首先检查启用状态。

## ⚠️ 注意事项

1. **抽象类**: 不能直接实例化，必须通过子类使用
2. **Timer配置**: 子类需要在 `_ready()` 中配置Timer属性
3. **信号连接**: 需要手动连接Timer的timeout信号到onTimeout方法
4. **启用检查**: 建议在超时处理中检查isEnabled状态

## 🔗 相关组件

- [CooldownComponent](CooldownComponent.md) - 冷却时间管理组件
- [ModifyOnTimerComponent](ModifyOnTimerComponent.md) - 定时修改组件
- [DamageOverTimeComponent](../Combat/DamageOverTimeComponent.md) - 持续伤害组件
- [TimerAgentComponentBase](../AI/TimerAgentComponentBase.md) - AI计时器基类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 