# PlatformerAnimationComponent

## 概述
`PlatformerAnimationComponent` 是一个平台游戏动画控制组件，根据实体的移动状态和CharacterBodyComponent、InputComponent的状态自动播放不同的动画。支持待机、行走、跳跃、下落等经典平台游戏动画，并能根据移动方向自动翻转精灵。

**继承关系：** `PlatformerAnimationComponent` → `Component` → `Node`

**要求：** AnimatedSprite2D，位于InputComponent和CharacterBodyComponent之后

## 主要特性
- 🎮 经典平台游戏动画支持
- 🚶 待机、行走、跳跃、下落动画
- 🔄 基于移动方向的自动翻转
- 📊 与CharacterBodyComponent和InputComponent集成
- ⚡ 性能优化的更新机制
- 🎯 自动精灵检测
- 🔧 可配置的动画名称

## 依赖组件
- **AnimatedSprite2D** (必需) - 动画精灵节点
- **CharacterBodyComponent** (必需) - 角色物理体组件
- **InputComponent** (可选) - 输入控制组件
- **ClimbComponent** (可选) - 攀爬组件（未来功能）

## 导出属性

### `animatedSprite: AnimatedSprite2D`
要控制的动画精灵节点。

**类型：** `AnimatedSprite2D`  
**自动检测：** 如果未指定，会自动使用实体的sprite属性或查找子节点  
**setter特性：** 自动控制_process的启用状态

### `idleAnimation: StringName = &"idle"`
待机动画名称。

**类型：** `StringName`  
**默认值：** `&"idle"`  
**用途：** 实体静止时播放的动画

### `walkAnimation: StringName = &"walk"`
行走动画名称。

**类型：** `StringName`  
**默认值：** `&"walk"`  
**用途：** 实体水平移动时播放的动画

### `jumpAnimation: StringName = &"jump"`
跳跃动画名称。

**类型：** `StringName`  
**默认值：** `&"jump"`  
**用途：** 实体向上移动（负Y速度）时播放的动画

### `fallAnimation: StringName = &"fall"`
下落动画名称。

**类型：** `StringName`  
**默认值：** `&"fall"`  
**用途：** 实体向下移动（正Y速度）时播放的动画

### `flipWhenWalkingLeft: bool = true`
向左移动时是否翻转精灵。

**类型：** `bool`  
**默认值：** `true`  
**功能：** 根据移动方向自动设置flip_h属性

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**setter特性：** 自动控制_process的启用状态以优化性能

## 状态属性

### `characterBodyComponent: CharacterBodyComponent` (只读)
关联的角色物理体组件。

**类型：** `CharacterBodyComponent`  
**获取：** `@onready var characterBodyComponent: CharacterBodyComponent = parentEntity.findFirstComponentSubclass(CharacterBodyComponent)`

### `body: CharacterBody2D` (只读)
CharacterBody2D节点引用。

**类型：** `CharacterBody2D`  
**获取：** 从characterBodyComponent获取

### `inputComponent: InputComponent` (只读，可选)
关联的输入组件。

**类型：** `InputComponent`  
**获取：** `coComponents.get(&"InputComponent")`  
**用途：** 检测输入方向变化

### `climbComponent: ClimbComponent` (只读，可选)
关联的攀爬组件。

**类型：** `ClimbComponent`  
**获取：** `coComponents.get(&"ClimbComponent")`  
**状态：** 未来功能，暂未使用

## 核心方法

### `onInputComponent_didChangeHorizontalDirection() -> void`
输入组件水平方向改变的处理。

**功能：** 根据当前输入方向更新精灵的flip_h属性

### `_process(_delta: float) -> void`
每帧动画更新处理。

**功能：**
1. 检测移动状态
2. 按优先级选择动画（idle < walk < jump/fall）
3. 播放对应动画
4. 处理精灵翻转（如果没有InputComponent）

## 信号

无独有信号，主要监听InputComponent的didChangeHorizontalDirection信号。

## 使用示例

### 基本平台游戏角色
```gdscript
# 设置基本的平台游戏角色动画
func setupPlatformerCharacter():
    var entity = preload("res://Entities/PlayerEntity.tscn").instantiate()
    var animComponent = entity.get_node("PlatformerAnimationComponent")
    
    # 配置动画名称（如果与默认值不同）
    animComponent.idleAnimation = &"player_idle"
    animComponent.walkAnimation = &"player_walk"
    animComponent.jumpAnimation = &"player_jump"
    animComponent.fallAnimation = &"player_fall"
    
    # 启用左右翻转
    animComponent.flipWhenWalkingLeft = true
    
    add_child(entity)
```

### 自定义动画控制
```gdscript
# 扩展PlatformerAnimationComponent添加更多动画
class_name ExtendedPlatformerAnimation
extends PlatformerAnimationComponent

@export var runAnimation: StringName = &"run"
@export var crouchAnimation: StringName = &"crouch"
@export var attackAnimation: StringName = &"attack"

@export var runSpeedThreshold: float = 100.0
var isAttacking: bool = false

func _ready():
    super._ready()
    
    # 连接攻击输入
    if inputComponent:
        inputComponent.didPressAction.connect(_on_action_pressed)

func _on_action_pressed(actionName: String):
    if actionName == "attack":
        startAttack()

func _process(delta: float):
    # 如果正在攻击，不更新其他动画
    if isAttacking:
        return
    
    # 检查是否在下蹲
    if inputComponent and inputComponent.isActionPressed("crouch"):
        animatedSprite.play(crouchAnimation)
        return
    
    # 调用父类的基本动画逻辑
    super._process(delta)
    
    # 检查是否应该播放跑步动画
    if not walkAnimation.is_empty() and not is_zero_approx(body.velocity.x):
        if abs(body.velocity.x) >= runSpeedThreshold:
            animatedSprite.play(runAnimation)

func startAttack():
    isAttacking = true
    animatedSprite.play(attackAnimation)
    
    # 等待攻击动画完成
    await animatedSprite.animation_finished
    isAttacking = false
```

### 状态机风格的动画控制
```gdscript
# 使用状态机管理动画
class_name StateMachineAnimation
extends PlatformerAnimationComponent

enum AnimationState {
    IDLE,
    WALK,
    RUN,
    JUMP,
    FALL,
    ATTACK,
    HURT
}

var currentState: AnimationState = AnimationState.IDLE
var stateTimer: float = 0.0

func _process(delta: float):
    stateTimer += delta
    
    var newState = determineState()
    if newState != currentState:
        changeState(newState)
    
    updateStateAnimation()

func determineState() -> AnimationState:
    # 优先级：受伤 > 攻击 > 空中状态 > 移动状态
    
    if isAttacking:
        return AnimationState.ATTACK
    
    if not characterBodyComponent.isOnFloor:
        if body.velocity.y < 0:
            return AnimationState.JUMP
        else:
            return AnimationState.FALL
    
    var horizontalSpeed = abs(body.velocity.x)
    if horizontalSpeed > runSpeedThreshold:
        return AnimationState.RUN
    elif horizontalSpeed > 1.0:
        return AnimationState.WALK
    else:
        return AnimationState.IDLE

func changeState(newState: AnimationState):
    print("动画状态切换: ", AnimationState.keys()[currentState], " -> ", AnimationState.keys()[newState])
    currentState = newState
    stateTimer = 0.0

func updateStateAnimation():
    match currentState:
        AnimationState.IDLE:
            animatedSprite.play(idleAnimation)
        AnimationState.WALK:
            animatedSprite.play(walkAnimation)
        AnimationState.RUN:
            animatedSprite.play(runAnimation)
        AnimationState.JUMP:
            animatedSprite.play(jumpAnimation)
        AnimationState.FALL:
            animatedSprite.play(fallAnimation)
        AnimationState.ATTACK:
            animatedSprite.play(attackAnimation)
        AnimationState.HURT:
            animatedSprite.play(&"hurt")
```

### 多角色支持
```gdscript
# 支持不同角色的动画配置
class_name CharacterSpecificAnimation
extends PlatformerAnimationComponent

enum CharacterType {
    PLAYER,
    ENEMY_GOBLIN,
    ENEMY_ORC,
    NPC
}

@export var characterType: CharacterType = CharacterType.PLAYER

func _ready():
    super._ready()
    setupCharacterAnimations()

func setupCharacterAnimations():
    match characterType:
        CharacterType.PLAYER:
            idleAnimation = &"player_idle"
            walkAnimation = &"player_walk"
            jumpAnimation = &"player_jump"
            fallAnimation = &"player_fall"
        
        CharacterType.ENEMY_GOBLIN:
            idleAnimation = &"goblin_idle"
            walkAnimation = &"goblin_walk"
            jumpAnimation = &"goblin_jump"
            fallAnimation = &"goblin_fall"
        
        CharacterType.ENEMY_ORC:
            idleAnimation = &"orc_idle"
            walkAnimation = &"orc_walk"
            jumpAnimation = &"orc_jump"
            fallAnimation = &"orc_fall"
        
        CharacterType.NPC:
            idleAnimation = &"npc_idle"
            walkAnimation = &"npc_walk"
            jumpAnimation = &""  # NPC不跳跃
            fallAnimation = &""
```

### 动画事件处理
```gdscript
# 处理动画帧事件
class_name EventDrivenAnimation
extends PlatformerAnimationComponent

func _ready():
    super._ready()
    
    # 连接动画信号
    animatedSprite.animation_finished.connect(_on_animation_finished)
    animatedSprite.frame_changed.connect(_on_frame_changed)

func _on_animation_finished():
    var animName = animatedSprite.animation
    print("动画完成: ", animName)
    
    match animName:
        &"attack":
            # 攻击动画完成，检查命中
            checkAttackHit()
        &"hurt":
            # 受伤动画完成，恢复控制
            resumeControl()
        &"special_move":
            # 特殊技能完成，重置状态
            resetSpecialMoveState()

func _on_frame_changed():
    var animName = animatedSprite.animation
    var frame = animatedSprite.frame
    
    # 在特定帧触发事件
    match animName:
        &"attack":
            if frame == 3:  # 攻击判定帧
                performAttackHit()
        &"walk":
            if frame in [2, 6]:  # 脚步声帧
                playFootstepSound()
        &"jump":
            if frame == 1:  # 起跳音效帧
                playJumpSound()

func performAttackHit():
    print("执行攻击判定")
    # 攻击判定逻辑

func playFootstepSound():
    $FootstepSFX.play()

func playJumpSound():
    $JumpSFX.play()
```

### 动画平滑过渡
```gdscript
# 带平滑过渡的动画控制
class_name SmoothAnimation
extends PlatformerAnimationComponent

@export var transitionSpeed: float = 5.0
var targetAnimation: StringName
var isTransitioning: bool = false

func _process(delta: float):
    # 先确定目标动画
    determineTargetAnimation()
    
    # 如果目标动画与当前不同，开始过渡
    if targetAnimation != animatedSprite.animation:
        if not isTransitioning:
            startTransition(targetAnimation)

func determineTargetAnimation():
    # 复制父类的动画选择逻辑
    var animationToPlay: StringName
    
    if flipWhenWalkingLeft and not inputComponent:
        animatedSprite.flip_h = signf(characterBodyComponent.previousVelocity.x) < 0
    
    # 待机动画
    if not idleAnimation.is_empty() and is_zero_approx(body.velocity.x):
        animationToPlay = idleAnimation
    
    # 行走动画
    if not walkAnimation.is_empty() and not is_zero_approx(body.velocity.x):
        animationToPlay = walkAnimation
    
    # 空中动画
    if not characterBodyComponent.isOnFloor:
        var verticalDirection: float = signf(body.velocity.y)
        if not jumpAnimation.is_empty() and verticalDirection < 0.0:
            animationToPlay = jumpAnimation
        elif not fallAnimation.is_empty() and verticalDirection > 0.0:
            animationToPlay = fallAnimation
    
    targetAnimation = animationToPlay

func startTransition(newAnimation: StringName):
    isTransitioning = true
    
    # 创建过渡效果
    var tween = create_tween()
    tween.tween_property(animatedSprite, "modulate:a", 0.5, 1.0 / transitionSpeed)
    tween.tween_callback(func(): changeAnimation(newAnimation))
    tween.tween_property(animatedSprite, "modulate:a", 1.0, 1.0 / transitionSpeed)
    tween.tween_callback(func(): isTransitioning = false)

func changeAnimation(newAnimation: StringName):
    animatedSprite.play(newAnimation)
```

### 性能优化版本
```gdscript
# 大量实体时的性能优化版本
class_name OptimizedAnimation
extends PlatformerAnimationComponent

@export var updateFrequency: float = 10.0  # 每秒更新次数
@export var enableOnlyWhenVisible: bool = true

var updateTimer: float = 0.0
var isVisible: bool = true

func _ready():
    super._ready()
    
    if enableOnlyWhenVisible:
        # 监听可见性变化
        animatedSprite.visibility_changed.connect(_on_visibility_changed)

func _process(delta: float):
    if not isEnabled or (enableOnlyWhenVisible and not isVisible):
        return
    
    updateTimer += delta
    if updateTimer >= 1.0 / updateFrequency:
        updateAnimation()
        updateTimer = 0.0

func updateAnimation():
    # 只在必要时更新动画
    var newAnimation = determineAnimation()
    if newAnimation != animatedSprite.animation:
        animatedSprite.play(newAnimation)

func determineAnimation() -> StringName:
    # 简化的动画确定逻辑
    if not characterBodyComponent.isOnFloor:
        return jumpAnimation if body.velocity.y < 0 else fallAnimation
    elif not is_zero_approx(body.velocity.x):
        return walkAnimation
    else:
        return idleAnimation

func _on_visibility_changed():
    isVisible = animatedSprite.visible
    if not isVisible:
        set_process(false)
    else:
        set_process(isEnabled)
```

## 设计模式

### 状态模式
- **动画状态：** 不同移动状态对应不同动画
- **状态切换：** 基于物理状态自动切换动画
- **优先级管理：** 动画按重要性优先级播放

### 观察者模式
- **输入监听：** 监听InputComponent的方向变化信号
- **状态响应：** 根据物理组件状态自动响应
- **事件驱动：** 基于信号的动画更新

### 策略模式
- **翻转策略：** 支持不同的精灵翻转策略
- **动画策略：** 可配置的动画名称映射
- **更新策略：** 支持不同的性能优化策略

## 技术细节

### 动画优先级
```gdscript
# 优先级：idle < walk < jump/fall
# 后面的检查会覆盖前面的结果
```

### 精灵翻转逻辑
```gdscript
# 使用InputComponent时
sprite.flip_h = signf(inputComponent.horizontalInput) < 0

# 无InputComponent时
animatedSprite.flip_h = signf(characterBodyComponent.previousVelocity.x) < 0
```

### 性能优化
- set_process()基于isEnabled和animatedSprite的有效性
- 避免不必要的动画切换
- 可选的可见性检查

### Y速度检查避免
注释中提到避免在idle和walk检查中包含Y速度，因为可能导致意外的walk动画。

## 注意事项

1. **组件顺序：** 必须在InputComponent和CharacterBodyComponent之后
2. **精灵检测：** 确保有合适的AnimatedSprite2D节点
3. **动画名称：** 确保动画资源中有对应的动画名称
4. **性能考虑：** 大量实体时考虑降低更新频率
5. **Y速度敏感：** 微小的Y速度变化可能影响动画选择

## 相关组件
- `CharacterBodyComponent` - 提供物理状态信息
- `InputComponent` - 提供输入方向信息
- `ClimbComponent` - 攀爬功能（未来支持）
- `AnimatedSprite2D` - 实际的动画播放节点 