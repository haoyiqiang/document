# ClimbComponent API

## 概述
`ClimbComponent` 是一个复杂的垂直移动控制组件，允许角色在梯子、绳索或悬崖等可攀爬区域进行垂直移动。它处理各种边缘情况，如跳跃取消攀爬、离开区域停止攀爬、半空中抓住梯子等复杂交互。

**继承链**：`AreaContactComponent` → `ClimbComponent`

## 依赖要求
- **CharacterBodyComponent**：角色物理体组件
- **PlatformerPhysicsComponent**：平台游戏物理组件
- **InputComponent**：输入控制组件
- **JumpComponent**：跳跃组件（在ClimbComponent之后）

**重要**：此组件必须在JumpComponent和PlatformerPhysicsComponent **之前** 在场景树中，以便正确抑制输入事件。

## 主要特性
- 🧗 **垂直攀爬**：在"climbable"组的Area2D中进行垂直移动
- 🎯 **智能边界**：水平和垂直边界限制，防止移出攀爬区域
- 🔄 **边缘处理**：处理跳跃取消、地面行走退出等复杂情况
- 📍 **位置校正**：自动对齐到攀爬区域或智能引导进入
- ⚡ **重力控制**：攀爬时自动禁用重力系统
- 🎮 **输入抑制**：可选择性抑制水平输入防止干扰

## 导出属性

### 边界控制
- **shouldConfineHorizontally** (`bool = true`)：攀爬时水平限制在区域内
  - 防止任何物理源造成的区域外移动
- **shouldConfineVertically** (`bool = true`)：攀爬时垂直限制在区域内
  - 防止超出攀爬区域的垂直移动

### 位置校正
- **shouldSnapToClimbableArea** (`bool = false`)：瞬间对齐到攀爬区域
  - 在地面时被`shouldWalkIntoClimbableArea`覆盖
- **shouldWalkIntoClimbableArea** (`bool = true`)：在地面时智能引导进入区域
  - 调整`PlatformerPhysicsComponent.inputDirection`使角色走向区域内部

### 输入控制
- **shouldAllowHorizontalInput** (`bool = true`)：是否允许水平输入
  - 在地面或平台上始终允许行走
  - 重要：组件必须在PlatformerPhysicsComponent之前以抑制输入
- **isPlayerControlled** (`bool = true`)：是否为玩家控制
- **cancelClimbInputActionName** (`StringName = jump`)：取消攀爬的输入动作

## 状态属性

### 攀爬状态
- **isInClimbableArea** (`bool`)：是否在可攀爬区域内
- **isClimbing** (`bool`)：是否正在攀爬
- **activeClimbingArea** (`Area2D`)：当前激活的攀爬区域

### 输入状态
- **lastVerticalInput** (`float`)：最后的垂直输入值
- **lastVerticalInputDirection** (`int`)：垂直输入方向（-1上，+1下）
- **isLastVerticalInputZero** (`bool`)：垂直输入是否为零

### 区域信息
- **activeClimbingAreaBounds** (`Rect2`)：活动攀爬区域的局部边界
- **activeClimbingAreaBoundsGlobal** (`Rect2`)：活动攀爬区域的全局边界

## 信号

### 区域事件
- **didEnterClimbableArea(area: Area2D)**：进入可攀爬区域
- **didExitClimbableArea(area: Area2D)**：离开可攀爬区域

### 攀爬事件
- **didStartClimb(area: Area2D)**：开始攀爬
- **didEndClimb(area: Area2D)**：结束攀爬

## 使用示例

### 基础攀爬设置
```gdscript
# 设置标准的攀爬功能
@export var climb_component: ClimbComponent

func _ready():
    climb_component.shouldConfineHorizontally = true
    climb_component.shouldConfineVertically = true
    climb_component.shouldAllowHorizontalInput = false  # 攀爬时锁定水平移动
```

### 灵活攀爬系统
```gdscript
# 允许攀爬时的水平移动
@export var climb_component: ClimbComponent

func setup_flexible_climbing():
    climb_component.shouldAllowHorizontalInput = true
    climb_component.shouldConfineHorizontally = false  # 允许从梯子侧面离开
    climb_component.shouldWalkIntoClimbableArea = true  # 智能进入
```

### 攀爬事件响应
```gdscript
# 响应攀爬事件
@export var climb_component: ClimbComponent
@export var animation_player: AnimationPlayer

func _ready():
    climb_component.didStartClimb.connect(on_climb_start)
    climb_component.didEndClimb.connect(on_climb_end)

func on_climb_start(area: Area2D):
    animation_player.play("climb_start")
    GlobalSonic.playSound(climb_grab_sound)

func on_climb_end(area: Area2D):
    animation_player.play("climb_end")
```

### 攀爬能力检查
```gdscript
# 基于角色能力的攀爬控制
@export var climb_component: ClimbComponent
@export var character_stats: StatsComponent

func _ready():
    climb_component.didEnterClimbableArea.connect(check_climb_ability)

func check_climb_ability(area: Area2D):
    if not character_stats.has_climbing_skill():
        climb_component.isEnabled = false
        GlobalUI.showMessage("需要攀爬技能！")
    else:
        climb_component.isEnabled = true
```

### 条件攀爬取消
```gdscript
# 自定义攀爬取消条件
@export var climb_component: ClimbComponent
@export var stamina_component: StaminaComponent

func _ready():
    stamina_component.stamina.valueChanged.connect(check_climbing_stamina)

func check_climbing_stamina():
    if climb_component.isClimbing and stamina_component.stamina.value <= 0:
        # 体力耗尽时强制结束攀爬
        climb_component.isClimbing = false
        GlobalUI.showMessage("体力不足，无法继续攀爬！")
```

## 设计模式

### 攀爬系统管理器
```gdscript
# 统一管理攀爬相关系统
class_name ClimbingSystemManager
extends Node

@export var climb_component: ClimbComponent
@export var stamina_component: StaminaComponent
@export var sound_component: AudioStreamPlayer2D

var climb_stamina_cost: float = 10.0

func _ready():
    climb_component.didStartClimb.connect(start_stamina_drain)
    climb_component.didEndClimb.connect(stop_stamina_drain)

func start_stamina_drain(_area: Area2D):
    # 开始消耗体力
    var tween = create_tween()
    tween.set_loops()
    tween.tween_callback(drain_stamina).set_delay(1.0)

func drain_stamina():
    if climb_component.isClimbing:
        stamina_component.stamina.value -= climb_stamina_cost
```

### 多层攀爬区域
```gdscript
# 处理重叠的攀爬区域
@export var climb_component: ClimbComponent

func _ready():
    climb_component.didEnterClimbableArea.connect(on_enter_climbable)
    climb_component.didExitClimbableArea.connect(on_exit_climbable)

func on_enter_climbable(area: Area2D):
    # 记录区域类型（梯子、绳索、岩壁等）
    var climb_type = area.get_meta("climb_type", "default")
    update_climb_parameters(climb_type)

func update_climb_parameters(climb_type: String):
    match climb_type:
        "rope":
            climb_component.shouldAllowHorizontalInput = true  # 绳索允许摇摆
        "ladder":
            climb_component.shouldAllowHorizontalInput = false  # 梯子固定攀爬
        "cliff":
            climb_component.shouldConfineVertically = false    # 悬崖可以超出边界
```

### 攀爬动画控制
```gdscript
# 复杂的攀爬动画系统
class_name ClimbAnimationController
extends Node

@export var climb_component: ClimbComponent
@export var animation_tree: AnimationTree

func _ready():
    climb_component.connectSignals()
    climb_component.didStartClimb.connect(set_climb_animation)
    climb_component.didEndClimb.connect(set_idle_animation)

func _physics_process(_delta):
    if climb_component.isClimbing:
        update_climb_animation_speed()

func update_climb_animation_speed():
    var input_strength = abs(climb_component.lastVerticalInput)
    animation_tree.set("parameters/climb_speed/scale", input_strength)

func set_climb_animation(_area: Area2D):
    animation_tree.set("parameters/conditions/climbing", true)

func set_idle_animation(_area: Area2D):
    animation_tree.set("parameters/conditions/climbing", false)
```

## 技术细节

### 区域检测机制
- 继承自`AreaContactComponent`
- 监控"climbable"组的Area2D
- 支持重叠区域的正确处理

### 重力控制集成
- 攀爬时自动禁用`PlatformerPhysicsComponent.isGravityEnabled`
- 结束攀爬时恢复重力
- 确保物理状态的一致性

### 边界限制算法
- 实时计算攀爬区域的全局边界
- 在角色移动后校正位置
- 支持复杂形状的攀爬区域

### 输入处理优先级
- 组件顺序确定输入处理优先级
- 在PlatformerPhysicsComponent之前处理
- 选择性抑制特定方向的输入

## 注意事项

### 组件顺序要求
- **必须**在JumpComponent和PlatformerPhysicsComponent之前
- 确保正确的输入抑制和重力控制
- 错误顺序可能导致功能冲突

### 性能考虑
- 复杂的碰撞检测和边界计算
- 在大量攀爬区域时可能影响性能
- 建议合理设计关卡布局

### 设计复杂性
- 处理多种边缘情况
- 与其他运动系统的交互复杂
- 需要仔细测试各种场景

## 相关组件
- [`JumpComponent`](./JumpComponent.md) - 跳跃控制
- [`InputComponent`](./InputComponent.md) - 输入处理
- [`CharacterBodyComponent`](./CharacterBodyComponent.md) - 角色物理
- [`AreaCollisionComponent`](./AreaCollisionComponent.md) - 区域碰撞检测 