# StatsComponent

## 概述
`StatsComponent` 是一个统计数据管理组件，存储和管理实体的各种属性数据（如生命值、法力值、弹药等）。提供多种访问方式和便捷的操作接口。

**继承关系：** `StatsComponent` → `Component` → `Node`

## 主要特性
- 📊 统一管理多个Stat资源
- 🔍 支持名称和UID两种访问方式
- ⚡ 缓存机制优化性能
- 🛡️ 安全的访问方法避免运行时崩溃
- 🎮 便捷的UI集成接口
- 🔄 支持统计数据重置

## 导出属性

### `stats: Array[Stat]`
包含实体所有统计数据的数组。

**类型：** `Array[Stat]`  
**特性：** 设置时自动调用 `cacheStats()` 更新缓存  
**用途：** 定义实体的所有统计属性

### `shouldResetResourcesOnReady: bool = false`
是否在Ready时自动重置所有统计数据到默认值。

**类型：** `bool`  
**默认值：** `false`  
**用途：** 用于游戏重启等场景的自动重置

## 状态属性

### `statsDictionary: Dictionary[StringName, Stat]`
按名称缓存的统计数据字典，提供快速访问。

**访问方式：** `statsComponent.statsDictionary.health.value`  
**注意：** 建议使用 `getStat()` 方法安全访问

### `statsUIDDictionary: Dictionary[StringName, Stat]`
按ResourceUID缓存的统计数据字典，提供名称无关的访问。

**访问方式：** `statsComponent.statsUIDDictionary.bah69kvu0xeae.value`  
**注意：** UID前缀"uid://"已被移除

## 核心方法

### `cacheStats() -> int`
将stats数组中的统计数据缓存到字典中以提高访问性能。

**返回：** 缓存的统计数据数量  
**功能：** 
- 清除旧缓存
- 按名称和UID建立新缓存
- 自动在stats属性变更时调用

### `resetStats() -> void`
重置所有统计数据到默认值。

**实现：** 对每个stat执行 `duplicate()` 操作

## 访问接口

### `getStat(statName: StringName) -> Stat`
获取指定名称的统计数据，优先从缓存查找。

**参数：** `statName` - 统计数据名称  
**返回：** 对应的Stat对象，未找到时返回null  
**性能：** 缓存查找优于数组扫描

### `getStatByUID(uidPath: StringName) -> Stat`
通过ResourceUID获取统计数据，支持名称无关访问。

**参数：** `uidPath` - 资源UID路径（如"uid://bah69kvu0xeae"）  
**返回：** 对应的Stat对象，未找到时返回null  
**优势：** 即使Stat名称或文件名改变也能正确引用

### `getStatIfHasValue(statName: StringName, requiredValue: int) -> Stat`
获取指定统计数据（仅当其值大于等于要求值时）。

**参数：**
- `statName` - 统计数据名称
- `requiredValue` - 最低要求值

**返回：** 满足条件的Stat对象，否则返回null

### `findStat(nameToSearch: StringName) -> Stat`
在stats数组中查找指定名称的统计数据。

**参数：** `nameToSearch` - 要查找的名称  
**性能：** 比getStat慢，但会将找到的结果缓存  
**用途：** 当缓存中没有时的备用查找方法

## 资源管理

### `canSpend(statName: StringName, amount: int) -> bool`
检查是否有足够的资源可以消耗。

**参数：**
- `statName` - 统计数据名称
- `amount` - 要消耗的数量

**返回：** true表示资源充足

### `spend(statName: StringName, amount: int) -> bool`
消耗指定数量的资源。

**参数：**
- `statName` - 统计数据名称  
- `amount` - 消耗数量（负数表示增加）

**返回：** true表示消耗成功  
**行为：** 只有资源充足时才会执行消耗

## 便捷函数

### `getStatValue(statName: StringName) -> int`
获取指定统计数据的数值。

**返回：** 统计数据的值，未找到时返回0

### `changeStatValue(statName: StringName, difference: int) -> void`
修改指定统计数据的值。

**参数：**
- `statName` - 统计数据名称
- `difference` - 变化量（正负均可）

### `setStatToMax(statName: StringName) -> void`
将指定统计数据设置为最大值。

### `setStatToMin(statName: StringName) -> void`
将指定统计数据设置为最小值。

## 使用示例

### 基本设置
```gdscript
# 获取StatsComponent
var statsComponent = $StatsComponent

# 安全获取生命值
var healthStat = statsComponent.getStat("health")
if healthStat:
    print("当前生命值:", healthStat.value)
```

### 资源消耗系统
```gdscript
# 检查并消耗弹药
func fireWeapon():
    var statsComponent = $StatsComponent
    if statsComponent.canSpend("ammo", 1):
        if statsComponent.spend("ammo", 1):
            print("开火成功！")
        else:
            print("消耗失败")
    else:
        print("弹药不足")
```

### UI集成示例
```gdscript
# 连接到UI按钮信号
func _ready():
    var statsComponent = $StatsComponent
    # 生命值加满按钮
    $UI/MaxHealthButton.pressed.connect(
        statsComponent.setStatToMax.bind("health")
    )
    # 法力值减少按钮
    $UI/DecreaseManaButton.pressed.connect(
        statsComponent.changeStatValue.bind("mana", -10)
    )
```

### UID访问示例
```gdscript
# 使用UID进行名称无关访问
var healthByUID = statsComponent.getStatByUID("uid://bah69kvu0xeae")
if healthByUID:
    print("通过UID获取的生命值:", healthByUID.value)
```

### 批量操作
```gdscript
# 遍历所有统计数据
func debugAllStats():
    var statsComponent = $StatsComponent
    for statName in statsComponent.statsDictionary:
        var stat = statsComponent.statsDictionary[statName]
        print(statName, ": ", stat.value, "/", stat.max)
```

## 设计模式

### 缓存策略
- **主缓存：** statsDictionary提供按名称的快速访问
- **UID缓存：** statsUIDDictionary提供名称无关的稳定访问
- **自动更新：** stats数组变更时自动重建缓存

### 安全访问
- 使用 `getStat()` 而不是直接访问字典
- 在操作前检查Stat对象是否为空
- 使用 `canSpend()` 在消耗前验证资源

### UI集成
- 便捷函数支持直接绑定到UI信号
- 使用 `UI/Lists/StatsList.gd` 自动显示和更新

## 注意事项

1. **性能优化：** 访问频繁的统计数据会被自动缓存
2. **空值检查：** 始终检查getStat()的返回值
3. **UID稳定性：** UID访问不受名称变更影响
4. **资源消耗：** spend()方法包含安全检查
5. **UI友好：** 便捷函数适合绑定到UI事件

## 相关组件
- `StatModifierComponent` - 定时修改统计数据
- `HealthComponent` - 专门的生命值管理
- `StatModifierOnDeathComponent` - 死亡时的统计数据修改
- `UI/Lists/StatsList` - 统计数据列表UI组件