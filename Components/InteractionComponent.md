# InteractionComponent

## 概述
`InteractionComponent` 是一个交互区域组件，当玩家输入交互动作时可以触发交互效果。它与拥有`InteractionControlComponent`的实体进行交互，支持自动交互、条件检查、载荷执行等功能。

**继承关系：** `InteractionComponent` → `Component` → `Node`

## 主要特性
- 🎯 基于Area2D的交互检测
- 🔄 载荷系统执行交互效果
- 🤖 自动交互支持
- 🏷️ 交互标签和描述系统
- 👁️ 可视化交互指示器
- ✅ 条件检查机制
- 🔧 高度可配置的交互行为

## 依赖节点
- **Area2D** (父节点) - 必须作为Area2D的子组件

## 导出属性

### `payload: Payload`
交互的效果载荷。

**类型：** `Payload`  
**描述：** 定义交互被触发时执行的具体效果  
**参数传递：**
- `source` - 此InteractionComponent
- `target` - InteractionControlComponent的父实体

### `automatic: bool = false`
是否自动触发交互。

**类型：** `bool`  
**默认值：** `false`  
**功能：** 启用时，任何InteractionControlComponent进入碰撞区域都会自动触发交互  
**适用场景：** 传送门、陷阱、自动收集等

### `interactionIndicator: CanvasItem`
交互指示器节点。

**类型：** `CanvasItem` (Node2D 或 Control)  
**用途：** 当有InteractionControlComponent在范围内时显示的视觉指示器  
**示例：** 按钮提示、光效、UI面板等

### `alwaysShowIndicator: bool`
是否始终显示指示器。

**类型：** `bool`  
**默认值：** `false`  
**功能：** true时即使没有玩家在范围内也显示指示器

### `labelText: String`
交互的简短标签。

**类型：** `String`  
**用途：** 在UI中显示的交互名称  
**示例：** "开门"、"砍伐树木"、"对话"  
**setter特性：** 修改时自动更新标签显示

### `description: String`
交互的详细描述。

**类型：** `String`  
**用途：** 在UI工具提示中显示的详细信息  
**示例：** "砍伐树木需要斧头，获得2个木材"  
**setter特性：** 修改时自动更新工具提示

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**setter特性：** 自动控制Area2D的监听状态和指示器可见性

## 状态属性

### `selfAsArea: Area2D` (只读)
获取父Area2D节点的引用。

**类型：** `Area2D`  
**获取方式：** `self.get_node(^".") as Area2D`  
**缓存：** 首次访问时缓存引用

## 信号

### `didEnterInteractionArea(entity: Entity, interactionControlComponent: InteractionControlComponent)`
当InteractionControlComponent进入交互区域时发出。

### `didExitInteractionArea(entity: Entity, interactionControlComponent: InteractionControlComponent)`
当InteractionControlComponent离开交互区域时发出。

### `didDenyInteraction(interactorEntity: Entity)`
当交互被拒绝时发出。

### `willPerformInteraction(interactorEntity: Entity)`
即将执行交互时发出。

### `didPerformInteraction(result: Variant)`
交互执行完成后发出。

## 核心方法

### `requestToInteract(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool`
处理交互请求。

**返回：** bool - 交互是否被批准  
**流程：**
1. 检查启用状态
2. 调用条件检查
3. 批准或拒绝交互

### `performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant`
执行交互。

**返回：** Variant - 载荷执行结果  
**流程：**
1. 发出`willPerformInteraction`信号
2. 执行载荷
3. 发出`didPerformInteraction`信号

### `checkInteractionConditions(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool`
检查交互条件（虚方法）。

**返回：** bool - 默认返回`isEnabled`  
**用途：** 子类可重写实现自定义条件检查

## 使用示例

### 基本交互设置
```gdscript
# 简单的门交互
func setupDoorInteraction():
    var interactionComponent = $InteractionComponent
    
    # 设置交互信息
    interactionComponent.labelText = "开门"
    interactionComponent.description = "需要钥匙才能开门"
    
    # 创建开门载荷
    var doorPayload = ActionPayload.new()
    doorPayload.action = "open_door"
    interactionComponent.payload = doorPayload
    
    # 连接信号
    interactionComponent.didPerformInteraction.connect(_on_door_opened)

func _on_door_opened(result):
    if result:
        print("门已打开!")
        $Door.play("open")
    else:
        print("门无法打开，可能需要钥匙")
```

### 条件交互系统
```gdscript
# 需要特定物品的交互
class_name ConditionalInteractionComponent
extends InteractionComponent

@export var requiredItem: String = "key"
@export var consumeItem: bool = true

func checkInteractionConditions(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> bool:
    if not super.checkInteractionConditions(interactorEntity, interactionControlComponent):
        return false
    
    # 检查玩家是否有所需物品
    var inventory = interactorEntity.get_node("InventoryComponent")
    if not inventory:
        return false
    
    if not inventory.hasItem(requiredItem):
        return false
    
    return true

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant:
    # 先执行基础交互
    var result = super.performInteraction(interactorEntity, interactionControlComponent)
    
    if result and consumeItem:
        # 交互成功时消耗物品
        var inventory = interactorEntity.get_node("InventoryComponent")
        if inventory:
            inventory.removeItem(requiredItem, 1)
    
    return result
```

## 设计模式

### 策略模式
- **载荷系统：** 通过不同的Payload实现不同的交互效果
- **条件检查：** 通过重写checkInteractionConditions实现不同条件
- **指示器策略：** 支持不同类型的可视化指示器

### 观察者模式
- **信号通知：** 交互状态变化时通知其他系统
- **事件驱动：** 基于信号的交互响应
- **松耦合：** UI和游戏逻辑通过信号解耦

### 模板方法模式
- **交互流程：** 定义标准的交互处理流程
- **扩展点：** 子类可重写条件检查和交互效果
- **不变部分：** 信号发送和基础逻辑由基类处理

## 注意事项

1. **节点结构：** 必须作为Area2D的子节点
2. **信号连接：** 需要连接Area2D的area_entered和area_exited信号
3. **载荷参数：** 载荷的source是InteractionComponent，target是交互者
4. **条件检查：** 子类重写条件检查时需调用super
5. **异步安全：** 使用set_deferred避免信号处理中的状态冲突

## 相关组件
- `InteractionControlComponent` - 交互控制组件，发起交互
- `InteractionWithCostComponent` - 带成本的交互组件扩展
- `CollectibleComponent` - 可收集物品组件，相似功能
- `InventoryComponent` - 物品容器，常用于交互条件检查