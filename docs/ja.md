# 2D Shell — すべての 2D ゲームを描くための枠

2D Shell は、2D ゲームが中身を満たす空のスクリーンです。現在の場所の絵、上の時計とツールバー、パネルの置き場所、通知の隅から成ります。単独では何も表示しません。コンテンツ MOD がそれぞれのレイヤーを重ねます。Dialog Engine はビジュアルノベルのテキストボックスと選択肢を、RPG MOD は歩き回れるタイルマップを追加します。ノベルもタイルマップの RPG も、同じ枠、同じツールバー、同じメニューを共有するのです。

すべての 2D ゲームがこれを含みます。Cthangover、My Novel、RPG Plus です。

![二つのゲームで同じシェル:テキストボックスと選択肢つきの Cthangover のシーンと、タイルマップつきの RPG Plus の村。どちらも時計が左上、ツールバーが右上にあります。](assets/two-games.jpg)

## 画面に見えるもの

![Cthangover のシーン:台所の背景、左上の時計と配給カウンター、右上のツールバー、選択肢「Leave」「Cook」、下のテキストボックス。](assets/novel-scene.jpg)

このシーンでシェルが担っているのは:

- **場所の絵** — 台所。時刻に応じた色調と照明つき(Time Engine);
- **上のバー** — 左上の時計、右上のツールバー、中央のセーブインジケーター、夜には携帯ランプ;
- **通知の隅** — 時計の下;
- **パネルの空き** — インベントリ、マップ、日誌はシーンの上に開きます。

テキストボックスと選択肢は Dialog Engine から、配給カウンターは Cooking から来ています。どちらも各 MOD がシェルに重ねるレイヤーです。

## レイヤー:後ろから前へ

```text
BaseScene
└── Scene
    ├── ModFirstPanel        すべての後ろに置かれる MOD のレイヤー
    ├── ViewBox              場所そのもの:照明、背景、前景、Lastground
    ├── (content layer)      コンテンツ MOD が追加するレイヤー。例:ダイアログのキャンバスや RPG の世界
    ├── TopView              時計、ツールバー、セーブインジケーター、ランプ
    ├── ModBeforeUIPanel     ツールバーのパネル:インベントリ、マップ、日誌、メニュー…
    ├── NotificationPanel    左上隅のメッセージ
    └── ModLastPanel         すべての上に重なるオーバーレイ。例:作業台のウィンドウ
```

後のレイヤーほど手前に描かれます。

## MOD 作者向け

**シーンパッチでコンテンツレイヤーを追加する。** パッチとは、MOD の `patches/` フォルダーにシェルのシーンと同じパスで置く `.tscn` のことです。シェルのシーンをインスタンス化し、ノードを追加・変更します。Dialog Engine のパッチは、ダイアログのキャンバスを ViewBox の直後に挿入します:

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

RPG MOD も同じやり方で `RpgWorldHost` を足します。両方を含むゲームではタイルマップが表示され、その上にセリフが重なります。

**パッチなしでパネルを追加する。** ツールバーのパネルやオーバーレイは、manifest の `ui` セクションでシェルのスロットの一つ(`ModBeforeUIPanel`、`ModLastPanel`、ビューボックスの `Lastground`)を指定して宣言します。フィールドの詳細は UI MOD のガイドを見てください。

**依存関係を宣言する。** 2D の枠を必要とする MOD は `depends` に `shell2d_base` を書きます。シェル自身が Time Engine と UI を連れてきます。

## うまく動かないとき

1. **ツールバーだけの真っ黒な画面。** シェルを満たすコンテンツ MOD がありません(Dialog Engine か RPG MOD がゲームに含まれていない)、またはシーンに背景がありません。
2. **パネルがシーンの後ろに表示される。** その manifest が、描画が早すぎるスロットを指定しています。`ModBeforeUIPanel` か `ModLastPanel` を使ってください。
