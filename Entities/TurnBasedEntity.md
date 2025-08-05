# TurnBasedEntity

## 概述
`TurnBasedEntity` 是回合制游戏实体的基类，继承自 `Entity` 类。专门为回合制游戏设计，提供了回合管理、行动点系统、状态控制等回合制特有的功能。

**继承关系：** `TurnBasedEntity` → `Entity` → `Node2D` → `Node`

## 主要特性
- ⏰ 回合制时间管理
- 🎯 行动点系统
- 📊 回合状态控制
- 🎮 回合制输入处理
- 🔄 回合切换逻辑
- 📋 行动队列管理

## 导出属性

### `turnOrder: int = 0`
回合顺序。

**类型：** `int`  
**默认值：** `0`  
**说明：** 决定实体在回合中的执行顺序

### `actionPoints: int = 3`
行动点数量。

**类型：** `int`  
**默认值：** `3`  
**说明：** 每回合可用的行动点

### `maxActionPoints: int = 3`
最大行动点数量。

**类型：** `int`  
**默认值：** `3`  
**说明：** 行动点的上限

### `isActive: bool = false`
是否处于活动状态。

**类型：** `bool`  
**默认值：** `false`  
**说明：** 当前是否为该实体的回合

## 状态属性

### `currentTurn: int = 0`
当前回合数。

**类型：** `int`  
**说明：** 跟踪当前回合进度

### `pendingActions: Array`
待执行行动队列。

**类型：** `Array`  
**说明：** 存储等待执行的行动

## 核心方法

### `_ready() -> void`
回合制实体准备就绪时调用。

**功能：**
1. 调用父类的 `_ready()` 方法
2. 初始化回合制特定功能
3. 设置回合状态

### `startTurn() -> void`
开始实体的回合。

**功能：**
1. 设置活动状态
2. 重置行动点
3. 处理回合开始逻辑
4. 发出回合开始信号

### `endTurn() -> void`
结束实体的回合。

**功能：**
1. 清除活动状态
2. 处理回合结束逻辑
3. 发出回合结束信号
4. 通知回合管理器

### `canTakeAction() -> bool`
检查是否可以执行行动。

**返回：** 是否可以执行行动  
**条件：**
- 实体处于活动状态
- 有足够的行动点
- 没有其他限制

### `spendActionPoints(cost: int) -> bool`
消耗行动点。

**参数：** `cost` - 消耗的行动点数量  
**返回：** 是否成功消耗  
**功能：**
1. 检查是否有足够的行动点
2. 扣除行动点
3. 更新状态

### `addAction(action: Dictionary) -> void`
添加行动到队列。

**参数：** `action` - 行动数据  
**功能：**
1. 验证行动有效性
2. 添加到待执行队列
3. 更新行动状态

### `executeActions() -> void`
执行所有待执行行动。

**功能：**
1. 遍历行动队列
2. 执行每个行动
3. 清理已执行行动
4. 更新实体状态

## 回合管理

### `onTurnStart() -> void`
回合开始时调用。

**功能：** 处理回合开始时的逻辑

### `onTurnEnd() -> void`
回合结束时调用。

**功能：** 处理回合结束时的逻辑

### `onActionExecuted(action: Dictionary) -> void`
行动执行后调用。

**参数：** `action` - 执行的行动数据  
**功能：** 处理行动执行后的逻辑

## 使用示例

### 基础回合制实体创建
```gdscript
# 创建回合制实体
var turnEntity = TurnBasedEntity.new()
turnEntity.turnOrder = 1
turnEntity.actionPoints = 2

# 添加到场景
add_child(turnEntity)
```

### 回合管理
```gdscript
# 开始回合
turnEntity.startTurn()

# 检查是否可以行动
if turnEntity.canTakeAction():
    # 执行行动
    turnEntity.spendActionPoints(1)
    
# 结束回合
turnEntity.endTurn()
```

### 行动系统
```gdscript
# 添加行动到队列
var action = {
    "type": "move",
    "target": Vector2(100, 100),
    "cost": 1
}
turnEntity.addAction(action)

# 执行所有行动
turnEntity.executeActions()
```

## 注意事项

1. **回合顺序** - 确保正确设置 `turnOrder` 以控制执行顺序
2. **行动点管理** - 合理分配和使用行动点
3. **状态同步** - 确保回合状态与游戏状态同步
4. **行动验证** - 在执行行动前验证其有效性
5. **性能考虑** - 回合制游戏通常需要精确的状态管理

## 相关组件

- **TurnBasedComponent** - 回合制基础组件
- **TurnBasedAnimationComponent** - 回合制动画组件
- **ActionComponent** - 行动管理组件
- **StatsComponent** - 属性管理组件 