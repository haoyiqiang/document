# TextInteractionComponent API

> **继承关系**: Component > InteractionComponent > TextInteractionComponent

文本交互组件，用于显示和循环播放文本字符串序列，适用于标识牌、静态NPC对话等场景。

## ✨ 主要特性

- 📝 多文本字符串循环显示
- 🎨 支持不同颜色的文本显示
- 🔄 自动文本索引循环
- 🎯 交互式文本切换
- 📊 完成最后文本时的特殊信号
- ⏰ 支持自动化文本播放

## 📊 导出属性

### 文本设置
| 属性 | 类型 | 默认值 | 描述 |
|------|------|--------|------|
| `textStrings` | `PackedStringArray` | `["String1", "String2", "String3"]` | 要循环显示的文本字符串列表 |
| `textColors` | `Array[Color]` | `[Color.WHITE, Color.YELLOW]` | 文本颜色数组，可与文本数组大小不同 |

## 📡 信号

```gdscript
signal didDisplayFinalText()
```

| 信号 | 触发时机 | 用途 |
|------|----------|------|
| `didDisplayFinalText` | 显示最后一个文本时 | 标记对话序列完成，触发特殊逻辑 |

## 🛠️ 主要方法

### 交互处理
```gdscript
func performInteraction(interactorEntity: Entity, interactionControlComponent: InteractionControlComponent) -> String
```
**描述**: 处理玩家交互，切换到下一个文本  
**返回**: 当前显示的文本标签

### 文本显示
```gdscript
func displayNextText() -> void
```
**描述**: 显示下一个文本，可由Timer或其他脚本调用

### 文本应用
```gdscript
func applyTextFromArray(indexOverride: int = currentTextIndex) -> void
```
**描述**: 从数组应用指定索引的文本和颜色

### 索引递增
```gdscript
func incrementIndices() -> void
```
**描述**: 递增文本和颜色索引，支持循环

## 🎯 使用示例

### 基础标识牌

```gdscript
# 在编辑器中设置
# textStrings = ["欢迎来到村庄!", "这里有友善的村民", "记得访问商店"]
# textColors = [Color.WHITE, Color.CYAN, Color.YELLOW]

# SignPost.gd
extends Entity

func _ready():
    var textInteraction = $TextInteractionComponent
    textInteraction.didDisplayFinalText.connect(onSignComplete)

func onSignComplete():
    print("玩家已读完标识牌")
```

### NPC对话系统

```gdscript
# NPCDialogue.gd
extends Entity

@export var npcName: String = "村民"
@export var dialogueLines: PackedStringArray = [
    "你好，旅行者！",
    "这个村子最近很平静。",
    "如果你需要补给，去找铁匠吧。",
    "祝你旅途愉快！"
]

func _ready():
    setupDialogue()

func setupDialogue():
    var textComponent = $TextInteractionComponent
    textComponent.textStrings = dialogueLines
    textComponent.textColors = [
        Color.WHITE,      # 问候
        Color.LIGHT_BLUE, # 信息
        Color.YELLOW,     # 建议
        Color.GREEN       # 祝福
    ]
    
    textComponent.didDisplayFinalText.connect(onDialogueComplete)

func onDialogueComplete():
    # 对话结束后的特殊行为
    print(npcName + "的对话结束")
    # 可以触发任务、给予奖励等
```

### 自动播放文本

```gdscript
# AutoTextDisplay.gd
extends Entity

@export var autoPlayInterval: float = 2.0

func _ready():
    var textComponent = $TextInteractionComponent
    var timer = Timer.new()
    add_child(timer)
    
    timer.wait_time = autoPlayInterval
    timer.autostart = true
    timer.timeout.connect(textComponent.displayNextText)
    
    textComponent.didDisplayFinalText.connect(onAutoTextComplete)

func onAutoTextComplete():
    # 自动播放完成后停止
    $Timer.stop()
```

### 多语言支持

```gdscript
# MultiLanguageText.gd
extends Entity

var textData: Dictionary = {
    "en": ["Hello!", "Welcome!", "Goodbye!"],
    "zh": ["你好！", "欢迎！", "再见！"],
    "ja": ["こんにちは！", "いらっしゃいませ！", "さようなら！"]
}

func _ready():
    setupLanguage(getCurrentLanguage())

func setupLanguage(language: String):
    var textComponent = $TextInteractionComponent
    if textData.has(language):
        textComponent.textStrings = textData[language]

func getCurrentLanguage() -> String:
    # 获取当前语言设置
    return "zh"
```

### 条件文本显示

```gdscript
# ConditionalText.gd
extends Entity

@export var playerStats: StatsComponent

var morningTexts: PackedStringArray = ["早上好！", "新的一天开始了"]
var eveningTexts: PackedStringArray = ["晚上好！", "夜晚来临了"]

func _ready():
    updateTextBasedOnTime()

func updateTextBasedOnTime():
    var textComponent = $TextInteractionComponent
    var currentHour = Time.get_datetime_dict_from_system().hour
    
    if currentHour < 18:
        textComponent.textStrings = morningTexts
        textComponent.textColors = [Color.YELLOW, Color.ORANGE]
    else:
        textComponent.textStrings = eveningTexts
        textComponent.textColors = [Color.PURPLE, Color.DARK_BLUE]
```

## 🏛️ 设计模式

### 迭代器模式
- **序列管理**: 文本数组的有序遍历
- **循环迭代**: 到达末尾后重新开始
- **状态跟踪**: 当前索引位置记录

### 观察者模式
- **事件通知**: didDisplayFinalText信号
- **状态变化**: 文本切换时的自动更新

## 🔧 技术细节

### 索引管理
```gdscript
func incrementIndices() -> void:
    currentTextIndex += 1
    if currentTextIndex >= textStrings.size():
        currentTextIndex = 0
        
    currentColorIndex += 1
    if currentColorIndex >= textColors.size():
        currentColorIndex = 0
```

### 颜色应用
```gdscript
var labelControl: Label = self.interactionIndicator as Label
if labelControl:
    labelControl.label_settings.font_color = textColors[currentColorIndex]
```

### 最终文本检测
```gdscript
if currentTextIndex == textStrings.size() - 1:
    didDisplayFinalText.emit()
```

## ⚠️ 注意事项

1. **数组大小**: textColors数组可以与textStrings大小不同，会自动循环
2. **Label节点**: 需要interactionIndicator是Label类型才能应用颜色
3. **继承特性**: 继承自InteractionComponent，具备所有交互功能
4. **索引管理**: 文本和颜色索引独立循环
5. **自动化**: displayNextText()可被外部Timer调用

## 🔗 相关组件

- [InteractionComponent](./InteractionComponent.md) - 基础交互组件
- [InteractionControlComponent](./InteractionControlComponent.md) - 交互控制
- [LabelComponent](./LabelComponent.md) - 文本显示组件
- [InteractionWithCostComponent](./InteractionWithCostComponent.md) - 有成本的交互

---

**版本**: Comedot Framework v1.0  
**文档更新**: 2024-12-21 