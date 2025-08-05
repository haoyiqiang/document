# VelocityClampComponent API

> **继承关系**: Component > CharacterBodyDependentComponentBase > VelocityClampComponent  
> **物理类型**: 速度限制

为实体的CharacterBody2D.velocity设置最大和最小限制的组件，用于限制多个组件影响移动时的速度范围，防止速度过快或异常移动。

## ✨ 主要特性

- 🎯 X/Y轴独立速度限制
- ⚡ 最大/最小速度控制  
- 🔄 可启用/禁用功能
- 🚀 防止"火箭效应"
- 🏃 支持最小移动速度

## 📊 导出属性

### 速度限制设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `maximumVelocityX` | `float` | `100.0` | X轴最大速度 (0-5000) |
| `maximumVelocityY` | `float` | `100.0` | Y轴最大速度 (0-5000) |
| `minimumVelocityX` | `float` | `0.0` | X轴最小速度，>0时强制向右移动 |
| `minimumVelocityY` | `float` | `0.0` | Y轴最小速度，>0时强制向下移动 |
| `isEnabled` | `bool` | `true` | 是否启用速度限制 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `CharacterBodyComponent` | **基类依赖** | 提供CharacterBody2D访问 |

### 处理顺序
- **之前**: CharacterBodyComponent
- **之后**: 控制组件(PlatformerControlComponent等)

## 🎯 使用示例

### 基础速度限制
```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── CharacterBodyComponent
#     ├── PlatformerControlComponent
#     └── VelocityClampComponent (max: 200, min: 0)
```

### 防止火箭效应
```gdscript
# 适用于有GunComponent的实体
# 防止快速射击导致的反冲力累积
@export var maximumVelocityX: float = 150.0
@export var maximumVelocityY: float = 100.0
```

### 动态速度控制
```gdscript
extends VelocityClampComponent

@export var normalMaxSpeed: float = 100.0
@export var sprintMaxSpeed: float = 200.0

func _physics_process(delta: float):
    # 动态调整最大速度
    if Input.is_action_pressed("sprint"):
        maximumVelocityX = sprintMaxSpeed
        maximumVelocityY = sprintMaxSpeed
    else:
        maximumVelocityX = normalMaxSpeed
        maximumVelocityY = normalMaxSpeed
    
    super._physics_process(delta)
```

## 🔧 技术细节

### 速度限制算法
```gdscript
func _physics_process(_delta: float) -> void:
    var velocity = body.velocity
    var signX = signf(velocity.x)
    var signY = signf(velocity.y)
    
    # 最大速度限制
    if maximumVelocityX > 0 and absf(velocity.x) > maximumVelocityX:
        body.velocity.x = maximumVelocityX * signX
    
    # 最小速度限制  
    if minimumVelocityX > 0 and absf(velocity.x) < minimumVelocityX:
        body.velocity.x = minimumVelocityX * signX
```

## ⚠️ 注意事项

1. **最小速度问题**: 当前最小速度只能为正值，会导致持续移动
2. **对角线移动**: 需要修正对角线移动的速度计算
3. **处理顺序**: 必须在CharacterBodyComponent之前，控制组件之后
4. **性能优化**: 使用isEnabled setter减少不必要的处理

## 🔗 相关组件

- [CharacterBodyDependentComponentBase](CharacterBodyDependentComponentBase.md) - 基类组件
- [CharacterBodyComponent](CharacterBodyComponent.md) - 角色体组件
- [GunComponent](./GunComponent.md) - 射击组件(反冲力)
- [KnockbackOnHitComponent](./KnockbackOnHitComponent.md) - 击退组件

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 