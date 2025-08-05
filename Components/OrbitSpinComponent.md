# OrbitSpinComponent

## 概述
`OrbitSpinComponent` 是一个轨道旋转组件，用于让子节点围绕指定中心点进行圆周运动。支持多个卫星节点的自动分布、潮汐锁定、旋转方向控制等功能，适用于创建行星系统、旋转武器、轨道防御等效果。

**继承关系：** `OrbitSpinComponent` → `Component` → `Node`

## 主要特性
- 🌍 多卫星节点轨道运动
- ⚡ 自动角度分布算法
- 🔄 顺时针/逆时针旋转控制
- 🎯 潮汐锁定功能（始终朝向轨道中心）
- 📐 旋转偏移角度支持
- 🎮 动态添加/移除卫星节点
- ⚙️ 实时轨道参数调整
- 🧹 自动子节点检测和配置

## 依赖节点
- **子节点: Node2D** - 自动检测所有子节点作为卫星节点

## 导出属性

### `orbitRadius: float = 100.0`
轨道半径。

**类型：** `float`  
**默认值：** `100.0`  
**单位：** 像素  
**说明：** 卫星节点到轨道中心的距离

### `orbitPeriod: float = 2.0`
轨道周期。

**类型：** `float`  
**默认值：** `2.0`  
**单位：** 秒  
**说明：** 完成一圈旋转所需的时间

### `orbitCenter: Node2D`
轨道中心节点。

**类型：** `Node2D`  
**默认值：** `null`  
**说明：** 如果未设置，将使用父实体作为轨道中心

### `enableTidalLock: bool = true`
是否启用潮汐锁定。

**类型：** `bool`  
**默认值：** `true`  
**说明：** 启用后卫星节点会始终朝向轨道中心

### `rotationOffset: float = 0.0`
旋转偏移角度。

**类型：** `float`  
**默认值：** `0.0`  
**单位：** 弧度  
**说明：** 在潮汐锁定基础上的额外旋转偏移

### `clockwise: bool = true`
是否顺时针旋转。

**类型：** `bool`  
**默认值：** `true`  
**说明：** `true` 为顺时针，`false` 为逆时针

## 状态属性

### `satelliteNodes: Array[Node2D]`
卫星节点列表。

**类型：** `Array[Node2D]`  
**说明：** 存储所有参与轨道运动的子节点

### `satelliteAngularOffsets: Array[float]`
卫星节点角度偏移列表。

**类型：** `Array[float]`  
**说明：** 存储每个卫星节点的初始角度偏移

### `orbitTime: float = 0.0`
轨道时间累计。

**类型：** `float`  
**说明：** 用于计算当前轨道角度的累计时间

## 核心方法

### `setupOrbitNodes() -> void`
设置轨道节点。

**功能：**
1. 如果没有设置轨道中心，使用父实体
2. 自动获取所有子节点作为卫星节点
3. 为每个卫星节点设置均匀分布的角度偏移
4. 输出设置信息

### `updateOrbitPositions() -> void`
更新轨道位置。

**功能：**
1. 检查轨道中心是否存在
2. 遍历所有卫星节点
3. 计算每个节点的轨道位置
4. 更新节点全局位置
5. 应用潮汐锁定旋转（如果启用）

### `addSatellite(satellite: Node2D, angleOffset: float = 0.0) -> void`
添加卫星节点。

**参数：**
- `satellite` - 要添加的卫星节点
- `angleOffset` - 角度偏移（可选）

**功能：**
1. 临时隐藏卫星节点
2. 将节点添加为子节点
3. 重新设置轨道节点
4. 立即设置到正确的轨道位置
5. 应用潮汐锁定（如果启用）
6. 显示卫星节点

### `removeSatellite(satellite: Node2D) -> void`
移除卫星节点。

**参数：** `satellite` - 要移除的卫星节点  
**功能：**
1. 从卫星节点列表中移除
2. 从角度偏移列表中移除
3. 输出移除信息

## 配置方法

### `setOrbitRadius(radius: float) -> void`
设置轨道半径。

**参数：** `radius` - 新的轨道半径

### `setOrbitPeriod(period: float) -> void`
设置轨道周期。

**参数：** `period` - 新的轨道周期（秒）

### `setOrbitSpeed(speed: float) -> void`
设置轨道速度（向后兼容）。

**参数：** `speed` - 角速度（弧度/秒）  
**说明：** 将角速度转换为轨道周期

### `setEnableTidalLock(enable: bool) -> void`
设置潮汐锁定。

**参数：** `enable` - 是否启用潮汐锁定

### `setRotationOffset(offset: float) -> void`
设置旋转偏移。

**参数：** `offset` - 旋转偏移角度（弧度）

### `setClockwise(isClockwise: bool) -> void`
设置旋转方向。

**参数：** `isClockwise` - 是否顺时针旋转

### `toggleRotationDirection() -> void`
切换旋转方向。

**功能：** 在顺时针和逆时针之间切换

## 生命周期

### `_ready() -> void`
组件初始化。

**功能：** 调用 `setupOrbitNodes()` 设置轨道节点

### `_physics_process(delta: float) -> void`
物理更新。

**参数：** `delta` - 时间增量  
**功能：**
1. 计算角速度
2. 根据旋转方向调整角速度符号
3. 更新轨道时间
4. 更新轨道位置

## 使用示例

### 基础配置
```gdscript
# 创建轨道旋转组件
var orbitComponent = OrbitSpinComponent.new()
orbitComponent.orbitRadius = 150.0
orbitComponent.orbitPeriod = 3.0
orbitComponent.clockwise = true
orbitComponent.enableTidalLock = true
```

### 动态添加卫星
```gdscript
# 添加卫星节点
var satellite = Node2D.new()
orbitComponent.addSatellite(satellite, PI / 4)  # 45度偏移
```

### 实时调整参数
```gdscript
# 动态调整轨道参数
orbitComponent.setOrbitRadius(200.0)
orbitComponent.setOrbitPeriod(1.5)
orbitComponent.toggleRotationDirection()
```

### 多卫星系统
```gdscript
# 创建多卫星系统
var orbitComponent = OrbitSpinComponent.new()
orbitComponent.orbitRadius = 100.0
orbitComponent.orbitPeriod = 2.0

# 添加多个卫星（会自动均匀分布）
for i in range(3):
    var satellite = createSatellite()
    orbitComponent.addSatellite(satellite)
```

## 注意事项

1. **子节点检测：** 组件会自动检测所有 `Node2D` 类型的子节点作为卫星
2. **角度分布：** 多个卫星会自动均匀分布在轨道上
3. **潮汐锁定：** 启用后卫星会始终朝向轨道中心，适合行星系统
4. **性能考虑：** 大量卫星节点可能影响性能，建议合理控制数量
5. **坐标系统：** 使用全局坐标系统，确保轨道中心位置正确
6. **时间单位：** 轨道周期使用秒为单位，角度使用弧度

## 应用场景

- **行星系统：** 创建太阳系或类似的天体系统
- **旋转武器：** 制作旋转的防御塔或攻击装置
- **粒子效果：** 创建旋转的粒子或特效
- **UI元素：** 制作旋转的界面元素
- **游戏机制：** 实现轨道防御、旋转平台等游戏机制

## 相关组件

- **Component** - 基础组件系统
- **Node2D** - 2D节点基类
- **TimerComponentBase** - 时间控制相关
- **MovementComponent** - 运动系统相关 