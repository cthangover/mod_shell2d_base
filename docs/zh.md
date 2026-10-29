# 2D Shell — 每一款 2D 游戏都在其中绘制的框架

2D Shell 是一块空白的屏幕，等 2D 游戏来填充：当前地点的画面、顶部的时钟和工具栏、面板的容身之处，以及通知一角。它自己什么都不显示。内容模组把自己的图层放进这里——Dialog Engine 加上视觉小说的文本框和选项，RPG 模组加上可行走的图块地图——因此小说游戏和图块 RPG 共用同一个框架、同一条工具栏和同一套菜单。

每款 2D 游戏都包含它：Cthangover、My Novel 和 RPG Plus。

![同一个外壳出现在两款游戏中：带文本框和选项的 Cthangover 场景，带图块地图的 RPG Plus 村庄；两者的时钟都在左上角，工具栏都在右上角。](assets/two-games.jpg)

## 屏幕上看到的东西

![一幕 Cthangover 场景：厨房背景，左上角是时钟和口粮计数器，右上角是工具栏，选项为“Leave”和“Cook”，底部是文本框。](assets/novel-scene.jpg)

在这一幕中，外壳提供了：

- **地点的画面** — 厨房，带有随时间变化的色调和灯光（Time Engine）；
- **顶栏** — 左上角时钟、右上角工具栏，中间是存档指示器，夜间还有便携提灯；
- **通知一角** — 时钟下方；
- **面板空间** — 背包、地图或日志会叠在场景上方打开。

文本框和选项来自 Dialog Engine，口粮计数器来自 Cooking——它们都是这些模组加到外壳上的图层。

## 图层：从后到前

```text
BaseScene
└── Scene
    ├── ModFirstPanel        各模组垫在最后面的图层
    ├── ViewBox              场景本身：灯光、背景、前景、Lastground
    ├── (content layer)      内容模组添加的图层，例如对话框画布或 RPG 世界
    ├── TopView              时钟、工具栏、存档指示器、提灯
    ├── ModBeforeUIPanel     工具栏面板：背包、地图、日志、菜单…
    ├── NotificationPanel    左上角的消息
    └── ModLastPanel         盖在一切之上的覆盖层，例如工作台窗口
```

靠后的图层绘制在靠前图层之上。

## 给模组作者

**用场景补丁添加内容图层。** 补丁是放在你模组 `patches/` 文件夹里的一个 `.tscn`，路径与外壳场景相同。它实例化外壳场景并增加或修改节点。Dialog Engine 的补丁就是把对话框画布插到 ViewBox 之后：

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

RPG 模组用同样的方式添加自己的 `RpgWorldHost`，因此两者齐全的游戏会显示图块地图，其上再叠加对话文本。

**不用补丁添加面板。** 工具栏面板或覆盖层在 manifest 的 `ui` 一节中声明，并使用外壳的某个槽位：`ModBeforeUIPanel`、`ModLastPanel`，或视图框的 `Lastground`。字段说明见 UI 模组的指南。

**声明依赖。** 需要 2D 框架的模组应在 `depends` 中列出 `shell2d_base`；外壳本身会带上 Time Engine 和 UI。

## 出现问题时

1. **黑屏，只有工具栏。** 没有内容模组填充外壳（游戏缺少 Dialog Engine 或 RPG 模组），或者场景没有背景。
2. **面板出现在场景后面。** 它的 manifest 用了绘制过早的槽位；改用 `ModBeforeUIPanel` 或 `ModLastPanel`。
