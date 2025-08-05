# InjectorComponent API

## 概述
`InjectorComponent` 将其所有子节点（通常是其他组件）转移到第一个与之碰撞的实体。适用于应用buff/debuff效果，例如毒箭子弹给受害者添加`DamageOverTimeComponent`等临时效果组件。

**继承链**：`Component` → `InjectorComponent`

## 主要特性
- 💉 **节点转移**：将子节点转移到碰撞的实体
- 🎯 **碰撞触发**：通过Area2D或Body碰撞触发
- 🔧 **可配置**：支持多种转移行为配置
- 📍 **变换保持**：可选择保持全局变换
- ⚡ **自动管理**：自动设置子节点的process模式

## 导出属性

### 转移配置
- **shouldKeepGlobalTransform** (`bool = false`)：转移后是否保持全局变换
- **shouldHideBeforeTransfer** (`bool = false`)：转移前是否隐藏子节点
- **shouldShowAfterTransfer** (`bool = true`)：转移后是否显示子节点

### 处理模式配置
- **shouldSetProcessModeBeforeTransfer** (`bool = true`)：转移前是否设置处理模式
- **processModeBeforeTransfer** (`ProcessMode = PROCESS_MODE_DISABLED`)：转移前的处理模式
- **shouldSetProcessModeAfterTransfer** (`bool = true`)：转移后是否设置处理模式
- **processModeAfterTransfer** (`ProcessMode = PROCESS_MODE_INHERIT`)：转移后的处理模式

### 控制配置
- **isEnabled** (`bool = true`)：是否启用注入功能

## 状态属性
- **isInjecting** (`bool`)：是否正在进行注入操作

## 信号
- **willInject**(targetParent: Node)：即将注入时发出
- **didInject**(node: Node, newParent: Node)：每个子节点转移后发出

## 主要方法
- **inject**(newParentEntity: Entity, keepGlobalTransform: bool = shouldKeepGlobalTransform) → `Array[Node]`：执行注入操作
- **onAreaOrBodyEntered**(areaOrBody: Node2D) → `void`：Area2D或Body进入时的处理

## 使用示例

### 基础Buff注入
```gdscript
# 创建临时buff效果
@export var injector: InjectorComponent

func _ready():
    # 连接注入信号
    injector.willInject.connect(_on_will_inject)
    injector.didInject.connect(_on_did_inject)
    
    # 配置注入行为
    injector.shouldKeepGlobalTransform = false
    injector.shouldShowAfterTransfer = true

func _on_will_inject(target_entity: Entity):
    print(f"即将向 {target_entity.name} 注入效果")

func _on_did_inject(node: Node, new_parent: Node):
    print(f"成功注入组件: {node.name} → {new_parent.name}")
```

### 毒箭系统
```gdscript
# 毒箭注入毒素效果
@export var injector: InjectorComponent
@export var damage_over_time_component: DamageOverTimeComponent

func _ready():
    # 将毒素组件作为子节点
    injector.add_child(damage_over_time_component)
    damage_over_time_component.allowNonEntityParent = true
    
    # 配置毒素效果
    damage_over_time_component.damage = 5.0
    damage_over_time_component.interval = 1.0
    damage_over_time_component.duration = 10.0
    
    # 连接Area2D信号
    var area = get_node("Area2D") as Area2D
    area.body_entered.connect(injector.onAreaOrBodyEntered)
    
    injector.willInject.connect(_on_poison_inject)

func _on_poison_inject(target: Entity):
    # 播放毒素效果
    spawn_poison_visual_effect(target.global_position)
    
    # 销毁箭矢实体
    parentEntity.queue_free()
```

### 治疗法术系统
```gdscript
# 治疗法术注入治疗效果
@export var injector: InjectorComponent

func create_heal_over_time():
    # 创建治疗组件
    var heal_component = preload("res://components/HealOverTimeComponent.tscn").instantiate()
    heal_component.allowNonEntityParent = true
    heal_component.healAmount = 10.0
    heal_component.interval = 2.0
    heal_component.duration = 15.0
    
    injector.add_child(heal_component)

func create_heal_visual():
    # 创建治疗视觉效果
    var heal_visual = preload("res://effects/HealAura.tscn").instantiate()
    heal_visual.allowNonEntityParent = true
    
    injector.add_child(heal_visual)

func _ready():
    create_heal_over_time()
    create_heal_visual()
    
    # 只对友军生效
    var area = get_node("Area2D") as Area2D
    area.body_entered.connect(_on_body_entered)

func _on_body_entered(body: Node2D):
    var entity = body.get_parent() if body.get_parent() is Entity else Tools.findFirstParentOfType(body, Entity)
    if entity:
        var faction = entity.findFirstComponentSubclass(FactionComponent)
        if faction and faction.faction == "ally":
            injector.onAreaOrBodyEntered(body)
```

### 复杂状态效果
```gdscript
# 复杂状态效果注入器
@export var injector: InjectorComponent
@export var effect_type: String = "fire"
@export var effect_duration: float = 5.0

func _ready():
    setup_effect_components()
    
    var area = get_node("Area2D") as Area2D
    area.area_entered.connect(injector.onAreaOrBodyEntered)
    
    injector.didInject.connect(_on_effect_applied)

func setup_effect_components():
    match effect_type:
        "fire":
            create_fire_effect()
        "ice":
            create_ice_effect()
        "electric":
            create_electric_effect()

func create_fire_effect():
    # 燃烧伤害
    var burn_damage = preload("res://components/BurnDamageComponent.tscn").instantiate()
    burn_damage.allowNonEntityParent = true
    burn_damage.damage = 3.0
    burn_damage.duration = effect_duration
    injector.add_child(burn_damage)
    
    # 火焰视觉效果
    var fire_visual = preload("res://effects/FireEffect.tscn").instantiate()
    fire_visual.allowNonEntityParent = true
    injector.add_child(fire_visual)

func create_ice_effect():
    # 冰冻减速
    var slow_component = preload("res://components/SlowComponent.tscn").instantiate()
    slow_component.allowNonEntityParent = true
    slow_component.speedMultiplier = 0.5
    slow_component.duration = effect_duration
    injector.add_child(slow_component)
    
    # 冰霜视觉效果
    var ice_visual = preload("res://effects/IceEffect.tscn").instantiate()
    ice_visual.allowNonEntityParent = true
    injector.add_child(ice_visual)

func create_electric_effect():
    # 电击眩晕
    var stun_component = preload("res://components/StunComponent.tscn").instantiate()
    stun_component.allowNonEntityParent = true
    stun_component.duration = effect_duration * 0.5
    injector.add_child(stun_component)
    
    # 电击连锁伤害
    var chain_damage = preload("res://components/ChainDamageComponent.tscn").instantiate()
    chain_damage.allowNonEntityParent = true
    chain_damage.chainRange = 100.0
    chain_damage.maxTargets = 3
    injector.add_child(chain_damage)

func _on_effect_applied(node: Node, target: Node):
    # 播放应用效果音效
    var audio = AudioStreamPlayer2D.new()
    match effect_type:
        "fire":
            audio.stream = preload("res://audio/fire_apply.ogg")
        "ice":
            audio.stream = preload("res://audio/ice_apply.ogg")
        "electric":
            audio.stream = preload("res://audio/electric_apply.ogg")
    
    target.add_child(audio)
    audio.play()
    audio.finished.connect(audio.queue_free)
```

### 装备强化系统
```gdscript
# 装备强化注入器
@export var injector: InjectorComponent
@export var enhancement_type: String = "damage"
@export var enhancement_power: float = 1.5

func _ready():
    create_enhancement_components()
    
    # 手动触发注入（不通过碰撞）
    injector.willInject.connect(_on_enhancement_start)

func create_enhancement_components():
    match enhancement_type:
        "damage":
            var damage_modifier = StatModifierComponent.new()
            damage_modifier.allowNonEntityParent = true
            damage_modifier.statName = "damage"
            damage_modifier.modifierValue = enhancement_power
            damage_modifier.modifierType = StatModifierComponent.ModifierType.MULTIPLY
            injector.add_child(damage_modifier)
        
        "speed":
            var speed_modifier = StatModifierComponent.new()
            speed_modifier.allowNonEntityParent = true
            speed_modifier.statName = "speed"
            speed_modifier.modifierValue = enhancement_power
            speed_modifier.modifierType = StatModifierComponent.ModifierType.MULTIPLY
            injector.add_child(speed_modifier)
        
        "defense":
            var defense_modifier = StatModifierComponent.new()
            defense_modifier.allowNonEntityParent = true
            defense_modifier.statName = "defense"
            defense_modifier.modifierValue = enhancement_power
            defense_modifier.modifierType = StatModifierComponent.ModifierType.ADD
            injector.add_child(defense_modifier)

func apply_enhancement_to_entity(target_entity: Entity):
    injector.inject(target_entity)

func _on_enhancement_start(target: Entity):
    # 播放强化特效
    var enhancement_effect = preload("res://effects/EnhancementEffect.tscn").instantiate()
    target.add_child(enhancement_effect)
```

## 设计模式

### 效果工厂
```gdscript
# 效果组件工厂
class_name EffectFactory
extends Node

static func create_damage_over_time(damage: float, duration: float, interval: float = 1.0) -> Component:
    var component = preload("res://components/DamageOverTimeComponent.tscn").instantiate()
    component.allowNonEntityParent = true
    component.damage = damage
    component.duration = duration
    component.interval = interval
    return component

static func create_heal_over_time(heal: float, duration: float, interval: float = 2.0) -> Component:
    var component = preload("res://components/HealOverTimeComponent.tscn").instantiate()
    component.allowNonEntityParent = true
    component.healAmount = heal
    component.duration = duration
    component.interval = interval
    return component

static func create_stat_modifier(stat_name: String, value: float, type: int) -> Component:
    var component = StatModifierComponent.new()
    component.allowNonEntityParent = true
    component.statName = stat_name
    component.modifierValue = value
    component.modifierType = type
    return component
```

### 注入器管理器
```gdscript
# 全局注入器管理系统
class_name InjectorManager
extends Node

var active_injectors: Array[InjectorComponent] = []

func register_injector(injector: InjectorComponent):
    active_injectors.append(injector)

func create_temporary_effect(target: Entity, components: Array[Node], duration: float = 5.0):
    # 创建临时注入器
    var temp_injector = InjectorComponent.new()
    
    for component in components:
        temp_injector.add_child(component)
    
    # 立即注入
    temp_injector.inject(target)
    
    # 设置清理定时器
    if duration > 0:
        var timer = Timer.new()
        timer.wait_time = duration
        timer.one_shot = true
        timer.timeout.connect(func(): cleanup_temporary_effects(target, components))
        add_child(timer)
        timer.start()

func cleanup_temporary_effects(target: Entity, components: Array[Node]):
    for component in components:
        if is_instance_valid(component) and component.get_parent() == target:
            component.queue_free()
```

## 技术细节

### 节点转移过程
1. 检查是否启用且不在注入中
2. 遍历所有子节点
3. 使用reparent()转移节点
4. 设置新的owner
5. 应用可见性和处理模式设置

### 碰撞检测
- 支持Area2D和RigidBody2D碰撞
- 自动查找Entity父节点
- 需要手动连接碰撞信号

### 组件准备
- 子组件需要设置allowNonEntityParent = true
- 在转移前可设置为暂停状态
- 转移后恢复正常处理

## 注意事项

### 组件配置
- 确保子组件设置了allowNonEntityParent标志
- 避免子组件在转移前执行重要逻辑
- 考虑组件间的依赖关系

### 性能考虑
- 避免频繁的节点转移操作
- 大量子节点时考虑分批处理
- 注意内存管理和垃圾回收

### 碰撞设置
- 需要手动连接Area2D/Body信号
- 确保碰撞检测层和掩码正确
- 处理多重碰撞的情况

## 相关组件
- [`StatModifierComponent`](../Data/StatModifierComponent.md) - 统计修改器
- [`DamageOverTimeComponent`](../Combat/DamageOverTimeComponent.md) - 持续伤害
- [`Component`](../Component.md) - 组件基类 