# StationaryHidingPositionControlComponent API

> **继承关系**: Component > PositionControlComponent > StationaryHidingPositionControlComponent  
> **控制类型**: 自动隐藏位置控制  
> **依赖组件**: 可与MouseTrackingComponent配合

继承自PositionControlComponent的智能隐藏控制组件。当没有玩家输入时自动隐藏实体，适用于瞄准光标、UI元素等需要智能显示/隐藏的组件。

## ✨ 主要特性

- 🫥 无输入时自动隐藏
- 🎮 支持手柄和鼠标双重控制
- 📺 平滑淡入淡出动画
- ⏰ 可配置隐藏延迟
- 🖱️ 鼠标移动智能显示
- 🔄 完全兼容位置控制功能

## 📊 导出属性

### 隐藏控制
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldHideOnReady` | `bool` | `true` | 组件就绪时是否立即隐藏实体 |

### 动画设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `animationDuration` | `float` | `0.5` | 淡入淡出动画持续时间 |

### 继承属性
继承自PositionControlComponent的所有属性：
- `speed` - 移动速度
- `shouldUseSecondaryAxis` - 是否使用副轴输入
- `isEnabled` - 是否启用控制

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `hidingTimer` | `Timer` | 隐藏延迟计时器 |
| `tween` | `Tween` | 淡入淡出动画补间 |
| `haveMouseTrackingComponent` | `bool` | 是否存在鼠标跟踪组件 |

## 🎯 使用示例

### 基础瞄准光标

```gdscript
# Entity Scene Structure:
# └── AimCursor (Node2D)
#     ├── Sprite2D
#     ├── StationaryHidingPositionControlComponent
#     │   ├── HidingTimer
#     │   └── [设置]
#     │       shouldHideOnReady: true
#     │       speed: 300.0
#     └── [可选] MouseTrackingComponent
```

### 智能UI光标系统

```gdscript
# SmartCursorComponent.gd
extends StationaryHidingPositionControlComponent

@export var adaptiveSpeed: bool = true
@export var baseSpeed: float = 200.0
@export var maxSpeed: float = 600.0
@export var showOnTargetFound: bool = true
@export var targetDetectionRadius: float = 50.0

var targetsInRange: Array[Node2D] = []
var isOverTarget: bool = false

func _ready():
    super._ready()
    setupTargetDetection()

func setupTargetDetection():
    # 设置区域检测
    var area = Area2D.new()
    var shape = CircleShape2D.new()
    shape.radius = targetDetectionRadius
    var collision = CollisionShape2D.new()
    collision.shape = shape
    
    parentEntity.add_child(area)
    area.add_child(collision)
    
    area.area_entered.connect(_on_target_entered)
    area.area_exited.connect(_on_target_exited)
    area.body_entered.connect(_on_target_entered)
    area.body_exited.connect(_on_target_exited)

func _process(delta: float):
    # 自适应速度
    if adaptiveSpeed:
        adjustSpeedBasedOnContext()
    
    # 处理目标悬停显示
    if showOnTargetFound and isOverTarget:
        showEntity()
    
    super._process(delta)

func adjustSpeedBasedOnContext():
    if isOverTarget:
        # 在目标上方时降低速度，提高精度
        speed = baseSpeed * 0.5
    elif targetsInRange.size() > 0:
        # 目标附近时中等速度
        speed = baseSpeed * 0.8
    else:
        # 自由移动时最大速度
        speed = maxSpeed

func _on_target_entered(target: Node):
    if target.has_method("is_targetable") and target.is_targetable():
        targetsInRange.append(target)
        isOverTarget = true
        
        # 目标悬停时显示光标
        if showOnTargetFound:
            showEntity()
        
        print("Target detected: ", target.name)

func _on_target_exited(target: Node):
    targetsInRange.erase(target)
    isOverTarget = targetsInRange.size() > 0
    
    print("Target lost: ", target.name)

func showEntity():
    # 覆盖父类方法，添加额外效果
    super.showEntity()
    
    # 添加脉冲效果表示找到目标
    if isOverTarget:
        createPulseEffect()

func createPulseEffect():
    if tween:
        tween.kill()
    
    tween = parentEntity.create_tween()
    tween.set_loops(2)
    
    var originalScale = parentEntity.scale
    var pulseScale = originalScale * 1.2
    
    tween.tween_property(parentEntity, "scale", pulseScale, 0.1)
    tween.tween_property(parentEntity, "scale", originalScale, 0.1)

func getTargetsInRange() -> Array[Node2D]:
    return targetsInRange.duplicate()

func setTargetDetectionRadius(radius: float):
    targetDetectionRadius = radius
    # 更新碰撞形状...
```

### 上下文敏感光标

```gdscript
# ContextSensitiveCursor.gd
extends StationaryHidingPositionControlComponent

@export var contextualAppearance: bool = true
@export var interactionModeSprite: Texture2D
@export var combatModeSprite: Texture2D
@export var navigationModeSprite: Texture2D

enum CursorMode {
    HIDDEN,
    INTERACTION,
    COMBAT,
    NAVIGATION
}

var currentMode: CursorMode = CursorMode.HIDDEN
var sprite: Sprite2D

func _ready():
    super._ready()
    sprite = parentEntity.get_node("Sprite2D")
    updateCursorAppearance()

func _process(delta: float):
    super._process(delta)
    
    if contextualAppearance:
        updateContextualMode()

func updateContextualMode():
    var newMode = determineContextualMode()
    
    if newMode != currentMode:
        setMode(newMode)

func determineContextualMode() -> CursorMode:
    # 检查是否在UI上
    if isOverInteractable():
        return CursorMode.INTERACTION
    
    # 检查是否在敌人附近
    if isNearCombatTarget():
        return CursorMode.COMBAT
    
    # 检查是否有输入
    if not lastInput.is_zero_approx():
        return CursorMode.NAVIGATION
    
    return CursorMode.HIDDEN

func isOverInteractable() -> bool:
    var space_state = parentEntity.get_world_2d().direct_space_state
    var query = PhysicsPointQueryParameters2D.new()
    query.position = parentEntity.global_position
    query.collision_mask = 1  # UI层
    
    var result = space_state.intersect_point(query)
    return not result.is_empty()

func isNearCombatTarget() -> bool:
    var enemies = get_tree().get_nodes_in_group("enemies")
    
    for enemy in enemies:
        var distance = parentEntity.global_position.distance_to(enemy.global_position)
        if distance < 100.0:  # 100像素范围内
            return true
    
    return false

func setMode(mode: CursorMode):
    currentMode = mode
    updateCursorAppearance()
    
    match mode:
        CursorMode.HIDDEN:
            hideEntity()
        CursorMode.INTERACTION:
            showEntity()
            speed = baseSpeed * 0.6  # 交互时降速
        CursorMode.COMBAT:
            showEntity()
            speed = baseSpeed * 1.2  # 战斗时提速
        CursorMode.NAVIGATION:
            showEntity()
            speed = baseSpeed

func updateCursorAppearance():
    if not sprite:
        return
    
    match currentMode:
        CursorMode.INTERACTION:
            sprite.texture = interactionModeSprite
            sprite.modulate = Color.CYAN
        CursorMode.COMBAT:
            sprite.texture = combatModeSprite
            sprite.modulate = Color.RED
        CursorMode.NAVIGATION:
            sprite.texture = navigationModeSprite
            sprite.modulate = Color.WHITE
        _:
            sprite.modulate = Color.WHITE

func showEntity():
    super.showEntity()
    updateCursorAppearance()

# 自定义输入处理
func _input(event: InputEvent):
    super._input(event)
    
    # 右键切换模式
    if event.is_action_pressed("right_click"):
        cycleCursorMode()

func cycleCursorMode():
    var modes = [CursorMode.NAVIGATION, CursorMode.INTERACTION, CursorMode.COMBAT]
    var currentIndex = modes.find(currentMode)
    var nextIndex = (currentIndex + 1) % modes.size()
    setMode(modes[nextIndex])
    
    print("Cursor mode changed to: ", CursorMode.keys()[currentMode])
```

### 多人游戏光标同步

```gdscript
# MultiplayerCursor.gd
extends StationaryHidingPositionControlComponent

@export var playerId: int = 0
@export var syncToNetwork: bool = true
@export var showOtherPlayers: bool = true
@export var playerColors: Array[Color] = [Color.RED, Color.BLUE, Color.GREEN, Color.YELLOW]

var networkCursors: Dictionary = {}
var lastSyncTime: float = 0.0
var syncInterval: float = 0.1  # 10fps同步

func _ready():
    super._ready()
    setupMultiplayerSync()

func setupMultiplayerSync():
    if not syncToNetwork:
        return
    
    # 连接多人游戏信号
    if multiplayer.has_method("peer_connected"):
        multiplayer.peer_connected.connect(_on_peer_connected)
        multiplayer.peer_disconnected.connect(_on_peer_disconnected)
    
    # 设置玩家颜色
    if playerId < playerColors.size():
        var sprite = parentEntity.get_node_or_null("Sprite2D")
        if sprite:
            sprite.modulate = playerColors[playerId]

func _process(delta: float):
    super._process(delta)
    
    if syncToNetwork:
        handleNetworkSync(delta)

func handleNetworkSync(delta: float):
    lastSyncTime += delta
    
    if lastSyncTime >= syncInterval:
        syncCursorPosition()
        lastSyncTime = 0.0

@rpc("unreliable")
func syncCursorPosition():
    if not multiplayer.is_server():
        return
    
    var cursorData = {
        "player_id": playerId,
        "position": parentEntity.global_position,
        "visible": parentEntity.visible,
        "mode": currentMode if has_method("getCurrentMode") else 0
    }
    
    rpc("receiveCursorUpdate", cursorData)

@rpc("unreliable") 
func receiveCursorUpdate(cursorData: Dictionary):
    var otherPlayerId = cursorData.player_id
    
    if otherPlayerId == playerId:
        return  # 忽略自己的数据
    
    updateOtherPlayerCursor(cursorData)

func updateOtherPlayerCursor(cursorData: Dictionary):
    var otherPlayerId = cursorData.player_id
    
    if not networkCursors.has(otherPlayerId):
        createNetworkCursor(otherPlayerId)
    
    var otherCursor = networkCursors[otherPlayerId]
    otherCursor.global_position = cursorData.position
    otherCursor.visible = cursorData.visible and showOtherPlayers

func createNetworkCursor(otherPlayerId: int):
    var cursorNode = Node2D.new()
    var sprite = Sprite2D.new()
    
    # 复制当前光标的外观
    var mySprite = parentEntity.get_node_or_null("Sprite2D")
    if mySprite:
        sprite.texture = mySprite.texture
        sprite.scale = mySprite.scale * 0.8  # 稍小一些
    
    # 设置玩家颜色
    if otherPlayerId < playerColors.size():
        sprite.modulate = playerColors[otherPlayerId]
        sprite.modulate.a = 0.7  # 半透明
    
    cursorNode.add_child(sprite)
    get_tree().current_scene.add_child(cursorNode)
    
    networkCursors[otherPlayerId] = cursorNode
    
    print("Created network cursor for player: ", otherPlayerId)

func _on_peer_connected(id: int):
    print("Player connected: ", id)

func _on_peer_disconnected(id: int):
    if networkCursors.has(id):
        networkCursors[id].queue_free()
        networkCursors.erase(id)
        print("Removed cursor for disconnected player: ", id)

func _exit_tree():
    # 清理网络光标
    for cursor in networkCursors.values():
        if cursor:
            cursor.queue_free()
    networkCursors.clear()
```

### 辅助功能支持光标

```gdscript
# AccessibilityCursor.gd
extends StationaryHidingPositionControlComponent

@export var voiceAnnouncements: bool = false
@export var snapToTargets: bool = false
@export var snapDistance: float = 30.0
@export var highlightTargets: bool = true
@export var contrastMode: bool = false

var tts: TTSInterface
var snapTargets: Array[Node2D] = []
var currentHighlight: Node2D

func _ready():
    super._ready()
    setupAccessibility()

func setupAccessibility():
    # 设置TTS（文字转语音）
    if voiceAnnouncements and Engine.has_singleton("TTS"):
        tts = Engine.get_singleton("TTS")
    
    # 设置对比度模式
    if contrastMode:
        enableContrastMode()
    
    # 扫描可吸附目标
    if snapToTargets:
        updateSnapTargets()

func _process(delta: float):
    super._process(delta)
    
    if snapToTargets:
        handleTargetSnapping()
    
    if highlightTargets:
        updateTargetHighlight()

func handleTargetSnapping():
    if lastInput.is_zero_approx():
        return
    
    var nearestTarget = findNearestSnapTarget()
    if nearestTarget:
        var distance = parentEntity.global_position.distance_to(nearestTarget.global_position)
        if distance < snapDistance:
            # 吸附到目标
            parentEntity.global_position = nearestTarget.global_position
            announceTarget(nearestTarget)

func findNearestSnapTarget() -> Node2D:
    var nearest: Node2D = null
    var minDistance = snapDistance
    
    for target in snapTargets:
        if not target or not is_instance_valid(target):
            continue
        
        var distance = parentEntity.global_position.distance_to(target.global_position)
        if distance < minDistance:
            minDistance = distance
            nearest = target
    
    return nearest

func updateTargetHighlight():
    var nearestTarget = findNearestSnapTarget()
    
    if nearestTarget != currentHighlight:
        # 清除旧高亮
        if currentHighlight:
            removeHighlight(currentHighlight)
        
        # 添加新高亮
        if nearestTarget:
            addHighlight(nearestTarget)
        
        currentHighlight = nearestTarget

func addHighlight(target: Node2D):
    var highlight = ColorRect.new()
    highlight.color = Color.YELLOW
    highlight.color.a = 0.3
    highlight.size = Vector2(40, 40)
    highlight.position = Vector2(-20, -20)
    highlight.name = "AccessibilityHighlight"
    
    target.add_child(highlight)

func removeHighlight(target: Node2D):
    var highlight = target.get_node_or_null("AccessibilityHighlight")
    if highlight:
        highlight.queue_free()

func announceTarget(target: Node2D):
    if not voiceAnnouncements or not tts:
        return
    
    var announcement = "Target: " + target.name
    if target.has_method("getAccessibilityDescription"):
        announcement = target.getAccessibilityDescription()
    
    tts.speak(announcement)

func updateSnapTargets():
    snapTargets.clear()
    
    # 收集可交互的目标
    var interactables = get_tree().get_nodes_in_group("interactable")
    var buttons = get_tree().get_nodes_in_group("ui_buttons")
    
    snapTargets.append_array(interactables)
    snapTargets.append_array(buttons)
    
    print("Updated snap targets: ", snapTargets.size())

func enableContrastMode():
    # 高对比度设置
    var sprite = parentEntity.get_node_or_null("Sprite2D")
    if sprite:
        sprite.modulate = Color.WHITE
        # 添加黑色边框
        var outline = Sprite2D.new()
        outline.texture = sprite.texture
        outline.modulate = Color.BLACK
        outline.scale = Vector2(1.1, 1.1)
        outline.z_index = sprite.z_index - 1
        sprite.add_sibling(outline)

func _input(event: InputEvent):
    super._input(event)
    
    # 辅助功能快捷键
    if event.is_action_pressed("accessibility_announce"):
        announceCurrentContext()
    elif event.is_action_pressed("accessibility_snap_next"):
        snapToNextTarget()

func announceCurrentContext():
    if not voiceAnnouncements or not tts:
        return
    
    var context = "Cursor at " + str(parentEntity.global_position)
    if currentHighlight:
        context += ". Near " + currentHighlight.name
    
    tts.speak(context)

func snapToNextTarget():
    if snapTargets.is_empty():
        updateSnapTargets()
        return
    
    var currentIndex = snapTargets.find(currentHighlight) if currentHighlight else -1
    var nextIndex = (currentIndex + 1) % snapTargets.size()
    var nextTarget = snapTargets[nextIndex]
    
    if nextTarget:
        parentEntity.global_position = nextTarget.global_position
        updateTargetHighlight()
        announceTarget(nextTarget)
```

## 🔧 技术细节

### 隐藏逻辑流程
```gdscript
func _process(delta: float) -> void:
    if lastInput.is_zero_approx():
        # 无输入时启动隐藏计时器
        if parentEntity.visible and hidingTimer.is_stopped():
            hidingTimer.start()
    else:
        # 有输入时立即显示
        showEntity()
```

### 鼠标兼容性
```gdscript
func _input(event: InputEvent) -> void:
    if haveMouseTrackingComponent and event is InputEventMouseMotion:
        if not parentEntity.visible:
            showEntity()
```

### 淡入淡出动画
```gdscript
func hideEntity() -> void:
    tween = parentEntity.create_tween()
    var fadedModulate = parentEntity.modulate
    fadedModulate.a = 0
    tween.tween_property(parentEntity, "modulate", fadedModulate, animationDuration)
    tween.tween_property(parentEntity, "visible", false, 0)
```

## ⚠️ 注意事项

1. **组件冲突**: 可能与MouseTrackingComponent产生冲突，但设计上是兼容的
2. **初始状态**: shouldHideOnReady为true时，组件启动时实体会被隐藏
3. **动画性能**: 使用Tween实现平滑动画，大量实例可能影响性能
4. **输入检测**: 同时检测手柄和鼠标输入，确保多种控制方式兼容
5. **继承关系**: 完全继承PositionControlComponent的所有功能

## 🔗 相关组件

- [PositionControlComponent](PositionControlComponent.md) - 基础位置控制组件
- [MouseTrackingComponent](MouseTrackingComponent.md) - 鼠标跟踪组件
- [InputComponent](InputComponent.md) - 输入管理组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 