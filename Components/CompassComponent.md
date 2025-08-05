# CompassComponent API

> **继承关系**: Component > CompassComponent  
> **视觉类型**: 指南针指示

跟踪目标节点并显示指南针指示器的组件，用于显示目标方向和距离。支持即时或渐进旋转，可在接近目标时自动隐藏，适用于导航系统、任务指示器、雷达等功能。

## ✨ 主要特性

- 🧭 精确方向指示
- 📍 目标节点跟踪
- ⚡ 即时或渐进旋转
- 👁️ 接近时自动隐藏
- 🎨 可自定义指示器
- 📏 距离感知显示

## 📊 导出属性

### 目标跟踪
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `nodeToTrack` | `Node2D` | `null` | 要跟踪的目标节点 |
| `compassIndicatorOverride` | `Node2D` | `null` | 自定义指南针指示器节点 |

### 旋转控制
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldRotateInstantly` | `bool` | `false` | 是否瞬间旋转到目标方向 |
| `rotationSpeed` | `float` | `10.0` | 旋转速度（0.1-20） |

### 显示控制
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `shouldDisappearWhenNear` | `bool` | `true` | 接近目标时是否隐藏指南针 |
| `proximityDistance` | `float` | `100.0` | 接近距离阈值（像素） |

## 🎯 使用示例

### 基础导航指南针

```gdscript
# Entity Scene Structure:
# └── Player (CharacterBody2D)
#     ├── Sprite2D
#     ├── CompassComponent
#     │   nodeToTrack: [Destination引用]
#     │   rotationSpeed: 5.0
#     │   shouldDisappearWhenNear: true
#     └── CompassIndicator (Node2D)  # %CompassIndicator
#         └── Sprite2D  # 箭头图标
```

### 任务目标指示器

```gdscript
# QuestCompassComponent.gd
extends CompassComponent

@export var questManager: QuestManager
@export var showDistance: bool = true
@export var showObjectiveName: bool = true
@export var compassUI: Control

var currentObjective: QuestObjective
var distanceLabel: Label
var objectiveLabel: Label

func _ready():
    super._ready()
    setupUI()
    connectQuestManager()

func setupUI():
    if compassUI:
        distanceLabel = compassUI.get_node_or_null("DistanceLabel")
        objectiveLabel = compassUI.get_node_or_null("ObjectiveLabel")

func connectQuestManager():
    if questManager:
        questManager.active_objective_changed.connect(_on_objective_changed)
        questManager.quest_completed.connect(_on_quest_completed)

func _process(delta: float):
    super._process(delta)
    
    if currentObjective and nodeToTrack:
        updateDistanceDisplay()
        updateObjectiveDisplay()

func updateDistanceDisplay():
    if not showDistance or not distanceLabel:
        return
    
    var distance = parentEntity.global_position.distance_to(nodeToTrack.global_position)
    distanceLabel.text = formatDistance(distance)
    
    # 距离颜色编码
    if distance < proximityDistance:
        distanceLabel.modulate = Color.GREEN
    elif distance < proximityDistance * 3:
        distanceLabel.modulate = Color.YELLOW  
    else:
        distanceLabel.modulate = Color.RED

func formatDistance(distance: float) -> String:
    if distance < 1000:
        return str(int(distance)) + "m"
    else:
        return "%.1f" % (distance / 1000.0) + "km"

func updateObjectiveDisplay():
    if not showObjectiveName or not objectiveLabel:
        return
    
    if currentObjective:
        objectiveLabel.text = currentObjective.title
        objectiveLabel.visible = true
    else:
        objectiveLabel.visible = false

func _on_objective_changed(objective: QuestObjective):
    currentObjective = objective
    
    if objective and objective.targetNode:
        nodeToTrack = objective.targetNode
        print("Compass now pointing to: ", objective.title)
    else:
        nodeToTrack = null
        print("No active objective")

func _on_quest_completed():
    currentObjective = null
    nodeToTrack = null
    hideCompass()

func hideCompass():
    var compass = getCompassIndicator()
    if compass:
        compass.visible = false

func getCompassIndicator() -> Node2D:
    if compassIndicatorOverride:
        return compassIndicatorOverride
    else:
        return %CompassIndicator if has_node("%CompassIndicator") else null
```

### 多目标雷达系统

```gdscript
# RadarCompassComponent.gd
extends CompassComponent

@export var radarRange: float = 500.0
@export var targetGroups: Array[String] = ["enemies", "treasures"]
@export var showMultipleTargets: bool = true
@export var maxTargets: int = 5

var detectedTargets: Array[Node2D] = []
var targetIndicators: Array[Node2D] = []
var scanTimer: float = 0.0
var scanInterval: float = 1.0  # 每秒扫描一次

func _ready():
    super._ready()
    createTargetIndicators()

func _process(delta: float):
    # 定期扫描目标
    scanTimer += delta
    if scanTimer >= scanInterval:
        scanForTargets()
        scanTimer = 0.0
    
    # 更新多目标显示
    if showMultipleTargets:
        updateMultipleTargets(delta)
    else:
        # 单目标模式，使用最近的目标
        var nearestTarget = findNearestTarget()
        if nearestTarget != nodeToTrack:
            nodeToTrack = nearestTarget
    
    super._process(delta)

func scanForTargets():
    detectedTargets.clear()
    
    for groupName in targetGroups:
        var groupNodes = get_tree().get_nodes_in_group(groupName)
        for node in groupNodes:
            var distance = parentEntity.global_position.distance_to(node.global_position)
            if distance <= radarRange:
                detectedTargets.append(node)
    
    # 按距离排序
    detectedTargets.sort_custom(compareByDistance)
    
    # 限制目标数量
    if detectedTargets.size() > maxTargets:
        detectedTargets = detectedTargets.slice(0, maxTargets)
    
    print("Radar detected ", detectedTargets.size(), " targets")

func compareByDistance(a: Node2D, b: Node2D) -> bool:
    var distA = parentEntity.global_position.distance_to(a.global_position)
    var distB = parentEntity.global_position.distance_to(b.global_position)
    return distA < distB

func findNearestTarget() -> Node2D:
    if detectedTargets.is_empty():
        return null
    return detectedTargets[0]  # 已按距离排序

func createTargetIndicators():
    # 为每个可能的目标创建指示器
    for i in maxTargets:
        var indicator = createIndicator()
        indicator.visible = false
        indicator.name = "TargetIndicator" + str(i)
        parentEntity.add_child(indicator)
        targetIndicators.append(indicator)

func createIndicator() -> Node2D:
    var indicator = Node2D.new()
    var sprite = Sprite2D.new()
    
    # 加载指示器图标
    sprite.texture = preload("res://UI/Icons/radar_blip.png")
    sprite.scale = Vector2(0.5, 0.5)
    
    indicator.add_child(sprite)
    return indicator

func updateMultipleTargets(delta: float):
    # 隐藏所有指示器
    for indicator in targetIndicators:
        indicator.visible = false
    
    # 显示检测到的目标
    for i in detectedTargets.size():
        if i >= targetIndicators.size():
            break
        
        var target = detectedTargets[i]
        var indicator = targetIndicators[i]
        
        # 计算方向
        var direction = (target.global_position - parentEntity.global_position).normalized()
        var angle = direction.angle()
        
        # 设置指示器位置（围绕父实体的圆周）
        var radius = 80.0
        indicator.position = direction * radius
        indicator.rotation = angle
        indicator.visible = true
        
        # 根据目标类型设置颜色
        var sprite = indicator.get_child(0) as Sprite2D
        if target.is_in_group("enemies"):
            sprite.modulate = Color.RED
        elif target.is_in_group("treasures"):
            sprite.modulate = Color.GOLD
        else:
            sprite.modulate = Color.WHITE
        
        # 根据距离调整透明度
        var distance = parentEntity.global_position.distance_to(target.global_position)
        var alpha = 1.0 - (distance / radarRange)
        sprite.modulate.a = clamp(alpha, 0.3, 1.0)

func setRadarRange(range: float):
    radarRange = range
    scanForTargets()  # 重新扫描

func addTargetGroup(groupName: String):
    if groupName not in targetGroups:
        targetGroups.append(groupName)
        scanForTargets()

func removeTargetGroup(groupName: String):
    targetGroups.erase(groupName)
    scanForTargets()
```

### 导航记忆系统

```gdscript
# NavigationMemoryCompass.gd
extends CompassComponent

@export var rememberLocations: bool = true
@export var maxMemoryLocations: int = 10
@export var autoSaveInterval: float = 30.0

var visitedLocations: Array[Dictionary] = []
var waypointMarkers: Array[Node2D] = []
var autoSaveTimer: float = 0.0
var currentWaypointIndex: int = 0

func _ready():
    super._ready()
    loadVisitedLocations()
    createWaypointMarkers()

func _process(delta: float):
    super._process(delta)
    
    if rememberLocations:
        autoSaveTimer += delta
        if autoSaveTimer >= autoSaveInterval:
            saveCurrentLocation()
            autoSaveTimer = 0.0
    
    updateWaypointMarkers()

func saveCurrentLocation():
    var location = {
        "position": parentEntity.global_position,
        "timestamp": Time.get_time_dict_from_system(),
        "scene": get_tree().current_scene.scene_file_path,
        "name": generateLocationName()
    }
    
    # 检查是否与已有位置太近
    for existing in visitedLocations:
        var distance = location.position.distance_to(existing.position)
        if distance < 100.0:  # 100像素范围内认为是同一位置
            return
    
    visitedLocations.append(location)
    
    # 限制记忆数量
    if visitedLocations.size() > maxMemoryLocations:
        visitedLocations.pop_front()
    
    saveVisitedLocations()
    print("Location saved: ", location.name)

func generateLocationName() -> String:
    var time = Time.get_time_dict_from_system()
    return "Location_%02d%02d_%02d%02d" % [time.hour, time.minute, time.second, visitedLocations.size()]

func loadVisitedLocations():
    var file = FileAccess.open("user://visited_locations.save", FileAccess.READ)
    if file:
        var json_string = file.get_as_text()
        file.close()
        
        var json = JSON.new()
        var parse_result = json.parse(json_string)
        if parse_result == OK:
            visitedLocations = json.data

func saveVisitedLocations():
    var file = FileAccess.open("user://visited_locations.save", FileAccess.WRITE)
    if file:
        var json_string = JSON.stringify(visitedLocations)
        file.store_string(json_string)
        file.close()

func createWaypointMarkers():
    for i in maxMemoryLocations:
        var marker = Node2D.new()
        var sprite = Sprite2D.new()
        sprite.texture = preload("res://UI/Icons/waypoint.png")
        sprite.scale = Vector2(0.3, 0.3)
        sprite.modulate = Color.CYAN
        sprite.modulate.a = 0.7
        
        marker.add_child(sprite)
        marker.visible = false
        get_tree().current_scene.add_child(marker)
        waypointMarkers.append(marker)

func updateWaypointMarkers():
    # 隐藏所有标记
    for marker in waypointMarkers:
        marker.visible = false
    
    # 显示访问过的位置
    for i in visitedLocations.size():
        if i >= waypointMarkers.size():
            break
        
        var location = visitedLocations[i]
        var marker = waypointMarkers[i]
        
        marker.global_position = location.position
        marker.visible = true
        
        # 当前目标位置特殊显示
        if nodeToTrack and location.position.distance_to(nodeToTrack.global_position) < 50.0:
            marker.modulate = Color.YELLOW
            marker.scale = Vector2(1.2, 1.2)
        else:
            marker.modulate = Color.CYAN
            marker.scale = Vector2(1.0, 1.0)

func navigateToLocation(index: int):
    if index >= 0 and index < visitedLocations.size():
        var location = visitedLocations[index]
        
        # 创建临时目标节点
        var target = Node2D.new()
        target.global_position = location.position
        get_tree().current_scene.add_child(target)
        
        nodeToTrack = target
        currentWaypointIndex = index
        
        print("Navigating to: ", location.name)

func navigateToNearest():
    if visitedLocations.is_empty():
        return
    
    var nearestIndex = 0
    var minDistance = INF
    
    for i in visitedLocations.size():
        var distance = parentEntity.global_position.distance_to(visitedLocations[i].position)
        if distance < minDistance:
            minDistance = distance
            nearestIndex = i
    
    navigateToLocation(nearestIndex)

func _input(event: InputEvent):
    if event.is_action_pressed("next_waypoint"):
        navigateToNextWaypoint()
    elif event.is_action_pressed("prev_waypoint"):
        navigateToPreviousWaypoint()
    elif event.is_action_pressed("mark_location"):
        saveCurrentLocation()

func navigateToNextWaypoint():
    if visitedLocations.is_empty():
        return
    
    currentWaypointIndex = (currentWaypointIndex + 1) % visitedLocations.size()
    navigateToLocation(currentWaypointIndex)

func navigateToPreviousWaypoint():
    if visitedLocations.is_empty():
        return
    
    currentWaypointIndex = (currentWaypointIndex - 1 + visitedLocations.size()) % visitedLocations.size()
    navigateToLocation(currentWaypointIndex)
```

## 🔧 技术细节

### 旋转计算
```gdscript
func _process(delta: float) -> void:
    var compass = compassIndicatorOverride or %CompassIndicator
    var targetPosition = nodeToTrack.global_position
    
    if shouldRotateInstantly:
        compass.look_at(targetPosition)
    else:
        var rotateTo = parentEntity.global_position.angle_to_point(targetPosition)
        compass.global_rotation = rotate_toward(
            compass.global_rotation, 
            rotateTo, 
            rotationSpeed * delta
        )
```

### 距离检测
```gdscript
if shouldDisappearWhenNear:
    var distance = parentEntity.global_position.distance_to(targetPosition)
    compass.visible = distance >= proximityDistance
```

## ⚠️ 注意事项

1. **目标有效性**: nodeToTrack必须有效，否则组件无法正常工作
2. **指示器节点**: 默认使用%CompassIndicator，可通过compassIndicatorOverride自定义
3. **旋转性能**: 每帧计算角度和插值，大量实例需考虑性能
4. **距离计算**: 每帧计算距离用于接近检测
5. **持续旋转**: 即使指南针隐藏时也会继续旋转计算

## 🔗 相关组件

- [NodeFacingComponent](NodeFacingComponent.md) - 节点朝向组件
- [CameraComponent](CameraComponent.md) - 相机组件
- [LabelComponent](LabelComponent.md) - 标签组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 