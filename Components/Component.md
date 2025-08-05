# Component API 文档

## 概述
`Component` 是组合框架的核心类。它是一个表示游戏角色或对象的独特行为或属性的节点。

由Component子节点组成的父节点是一个Entity。Entity是"脚手架"，而Components执行实际的工作。Components可以在不同类型的实体中重用，例如用于玩家角色和怪物的HealthComponent。

## 继承
`Component` → `Node`

## 主要特性
- 抽象基类，所有组件的父类
- 自动管理与父Entity的关系
- 支持组件间依赖关系验证
- 提供调试功能

## 导出属性

### 高级参数

#### `shouldCheckGrandparentsForEntity: bool = false`
**⚠️ 高级选项！**
如果父节点不是Entity，是否应该检查所有祖父/曾祖父节点直到在场景树层次结构中找到Entity？
被 `allowNonEntityParent` 覆盖。

#### `allowNonEntityParent: bool = false`
**⚠️ 高级选项！**
允许此组件添加到非Entity节点吗？
覆盖 `shouldCheckGrandparentsForEntity`。

## 核心属性

### `parentEntity: Entity`
此组件所属的父Entity。设置时会自动更新相关状态。

### `coComponents: Dictionary[StringName, Component]`
父Entity中其他组件的字典。
- 访问方式：`coComponents.ComponentClassName`
- 安全访问：`coComponents.get(&"ComponentClassName")` （如果组件不存在返回null）

## 信号

### `willRemoveFromEntity`
在 `NOTIFICATION_UNPARENTED` 时发出。子类可以连接此信号来执行特定的清理操作。

## 主要方法

### 验证方法

#### `getRequiredComponents() -> Array[Script]`
**必须被子类重写**
返回此组件依赖的其他组件类型列表。

#### `checkRequiredComponents() -> bool`
检查是否满足所有必需的组件依赖。

### 生命周期方法

#### `validateParent() -> void`
验证父节点并建立与Entity的关系。在NOTIFICATION_PARENTED时调用。

#### `findParentEntity(checkGrandparents: bool = shouldCheckGrandparentsForEntity) -> Entity`
在场景树中向上搜索Entity类型的父节点或祖父节点。

#### `registerEntity(newParentEntity: Entity) -> void`
将此组件注册到指定的Entity。

#### `removeFromEntity(shouldFree: bool = true) -> void`
从父Entity移除此组件，可选择是否释放内存。

## 使用示例

```gdscript
# 自定义组件示例
class_name MyCustomComponent
extends Component

# 重写必需组件依赖
func getRequiredComponents() -> Array[Script]:
    return [HealthComponent, MovementComponent]

# 在_ready中访问同级组件
func _ready() -> void:
    var healthComponent = coComponents.get(&"HealthComponent")
    if healthComponent:
        healthComponent.health_changed.connect(_on_health_changed)

func _on_health_changed(new_health: float) -> void:
    print("Health changed to: ", new_health)
```

## 注意事项

1. **抽象类**：Component是抽象类，不能直接实例化
2. **Entity关系**：组件应该总是添加到Entity节点下
3. **依赖检查**：重写`getRequiredComponents()`来声明组件依赖
4. **高级选项**：谨慎使用高级参数，可能影响性能或导致错误

## 相关类
- `Entity` - 组件的容器
- `NodeModifierComponentBase` - 用于修改节点的组件基类
- `DebugComponent` - 调试显示组件 