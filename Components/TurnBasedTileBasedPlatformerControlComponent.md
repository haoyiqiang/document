# TurnBasedTileBasedPlatformerControlComponent API 文档

## 概述
`TurnBasedTileBasedPlatformerControlComponent` 是专为回合制游戏设计的瓦片平台控制组件，提供平台游戏风格的伪物理效果。该组件只处理水平移动输入，垂直移动通过跳跃和重力系统实现。

## 继承关系
```
Component
└── TurnBasedComponent
    └── TurnBasedTileBasedControlComponent
        └── TurnBasedTileBasedPlatformerControlComponent
```

## 主要功能
- **水平移动**: 只处理左右移动输入
- **垂直限制**: 忽略直接的垂直输入
- **平台物理**: 配合重力组件实现平台游戏效果
- **跳跃集成**: 支持跳跃系统控制垂直移动

## 必需组件
- `TurnBasedEntity` - 回合制实体
- `TileBasedPositionComponent` - 瓦片位置组件
- `TurnBasedTileBasedGravityComponent` - 重力组件（推荐）

## 使用示例

### 基本设置
```gdscript
# 在场景中添加平台控制组件
@onready var platformer_control = $TurnBasedTileBasedPlatformerControlComponent

func _ready():
    # 确保有必需的组件
    assert(has_component(TileBasedPositionComponent))
    assert(has_component(TurnBasedEntity))
    
    # 配置组件
    platformer_control.isEnabled = true
```

### 与重力组件配合
```gdscript
# 设置完整的平台游戏系统
func setup_platformer_entity():
    # 添加重力组件
    var gravity_component = TurnBasedTileBasedGravityComponent.new()
    add_child(gravity_component)
    
    # 配置平台控制
    var platformer_control = TurnBasedTileBasedPlatformerControlComponent.new()
    add_child(platformer_control)
    
    print("平台游戏实体设置完成")
```

## 扩展示例

### 自定义移动限制
```gdscript
# 创建自定义移动限制
class_name CustomPlatformerControl
extends TurnBasedTileBasedPlatformerControlComponent

@export var can_move_while_falling: bool = false
@export var movement_speed: int = 1

func _input(event: InputEvent) -> void:
    if not isEnabled or not event.is_action_type():
        return
    
    # 检查是否可以在下落时移动
    if not can_move_while_falling:
        var gravity_component = coComponents.TurnBasedTileBasedGravityComponent
        if gravity_component and gravity_component.is_falling:
            return
    
    # 处理水平移动
    if event.is_action_pressed(GlobalInput.Actions.moveLeft) \
    or event.is_action_pressed(GlobalInput.Actions.moveRight):
        
        var input_vector = Input.get_vector(
            GlobalInput.Actions.moveLeft, 
            GlobalInput.Actions.moveRight, 
            GlobalInput.Actions.moveUp, 
            GlobalInput.Actions.moveDown
        )
        
        # 限制移动速度
        input_vector.x = clamp(input_vector.x, -movement_speed, movement_speed)
        input_vector.y = 0  # 忽略垂直输入
        
        self.recentInputVector = input_vector
        validateMove()
```

## 技术细节

### 输入处理
- 只处理水平移动输入（左右）
- 自动将垂直输入设为0
- 垂直移动通过跳跃和重力系统实现

### 平台物理
- 与重力组件配合实现下落效果
- 支持跳跃系统控制垂直移动
- 提供经典平台游戏的移动感受

## 注意事项

1. **实验性功能**: 标记为实验性功能，API可能会发生变化
2. **依赖要求**: 需要配合重力组件使用
3. **输入限制**: 只处理水平移动，垂直移动需要其他组件
4. **回合制约束**: 移动受回合制系统约束

## 相关组件
- `TurnBasedTileBasedControlComponent` - 父类组件
- `TurnBasedTileBasedGravityComponent` - 重力组件
- `TileBasedPositionComponent` - 瓦片位置组件
- `TurnBasedComponent` - 回合制基类

## 版本信息
- **状态**: 实验性功能
- **兼容性**: 需要完整的回合制系统支持
- **依赖**: 需要重力组件实现完整的平台游戏效果 