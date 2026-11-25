# Báo cáo Day 13 — Nguyễn Thiên Phúc, 03339

## 1. Kết luận trước
- Mã số: `03339` · Task: `D13-03339` · 10 frame: `s01f13`, `s01f47`, `s02f08`, `s02f32`, `s03f03`, `s03f52`, `s04f15`, `s04f45`, `s05f21`, `s05f36`.
- Hệ điều hành: Windows · Trình duyệt: Chrome · Phiên bản CVAT: 2.74.1 · MSSV bạn ngồi cạnh: .
- Tổng cộng hiện có **50 cuboid** trên 10 frame; frame ít nhất là `s02f32` (**4 cuboid**) do đoạn đường thẳng có dải phân cách cứng sát bên trái và phần lớn các xe khác ở xa hoặc bị che khuất. Sau khi sửa 2 cuboid từ góp ý tự kiểm, phiếu `selfcheck` hiện chỉ còn cờ **4/50 cuboid**. Mình tin nhất ở **hướng yaw** (các xe cùng chiều quanh 270°, ngược chiều quanh 90°) và kích thước xe con ở gần; phần còn yếu nhất là **căn đáy chạm đường (`z_bottom`)** ở các xe xa/xe buýt lớn và giữ gọn chiều rộng hộp ở xa.
- Cách dùng CVAT:
  - Đổi góc nhìn: kéo chuột trái để xoay khung 3D (xoay thẳng đứng để nhìn từ trên xuống), kéo chuột phải để dời, cuộn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc mặc định.
  - Vẽ và chỉnh cuboid: chọn công cụ vẽ cuboid → chọn label → **Shape** (không dùng Track), nháy đúp vào cụm điểm để đặt hộp; chỉnh tâm, dài, rộng, yaw ở **Top** và chỉnh độ cao/đáy ở **Side**, **Front**.
  - Hướng & kiểm tra camera: bật **Appearance → Cuboid orientation** để căn vạch đỏ `+X` về đầu xe, rồi bấm Save và chạy `python scripts/lab.py preview` để đối chiếu hộp trên 7 ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f13-3d.png`, `t1-s01f13-top.png`, `t1-s01f47-3d.png`, `t1-s01f47-top.png` (ảnh nhìn từ trên xuống có đánh dấu phía trước, `+x`, `+y`).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f13` | Phía trước (`−y`, khớp `cam01`/`cam03`) ở **cạnh trên**; `+x` (vạch đỏ) chỉ sang **trái**; `+y` (vạch xanh lá) chỉ xuống **cạnh dưới** (ra sau, khớp `cam09`). | Mặt đường phẳng (`z` phía trước ≈ −1,73 m, phía sau ≈ −1,74 m), thấp hơn LiDAR khoảng **1,7 m**. | Khoảng **50–55 m** (xe buýt phía trước ~35 m và xe con phía sau trong `cam09` cách ~50–55 m). Góc mù camera ở khe hẹp giữa các quạt camera hai bên sát thân xe và vùng sau nhà chờ xe buýt. | Xe con (Car), xe buýt (Bus), người đi bộ (Pedestrian), dải phân cách cây bụi, nhà chờ xe buýt và trụ/biển báo. | Phía sau (`+y`, `cam09`) có các xe con chạy tới ở khoảng 20–55 m; bên trái (`+x`, `cam06`) có xe con xanh vượt ngang đầu-trái và xe ngược chiều bên kia dải phân cách; bên phải (`−x`, `cam07`/`cam08`) có nhà chờ xe buýt và 1 người đi bộ đứng chờ. | Cụm điểm rời rạc bên kia dải phân cách bên trái (`+x > 15 m`) và ở tầm ~55–60 m phía sau chỉ còn vài điểm thưa, không phân biệt được là xe hay bụi cây/cột. |
| `s01f47` | Phía trước (`−y`, khớp `cam01`/`cam03`) ở **cạnh trên**; `+x` (vạch đỏ) chỉ sang **trái**; `+y` (vạch xanh lá) chỉ xuống **cạnh dưới** (ra sau, khớp `cam09`). | Mặt đường phẳng (`z` phía trước ≈ −1,78 m, phía sau ≈ −1,79 m), thấp hơn LiDAR khoảng **1,7–1,8 m**. | Khoảng **50–55 m** (hàng xe nối đuôi phía trước kéo dài đến ~50 m, phía sau còn nhận ra xe đến ~30–40 m). Góc mù lớn do bóng che phía sau hàng xe dừng phía trước (~15–40 m) và dải cây bên phải. | Xe con (Car), xe MPV/van tối màu, xe hai bánh có người lái (Cyclist), người đi bộ/xe đạp ở lề phải, giá long môn và cây xanh. | Phía sau (`+y`, `cam09`) có 1 xe con trắng bám sát ngay sau xe thu dữ liệu (~8 m) và 1 xe khác cách ~30 m; bên phải (`−x`, `cam07`/`cam08`) có xe hai bánh và người đi bộ sát dải cây; bên trái (`+x`, `cam05`) có xe ngược chiều bên kia dải phân cách. | Các cụm điểm chồng lấn và bị che khuất sau hàng xe dừng phía trước (`−y` ~20–40 m) chỉ thấy phần nóc/cạnh mảnh, cùng một số cụm nhỏ lẫn trong tán cây/hàng rào bên phải (`−x` ~12–20 m). |

Từ chính hai frame trên:
- **3 điều chỉ camera cho biết:**
  1. Màu sắc, loại phương tiện chi tiết và vạch kẻ đường (xe con trắng/xanh, xe buýt sơn nhiều màu, vạch kẻ vàng/trắng trên mặt đường).
  2. Phân biệt rõ người đi bộ đứng ở nhà chờ (`s01f13`, `cam08`) hoặc người đi xe hai bánh lẫn sát bụi cây bên phải (`s01f47`, `cam07`/`cam08`) với cột/bụi cây khi cụm điểm LiDAR quá mảnh.
  3. Ngữ cảnh phía trên cao và chi tiết biển báo/hạ tầng giao thông (giá long môn chỉ dẫn làn đường ở `s01f47`, đèn tín hiệu, kết cấu nhà chờ xe buýt ở `s01f13`).
- **3 điều chỉ LiDAR cho biết:**
  1. Khoảng cách hình học chính xác theo mét (`m`) tới từng vật thể (ví dụ xe bám sát phía sau trong `s01f47` cách ~8 m, hàng xe phía trước trải từ ~10 m tới ~50 m).
  2. Độ cao và độ phẳng mặt đường so với cảm biến (`z ≈ −1,7 m`) cùng kích thước dài–rộng–cao thực bằng mét của từng xe.
  3. Góc nhìn tổng thể 360° từ trên xuống (BEV) và vùng bóng che (occlusion shadow) phía sau xe buýt, nhà chờ hoặc hàng xe dừng trước mặt.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f08` | Cạnh trên (`−y`) | Giữa ngã tư rộng, có xe buýt lớn chạy cắt ngang bên trái (`cam05`/`cam06`) và xe đạp băng qua ngã tư bên phải (`cam07`/`cam08`). |
| `s02f32` | Cạnh trên (`−y`) | Hàng rào dải phân cách sát bên trái phản xạ dày và tạo vệt bóng che lớn phía sau-trái cùng với xe đi sát bên (`cam05`/`cam09`). |
| `s03f03` | Cạnh trên (`−y`) | Hai bên đường có vách tôn công trình cao khép kín tầm nhìn ngang; mật độ xe sát nhau ở cả trước, trái, sau; cao độ mặt đường quanh `z ≈ −1,4 m`. |
| `s03f52` | Cạnh trên (`−y`) | Xe buýt dài ở làn ngược chiều bên trái (`cam05`/`cam06`) tạo bóng che dài sang bên trái; có xe hai bánh ở làn bên phải (`cam01`/`cam07`). |
| `s04f15` | Cạnh trên (`−y`) | Phía bên trái ngoài rào chắn có khu vực hố/độ dốc sâu hơn mặt đường; xe con trắng cắt ngang ở làn sát bên phải (`cam07`/`cam08`). |
| `s04f45` | Cạnh trên (`−y`) | Đoạn đường hơi cong nhẹ sang trái về phía trước; vách tôn bên phải và hàng rào vàng sát bên trái che tầm nhìn thấp bên kia rào. |
| `s05f21` | Cạnh trên (`−y`) | Gần vạch sang đường ở ngã tư, có xe trắng đi sát bên trái (`cam05`/`cam06`) và người đi xe ba bánh/xe máy ở làn phải (`cam03`). |
| `s05f36` | Cạnh trên (`−y`) | Đường thoáng hơn, ít xe ở gần; bên phải có các tia LiDAR xuyên qua khoảng trống dải cây bụi tạo nhiều cụm điểm lá cây rời rạc. |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f13`, `idx 1` (`dist_m = 5,12 m`) | `Car` | Đúng (`yaw_deg = 268,2°`, dọc thân xe về phía trước `−y`) | Chạm đường (`z_bottom = −1,613 m`, `ground_gap = +0,14 m`) | Đã kiểm Top/Side/Front và `preview` trên `cam01`/`cam06` | Đạt (`4,80 × 1,77 × 1,43 m`, trong khoảng sedan `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f13`, `idx 3` (`dist_m = 29,61 m`) | `Bus` | Đúng (`yaw_deg = 285,4°`, hướng về phía trước `−y`) | Chạm đường (`z_bottom = −1,731 m`, `ground_gap = +0,02 m`) | Đã kiểm Top/Side/Front và `preview` trên `cam01`/`cam03` | Đạt (`12,50 × 2,50 × 3,50 m`, trong khoảng xe buýt thường `10,0–12,5 × 2,50–2,60 × 3,00–3,50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s02f08`, `idx 4` (`dist_m = 9,59 m`) | `Cyclist` | Đúng (`yaw_deg = 273,4°`, hướng về phía trước `−y`) | Chạm đường (`z_bottom = −1,876 m`, `ground_gap = −0,09 m`) | Đã kiểm Top/Side/Front và `preview` trên `cam07`/`cam08` | Chiều dài, cao đạt (`1,85 × 1,79 m` trong khoảng `1,6–1,9 × 1,60–1,90 m`); chiều rộng đã thu gọn từ `1,53 m` xuống `1,00 m` ở Bước 5 (hơi rộng hơn mức xe đạp `0,50–0,70 m` để bao hết tay lái/người, không còn bị `selfcheck` cờ) |

- **Vòng hiệu chuẩn:** TODO(học viên): điền 1–2 câu về điểm lệch của cuboid bạn so với cả lớp ở task chung `D13-HC485087` và quy tắc guideline cả lớp đã chốt.
- **Vật bỏ qua vì không đủ cơ sở:** Các cụm điểm rất thưa phía xa (> 40–50 m) hoặc bị che khuất hoàn toàn sau hàng xe/nhà chờ ở `s01f13`, `s01f47` không đủ 15–20% thể tích để xác định kích thước (`LOW_VISIBILITY` / `UNCERTAIN_SCOPE`).

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f08` · `t4-1-bong-che.png` (bài cá nhân `D13-03339`) | Giả thuyết: thân xe buýt lớn (`#1 Bus` ở `14,61 m` và `#3 Bus` ở `17,79 m`) chắn toàn bộ chùm tia LiDAR đi qua phía sau chúng. | Ngay phía sau hai xe buýt xuất hiện các vùng quạt trống rỗng hoàn toàn, không có tia mặt đường hay vật thể nào dù hai bên vùng quạt vẫn dày điểm. | Không xem vùng trống sau xe buýt là đường trống; nếu có vật bị che một phần phía sau cần bật **occluded** và bao trọn phần kích thước suy luận được. |
| 2. Thưa ở xa | `s05f36` · `t4-2-thua-o-xa.png` (bài cá nhân `D13-03339`) | Giả thuyết: mật độ tia LiDAR thưa dần theo khoảng cách (phân kỳ góc ngang và đứng), nên ở tầm > 40 m rất ít tia trúng vào vật. | Xe con `#5 Car` ở `dist_m = 40,60 m` chỉ còn **99 điểm** tạo thành vài vạch rời rạc; các vật xa hơn (> 45–50 m) chỉ còn vài chấm đơn lẻ. | Áp dụng quy tắc **15–20% thể tích**: xe `#5` vẫn đủ khung chữ nhật để vẽ (đối chiếu bảng 4.3), các vật xa hơn không đủ cơ sở thì bỏ qua và ghi `LOW_VISIBILITY`. |
| 3. Phản xạ thấp / ít điểm bất thường | `s01f47` · `t4-3-phan-xa-thap.png` (bài cá nhân `D13-03339`) | Giả thuyết: bề mặt kính sau tối màu hoặc góc tới xiên/vật liệu hấp thụ tia hồng ngoại của xe `#3 Car` (`dist_m = 9,81 m`) làm phản xạ yếu hơn xe cùng tầm. | Dù ở gần hơn (`9,81 m`), xe `#3 Car` chỉ có **850 điểm** (cường độ trung bình `0,1`) và phần kính sau khá thưa so với xe `#4 Car` kế bên (`11,08 m`, **1435 điểm**, cường độ `1,3`). | Không bỏ sót vật và không thu hẹp hộp chỉ theo phần điểm phản xạ rõ; phải đối chiếu `cam01`/`cam03` và giữ kích thước chuẩn của `Car`. |
| 4. Điểm lạ | `s03f03` · `t4-4-diem-la.png` (bài cá nhân `D13-03339`) | Giả thuyết: phản xạ đa đường (multipath) dưới gầm xe hoặc mặt đường bóng tạo ra điểm giả nằm sâu dưới mặt đường. | Xuất hiện cụm **5 điểm** tại `(x ≈ 2,7 m; y ≈ −13,2 m; z ≈ −6,3 m)`, tức thấp hơn mặt đường phía trước khoảng **4,8 m** dù mặt đường hoàn toàn phẳng. | **Không gán thành vật** theo guideline; bỏ qua cụm điểm nhiễu này khi vẽ cuboid. |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-03339-bangA.csv`; số điểm lấy từ cột `points_in` của `submission/D13-03339-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f13`, `idx 1`, `Car` | −0,897 m | 1,433 m | −1,613 m | 5,12 m | 5,19 m | 268,2° | Chạm đường (`ground_gap = +0,14 m`, lệch < 0,3 m) |
| `s01f13`, `idx 3`, `Bus` | 0,019 m | 3,500 m | −1,731 m | 29,61 m | 29,61 m | 285,4° | Chạm đường (`ground_gap = +0,02 m`, sát mặt đường) |
| `s01f13`, `idx 5`, `Car` | −1,252 m | 1,715 m | −2,110 m | 28,75 m | 28,77 m | 277,4° | Chìm (`ground_gap = −0,36 m`, lệch > 0,3 m dưới −1,7 m, cần kéo đáy lên ở Side/Front) |

- **(a) Cuboid gần nhất và xa nhất:**
  - Gần nhất: `s03f03`, `idx 4` (`Car`) — `dist_m = 3,71 m`, `r = 3,88 m`.
  - Xa nhất: `s05f36`, `idx 5` (`Car`) — `dist_m = 40,60 m`, `r = 40,62 m`.
- **(b) So sánh kích thước với bảng 4.3:**
  - `Cyclist` (`s03f52`, `idx 4`): `l = 1,717 m`, `w = 0,500 m`, `h = 1,800 m` — cả 3 chiều nằm gọn trong khoảng xe đạp có người (`1,6–1,9 × 0,50–0,70 × 1,60–1,90 m`). Riêng `s02f08`, `idx 4` sau khi thu gọn từ `1,528 m` xuống `1,000 m` (`l = 1,853 m`, `h = 1,787 m`) có `w = 1,000 m` vẫn hơi rộng hơn khoảng xe đạp (`0,50–0,70 m`) do bao rộng hai bên tay lái/người.
  - `Car` (`s01f13`, `idx 1`): `l = 4,801 m`, `w = 1,768 m`, `h = 1,433 m`. Cả 3 chiều đều nằm trong khoảng chuẩn của sedan/hatchback (`3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`). Riêng xe ở xa `s05f36`, `idx 5` có `w = 2,860 m` ngoài khoảng (`1,4–2,3 m`) do ít điểm nên hộp bị nới rộng quá thân xe.
- **(c) Góc `yaw_deg` xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước: `s01f13 #1 Car` (`268,2°`), `s01f47 #3 Car` (`266,3°`), `s04f15 #4 Car` (`270,4°`) — đều nằm sát **270°**.
  - Xe ngược chiều (làn đối diện bên trái `+x`): `s02f32 #1 Car` (`89,4°`), `s05f36 #3 Car` (`90,8°`), `s05f36 #4 Car` (`90,8°`) — đều nằm sát **90°**.

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s04f45`, `idx 2`, `Car` | 7,25 m | 2701 |
| `s05f21`, `idx 3`, `Car` | 15,28 m | 523 |
| `s04f45`, `idx 3`, `Car` | 23,69 m | 319 |
| `s04f45`, `idx 5`, `Car` | 34,43 m | 92 |

- **Quy luật mật độ:** Khi khoảng cách tăng **gấp đôi** (từ `7,25 m` lên `15,28 m`, hoặc từ `15,28 m` lên `34,43 m`) thì số điểm trong cuboid giảm khoảng **4–5 lần** (còn khoảng **18–20%**), khớp với đường cong ở slide bài giảng (trung vị xe con: ~1968 điểm ở 0–10 m, ~718 ở 10–20 m, ~214 ở 20–30 m, ~72 ở 30–40 m).
- **Tự kiểm (`preview` & `selfcheck`):**
  - Hai cuboid đã sửa sau `selfcheck` (sau đó Save và chạy lại `sync`):
    1. `s02f08`, `idx 4` (`Cyclist`): thu hẹp chiều rộng `width` từ `1,528 m` xuống `1,000 m` (số điểm trong hộp từ `568` còn `527`), hết cờ vượt khoảng rộng.
    2. `s03f52`, `idx 4` (`Cyclist`): nâng đáy cuboid lên từ `z_bottom = −2,018 m` (`ground_gap = −0,26 m`) thành `z_bottom = −1,925 m` (`z = −1,025 m`, `ground_gap = −0,17 m`), hết cờ đáy chìm.
  - Những cảnh báo giữ nguyên (`4/50` cuboid) và lý do:
    - `s01f13 #5 Car` (`ground_gap = −0,36 m`), `s02f08 #1 Bus` (`ground_gap = −0,40 m`), `s02f32 #2 Car` (`ground_gap = −0,40 m`): đối chiếu lại trên ảnh `work/xem/` và point cloud thấy hộp vẫn khớp thân xe; sai lệch `ground_gap` do điểm mặt đường quanh rìa hộp thưa hoặc thân xe buýt rộng ở ngã tư.
    - `s05f36 #5 Car` (`width = 2,86 m` tại `dist_m = 40,60 m`): xe ở xa chỉ còn `99` điểm thưa nên giữ hộp rộng hơn một chút để bao trọn cụm điểm rìa, tránh cắt thiếu thân xe.
- **Đổi kiểm:** 

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Ý nghĩa từ phía model học trên nhãn:**
  - Nhãn 3D phải đo bằng **mét** chứ không phải pixel vì model nhận thức 3D (dự đoán quỹ đạo, khoảng cách phanh, tránh va chạm) cần kích thước và tọa độ vật lý thật trong không gian 3D bất biến theo góc nhìn, trong khi ảnh 2D theo pixel bị biến dạng phối cảnh theo khoảng cách.
  - Vật thưa điểm cần quy tắc riêng (tối thiểu 15–20% thể tích mới vẽ) vì nếu người gán nhãn đoán kích thước và tâm hộp trên cụm chỉ có vài điểm rời rạc, nhiễu nhãn sẽ làm model học sai hình học và tăng dự đoán sai (false positive) hoặc lệch IoU.
- **(b) Dùng AI:**
  - Công cụ: Trợ lý AI trong IDE (Antigravity / Gemini).
  - Câu hỏi chính: Hướng dẫn quy trình từng bước của bài lab, đọc kiểm tra đặc điểm point cloud/camera trên 10 frame.
  - Cách dùng kết quả: Đối chiếu từng số liệu trực tiếp với `submission/D13-03339-bangA.csv`, `submission/D13-03339-tukiem.csv`, ảnh `work/xem/` và giao diện CVAT trên máy trước khi đưa vào báo cáo.
- **(c) Điều còn chưa chắc:**
  - Độ chính xác của `z_bottom` ở các xe cách trên 25 m khi mật độ điểm mặt đường xung quanh hộp thưa dần; đồng thời frame đầu `s01f13` (5 cuboid) và `s02f32` (4 cuboid) hiện vẫn ít hơn mức khuyến nghị (8 và 5 cuboid) vì các cụm điểm còn lại ở xa hoặc bị dải phân cách/nhà chờ che khuất khá nhiều.
