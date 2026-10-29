# 2D Shell — 모든 2D 게임이 그려지는 틀

2D Shell은 2D 게임이 채워 넣는 빈 화면입니다. 현재 장소의 그림, 위쪽의 시계와 툴바, 패널이 들어갈 자리, 알림 구석으로 이루어집니다. 스스로는 아무것도 보여 주지 않습니다. 콘텐츠 모드가 각자의 레이어를 얹습니다. Dialog Engine은 비주얼 노벨의 텍스트 상자와 선택지를, RPG 모드는 걸어 다닐 수 있는 타일 맵을 더합니다. 덕분에 노벨 게임과 타일 맵 RPG가 같은 틀, 같은 툴바, 같은 메뉴를 공유합니다.

모든 2D 게임에 들어 있습니다. Cthangover, My Novel, RPG Plus가 그렇습니다.

![두 게임에서 같은 셸: 텍스트 상자와 선택지가 있는 Cthangover 장면, 타일 맵이 있는 RPG Plus 마을. 둘 다 시계는 왼쪽 위, 툴바는 오른쪽 위에 있습니다.](assets/two-games.jpg)

## 화면에 보이는 것들

![Cthangover 장면: 부엌 배경, 왼쪽 위 시계와 배급 카운터, 오른쪽 위 툴바, Leave와 Cook 선택지, 아래 텍스트 상자.](assets/novel-scene.jpg)

이 장면에서 셸이 제공하는 것:

- **장소의 그림** — 부엌. 시간대에 따른 색조와 조명 포함(Time Engine);
- **위쪽 바** — 왼쪽 위 시계, 오른쪽 위 툴바, 가운데 저장 표시, 밤에는 휴대용 램프;
- **알림 구석** — 시계 아래;
- **패널 자리** — 인벤토리, 지도, 일지는 장면 위로 열립니다.

텍스트 상자와 선택지는 Dialog Engine이, 배급 카운터는 Cooking이 가져옵니다. 이것들은 해당 모드들이 셸에 얹는 레이어입니다.

## 레이어: 뒤에서 앞으로

```text
BaseScene
└── Scene
    ├── ModFirstPanel        모든 것 뒤에 깔리는 모드 레이어
    ├── ViewBox              장면 그 자체: 조명, 배경, 전경, Lastground
    ├── (content layer)      콘텐츠 모드가 추가하는 레이어. 예: 대화 캔버스나 RPG 세계
    ├── TopView              시계, 툴바, 저장 표시, 램프
    ├── ModBeforeUIPanel     툴바 패널: 인벤토리, 지도, 일지, 메뉴…
    ├── NotificationPanel    왼쪽 위 알림 메시지
    └── ModLastPanel         모든 것 위에 겹쳐지는 오버레이. 예: 작업대 창
```

나중 레이어가 먼저 그려진 레이어 위에 놓입니다.

## 모드 제작자를 위해

**장면 패치로 콘텐츠 레이어 추가하기.** 패치는 모드의 `patches/` 폴더에 셸 장면과 같은 경로로 두는 `.tscn`입니다. 셸 장면을 인스턴스화하고 노드를 더하거나 바꿉니다. Dialog Engine의 패치는 대화 캔버스를 ViewBox 바로 뒤에 삽입합니다:

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

RPG 모드도 같은 방식으로 자기 `RpgWorldHost`를 더합니다. 둘 다 있는 게임에서는 타일 맵 위로 대사가 겹쳐 보입니다.

**패치 없이 패널 추가하기.** 툴바 패널이나 오버레이는 manifest의 `ui` 섹션에 셸 슬롯 하나(`ModBeforeUIPanel`, `ModLastPanel`, 뷰 박스의 `Lastground`)를 지정해 선언합니다. 필드는 UI 모드 가이드를 참고하세요.

**의존성 선언하기.** 2D 틀이 필요한 모드는 `depends`에 `shell2d_base`를 적습니다. 셸 자체가 Time Engine과 UI를 데려옵니다.

## 문제가 생겼을 때

1. **툴바만 있는 검은 화면.** 셸을 채우는 콘텐츠 모드가 없는 것입니다(게임에 Dialog Engine이나 RPG 모드가 없음). 장면에 배경이 없을 수도 있습니다.
2. **패널이 장면 뒤에 나타난다.** 그 manifest이 너무 일찍 그려지는 슬롯을 지정한 것입니다. `ModBeforeUIPanel`이나 `ModLastPanel`을 쓰세요.
