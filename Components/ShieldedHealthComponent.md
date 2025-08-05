# ShieldedHealthComponent

## 概述
`ShieldedHealthComponent` 是 `HealthComponent` 的子类，实现了护盾/护甲系统。护盾会在生命值受到伤害前吸收伤害，提供额外的防护层。

**继承关系：** `ShieldedHealthComponent` → `HealthComponent` → `Component` → `Node`

## 主要特性
- 🛡️ 护盾优先吸收伤害
- ⚡ 护盾充能系统
- 🔄 可选的护盾满时治疗
- 🎮 动态启用/禁用护盾
- 📡 丰富的护盾状态信号
- 🎯 护盾状态查询

## 导出属性

### `shield: Stat`
护盾统计数据，在生命值受损前吸收伤害。

**类型：** `Stat`  
**用途：** 定义护盾的当前值、最大值等属性  
**特性：** 连接到 `changed` 信号以监听状态变化

### `shouldHealOnMaxRecharge: bool = false`
当护盾达到最大值且组件启用时，是否将充能量转换为生命值治疗。

**类型：** `bool`  
**默认值：** `false`  
**注意：** 不处理"剩余"充能值（如护盾9/10时充能100，只会恢复到10，不会治疗）

### `isShieldEnabled: bool = true`
护盾系统是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**影响：** 
- `false` 时所有伤害直接作用于生命值
- `false` 时禁用充能功能

## 状态属性

### `isProtecting: bool` (只读)
护盾是否正在提供保护。

**条件：** `isShieldEnabled && shield.value > 0`  
**用途：** 快速判断当前是否受护盾保护

## 信号

### `shieldDidDecrease(difference: int)`
护盾值减少时触发。

**参数：** `difference` - 变化量（负数）  
**用途：** 监听护盾受损事件，可与 `shieldDidIncrease` 连接同一处理函数

### `shieldDidIncrease(difference: int)`
护盾值增加时触发。

**参数：** `difference` - 变化量（正数）  
**用途：** 监听护盾恢复事件

### `shieldDidZero()`
护盾值降至0或以下时触发。

**用途：** 处理护盾破碎事件，如播放特效或音效

### `shieldDidMax()`
护盾值达到最大值时触发。

**用途：** 处理护盾充满事件

## 核心方法

### `damage(damageAmount: int) -> int`
对实体造成伤害，优先消耗护盾。

**参数：** `damageAmount` - 伤害量（必须为正数）  
**返回：** 剩余护盾值  
**逻辑：**
- 护盾启用且有值时：消耗护盾
- 护盾耗尽或禁用时：伤害生命值

### `recharge(rechargeAmount: int) -> int`
为护盾充能。

**参数：** `rechargeAmount` - 充能量（必须为正数）  
**返回：** 剩余护盾值  
**特性：**
- 护盾禁用时不执行任何操作
- 护盾满时可选择治疗生命值
- 不处理超出最大值的"剩余"充能

## 使用示例

### 基本护盾设置
```gdscript
# 获取ShieldedHealthComponent
var shieldedHealth = $ShieldedHealthComponent

# 设置护盾资源
var shieldStat = load("res://Resources/Stats/ShieldStat.tres")
shieldedHealth.shield = shieldStat

# 连接护盾信号
shieldedHealth.shieldDidZero.connect(_on_shield_broken)
shieldedHealth.shieldDidMax.connect(_on_shield_full)
```

### 伤害处理
```gdscript
# 造成伤害
func dealDamageToEntity(entity: Entity, damage: int):
    var health = entity.findFirstComponent(ShieldedHealthComponent)
    if health:
        var remainingShield = health.damage(damage)
        print("剩余护盾: ", remainingShield)
```

### 护盾充能系统
```gdscript
# 护盾充能站
func _on_shield_station_used():
    var player = $Player
    var shieldedHealth = player.findFirstComponent(ShieldedHealthComponent)
    
    if shieldedHealth:
        var chargedAmount = shieldedHealth.recharge(50)
        print("护盾充能至: ", chargedAmount)

# 带治疗的充能
func setupAdvancedShield():
    var shieldedHealth = $ShieldedHealthComponent
    # 护盾满时继续充能会治疗生命值
    shieldedHealth.shouldHealOnMaxRecharge = true
```

### 动态护盾控制
```gdscript
# EMP攻击暂时禁用护盾
func empAttack():
    var shieldedHealth = $ShieldedHealthComponent
    shieldedHealth.isShieldEnabled = false
    
    # 3秒后恢复护盾
    await get_tree().create_timer(3.0).timeout
    shieldedHealth.isShieldEnabled = true
    print("护盾系统重新上线")

# 检查护盾状态
func checkShieldStatus():
    var shieldedHealth = $ShieldedHealthComponent
    if shieldedHealth.isProtecting:
        print("受护盾保护")
    else:
        print("护盾失效")
```

### 护盾信号处理
```gdscript
func _ready():
    var shieldedHealth = $ShieldedHealthComponent
    
    # 护盾破碎效果
    shieldedHealth.shieldDidZero.connect(func():
        # 播放破碎音效
        $AudioPlayer.play()
        # 显示警告UI
        $UI/ShieldWarning.show()
    )
    
    # 护盾充满效果
    shieldedHealth.shieldDidMax.connect(func():
        # 播放充满音效
        $ShieldFullSound.play()
        # 隐藏充能UI
        $UI/ChargingIndicator.hide()
    )
    
    # 护盾变化通用处理
    var updateShieldUI = func(difference: int):
        $UI/ShieldBar.value = shieldedHealth.shield.value
        $UI/ShieldBar.max_value = shieldedHealth.shield.max
    
    shieldedHealth.shieldDidIncrease.connect(updateShieldUI)
    shieldedHealth.shieldDidDecrease.connect(updateShieldUI)
```

### 与其他系统集成
```gdscript
# 与武器系统集成
func _on_weapon_fired():
    var shieldedHealth = $ShieldedHealthComponent
    # 开火时消耗少量护盾（能量武器设定）
    if shieldedHealth.isProtecting:
        shieldedHealth.damage(2)

# 与升级系统集成
func upgradeShield(level: int):
    var shieldedHealth = $ShieldedHealthComponent
    # 提升护盾最大值
    shieldedHealth.shield.max += level * 20
    # 立即充满护盾
    shieldedHealth.shield.value = shieldedHealth.shield.max
```

## 设计模式

### 分层防护系统
护盾系统实现了经典的分层防护模式：
1. **第一层：** 护盾吸收伤害
2. **第二层：** 生命值承受伤害

### 状态管理
- **护盾启用状态：** 控制整个系统开关
- **护盾值状态：** 实时反映保护能力
- **组合状态：** `isProtecting` 提供便捷查询

### 信号驱动架构
所有状态变化都通过信号通知，支持：
- UI更新
- 音效播放
- 视觉特效
- 游戏逻辑响应

## 注意事项

1. **参数验证：** 伤害和充能参数必须为正数
2. **护盾禁用：** 禁用时充能不起作用
3. **剩余处理：** 充能时不处理超出最大值的部分
4. **信号连接：** 确保正确连接护盾状态信号
5. **继承行为：** 保留所有HealthComponent的功能

## 相关组件
- `HealthComponent` - 父类，管理生命值
- `DamageComponent` - 造成伤害的组件
- `DamageReceivingComponent` - 接收伤害的组件
- `StatModifierComponent` - 可用于护盾再生
- `FactionComponent` - 决定是否应受到伤害