# CornerCollisionComponent API

> **继承关系**: Component > CornerCollisionComponent  
> **物理类型**: 角落碰撞检测

在实体Sprite2D的四个角落放置Area2D节点，帮助其他组件快速检测特定方向的地板、墙壁和天花板等。默认碰撞掩码设置为仅检测地形。

## ✨ 主要特性

- 🎯 四角精确碰撞检测
- 🧭 方向性碰撞状态（上下左右）
- 📐 自动适配Sprite2D尺寸
- 🔧 可编辑的子节点Area2D
- ⚡ 可启用/禁用功能

## 📊 导出属性

### 检测设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `fitAreasAutomatically` | `bool` | `true` | 是否自动适配Area2D位置到Sprite2D角落 |
| `isEnabled` | `bool` | `true` | 是否启用碰撞检测 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `Sprite2D` | **父实体** | 用于确定角落位置 |

### 子节点结构
```
CornerCollisionComponent
└── Areas (Node2D)
    ├── AreaNW (Area2D) - 西北角
    ├── AreaNE (Area2D) - 东北角
    ├── AreaSE (Area2D) - 东南角
    └── AreaSW (Area2D) - 西南角
```

## 📊 状态属性

### 碰撞计数
| 属性 | 类型 | 描述 |
|------|------|------|
| `areaNWCollisionCount` | `int` | 西北角碰撞数量 |
| `areaNECollisionCount` | `int` | 东北角碰撞数量 |
| `areaSECollisionCount` | `int` | 东南角碰撞数量 |
| `areaSWCollisionCount` | `int` | 西南角碰撞数量 |

### 方向状态
| 属性 | 类型 | 描述 |
|------|------|------|
| `isCollidingOnLeft` | `bool` | 是否与左侧碰撞 |
| `isCollidingOnRight` | `bool` | 是否与右侧碰撞 |
| `isCollidingOnTop` | `bool` | 是否与顶部碰撞 |
| `isCollidingOnBottom` | `bool` | 是否与底部碰撞 |

## 🎯 使用示例

### 基础角落检测

```gdscript
# Entity Scene Structure:
# └── Entity (CharacterBody2D)
#     ├── Sprite2D
#     └── CornerCollisionComponent (可编辑子节点)
#         └── Areas
#             ├── AreaNW
#             ├── AreaNE
#             ├── AreaSE
#             └── AreaSW
```

### 平台跳跃检测

```gdscript
# PlatformJumpDetector.gd
extends Node

@export var cornerCollision: CornerCollisionComponent
@export var jumpComponent: JumpComponent

func _ready():
    if not cornerCollision:
        cornerCollision = get_parent().findFirstComponentSubclass(CornerCollisionComponent)

func _physics_process(_delta: float):
    checkJumpConditions()

func checkJumpConditions():
    if not cornerCollision:
        return
    
    # 检查是否站在地面上
    if cornerCollision.isCollidingOnBottom:
        jumpComponent.canJump = true
    
    # 检查头部碰撞（停止上升）
    if cornerCollision.isCollidingOnTop:
        var body = get_parent().body
        if body.velocity.y < 0:  # 正在上升
            body.velocity.y = 0
    
    # 检查墙壁滑动
    if cornerCollision.isCollidingOnLeft or cornerCollision.isCollidingOnRight:
        handleWallSliding()

func handleWallSliding():
    # 实现墙壁滑动逻辑
    pass
```

### 智能AI导航

```gdscript
# CornerBasedAI.gd
extends Node

@export var cornerCollision: CornerCollisionComponent
@export var movementComponent: LinearMotionComponent
@export var turnAroundOnWall: bool = true

func _ready():
    if cornerCollision:
        # 连接碰撞事件
        cornerCollision.onAreaEntered.connect(_on_area_entered)

func _physics_process(_delta: float):
    updateAIBehavior()

func updateAIBehavior():
    if not cornerCollision:
        return
    
    # 检测前方墙壁
    var facingRight = movementComponent.direction.x > 0
    var hitWall = false
    
    if facingRight and cornerCollision.isCollidingOnRight:
        hitWall = true
    elif not facingRight and cornerCollision.isCollidingOnLeft:
        hitWall = true
    
    # 转身逻辑
    if hitWall and turnAroundOnWall:
        movementComponent.direction.x *= -1
    
    # 检查悬崖
    checkCliffDetection(facingRight)

func checkCliffDetection(facingRight: bool):
    # 检查脚下是否有地面
    var frontGroundCheck = false
    
    if facingRight:
        frontGroundCheck = cornerCollision.areaSECollisionCount > 0
    else:
        frontGroundCheck = cornerCollision.areaSWCollisionCount > 0
    
    # 如果前方没有地面，停止移动或转身
    if not frontGroundCheck:
        if turnAroundOnWall:
            movementComponent.direction.x *= -1
        else:
            movementComponent.direction.x = 0

func _on_area_entered(area: Area2D):
    print("Corner collision detected: ", area.name)
```

### 环境交互检测

```gdscript
# EnvironmentInteraction.gd
extends Node

@export var cornerCollision: CornerCollisionComponent

func _ready():
    if cornerCollision:
        setupCollisionSignals()

func setupCollisionSignals():
    cornerCollision.areaNW.body_entered.connect(_on_corner_collision.bind("NW"))
    cornerCollision.areaNE.body_entered.connect(_on_corner_collision.bind("NE"))
    cornerCollision.areaSE.body_entered.connect(_on_corner_collision.bind("SE"))
    cornerCollision.areaSW.body_entered.connect(_on_corner_collision.bind("SW"))

func _on_corner_collision(corner: String, body: Node2D):
    print("Corner ", corner, " hit: ", body.name)
    
    # 检查不同类型的环境
    if body.is_in_group("water"):
        handleWaterCollision(corner)
    elif body.is_in_group("spikes"):
        handleSpikeCollision(corner)
    elif body.is_in_group("ladders"):
        handleLadderCollision(corner)

func handleWaterCollision(corner: String):
    match corner:
        "SW", "SE":  # 脚部接触水
            print("Entered water")
            # 播放水花效果
        "NW", "NE":  # 头部接触水
            print("Swimming underwater")

func handleSpikeCollision(corner: String):
    # 尖刺伤害逻辑
    var healthComponent = get_parent().findFirstComponentSubclass(HealthComponent)
    if healthComponent:
        healthComponent.takeDamage(10)

func handleLadderCollision(corner: String):
    # 梯子攀爬逻辑
    var climbComponent = get_parent().findFirstComponentSubclass(ClimbComponent)
    if climbComponent:
        climbComponent.enableClimbing()
```

### 自定义碰撞过滤

```gdscript
# CustomCornerCollision.gd
extends CornerCollisionComponent

@export var detectOnlyTerrain: bool = true
@export var detectEnemies: bool = false
@export var detectItems: bool = false

func _ready():
    super._ready()
    setupCollisionLayers()

func setupCollisionLayers():
    var terrainLayer = 1 << 0   # 第1层
    var enemyLayer = 1 << 1     # 第2层
    var itemLayer = 1 << 2      # 第3层
    
    var maskToSet = 0
    
    if detectOnlyTerrain:
        maskToSet |= terrainLayer
    if detectEnemies:
        maskToSet |= enemyLayer
    if detectItems:
        maskToSet |= itemLayer
    
    # 设置所有角落Area2D的碰撞掩码
    for area in [areaNW, areaNE, areaSE, areaSW]:
        area.collision_mask = maskToSet

func updateFlags():
    super.updateFlags()
    
    # 添加自定义的碰撞类型检测
    checkSpecialCollisions()

func checkSpecialCollisions():
    if detectEnemies:
        checkEnemyCollisions()
    if detectItems:
        checkItemCollisions()

func checkEnemyCollisions():
    var enemyCount = 0
    for area in [areaNW, areaNE, areaSE, areaSW]:
        for body in area.get_overlapping_bodies():
            if body.is_in_group("enemies"):
                enemyCount += 1
    
    if enemyCount > 0:
        print("Enemy contact detected!")

func checkItemCollisions():
    for area in [areaNW, areaNE, areaSE, areaSW]:
        for area_overlapping in area.get_overlapping_areas():
            if area_overlapping.is_in_group("items"):
                print("Item nearby: ", area_overlapping.name)
```

## 🔧 技术细节

### 角落位置设置
```gdscript
func setAreaPositions() -> void:
    var spriteRect: Rect2 = sprite.get_rect()
    
    areaNW.position = Tools.getRectCorner(spriteRect, Tools.CompassVectors.northWest)
    areaNE.position = Tools.getRectCorner(spriteRect, Tools.CompassVectors.northEast)
    areaSE.position = Tools.getRectCorner(spriteRect, Tools.CompassVectors.southEast)
    areaSW.position = Tools.getRectCorner(spriteRect, Tools.CompassVectors.southWest)
```

### 方向状态更新
```gdscript
func updateFlags() -> void:
    updateCollisionCount()
    isCollidingOnLeft   = (areaNWCollisionCount >= 1) or (areaSWCollisionCount >= 1)
    isCollidingOnRight  = (areaNECollisionCount >= 1) or (areaSECollisionCount >= 1)
    isCollidingOnTop    = (areaNWCollisionCount >= 1) or (areaNECollisionCount >= 1)
    isCollidingOnBottom = (areaSWCollisionCount >= 1) or (areaSECollisionCount >= 1)
```

## ⚠️ 注意事项

1. **Sprite2D依赖**: 需要父实体包含Sprite2D确定角落位置
2. **可编辑子节点**: Area2D节点可在编辑器中调整
3. **碰撞掩码**: 默认设置为仅检测地形层
4. **性能考虑**: 每个角落的碰撞检测都会影响性能
5. **坐标系统**: 使用Sprite2D的本地坐标系

## 🔗 相关组件

- [AreaCollisionComponent](AreaCollisionComponent.md) - Area碰撞检测组件
- [PlatformerPhysicsComponent](PlatformerPhysicsComponent.md) - 平台物理组件
- [JumpComponent](./JumpComponent.md) - 跳跃组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 