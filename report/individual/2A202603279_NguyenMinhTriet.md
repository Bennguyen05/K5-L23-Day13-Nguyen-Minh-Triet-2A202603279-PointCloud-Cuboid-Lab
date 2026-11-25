# Báo cáo Day 13 — Nguyễn Minh Triết, 2A202603279

## 1. Kết luận trước
Mã số `2A202603279`, task `D13-2A202603279` (CVAT task 26). 10 frame: s01f16, s01f56, s02f27, s02f55, s03f03, s03f20, s04f05, s04f31, s05f12, s05f39. Môi trường: Ubuntu (WSL) trên Windows, trình duyệt Microsoft Edge, CVAT 2.81.0 chạy bằng Docker. Người đổi kiểm: 2A202603264.

Tổng cộng **88 cuboid** (Car 73, Bus 4, Cyclist 4, Pedestrian 5, Truck 2). Frame ít cuboid nhất là s05f39 với 5 hộp, vì cảnh chỉ có ngần ấy vật và tôi chỉ vẽ những vật tôi chắc chắn. `selfcheck` cờ **1/88** cuboid (s03f20 #9). Tôi đã xóa các hộp có `dist_m` trên 50 m; trong bảng A hiện còn một hộp ở 52.82 m (s02f55 #4), nằm ngoài phạm vi chấm.

Phần tôi tin nhất là **hướng**: trong 73 xe con, 49 xe có `yaw_deg` gần 270° (cùng chiều) và 23 xe gần 90° (ngược chiều), chỉ một xe (s03f20 #7) ở 160,5°. Phần yếu nhất là **vật xa, thưa điểm** (ví dụ s02f27 #9 cách 47.54 m chỉ còn 40 điểm) và **độ cao đáy** ở một số frame: 17 hộp có z đáy lệch hơn 0,3 m so với −1,7 m, dù `ground_gap_m` của chúng gần 0 nên đáy vẫn chạm mặt đường cục bộ.

Cách dùng CVAT: (1) đổi góc nhìn bằng cách kéo chuột trái để xoay khung 3D cho tới khi nhìn thẳng xuống, kéo chuột phải để dời và lăn con lăn để phóng to; (2) vẽ cuboid bằng nút vẽ cuboid, chọn label, bấm **Shape**, nháy đúp vào cụm điểm để đặt, rồi chỉnh tâm, dài, rộng, yaw ở khung Top và chỉ chỉnh độ cao, đáy ở khung Side, Front; (3) bật **Cuboid orientation** để căn vạch đỏ về đầu xe, đối chiếu ảnh camera, và dùng `python scripts/lab.py preview` (nằm trong `sync`) để xem cuboid trên ảnh camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f16 và s01f56. Ảnh: `t1-s01f16-3d`, `t1-s01f16-top`, `t1-s01f56-3d`, `t1-s01f56-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f16 | Phía trước (−y) ở **cạnh phải** của ảnh nhìn từ trên xuống (xe trắng và xe buýt trong cam01 nằm về phía này); +x hướng lên trên ảnh (bên trái xe), +y sang trái ảnh (ra sau) | Phẳng; z đáy trung vị của xe −1,69 m, tức mặt đường thấp hơn LiDAR khoảng 1,7 m | khoảng 47 m (xe xa nhất đã gán nhãn, `dist_m` 46,96) | Xe con, 1 xe buýt | 5 hộp phía sau, 1 hộp bên phải; cam05 thấy một xe con phía sau bên trái | Các cụm điểm rải rác ở rìa xa phía bên kia đường (tán cây, nhà ven đường), chưa phân loại được thành vật cụ thể |
| s01f56 | Phía trước (−y) ở **cạnh phải** của ảnh nhìn từ trên xuống (cam01 thấy các xe trắng ở phía trước); +x lên trên ảnh, +y sang trái ảnh | Phẳng; z đáy trung vị của xe −1,75 m. Người đi bộ và xe hai bánh có z đáy khoảng −1,32 m, cao hơn mặt đường xe chạy khoảng 0,4 m (giả thuyết: họ đứng trên vỉa hè) | khoảng 22 m (xe xa nhất đã gán nhãn) | Xe con, 2 người đi bộ, 1 xe hai bánh | 2 hộp phía sau, 3 hộp bên phải; cam05 thấy xe trắng phía sau bên trái | Các cụm điểm dạng tán cây ở rìa trên của cảnh, chưa phân loại được |

**Chỉ camera cho biết:** (1) màu xe (xe trắng, xe buýt đỏ trắng, xe xanh ở cam05); (2) biển chỉ đường xanh phía trước ở s01f56 và vạch kẻ làn trên mặt đường; (3) loại xe cụ thể và đầu hay đuôi xe (đèn trước, đèn sau) để chọn hướng cuboid. **Chỉ LiDAR cho biết:** (1) khoảng cách và vị trí theo mét của từng vật; (2) kích thước thật dài × rộng × cao của hộp; (3) vật ở hướng không camera nào chiếu tới, ví dụ vật phía sau trong vùng giữa các camera.

Tám frame lướt (số liệu lấy từ bảng A; hướng "phía trước" ghi theo góc nhìn mặc định, chưa kiểm lại từng frame):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f27 | cạnh phải (góc nhìn mặc định) | Hộp xe xa nhất s02f27 #9 cách 47,54 m chỉ còn 40 điểm |
| s02f55 | cạnh phải (góc nhìn mặc định) | Có xe cách 52,82 m (s02f55 #4), ngoài phạm vi 50 m |
| s03f03 | cạnh phải (góc nhìn mặc định) | Có một xe tải (s03f03 #9) |
| s03f20 | cạnh phải (góc nhìn mặc định) | Ba người đi bộ đứng sát nhau (cờ s03f20 #9); một xe buýt ngắn 6,03 m |
| s04f05 | cạnh phải (góc nhìn mặc định) | Xe gần nhất cả bộ (s04f05 #1, 3,70 m) với 6586 điểm |
| s04f31 | cạnh phải (góc nhìn mặc định) | Xe buýt ngược chiều ở 34,19 m (`yaw_deg` 91,0°) |
| s05f12 | cạnh phải (góc nhìn mặc định) | Một xe tải và hai xe hai bánh |
| s05f39 | cạnh phải (góc nhìn mặc định) | Chỉ 5 hộp; có một xe buýt ở 25,50 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`. Cả ba ảnh có đủ khung Top, Side, Front của cuboid đang chọn và đã bật Cuboid orientation.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con | s02f27 #1 | Car | `yaw_deg` = 273,0°, xe cùng chiều | z đáy = −1,854 m, `ground_gap_m` = 0,03 | Có: ảnh `t2-a-car` có Top, Side, Front và mũi tên hướng | 3,956 × 1,728 × 1,527 m: trong khoảng sedan/hatchback |
| (b) xe buýt hoặc xe tải | s04f31 #4 | Bus | `yaw_deg` = 91,0°, xe ngược chiều | z đáy = −1,843 m | Có: ảnh `t2-b-bus-truck` có Top, Side, Front và mũi tên hướng | dài 10,379 m và cao 3,075 m trong khoảng xe buýt thường; rộng 2,719 m **vượt nhẹ** 2,50–2,60 m |
| (c) người đi bộ | s03f20 #11 | Pedestrian | `yaw_deg` = 260,4° | z đáy = −1,499 m | Có: ảnh `t2-c-ped-cyclist` có Top, Side, Front và mũi tên hướng | 0,747 × 0,489 × 1,645 m: dài và cao trong khoảng, rộng **thấp hơn nhẹ** 0,5 m |

Vòng hiệu chuẩn: tôi chưa tham gia vòng hiệu chuẩn của lớp nên chưa có kết quả so sánh với các bạn. Vật bỏ qua vì không đủ cơ sở: không có; tôi chỉ vẽ những vật tôi chắc chắn và không ghi nhận vật nghi vấn nào.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Frame mượn thì ghi rõ nguồn: task chung, hoặc MSSV của bạn cho mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s03f20, `t4-1-bong-che` | Xe và người đứng sát nhau nên vật gần che vật phía sau | Các cụm xe ở gần che nhau; phía sau cụm hộp gần như không có điểm nào, đó là vùng **chưa quan sát được**, không phải đường trống | Bật occluded; cuboid bao phần suy luận được |
| 2. Thưa ở xa | s02f55, `t4-2-thua-o-xa` (hộp s02f55 #4, cách 52,82 m, còn 45 điểm; hộp này ngoài 50 m nên không được chấm, tôi chỉ dùng làm ví dụ minh họa) | Khoảng cách xa nên chùm tia LiDAR thưa, khoảng cách giữa các tia lớn hơn kích thước chi tiết của xe | Cuboid xe ở xa chỉ còn vài chục đến vài trăm điểm rải rác trong hộp; các khung Top, Side, Front gần như trống | Vẽ nếu thấy khoảng 15–20% thể tích; không thì bỏ qua và ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s01f56, `t4-3-phan-xa-thap` | Giả thuyết: xe đen có lớp sơn tối hấp thụ nhiều tia laser hơn, nên phản xạ về yếu | Hai xe ngay phía trước ego đứng cạnh nhau, một xe đen bên trái và một xe trắng bên phải, ở cùng khoảng cách; xe đen thu được ít điểm hơn xe trắng | Ghi giả thuyết, không khẳng định; vẫn vẽ vì thấy rõ hình dạng và ảnh camera xác nhận vật |
| 4. Điểm lạ | s02f35 trong **task chung** `D13-HC485087`, `t4-4-diem-la` | Giả thuyết: nhiễu đo hoặc phản xạ đa đường; chưa rõ nguyên nhân | Một vệt điểm lẻ loi nằm sâu dưới mặt đường, tách khỏi đám điểm chính, và vài vệt điểm chéo ở trên cao; không khớp với vật nào trong ảnh camera | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603279-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m, nhưng mặt đường thay đổi theo frame: đáy lệch hơn 0,3 m so với −1,7 m chưa chắc là lơ lửng hay chìm nếu `ground_gap_m` vẫn gần 0.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s04f05 #1 Car | -0.782 | 1.763 | -1.663 | 3.70 | 3.78 | 268.8 | chạm đường (−1,664 m, trong ±0,3 m) (`ground_gap_m` = 0.10) |
| s01f16 #1 Car | -1.056 | 1.475 | -1.794 | 9.99 | 10.05 | 270.5 | chạm đường (−1,794 m, trong ±0,3 m) (`ground_gap_m` = 0.03) |
| s03f20 #2 Car | -1.358 | 1.685 | -2.200 | 27.14 | 27.18 | 83.4 | z đáy −2,200 m lệch hơn 0,3 m so với −1,7 m nhưng gap nhỏ, nhiều khả năng mặt đường ở đây thấp hơn (`ground_gap_m` = 0.09) |
| s01f56 #7 Pedestrian | -0.484 | 1.673 | -1.321 | 13.67 | 13.68 | 271.6 | z đáy −1,320 m lệch hơn 0,3 m so với −1,7 m nhưng gap gần 0, nhiều khả năng mặt đường ở đây cao hơn (`ground_gap_m` = -0.02) |

(a) Cuboid **gần nhất**: s04f05 #1 Car, `dist_m` = 3.70 m, r = 3.78 m. Cuboid **xa nhất trong phạm vi 50 m**: s02f27 #9 Car, `dist_m` = 47.54 m, r = 47.55 m. Hộp xa nhất trong bảng A là s02f55 #4 (52.82 m, r = 52.82 m) nằm ngoài 50 m nên không được chấm.

(b) Người đi bộ s03f20 #9: dài 0,948, rộng 0,561, cao 1,645 m so với bảng 4.3 (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m): chiều dài vượt 0,048 m. Lý do: người này đi sải chân rộng hơn bình thường, nên để bao hết toàn bộ điểm tôi phải để hộp vượt chỉ tiêu một ít. Xe con s04f05 #1: 5,056 × 2,004 × 1,763 m nằm trong khoảng SUV/minivan (4,4–5,3 × 1,80–2,10 × 1,65–2,10 m).

(c) `yaw_deg` xe cùng chiều phía trước: s01f16 #1 = 270,5°, gần 270°. Xe ngược chiều: s02f27 #2 = 89,1°, gần 90°. Trong 73 xe con, 49 xe có yaw gần 270°, 23 xe gần 90°, và một xe (s03f20 #7) ở 160,5°.

Mật độ theo khoảng cách (4 cuboid xe con ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f56 #2 Car | 6,75 | 1450 |
| s04f05 #2 Car | 14,22 | 774 |
| s03f20 #2 Car | 27,14 | 304 |
| s02f55 #7 Car | 40,42 | 45 |

Quy luật: xa gấp khoảng 6 lần (6,75 m lên 40,42 m) thì số điểm giảm khoảng 32 lần (1450 xuống 45), gần với quy luật số điểm tỉ lệ nghịch bình phương khoảng cách. So với đường cong ở slide (xe con: khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m), cuboid của tôi cùng xu hướng; số điểm của tôi cao hơn một chút ở 10–30 m. Số điểm của từng hộp còn phụ thuộc vào che khuất.

Tự kiểm: sau `preview` / `selfcheck` tôi thực hiện hai thay đổi: sửa lại hộp Pedestrian ở s03f20 (hộp bị cờ) và xóa các hộp có `dist_m` trên 50 m. Cảnh báo **giữ nguyên**: s03f20 #9 Pedestrian: "64 điểm sát ngay ngoài mép". Hai người đi sát nhau nên cuboid của họ chồng nhẹ ở mép; đã xem lại trên Top và Front, hộp bao đủ điểm của người #9 và các điểm sát mép nhiều khả năng thuộc người bên cạnh, nên tôi giữ nguyên.

Đổi kiểm với 2A202603264: hai con số đưa bạn đối chiếu với bảng A là `dist_m` của s04f05 #1 (3.70 m) và `yaw_deg` của s01f16 #1 (270,5°). Kết quả: khớp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model 3D học vị trí, kích thước và hướng của vật trong không gian thật, nên nhãn phải đo bằng mét (tâm, dài, rộng, cao, yaw) thì mới dùng trực tiếp để tránh va chạm; nhãn pixel mất thông tin khoảng cách và kích thước thật. Vật thưa điểm cần quy tắc riêng vì nhìn quá ít điểm, người gán dễ đoán khác nhau; ép đoán sẽ tạo nhãn nhiễu làm model học sai, nên guideline đặt ngưỡng (khoảng 15–20% thể tích) và cho phép bỏ qua kèm ghi chú.

(b) Dùng AI: Claude (Anthropic). Câu hỏi chính: hướng dẫn cài đặt dữ liệu, CVAT và quy trình lab; giải thích cảnh báo `selfcheck`; đọc bảng A và ảnh chụp để soạn nháp báo cáo. Tôi lấy số liệu trực tiếp từ bảng A và phiếu tự kiểm của mình, và tự vẽ, tự sửa các cuboid trên CVAT.

(c) Điều còn chưa chắc: không có hộp nào tôi còn chưa chắc, vì tôi chỉ vẽ những vật tôi chắc chắn và bỏ qua vật quá thưa điểm. Một số hộp có z đáy lệch hơn 0,3 m so với −1,7 m nhưng `ground_gap_m` gần 0, nhiều khả năng do mặt đường ở đó không cùng độ cao với phần còn lại của cảnh.
