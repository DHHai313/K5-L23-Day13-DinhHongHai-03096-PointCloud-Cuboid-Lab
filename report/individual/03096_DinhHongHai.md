# Báo cáo Day 13 — Đinh Hồng Hải, 03096

## 1. Kết luận trước

- **Thông tin chung:** Mã số `03096`, task `D13-03096`; 10 frame được giao gồm `s01f35, s01f55, s02f06, s02f55, s03f23, s03f49, s04f43, s04f59, s05f12, s05f42`. Hệ điều hành Windows 11, trình duyệt Chrome, CVAT phiên bản `2.81.0` (kiểm tra trong tài khoản → About); làm cá nhân độc lập (không có bạn ngồi cạnh).
- **Tổng kết kết quả:** Đã vẽ tổng cộng **68 cuboid** trên 10 frame (`s01f35`: 3, `s01f55`: 7, `s02f06`: 10, `s02f55`: 6, `s03f23`: 11, `s03f49`: 7, `s04f43`: 6, `s04f59`: 4, `s05f12`: 7, `s05f42`: 7). Frame ít cuboid nhất là `s01f35` (3 cuboid) do đường đại lộ khá vắng phương tiện trong bán kính 50 m (chỉ có 2 xe con ở gần, các xe khác ở rất xa trên 50 m và xe hai bánh trên vỉa hè không có người lái). Công cụ `selfcheck` gắn cảnh báo cho **22/68 cuboid** (chủ yếu là `width` hơi rộng hơn khoảng chuẩn ở một số xe buýt/xe con và một vài hộp có đáy hơi chênh so với mặt đường). Phần mình tự tin nhất là xác định hướng đầu xe (yaw) cho các xe chạy cùng chiều/ngược chiều và gán đúng class xe con, xe buýt; phần còn yếu nhất là ước lượng độ rộng `width` cho xe cỡ lớn ở góc nhìn chéo và căn độ cao đáy cho các vật ở xa trên 30 m.
- **Cách dùng CVAT (3 thao tác chính):**
  1. Đổi góc nhìn trong khung 3D (Perspective): kéo chuột trái để xoay (xoay thẳng từ trên xuống để quan sát toàn cảnh 360°), kéo chuột phải để dời khung nhìn, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc nhìn mặc định và bấm biểu tượng bánh răng trên từng khung ảnh bên phải để chuyển đủ 7 camera.
  2. Vẽ và chỉnh cuboid: chọn công cụ vẽ cuboid bên trái → chọn đúng label → bấm **Shape** (không bấm Track) → nháy đúp chuột vào cụm điểm để đặt hộp, sau đó chọn hộp để chỉnh vị trí, dài/rộng và góc yaw ở khung **Top**, chỉnh độ cao và căn đáy chạm mặt đường ở **Side** và **Front** (không xoay ở Side/Front).
  3. Bật mũi tên hướng và kiểm tra trên ảnh camera: tích **Cuboid orientation** trong mục **Appearance** để căn vạch đỏ ngắn (`+X`) chỉ về đầu xe, bấm Save (`Ctrl+S`), rồi chạy `python scripts/lab.py preview` (hoặc `sync`) để xem cuboid chiếu lên cả 7 ảnh camera tại `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ: `s01f35` và `s01f55`. Ảnh chụp: `t1-s01f35-3d.png`, `t1-s01f35-top.png`, `t1-s01f55-3d.png`, `t1-s01f55-top.png` (trên hai ảnh nhìn từ trên xuống có đánh dấu phía trước `-y`, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; `+x`, `+y` chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f35` | Phía trước (`-y`) ở **cạnh trên**; `+x` chỉ sang **trái**, `+y` chỉ **xuống dưới** | Khá phẳng (đại lộ nhiều làn thẳng); thấp hơn LiDAR khoảng `1,7 m` (`z ≈ -1,68 m`) | Khoảng `45–50 m` (các xe con chạy xa phía trước trên `cam03`) | Xe con (`Car`), xe hai bánh dựng vỉa hè không có người | 1 xe con trắng bám ngay phía sau cách khoảng `10,5 m` (thấy trên `cam09`, `cam05`, không có trong `cam01`); 1 xe con trắng phía trước bên trái cách khoảng `9,3 m` (`cam06`) | Cụm điểm rải rác phía bên kia dải phân cách cây xanh bên trái cách khoảng `30–40 m`, bị tán cây che nhiều nên không chắc là xe đỗ hay kết cấu công trình |
| `s01f55` | Phía trước (`-y`) ở **cạnh trên**; `+x` chỉ sang **trái**, `+y` chỉ **xuống dưới** | Khá phẳng (đoạn gần nút giao nhiều làn); thấp hơn LiDAR khoảng `1,6 m` (`z ≈ -1,58 m`) | Khoảng `40–45 m` (dãy xe nối đuôi phía trước ở làn chờ đèn) | Nhiều xe con (`Car`) cùng chiều, người đi bộ/xe hai bánh ở vỉa hè xa phía trước bên phải | 2 xe con trắng bám ngay phía sau và phía sau bên trái cách khoảng `7,2 m` và `7,6 m` (chỉ thấy trên `cam09` và `cam05`, không có trong `cam01`) | Cụm điểm dày dọc hàng cây và vỉa hè bên phải cách `10–20 m` (`cam07`), nhiều xe đạp dựng san sát lẫn với gốc cây và người đứng khuất, khó tách từng vật trên point cloud |

So sánh thông tin giữa camera và LiDAR (từ hai frame `s01f35`, `s01f55`):
- **3 điều chỉ camera cho biết:**
  1. Màu sắc, đặc điểm nhận diện vật thể và phân biệt vật thật với vật tĩnh (ví dụ phân biệt xe trắng/xám, nhận biết biển quảng cáo trên dải phân cách bên trái `s01f35` không phải thùng xe tải, và thấy rõ xe hai bánh trên vỉa hè không có người ngồi).
  2. Vạch kẻ làn đường, mũi tên chỉ hướng di chuyển trên mặt đường và hệ thống biển chỉ dẫn/tín hiệu giao thông treo phía trên (`cam01`, `cam03`).
  3. Phân biệt chắc chắn đầu xe với đuôi xe (nhờ đèn pha, lưới tản nhiệt và đèn hậu) khi xe ở xa hoặc bị che một phần.
- **3 điều chỉ LiDAR cho biết:**
  1. Khoảng cách chính xác theo đơn vị mét từ cảm biến đến từng vật thể (như hai xe phía sau ở `s01f55` cách đúng `7,17 m` và `7,62 m`, xe bên trái `s01f35` cách `9,34 m`).
  2. Kích thước hình học 3D thực tế dài × rộng × cao theo mét và độ cao mặt đường quanh xe (`z ≈ -1,6 m` đến `-1,7 m`).
  3. Tầm nhìn toàn cảnh 360° liên tục không có góc mù giữa các khung ảnh, giúp định vị vật nằm ở vùng chuyển tiếp hoặc góc khuất giữa hai camera.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f06` | Cạnh trên | Nút giao lớn có nhiều xe buýt đi ngược chiều bên trái và xe đi cắt ngang phía trước bên phải; có cụm điểm nằm sâu dưới mặt đường phía trước bên phải |
| `s02f55` | Cạnh trên | Đường hẹp hai chiều có hàng rào giữa, hai bên nhiều cây cối dày và sườn đất dốc thấp bên phải |
| `s03f23` | Cạnh trên | Có trạm xe buýt bên phải, người đi bộ ở khu vực phía sau bên phải và xe đỗ trong bãi cao hơn mặt đường chính |
| `s03f49` | Cạnh trên | Xe phía trước khá dày, có người đi xe đạp sát rào chắn bên phải (`cam07`/`cam08`) và xe buýt ở xa phía trước bên trái |
| `s04f43` | Cạnh trên | Có hàng rào vàng bên trái và tường quây công trình cao dọc bên phải tạo vùng bóng che rộng phía sau tường |
| `s04f59` | Cạnh trên | Đoạn đường dọc tường quây công trình bên phải, mật độ xe trong bán kính `30 m` thưa hơn (4 xe rõ) |
| `s05f12` | Cạnh trên | Ngã tư rộng có xe ngược chiều bên trái và xe tải phía sau; có cụm điểm lạ nằm sâu dưới mặt đường (`z ≈ -10,5 m`) ở phía trước bên trái |
| `s05f42` | Cạnh trên | Xe van trắng và SUV đen ở rất gần bên trái (`3,8–5,8 m`) tạo vùng bóng che lớn kéo dài phía sau chúng; có xe buýt bám sau |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f35`, `#1` | `Car` | Đúng (`yaw_deg = 273,8°`, cùng chiều phía trước) | Chạm đường (`z_bottom = -1,795 m`, `ground_gap_m = -0,02 m`) | Đã kiểm Top/Side/Front và `cam01`/`cam06` | `l = 4,572 m`, `w = 2,169 m`, `h = 1,677 m` (`l`, `h` trong khoảng Car `3,8–5,3 m` và `1,40–2,10 m`; `w` nhỉnh `0,07 m` so với mức `2,10 m`) |
| (b) xe buýt hoặc xe tải | `s02f06`, `#3` | `Bus` | Đúng (`yaw_deg = 85,2°`, ngược chiều phía trước bên trái) | Chạm đường (`z_bottom = -2,009 m`, `ground_gap_m = +0,06 m`) | Đã kiểm Top/Side/Front và `cam01`/`cam03`/`cam06` | `l = 11,692 m`, `w = 3,358 m`, `h = 3,374 m` (`l`, `h` trong bảng 4.3 của xe buýt thường `10,0–12,5 m` và `3,0–3,5 m`; `w` rộng hơn chuẩn `2,5–2,6 m` nên bị cờ cần thu gọn) |
| (c) người đi bộ hoặc xe hai bánh | `s03f49`, `#4` | `Cyclist` | Đúng (`yaw_deg = 271,3°`, đi cùng chiều bên phải) | Chạm đường (`z_bottom = -1,763 m`, `ground_gap_m = 0,00 m`) | Đã kiểm Top/Side/Front và `cam07`/`cam08` | `l = 1,628 m`, `w = 0,636 m`, `h = 1,676 m` (cả 3 chiều nằm gọn trong bảng 4.3 của xe đạp có người: dài `1,6–1,9 m`, rộng `0,50–0,70 m`, cao `1,60–1,90 m`) |

- **Vòng hiệu chuẩn:** Bỏ qua phần tạo task chung theo hướng dẫn của lab coach trong buổi học. *(Ghi chú chọn mẫu: do frame `s01f35` và `s01f55` không có xe buýt/xe tải thật và không có xe hai bánh có người lái rõ trong tầm, mẫu (b) lấy từ frame `s02f06` và mẫu (c) lấy từ frame `s03f49`.)*
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f35`: dãy xe hai bánh dựng trên vỉa hè bên phải (`cam07`/`cam08`, cách khoảng `6–15 m`) không có người lái nên không thuộc `Cyclist` theo guideline; các xe con xa phía trước ngoài `50 m` trên `cam03` quá thưa điểm (`LOW_VISIBILITY`, `UNCERTAIN_SCOPE`).
  - `s01f55`: các xe con đi xa hơn phía trước bị hàng xe gần `#1`, `#2`, `#3` che gần hết thân (`cam03`) và cụm người đứng lẫn trong hàng xe đạp dựng sát tán cây trên vỉa hè bên phải (`cam07`) không nhìn đủ 15–20% thể tích (`LOW_VISIBILITY`).
  - `s04f59`: nhóm xe hai bánh đi sát tường quây phía bên kia rào chắn bên trái (`cam06`) bị rào che khuất nhiều và quá ít điểm LiDAR (`LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s05f42` / `t4-1-bong-che.png` / frame cá nhân | **Giả thuyết:** các xe lớn ở rất gần bên trái (`#1` xe van trắng cách `5,80 m`, `#4` xe SUV đen cách `3,76 m`) và xe buýt `#7` phía sau (`21,16 m`) chặn chùm tia LiDAR không xuyên qua được | Xuất hiện các hình quạt tối trống hoàn toàn phía sau thân các xe này, trong khi hai bên ngoài góc che mật độ điểm mặt đường vẫn dày | Vùng trống là khu vực chưa quan sát được chứ không phải đường trống; với vật bị che khuất một phần nằm ở rìa vùng bóng che cần bật `occluded` và vẽ cuboid bao trọn phần suy luận được |
| 2. Thưa ở xa | `s02f55` / `t4-2-thua-o-xa.png` / frame cá nhân | **Giả thuyết:** các tia của LiDAR 40 kênh phân kỳ theo góc khi truyền xa làm khoảng cách giữa các tia tăng nhanh và số tia trúng một thân xe giảm mạnh theo khoảng cách | Trên cùng làn đường phía sau, xe con `#4` ở `34,76 m` chỉ còn `125` điểm và xe con `#5` ở `52,80 m` chỉ còn `50` điểm rời rạc (so với hơn `1000` điểm ở `10–13 m`) | Chỉ vẽ khi thấy khoảng 15–20% thể tích (hoặc đủ cơ sở suy luận từ camera) và nằm trong phạm vi `≤ 50 m`; nếu quá thưa không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s04f43` / `t4-3-phan-xa-thap.png` / frame cá nhân | **Giả thuyết:** lớp sơn tối màu và mặt kính cửa sổ hấp thụ tia hồng ngoại hoặc phản xạ gương lệch hướng cảm biến khiến cường độ hồi tia thấp và rớt điểm | Ở cùng khoảng cách gần `~7 m` trong cùng một frame, xe sedan màu đỏ sẫm `#2` (`dist_m = 7,20 m`) chỉ thu được `1907` điểm (mật độ ở vùng kính và thân sau thưa hơn rõ rệt) so với xe SUV trắng `#1` ngay cạnh (`dist_m = 6,47 m`, `3341` điểm) | Ghi nguyên nhân ở mức giả thuyết (không khẳng định); đối chiếu ảnh camera (`cam01`/`cam07`) để căn đủ thân xe thay vì thu nhỏ theo vùng rớt điểm, đồng thời không nới hộp theo quầng sáng cường độ cao |
| 4. Điểm lạ | `s05f12` / `t4-4-diem-la.png` / frame cá nhân | **Giả thuyết:** phản xạ đa đường (multipath) khi tia LiDAR dội qua bề mặt kính xe hoặc mặt đường bóng tại ngã tư làm kéo dài thời gian bay và tính sai vị trí điểm xuống dưới mặt đất | Có một cụm `10` điểm nằm cô lập ở phía trước bên trái, cách khoảng `22,6 m`, tại độ cao `z ≈ -10,5 m` — thấp hơn mặt đường phẳng quanh đó (`z ≈ -1,7 m`) gần `8,8 m` | Không gán các cụm điểm lạ thành vật thể và không kéo đáy cuboid xuống bao các điểm nhiễu dưới mặt đất |

## 5. Số từ cuboid của tôi (Bước 5)

Các số liệu dưới đây lấy trực tiếp từ `submission/D13-03096-bangA.csv` và `submission/D13-03096-tukiem.csv`.

| Cuboid (frame, idx, class) | `z` | `h` | `z đáy = z − h/2` | `dist_m` | `r = √(x²+y²+z²)` | `yaw_deg` | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f55`, `#2`, `Car` | `-1,022 m` | `1,536 m` | `-1,790 m` | `9,53 m` | `9,58 m` | `269,5°` | Chạm đường (sát mức `-1,7 m`, `ground_gap_m = +0,02 m`) |
| `s03f23`, `#5`, `Car` | `-0,101 m` | `1,484 m` | `-0,843 m` | `35,34 m` | `35,34 m` | `17,1°` | Lơ lửng so với mặt đường chính (cao hơn `-1,7 m` khoảng `0,86 m`, `ground_gap_m = +0,46 m`; thực tế xe đỗ trên bãi/vỉa hè cao phía sau bên phải) |
| `s05f42`, `#3`, `Car` | `-1,210 m` | `1,941 m` | `-2,181 m` | `14,15 m` | `14,20 m` | `277,7°` | Chìm (thấp hơn `-1,7 m` khoảng `0,48 m`, `ground_gap_m = -0,32 m`; cần nâng đáy lên ở Side/Front) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s05f42`, `#4` (`Car`, xe SUV đen sát bên trái) có `dist_m = 3,76 m` và `r = 3,84 m`.
  - Xa nhất trong toàn bảng A: `s02f55`, `#5` (`Car`, xe ngược chiều phía sau bên trái) có `dist_m = 52,80 m` và `r = 52,80 m` (vượt ranh giới `50 m` nên không được tính điểm). Nếu xét riêng trong phạm vi chấm `≤ 50 m`, cuboid xa nhất là `s02f06`, `#10` (`Car`) có `dist_m = 45,26 m` và `r = 45,28 m`.
- **(b) So kích thước với bảng 4.3:**
  - Người đi bộ (`s03f23`, `#3`, `Pedestrian`): `l = 0,903 m`, `w = 0,849 m`, `h = 2,089 m`. So với bảng 4.3 của người lớn (`l`: `0,4–0,9 m`, `w`: `0,5–0,9 m`, `h`: `1,50–1,95 m`), chiều rộng `w` nằm trong khoảng, chiều dài `l` chỉ nhỉnh `0,003 m`, còn chiều cao `h` vượt khoảng `0,14 m` do người ở xa (`23,94 m`, `81` điểm) nên phần đỉnh hộp bị kéo hơi rộng so với đầu người. (Với xe hai bánh `s03f49`, `#4`, `Cyclist`: `l = 1,628 m`, `w = 0,636 m`, `h = 1,676 m` — cả 3 chiều nằm hoàn toàn trong khoảng chuẩn của xe đạp có người.)
  - Xe con (`s01f55`, `#2`, `Car`, sedan màu trắng): `l = 5,179 m`, `w = 1,697 m`, `h = 1,536 m`. So với bảng 4.3 của sedan/hatchback (`l`: `3,8–5,0 m`, `w`: `1,70–1,90 m`, `h`: `1,40–1,60 m`), chiều cao `h` nằm chuẩn trong khoảng, chiều rộng `w` sát ngưỡng dưới (`1,70 m`), còn chiều dài `l` vượt nhẹ `0,18 m` do kéo hộp hơi dư một chút ở phần đầu/đuôi xe.
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước (`s03f49`, `#1`, `Car`): `yaw_deg = 270,2°` — rất sát góc lý thuyết `270°` (đầu xe hướng theo trục `-y`).
  - Xe ngược chiều (`s02f55`, `#1`, `Car`): `yaw_deg = 90,8°` — rất sát góc lý thuyết `90°` (đầu xe hướng ngược lại theo trục `+y`).

Mật độ điểm theo khoảng cách (4 cuboid lớp `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | `dist_m` | Số điểm (`points_in`) |
|---|---|---|
| `s01f35`, `#1`, `Car` | `9,34 m` | `1847` |
| `s02f06`, `#4`, `Car` | `17,53 m` | `509` |
| `s03f23`, `#1`, `Car` | `23,08 m` | `271` |
| `s03f23`, `#2`, `Car` | `39,29 m` | `85` |

- **Quy luật mật độ và so với slide bài giảng:** Khi khoảng cách tăng gấp ~`1,9` lần (từ `9,34 m` lên `17,53 m`), số điểm trên một xe con giảm từ `1847` xuống `509` (còn khoảng `27,6%`, tức giảm ~`3,6` lần); khi xa gấp ~`4,2` lần (lên `39,29 m`), số điểm chỉ còn `85` (khoảng `4,6%`, giảm gần `22` lần), xấp xỉ quy luật giảm theo nghịch đảo bình phương khoảng cách và bám rất sát đường cong trung vị trên slide (`1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`).
- **Tự kiểm sau `preview` và `selfcheck`:**
  - Trong quá trình vẽ và soát lại trên CVAT trước khi xuất bảng A hiện tại, đã chỉnh hướng yaw ngược và căn lại đáy cho `s02f06`, `#3` (`Bus`, căn đáy về `ground_gap_m = +0,06 m` và hướng ngược chiều `85,2°`) và `s03f49`, `#4` (`Cyclist`, căn sát cụm điểm về `ground_gap_m = 0,00 m` và `yaw_deg = 271,3°`). Sau khi chạy `sync` xem `work/xem/` và `-tukiem.csv`, cũng phát hiện thêm một số hộp cần lưu ý sửa tiếp trên CVAT (như `s01f35` `#3`, `s01f55` `#7`, `s05f12` `#6` và đáy của `s05f42` `#3`, `s02f06` `#9`).
  - Các cảnh báo **giữ nguyên** kèm lý do:
    - `s03f23`, `#5` và `#6` (`Car`, cờ đáy cao hơn mặt đường `0,46 m` và `0,51 m`): giữ nguyên vì qua `cam08`/`cam09` hai xe này đỗ trong bãi/sân cao hơn mặt đường chính phía sau bên phải, đáy hộp vẫn chạm đúng nền nơi xe đỗ chứ không bị vẽ lơ lửng.
    - `s02f55`, `#6` (`Car`, `width = 2,30 m`) và `s02f06`, `#10` (`Car`, `width = 2,35 m`): chỉ vượt ngưỡng cứng `2,3 m` của công cụ từ `0,00–0,05 m`, bao cả phần gương chiếu hậu và sai số nhẹ ở tầm xa `22,8–45,3 m` nên chấp nhận giữ nguyên.
    - `s05f12`, `#7` (`Car`, `width = 2,37 m`, `height = 2,36 m`): xe van thân cao cỡ lớn nên chiều cao và chiều rộng hơi vượt ngưỡng mặc định của xe con thông thường.
- **Đổi kiểm:** Do làm bài cá nhân độc lập và không có bạn ngồi cạnh, mình tự đối chiếu chéo lại 2 con số trong báo cáo với file `submission/D13-03096-bangA.csv`: `dist_m = 3,76 m` của `s05f42` `#4` (`Car`) và `z_bottom = -1,790 m` của `s01f55` `#2` (`Car`) — cả hai đều khớp hoàn toàn với bảng A.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Ý nghĩa của nhãn từ phía model học máy:**
  1. Nhãn 3D phải đo bằng **mét** thay vì pixel vì kích thước theo pixel thay đổi liên tục theo khoảng cách và tiêu cự từng camera, trong khi mô hình nhận thức 3D và hệ thống lập kế hoạch chuyển động của xe tự lái cần biết chính xác vị trí, kích thước vật lý và khoảng cách thực tế theo mét trong không gian quanh xe để tính toán vùng an toàn và quỹ đạo tránh va chạm.
  2. Vật thưa điểm cần quy tắc riêng (chỉ gán khi thấy đủ ~15–20% thể tích hoặc có đủ cơ sở từ camera, không đoán bừa) vì nếu người gán nhãn tự bịa tâm, kích thước hoặc hướng cho những cụm chỉ có vài điểm rời rạc thì dữ liệu huấn luyện sẽ chứa nhãn nhiễu, khiến model học sai mối liên hệ giữa hình dạng point cloud và bounding box (dẫn tới tăng báo động giả hoặc dự đoán lệch nghiêm trọng khi chạy thực tế).
- **(b) Cách dùng AI:**
  - Công cụ: Antigravity (trợ lý AI tích hợp trong workspace).
  - Câu hỏi chính: hướng dẫn cấu trúc dữ liệu, các bước cài `.venv` Python 3.11+, lệnh tạo task/sync/pack/check của bài lab và cách kiểm tra đối chiếu số liệu báo cáo.
  - Cách dùng kết quả: làm theo hướng dẫn thao tác trên CVAT và terminal, tự vẽ toàn bộ 68 cuboid trên CVAT, chạy `sync` lấy số liệu thực tế và rà soát lại các mục trong báo cáo.
- **(c) Điều còn chưa chắc:**
  - Ở frame `s01f35` (`#3`, đang gán `Truck`), khi xem lại ảnh `work/xem/D13-03096-s01f35.jpg` trên `cam05`/`cam06` nhận ra cụm điểm đó thực chất là biển quảng cáo lớn đặt trên dải phân cách chứ không phải xe tải.
  - Ở frame `s01f55` (`#7`, đang gán `Cyclist`), xem lại trên `cam01`/`cam07` thấy đó là dãy xe đạp dựng trên vỉa hè không có người lái (đúng guideline là không vẽ).
  - Ở frame `s05f12` (`#6`, xe con màu vàng nhỏ phía trước bên trái trên `cam03`), xe đang hướng mặt về phía xe thu dữ liệu (ngược chiều, lẽ ra `yaw_deg ≈ 90°`) nhưng hiện hộp đang để `270,5°` (ngược 180°).
  - Một số hộp xe buýt (`s02f06` `#3`, `#8`, `#9`; `s03f49` `#7`; `s05f42` `#7`) và xe tải (`s05f12` `#4`) có `width` trong khoảng `3,36–3,88 m`, hơi rộng hơn thân xe thực tế (`2,5–2,6 m`), và `s02f55` `#5` nằm ở `52,80 m` vượt nhẹ ranh giới `50 m`.
