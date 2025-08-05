# NodeModifierComponentBase API 文档

## 概述
`NodeModifierComponentBase` 是用于添加或移除其他组件、节点或移除父Entity本身的组件基类。

## 继承
`NodeModifierComponentBase` → `Component` → `Node`

## 主要特性
- 动态添加/移除组件和节点
- 支持移除整个Entity
- 执行自定义Payload
- 有序的修改操作流程

## 导出属性

#### `shouldRemoveEntity: bool`
是否移除整个Entity。注意：这会阻止组件或节点的添加/移除操作。

#### `nodesToRemove: Array[Node]`
要移除的节点数组。
- 执行顺序：在 `componentsToRemove` 之前
- 被 `shouldRemoveEntity` 覆盖

#### `componentsToRemove: Array[Script]`
要移除的组件类型数组。
- 执行顺序：在 `nodesToRemove` 之后，`componentsToCreate` 之前
- 被 `shouldRemoveEntity` 覆盖

#### `componentsToCreate: Array[Script]`
要创建的组件类型数组。
- 执行顺序：在 `componentsToRemove` 之后
- 被 `shouldRemoveEntity` 覆盖

#### `payload: Payload`
可选的Payload执行器。
- `source` 是此组件的父Entity
- `target` 依赖于子类实现（默认为null）
- 最后执行

#### `isEnabled: bool = true`
是否启用此组件的功能。

## 状态属性

#### `savedParentEntity: Entity`
保存的父Entity引用。在组件自身被移除时保持Entity引用有效。

## 信号

### `willRemoveEntity`
即将移除Entity时发出。

### `didAddComponents(components: Array[Component])`
添加组件完成后发出，传递新创建的组件数组。

## 主要方法

### 核心操作方法

#### `removeEntity() -> void`
移除整个父Entity。
- 检查 `isEnabled` 和 `shouldRemoveEntity`
- 发出 `willRemoveEntity` 信号
- 调用 `requestDeletionOfParentEntity()`

#### `removeNodes() -> void`
移除指定的节点。
- 从父节点移除并调用 `queue_free()`
- 保存父Entity引用

#### `removeComponents() -> void`
移除指定类型的组件。
- 调用Entity的 `removeComponents()` 方法
- 保存父Entity引用

#### `createComponents(entityOverride: Entity = savedParentEntity) -> Array[Component]`
创建新组件。
- 返回新创建的组件数组
- 发出 `didAddComponents` 信号
- 可指定目标Entity（默认使用保存的父Entity）

#### `executePayload(target: Variant) -> void`
执行Payload。
- 使用父Entity作为source
- target参数可自定义

#### `performAllModifications() -> void`
按顺序执行所有修改操作：
1. 如果 `shouldRemoveEntity` 为true，只执行 `removeEntity()`
2. 否则依次执行：
   - `removeNodes()`
   - `removeComponents()`
   - `createComponents()`
   - `executePayload(null)`

## 使用示例

### 基本用法
```gdscript
# 创建修改器组件
var modifier = NodeModifierComponentBase.new()

# 配置要移除的组件
modifier.componentsToRemove = [OldMovementComponent, OldHealthComponent]

# 配置要添加的组件
modifier.componentsToCreate = [NewMovementComponent, NewHealthComponent]

# 连接信号
modifier.didAddComponents.connect(_on_components_added)

# 执行修改
modifier.performAllModifications()
```

### 移除Entity示例
```gdscript
# 配置为移除整个Entity
modifier.shouldRemoveEntity = true
modifier.willRemoveEntity.connect(_on_entity_will_be_removed)
modifier.performAllModifications()
```

### 带Payload的示例
```gdscript
# 创建自定义Payload
var customPayload = MyCustomPayload.new()
modifier.payload = customPayload

# 设置组件修改
modifier.componentsToCreate = [SomeComponent]
modifier.performAllModifications()
# Payload将在组件创建后执行
```

## 执行顺序

1. **Entity移除模式**（如果 `shouldRemoveEntity = true`）：
   - 只执行 `removeEntity()`

2. **常规修改模式**：
   1. `removeNodes()` - 移除节点
   2. `removeComponents()` - 移除组件
   3. `createComponents()` - 创建组件
   4. `executePayload(null)` - 执行Payload

## 注意事项

1. **Entity引用**：组件保存父Entity引用，防止在自身被移除时失效
2. **启用状态**：所有操作都会检查 `isEnabled` 状态
3. **信号时机**：信号在适当的时机发出，便于外部系统响应
4. **子类实现**：这是抽象基类，通常需要子类来定义具体的触发条件

## 相关子类示例
- `ModifyOnCollisionComponent` - 基于物理碰撞的修改
- `ModifyOnTimerComponent` - 基于定时器的修改

## 相关类
- `Component` - 基础组件类
- `Entity` - 实体容器
- `Payload` - 载荷执行器 