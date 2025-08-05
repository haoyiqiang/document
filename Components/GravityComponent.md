# GravityComponent

## 概述
`GravityComponent` 是一个重力物理组件，每帧向父实体的`CharacterBody2D`应用重力效果。提供简单而高效的重力模拟，适用于不需要完整平台物理系统的场景。

**继承关系：** `GravityComponent` → `CharacterBodyDependentComponentBase` → `Component` → `Node`

## 主要特性
- 🌍 基于项目设置的重力模拟
- ⚖️ 可调节重力倍数
- 🚀 轻量级实现
- 🔧 简单配置界面
- ⚠️ 冲突检测和警告

## 依赖组件
- **CharacterBodyComponent** (必需) - 角色身体控制

## 导出属性

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`

### `gravityScale: float = 1.0`
重力倍数，相对于项目设置中的重力值。

**类型：** `float`  
**范围：** `-10` 到 `10`  
**步进：** `0.05`  
**默认值：** `1.0`  
**说明：**
- `1.0` = 正常重力
- `0.5` = 一半重力
- `2.0` = 双倍重力
- `-1.0` = 反向重力（向上）

## 状态属性

### `gravity: float`
从项目设置获取的重力值。

**类型：** `float`  
**来源：** `Settings.gravity`  
**只读：** 是

## 核心方法

### `_physics_process(delta: float) -> void`
物理帧更新，应用重力效果。

**参数：** `delta` - 物理帧时间间隔  
**逻辑：**
1. 检查组件是否启用
2. 检查角色是否在地面
3. 如果不在地面，应用重力到Y轴速度
4. 设置移动标志

## 使用示例

### 基本重力设置
```gdscript
# 设置正常重力
func setupNormalGravity():
    var gravityComponent = $GravityComponent
    
    gravityComponent.gravityScale = 1.0
    gravityComponent.isEnabled = true

# 设置月球重力（较弱）
func setupMoonGravity():
    var gravityComponent = $GravityComponent
    
    gravityComponent.gravityScale = 0.166  # 约为地球重力的1/6
    gravityComponent.isEnabled = true
```

### 特殊重力效果
```gdscript
# 反重力效果
func enableAntiGravity():
    var gravityComponent = $GravityComponent
    
    gravityComponent.gravityScale = -1.0
    print("反重力已启用!")

# 零重力（太空环境）
func enableZeroGravity():
    var gravityComponent = $GravityComponent
    
    gravityComponent.gravityScale = 0.0
    # 或者直接禁用组件
    # gravityComponent.isEnabled = false

# 强重力（类似木星）
func enableHeavyGravity():
    var gravityComponent = $GravityComponent
    
    gravityComponent.gravityScale = 2.36  # 约为地球重力的2.36倍
```

### 动态重力调整
```gdscript
# 根据游戏状态调整重力
func adjustGravityByEnvironment(environmentType: String):
    var gravityComponent = $GravityComponent
    
    match environmentType:
        "earth":
            gravityComponent.gravityScale = 1.0
        "moon":
            gravityComponent.gravityScale = 0.166
        "mars":
            gravityComponent.gravityScale = 0.377
        "jupiter":
            gravityComponent.gravityScale = 2.36
        "space":
            gravityComponent.gravityScale = 0.0
        "underwater":
            gravityComponent.gravityScale = 0.3  # 浮力效果
        "storm":
            gravityComponent.gravityScale = 1.5  # 下压气流

# 道具影响重力
func applyGravityBoots(duration: float):
    var gravityComponent = $GravityComponent
    var originalScale = gravityComponent.gravityScale
    
    # 重力靴减少重力影响
    gravityComponent.gravityScale = 0.5
    
    # 定时恢复
    await get_tree().create_timer(duration).timeout
    gravityComponent.gravityScale = originalScale
```

## 设计模式

### 单一职责模式
- **专注重力：** 只处理重力物理效果
- **简单接口：** 最少的配置参数
- **职责清晰：** 不包含其他物理模拟

### 组合模式
- **物理集成：** 与CharacterBodyComponent协作
- **设置依赖：** 使用项目全局重力设置
- **组件协作：** 设置移动标志通知其他组件

### 策略模式
- **重力倍数：** 通过gravityScale实现不同重力策略
- **启用控制：** 通过isEnabled实现开关策略
- **灵活配置：** 支持正负重力倍数

## 技术细节

### 重力计算
重力应用公式：
```gdscript
body.velocity.y += (gravity * gravityScale) * delta
```

### 地面检测
只有在不在地面时才应用重力：
```gdscript
if not characterBodyComponent.isOnFloor:
    # 应用重力
```

### 运动模式设置
自动设置CharacterBody2D为GROUNDED模式：
```gdscript
parentEntity.body.motion_mode = CharacterBody2D.MOTION_MODE_GROUNDED
```

### 冲突检测
检测与PlatformerPhysicsComponent的冲突：
```gdscript
if coComponents.get("PlatformerPhysicsComponent"):
    printWarning("Both components process gravity; Remove one!")
```

## 注意事项

1. **组件冲突：** 不要与PlatformerPhysicsComponent同时使用
2. **组件顺序：** 必须在控制组件之前
3. **重力方向：** 默认向下（正Y方向）
4. **性能考虑：** 轻量级实现，适合大量实体
5. **地面检测：** 依赖CharacterBodyComponent的地面检测

## 相关组件
- `CharacterBodyComponent` - 角色身体控制，提供地面检测
- `PlatformerPhysicsComponent` - 完整平台物理系统（不可同时使用）
- `OverheadPhysicsComponent` - 俯视角物理系统
- `JumpComponent` - 跳跃组件，与重力配合实现跳跃物理