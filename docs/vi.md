# 2D Shell — khung vẽ của mọi game 2D

2D Shell là màn hình trống mà các game 2D lấp đầy: bức tranh về nơi đang đứng, đồng hồ và thanh công cụ ở trên, chỗ cho các bảng điều khiển, và góc thông báo. Tự nó nó không hiện gì cả. Các mod nội dung đặt lớp của mình lên đó — Dialog Engine thêm hộp thoại và các lựa chọn của visual novel, còn mod RPG thêm bản đồ tile đi lại được — nên một game novel và một game tile-map RPG cùng dùng một khung, một thanh công cụ và một bộ menu.

2D Shell có mặt trong mọi game 2D: Cthangover, My Novel và RPG Plus.

![Cùng một lớp vỏ trong hai game: cảnh Cthangover với hộp thoại và các lựa chọn, và làng của RPG Plus với bản đồ tile; cả hai đều có đồng hồ ở góc trên trái và thanh công cụ ở góc trên phải.](assets/two-games.jpg)

## Bạn thấy gì trên màn hình

![Một cảnh Cthangover: nền căn bếp, đồng hồ và bộ đếm khẩu phần trên bên trái, thanh công cụ trên bên phải, hai lựa chọn "Leave" và "Cook", và hộp văn bản ở dưới.](assets/novel-scene.jpg)

Trong cảnh này, lớp vỏ cung cấp:

- **bức tranh nơi chốn** — căn bếp, kèm sắc màu theo giờ và ánh đèn (Time Engine);
- **thanh trên cùng** — đồng hồ góc trên trái, thanh công cụ góc trên phải, ở giữa là biểu tượng lưu và đèn cầm tay ban đêm;
- **góc thông báo** dưới đồng hồ;
- **chỗ cho bảng điều khiển** — túi đồ, bản đồ hay nhật ký mở đè lên cảnh.

Hộp văn bản và các lựa chọn đến từ Dialog Engine, bộ đếm khẩu phần đến từ Cooking — đây là các lớp mà những mod này đặt lên vỏ.

## Các lớp, từ sau ra trước

```text
BaseScene
└── Scene
    ├── ModFirstPanel        các lớp của mod nằm sau tất cả
    ├── ViewBox              chính là nơi chốn: đèn, nền, tiền cảnh, Lastground
    ├── (content layer)      lớp do mod nội dung thêm vào, ví dụ canvas hội thoại hay thế giới RPG
    ├── TopView              đồng hồ, thanh công cụ, biểu tượng lưu, đèn
    ├── ModBeforeUIPanel     các bảng trên thanh công cụ: túi đồ, bản đồ, nhật ký, menu…
    ├── NotificationPanel    tin nhắn ở góc trên bên trái
    └── ModLastPanel         lớp phủ trên cùng tất cả, ví dụ cửa sổ bàn chế tạo
```

Lớp nào vẽ sau sẽ đè lên lớp vẽ trước.

## Cho tác giả mod

**Thêm lớp nội dung bằng bản vá cảnh.** Bản vá (patch) là một tệp `.tscn` trong thư mục `patches/` của mod bạn, đặt cùng đường dẫn với cảnh của lớp vỏ. Nó khởi tạo cảnh của vỏ rồi thêm hoặc sửa node. Patch của Dialog Engine chèn canvas hội thoại ngay sau ViewBox:

```text
patches/scenes/ui/base_scene.tscn
```

```text
[ext_resource type="PackedScene" path="res://mods/shell2d_base/scenes/ui/base_scene.tscn" id="1_target"]
[ext_resource type="PackedScene" path="res://scenes/ui/dialog_canvas.tscn" id="3_dialog"]

[node name="BaseScene" instance=ExtResource("1_target")]

[node name="DialogCanvas" parent="Scene" index="2" instance=ExtResource("3_dialog")]
```

Mod RPG cũng thêm `RpgWorldHost` của mình theo cách đó, nên game có cả hai sẽ hiện bản đồ tile, phía trên là lời thoại.

**Thêm bảng điều khiển không cần patch.** Bảng trên thanh công cụ hay lớp phủ được khai báo trong mục `ui` của manifest với một trong các khe của vỏ: `ModBeforeUIPanel`, `ModLastPanel` hoặc `Lastground` của khung xem. Các trường xem trong hướng dẫn của mod UI.

**Khai báo phụ thuộc.** Mod cần khung 2D hãy liệt kê `shell2d_base` trong `depends`; bản thân lớp vỏ kéo theo Time Engine và UI.

## Khi có trục trặc

1. **Màn hình đen, chỉ còn thanh công cụ.** Không mod nội dung nào đang lấp đầy lớp vỏ (thiếu Dialog Engine hoặc mod RPG trong game), hoặc cảnh không có nền.
2. **Bảng điều khiển hiện sau cảnh.** Manifest của nó nêu một khe bị vẽ quá sớm; hãy dùng `ModBeforeUIPanel` hoặc `ModLastPanel`.
