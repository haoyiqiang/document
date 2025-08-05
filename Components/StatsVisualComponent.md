# StatsVisualComponent API

## 概述
`StatsVisualComponent` 是一个统计数据可视化组件，当指定的Stat数值发生变化时自动显示TextBubble和其他UI效果。它可以监控多个统计数据，并在组件位置而非实体位置显示视觉效果。

**继承链**：`Component` → `StatsVisualComponent`

## 主要特性
- 📊 **多数据监控**：同时监控多个Stat对象的变化
- 💬 **文字气泡**：自动显示Stat变化的TextBubble
- 🔄 **动态管理**：支持运行时添加/移除统计数据
- 🎨 **颜色配置**：可配置气泡颜色和显示格式
- 🔗 **自动集成**：可从StatsComponent自动获取统计数据
- ⚡ **性能友好**：基于信号的高效更新机制

## 导出属性

### 监控配置
- **statsToMonitor** (`Array[Stat]`)：要监控的统计数据数组
- **statsToExclude** (`Array[Stat]`)：要排除的统计数据数组
- **shouldCopyFromStatsComponent** (`bool = true`)：是否从StatsComponent复制统计数据

### 视觉配置
- **shouldAppendDisplayName** (`bool = false`)：是否在气泡中显示Stat的显示名称
- **shouldColorBubble** (`bool = true`)：是否为气泡着色

### 控制选项
- **isEnabled** (`bool = true`)：是否启用组件功能

## 主要方法

### 生命周期
- **connectSignals()** → `void`：连接所有监控Stat的信号
- **onStatChanged(stat: Stat)** → `void`：Stat变化时的回调处理

## 使用示例

### 基础数据监控
```gdscript
# 监控生命值和法力值变化
@export var stats_visual: StatsVisualComponent
@export var health_stat: Stat
@export var mana_stat: Stat

func _ready():
    stats_visual.statsToMonitor = [health_stat, mana_stat]
    stats_visual.shouldColorBubble = true
    stats_visual.shouldAppendDisplayName = true
```

### 自动集成StatsComponent
```gdscript
# 自动从StatsComponent获取所有统计数据
@export var stats_visual: StatsVisualComponent
@export var stats_component: StatsComponent

func _ready():
    # shouldCopyFromStatsComponent默认为true，会自动复制
    stats_visual.shouldAppendDisplayName = false  # 不显示名称，只显示数值
    stats_visual.shouldColorBubble = true
```

### 选择性监控
```gdscript
# 监控特定类型的统计数据
@export var stats_visual: StatsVisualComponent
@export var stats_component: StatsComponent

func setup_selective_monitoring():
    # 复制所有统计数据
    stats_visual.shouldCopyFromStatsComponent = true
    
    # 但排除不想显示的数据
    var private_stats = [
        stats_component.findStatByName("experience"),
        stats_component.findStatByName("skill_points")
    ]
    stats_visual.statsToExclude = private_stats
```

### 战斗数据显示
```gdscript
# 专门显示战斗相关数据
@export var combat_stats_visual: StatsVisualComponent
@export var health: Stat
@export var shield: Stat
@export var energy: Stat

func setup_combat_display():
    combat_stats_visual.statsToMonitor = [health, shield, energy]
    combat_stats_visual.shouldAppendDisplayName = true  # 显示"生命值: 100"
    combat_stats_visual.shouldColorBubble = true
    
    # 为组件设置特定位置
    combat_stats_visual.position = Vector2(0, -50)  # 显示在实体上方
```

### 动态统计监控
```gdscript
# 运行时动态添加监控
@export var stats_visual: StatsVisualComponent

func add_temporary_stat_monitoring(temp_stat: Stat):
    stats_visual.statsToMonitor.append(temp_stat)
    # 手动连接信号
    temp_stat.changed.connect(stats_visual.onStatChanged.bind(temp_stat))

func remove_stat_monitoring(stat: Stat):
    stats_visual.statsToMonitor.erase(stat)
    # 断开信号连接
    if stat.changed.is_connected(stats_visual.onStatChanged):
        stat.changed.disconnect(stats_visual.onStatChanged)
```

## 设计模式

### 分层数据显示
```gdscript
# 为不同类型的数据创建不同的显示层
class_name LayeredStatsDisplay
extends Node

@export var core_stats_visual: StatsVisualComponent      # 核心属性
@export var combat_stats_visual: StatsVisualComponent    # 战斗属性
@export var resource_stats_visual: StatsVisualComponent  # 资源属性

func setup_layered_display(stats_component: StatsComponent):
    # 核心属性：生命、法力、体力
    core_stats_visual.statsToMonitor = [
        stats_component.health,
        stats_component.mana,
        stats_component.stamina
    ]
    core_stats_visual.position = Vector2(0, -60)
    
    # 战斗属性：攻击、防御、速度
    combat_stats_visual.statsToMonitor = [
        stats_component.attack,
        stats_component.defense,
        stats_component.speed
    ]
    combat_stats_visual.position = Vector2(0, -40)
    
    # 资源属性：金币、经验
    resource_stats_visual.statsToMonitor = [
        stats_component.gold,
        stats_component.experience
    ]
    resource_stats_visual.position = Vector2(0, -20)
```

### 条件显示系统
```gdscript
# 基于条件显示不同的统计数据
@export var stats_visual: StatsVisualComponent
@export var debug_mode: bool = false

func update_display_mode():
    if debug_mode:
        # 调试模式显示所有数据
        stats_visual.shouldCopyFromStatsComponent = true
        stats_visual.statsToExclude.clear()
        stats_visual.shouldAppendDisplayName = true
    else:
        # 正常模式只显示重要数据
        stats_visual.shouldCopyFromStatsComponent = false
        stats_visual.statsToMonitor = [
            health_stat,
            mana_stat
        ]
        stats_visual.shouldAppendDisplayName = false
```

### 智能过滤显示
```gdscript
# 根据数值变化的重要性过滤显示
class_name SmartStatsFilter
extends Node

@export var stats_visual: StatsVisualComponent
@export var min_change_threshold: float = 5.0

var original_on_stat_changed: Callable

func _ready():
    # 拦截原始的onStatChanged方法
    original_on_stat_changed = stats_visual.onStatChanged
    stats_visual.onStatChanged = filtered_on_stat_changed

func filtered_on_stat_changed(stat: Stat):
    # 只有变化足够大时才显示
    if abs(stat.lastDifference) >= min_change_threshold:
        original_on_stat_changed.call(stat)
```

## 技术细节

### 信号连接机制
- 在`_ready()`时连接所有监控Stat的`changed`信号
- 使用`bind()`传递Stat参数到回调函数
- 支持运行时动态连接/断开信号

### 位置控制
- 气泡显示在**组件位置**而非实体位置
- 可通过设置组件position创建偏移效果
- 支持多个组件在不同位置显示不同数据

### 自动集成逻辑
- 通过`coComponents`获取同实体的StatsComponent
- `shouldCopyFromStatsComponent`为true时自动添加所有Stats
- 排除列表优先级高于监控列表

## 注意事项

### 性能考虑
- 大量统计数据监控可能影响性能
- 建议合理使用排除列表减少不必要的监控
- 频繁变化的数据可能产生大量气泡

### 位置设计
- 组件位置决定气泡显示位置
- 多个StatsVisualComponent可实现分层显示
- 注意避免气泡重叠影响可读性

### 运行时管理
- 支持运行时添加/移除统计数据监控
- 需要手动管理信号连接/断开
- 动态变更时注意内存和性能

## 相关组件
- [`StatsComponent`](../Data/StatsComponent.md) - 统计数据管理
- [`HealthVisualComponent`](../Visual/HealthVisualComponent.md) - 生命值视觉效果
- [`CollectibleComponent`](../Objects/CollectibleComponent.md) - 可拾取物品（CollectibleStatComponent） 