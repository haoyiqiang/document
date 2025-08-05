# TileBasedControlComponent API

## 概述
`TileBasedControlComponent` 为`TileBasedPositionComponent`提供玩家输入控制。它将方向键输入转换为瓦片网格的移动指令，支持连续移动和单步移动模式。

**继承链**：`Component` → `TileBasedControlComponent`

## 依赖要求
- **TileBasedPositionComponent**：必需的瓦片位置管理组件

## 主要特性
- 🎮 **方向输入**：响应上下左右方向键
- 🔄 **移动模式**：支持连续移动和单步移动
- ↗️ **对角线控制**：可选择是否允许对角线移动
- ⏱️ **移动延迟**：内置Timer控制移动频率
- 🎯 **精确控制**：基于瓦片网格的精确位置控制

## 导出属性

### 移动配置
- **shouldAllowDiagonals** (`bool = false`)：是否允许对角线移动
- **shouldMoveContinuously** (`bool = true`)：是否连续移动
  - true：长按方向键持续移动
  - false：每次按键只移动一步
- **isEnabled** (`bool = true`)：是否启用输入控制

## 状态属性
- **recentInputVector** (`Vector2i`)：最近的输入方向向量

## 子节点
- **Timer**：控制移动间隔的定时器

## 主要方法
- **move**() → `void`：执行移动操作，不依赖isEnabled状态

## 使用示例

### 基础瓦片控制
```gdscript
# 设置基本的瓦片控制
@export var tile_control: TileBasedControlComponent

func _ready():
    tile_control.shouldAllowDiagonals = false
    tile_control.shouldMoveContinuously = true
```

### 回合制移动
```gdscript
# 回合制游戏的单步移动
@export var tile_control: TileBasedControlComponent

func setup_turn_based_movement():
    tile_control.shouldMoveContinuously = false
    tile_control.shouldAllowDiagonals = true
```

### 条件移动控制
```gdscript
# 基于游戏状态的移动控制
@export var tile_control: TileBasedControlComponent

func update_movement_permissions():
    # 战斗中禁用移动
    tile_control.isEnabled = not GameState.is_in_combat
    
    # 特殊技能允许对角线移动
    tile_control.shouldAllowDiagonals = player_stats.has_diagonal_movement
```

## 技术细节

### 输入处理
- 使用GlobalInput.Actions的标准移动动作
- 支持对角线移动的智能向量转换
- 自动处理输入事件的消费

### 移动限制
- Timer防止过快移动
- 检查TileBasedPositionComponent的可用性
- 输入向量的标准化处理

## 注意事项

### 组件依赖
- 必须与TileBasedPositionComponent配合使用
- 确保正确的组件初始化顺序

### 性能考虑
- 连续移动模式在_physics_process中运行
- 单步移动模式仅在输入时处理

## 相关组件
- [`TileBasedPositionComponent`](../Movement/TileBasedPositionComponent.md) - 瓦片位置管理
- [`InputComponent`](../Control/InputComponent.md) - 通用输入控制