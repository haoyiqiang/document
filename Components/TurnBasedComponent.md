# TurnBasedComponent

## 概述
`TurnBasedComponent` 是所有回合制组件的抽象基类，为回合制游戏系统提供核心框架。每回合，父`TurnBasedEntity`会按顺序在其所有组件上调用`processTurnBegin`、`processTurn`和`processTurnEnd`方法。

**继承关系：** `TurnBasedComponent` → `Component` → `Node`
**类型：** 抽象基类 (abstract class)

## 主要特性
- 🔄 标准化的回合制处理流程
- 📊 内置回合状态管理
- 🎯 信号驱动的事件系统
- ⏸️ 支持异步动画和延迟
- 🎮 三阶段回合处理架构
- 🔧 可配置的启用/禁用控制

## 依赖要求
- **TurnBasedEntity** (必需) - 父实体必须是回合制实体
- **AnimatedSprite2D** (推荐) - 用于回合制动画

## 导出属性

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**功能：** 禁用时跳过所有回合处理方法

## 状态属性

### `currentTurn: int` (只读)
当前回合数。

**类型：** `int`  
**来源：** `TurnBasedCoordinator.currentTurn`  
**注意：** 只读属性，不可直接设置

### `currentTurnState: TurnBasedCoordinator.TurnBasedState` (只读)
当前回合状态。

**类型：** `TurnBasedCoordinator.TurnBasedState`  
**来源：** `TurnBasedCoordinator.currentTurnState`  
**状态值：** 回合制协调器定义的状态枚举

### `turnsProcessed: int` (只读)
已处理的回合总数。

**类型：** `int`  
**来源：** `TurnBasedCoordinator.turnsProcessed`  
**用途：** 统计和调试信息

## 信号

### 回合开始信号
- **`willBeginTurn`** - 即将开始回合时发出
- **`didBeginTurn`** - 回合开始处理完成后发出

### 回合更新信号
- **`willUpdateTurn`** - 即将执行回合主逻辑时发出
- **`didUpdateTurn`** - 回合主逻辑完成后发出

### 回合结束信号
- **`willEndTurn`** - 即将结束回合时发出
- **`didEndTurn`** - 回合结束处理完成后发出

## 核心方法

### 信号处理方法 (不可重写)

#### `processTurnBeginSignals() -> void`
处理回合开始的信号流。

**警告：** 子类不得重写此方法  
**流程：**
1. 检查启用状态
2. 发出`willBeginTurn`信号
3. 调用`processTurnBegin()`
4. 发出`didBeginTurn`信号

#### `processTurnUpdateSignals() -> void`
处理回合更新的信号流。

**警告：** 子类不得重写此方法  
**流程：**
1. 检查启用状态
2. 发出`willUpdateTurn`信号
3. 调用`processTurnUpdate()`
4. 发出`didUpdateTurn`信号

#### `processTurnEndSignals() -> void`
处理回合结束的信号流。

**警告：** 子类不得重写此方法  
**流程：**
1. 检查启用状态
2. 发出`willEndTurn`信号
3. 调用`processTurnEnd()`
4. 发出`didEndTurn`信号

### 抽象方法 (需要子类实现)

#### `processTurnBegin() -> void`
回合开始时的预处理活动。

**用途：** 动画、治疗效果、状态准备等  
**示例：** 回合开始动画、回血、解除某些状态  
**子类必须实现**

#### `processTurnUpdate() -> void`
回合的主要活动。

**用途：** 移动、战斗、主要游戏逻辑  
**示例：** 角色移动、攻击行为、技能释放  
**子类必须实现**

#### `processTurnEnd() -> void`
回合结束时的后处理活动。

**用途：** 动画、伤害效果、日志记录、清理  
**示例：** 毒素伤害、回合结束动画、状态清理  
**子类必须实现**

## 使用示例

### 基本回合制组件实现
```gdscript
# 简单的回合制移动组件
class_name TurnBasedMovementComponent
extends TurnBasedComponent

@export var movementSpeed: int = 2
@export var movementAnimation: String = "walk"

var targetPosition: Vector2
var originalPosition: Vector2

func _ready():
    super._ready()
    originalPosition = parentEntity.global_position

func processTurnBegin():
    if debugMode: printLog("开始移动回合")
    
    # 播放准备动画
    var animatedSprite = parentEntity.get_node("AnimatedSprite2D")
    if animatedSprite:
        animatedSprite.play("idle")
    
    # 计算目标位置
    calculateTargetPosition()

func processTurnUpdate():
    if debugMode: printLog("执行移动")
    
    # 播放移动动画
    var animatedSprite = parentEntity.get_node("AnimatedSprite2D")
    if animatedSprite:
        animatedSprite.play(movementAnimation)
    
    # 执行移动
    await moveToTarget()

func processTurnEnd():
    if debugMode: printLog("移动回合结束")
    
    # 停止动画
    var animatedSprite = parentEntity.get_node("AnimatedSprite2D")
    if animatedSprite:
        animatedSprite.play("idle")

func calculateTargetPosition():
    # AI逻辑：选择移动方向
    var directions = [Vector2.UP, Vector2.DOWN, Vector2.LEFT, Vector2.RIGHT]
    var randomDirection = directions[randi() % directions.size()]
    targetPosition = parentEntity.global_position + randomDirection * movementSpeed * 32

func moveToTarget():
    var tween = create_tween()
    tween.tween_property(parentEntity, "global_position", targetPosition, 1.0)
    await tween.finished
```

## 设计模式

### 模板方法模式
- **标准流程：** 定义回合处理的标准三阶段流程
- **扩展点：** 子类实现具体的回合逻辑
- **不变部分：** 信号发送和状态管理由基类处理

### 观察者模式
- **信号系统：** 通过信号通知其他系统回合状态变化
- **松耦合：** UI和游戏逻辑通过信号解耦
- **事件驱动：** 基于事件的回合制架构

### 状态模式
- **回合状态：** 与TurnBasedCoordinator协作管理回合状态
- **组件状态：** 每个组件维护自己的回合内状态
- **全局状态：** 统一的回合制状态管理

## 技术细节

### 异步处理
所有回合处理方法支持`await`：
```gdscript
@warning_ignore("redundant_await")
await self.processTurnUpdate()
```

### 分阶段执行
回合制系统采用分阶段执行：
1. 所有组件执行Begin阶段
2. 所有组件执行Update阶段  
3. 所有组件执行End阶段

### 调试支持
自定义日志格式包含回合和阶段信息：
```gdscript
Debug.printLog(message, str(object, " ", TurnBasedCoordinator.logStateIndicator, self.currentTurn), "lightBlue", "cyan")
```

### 组自动加入
组件会自动加入回合制组：
```gdscript
self.add_to_group(Global.Groups.turnBased, true)
```

## 注意事项

1. **父实体要求：** 父实体必须是TurnBasedEntity
2. **方法实现：** 子类必须实现三个抽象方法
3. **信号方法：** 不要重写`processTurn*Signals()`方法
4. **异步支持：** 使用`await`处理动画和延迟
5. **启用检查：** `isEnabled = false`时跳过所有处理

## 常见应用场景
- 回合制移动系统
- 回合制战斗组件
- 状态效果管理
- 资源回复机制
- AI决策系统
- UI状态更新

## 相关组件
- `TurnBasedEntity` - 回合制实体，管理组件执行顺序
- `TurnBasedCoordinator` - 回合制协调器，全局状态管理
- `TurnBasedAnimationComponent` - 回合制动画组件
- `HealthComponent` - 生命值组件，常与回合制战斗配合