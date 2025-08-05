# CameraComponent API 文档

## 概述
`CameraComponent` 是一个Camera2D节点组件，具有各种选项，如在父Entity被销毁时附加到祖父节点，以保持屏幕视图位置。

**警告**：可能在场景开始时出现初始延迟或不期望的平移，特别是启用了 `position_smoothing_enabled` 时。

## 继承
`CameraComponent` → `Component` → `Node`

## 主要特性
- 智能父节点重新附加
- 边界限制
- 鼠标跟踪
- 前瞻功能
- 缩放弹跳效果

## 解决方案
为了解决初始延迟问题，在播放器Entity外添加独立的Camera2D，与此CameraComponent位置相同，然后添加RemoteTransform2D作为此组件的子节点，链接到外部Camera2D。

## 导出属性

### 基础设置

#### `shouldAttachToGrandparentOnEntityRemoval: bool = true`
Entity移除时是否分离并重新附加到Entity的父节点。
- 防止玩家死亡时场景视图跳转
- **注意**：需要设置 `Component.allowNonEntityParent = true`

#### `shouldClampToBoundary: bool = false`
是否将摄像机限制在边界矩形范围内。

#### `boundary: Area2D`
如果启用 `shouldClampToBoundary`，用于限制摄像机位置的Area2D边界。

### 跟踪功能

#### `shouldTrackMouse: bool = false` **（实验性）**
是否每帧将摄像机移动到鼠标位置。
- 被 `shouldLookAhead` 覆盖

#### `shouldLookAhead: bool = false` **（实验性）**
是否将摄像机进一步移动到屏幕边缘朝向鼠标指针。
- 覆盖 `shouldTrackMouse`

#### `lookAheadDeadZone: float = 64` **（实验性）**
前瞻目标（鼠标指针）距离屏幕中心多远时开始移动摄像机偏移。

### 特效功能

#### `shouldBounceZoom: bool = false` **（实验性）**
是否"弹跳"或"摇头"摄像机缩放进出屏幕。用于诱发眩晕。

#### `zoomTimerMax: float = 0.2`
缩放方向在"进"和"出"之间翻转的秒数（范围：0.0-10.0）。

#### `zoomDirection: float = 0.2`
缩放的距离/强度。运行时会交换符号/"方向"（范围：0.0-10.0）。

## 状态属性

#### `selfAsCamera: Camera2D`
此组件自身作为Camera2D的引用。

#### `camera: Camera2D`
实际使用的摄像机。可以是外部Camera2D节点。

#### `zoomFlipTimer: float`
缩放弹跳的内部计时器。

## 主要方法

### 附加管理

#### `registerEntity(newParentEntity: Entity) -> void`
重写以在新父Entity是Entity时重新连接信号。

#### `connectSignals() -> void`
连接父Entity的删除前信号。

#### `onEntity_preDelete() -> void`
Entity删除前的处理，触发重新附加。

#### `attachToGrandparent() -> void`
将摄像机从Entity分离并重新附加到Entity的父节点或场景根节点。

### 边界管理

#### `clampToBoundary() -> void`
根据边界Area2D设置摄像机限制。

## 使用示例

### 基本玩家摄像机
```gdscript
# 设置摄像机组件
var cameraComponent = CameraComponent.new()
playerEntity.add_child(cameraComponent)

# 配置基本属性
cameraComponent.shouldAttachToGrandparentOnEntityRemoval = true
cameraComponent.allowNonEntityParent = true  # 重要：允许非Entity父节点
```

### 边界限制设置
```gdscript
# 创建边界区域
var boundaryArea = Area2D.new()
var boundaryShape = RectangleShape2D.new()
boundaryShape.size = Vector2(1920, 1080)  # 设置边界大小

var collisionShape = CollisionShape2D.new()
collisionShape.shape = boundaryShape
boundaryArea.add_child(collisionShape)

# 应用到摄像机
cameraComponent.boundary = boundaryArea
cameraComponent.shouldClampToBoundary = true
```

### 鼠标跟踪
```gdscript
# 启用鼠标跟踪
cameraComponent.shouldTrackMouse = true

# 或者启用前瞻（更高级）
cameraComponent.shouldLookAhead = true
cameraComponent.lookAheadDeadZone = 100  # 鼠标需要离中心100像素才开始前瞻
```

### 缩放弹跳效果
```gdscript
# 启用眩晕效果
cameraComponent.shouldBounceZoom = true
cameraComponent.zoomTimerMax = 0.5  # 每0.5秒改变方向
cameraComponent.zoomDirection = 0.3  # 缩放强度
```

### 外部摄像机设置
```gdscript
# 解决初始延迟的高级设置
# 1. 在场景中创建独立的Camera2D
var externalCamera = Camera2D.new()
get_tree().current_scene.add_child(externalCamera)
externalCamera.global_position = playerEntity.global_position

# 2. 创建RemoteTransform2D链接
var remoteTransform = RemoteTransform2D.new()
cameraComponent.add_child(remoteTransform)
remoteTransform.remote_path = externalCamera.get_path()

# 3. 设置组件使用外部摄像机
cameraComponent.camera = externalCamera
```

## 高级功能

### 前瞻系统详解
```gdscript
# 前瞻系统的工作原理
func _input(event: InputEvent) -> void:
    if shouldLookAhead and event is InputEventMouseMotion:
        # 获取未缩放的视口尺寸
        var viewport: Rect2 = camera.get_viewport_rect()
        # 获取鼠标相对于屏幕中心的位置
        var target: Vector2 = event.position - (viewport.size * 0.5)
        
        if target.length() < lookAheadDeadZone:
            camera.offset = Vector2.ZERO  # 死区内不偏移
        else:
            # 计算偏移量
            camera.offset = target.normalized() * (target.length() - lookAheadDeadZone) * 0.5
```

### 平滑跟踪
```gdscript
# 平滑的鼠标跟踪
func _process(delta: float) -> void:
    if shouldTrackMouse:
        var targetPosition = camera.get_global_mouse_position()
        camera.global_position = camera.global_position.lerp(targetPosition, delta * 2.0)
```

### 动态边界
```gdscript
# 运行时改变边界
func change_camera_bounds(newBounds: Rect2):
    if boundary:
        # 更新Area2D的形状
        var shape = boundary.get_child(0).shape as RectangleShape2D
        shape.size = newBounds.size
        boundary.global_position = newBounds.position
        
        # 重新应用边界
        cameraComponent.clampToBoundary()
```

## 重新附加系统

### Entity死亡时的摄像机处理
```gdscript
# 当玩家死亡时，摄像机会自动重新附加
# 1. Entity发出preDelete信号
# 2. CameraComponent接收信号
# 3. 摄像机分离并重新附加到祖父节点
# 4. 保持当前视图位置

# 手动触发重新附加
func respawn_player():
    # 玩家重生后，可能需要重新附加摄像机
    var oldCameraPosition = cameraComponent.camera.global_position
    cameraComponent.attachToGrandparent()
    # 保持位置
    cameraComponent.camera.global_position = oldCameraPosition
```

## 性能优化

### 条件处理
```gdscript
# 组件自动优化处理
# 只在需要时启用每帧处理
self.set_process(shouldTrackMouse or shouldBounceZoom)
self.set_process_input(shouldLookAhead)
```

### 大世界优化
```gdscript
# 大世界中的摄像机优化
func optimize_for_large_world():
    # 禁用位置平滑以减少计算
    camera.position_smoothing_enabled = false
    
    # 减少缩放弹跳频率
    zoomTimerMax = 1.0  # 更慢的弹跳
    
    # 使用较小的前瞻死区
    lookAheadDeadZone = 32
```

## 调试工具

### 可视化调试
```gdscript
func _draw():
    if debugMode:
        # 绘制边界
        if boundary:
            var bounds = Tools.getShapeGlobalBounds(boundary)
            draw_rect(bounds, Color.RED, false, 2)
        
        # 绘制前瞻死区
        if shouldLookAhead:
            draw_circle(Vector2.ZERO, lookAheadDeadZone, Color.YELLOW, false, 1)
```

### 状态监控
```gdscript
func showDebugInfo() -> void:
    if debugMode:
        Debug.addComponentWatchList(self, {
            camera_position = camera.global_position,
            camera_offset = camera.offset,
            is_attached_to_entity = parentEntity != null,
            boundary_active = shouldClampToBoundary,
            zoom_level = camera.zoom
        })
```

## 注意事项

1. **初始位置**：可能出现初始延迟或跳跃
2. **父节点要求**：重新附加时需要设置 `allowNonEntityParent`
3. **性能影响**：某些功能每帧更新，注意性能
4. **信号连接**：确保正确连接Entity的preDelete信号
5. **外部摄像机**：使用RemoteTransform2D可以解决某些问题

## 相关组件
- `RemoteTransform2D` - 远程变换链接
- `Entity` - 实体容器
- `DebugComponent` - 调试显示
- 相关脚本：`CameraMouseTracking.gd`、`ClampCameraToArea.gd` 