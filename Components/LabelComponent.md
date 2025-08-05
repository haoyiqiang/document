# LabelComponent API

> **继承关系**: Component > LabelComponent  
> **视觉类型**: 文本标签显示

在实体上方显示带动画的文本标签组件。支持自定义动画效果、文本样式和显示时机，适用于伤害数字、状态提示、对话气泡等场景。

## ✨ 主要特性

- 📝 动态文本显示
- 🎬 内置动画效果
- 🎨 可自定义样式
- ⚡ 异步动画支持
- 🔄 自动隐藏管理
- 🎯 可选父节点绑定

## 📊 导出属性

### 节点设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `parentNodeOverride` | `Node2D` | `null` | 自定义父节点（为空则使用父实体） |

### 子节点
| 节点 | 类型 | 描述 |
|------|------|------|
| `%Label` | `Label` | 显示文本的标签节点 |
| `%AnimationPlayer` | `AnimationPlayer` | 动画播放器 |

## 🎯 使用示例

### 基础伤害数字显示

```gdscript
# Entity Scene Structure:
# └── Enemy (CharacterBody2D)
#     ├── Sprite2D
#     ├── HealthComponent
#     ├── LabelComponent
#     │   ├── Label  # %Label
#     │   └── AnimationPlayer  # %AnimationPlayer
#     └── DamageReceivingComponent
#         └── [连接到伤害显示]

# DamageReceivingComponent.gd
extends Component

@export var damageLabel: LabelComponent

func _ready():
    # 连接伤害事件
    damage_received.connect(_on_damage_received)

func _on_damage_received(amount: int):
    if damageLabel:
        damageLabel.display(str(amount), "damage_popup")
```

### 增强型伤害数字系统

```gdscript
# EnhancedDamageLabel.gd
extends LabelComponent

@export var showCriticalHits: bool = true
@export var showHealingNumbers: bool = true
@export var colorByDamageType: bool = true
@export var floatingOffset: Vector2 = Vector2(0, -30)

var damageColors = {
    "normal": Color.WHITE,
    "critical": Color.YELLOW,
    "fire": Color.RED,
    "ice": Color.CYAN,
    "poison": Color.GREEN,
    "healing": Color.LIGHT_GREEN
}

func _ready():
    super._ready()
    setupDamageAnimations()

func setupDamageAnimations():
    # 创建多种伤害动画
    createDamageAnimation("normal_damage", Vector2(0, -20), 1.0, Color.WHITE)
    createDamageAnimation("critical_damage", Vector2(0, -30), 1.5, Color.YELLOW)
    createDamageAnimation("healing_popup", Vector2(0, -15), 1.2, Color.LIGHT_GREEN)
    createDamageAnimation("status_effect", Vector2(30, -10), 0.8, Color.PURPLE)

func createDamageAnimation(name: String, offset: Vector2, scale: float, color: Color):
    var animation = Animation.new()
    animation.length = 2.0
    
    # 位置轨道
    var positionTrack = animation.add_track(Animation.TYPE_POSITION_3D)
    animation.track_set_path(positionTrack, "%Label:position")
    animation.track_insert_key(positionTrack, 0.0, Vector2.ZERO)
    animation.track_insert_key(positionTrack, 0.5, offset)
    animation.track_insert_key(positionTrack, 2.0, offset + Vector2(0, -20))
    
    # 缩放轨道
    var scaleTrack = animation.add_track(Animation.TYPE_SCALE_3D)
    animation.track_set_path(scaleTrack, "%Label:scale")
    animation.track_insert_key(scaleTrack, 0.0, Vector2(0.1, 0.1))
    animation.track_insert_key(scaleTrack, 0.2, Vector2(scale, scale))
    animation.track_insert_key(scaleTrack, 2.0, Vector2(0.1, 0.1))
    
    # 透明度轨道
    var alphaTrack = animation.add_track(Animation.TYPE_VALUE)
    animation.track_set_path(alphaTrack, "%Label:modulate:a")
    animation.track_insert_key(alphaTrack, 0.0, 1.0)
    animation.track_insert_key(alphaTrack, 1.5, 1.0)
    animation.track_insert_key(alphaTrack, 2.0, 0.0)
    
    %AnimationPlayer.add_animation(name, animation)

func showDamage(amount: int, damageType: String = "normal", isCritical: bool = false):
    var text = str(amount)
    var animationName = "normal_damage"
    
    # 判断显示类型
    if amount < 0:  # 治疗
        text = "+" + str(abs(amount))
        animationName = "healing_popup"
        damageType = "healing"
    elif isCritical:
        text = str(amount) + "!"
        animationName = "critical_damage"
        damageType = "critical"
    
    # 设置颜色
    if colorByDamageType and damageType in damageColors:
        label.modulate = damageColors[damageType]
    
    display(text, animationName)

func showStatusEffect(effectName: String, isPositive: bool = true):
    var color = Color.GREEN if isPositive else Color.RED
    label.modulate = color
    display(effectName, "status_effect")

func showLevelUp():
    label.modulate = Color.GOLD
    display("LEVEL UP!", "critical_damage")

func showMissed():
    label.modulate = Color.GRAY
    display("MISS", "normal_damage")

func showBlocked():
    label.modulate = Color.BLUE
    display("BLOCKED", "normal_damage")

func showCriticalHit():
    label.modulate = Color.YELLOW
    display("CRITICAL!", "critical_damage")
```

### 对话气泡系统

```gdscript
# DialogBubbleComponent.gd
extends LabelComponent

@export var maxLineLength: int = 30
@export var typewriterSpeed: float = 50.0  # 字符每秒
@export var showSpeakerName: bool = true
@export var autoAdvance: bool = false
@export var autoAdvanceDelay: float = 3.0

var currentDialog: Array[String] = []
var currentLineIndex: int = 0
var typewriterTween: Tween
var speakerNameLabel: Label

func _ready():
    super._ready()
    setupDialogBubble()

func setupDialogBubble():
    # 创建说话者姓名标签
    if showSpeakerName:
        speakerNameLabel = Label.new()
        speakerNameLabel.name = "SpeakerName"
        speakerNameLabel.position = Vector2(0, -40)
        speakerNameLabel.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
        speakerNameLabel.add_theme_color_override("font_color", Color.YELLOW)
        label.add_sibling(speakerNameLabel)
    
    # 设置标签样式
    label.autowrap_mode = TextServer.AUTOWRAP_WORD_SMART
    label.custom_minimum_size = Vector2(200, 50)
    label.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
    label.vertical_alignment = VERTICAL_ALIGNMENT_CENTER
    
    # 创建背景
    var background = NinePatchRect.new()
    background.texture = preload("res://UI/dialog_bubble.png")
    background.position = Vector2(-110, -35)
    background.size = Vector2(220, 70)
    background.z_index = -1
    label.add_sibling(background)

func showDialog(speakerName: String, dialogLines: Array[String]):
    currentDialog = dialogLines
    currentLineIndex = 0
    
    if showSpeakerName and speakerNameLabel:
        speakerNameLabel.text = speakerName
        speakerNameLabel.visible = true
    
    showNextLine()

func showNextLine():
    if currentLineIndex >= currentDialog.size():
        hideDialog()
        return
    
    var line = currentDialog[currentLineIndex]
    line = wrapText(line, maxLineLength)
    
    if typewriterSpeed > 0:
        showTypewriter(line)
    else:
        display(line, "dialog_appear")
    
    currentLineIndex += 1

func wrapText(text: String, maxLength: int) -> String:
    var words = text.split(" ")
    var lines = []
    var currentLine = ""
    
    for word in words:
        if currentLine.length() + word.length() + 1 <= maxLength:
            if currentLine.length() > 0:
                currentLine += " "
            currentLine += word
        else:
            if currentLine.length() > 0:
                lines.append(currentLine)
            currentLine = word
    
    if currentLine.length() > 0:
        lines.append(currentLine)
    
    return "\n".join(lines)

func showTypewriter(text: String):
    label.text = ""
    label.visible = true
    
    if typewriterTween:
        typewriterTween.kill()
    
    typewriterTween = create_tween()
    
    var duration = text.length() / typewriterSpeed
    var characters = text.length()
    
    for i in range(characters + 1):
        var partialText = text.substr(0, i)
        typewriterTween.tween_callback(func(): label.text = partialText)
        typewriterTween.tween_delay(duration / characters)
    
    await typewriterTween.finished
    
    if autoAdvance:
        await get_tree().create_timer(autoAdvanceDelay).timeout
        showNextLine()

func hideDialog():
    if speakerNameLabel:
        speakerNameLabel.visible = false
    
    playAnimation("dialog_disappear")
    currentDialog.clear()
    currentLineIndex = 0

func _input(event: InputEvent):
    if event.is_action_pressed("ui_accept") and label.visible:
        if typewriterTween and typewriterTween.is_valid():
            # 跳过打字机效果
            typewriterTween.kill()
            label.text = currentDialog[currentLineIndex - 1] if currentLineIndex > 0 else ""
        else:
            # 显示下一行
            showNextLine()
```

### 状态提示系统

```gdscript
# StatusIndicatorComponent.gd
extends LabelComponent

@export var statusIcons: Dictionary = {}
@export var showStatusDuration: bool = true
@export var statusColors: Dictionary = {}

var activeStatuses: Dictionary = {}
var statusTimer: Timer

func _ready():
    super._ready()
    setupStatusSystem()

func setupStatusSystem():
    # 创建状态计时器
    statusTimer = Timer.new()
    statusTimer.wait_time = 0.5
    statusTimer.timeout.connect(_on_status_update)
    add_child(statusTimer)
    statusTimer.start()
    
    # 预设状态颜色
    statusColors = {
        "poison": Color.GREEN,
        "fire": Color.RED,
        "ice": Color.CYAN,
        "shield": Color.BLUE,
        "speed": Color.YELLOW,
        "strength": Color.ORANGE,
        "weakness": Color.PURPLE,
        "paralysis": Color.GRAY
    }

func addStatus(statusName: String, duration: float = -1):
    activeStatuses[statusName] = {
        "duration": duration,
        "startTime": Time.get_time_dict_from_system()
    }
    
    updateStatusDisplay()

func removeStatus(statusName: String):
    if statusName in activeStatuses:
        activeStatuses.erase(statusName)
        updateStatusDisplay()

func updateStatusDisplay():
    if activeStatuses.is_empty():
        label.visible = false
        return
    
    var statusText = ""
    var displayColor = Color.WHITE
    
    for statusName in activeStatuses.keys():
        var status = activeStatuses[statusName]
        var statusDisplay = statusName.to_upper()
        
        if showStatusDuration and status.duration > 0:
            statusDisplay += " (" + str(int(status.duration)) + "s)"
        
        if statusText.length() > 0:
            statusText += "\n"
        statusText += statusDisplay
        
        # 使用第一个状态的颜色
        if statusName in statusColors:
            displayColor = statusColors[statusName]
    
    label.modulate = displayColor
    display(statusText, "status_indicator")

func _on_status_update():
    var toRemove = []
    
    for statusName in activeStatuses.keys():
        var status = activeStatuses[statusName]
        if status.duration > 0:
            status.duration -= 0.5
            if status.duration <= 0:
                toRemove.append(statusName)
    
    for statusName in toRemove:
        removeStatus(statusName)
    
    if not activeStatuses.is_empty():
        updateStatusDisplay()
```

### 交互提示组件

```gdscript
# InteractionPromptComponent.gd
extends LabelComponent

@export var promptText: String = "Press E to interact"
@export var showOnlyWhenNear: bool = true
@export var interactionDistance: float = 100.0
@export var fadeInOut: bool = true

var player: Node2D
var isPlayerNear: bool = false

func _ready():
    super._ready()
    setupInteractionPrompt()

func setupInteractionPrompt():
    # 查找玩家
    player = get_tree().get_first_node_in_group("player")
    
    if not player:
        print("Warning: No player found for interaction prompt")
        return
    
    # 设置提示样式
    label.horizontal_alignment = HORIZONTAL_ALIGNMENT_CENTER
    label.add_theme_color_override("font_color", Color.YELLOW)
    
    # 创建背景
    var background = ColorRect.new()
    background.color = Color.BLACK
    background.color.a = 0.7
    background.position = Vector2(-50, -15)
    background.size = Vector2(100, 30)
    background.z_index = -1
    label.add_sibling(background)
    
    # 初始隐藏
    hidePrompt()

func _process(delta: float):
    if not player or not showOnlyWhenNear:
        return
    
    var distance = parentEntity.global_position.distance_to(player.global_position)
    var shouldShow = distance <= interactionDistance
    
    if shouldShow != isPlayerNear:
        isPlayerNear = shouldShow
        
        if isPlayerNear:
            showPrompt()
        else:
            hidePrompt()

func showPrompt():
    if fadeInOut:
        display(promptText, "fade_in")
    else:
        label.text = promptText
        label.visible = true

func hidePrompt():
    if fadeInOut:
        playAnimation("fade_out")
    else:
        label.visible = false

func setPromptText(text: String):
    promptText = text
    if label.visible:
        label.text = text

func setInteractionDistance(distance: float):
    interactionDistance = distance
```

## 🔧 技术细节

### 基础显示方法
```gdscript
func display(text: String, animation: StringName = Animations.blink) -> void:
    label.text = text
    playAnimation(animation)
```

### 动画播放
```gdscript
func playAnimation(animationName: StringName) -> void:
    %AnimationPlayer.stop()
    label.visible = true
    %AnimationPlayer.play(animationName)
    await %AnimationPlayer.animation_finished
    label.visible = false
```

### 闪烁效果
```gdscript
func blink(text: String) -> void:
    label.text = text
    playAnimation("blink")
```

## ⚠️ 注意事项

1. **子节点依赖**: 需要%Label和%AnimationPlayer子节点
2. **动画资源**: 需要预先创建动画资源
3. **异步操作**: playAnimation方法使用await，注意调用时机
4. **父节点设置**: parentNodeOverride为空时使用父实体
5. **可见性管理**: 组件自动管理标签的显示/隐藏

## 🔗 相关组件

- [HealthVisualComponent](HealthVisualComponent.md) - 生命值视觉效果
- [DamageVisualComponent](DamageVisualComponent.md) - 伤害视觉效果
- [StatsVisualComponent](StatsVisualComponent.md) - 统计数据视觉效果
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 