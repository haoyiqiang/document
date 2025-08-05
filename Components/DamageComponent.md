# DamageComponent API 文档

## 概述
`DamageComponent` 当此组件的Area2D"攻击框"与 `DamageReceivingComponent` 的"受击框"碰撞时，对另一个实体造成伤害，然后传递给受害者实体的 `HealthComponent`。

如果两个实体都有 `FactionComponent`，则只有当实体不共享任何阵营时才会造成伤害。如果缺少 `FactionComponent`，则总是造成伤害。

## 继承
`DamageComponent` → `Component` → `Node`

## 要求
此组件必须是代表"攻击框"的Area2D。

## 主要特性
- 基于Area2D碰撞的伤害系统
- 支持阵营检查
- 命中率和闪避机制
- 持续伤害支持
- 灵活的伤害修正器

## 导出属性

### 伤害设置

#### `damageOnCollision: int = 1`
首次碰撞时造成的伤害量。
- 适用于子弹等碰撞后消失的对象
- 可设为0，让其他脚本处理信号并应用伤害
- **重要**：使用 `damageOnCollisionWithModifier` 获取包含修正器的实际伤害值

#### `damageModifier: Stat` **（实验性）**
可选的伤害修正器Stat。
- Stat的值会加到或减去基础 `damageOnCollision`
- 允许升级、增益或减益轻松增加/减少攻击力
- `damageOnCollision` 可设为0来仅使用Stat作为基础伤害值

#### `damagePerSecond: float = 0` **（实验性）**
持续伤害值。
- 适用于怪物或危险物等在场景中保持的对象
- 可能在首次碰撞的同一帧造成持续伤害

### 命中机制

#### `hitChance: int = 100`
命中率百分比。
- 小于100时，与 `DamageReceivingComponent` 的碰撞可能偶尔被忽略
- 最终命中率 = `hitChance` - `DamageReceivingComponent.missChance`
- 可设置大于100来确保角色总是命中

#### `friendlyFire: bool = false`
同阵营是否互相伤害。

#### `shouldEmitBubbleOnMiss: bool = true`
命中率失败时是否显示"MISS"文本。

### 实体移除设置

#### `removeEntityOnCollisionWithReceiver: bool = false`
与 `DamageReceivingComponent` 碰撞时是否移除父实体。
- 适用于被所有接收者阻挡的"子弹"实体
- **警告**：即使没有实际伤害也会执行
- **提示**：仅在实际造成伤害时移除，使用 `removeEntityOnApplyingDamage`

#### `removeEntityOnApplyingDamage: bool = false`
对敌对阵营造成伤害时是否移除父实体。
- 适用于不应被发射者阻挡的"子弹"实体
- 无敌对阵营或友军伤害时忽略
- **重要**：不要对持续性"危险物"如尖刺或酸池设置为true

#### `isEnabled: bool = true`
是否启用组件。同时影响Area2D的监控状态。

## 核心属性

### `initiatorEntity: Entity` **（实验性）**
"发起"伤害或发射子弹的"攻击者"实体。
- 可能是玩家实体、怪物或酸池等"危险物"
- 对于"子弹"实体，这不是子弹实体本身，而是发射子弹的实体
- 如果在 `_ready` 时为null，会设置为此组件的 `parentEntity`

### `damageReceivingComponentsInContact: Array[DamageReceivingComponent]`
当前在碰撞接触中的DamageReceivingComponent列表。

### `damageOnCollisionWithModifier: int`
包含基础伤害和修正器的总伤害值（只读）。

### `area: Area2D`
此组件代表的"攻击框"Area2D。

## 信号

### 碰撞事件
- `didCollideReceiver(damageReceivingComponent: DamageReceivingComponent)` - 与接收器碰撞时发出
- `didLeaveReceiver(damageReceivingComponent: DamageReceivingComponent)` - 离开接收器时发出

### 命中判定
- `willCalculateChance(damageReceivingComponent: DamageReceivingComponent)` - 计算命中率前发出
- `didSucceed(damageReceivingComponent: DamageReceivingComponent, totalChance: int, roll: int)` - 命中成功时发出
- `didMiss(damageReceivingComponent: DamageReceivingComponent, totalChance: int, roll: int)` - 命中失败时发出

## 主要方法

### 碰撞处理

#### `onAreaEntered(areaEntered: Area2D) -> void`
Area2D进入时的处理方法。
- 检查是否为DamageReceivingComponent
- 避免自我碰撞
- 触发伤害计算和实体移除逻辑

#### `onAreaExited(areaExited: Area2D) -> void`
Area2D离开时的处理方法。
- 从接触列表中移除
- 发出离开信号

#### `getDamageReceivingComponent(collidingArea: Area2D) -> DamageReceivingComponent`
将Area2D转换为DamageReceivingComponent。
- 如果无法转换返回null
- 检查是否为自身实体

#### `causeCollisionDamage(damageReceivingComponent: DamageReceivingComponent) -> void`
调用DamageReceivingComponent.processCollision来造成伤害。

## 使用示例

### 基本子弹设置
```gdscript
# 设置子弹伤害组件
var damageComponent = DamageComponent.new()
bulletEntity.add_child(damageComponent)

# 配置基本伤害
damageComponent.damageOnCollision = 25
damageComponent.hitChance = 95
damageComponent.removeEntityOnApplyingDamage = true

# 连接碰撞信号
damageComponent.didCollideReceiver.connect(_on_bullet_hit)
```

### 带修正器的设置
```gdscript
# 创建伤害修正器
var damageBuff = Stat.new()
damageBuff.value = 10

# 应用修正器
damageComponent.damageOnCollision = 20
damageComponent.damageModifier = damageBuff
# 实际伤害 = 20 + 10 = 30
```

### 持续伤害危险物
```gdscript
# 设置酸池等持续伤害
damageComponent.damageOnCollision = 0  # 无首次伤害
damageComponent.damagePerSecond = 5.0  # 每秒5点伤害
damageComponent.removeEntityOnCollisionWithReceiver = false  # 不移除
damageComponent.removeEntityOnApplyingDamage = false  # 不移除
```

### 命中率设置
```gdscript
# 设置命中率机制
damageComponent.hitChance = 80  # 80%命中率
damageComponent.shouldEmitBubbleOnMiss = true  # 显示MISS

# 连接命中事件
damageComponent.didSucceed.connect(_on_hit_success)
damageComponent.didMiss.connect(_on_hit_miss)

func _on_hit_success(receiver, chance, roll):
    print("命中成功！概率:", chance, "%, 骰子:", roll)

func _on_hit_miss(receiver, chance, roll):
    print("攻击落空！概率:", chance, "%, 骰子:", roll)
```

### 阵营友军伤害
```gdscript
# 允许友军伤害（如手榴弹）
damageComponent.friendlyFire = true

# 禁用友军伤害（默认）
damageComponent.friendlyFire = false
```

## 物理设置

### 碰撞层级配置
```gdscript
# 玩家武器设置
# collision_layer = "players" 
# collision_mask = "enemies"

# 敌人武器设置  
# collision_layer = "enemies"
# collision_mask = "players"

# 危险物设置
# collision_layer = "hazards"
# collision_mask = "players | enemies"
```

## 设计模式

### 主动vs被动伤害系统
- **DamageComponent**：主动对象，发起战斗并调用伤害处理代码
- **DamageReceivingComponent**：被动对象，接收和处理伤害

### 实体移除策略
1. **总是移除**：`removeEntityOnCollisionWithReceiver = true`
2. **仅敌对移除**：`removeEntityOnApplyingDamage = true`
3. **永不移除**：两者都为false（危险物）

## 注意事项

1. **物理层级**：正确设置collision_layer和collision_mask
2. **性能考虑**：适当使用isEnabled来控制处理
3. **自我碰撞**：系统自动防止自我伤害
4. **信号延迟**：某些移除操作使用call_deferred避免物理回调错误
5. **伤害确认**：移除实体不保证目标血量实际减少

## 相关组件
- `DamageReceivingComponent` - 伤害接收组件
- `HealthComponent` - 生命值组件
- `FactionComponent` - 阵营组件
- `DamageRepeatingComponent` - 重复伤害组件
- `DamageOverTimeComponent` - 持续伤害组件 