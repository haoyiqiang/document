# DamageVisualComponent API

## 概述
`DamageVisualComponent` 是一个专门用于显示伤害视觉效果的组件。当实体受到伤害时，它会提供红色着色、闪烁动画和文字气泡等视觉反馈，增强游戏的视觉体验。

**继承链**：`Component` → `DamageVisualComponent`

**实验性功能**：此组件为实验性质，API可能在后续版本中变更。

## 依赖要求
- **DamageReceivingComponent**：监听伤害接收事件
- **HealthComponent**：获取生命值信息（支持ShieldedHealthComponent）

## 主要特性
- 🎨 **伤害着色**：根据生命值动态调整红色色调
- ⚡ **闪烁动画**：受伤时播放闪烁效果
- 💬 **文字气泡**：显示伤害数值或剩余生命值
- 🔄 **实时更新**：自动响应伤害和治疗事件
- 🛡️ **护盾支持**：兼容ShieldedHealthComponent

## 导出属性

### 视觉效果
- **shouldTint** (`bool = false`)：是否启用红色着色效果
  - 当启用时，随着生命值降低增加红色强度
  - 动态调整实体的`modulate`属性
- **shouldEmitBubble** (`bool = true`)：是否显示伤害文字气泡
- **shouldShowRemainingHealth** (`bool = false`)：气泡显示剩余生命值而非伤害差值

## 信号响应

### DamageReceivingComponent事件
- **didReceiveDamage**：当受到伤害时触发视觉效果

## 主要方法

### 视觉效果控制
- **updateTint()** → `void`：更新红色着色效果
- **animate(damageAmount: int)** → `void`：播放伤害动画
- **emitBubble(damageAmount: int)** → `void`：显示文字气泡

## 使用示例

### 基础伤害视觉
```gdscript
# 设置基本的伤害视觉效果
@export var damage_visual: DamageVisualComponent

func _ready():
    damage_visual.shouldTint = true
    damage_visual.shouldEmitBubble = true
    damage_visual.shouldShowRemainingHealth = false
```

### 高级伤害显示
```gdscript
# 创建丰富的伤害视觉体验
@export var damage_visual: DamageVisualComponent
@export var health_component: HealthComponent

func setup_damage_effects():
    # 启用着色和气泡
    damage_visual.shouldTint = true
    damage_visual.shouldEmitBubble = true
    
    # 对于Boss显示剩余血量
    if entity.has_method("is_boss") and entity.is_boss():
        damage_visual.shouldShowRemainingHealth = true
```

### 条件视觉效果
```gdscript
# 基于游戏状态的视觉效果
@export var damage_visual: DamageVisualComponent

func update_visual_settings():
    # 困难模式下显示更明显的伤害效果
    if GameState.difficulty == GameState.Difficulty.HARD:
        damage_visual.shouldTint = true
    else:
        damage_visual.shouldTint = false
    
    # PvP模式下显示剩余血量
    damage_visual.shouldShowRemainingHealth = GameState.is_pvp_mode
```

### 自定义伤害反馈
```gdscript
# 扩展DamageVisualComponent添加自定义效果
class_name ExtendedDamageVisual
extends DamageVisualComponent

@export var screen_shake_intensity: float = 5.0
@export var damage_sound: AudioStream

func animate(damageAmount: int) -> void:
    super.animate(damageAmount)  # 调用原始动画
    
    # 添加屏幕震动
    if damageAmount > 10:
        CameraShake.shake(screen_shake_intensity, 0.3)
    
    # 播放伤害音效
    if damage_sound:
        GlobalSonic.playSound(damage_sound)
```

### 伤害类型特效
```gdscript
# 根据伤害类型显示不同效果
@export var damage_visual: DamageVisualComponent
@export var fire_material: ShaderMaterial
@export var ice_material: ShaderMaterial

func on_damage_received(damage_component: DamageComponent, amount: int, _factions: int):
    # 基础视觉效果
    damage_visual.animate(amount)
    
    # 根据伤害类型添加特殊效果
    match damage_component.damage_type:
        DamageComponent.Type.FIRE:
            apply_fire_effect()
        DamageComponent.Type.ICE:
            apply_ice_effect()
        DamageComponent.Type.ELECTRIC:
            apply_electric_effect()

func apply_fire_effect():
    var sprite = parentEntity.get_node("Sprite2D") as Sprite2D
    if sprite:
        sprite.material = fire_material
        await get_tree().create_timer(0.5).timeout
        sprite.material = null
```

## 设计模式

### 视觉反馈系统
```gdscript
# 统一的视觉反馈管理
class_name VisualFeedbackManager
extends Node

@export var damage_visual: DamageVisualComponent
@export var health_visual: HealthVisualComponent

func setup_visual_feedback():
    # 伤害效果：显示伤害值
    damage_visual.shouldEmitBubble = true
    damage_visual.shouldShowRemainingHealth = false
    
    # 生命值效果：显示治疗值
    health_visual.shouldEmitBubble = true
    health_visual.shouldTint = false  # 避免与伤害着色冲突
```

### 状态同步
```gdscript
# 与其他系统同步状态
@export var damage_visual: DamageVisualComponent
@export var health_bar_ui: HealthBarUI

func sync_visual_states():
    # 同步着色状态
    if damage_visual.shouldTint:
        health_bar_ui.should_show_danger_color = true
    
    # 同步显示模式
    health_bar_ui.show_numbers = damage_visual.shouldShowRemainingHealth
```

## 技术细节

### 着色算法
- 红色强度 = `(1.0 - health_percentage) * 5.0`
- 使用`Animations.tweenProperty()`进行平滑过渡
- 着色应用到整个实体的`modulate`属性

### 性能优化
- 仅在受到伤害时更新视觉效果
- 使用对象池技术管理TextBubble
- 避免频繁的材质创建和销毁

### 兼容性
- 支持HealthComponent和ShieldedHealthComponent
- 自动检测护盾系统并适配显示

## 注意事项

### 实验性提醒
- 此组件为实验性质，未来版本可能有重大变更
- 建议在正式项目中谨慎使用
- API稳定性无法保证

### 性能考虑
- 频繁的着色更新可能影响性能
- 大量实体同时受伤时需要优化
- 考虑使用对象池管理视觉效果

### 与HealthVisualComponent的区别
- DamageVisualComponent专注于伤害事件
- HealthVisualComponent监控生命值状态
- 两者可以同时使用但需要协调

## 相关组件
- [`HealthVisualComponent`](./HealthVisualComponent.md) - 生命值视觉效果
- [`DamageReceivingComponent`](./DamageReceivingComponent.md) - 伤害接收
- [`HealthComponent`](./HealthComponent.md) - 生命值管理
- [`ShieldedHealthComponent`](./ShieldedHealthComponent.md) - 护盾生命值