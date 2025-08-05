# MouseRotationComponent

## 概述
`MouseRotationComponent` 使实体或指定节点旋转以面向鼠标指针。常用于瞄准系统、武器定向等需要精确鼠标控制的场景。

**继承关系：** `MouseRotationComponent` → `Component` → `Node`

## 主要特性
- 🎯 精确的鼠标跟踪旋转
- ⚙️ 可配置旋转速度和模式
- 🎮 与手柄控制兼容
- 🖱️ 自定义鼠标指针
- 🔄 动态启用/禁用
- 📊 旋转状态检测

## 导出属性

### `nodeToRotate: Node2D = null`
要旋转的目标节点，默认为父实体。

**类型：** `Node2D`  
**默认值：** `null` (使用parent Entity)  
**用途：** 允许旋转特定子节点，如武器组件

### `rotationSpeed: float = 5.0`
旋转速度系数。

**类型：** `float`  
**范围：** `0.1 - 20.0`  
**默认值：** `5.0`  
**影响：** 仅在非瞬时旋转模式下有效

### `shouldRotateInstantly: bool = false`
是否瞬时旋转到鼠标方向。

**类型：** `bool`  
**默认值：** `false`  
**行为：**
- `true`: 立即对准鼠标位置
- `false`: 平滑旋转到目标方向

### `targetingCursor: Texture2D`
启用时显示的自定义鼠标指针纹理。

**类型：** `Texture2D`  
**用途：** 提供视觉反馈表示瞄准模式

### `shouldDisableOnTurningInput: bool = true`
是否在接收转向输入时自动禁用组件。

**类型：** `bool`  
**默认值：** `true`  
**功能：** 允许鼠标和手柄控制共存但不冲突

### `isEnabled: bool = true`
组件是否启用的标志。

**类型：** `bool`  
**默认值：** `true`  
**特性：** 自动控制 `set_physics_process()` 状态

## 状态属性

### `previousRotation: float`
上一帧的全局旋转角度。

**用途：** 用于计算旋转变化

### `didRotateThisFrame: bool`
当前帧是否发生了旋转。

**用途：** 供其他组件检测旋转状态

### `haveTurningControlComponent: bool` (只读)
实体是否具有TurningControlComponent。

**用途：** 检测控制冲突

## 依赖组件

### `inputComponent: InputComponent` (可选)
用于解决与TurningControlComponent的冲突。

**类型：** `InputComponent` (可选依赖)  
**用途：** 监听转向输入事件

## 核心方法

### `setMouseCursor(useTargetingCursor: bool = isEnabled) -> void`
设置鼠标指针样式。

**参数：** `useTargetingCursor` - 是否使用瞄准指针  
**功能：**
- 启用时显示自定义指针
- 禁用时恢复默认箭头

### `oninputComponent_didProcessInput(event: InputEvent) -> void`
处理输入事件以管理与手柄控制的兼容性。

**参数：** `event` - 输入事件  
**逻辑：**
- 转向输入时禁用鼠标控制
- 鼠标按键时重新启用

## 使用示例

### 基本武器瞄准
```gdscript
# 设置武器瞄准
func setupWeaponAiming():
    var mouseRotation = $MouseRotationComponent
    var gunNode = $GunComponent
    
    # 指定旋转武器而非整个实体
    mouseRotation.nodeToRotate = gunNode
    
    # 设置瞄准指针
    mouseRotation.targetingCursor = load("res://UI/Cursors/Crosshair.png")
    
    # 瞬时瞄准以提高精度
    mouseRotation.shouldRotateInstantly = true
```

### 平滑旋转控制
```gdscript
# 设置平滑旋转（如坦克炮塔）
func setupTurretRotation():
    var mouseRotation = $MouseRotationComponent
    
    # 较慢的旋转速度模拟重型装备
    mouseRotation.rotationSpeed = 2.0
    mouseRotation.shouldRotateInstantly = false
    
    # 旋转炮塔节点
    mouseRotation.nodeToRotate = $TurretNode
```

### 动态控制系统
```gdscript
# 切换瞄准模式
func toggleAimingMode(enabled: bool):
    var mouseRotation = $MouseRotationComponent
    mouseRotation.isEnabled = enabled
    
    if enabled:
        print("瞄准模式启用")
        $UI/AimingUI.show()
    else:
        print("瞄准模式禁用")
        $UI/AimingUI.hide()

# 检测旋转状态
func _process(delta):
    var mouseRotation = $MouseRotationComponent
    if mouseRotation.didRotateThisFrame:
        # 更新瞄准UI或触发音效
        updateAimingIndicator()
```

### 混合控制支持
```gdscript
# 支持鼠标和手柄共存
func _ready():
    var mouseRotation = $MouseRotationComponent
    
    # 启用控制冲突管理
    mouseRotation.shouldDisableOnTurningInput = true
    
    # 监听控制切换
    if mouseRotation.inputComponent:
        mouseRotation.inputComponent.didProcessInput.connect(_on_input_changed)

func _on_input_changed(event: InputEvent):
    # 自定义控制切换逻辑
    if event is InputEventJoypadMotion:
        print("手柄控制激活")
    elif event is InputEventMouse:
        print("鼠标控制激活")
```

### 高级瞄准系统
```gdscript
# 瞄准辅助功能
func setupAdvancedAiming():
    var mouseRotation = $MouseRotationComponent
    
    # 根据距离调整旋转速度
    func adjustAimingSpeed():
        var mousePos = get_global_mouse_position()
        var distance = global_position.distance_to(mousePos)
        
        # 远距离时提高精度
        if distance > 500:
            mouseRotation.rotationSpeed = 8.0
        else:
            mouseRotation.rotationSpeed = 3.0
    
    # 每帧调整
    adjustAimingSpeed()

# 瞄准预测
func setupPredictiveAiming():
    var mouseRotation = $MouseRotationComponent
    
    # 为移动目标添加预测偏移
    func calculateLeadTarget(targetVelocity: Vector2) -> Vector2:
        var mousePos = get_global_mouse_position()
        var distance = global_position.distance_to(mousePos)
        var timeToTarget = distance / 1000.0  # 假设弹速
        
        return mousePos + targetVelocity * timeToTarget
```

### 与其他组件集成
```gdscript
# 与GunComponent集成
func _ready():
    var mouseRotation = $MouseRotationComponent
    var gunComponent = $GunComponent
    
    # 武器开火时短暂锁定瞄准
    gunComponent.didFire.connect(func():
        mouseRotation.isEnabled = false
        await get_tree().create_timer(0.1).timeout
        mouseRotation.isEnabled = true
    )

# 与CameraComponent集成
func setupCameraFollowAiming():
    var mouseRotation = $MouseRotationComponent
    var camera = $CameraComponent
    
    # 相机跟随瞄准方向
    if mouseRotation.didRotateThisFrame:
        var aimDirection = global_position.direction_to(get_global_mouse_position())
        camera.offset = aimDirection * 100  # 轻微偏移
```

## 设计模式

### 控制冲突解决
- **互斥控制：** 鼠标与手柄控制不会同时激活
- **智能切换：** 根据输入类型自动切换控制模式
- **用户反馈：** 显示当前激活的控制方式

### 性能优化
- **条件处理：** 只在启用时运行physics_process
- **变化检测：** 跟踪旋转变化以优化其他系统
- **缓存计算：** 避免重复的角度计算

### 灵活目标
- **默认行为：** 旋转整个实体
- **自定义目标：** 可指定特定子节点
- **动态切换：** 运行时改变旋转目标

## 注意事项

1. **鼠标指针：** 确保自定义指针纹理分辨率适中
2. **性能考虑：** 大量实体时考虑减少旋转频率
3. **控制冲突：** 正确配置与其他控制组件的交互
4. **物理交互：** 旋转可能影响物理碰撞
5. **UI遮挡：** 考虑UI元素对鼠标位置的影响

## 相关组件
- `InputComponent` - 输入处理，解决控制冲突
- `TurningControlComponent` - 手柄转向控制
- `GunComponent` - 武器系统，常用瞄准目标
- `CameraComponent` - 相机跟随瞄准
- `ActionTargetingMouseComponent` - 基于鼠标的动作目标选择 