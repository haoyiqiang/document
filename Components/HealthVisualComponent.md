# HealthVisualComponent

## 概述
`HealthVisualComponent` 是一个生命值视觉效果组件，当`HealthComponent`的生命值发生变化时显示视觉效果和指示器。支持闪烁动画、颜色渐变、文字气泡等多种视觉反馈，为玩家提供直观的生命值变化信息。

**继承关系：** `HealthVisualComponent` → `Component` → `Node`
**实验特性：** 标记为@experimental

## 主要特性
- ✨ 受伤时闪烁动画效果
- 🎨 基于生命值的颜色渐变
- 💬 文字气泡显示数值变化
- 🎯 自动节点检测
- ⚙️ 高度可配置的视觉效果
- 🔄 支持治疗和伤害的不同显示
- 📊 生命值百分比可视化

## 依赖组件
- **HealthComponent** (必需) - 生命值管理组件，也支持ShieldedHealthComponent

## 导出属性

### `nodeToAnimate: CanvasItem`
要显示效果的目标节点。

**类型：** `CanvasItem`  
**自动检测：** 如果未指定，会自动查找第一个AnimatedSprite2D或Sprite2D兄弟节点  
**用途：** 应用闪烁、渐变等视觉效果的目标

### `blinkCount: int = 3`
闪烁动画的次数。

**类型：** `int`  
**默认值：** `3`  
**用途：** 受伤时的闪烁次数

### `blinkDuration: float = 0.05`
闪烁动画的速度。

**类型：** `float`  
**默认值：** `0.05`  
**单位：** 秒  
**用途：** 单次闪烁（隐藏-显示）的持续时间

### `shouldTint: bool = false`
是否启用颜色渐变效果。

**类型：** `bool`  
**默认值：** `false`  
**实验特性：** 标记为@experimental  
**功能：** 启用时根据生命值百分比添加红色渐变

### `shouldEmitBubble: bool = true`
是否显示文字气泡。

**类型：** `bool`  
**默认值：** `true`  
**功能：** 显示代表当前生命值或变化量的TextBubble

### `detachedBubbles: bool = false`
文字气泡是否与实体分离。

**类型：** `bool`  
**默认值：** `false`  
**功能：** true时气泡不会跟随实体移动

### `shouldShowRemainingHealth: bool = false`
是否显示剩余生命值。

**类型：** `bool`  
**默认值：** `false`  
**功能：** true时气泡显示剩余生命值，false时显示变化量

## 状态属性

### `healthComponent: HealthComponent` (只读)
关联的生命值组件引用。

**类型：** `HealthComponent`  
**自动检测：** 通过`parentEntity.findFirstComponentSubclass(HealthComponent)`获取  
**支持：** 也兼容ShieldedHealthComponent

## 核心方法

### `connectSignals() -> void`
连接生命值变化信号。

**功能：** 连接HealthComponent的healthDidDecrease和healthDidIncrease信号

### `onHealthComponent_healthChanged(difference: int) -> void`
生命值变化事件处理。

**参数：** `difference` - 生命值变化量（正数为治疗，负数为伤害）  
**功能：**
1. 调用动画效果
2. 发射文字气泡（如果启用）

### `animate(difference: int) -> void`
执行视觉动画效果。

**参数：** `difference` - 生命值变化量  
**功能：**
1. 伤害时播放闪烁动画
2. 更新颜色渐变

### `updateTint() -> void`
更新颜色渐变效果。

**功能：** 根据生命值百分比计算红色渐变强度

### `emitBubble(difference: int) -> void`
发射文字气泡。

**参数：** `difference` - 生命值变化量  
**功能：**
1. 生成显示文本（剩余值或变化量）
2. 设置颜色（绿色为治疗，橙色为伤害）
3. 创建TextBubble

## 使用示例

### 基本设置
```gdscript
# 为实体设置生命值视觉效果
func setupHealthVisuals():
    var healthVisual = $HealthVisualComponent
    
    # 启用基本效果
    healthVisual.shouldEmitBubble = true
    healthVisual.blinkCount = 5
    healthVisual.blinkDuration = 0.1
    
    # 指定动画目标（可选，会自动检测）
    healthVisual.nodeToAnimate = $Sprite2D

# 在实体受到伤害时会自动显示：
# 1. 闪烁5次
# 2. 显示伤害数值气泡（如-10）
```

### 高级视觉效果
```gdscript
# 启用所有视觉效果
func setupAdvancedHealthVisuals():
    var healthVisual = $HealthVisualComponent
    
    # 启用颜色渐变
    healthVisual.shouldTint = true
    
    # 自定义闪烁效果
    healthVisual.blinkCount = 8
    healthVisual.blinkDuration = 0.03  # 更快的闪烁
    
    # 显示剩余生命值而不是变化量
    healthVisual.shouldShowRemainingHealth = true
    
    # 连接额外的信号处理
    var healthComponent = $HealthComponent
    healthComponent.healthDidDecrease.connect(_on_health_decreased)
    healthComponent.healthDidIncrease.connect(_on_health_increased)

func _on_health_decreased(amount: int):
    print("受到 ", abs(amount), " 点伤害!")
    
    # 播放受伤音效
    $SFX/HurtSound.play()
    
    # 显示屏幕震动
    Camera2D.add_trauma(0.3)

func _on_health_increased(amount: int):
    print("恢复 ", amount, " 点生命值!")
    
    # 播放治疗音效
    $SFX/HealSound.play()
    
    # 显示治疗粒子效果
    $VFX/HealParticles.emit()
```

### 自定义文字气泡
```gdscript
# 自定义气泡样式和位置
func setupCustomBubbles():
    var healthVisual = $HealthVisualComponent
    
    # 启用分离的气泡
    healthVisual.detachedBubbles = true
    healthVisual.shouldEmitBubble = true
    
    # 连接信号以自定义气泡
    healthVisual.healthComponent.healthDidDecrease.connect(_on_damage_bubble)
    healthVisual.healthComponent.healthDidIncrease.connect(_on_heal_bubble)

func _on_damage_bubble(amount: int):
    # 创建自定义伤害气泡
    var bubble = TextBubble.create(
        "-%d" % abs(amount), 
        get_tree().current_scene, 
        global_position + Vector2(0, -30)
    )
    
    # 自定义样式
    bubble.label.label_settings.font_color = Color.RED
    bubble.label.label_settings.font_size = 24
    
    # 添加动画
    var tween = create_tween()
    tween.parallel().tween_property(bubble, "position", bubble.position + Vector2(0, -50), 1.0)
    tween.parallel().tween_property(bubble, "modulate:a", 0.0, 1.0)
    tween.tween_callback(bubble.queue_free)

func _on_heal_bubble(amount: int):
    # 创建自定义治疗气泡
    var bubble = TextBubble.create(
        "+%d" % amount, 
        get_tree().current_scene, 
        global_position + Vector2(0, -30)
    )
    
    # 治疗气泡样式
    bubble.label.label_settings.font_color = Color.GREEN
    bubble.label.label_settings.font_size = 20
    
    # 向上浮动动画
    var tween = create_tween()
    tween.parallel().tween_property(bubble, "position", bubble.position + Vector2(0, -40), 0.8)
    tween.parallel().tween_property(bubble, "modulate:a", 0.0, 0.8)
    tween.tween_callback(bubble.queue_free)
```

### Boss血条集成
```gdscript
# Boss实体的生命值可视化
class_name BossHealthVisual
extends HealthVisualComponent

@export var bossHealthBarUI: Control
@export var healthBarFill: ProgressBar

func _ready():
    super._ready()
    
    # 连接额外的UI更新
    healthComponent.healthDidChange.connect(_update_boss_health_bar)
    
    # 初始化血条
    _update_boss_health_bar(0)

func _update_boss_health_bar(difference: int):
    if not healthBarFill:
        return
    
    var health = healthComponent.health
    healthBarFill.value = health.value
    healthBarFill.max_value = health.maxValue
    
    # 更新血条颜色
    var healthPercent = health.percentage / 100.0
    if healthPercent > 0.6:
        healthBarFill.modulate = Color.GREEN
    elif healthPercent > 0.3:
        healthBarFill.modulate = Color.YELLOW
    else:
        healthBarFill.modulate = Color.RED

func animate(difference: int):
    super.animate(difference)
    
    # Boss特殊效果
    if difference < 0:
        # 受伤时屏幕震动
        Camera2D.add_trauma(0.5)
        
        # 血条震动
        if bossHealthBarUI:
            var tween = create_tween()
            tween.tween_property(bossHealthBarUI, "position", bossHealthBarUI.position + Vector2(5, 0), 0.05)
            tween.tween_property(bossHealthBarUI, "position", bossHealthBarUI.position - Vector2(5, 0), 0.05)
            tween.tween_property(bossHealthBarUI, "position", bossHealthBarUI.position, 0.05)
```

### 多层次生命值显示
```gdscript
# 支持护盾的生命值可视化
func setupShieldedHealthVisuals():
    var healthVisual = $HealthVisualComponent
    
    # 检查是否有护盾生命值组件
    var shieldedHealth = $ShieldedHealthComponent
    if shieldedHealth:
        # 连接护盾特有信号
        shieldedHealth.shieldDidDecrease.connect(_on_shield_damaged)
        shieldedHealth.shieldDidIncrease.connect(_on_shield_recharged)
        shieldedHealth.shieldDidBreak.connect(_on_shield_broken)
        shieldedHealth.shieldDidRestore.connect(_on_shield_restored)

func _on_shield_damaged(amount: int):
    # 护盾受损时的特殊效果
    var bubble = TextBubble.create(
        "-%d (护盾)" % abs(amount), 
        parentEntity
    )
    bubble.label.label_settings.font_color = Color.CYAN

func _on_shield_recharged(amount: int):
    # 护盾恢复
    var bubble = TextBubble.create(
        "+%d (护盾)" % amount, 
        parentEntity
    )
    bubble.label.label_settings.font_color = Color.LIGHT_BLUE

func _on_shield_broken():
    # 护盾破碎特效
    print("护盾破碎!")
    $VFX/ShieldBreakEffect.emit()

func _on_shield_restored():
    # 护盾恢复特效
    print("护盾恢复!")
    $VFX/ShieldRestoreEffect.emit()
```

### 团队生命值同步
```gdscript
# 团队成员生命值可视化
class_name TeamHealthVisual
extends HealthVisualComponent

@export var teamMemberID: int
@export var shouldUpdateTeamUI: bool = true

func _ready():
    super._ready()
    
    if shouldUpdateTeamUI:
        healthComponent.healthDidChange.connect(_update_team_ui)

func _update_team_ui(difference: int):
    # 更新团队UI中的生命值显示
    var teamUI = get_node("/UI/TeamHealthDisplay")
    if teamUI and teamUI.has_method("updateMemberHealth"):
        teamUI.updateMemberHealth(teamMemberID, healthComponent.health)

func emitBubble(difference: int):
    # 为团队成员使用不同的气泡样式
    var text: String = str(healthComponent.health.value) if shouldShowRemainingHealth else "%+d" % difference
    
    # 根据团队角色设置颜色
    var color: Color
    if difference > 0:
        color = Color.GREEN
    else:
        match teamMemberID:
            0: color = Color.RED      # 队长
            1: color = Color.BLUE     # 法师
            2: color = Color.YELLOW   # 射手
            _: color = Color.ORANGE   # 其他
    
    if not detachedBubbles:
        TextBubble.create(text, self.parentEntity) \
            .label.label_settings.font_color = color
    else:
        TextBubble.create(text, parentEntity.get_parent(), parentEntity.global_position) \
            .label.label_settings.font_color = color
```

### 性能优化版本
```gdscript
# 大量实体时的性能优化
class_name OptimizedHealthVisual
extends HealthVisualComponent

@export var enableEffectsOnlyWhenVisible: bool = true
@export var maxBubbleCount: int = 3

var bubbleCount: int = 0
var isOnScreen: bool = false

func _ready():
    super._ready()
    
    if enableEffectsOnlyWhenVisible:
        # 只在可见时启用效果
        var viewport = get_viewport()
        if viewport:
            viewport.get_camera_2d().enabled_changed.connect(_check_visibility)

func _check_visibility():
    var camera = get_viewport().get_camera_2d()
    if not camera:
        return
    
    var screen_rect = camera.get_screen_center_position()
    var entity_pos = parentEntity.global_position
    
    # 简单的可见性检查
    var distance = screen_rect.distance_to(entity_pos)
    isOnScreen = distance < 500  # 可调整的距离

func animate(difference: int):
    if enableEffectsOnlyWhenVisible and not isOnScreen:
        return
    
    super.animate(difference)

func emitBubble(difference: int):
    if enableEffectsOnlyWhenVisible and not isOnScreen:
        return
    
    if bubbleCount >= maxBubbleCount:
        return
    
    bubbleCount += 1
    super.emitBubble(difference)
    
    # 减少气泡计数
    await get_tree().create_timer(2.0).timeout
    bubbleCount = max(0, bubbleCount - 1)
```

## 设计模式

### 观察者模式
- **信号监听：** 监听HealthComponent的生命值变化信号
- **自动响应：** 生命值变化时自动触发视觉效果
- **松耦合：** 视觉效果与生命值逻辑分离

### 策略模式
- **效果策略：** 不同的视觉效果可以独立启用/禁用
- **显示策略：** 支持显示变化量或剩余值的不同策略
- **气泡策略：** 支持附着和分离的气泡显示策略

### 装饰者模式
- **效果叠加：** 可以同时应用多种视觉效果
- **功能增强：** 在基础生命值功能上增加视觉反馈
- **可选装饰：** 每种效果都可以独立控制

## 技术细节

### 自动节点检测
```gdscript
if not nodeToAnimate: 
    nodeToAnimate = parentEntity.findFirstChildOfAnyTypes([AnimatedSprite2D, Sprite2D])
```

### 颜色渐变计算
```gdscript
var red: float = (1.0 - (health.percentage / 100.0)) * 5.0
```

### 防止重复信号
使用`@experimental`标记提醒开发者某些功能可能会变化。

### 性能考虑
- 只在必要时更新视觉效果
- 可选的屏幕可见性检查
- 限制同时显示的气泡数量

## 注意事项

1. **实验特性：** 部分功能标记为@experimental，可能在未来版本中变化
2. **性能影响：** 大量实体时考虑启用可见性检查
3. **节点依赖：** 需要找到合适的动画目标节点
4. **信号连接：** 确保HealthComponent正确发送信号
5. **气泡管理：** 避免创建过多的TextBubble影响性能

## 相关组件
- `HealthComponent` - 生命值管理，提供变化信号
- `ShieldedHealthComponent` - 护盾生命值，扩展功能支持
- `DamageVisualComponent` - 伤害特定的视觉效果
- `TextBubble` - 文字气泡显示系统
- `Animations` - 动画工具类，提供闪烁等效果 