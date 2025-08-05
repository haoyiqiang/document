# AreaContactComponent API

> **继承关系**: Component > AreaCollisionComponent > AreaContactComponent  
> **基于**: Area2D碰撞检测和接触管理

Area接触管理组件，维护与组件Area2D当前碰撞接触的所有Area2D、PhysicsBody2D或TileMapLayer的完整列表。

## ✨ 主要特性

- 📋 维护实时接触列表
- 🎯 支持群组过滤
- 🔄 自动管理进入/退出事件
- 📊 分离Area2D和PhysicsBody2D管理
- 🚀 高性能碰撞监控
- 🔍 调试信息显示

## 📊 导出属性

### 过滤设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `groupToInclude` | `StringName` | `""` | 只包含指定群组的物理节点，空则包含所有 |

继承自 [AreaCollisionComponent](AreaCollisionComponent.md) 的所有属性。

## 🎯 状态属性

### 接触列表
| 属性 | 类型 | 描述 |
|------|------|------|
| `areasInContact` | `Array[Area2D]` | 当前碰撞接触的Area2D列表 |
| `bodiesInContact` | `Array[Node2D]` | 当前碰撞接触的PhysicsBody2D或TileMapLayer列表 |

## 🛠️ 主要方法

### 接触管理
```gdscript
func readdAllContacts() -> void
```
**描述**: 清空并重新添加所有当前接触的物理节点

### 事件处理
```gdscript
func onAreaEntered(areaEntered: Area2D) -> void
func onBodyEntered(bodyEntered: Node2D) -> void
func onAreaExited(areaExited: Area2D) -> void
func onBodyExited(bodyExited: Node2D) -> void
```

## 🎯 使用示例

### 基础接触检测

```gdscript
# ContactDetector.gd
extends Entity

func _ready():
    setupContactDetection()

func setupContactDetection():
    var contactComponent = $AreaContactComponent
    
    # 连接信号
    contactComponent.didEnterArea.connect(onAreaEntered)
    contactComponent.didExitArea.connect(onAreaExited)
    contactComponent.didEnterBody.connect(onBodyEntered)
    contactComponent.didExitBody.connect(onBodyExited)

func onAreaEntered(area: Area2D):
    print("Area entered: ", area.name)
    print("Total areas in contact: ", $AreaContactComponent.areasInContact.size())

func onAreaExited(area: Area2D):
    print("Area exited: ", area.name)

func onBodyEntered(body: Node2D):
    print("Body entered: ", body.name)
    print("Total bodies in contact: ", $AreaContactComponent.bodiesInContact.size())

func onBodyExited(body: Node2D):
    print("Body exited: ", body.name)
```

### 群组过滤检测

```gdscript
# PlayerDetector.gd - 只检测玩家群组
extends Entity

func _ready():
    setupPlayerDetection()

func setupPlayerDetection():
    var contactComponent = $AreaContactComponent
    
    # 只检测"player"群组的物理体
    contactComponent.groupToInclude = "player"
    
    contactComponent.didEnterBody.connect(onPlayerEntered)
    contactComponent.didExitBody.connect(onPlayerExited)

func onPlayerEntered(player: Node2D):
    print("Player entered detection zone: ", player.name)
    # 玩家进入时的逻辑
    activatePlayerInteraction(player)

func onPlayerExited(player: Node2D):
    print("Player left detection zone: ", player.name)
    # 玩家离开时的逻辑
    deactivatePlayerInteraction(player)

func activatePlayerInteraction(player: Node2D):
    # 显示交互提示等
    pass

func deactivatePlayerInteraction(player: Node2D):
    # 隐藏交互提示等
    pass
```

### 多目标跟踪系统

```gdscript
# MultiTargetTracker.gd
extends Entity

var trackedTargets: Dictionary = {}

func _ready():
    setupMultiTargetTracking()

func setupMultiTargetTracking():
    var contactComponent = $AreaContactComponent
    
    # 设置为检测敌人群组
    contactComponent.groupToInclude = "enemies"
    
    contactComponent.didEnterBody.connect(onEnemyEntered)
    contactComponent.didExitBody.connect(onEnemyExited)

func onEnemyEntered(enemy: Node2D):
    # 开始跟踪敌人
    trackedTargets[enemy] = {
        "enterTime": Time.get_time_dict_from_system(),
        "lastPosition": enemy.global_position
    }
    
    print("Now tracking ", trackedTargets.size(), " enemies")
    
    # 如果是第一个敌人，开始跟踪逻辑
    if trackedTargets.size() == 1:
        startTrackingBehavior()

func onEnemyExited(enemy: Node2D):
    # 停止跟踪敌人
    trackedTargets.erase(enemy)
    
    print("Enemy lost. Now tracking ", trackedTargets.size(), " enemies")
    
    # 如果没有敌人了，停止跟踪
    if trackedTargets.is_empty():
        stopTrackingBehavior()

func startTrackingBehavior():
    # 开始跟踪行为（如激活AI、播放警报等）
    print("Target acquired - starting tracking")

func stopTrackingBehavior():
    # 停止跟踪行为
    print("All targets lost - stopping tracking")
```

### 区域控制系统

```gdscript
# AreaControllerEntity.gd
extends Entity

@export var maxCapacity: int = 5
var currentOccupants: Array[Node2D] = []

func _ready():
    setupAreaControl()

func setupAreaControl():
    var contactComponent = $AreaContactComponent
    
    contactComponent.didEnterBody.connect(onEntityEntered)
    contactComponent.didExitBody.connect(onEntityExited)

func onEntityEntered(entity: Node2D):
    if currentOccupants.size() >= maxCapacity:
        print("Area at capacity! Rejecting: ", entity.name)
        rejectEntity(entity)
        return
    
    currentOccupants.append(entity)
    print("Entity accepted: ", entity.name, " (", currentOccupants.size(), "/", maxCapacity, ")")
    
    # 给实体添加区域效果
    applyAreaEffect(entity, true)

func onEntityExited(entity: Node2D):
    currentOccupants.erase(entity)
    print("Entity left: ", entity.name, " (", currentOccupants.size(), "/", maxCapacity, ")")
    
    # 移除区域效果
    applyAreaEffect(entity, false)

func rejectEntity(entity: Node2D):
    # 推开实体或显示"区域已满"消息
    if entity.has_method("pushAway"):
        entity.pushAway(global_position)

func applyAreaEffect(entity: Node2D, isEntering: bool):
    # 应用或移除区域效果（如治疗、加速等）
    var effectComponent = entity.get_node_or_null("AreaEffectComponent")
    if effectComponent:
        effectComponent.setActive(isEntering)
```

### 动态接触分析

```gdscript
# ContactAnalyzer.gd
extends Entity

var contactHistory: Array[Dictionary] = []

func _ready():
    setupContactAnalysis()

func setupContactAnalysis():
    var contactComponent = $AreaContactComponent
    
    contactComponent.didEnterArea.connect(onContactStart)
    contactComponent.didExitArea.connect(onContactEnd)
    
    # 定期分析接触模式
    var timer = Timer.new()
    add_child(timer)
    timer.wait_time = 1.0
    timer.autostart = true
    timer.timeout.connect(analyzeContactPatterns)

func onContactStart(area: Area2D):
    contactHistory.append({
        "type": "enter",
        "node": area,
        "time": Time.get_unix_time_from_system()
    })

func onContactEnd(area: Area2D):
    contactHistory.append({
        "type": "exit", 
        "node": area,
        "time": Time.get_unix_time_from_system()
    })

func analyzeContactPatterns():
    var contactComponent = $AreaContactComponent
    
    print("=== Contact Analysis ===")
    print("Current areas in contact: ", contactComponent.areasInContact.size())
    print("Current bodies in contact: ", contactComponent.bodiesInContact.size())
    print("Contact events in history: ", contactHistory.size())
    
    # 清理旧历史记录（保留最近60秒）
    var cutoffTime = Time.get_unix_time_from_system() - 60
    contactHistory = contactHistory.filter(func(record): return record.time > cutoffTime)
```

### 性能监控组件

```gdscript
# ContactPerformanceMonitor.gd
extends Entity

func _ready():
    setupPerformanceMonitoring()

func setupPerformanceMonitoring():
    var contactComponent = $AreaContactComponent
    
    # 启用调试模式查看性能信息
    contactComponent.debugMode = true
    
    # 定期检查性能
    var timer = Timer.new()
    add_child(timer)
    timer.wait_time = 5.0
    timer.autostart = true
    timer.timeout.connect(checkPerformance)

func checkPerformance():
    var contactComponent = $AreaContactComponent
    
    var areaCount = contactComponent.areasInContact.size()
    var bodyCount = contactComponent.bodiesInContact.size()
    var totalContacts = areaCount + bodyCount
    
    if totalContacts > 20:
        print("Warning: High contact count (", totalContacts, ") may impact performance")
    
    Debug.addWatchValue("Areas in Contact", areaCount)
    Debug.addWatchValue("Bodies in Contact", bodyCount)
```

## 🏛️ 设计模式

### 观察者模式
- **主体**: Area2D的碰撞信号
- **观察者**: AreaContactComponent维护接触列表
- **通知**: 进入/退出事件的信号传播

### 集合管理模式
- **集合**: areasInContact和bodiesInContact数组
- **维护**: 自动添加和移除元素
- **查询**: 提供当前状态的实时访问

## 🔧 技术细节

### 接触列表维护
```gdscript
func onAreaEntered(areaEntered: Area2D) -> void:
    # 检查过滤条件
    if not groupToInclude.is_empty() and not areaEntered.is_in_group(groupToInclude):
        return
    
    areasInContact.append(areaEntered)
    self.onCollide(areaEntered)
    didEnterArea.emit(areaEntered)
```

### 群组过滤
```gdscript
# 只有空群组名或节点在指定群组中才会被包含
if not groupToInclude.is_empty() and not node.is_in_group(groupToInclude):
    return
```

### 调试信息显示
```gdscript
func showDebugInfo() -> void:
    if not debugMode: return
    Debug.addComponentWatchList(self, {
        areasInContact = areasInContact,
        bodiesInContact = bodiesInContact
    })
```

## ⚠️ 注意事项

1. **性能考虑**: 接触列表越长，性能影响越大，建议使用群组过滤
2. **群组过滤**: groupToInclude只支持单个群组，多群组需要自定义逻辑
3. **移除检查**: 退出事件不受isEnabled影响，但受shouldMonitorAreas/Bodies影响
4. **初始化**: 组件启动时会调用readdAllContacts()检测已存在的接触
5. **调试模式**: 启用debugMode会在每帧显示接触信息，影响性能

## 🔗 相关组件

- [AreaCollisionComponent](AreaCollisionComponent.md) - 基础碰撞检测
- [AreaComponentBase](AreaComponentBase.md) - Area基础组件
- [ModifyOnCollisionComponent](ModifyOnCollisionComponent.md) - 碰撞修改组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 