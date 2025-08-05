# JumpControlComponent 跳跃控制组件

## 📖 概述

**JumpControlComponent** 是一个专门处理跳跃功能的控制组件。它提供了完整的跳跃系统，包括基础跳跃、多段跳跃、墙跳、土狼跳跃（Coyote Jump）和短跳等高级功能，为2D平台游戏提供了丰富的跳跃机制。

继承自：`CharacterBodyDependentComponentBase`

## ⚙️ 功能特性

### 核心功能
- **基础跳跃**：响应跳跃输入，施加向上速度
- **多段跳跃**：支持二段跳及多段跳跃
- **墙跳**：允许角色从墙壁上弹跳
- **土狼跳跃**：离开平台后短时间内仍可跳跃
- **短跳**：提前释放按键可缩短跳跃高度
- **重力方向支持**：尊重 `CharacterBody2D.up_direction` 实现倒置重力

### 高级特性
- **跳跃状态管理**：跟踪当前跳跃次数和状态
- **计时器系统**：精确控制土狼跳跃和墙跳的时间窗口
- **输入缓冲**：缓存输入状态，确保响应性
- **调试支持**：详细的调试信息显示

## 🎛️ 导出属性

### 参数配置
```gdscript
@export var parameters: PlatformerJumpParameters
```
跳跃参数配置，包含所有跳跃相关的设置。

### 启用控制
```gdscript
@export var isEnabled: bool = true
```
是否启用跳跃控制功能。

## 🔧 子节点要求

### 定时器节点
- **CoyoteJumpTimer**: 土狼跳跃计时器
- **WallJumpTimer**: 墙跳计时器

## 📋 依赖关系

### 必需组件
- **CharacterBodyComponent**: 角色体组件，提供物理体访问
- **PlatformerPhysicsComponent**: 平台物理组件，处理重力和空中摩擦

### 组件顺序
必须在 PlatformerPhysicsComponent 之前执行。

## 🎮 使用方法

### 基本设置

1. **添加组件**：将 JumpControlComponent 作为子节点添加到实体
2. **配置参数**：设置 PlatformerJumpParameters 资源
3. **添加定时器**：确保 CoyoteJumpTimer 和 WallJumpTimer 子节点存在
4. **设置输入**：确保 GlobalInput.Actions.jump 动作已配置

### 参数配置示例

```gdscript
# 在 PlatformerJumpParameters 资源中配置
@export_range(0, 10, 1) var maxNumberOfJumps: int = 2                        # 最大跳跃次数
@export_range(-1000, 1000, 5) var jumpVelocity1stJump: float = 350           # 第一次跳跃速度
@export_range(-1000, 1000, 5) var jumpVelocity1stJumpShort: float = 150      # 短跳速度
@export_range(-1000, 1000, 5) var jumpVelocity2ndJump: float = 300           # 二段跳速度
@export var allowCoyoteJump: bool = true                                     # 允许土狼跳跃
@export_range(0, 10, 0.05) var coyoteJumpTimer: float = 0.15                 # 土狼跳跃时间窗口
@export var allowWallJump: bool = true                                       # 允许墙跳
@export_range(-1000, 1000, 10) var wallJumpVelocity: float = 300             # 墙跳Y轴速度
@export_range(10, 1000, 10) var wallJumpVelocityX: float = 150               # 墙跳X轴速度
@export_range(0, 10, 0.05) var wallJumpTimer: float = 0.1                    # 墙跳时间窗口
@export var decreaseJumpCountOnWallJump: bool = true                         # 墙跳是否消耗跳跃次数
```

**重要提示**：
- 跳跃速度值为正值，因为会自动乘以 `CharacterBody2D.up_direction`
- 对于倒置重力，修改 `CharacterBodyComponent` 的 `up_direction` 即可
- 短跳功能仅适用于第一次跳跃，不影响多段跳或墙跳

### 实体场景结构

```
PlayerEntity (CharacterBody2D)
├── CharacterBodyComponent
├── PlatformerPhysicsComponent  
├── JumpControlComponent
│   ├── CoyoteJumpTimer (Timer)
│   └── WallJumpTimer (Timer)
└── InputComponent
```

## 🔄 工作流程

### 输入处理
1. 检测跳跃输入事件
2. 缓存输入状态（按下、刚按下、刚释放）
3. 处理墙跳逻辑
4. 处理普通跳跃逻辑
5. 排队 move_and_slide 调用

### 移动后更新
1. 重置状态（在地面时重置跳跃次数）
2. 更新土狼跳跃状态
3. 更新墙跳状态
4. 清理输入缓存

### 跳跃类型判断

#### 普通跳跃
- 在地面或土狼跳跃时间内：执行第一次跳跃
- 在空中且未达到最大跳跃次数：执行多段跳跃

#### 短跳
- 在第一次跳跃中提前释放按键
- 速度大于短跳速度时，将速度限制为短跳速度

#### 墙跳
- 在墙上或刚离开墙壁的时间窗口内
- 施加水平和垂直速度
- 可选择是否消耗跳跃次数

#### 土狼跳跃
- 刚离开地面且正在下落时启动计时器
- 在计时器有效期内允许跳跃

## ⚡ 信号

该组件主要通过监听其他组件的信号工作：

### 监听的信号
- **CharacterBodyComponent.didMove**: 移动完成后的状态更新

## 🎯 使用场景

### 适用游戏类型
- **2D平台游戏**：提供丰富的跳跃机制
- **动作游戏**：精确的跳跃控制
- **冒险游戏**：探索导向的移动
- **解谜游戏**：基于跳跃的机制设计

### 典型应用
- 马里奥风格的平台跳跃
- 银河恶魔城类游戏的移动
- 超级肉肉男孩式的精确平台
- 空洞骑士风格的墙跳系统

## 🔧 自定义扩展

### 添加新跳跃类型
```gdscript
# 继承 JumpControlComponent 添加新功能
extends JumpControlComponent

func processCustomJump():
    # 自定义跳跃逻辑
    pass
```

### 特殊重力处理
```gdscript
# 支持动态重力方向变化
func updateGravityDirection(newUpDirection: Vector2):
    body.up_direction = newUpDirection
    # 跳跃速度会自动适应新的重力方向
```

## 🐛 调试功能

### 调试信息显示
当 `debugMode` 启用时，组件会显示：
- 当前状态
- 跳跃次数
- 计时器剩余时间
- 输入状态
- 速度信息

### 调试输出示例
```
—PlayerEntity.JumpControlComponent
state: jump
wallTimer: 0.0
coyoteTimer: 0.0
jumpInput: false
jumps: 1
```

## ⚠️ 注意事项

1. **组件顺序**：必须在 PlatformerPhysicsComponent 之前处理
2. **定时器配置**：确保子节点定时器正确设置
3. **参数调整**：跳跃参数需要根据游戏感觉精细调整
4. **重力兼容**：支持自定义重力方向，但需要正确配置
5. **输入映射**：确保跳跃动作在输入映射中正确定义

## 📚 相关组件

- **[CharacterBodyComponent](../Physics/CharacterBodyComponent.md)**: 提供物理体访问
- **[PlatformerPhysicsComponent](../Physics/PlatformerPhysicsComponent.md)**: 处理重力和物理
- **[ClimbComponent](./ClimbComponent.md)**: 攀爬功能的替代选择
- **[PlatformerControlComponent](./PlatformerControlComponent.md)**: 水平移动控制
- **[InputComponent](./InputComponent.md)**: 输入状态管理

## 📖 参考资源

- [Heart Platformer Tutorial](https://youtu.be/M8-JVjtJlIQ) - uHeartbeast 教程
- [Godot CharacterBody2D 文档](https://docs.godotengine.org/en/stable/classes/class_characterbody2d.html)
- [平台游戏物理最佳实践](https://docs.godotengine.org/en/stable/tutorials/physics/kinematic_character_2d.html)

---

*该组件为 Comedot 框架的一部分，遵循组件化架构设计原则。* 