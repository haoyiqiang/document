# PlayerEntity

## 概述
`PlayerEntity` 是玩家实体的基础实现，继承自 `Entity` 类。提供了玩家角色的基础功能，包括输入处理、移动控制、生命值管理等。

**继承关系：** `PlayerEntity` → `Entity` → `Node2D` → `Node`

## 主要特性
- 🎮 玩家输入处理
- 🏃 基础移动控制
- ❤️ 生命值管理
- 🎯 玩家特定行为
- 🔧 可扩展的组件系统

## 导出属性

### `playerId: int = 0`
玩家ID。

**类型：** `int`  
**默认值：** `0`  
**说明：** 用于区分多个玩家

### `playerName: String = "Player"`
玩家名称。

**类型：** `String`  
**默认值：** `"Player"`  
**说明：** 玩家的显示名称

## 核心方法

### `_ready() -> void`
玩家实体准备就绪时调用。

**功能：**
1. 调用父类的 `_ready()` 方法
2. 初始化玩家特定功能
3. 设置玩家标识

### `setupPlayerComponents() -> void`
设置玩家组件。

**功能：**
1. 添加玩家必需的组件
2. 配置组件参数
3. 建立组件间的依赖关系

### `handlePlayerInput() -> void`
处理玩家输入。

**功能：**
1. 读取输入设备
2. 处理移动输入
3. 处理动作输入
4. 更新玩家状态

## 使用示例

### 基础玩家创建
```gdscript
# 创建玩家实体
var player = PlayerEntity.new()
player.playerId = 1
player.playerName = "Hero"

# 添加到场景
add_child(player)
```

### 玩家配置
```gdscript
# 配置玩家属性
var player = PlayerEntity.new()
player.playerId = 0
player.playerName = "Player1"

# 设置位置
player.global_position = Vector2(100, 100)
```

## 注意事项

1. **组件依赖** - 确保玩家组件按正确顺序添加
2. **输入处理** - 玩家实体需要输入组件来处理用户输入
3. **生命周期** - 正确处理玩家的创建和销毁
4. **性能优化** - 玩家实体通常需要较高的更新频率

## 相关组件

- **InputComponent** - 输入处理组件
- **MovementComponent** - 移动控制组件
- **HealthComponent** - 生命值管理组件
- **CameraComponent** - 摄像机跟随组件 