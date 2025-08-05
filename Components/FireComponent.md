# FireComponent

## 概述
`FireComponent` 是一个通用的发射组件，继承自 `CooldownComponent`，支持任何类型的攻击实体发射。通过 `bulletEntity` PackedScene 和 `fireMode` 配置，可以支持激光、闪电、子弹等任何攻击实体类型。支持单一实体（SINGLE）和多实体（MULTIPLE）两种发射模式。

**继承关系：** `FireComponent` → `CooldownComponent` → `Component` → `Node`

## 主要特性
- 🎯 通用发射系统，支持任何继承自 Entity 的攻击实体
- 🔄 双发射模式：单一实体和多实体模式
- ⚡ 自动射击模式支持
- 🎮 弹药系统集成
- 🎨 阵营系统自动传递
- 🛡️ 伤害组件自动配置
- 🧹 自动清理无效实体，防止内存泄露
- 📊 完整的调试信息输出

## 依赖节点
- **Pivot/GunSprite/BulletEmitter: Marker2D** (子节点) - 发射位置标记

## 发射模式枚举

### `FireMode`
```gdscript
enum FireMode {
	SINGLE,         ## 单发射击模式 - 实体销毁，才会创建新的实体，使用场景：一次性箭矢
	MULTIPLE,       ## 连发射击模式 - 允许多个实体同时存在，使用场景：连发射击子弹
}
```

## 导出属性

### `fireMode: FireMode = FireMode.SINGLE`
发射模式配置。

**类型：** `FireMode`  
**默认值：** `FireMode.SINGLE`  
**说明：** 控制发射行为模式，影响实体生命周期管理

### `bulletEntity: PackedScene`
攻击实体场景资源。

**类型：** `PackedScene`  
**默认值：** `null`  
**要求：** 必须继承自 `Entity` 类  
**用途：** 定义要发射的攻击实体类型

### `bulletConfig: EntityConfig`
攻击实体配置资源。

**类型：** `EntityConfig`  
**默认值：** `null`  
**用途：** 用于设置实体的 export 属性，配置实体行为

### `ammo: Stat`
弹药统计资源（可选）。

**类型：** `Stat`  
**默认值：** `null`  
**用途：** 管理弹药数量，支持弹药消耗机制

### `ammoCost: int = 0`
每次射击消耗的弹药数量。

**类型：** `int`  
**默认值：** `0`  
**说明：** 当 `ammo` 配置时，每次发射会消耗指定数量的弹药

### `ammoDepletedMessage: String = "AMMO DEPLETED"`
弹药耗尽时显示的消息。

**类型：** `String`  
**默认值：** `"AMMO DEPLETED"`  
**用途：** 当弹药耗尽时通过 `LabelComponent` 显示提示信息

### `isEnabled: bool = true`
是否启用组件。

**类型：** `bool`  
**默认值：** `true`  
**用途：** 控制组件是否响应发射请求

### `autoFire: bool = false`
自动射击模式。

**类型：** `bool`  
**默认值：** `false`  
**说明：** 启用后会在 `_process()` 中自动调用 `fire()` 方法

## 状态属性

### `bulletEntityList: Array[Entity]`
攻击实体列表。

**类型：** `Array[Entity]`  
**用途：** 维护所有当前存在的攻击实体，用于生命周期管理

### `fireEmitter: Marker2D`
发射位置节点引用。

**类型：** `Marker2D`  
**获取：** `@onready var fireEmitter: Marker2D = $Pivot/GunSprite/BulletEmitter`  
**用途：** 定义攻击实体的发射位置

## 依赖组件

### `labelComponent: LabelComponent`
标签组件引用，用于显示弹药耗尽消息。

**获取：** 通过 `coComponents.get(&"LabelComponent")` 获取  
**用途：** 显示弹药相关的提示信息

## 信号

### `fireStarted(entity: Entity)`
发射开始信号。

**参数：** `entity` - 刚创建的攻击实体  
**时机：** 攻击实体进入场景树时发出

### `fireCompleted(entity: Entity)`
发射完成信号。

**参数：** `entity` - 已销毁的攻击实体  
**时机：** 攻击实体离开场景树时发出

### `didDepleteAmmo`
弹药耗尽信号。

**时机：** 弹药从大于0变为0或以下时发出

### `ammoInsufficient`
弹药不足信号。

**时机：** 尝试发射但弹药不足时发出

## 核心方法

### `fire(config: EntityConfig = bulletConfig) -> Entity`
开始发射攻击。

**参数：** `config` - 可选的实体配置，默认使用 `bulletConfig`  
**返回：** 创建的实体实例，失败时返回 `null`  
**功能：**
1. 清理无效实体
2. 验证发射条件
3. 创建并配置攻击实体
4. 启动冷却计时
5. 返回创建的实体

### `createEntity(config: EntityConfig) -> Entity`
创建攻击实体（虚拟方法，可被子类重写）。

**参数：** `config` - 实体配置  
**返回：** 创建的实体实例  
**功能：**
1. 检查弹药消耗
2. 根据发射模式决定是否创建新实体
3. 实例化实体
4. 配置并添加到场景
5. 返回实体实例

### `configureEntity(entity: Entity, config: EntityConfig) -> void`
配置攻击实体（虚拟方法，可被子类重写）。

**参数：** 
- `entity` - 要配置的实体
- `config` - 配置资源

**功能：**
1. 应用配置资源到实体
2. 设置调试标志
3. 连接生命周期信号
4. 配置伤害组件
5. 传递阵营信息

### `addEntityToScene(entity: Entity) -> void`
将攻击实体添加到场景。

**参数：** `entity` - 要添加的实体  
**功能：** 将实体添加为 `fireEmitter` 的子节点

### `cleanupInvalidEntities() -> void`
清理无效的实体，防止内存泄露。

**功能：**
1. 遍历 `bulletEntityList`
2. 检查实体有效性
3. 移除无效实体
4. 输出清理统计信息

### `useAmmo() -> bool`
使用弹药。

**返回：** 是否成功消耗弹药  
**功能：**
1. 检查是否需要消耗弹药
2. 验证弹药是否足够
3. 扣除弹药数量
4. 检查是否耗尽弹药
5. 发出相应信号

## 公共接口

### `canFire() -> bool`
检查是否可以发射。

**返回：** 是否可以发射  
**条件：** 组件启用且冷却已完成

### `getCurrentFireMode() -> FireMode`
获取当前发射模式。

**返回：** 当前的发射模式枚举值

### `getFireModeName() -> String`
获取发射模式名称（用于调试）。

**返回：** 发射模式的字符串表示

## 配置验证

### `validateFireModeConfiguration() -> bool`
验证发射模式配置。

**返回：** 配置是否有效  
**功能：**
1. 检查发射模式有效性
2. 验证攻击实体配置
3. 根据模式提供建议配置
4. 输出验证结果

## 调试功能

### `printFireDebugInfo() -> void`
输出调试信息。

**功能：**
1. 显示发射模式信息
2. 显示冷却状态
3. 列出所有攻击实体状态
4. 显示弹药信息（如果配置）

## 信号处理

### `_on_bullet_entity_tree_entered(node: Node) -> void`
攻击实体进入场景树时调用。

**参数：** `node` - 进入场景树的节点  
**功能：**
1. 验证节点类型
2. 发出 `fireStarted` 信号
3. 添加到实体列表

### `_on_bullet_entity_ready(entity: Entity) -> void`
攻击实体准备好时调用。

**参数：** `entity` - 准备好的实体  
**功能：** 输出调试信息

### `_on_bullet_entity_tree_exited(node: Node) -> void`
攻击实体离开场景树时调用。

**参数：** `node` - 离开场景树的节点  
**功能：**
1. 验证节点类型
2. 发出 `fireCompleted` 信号
3. 从实体列表移除

### `_on_entity_tree_exiting(entity: Entity) -> void`
实体即将离开场景树时的备用清理机制。

**参数：** `entity` - 即将离开的实体  
**功能：** 确保实体从列表中移除

## 使用示例

### 基础配置
```gdscript
# 配置单一发射模式
var fireComponent = FireComponent.new()
fireComponent.fireMode = FireComponent.FireMode.SINGLE
fireComponent.bulletEntity = preload("res://Entities/Bullet.tscn")
fireComponent.cooldown = 1.0
```

### 多发射模式配置
```gdscript
# 配置连发射击模式
var fireComponent = FireComponent.new()
fireComponent.fireMode = FireComponent.FireMode.MULTIPLE
fireComponent.bulletEntity = preload("res://Entities/Laser.tscn")
fireComponent.cooldown = 0.2
```

### 弹药系统集成
```gdscript
# 配置弹药消耗
var fireComponent = FireComponent.new()
fireComponent.ammo = ammoStat
fireComponent.ammoCost = 1
fireComponent.ammoDepletedMessage = "弹药耗尽！"
```

### 自动射击
```gdscript
# 启用自动射击
var fireComponent = FireComponent.new()
fireComponent.autoFire = true
fireComponent.cooldown = 0.5
```

## 注意事项

1. **内存管理：** 组件会自动清理无效实体，但建议定期调用 `cleanupInvalidEntities()`
2. **发射模式选择：** 
   - `SINGLE` 模式适合一次性攻击（如箭矢）
   - `MULTIPLE` 模式适合连发射击（如机关枪）
3. **弹药系统：** 需要配置 `ammo` 和 `ammoCost` 才能启用弹药消耗
4. **阵营传递：** 会自动将父实体的阵营信息传递给攻击实体
5. **伤害组件：** 会自动设置攻击实体的伤害组件发起者
6. **调试模式：** 启用 `debugMode` 可查看详细的发射信息

## 继承扩展

`FireComponent` 设计为可扩展的基类，子类可以重写以下方法来自定义行为：

- `createEntity()` - 自定义实体创建逻辑
- `configureEntity()` - 自定义实体配置逻辑
- `addEntityToScene()` - 自定义场景添加逻辑

## 相关组件

- **CooldownComponent** - 提供冷却时间管理
- **DamageComponent** - 伤害系统集成
- **FactionComponent** - 阵营系统集成
- **LabelComponent** - 消息显示支持 