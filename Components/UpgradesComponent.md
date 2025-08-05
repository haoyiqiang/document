# UpgradesComponent API

## 概述
`UpgradesComponent` 跟踪实体拥有的`Upgrade`资源。提供完整的升级系统，包括获取、升级、丢弃升级，以及基于`StatsComponent`的费用支付机制。

**继承链**：`Component` → `UpgradesComponent`

**相关系统**：与`UpgradeChoiceUI`和`UpgradeChoiceList`配合提供升级选择界面。

## 依赖要求
- **StatsComponent**：必需的统计数据组件，用于支付升级费用

## 主要特性
- 📦 **升级管理**：跟踪和管理所有已获得的升级
- 💰 **费用系统**：基于Stat资源的升级购买机制
- 📈 **等级提升**：支持升级的多级提升
- 🔍 **快速查找**：字典缓存机制提高查找性能
- 🔄 **资源重置**：支持游戏重启时的升级重置
- 📡 **事件系统**：完整的获取/丢弃事件通知

## 导出属性

### 升级管理
- **upgrades** (`Array[Upgrade]`)：实体拥有的所有升级列表
- **shouldResetResourcesOnReady** (`bool = false`)：初始化时是否重置升级资源

## 状态属性
- **upgradesDictionary** (`Dictionary[StringName, Upgrade]`)：按名称缓存的升级字典

## 信号

### 升级事件
- **didAcquire**(upgrade: Upgrade)：获得升级时发出（在Upgrade.didAcquire之前）
- **didDiscard**(upgrade: Upgrade)：丢弃升级时发出（在Upgrade.didDiscard之前）

## 主要方法

### 升级查找
- **getUpgrade**(upgradeName: StringName) → `Upgrade`：从缓存获取升级，失败时调用findUpgrade
- **findUpgrade**(nameToSearch: StringName) → `Upgrade`：在upgrades数组中搜索升级
- **cacheUpgrades**() → `int`：将所有升级缓存到字典中

### 升级管理
- **addUpgrade**(newUpgrade: Upgrade, overwrite: bool = false) → `bool`：添加新升级
- **addOrLevelUpUpgrade**(newUpgrade: Upgrade) → `int`：添加或升级现有升级
- **incrementUpgradeLevel**(upgrade: Upgrade) → `bool`：提升升级等级
- **discardUpgrade**(nameToDiscard: StringName) → `bool`：丢弃指定升级

### 费用处理
- **findPaymentStat**(upgradeToBuy: Upgrade) → `Stat`：查找支付升级所需的Stat

### 系统管理
- **resetUpgrades**() → `void`：重置所有升级资源

## 使用示例

### 基础升级系统
```gdscript
# 设置基本的升级管理
@export var upgrades_component: UpgradesComponent
@export var stats_component: StatsComponent

func _ready():
    upgrades_component.didAcquire.connect(_on_upgrade_acquired)
    upgrades_component.didDiscard.connect(_on_upgrade_discarded)

func _on_upgrade_acquired(upgrade: Upgrade):
    print(f"获得升级: {upgrade.name}, 等级: {upgrade.level}")

func _on_upgrade_discarded(upgrade: Upgrade):
    print(f"丢弃升级: {upgrade.name}")
```

### 升级购买系统
```gdscript
# 处理升级购买
@export var upgrades_component: UpgradesComponent
@export var available_upgrades: Array[Upgrade]

func try_purchase_upgrade(upgrade_name: StringName) -> bool:
    # 查找可购买的升级
    var upgrade_to_buy = find_available_upgrade(upgrade_name)
    if not upgrade_to_buy:
        print("升级不存在")
        return false
    
    # 尝试购买或升级
    var result_level = upgrades_component.addOrLevelUpUpgrade(upgrade_to_buy)
    if result_level > 0:
        print(f"升级成功，当前等级: {result_level}")
        return true
    else:
        print("升级失败，资源不足或其他错误")
        return false

func find_available_upgrade(name: StringName) -> Upgrade:
    for upgrade in available_upgrades:
        if upgrade.name == name:
            return upgrade
    return null
```

### 升级选择界面
```gdscript
# 升级选择UI系统
@export var upgrades_component: UpgradesComponent
@export var upgrade_buttons: Array[Button]
@export var available_upgrades: Array[Upgrade]

func _ready():
    setup_upgrade_choices()

func setup_upgrade_choices():
    for i in range(min(upgrade_buttons.size(), available_upgrades.size())):
        var button = upgrade_buttons[i]
        var upgrade = available_upgrades[i]
        
        button.text = f"{upgrade.name} - 等级 {get_current_level(upgrade)}"
        button.pressed.connect(func(): offer_upgrade(upgrade))
        
        # 检查是否可购买
        update_button_state(button, upgrade)

func get_current_level(upgrade: Upgrade) -> int:
    var current = upgrades_component.getUpgrade(upgrade.name)
    return current.level if current else 0

func update_button_state(button: Button, upgrade: Upgrade):
    var payment_stat = upgrades_component.findPaymentStat(upgrade)
    var can_afford = payment_stat and payment_stat.value >= upgrade.costValue
    
    button.disabled = not can_afford
    if not can_afford:
        button.text += " (资源不足)"

func offer_upgrade(upgrade: Upgrade):
    upgrades_component.addOrLevelUpUpgrade(upgrade)
    setup_upgrade_choices()  # 刷新界面
```

### 升级效果系统
```gdscript
# 基于升级的效果系统
@export var upgrades_component: UpgradesComponent
@export var base_damage: float = 10.0

func _ready():
    upgrades_component.didAcquire.connect(_on_upgrade_acquired)

func _on_upgrade_acquired(upgrade: Upgrade):
    # 根据升级类型应用效果
    match upgrade.name:
        "damage_boost":
            apply_damage_upgrade(upgrade)
        "health_boost":
            apply_health_upgrade(upgrade)
        "speed_boost":
            apply_speed_upgrade(upgrade)

func apply_damage_upgrade(upgrade: Upgrade):
    var bonus = upgrade.level * 5.0
    # 应用伤害加成
    update_weapon_damage(base_damage + bonus)

func get_total_damage_bonus() -> float:
    var damage_upgrade = upgrades_component.getUpgrade("damage_boost")
    return damage_upgrade.level * 5.0 if damage_upgrade else 0.0
```

### 升级保存系统
```gdscript
# 升级进度保存和加载
@export var upgrades_component: UpgradesComponent
var save_file_path = "user://upgrade_progress.save"

func save_upgrades():
    var save_data = {}
    for upgrade in upgrades_component.upgrades:
        save_data[upgrade.name] = {
            "level": upgrade.level,
            "acquired": true
        }
    
    var file = FileAccess.open(save_file_path, FileAccess.WRITE)
    file.store_string(JSON.stringify(save_data))
    file.close()

func load_upgrades():
    if not FileAccess.file_exists(save_file_path):
        return
    
    var file = FileAccess.open(save_file_path, FileAccess.READ)
    var save_data = JSON.parse_string(file.get_as_text())
    file.close()
    
    for upgrade_name in save_data:
        var upgrade_data = save_data[upgrade_name]
        # 重建升级状态
        restore_upgrade(upgrade_name, upgrade_data.level)

func restore_upgrade(name: StringName, level: int):
    # 找到基础升级并设置等级
    var base_upgrade = find_base_upgrade(name)
    if base_upgrade:
        var restored_upgrade = base_upgrade.duplicate()
        restored_upgrade.level = level
        upgrades_component.addUpgrade(restored_upgrade, true)
```

## 设计模式

### 升级管理器
```gdscript
# 全局升级管理系统
class_name UpgradeManager
extends Node

var player_upgrades: UpgradesComponent
var upgrade_registry: Dictionary[StringName, Upgrade] = {}

func _ready():
    load_upgrade_registry()
    
func register_upgrade(upgrade: Upgrade):
    upgrade_registry[upgrade.name] = upgrade

func get_available_upgrades(category: String = "") -> Array[Upgrade]:
    var available = []
    for upgrade in upgrade_registry.values():
        if category.is_empty() or upgrade.category == category:
            available.append(upgrade)
    return available

func can_afford_upgrade(upgrade: Upgrade, player: Entity) -> bool:
    var upgrades_comp = player.findFirstComponentSubclass(UpgradesComponent)
    if not upgrades_comp:
        return false
    
    var payment_stat = upgrades_comp.findPaymentStat(upgrade)
    return payment_stat and payment_stat.value >= upgrade.costValue
```

### 升级树系统
```gdscript
# 升级依赖树系统
class_name UpgradeTree
extends Node

@export var upgrades_component: UpgradesComponent
var upgrade_dependencies: Dictionary[StringName, Array[StringName]] = {}

func setup_dependencies():
    # 设置升级依赖关系
    upgrade_dependencies["advanced_weapon"] = ["basic_weapon"]
    upgrade_dependencies["master_skill"] = ["advanced_weapon", "speed_boost"]

func can_unlock_upgrade(upgrade_name: StringName) -> bool:
    if not upgrade_name in upgrade_dependencies:
        return true  # 无依赖的升级总是可解锁
    
    var required_upgrades = upgrade_dependencies[upgrade_name]
    for required in required_upgrades:
        if not upgrades_component.getUpgrade(required):
            return false
    
    return true

func get_unlockable_upgrades() -> Array[StringName]:
    var unlockable = []
    for upgrade_name in upgrade_dependencies.keys():
        if can_unlock_upgrade(upgrade_name):
            unlockable.append(upgrade_name)
    return unlockable
```

## 技术细节

### 升级处理顺序
1. 检查升级是否已存在
2. 查找支付所需的Stat
3. 调用upgrade.requestToAcquire()验证和扣费
4. 添加升级到数组和字典
5. 发出didAcquire信号
6. 调用upgrade.setAcquisition()
7. 执行upgrade.processPayload()应用效果

### 缓存机制
- upgradesDictionary提供O(1)查找性能
- getUpgrade()优先使用缓存，失败时fallback到线性搜索
- 缓存在findUpgrade()中自动更新

### 信号顺序
- 组件信号总是在Upgrade信号之前发出
- 确保信号处理器接收时状态一致性

## 注意事项

### 性能优化
- 使用getUpgrade()而非findUpgrade()获得更好性能
- 大量升级时考虑预缓存所有升级
- 避免重复的升级查找操作

### 资源管理
- shouldResetResourcesOnReady用于开发和测试
- 升级重置会复制资源避免共享状态
- 注意Upgrade资源的内存使用

### 线程安全
- 组件操作应在主线程进行
- 避免在Upgrade处理过程中修改upgrades数组

## 相关组件
- [`StatsComponent`](../Data/StatsComponent.md) - 统计数据管理
- [`Upgrade`](../../Resources/Upgrade.md) - 升级资源
- [`UpgradeChoiceUI`](../../UI/UpgradeChoiceUI.md) - 升级选择界面 