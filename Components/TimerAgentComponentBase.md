# TimerAgentComponentBase API 文档

## 概述
`TimerAgentComponentBase` 是依赖于Timer的简单AI代理组件的抽象基类。例如，每N秒随机攻击等功能。

**⚠️ 实验性功能**

## 继承
`TimerAgentComponentBase` → `Component` → `Node`

## 主要特性
- 基于定时器的AI行为框架
- 抽象基类，需要子类实现具体逻辑
- 简单的启用/禁用控制

## 导出属性

#### `isEnabled: bool = true`
是否启用此AI代理组件。

## 状态属性

#### `selfAsTimer: Timer`
将自身节点转换为Timer的引用。这要求组件实际上是一个Timer节点或继承自Timer。

## 主要方法

### 抽象方法

#### `onTimeout() -> void`
**必须在子类中重写**
定时器超时时调用的抽象方法。子类应该在此方法中实现具体的AI行为逻辑。

## 使用示例

```gdscript
# 创建自定义AI代理组件
class_name RandomAttackAI
extends TimerAgentComponentBase

@export var attackChance: float = 0.3

func _ready() -> void:
    # 配置定时器
    wait_time = 2.0
    timeout.connect(onTimeout)
    start()

# 重写抽象方法
func onTimeout() -> void:
    if not isEnabled:
        return
    
    # 随机攻击逻辑
    if randf() < attackChance:
        performAttack()

func performAttack() -> void:
    print("AI执行攻击！")
    # 实现攻击逻辑
```

## 设计模式

### 定时器基础
由于此组件继承自Component但需要Timer功能，建议的使用方式：

1. **场景结构**：将Timer节点作为基础，添加此脚本
2. **继承方式**：让AI组件同时具有Timer和Component功能

### 常见AI行为模式

```gdscript
# 巡逻AI示例
class_name PatrolAI
extends TimerAgentComponentBase

@export var patrolPoints: Array[Vector2] = []
var currentPointIndex: int = 0

func onTimeout() -> void:
    if not isEnabled or patrolPoints.is_empty():
        return
    
    # 移动到下一个巡逻点
    var targetPoint = patrolPoints[currentPointIndex]
    parentEntity.global_position = targetPoint
    currentPointIndex = (currentPointIndex + 1) % patrolPoints.size()
```

## 技术限制

### 脚本设置问题
注意代码中的注释：无法将此类设置为`abstract`，因为会在`Spawner.gd:99`的`sceneResource.instantiate()`处导致"Cannot set object script"错误。

## 最佳实践

1. **启用检查**：在`onTimeout()`中始终检查`isEnabled`状态
2. **性能优化**：根据需要启用/禁用定时器以节省资源
3. **错误处理**：在AI逻辑中添加适当的错误检查
4. **状态管理**：考虑AI的不同状态（空闲、攻击、巡逻等）

## 扩展建议

```gdscript
# 状态机AI示例
class_name StateMachineAI
extends TimerAgentComponentBase

enum AIState { IDLE, PATROL, CHASE, ATTACK }
var currentState: AIState = AIState.IDLE

func onTimeout() -> void:
    if not isEnabled:
        return
    
    match currentState:
        AIState.IDLE:
            handleIdleState()
        AIState.PATROL:
            handlePatrolState()
        AIState.CHASE:
            handleChaseState()
        AIState.ATTACK:
            handleAttackState()

func handleIdleState() -> void:
    # 空闲状态逻辑
    pass

func handlePatrolState() -> void:
    # 巡逻状态逻辑
    pass

func handleChaseState() -> void:
    # 追击状态逻辑
    pass

func handleAttackState() -> void:
    # 攻击状态逻辑
    pass
```

## 注意事项

1. **定时器配置**：需要在子类中配置定时器的间隔时间
2. **性能考虑**：频繁的定时器回调可能影响性能
3. **状态同步**：确保AI状态与游戏状态保持同步
4. **错误处理**：AI行为失败时应有适当的回退策略

## 相关类
- `Component` - 基础组件类
- `Timer` - Godot定时器节点
- `Entity` - 实体容器 