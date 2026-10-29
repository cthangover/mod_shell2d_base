# 2D Shell — the frame every 2D game is drawn in

2D Shell is the empty screen that 2D games fill: the picture of the current place, the clock and toolbar at the top, room for panels, and the notifications corner. It shows nothing by itself. Content mods put their own layers into it — the Dialog Engine adds the text box and choices of a visual novel, the RPG mod adds the walkable tile map — so a novel and a tile-map RPG share the same frame, the same toolbar and the same menus.

Every 2D game includes it: Cthangover, My Novel and RPG Plus.

![The same shell in two games: a Cthangover scene with the dialog box and choices, and the RPG Plus village with the tile map; both have the clock at the top left and the toolbar at the top right.](assets/two-games.jpg)

## What you see on screen

![A Cthangover scene: the kitchen background, the clock and a ration counter at the top left, the toolbar at the top right, the choices "Leave" and "Cook", and the text box at the bottom.](assets/novel-scene.jpg)

In this scene, the shell provides:

- **the picture of the place** — the kitchen, with its time-of-day shading and lights (Time Engine);
- **the top bar** — the clock at the top left, the toolbar at the top right, and in the middle the save indicator and, at night, the portable lamp;
- **the notifications corner** under the clock;
- **room for panels** — the bag, map or journal open above the scene.

The text box and the choices come from the Dialog Engine, and the ration counter from Cooking — they are layers that those mods add to the shell.

## Layers, from back to front

```text
BaseScene
└── Scene
    ├── ModFirstPanel        mods' layers behind everything
    ├── ViewBox              the place: lights, background, foreground, last ground
    ├── (content layer)      added by a content mod, e.g. the dialog canvas or the RPG world
    ├── TopView              clock, toolbar, save indicator, lamp
    ├── ModBeforeUIPanel     toolbar panels: bag, map, journal, menu…
    ├── NotificationPanel    messages in the top-left corner
    └── ModLastPanel         overlays above all, e.g. the crafting window
```

Later layers are drawn over earlier ones.

## For mod authors

**Add a content layer with a scene patch.** A patch is a `.tscn` in your mod's `patches/` folder at the same path as the shell's scene. It instances the shell's scene and adds or changes nodes. The Dialog Engine's patch adds the dialog canvas right after the view box:

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

The RPG mod adds its `RpgWorldHost` the same way, so a game with both shows the tile map and, over it, dialog lines.

**Add a panel without a patch.** A toolbar panel or an overlay is declared in your manifest's `ui` section with one of the shell's slots: `ModBeforeUIPanel`, `ModLastPanel` or the view box's `Lastground`. See the UI mod's guide for the fields.

**Depend on it.** A mod that needs the 2D frame lists `shell2d_base` in `depends`; the shell itself brings Time Engine and UI.

## If something goes wrong

1. **A black screen with only the toolbar.** No content mod is filling the shell (a Dialog Engine or RPG mod is missing from the game), or the scene has no background.
2. **A panel appears behind the scene.** Its manifest names a slot that is drawn too early; use `ModBeforeUIPanel` or `ModLastPanel`.
