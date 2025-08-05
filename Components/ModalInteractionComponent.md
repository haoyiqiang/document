# ModalInteractionComponent API

> **继承关系**: Component > InteractionComponent > ModalInteractionComponent  
> **交互类型**: 模态UI交互

显示UI控件并暂停父实体的交互组件。用于需要玩家关注的对话框、菜单或其他模态界面交互。组件不受暂停影响，支持游戏暂停状态下的UI操作。

## ✨ 主要特性

- 📱 模态UI界面显示
- ⏸️ 自动游戏暂停管理
- 📍 可配置的UI位置
- 🔄 完整的生命周期管理
- 🎯 不受暂停影响的处理

## 📊 导出属性

### UI设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `modalScene` | `PackedScene` | `null` | 要显示的模态UI场景 |
| `setViewGlobalPositionToEntity` | `bool` | `false` | 是否将UI位置设置为实体位置 |
| `viewPositionOffset` | `Vector2` | `Vector2.ZERO` | UI位置偏移量 |

## 🔗 依赖关系

### 必需组件
| 组件 | 关系 | 描述 |
|------|------|------|
| `InteractionComponent` | **基类** | 提供基础交互功能 |

### 状态属性
| 属性 | 类型 | 描述 |
|------|------|------|
| `currentModalUI` | `ModalUI` | 当前显示的模态UI实例 |

## 🎯 使用示例

### 基础对话框交互

```gdscript
# Entity Scene Structure:
# └── Entity (Area2D)
#     ├── CollisionShape2D
#     └── ModalInteractionComponent
#         modalScene: preload("res://UI/DialogBox.tscn")
```

### 商店界面交互

```gdscript
# ShopInteraction.gd
extends ModalInteractionComponent

@export var shopInventory: Array[InventoryItem] = []
@export var shopKeeperId: String = "merchant_001"

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant:
    # 检查玩家是否有货币
    var playerInventory = interactorEntity.findFirstComponentSubclass(InventoryComponent)
    if not playerInventory:
        showMessage("You need a bag to shop!")
        return false
    
    # 设置商店数据
    setupShopData()
    
    # 调用基类显示模态UI
    var result = super.performInteraction(interactorEntity, interactionControlComponent)
    
    # 处理购买结果
    handlePurchaseResult(result, playerInventory)
    
    return result

func setupShopData():
    # 准备商店数据
    GameState.currentShopInventory = shopInventory
    GameState.currentShopKeeper = shopKeeperId

func handlePurchaseResult(result: Variant, playerInventory: InventoryComponent):
    if result is Dictionary and result.has("purchased_items"):
        var purchasedItems = result["purchased_items"] as Array
        for item in purchasedItems:
            playerInventory.addItem(item)
            print("Purchased: ", item.displayName)

func showMessage(text: String):
    # 显示简单消息
    print(text)
```

### 动态UI位置设置

```gdscript
# PositionedModalInteraction.gd
extends ModalInteractionComponent

@export var followEntity: bool = true
@export var screenRelativePosition: Vector2 = Vector2(0.5, 0.3)

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant:
    # 动态设置UI位置
    if followEntity:
        setViewGlobalPositionToEntity = true
        viewPositionOffset = Vector2(0, -100)  # 实体上方
    else:
        setViewGlobalPositionToEntity = false
        # 设置为屏幕相对位置
        var viewport = get_viewport()
        var screenSize = viewport.get_visible_rect().size
        viewPositionOffset = screenSize * screenRelativePosition
    
    return super.performInteraction(interactorEntity, interactionControlComponent)
```

### 自定义模态UI组件

```gdscript
# CustomModalUI.gd (在模态场景中使用)
extends ModalUI

@export var dialogText: String = ""
@export var choices: Array[String] = []

var selectedChoice: int = -1

signal didFinish(result: Variant)

func _ready():
    setupDialog()
    connectButtons()

func setupDialog():
    $DialogText.text = dialogText
    
    # 动态创建选择按钮
    for i in range(choices.size()):
        var button = Button.new()
        button.text = choices[i]
        button.pressed.connect(_on_choice_selected.bind(i))
        $ChoicesContainer.add_child(button)

func connectButtons():
    $CloseButton.pressed.connect(_on_close_pressed)

func _on_choice_selected(choiceIndex: int):
    selectedChoice = choiceIndex
    var result = {
        "choice": choiceIndex,
        "choice_text": choices[choiceIndex],
        "dialog": dialogText
    }
    didFinish.emit(result)

func _on_close_pressed():
    selectedChoice = -1
    didFinish.emit(null)
```

### 多阶段交互系统

```gdscript
# QuestInteraction.gd
extends ModalInteractionComponent

@export var questId: String = ""
@export var questStage: int = 0

var questDialogs: Dictionary = {
    0: preload("res://UI/Dialogs/QuestStart.tscn"),
    1: preload("res://UI/Dialogs/QuestProgress.tscn"),
    2: preload("res://UI/Dialogs/QuestComplete.tscn")
}

func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant:
    # 根据任务状态选择不同的UI
    var currentStage = GameState.getQuestStage(questId)
    
    if currentStage in questDialogs:
        modalScene = questDialogs[currentStage]
    else:
        modalScene = questDialogs[0]  # 默认开始对话
    
    var result = super.performInteraction(interactorEntity, interactionControlComponent)
    
    # 更新任务状态
    updateQuestProgress(result, currentStage)
    
    return result

func updateQuestProgress(result: Variant, currentStage: int):
    if result is Dictionary:
        match result.get("action", ""):
            "accept_quest":
                GameState.setQuestStage(questId, 1)
                questStage = 1
            "complete_quest":
                GameState.setQuestStage(questId, 2)
                questStage = 2
                # 给予奖励
                giveQuestReward(result.get("rewards", []))

func giveQuestReward(rewards: Array):
    for reward in rewards:
        print("Quest reward: ", reward)
```

## 🔧 技术细节

### 模态交互流程
```gdscript
func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> Variant:
    # 1. 实例化模态场景
    var modalView: ModalUI = modalScene.instantiate()
    
    # 2. 设置处理模式（不受暂停影响）
    self.process_mode = Node.PROCESS_MODE_ALWAYS
    modalView.process_mode = Node.PROCESS_MODE_ALWAYS
    
    # 3. 设置UI位置
    if setViewGlobalPositionToEntity:
        modalView.global_position = parentEntity.global_position
    modalView.position += viewPositionOffset
    
    # 4. 暂停游戏并显示UI
    get_tree().paused = true
    get_tree().current_scene.add_child(modalView)
    
    # 5. 连接完成信号
    modalView.didFinish.connect(modalView_didFinish)
    
    return modalView.lastResult
```

### 清理和恢复
```gdscript
func modalView_didFinish(result: Variant) -> void:
    get_tree().current_scene.remove_child(currentModalUI)
    currentModalUI.queue_free()
    get_tree().paused = false
```

## ⚠️ 注意事项

1. **处理模式**: 组件和模态UI必须设置为PROCESS_MODE_ALWAYS
2. **暂停管理**: 自动处理游戏暂停和恢复
3. **内存管理**: 模态UI使用完后自动清理
4. **位置设置**: 支持实体相对位置和绝对位置
5. **继承要求**: 子类必须调用`super.performInteraction()`

## 🔗 相关组件

- [InteractionComponent](InteractionComponent.md) - 基础交互组件
- [InteractionControlComponent](./InteractionControlComponent.md) - 交互控制组件
- [Component](../Component.md) - 基础组件类

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 