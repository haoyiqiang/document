# TurnBasedStateUIComponent API

## 概述
`TurnBasedStateUIComponent` 显示回合制系统状态的UI界面，响应父`TurnBasedEntity`和全局`TurnBasedCoordinator`的信号。用于创建回合制游戏的状态显示界面。

**继承链**：`Component` → `TurnBasedComponent` → `TurnBasedStateUIComponent`

**⚠️ 重要**：仅在一个"主控"实体上使用此组件来管理回合制UI。

**🧪 实验性功能**：此组件仍在开发中，API可能会变更。

## 依赖要求
- **TurnBasedEntity**：必须是回合制实体
- **TurnBasedCoordinator**：全局回合制协调器

## 主要特性
- 📊 **状态显示**：实时显示当前回合状态
- 🎨 **颜色指示**：不同回合阶段的颜色反馈
- ⏱️ **延迟控制**：可覆盖协调器的延迟设置
- 📝 **消息系统**：可选的状态消息显示
- 🔄 **同步更新**：自动同步回合制系统状态

## 导出属性

### 延迟配置
- **delayBetweenEntities** (`float`)：实体间延迟，覆盖协调器设置
- **delayBetweenStates** (`float`)：状态间延迟，覆盖协调器设置

### 视觉配置
- **colorBegin** (`Color = Color.GREEN`)：开始阶段颜色
- **colorUpdate** (`Color = Color.YELLOW`)：更新阶段颜色
- **colorEnd** (`Color = Color.ORANGE`)：结束阶段颜色

### 显示配置
- **shouldShowMessages** (`bool = false`)：是否显示状态消息
- **shouldRepeatMessageAsLog** (`bool = false`)：是否同时输出到日志

## UI元素
- **turnLabel** (`Label`)：显示当前回合信息
- **stateColorRect** (`ColorRect`)：状态颜色指示器

## 主要方法

### UI更新
- **updateUI**() → `void`：更新UI显示
- **displayMessage**(message: String, color: Color, outlineSize: int) → `void`：显示状态消息

### 回合处理
- **processTurnBegin**() → `void`：处理回合开始
- **processTurnUpdate**() → `void`：处理回合更新
- **processTurnEnd**() → `void`：处理回合结束

## 使用示例

### 基础UI设置
```gdscript
# 设置基本的回合制UI
@export var ui_component: TurnBasedStateUIComponent

func _ready():
    ui_component.shouldShowMessages = true
    ui_component.delayBetweenEntities = 0.5
    ui_component.delayBetweenStates = 1.0
```

### 自定义颜色主题
```gdscript
# 设置自定义的状态颜色
@export var ui_component: TurnBasedStateUIComponent

func setup_dark_theme():
    ui_component.colorBegin = Color.CYAN
    ui_component.colorUpdate = Color.MAGENTA
    ui_component.colorEnd = Color.RED
```

## 技术细节

### 信号连接
- 连接TurnBasedCoordinator的所有状态信号
- 监听父实体的回合生命周期事件
- 自动同步UI状态与系统状态

### UI更新机制
- turnLabel显示当前和下一个实体
- stateColorRect根据当前状态变色
- 支持实验性的定时器条功能

### 消息系统
- 可选的临时标签消息显示
- 支持日志输出重复
- 颜色编码的状态消息

## 注意事项

### 单例使用
- 每个场景只应有一个此组件实例
- 多个实例可能导致UI冲突
- 建议放在UI管理器实体上

### 性能考虑
- 频繁的UI更新可能影响性能
- 考虑降低更新频率或使用批量更新
- 消息显示系统开销较大

### 实验性特性
- API可能在未来版本中变更
- 某些功能可能不稳定
- 建议在生产环境中谨慎使用

## 相关组件
- [`TurnBasedComponent`](./TurnBasedComponent.md) - 回合制基类
- [`TurnBasedCoordinator`](../../Managers/TurnBasedCoordinator.md) - 回合制协调器
- [`TurnBasedEntity`](../../Entities/TurnBasedEntity.md) - 回合制实体 