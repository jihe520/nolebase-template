

# 背景

最近，阿里的开源大模型刷屏了，我一直都是通义千问的高度使用用户，也本地部署过qwen系列模型，自己也是 AI 代码助手的高度依赖用户，同时，也是 AI 项目的开发者。
![image.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_cacd7128b7574c9a9933f0653e7fbffb.png)

![image.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_9232192f9a0f46448756ee798b69622b.png)

平时都是使用微软的github copilot。
作为一个学生，我可以白嫖 copilot ，所以我一直都是免费使用 需要付费的 copilot，对其他 AI 代码助手都不怎么感冒，心想：“免费的肯定不如付费的好”。但是 由于qwen开源模型在开源榜单上大杀四方，所以不得心生几分好感，不免下下来尝试下。
正好最近我正在开发一款开源桌宠软件，链接地址如下：
https://github.com/jihe520/Desktop-Pet-Godot

> 项目简介：🤖👾🐶一款由大语言模型驱动、Godot 制作的 AI 桌宠，旨在提供一个全能的、丰富的桌面AI宠物

![QQ_1729841526717.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_3cf15eaa45084e3b93d06ab5c31f391f.png)

**项目使用的是开源的 Godot 游戏引擎，使用的语言是自带的 gdscript ，语法类似 Python 但是和游戏引擎绑定更紧密，这个项目对 Copilot 来说，还是有很多难度，因为该语言语法API 变化快，godot3 和 godot4的语法发生大变化，许多LLM都是给的godot3 淘汰的语法，不能给出最新的语法，该语言也比较小众，缺少训练资料，让我来测测千问灵码能力。**

![QQ_1729841561710.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_ba28a1945d194185afceb8d0940b84ac.png)
难点：
- 更好的泛化能力，对小众语言学习能力强
- 对整个大项目理解程度高
- 训练数据集是否及时更新，能否适应语法API的更新变化
以上考察的难点，也是我最关心的点，也算是技术难点吧。

废话少说，接下来开始使用

首先，因为引擎集成ide，我们这里切换使用外部的ide vscode，装上lingma插件。
![QQ_1729842306412.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_ec369081da74463a841dcd11a451481b.png)

# 项目结构

为方便演示，我先让他，先整体认识我的项目
![QQ_1729842661429.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_d150a99580014fcb9fd7e04c297d8cdf.png)

#  解释代码

我忘记了 Globals.gd 里面的逻辑关系，我便让 qwen 给我解释下，他出乎我的意料，还给出了相关流程图，帮助我更好的理解。
![QQ_1729842872641.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_21564cebfbe042a0b0026d065475ef40.png)
# 解决 bug 

## bug1

我发现，保存预设时候，每次按钮会成倍添加
![QQ_1729845817590.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_304330963a5049ae8ca6ef0a2bf71aa8.png)
代码如下：

```python
extends Control

@onready var presets_container: GridContainer = %PresetsContainer

const PRESET_PANEL = preload("res://send/store_preset/preset_panel.tscn")

  

func _ready() -> void:

    Globals.add_new_preset_panel.connect(_load_presets)

    _load_presets()

  

func _load_presets():

    for preset in Globals.presets:

        var preset_panel : PresetPanel = PRESET_PANEL.instantiate()

        preset_panel.panel_type = PresetPanel.PanelType.PresetType

        preset_panel.label_name = preset

        preset_panel.preset = Globals.presets[preset]

        presets_container.add_child(preset_panel)
```

![QQ_1729845745098.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_c571d7a2b43e4d7d9856e7c7a67b04f7.png)

```python
func _load_presets():

        # 清除现有的预设面板

    for child in presets_container.get_children():

        if child is PresetPanel:

            child.queue_free()

  

    for preset in Globals.presets:

        var preset_panel : PresetPanel = PRESET_PANEL.instantiate()

        preset_panel.panel_type = PresetPanel.PanelType.PresetType

        preset_panel.label_name = preset

        preset_panel.preset = Globals.presets[preset]

        presets_container.add_child(preset_panel)
```

qwen 非常聪明，帮我排查出问题并给出解决措施。
ta正确使用了gdscript最新的api，具有很好的泛化能力和学习能力。
并且还在清除前做了个判断，保证代码的健壮性。

## bug2

当点击发送按钮，大模型没有返还内容，qwen带着我，排除问题

![QQ_1729843786991.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_d2621db0fcc24493ae25979b22091fe0.png)

首先怀疑，是请求模型的数据没有正确加载，我点击按钮时候发现，没有反应，我就让qwen帮我打印一些信息出来，方便调试。发现填写的api 和数据结构错误，很快的解决了。


不得不说，通义的补全速度很快，可能是网络原因，比copilot 快起码两倍以上，这点对写代码很重要。

# 结尾

最后，我有个非常困难的需求：将使用github action 将项目自动化打包。我本人也不是很懂这个github action
![QQ_1729847406897.png](https://ucc.alicdn.com/pic/developer-ecology/plzpj6bp7mjr4_8bd908d0b6414a318bb9727f5f2565bd.png)

qwen 也是轻松解决。

现在提交代码，完成


免费又好用，还不赶紧用起来。