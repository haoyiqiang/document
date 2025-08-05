# JumpComponent

## 概述
`JumpComponent` 是一个高级跳跃控制组件，支持多段跳跃、土狼跳、墙跳等平台游戏常见的跳跃机制。通过监听输入事件应用垂直速度，并考虑重力方向以支持反重力情况。

**继承关系：** `JumpComponent` → `CharacterBodyDependentComponentBase` → `Component` → `Node`

## 主要特性
- 🏃 多段跳跃系统
- 🐺 土狼跳（走出平台后的宽容跳跃）
- 🧗 墙跳机制
- ⚡ 短跳控制（提前释放按键）
- 🔄 支持反重力
- ⚙️ 高度可配置的参数

## 依赖组件
- **CharacterBodyComponent** (必需) - 角色身体控制
- **PlatformerPhysicsComponent** (必需) - 平台物理系统
- **InputComponent** (必需) - 输入处理

## 导出属性

### `parameters: PlatformerJumpParameters`
跳跃参数配置对象。

**类型：** `PlatformerJumpParameters`  
**包含：**
- 跳跃速度设置
- 最大跳跃次数
- 土狼跳和墙跳设置
- 定时器参数

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**特性：** 自动控制输入处理状态

## 状态枚举

### `State` 枚举
```gdscript
enum State { idle, jump }
```

当前跳跃状态：
- `idle` - 空闲状态
- `jump` - 跳跃状态

## 状态属性

### 输入状态
- `jumpInput: bool` - 当前跳跃按键状态
- `jumpInputJustPressed: bool` - 跳跃按键刚按下
- `jumpInputJustReleased: bool` - 跳跃按键刚释放

### 跳跃状态
- `currentNumberOfJumps: int` - 当前跳跃次数
- `didWallJump: bool` - 是否刚执行了墙跳
- `currentState: State` - 当前状态

### 定时器节点
- `coyoteJumpTimer: Timer` - 土狼跳计时器
- `wallJumpTimer: Timer` - 墙跳计时器

## 核心方法

### `oninputComponent_didProcessInput(event: InputEvent) -> void`
处理输入事件的主要方法。

**参数：** `event` - 输入事件  
**功能：**
- 缓存输入状态
- 调用墙跳和普通跳跃处理
- 设置移动标志

### `processJump() -> void`
处理普通跳跃和多段跳跃逻辑。

**功能：**
- 检查跳跃条件
- 处理土狼跳
- 执行短跳控制
- 应用跳跃速度

### `processWallJump() -> void`
处理墙跳逻辑。

**功能：**
- 检查墙跳条件
- 应用墙跳速度
- 管理跳跃次数

### `resetState() -> void`
重置跳跃状态。

**触发条件：** 角色着地时  
**功能：**
- 重置跳跃次数
- 停止土狼跳计时器

### `updateCoyoteJumpState() -> void`
更新土狼跳状态。

**功能：** 在角色走下平台时启动宽容跳跃计时器

### `updateWallJumpState() -> void`
更新墙跳状态。

**功能：** 在角色离开墙壁时启动墙跳计时器

## 跳跃参数详解

### 基础跳跃
- `jumpVelocity1stJump` - 首次跳跃速度
- `jumpVelocity1stJumpShort` - 短跳速度
- `jumpVelocity2ndJump` - 二段跳速度

### 多段跳跃
- `maxNumberOfJumps` - 最大跳跃次数
- `allowCoyoteJump` - 是否允许土狼跳
- `coyoteJumpTimer` - 土狼跳时间窗口

### 墙跳设置
- `allowWallJump` - 是否允许墙跳
- `wallJumpVelocity` - 墙跳垂直速度
- `wallJumpVelocityX` - 墙跳水平速度
- `wallJumpTimer` - 墙跳时间窗口
- `decreaseJumpCountOnWallJump` - 墙跳是否消耗跳跃次数

## 使用示例

### 基本跳跃设置
```gdscript
# 获取跳跃组件
var jumpComponent = $JumpComponent

# 配置基础跳跃
jumpComponent.parameters.jumpVelocity1stJump = -300
jumpComponent.parameters.jumpVelocity1stJumpShort = -150
jumpComponent.parameters.maxNumberOfJumps = 1

# 启用组件
jumpComponent.isEnabled = true
```

### 多段跳跃配置
```gdscript
# 设置双跳
func setupDoubleJump():
    var jumpComponent = $JumpComponent
    
    jumpComponent.parameters.maxNumberOfJumps = 2
    jumpComponent.parameters.jumpVelocity1stJump = -280
    jumpComponent.parameters.jumpVelocity2ndJump = -250

# 设置三段跳
func setupTripleJump():
    var jumpComponent = $JumpComponent
    
    jumpComponent.parameters.maxNumberOfJumps = 3
    # 递减的跳跃力度
    jumpComponent.parameters.jumpVelocity1stJump = -300
    jumpComponent.parameters.jumpVelocity2ndJump = -250
```

### 土狼跳设置
```gdscript
# 启用土狼跳以改善手感
func setupCoyoteJump():
    var jumpComponent = $JumpComponent
    
    jumpComponent.parameters.allowCoyoteJump = true
    jumpComponent.parameters.coyoteJumpTimer = 0.1  # 100ms宽容时间
    
    print("土狼跳已启用，宽容时间: 100ms")
```

### 墙跳配置
```gdscript
# 设置墙跳机制
func setupWallJump():
    var jumpComponent = $JumpComponent
    
    jumpComponent.parameters.allowWallJump = true
    jumpComponent.parameters.wallJumpVelocity = -280
    jumpComponent.parameters.wallJumpVelocityX = 200
    jumpComponent.parameters.wallJumpTimer = 0.15
    jumpComponent.parameters.decreaseJumpCountOnWallJump = false  # 墙跳不消耗跳跃次数
```

### 反重力支持
```gdscript
# 设置反重力跳跃
func setupInvertedGravity():
    var characterBody = $CharacterBodyComponent
    var jumpComponent = $JumpComponent
    
    # 反转重力方向
    characterBody.body.up_direction = Vector2.DOWN
    
    # 跳跃组件会自动适应重力方向
    print("反重力模式已启用")
```

### 动态跳跃能力
```gdscript
# 根据游戏状态调整跳跃能力
func updateJumpAbilities(playerLevel: int):
    var jumpComponent = $JumpComponent
    
    match playerLevel:
        1:
            # 基础跳跃
            jumpComponent.parameters.maxNumberOfJumps = 1
            jumpComponent.parameters.allowWallJump = false
        2:
            # 解锁双跳
            jumpComponent.parameters.maxNumberOfJumps = 2
        3:
            # 解锁墙跳
            jumpComponent.parameters.allowWallJump = true
        4:
            # 三段跳
            jumpComponent.parameters.maxNumberOfJumps = 3

# 临时增强跳跃（如道具效果）
func boostJumping(duration: float):
    var jumpComponent = $JumpComponent
    var originalJumps = jumpComponent.parameters.maxNumberOfJumps
    
    # 临时增加跳跃次数
    jumpComponent.parameters.maxNumberOfJumps += 1
    
    # 定时恢复
    await get_tree().create_timer(duration).timeout
    jumpComponent.parameters.maxNumberOfJumps = originalJumps
```

### 特殊跳跃效果
```gdscript
# 监听跳跃状态变化
func _ready():
    var jumpComponent = $JumpComponent
    
    # 连接状态变化（需要自定义信号）
    jumpComponent.connect("state_changed", _on_jump_state_changed)

func _on_jump_state_changed(newState):
    match newState:
        JumpComponent.State.jump:
            # 播放跳跃音效
            $SFX/JumpSound.play()
            # 显示跳跃特效
            $VFX/JumpEffect.emit()

# 检测跳跃类型
func detectJumpType():
    var jumpComponent = $JumpComponent
    
    if jumpComponent.didWallJump:
        print("执行了墙跳")
        $SFX/WallJumpSound.play()
    elif jumpComponent.currentNumberOfJumps > 1:
        print("执行了多段跳")
        $SFX/DoubleJumpSound.play()
```

### 平台游戏优化
```gdscript
# 优化平台游戏手感
func optimizeForPlatformer():
    var jumpComponent = $JumpComponent
    
    # 较长的土狼跳时间
    jumpComponent.parameters.allowCoyoteJump = true
    jumpComponent.parameters.coyoteJumpTimer = 0.15
    
    # 舒适的墙跳设置
    jumpComponent.parameters.allowWallJump = true
    jumpComponent.parameters.wallJumpTimer = 0.2
    
    # 短跳控制以实现精确跳跃
    jumpComponent.parameters.jumpVelocity1stJump = -320
    jumpComponent.parameters.jumpVelocity1stJumpShort = -160

# Metroidvania样式设置
func setupMetroidvania():
    var jumpComponent = $JumpComponent
    
    # 强调垂直移动
    jumpComponent.parameters.jumpVelocity1stJump = -350
    jumpComponent.parameters.maxNumberOfJumps = 1  # 开始只有单跳
    
    # 墙跳是关键移动方式
    jumpComponent.parameters.allowWallJump = true
    jumpComponent.parameters.wallJumpVelocityX = 180
    jumpComponent.parameters.decreaseJumpCountOnWallJump = false
```

## 设计模式

### 状态管理
- **计数系统：** 跟踪当前跳跃次数
- **计时器机制：** 土狼跳和墙跳的宽容时间
- **状态缓存：** 缓存输入状态避免重复查询

### 物理集成
- **速度控制：** 直接修改CharacterBody2D的velocity
- **方向感知：** 尊重up_direction以支持反重力
- **物理依赖：** 与PlatformerPhysicsComponent协作

### 输入处理
- **事件驱动：** 响应InputComponent的输入事件
- **AI友好：** 支持合成InputEvent用于AI控制
- **状态清理：** 每帧清理输入状态

## 技术细节

### 短跳实现
短跳通过检测按键提前释放实现：
- 仅在第一次跳跃时生效
- 比较当前速度与短跳速度
- 考虑重力方向避免下降时误触发

### 土狼跳实现
土狼跳提供离开平台后的跳跃宽容：
- 检测从地面到空中的转换
- 启动计时器提供宽容窗口
- 只在下降时激活

### 墙跳实现
墙跳允许从墙壁反弹：
- 检测墙壁法线方向
- 同时应用水平和垂直速度
- 可选择是否消耗跳跃次数

## 注意事项

1. **组件顺序：** 必须在PlatformerPhysicsComponent之前
2. **重力方向：** 自动适应CharacterBody2D的up_direction
3. **输入重复：** 防止键盘输入重复（待优化）
4. **物理同步：** 与move_and_slide()的时序配合
5. **调试输出：** 提供详细的调试信息

## 相关组件
- `CharacterBodyComponent` - 角色身体控制
- `PlatformerPhysicsComponent` - 平台物理系统，处理重力和摩擦
- `InputComponent` - 输入处理系统
- `ClimbComponent` - 攀爬系统，用于梯子/绳索
- `PlatformerControlComponent` - 平台游戏控制 