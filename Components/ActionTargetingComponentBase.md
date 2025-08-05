# ActionTargetingComponentBase 组件

## 概述

`ActionTargetingComponentBase` 是一个抽象基类，用于实现动作目标选择功能。它提示玩家或其他实体为特定的动作（Action）选择目标。

该组件通常用于需要选择目标的技能、法术或互动指令，例如：
- 技能/法术：如"火球术"，可以指向任何位置
- 互动指令：如"对话"或"检查"，需要目标为具有 `ActionTargetableComponent` 的实体

## 主要功能

### 核心机制
- **目标选择系统**：提供标准化的目标选择流程
- **UI 集成**：与全局 UI 系统集成，自动更新相关界面
- **信号系统**：通过信号与其他组件通信

### 状态管理
- **启用/禁用**：可动态控制组件的可用性
- **选择状态**：跟踪当前是否正在选择目标

## 导出属性

| 属性 | 类型 | 描述 |
|------|------|------|
| `action` | `Action` | 需要选择目标的动作 |
| `isEnabled` | `bool` | 是否启用组件（默认：true） |

## 信号

### didChooseTarget(entity: Entity)
当成功选择目标时触发。

**参数：**
- `entity`: 被选中的目标实体

### didCancel
当取消目标选择时触发。

## 主要方法

### chooseTarget(target: ActionTargetableComponent) -> Entity
选择目标并执行相应动作。

**参数：**
- `target`: 目标的 ActionTargetableComponent

**返回值：**
- `Entity`: 如果选择成功，返回目标实体；否则返回 `null`

**工作流程：**
1. 检查组件是否启用
2. 请求目标组件确认选择
3. 如果确认，发送相关信号并执行动作
4. 返回目标实体或 `null`

### cancelTargetSelection() -> void
取消目标选择并清理组件。

**执行操作：**
- 设置选择状态为 false
- 发送取消信号
- 通知全局 UI 系统
- 请求删除组件

## 依赖组件

- **ActionsComponent**: 用于执行选定的动作
- **ActionTargetableComponent**: 目标实体必须具有此组件才能被选择

## 使用示例

```gdscript
# 在场景中设置动作目标选择
@export var fireballAction: Action

func _ready():
    # 创建目标选择组件
    var targetingComponent = ActionTargetingComponentBase.new()
    targetingComponent.action = fireballAction
    targetingComponent.isEnabled = true
    
    # 连接信号
    targetingComponent.didChooseTarget.connect(_on_target_chosen)
    targetingComponent.didCancel.connect(_on_target_cancelled)
    
    # 添加到实体
    add_child(targetingComponent)

func _on_target_chosen(target_entity: Entity):
    print("选择了目标: ", target_entity.name)

func _on_target_cancelled():
    print("取消了目标选择")
```

## 注意事项

1. **抽象基类**：这是一个抽象基类，需要继承后实现具体的目标选择逻辑
2. **UI 同步**：组件会自动与全局 UI 系统同步状态
3. **生命周期管理**：取消选择时会自动请求删除组件
4. **权限检查**：只有当目标组件允许时才能选择目标

## 相关组件

- `ActionControlComponent`: 控制动作执行的组件
- `ActionsComponent`: 管理实体动作的组件
- `ActionTargetableComponent`: 可被选择为目标的组件
- `Action`: 动作资源类型

## 扩展建议

继承此基类时，通常需要实现：
- 具体的目标选择界面
- 目标高亮显示
- 范围检查逻辑
- 特殊的取消条件