# LevelUpComponent API

> **继承关系**: Component > LevelUpComponent  
> **节点类型**: Node2D

经验值等级系统组件，当经验值达到最大值时显示等级提升UI界面并处理相关逻辑。

## ✨ 主要特性

- 📈 自动监控经验值变化
- 🎊 等级提升时显示UI界面
- 🔄 支持经验值重置循环
- 🎮 可扩展的升级选择系统
- 📊 内置Node2D容器用于UI展示

## 📊 导出属性

### 核心设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `xp` | `Stat` | `null` | 经验值统计资源，达到最大值时触发等级提升 |
| `nodeToShowOnMaxXP` | `CanvasItem` | `null` | 经验值满时显示的UI节点 |
| `isEnabled` | `bool` | `true` | 是否启用组件功能 |

## 📡 信号

```gdscript
signal xpDidMax()
signal xpDidReset()
```

| 信号 | 触发时机 | 用途 |
|------|----------|------|
| `xpDidMax` | 经验值达到最大值时 | 显示等级提升UI，处理升级逻辑 |
| `xpDidReset` | 经验值重置时 | 处理等级提升完成后的清理工作 |

## 🛠️ 主要方法

### 信号连接
```gdscript
func connectSignals() -> void
```
**描述**: 连接经验值统计的信号到组件处理方法

### 经验值满处理
```gdscript
func onxp_didMax() -> void
```
**描述**: 当经验值达到最大值时的处理方法，显示UI并发射信号

### 重置经验值
```gdscript
func resetXP() -> void
```
**描述**: 隐藏UI界面并重置经验值到最小值，完成一个等级提升周期

## 🎯 使用示例

### 基础等级系统

```gdscript
# 设置经验值Stat资源
@export var playerXPStat: Stat

func _ready():
    var levelUpComponent = $LevelUpComponent
    levelUpComponent.xp = playerXPStat
    levelUpComponent.nodeToShowOnMaxXP = $LevelUpUI
    
    # 连接信号
    levelUpComponent.xpDidMax.connect(onLevelUp)
    levelUpComponent.xpDidReset.connect(onLevelUpComplete)

func onLevelUp():
    # 显示升级选择界面
    print("Level Up! Choose your upgrade...")

func onLevelUpComplete():
    print("Level up complete!")
```

### 与升级选择列表集成

```gdscript
# LevelUpWithChoices.gd
extends Node2D

@export var xpStat: Stat
@export var upgradeChoicesList: UpgradeChoicesList

func _ready():
    var levelUpComponent = $LevelUpComponent
    levelUpComponent.xp = xpStat
    levelUpComponent.nodeToShowOnMaxXP = upgradeChoicesList
    
    levelUpComponent.xpDidMax.connect(onShowUpgradeChoices)
    upgradeChoicesList.upgradeWasChosen.connect(onUpgradeChosen)

func onShowUpgradeChoices():
    # 刷新可用升级选项
    upgradeChoicesList.refreshUpgradeChoices()

func onUpgradeChosen(upgrade: Upgrade):
    # 应用选择的升级
    upgrade.apply(get_parent())
    
    # 重置经验值，准备下一级
    $LevelUpComponent.resetXP()
```

### 自动等级系统

```gdscript
# AutoLevelUp.gd
extends Node

@export var autoResetDelay: float = 2.0

func _ready():
    var levelUpComponent = $"../LevelUpComponent"
    levelUpComponent.xpDidMax.connect(onAutoLevelUp)

func onAutoLevelUp():
    # 自动选择升级并重置
    await get_tree().create_timer(autoResetDelay).timeout
    
    # 这里可以添加自动升级选择逻辑
    performAutoUpgrade()
    
    $"../LevelUpComponent".resetXP()

func performAutoUpgrade():
    # 实现自动升级逻辑
    pass
```

### 多角色等级系统

```gdscript
# MultiCharacterLevelSystem.gd
extends Node

var characterLevelComponents: Dictionary = {}

func setupCharacterLevelUp(character: Entity, xpStat: Stat):
    var levelUpComponent = preload("res://Components/Gameplay/LevelUpComponent.tscn").instantiate()
    character.add_child(levelUpComponent)
    
    levelUpComponent.xp = xpStat
    levelUpComponent.nodeToShowOnMaxXP = createCharacterLevelUI(character)
    
    levelUpComponent.xpDidMax.connect(onCharacterLevelUp.bind(character))
    characterLevelComponents[character] = levelUpComponent

func onCharacterLevelUp(character: Entity):
    print(character.name + " leveled up!")
    # 处理角色专属升级逻辑
```

## 🏛️ 设计模式

### 观察者模式
- **主体**: Stat资源的didMax信号
- **观察者**: LevelUpComponent监听经验值变化
- **响应**: 自动显示UI和处理升级逻辑

### 状态模式
- **正常状态**: UI隐藏，监听经验值变化
- **升级状态**: UI显示，等待玩家选择
- **重置状态**: 清理UI，重置经验值

## 🔧 技术细节

### 经验值监控
```gdscript
xp: Stat:
    set(newValue):
        if newValue != xp:
            xp = newValue
            if xp and self.is_node_ready(): connectSignals()
```
- 使用属性setter自动连接信号
- 支持运行时更换经验值资源

### UI管理
```gdscript
func onxp_didMax() -> void:
    nodeToShowOnMaxXP.visible = true
    xpDidMax.emit()

func resetXP() -> void:
    nodeToShowOnMaxXP.visible = false
    xp.setToMin()
    xpDidReset.emit()
```

## ⚠️ 注意事项

1. **Stat资源**: 必须设置有效的Stat资源作为经验值
2. **UI节点**: nodeToShowOnMaxXP必须是有效的CanvasItem
3. **初始状态**: UI节点在初始化时会被隐藏
4. **Node2D容器**: 组件本身是Node2D，可以作为UI容器使用
5. **信号连接**: 确保在设置xp属性后正确连接信号

## 🔗 相关组件

- [StatsComponent](../Data/StatsComponent.md) - 管理经验值等统计数据
- [UpgradesComponent](UpgradesComponent.md) - 处理升级资源
- [StatModifierComponent](../Data/StatModifierComponent.md) - 修改统计数据
- [UI/UpgradeChoicesList](../../UI/Lists/UpgradeChoicesList.md) - 升级选择列表

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 