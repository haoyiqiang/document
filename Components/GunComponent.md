# GunComponent API 文档

## 概述
`GunComponent` 实现射击武器功能，支持玩家控制和AI自动射击。

**重要**：此组件依赖 `_unhandled_input` 处理玩家输入，以允许UI元素接收输入而不触发射击。如果鼠标输入事件没有到达此组件，请检查覆盖节点的 `mouse_filter` 属性。

## 继承
`GunComponent` → `CooldownComponent` → `Component` → `Node`

## 主要特性
- 支持玩家控制和AI自动射击
- 弹药管理系统
- 冷却时间控制
- 灵活的子弹发射位置
- 实体速度继承
- 多种射击模式

## 导出属性

### 子弹设置

#### `bulletEntity: PackedScene`
射击时要实例化的实体。应该是一个Entity场景。

#### `bulletEmitter: Node2D`
新生成子弹的放置位置节点（如Marker2D）。
- 如果省略，使用组件内部的"BulletEmitter" Marker2D节点
- 子弹会添加到发射器的父节点作为子节点

#### `bulletPositionOffset: Vector2`
相对于 `bulletEmitter` 的调整位置偏移。

#### `bulletParentOverride: Node`
新子弹的可选父节点。
- 默认：子弹添加到实体的父节点或发射器的父节点

### 弹药系统

#### `ammo: Stat`
用作弹药的Stat资源。如果省略，不需要弹药即可开火。

#### `ammoCost: int = 0`
每发射击消耗的弹药。
- 0 = 无限弹药
- 负数会在射击时增加弹药

#### `ammoDepletedMessage: String = "AMMO DEPLETED"`
弹药用完时通过实体的LabelComponent显示的文本。

### 射击模式

#### `autoFire: bool = false`
是否自动射击，无需玩家输入。

#### `pressAgainToShoot: bool = false`
每发子弹是否需要重新按键。
- true：必须松开并重新按下按钮
- false：按住按钮持续射击

### 高级设置

#### `shouldAddEntityVelocity: bool = false` **（实验性）**
是否将父实体的速度添加到子弹。
- 需要 `CharacterBodyComponent`
- 子弹实体应有 `LinearMotionComponent`

#### `isPlayerControlled: bool = true`
是否接受玩家输入。禁用用于AI控制的敌人。

#### `isEnabled: bool = true`
是否启用组件。

## 信号

### `didFire(bullet: Entity)`
射击成功时发出，传递新创建的子弹实体。

### `didDepleteAmmo`
射击后弹药降到1以下时发出。

### `ammoInsufficient`
弹药不足时尝试射击会发出。

## 状态属性

### `isFireActionPressed: bool`
射击动作是否当前被按下。

### `wasFireActionJustPressed: bool`
射击动作是否刚刚被按下。

## 依赖组件

### `characterBodyComponent: CharacterBodyComponent`
当 `shouldAddEntityVelocity` 为true时需要。

### `labelComponent: LabelComponent`
用于显示弹药耗尽消息。

## 主要方法

### 射击系统

#### `fire(ignoreCooldown: bool = false) -> Entity`
发射子弹。

**参数**：
- `ignoreCooldown: bool = false` - 是否忽略冷却时间

**返回**：发射的子弹实体，失败返回null

**行为**：
1. 检查启用状态和冷却时间
2. 验证子弹实体配置
3. 创建新子弹
4. 检查并消耗弹药
5. 将子弹添加到场景
6. 发出信号并启动冷却

### 弹药管理

#### `useAmmo() -> bool`
扣除弹药并检查是否足够。

**返回**：是否有足够弹药

**行为**：
- 无弹药资源时总是返回true
- 检查弹药是否足够
- 扣除弹药成本
- 弹药耗尽时发出信号和显示消息

## 使用示例

### 基本玩家枪械
```gdscript
# 设置枪械组件
var gunComponent = GunComponent.new()
playerEntity.add_child(gunComponent)

# 配置基本属性
gunComponent.bulletEntity = preload("res://Entities/Bullet.tscn")
gunComponent.cooldownTime = 0.5  # 0.5秒冷却
gunComponent.isPlayerControlled = true

# 设置弹药
var ammoStat = Stat.new()
ammoStat.max = 30
ammoStat.value = 30
gunComponent.ammo = ammoStat
gunComponent.ammoCost = 1
```

### 机关枪设置
```gdscript
# 快速连射武器
gunComponent.cooldownTime = 0.1  # 快速射击
gunComponent.pressAgainToShoot = false  # 按住连射
gunComponent.ammoCost = 1
gunComponent.ammoDepletedMessage = "RELOAD!"
```

### AI敌人武器
```gdscript
# 敌人自动射击
gunComponent.autoFire = true
gunComponent.isPlayerControlled = false
gunComponent.cooldownTime = 2.0  # 每2秒射击一次
gunComponent.ammoCost = 0  # 无限弹药
```

### 霰弹枪模式
```gdscript
# 多发子弹（需要在fire()中手动实现）
func fireMultipleBullets():
    for i in range(5):  # 发射5发
        var bullet = gunComponent.fire(true)  # 忽略冷却
        if bullet:
            # 设置不同角度
            bullet.rotation_degrees += (i - 2) * 15
```

### 带速度继承的设置
```gdscript
# 子弹继承实体速度
gunComponent.shouldAddEntityVelocity = true
# 需要确保实体有CharacterBodyComponent
# 子弹需要有LinearMotionComponent
```

### 信号连接
```gdscript
# 连接射击事件
gunComponent.didFire.connect(_on_gun_fired)
gunComponent.didDepleteAmmo.connect(_on_ammo_depleted)
gunComponent.ammoInsufficient.connect(_on_ammo_insufficient)

func _on_gun_fired(bullet: Entity):
    print("射击！子弹：", bullet.name)
    # 播放射击音效
    AudioManager.playSound("gunshot")

func _on_ammo_depleted():
    print("弹药用完！")
    # 播放空膛音效
    AudioManager.playSound("empty_clip")

func _on_ammo_insufficient():
    print("弹药不足！")
    # 播放点击音效
    AudioManager.playSound("click")
```

### 自定义发射位置
```gdscript
# 使用自定义发射器
var customEmitter = Marker2D.new()
entity.add_child(customEmitter)
customEmitter.position = Vector2(50, 0)  # 武器前端
gunComponent.bulletEmitter = customEmitter

# 设置位置偏移
gunComponent.bulletPositionOffset = Vector2(10, 5)
```

## 输入系统

### 输入处理流程
1. `_unhandled_input()` 检测射击动作
2. `_process()` 根据射击模式处理射击逻辑
3. UI元素可以拦截输入，防止意外射击

### 射击模式判断
```gdscript
# 在_process中的射击逻辑
if autoFire and hasCooldownCompleted:
    fire()
elif not pressAgainToShoot and isFireActionPressed:
    fire()  # 按住连射
elif pressAgainToShoot and wasFireActionJustPressed:
    fire()  # 单发模式
```

## 性能优化

1. **条件处理**：根据模式启用/禁用输入处理
2. **弹药检查**：无弹药限制时跳过检查
3. **子弹父节点**：避免不必要的函数调用
4. **冷却管理**：继承自CooldownComponent的优化机制

## 场景结构建议

```
GunComponent (Node2D)
├── GunSprite (Sprite2D)          # 武器外观
├── BulletEmitter (Marker2D)      # 内置发射点
└── Timer (Timer)                 # 冷却计时器（继承自CooldownComponent）
```

## 注意事项

1. **输入过滤**：确保UI的mouse_filter设置正确
2. **弹药共享**：与StatsComponent共用弹药时使用相同Stat资源
3. **场景要求**：GunComponent需要是Node2D而非Node
4. **AI控制**：禁用玩家控制用于敌人
5. **速度继承**：需要正确的组件依赖

## 相关组件
- `CooldownComponent` - 冷却时间基类
- `CharacterBodyComponent` - 速度继承需要
- `LabelComponent` - 消息显示
- `LinearMotionComponent` - 子弹运动
- `DamageComponent` - 子弹伤害 