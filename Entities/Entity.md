# Entity

## 概述
`Entity` 是 Comedot 框架的核心实体基类，采用组合模式来构建游戏对象。每个 Entity 由多个独立的、可重用的 Component 组成，Entity 本身作为"脚手架"，而 Component 负责实现具体的游戏逻辑。

**继承关系：** `Entity` → `Node2D` → `Node`

## 主要特性
- 🔧 自动组件管理和注册
- ⚡ 高性能组件查找（字典查找）
- 🎮 智能节点查找（Sprite、Area、Body）
- 🛡️ 完整的生命周期管理
- 📊 分级调试和日志系统
- 🧹 自动内存管理和资源释放
- ⚙️ 帧级函数调用控制

## 导出属性

### `isLoggingEnabled: bool = true`
是否启用日志输出。

**类型：** `bool`  
**默认值：** `true`  
**说明：** 控制普通日志输出，不影响警告和错误

### `debugMode: bool = false`
是否启用调试模式。

**类型：** `bool`  
**默认值：** `false`  
**说明：** 启用更详细的调试信息，包括详细日志

### `sprite: Node2D`
主要视觉表示节点。

**类型：** `Node2D`  
**默认值：** `null`  
**说明：** 如果为 null，会自动查找 AnimatedSprite2D 或 Sprite2D

### `area: Area2D`
主要区域节点。

**类型：** `Area2D`  
**默认值：** `null`  
**说明：** 如果为 null，会自动查找 Area2D 子节点

### `body: CharacterBody2D`
主要角色体节点。

**类型：** `CharacterBody2D`  
**默认值：** `null`  
**说明：** 如果为 null，会自动查找 CharacterBody2D 子节点

## 状态属性

### `components: Dictionary[StringName, Component]`
组件字典。

**类型：** `Dictionary[StringName, Component]`  
**说明：** 以组件类名为键的组件字典，用于快速查找

### `functionsAlreadyCalledOnceThisFrame: Dictionary[StringName, Callable]`
本帧已调用函数字典。

**类型：** `Dictionary[StringName, Callable]`  
**说明：** 用于确保某些函数每帧只调用一次

## 信号

### `preDelete`
实体即将删除时发出。

**时机：** 收到 `NOTIFICATION_PREDELETE` 通知时

## 生命周期方法

### `_ready() -> void`
实体准备就绪时调用。

**功能：** 输出调试信息

### `_enter_tree() -> void`
实体进入场景树时调用。

**功能：**
1. 添加到实体组
2. 连接信号
3. 输出日志信息

### `_process(delta: float) -> void`
每帧调用。

**功能：** 清理本帧已调用函数字典

### `_exit_tree() -> void`
实体退出场景树时调用。

**功能：** 输出退出日志

### `_notification(what: int) -> void`
处理系统通知。

**参数：** `what` - 通知类型  
**功能：**
- `NOTIFICATION_PREDELETE` - 发出 preDelete 信号
- `NOTIFICATION_UNPARENTED` - 输出日志

## 组件管理

### `registerComponent(newComponent: Component) -> bool`
注册组件到组件字典。

**参数：** `newComponent` - 要注册的组件  
**返回：** 是否成功注册  
**功能：**
1. 获取组件类型名
2. 检查是否已存在同类型组件
3. 如果存在，移除旧组件
4. 注册新组件
5. 设置父实体引用

### `unregisterComponent(componentToRemove: Component) -> bool`
从组件字典中注销组件。

**参数：** `componentToRemove` - 要注销的组件  
**返回：** 是否成功注销  
**功能：**
1. 获取组件类型名
2. 检查组件是否存在
3. 验证组件实例匹配
4. 从字典中移除

### `hasComponent(type: Script) -> bool`
检查是否拥有指定类型的组件。

**参数：** `type` - 组件类型  
**返回：** 是否拥有该组件

### `getComponent(type: Script, findSubclasses: bool = false) -> Component`
获取指定类型的组件。

**参数：**
- `type` - 组件类型
- `findSubclasses` - 是否查找子类

**返回：** 找到的组件，未找到返回 null

### `addComponent(component: Component) -> void`
添加现有组件实例。

**参数：** `component` - 要添加的组件  
**功能：**
1. 添加为子节点
2. 设置所有者

### `addComponents(componentsToAdd: Array[Component]) -> int`
批量添加组件。

**参数：** `componentsToAdd` - 要添加的组件数组  
**返回：** 添加的组件数量  
**注意：** 组件必须按依赖顺序添加

### `createNewComponent(type: Script) -> Component`
创建新组件实例。

**参数：** `type` - 组件类型  
**返回：** 创建的组件实例  
**功能：**
1. 根据类型获取场景路径
2. 加载并实例化场景
3. 添加到实体

### `createNewComponents(componentTypesToCreate: Array[Script]) -> Array[Component]`
批量创建组件。

**参数：** `componentTypesToCreate` - 要创建的组件类型数组  
**返回：** 创建的组件实例数组

### `findChildrenComponents() -> Array[Component]`
查找所有子组件。

**返回：** 子组件数组  
**警告：** 可能较慢，建议使用 components 字典

### `findFirstComponentSubclass(type: Script) -> Component`
查找第一个匹配的子类组件。

**参数：** `type` - 基类类型  
**返回：** 找到的组件，未找到返回 null

### `removeComponent(componentType: Script, shouldFree: bool = true) -> bool`
移除指定类型的组件。

**参数：**
- `componentType` - 组件类型
- `shouldFree` - 是否释放组件

**返回：** 是否成功移除

### `removeComponents(componentTypes: Array[Script], shouldFree: bool = true) -> int`
批量移除组件。

**参数：**
- `componentTypes` - 要移除的组件类型数组
- `shouldFree` - 是否释放组件

**返回：** 移除的组件数量

## 节点查找

### `findFirstChildOfType(type: Variant, includeEntity: bool = true) -> Node`
查找第一个匹配类型的子节点。

**参数：**
- `type` - 节点类型
- `includeEntity` - 是否包含实体本身

**返回：** 找到的节点

### `findFirstChildOfAnyTypes(types: Array[Variant], returnEntityIfNoMatches: bool = true) -> Node`
查找第一个匹配任意类型的子节点。

**参数：**
- `types` - 节点类型数组
- `returnEntityIfNoMatches` - 无匹配时是否返回实体本身

**返回：** 找到的节点

### `findChildrenOfType(type: Variant) -> Array`
查找所有匹配类型的子节点。

**参数：** `type` - 节点类型  
**返回：** 匹配的节点数组

### `addSceneCopy(path: String) -> Node`
添加场景副本。

**参数：** `path` - 场景路径  
**返回：** 添加的节点

### `removeChildrenOfType(type: Variant, shouldFree: bool = true) -> int`
移除指定类型的所有子节点。

**参数：**
- `type` - 节点类型
- `shouldFree` - 是否释放节点

**返回：** 移除的节点数量

## 懒加载属性

### `getSprite() -> Node2D`
获取精灵节点。

**返回：** 精灵节点  
**功能：**
1. 如果 sprite 为 null，自动查找
2. 优先查找 AnimatedSprite2D，其次 Sprite2D
3. 缓存结果

### `getArea() -> Area2D`
获取区域节点。

**返回：** 区域节点  
**功能：**
1. 如果 area 为 null，自动查找
2. 优先检查实体本身是否为 Area2D
3. 其次查找子节点
4. 缓存结果

### `getBody() -> CharacterBody2D`
获取角色体节点。

**返回：** 角色体节点  
**功能：**
1. 如果 body 为 null，自动查找
2. 优先检查实体本身是否为 CharacterBody2D
3. 其次查找子节点
4. 缓存结果

## 工具方法

### `callOnceThisFrame(function: Callable, arguments: Array = []) -> void`
确保函数每帧只调用一次。

**参数：**
- `function` - 要调用的函数
- `arguments` - 函数参数

**功能：**
1. 检查函数是否已在本帧调用
2. 如果未调用，添加到已调用列表
3. 调用函数
4. 启用处理以在下一帧清理

### `requestDeletion() -> bool`
请求删除实体。

**返回：** 是否成功请求删除  
**功能：** 调用 `queue_free()`

## 日志系统

### `printLog(message: String = "", object: Variant = self.logName) -> void`
输出普通日志。

**参数：**
- `message` - 日志消息
- `object` - 日志对象

**条件：** 需要 `isLoggingEnabled` 为 true

### `printDebug(message: String = "") -> void`
输出调试日志。

**参数：** `message` - 调试消息  
**条件：** 需要 `debugMode` 为 true

### `printWarning(message: String = "") -> void`
输出警告日志。

**参数：** `message` - 警告消息  
**注意：** 不受 `isLoggingEnabled` 影响

### `printError(message: String = "") -> void`
输出错误日志。

**参数：** `message` - 错误消息  
**注意：** 不受 `isLoggingEnabled` 影响

### `printChange(variableName: String, previousValue: Variant, newValue: Variant, logAsDebug: bool = true) -> void`
输出变量变化日志。

**参数：**
- `variableName` - 变量名
- `previousValue` - 之前的值
- `newValue` - 新的值
- `logAsDebug` - 是否作为调试日志输出

**条件：** 需要 `debugMode` 为 true 且值发生变化

## 计算属性

### `logName: String`
日志名称。

**返回：** 实体名称的格式化字符串

### `logFullName: String`
完整日志名称。

**返回：** 包含节点名、实例和类名的详细信息

## 使用示例

### 基础实体创建
```gdscript
# 创建实体
var entity = Entity.new()

# 添加组件
entity.addComponent(HealthComponent.new())
entity.addComponent(MovementComponent.new())

# 添加到场景
add_child(entity)
```

### 组件管理
```gdscript
# 获取组件
var healthComponent = entity.getComponent(HealthComponent)
if healthComponent:
    healthComponent.heal(10)

# 检查组件存在
if entity.hasComponent(InputComponent):
    print("Entity has input component")

# 移除组件
entity.removeComponent(HealthComponent)
```

### 节点查找
```gdscript
# 获取精灵节点
var sprite = entity.getSprite()
if sprite:
    sprite.play("idle")

# 获取区域节点
var area = entity.getArea()
if area:
    area.monitoring = true

# 获取角色体节点
var body = entity.getBody()
if body:
    body.velocity = Vector2(100, 0)
```

### 批量操作
```gdscript
# 批量添加组件
var components = [
    HealthComponent.new(),
    MovementComponent.new(),
    InputComponent.new()
]
entity.addComponents(components)

# 批量创建组件
var componentTypes = [
    HealthComponent,
    MovementComponent,
    InputComponent
]
entity.createNewComponents(componentTypes)
```

## 注意事项

1. **组件注册** - 组件必须在 `NOTIFICATION_PARENTED` 时调用 `registerComponent()`
2. **组件注销** - 组件必须在 `NOTIFICATION_UNPARENTED` 时调用 `unregisterComponent()`
3. **依赖顺序** - 添加组件时必须按依赖顺序添加
4. **性能考虑** - 使用 `components` 字典而非 `findChildrenComponents()`
5. **内存管理** - 移除组件时注意是否应该释放
6. **调试模式** - 开发时启用 `debugMode` 获取详细信息

## 相关组件

- **Component** - 基础组件类
- **HealthComponent** - 生命值组件
- **MovementComponent** - 移动组件
- **InputComponent** - 输入组件 