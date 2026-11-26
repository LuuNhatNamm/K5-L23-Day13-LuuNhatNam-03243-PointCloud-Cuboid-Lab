# Báo cáo Day 13 — Lưu Nhật Nam, 2A202603243

## 1. Kết luận trước

Tôi làm bài trên task `D13-2A202603243`, CVAT local 2.82.0, Windows và Microsoft Edge. Báo cáo dùng hai file `D13-2A202603243-bangA.csv` và `D13-2A202603243-tukiem.csv` hiện có trong `submission/`.

Bảng A có **114 cuboid**, gồm 87 Car, 9 Truck, 4 Bus, 8 Cyclist và 6 Pedestrian. Phiếu tự kiểm còn cảnh báo ở **22/114 hộp**. Frame `s05f48` có ít nhất, với 4 hộp. Tôi đã rà lại frame này và không thấy thêm vật đủ cơ sở để vẽ, nên giữ số hộp theo những vật đã nhận ra và ghi nhận cảnh báo dưới mức 5 hộp/frame.

| Frame | Số hộp | Frame | Số hộp |
|---|---:|---|---:|
| s01f06 | 9 | s03f50 | 11 |
| s01f53 | 7 | s04f02 | 20 |
| s02f10 | 15 | s04f26 | 17 |
| s02f30 | 8 | s05f28 | 13 |
| s03f15 | 10 | s05f48 | 4 |

## 2. Đọc frame

Hai frame đọc kỹ là `s01f06` và `s01f53`. Ảnh gồm `t1-s01f06-3d.png`, `t1-s01f06-top.png`, `t1-s01f53-3d.png` và `t1-s01f53-top.png`. Hai ảnh top đã thêm dấu trục theo hệ VF.

| Frame | Hướng trên ảnh top đã lưu | Mặt đường | Vật đã gán xa nhất trong frame | Loại vật và vùng cần chú ý |
|---|---|---|---:|---|
| s01f06 | Phía trước −y chếch xuống phải, +x chếch lên phải, +y chếch lên trái | Cảnh đường nhìn tương đối phẳng. Mốc tham chiếu khoảng −1,7 m, cần kiểm nền quanh từng hộp | 42,80 m | Có Car và Truck ở phía trước, phía sau và hai bên. Dải điểm biên đường và vùng sau vật lớn cần đối chiếu camera |
| s01f53 | Phía trước −y sang phải hơi chếch lên, +x lên trên hơi chếch trái, +y sang trái hơi chếch xuống | Có mặt đường và biên đường. Đáy các Car đã gán nằm khoảng −1,850 đến −1,768 m | 16,17 m | Có Car và Cyclist. Cyclist nằm bên phải xe thu dữ liệu. Cụm điểm nhỏ ở xa và dải cây, biên đường khó nhận dạng chỉ bằng LiDAR |

Khoảng cách trên lấy từ vật đã gán trong Bảng A. Vật xa nhất thực sự nhận ra trong toàn cảnh cần được phân biệt với vật xa nhất đã vẽ hộp.

Ba thông tin camera bổ sung là màu thân xe, vạch sơn đường và hình ảnh bề mặt để phân biệt cây, hàng rào với xe. Ba thông tin LiDAR dùng để đo là tọa độ theo mét, khoảng cách đến tâm vật và cấu trúc cao thấp của cụm điểm. Mình dùng cả hai nguồn khi vật thiếu điểm hoặc đầu, đuôi xe khó phân biệt.

Với tám frame còn lại, phía trước theo hệ VF luôn là **−y**. Cạnh tương ứng trên màn hình thay đổi khi xoay Perspective.

| Frame | Hướng phía trước theo hệ VF | Điểm cần lưu ý từ bài hiện có |
|---|---|---|
| s02f10 | −y | Có Bus và các vật nhỏ. Ảnh bóng che lấy từ frame này |
| s02f30 | −y | Một số Car có cảnh báo chiều rộng |
| s03f15 | −y | Có hộp gần nhất toàn bộ bảng, Truck #4 |
| s03f50 | −y | Bus #8 ngoài 50 m. Car #11 ít điểm và có kích thước nhỏ bất thường |
| s04f02 | −y | Car #13 có cảnh báo đáy cao hơn nền địa phương |
| s04f26 | −y | Car #16 ở 48,18 m, xa nhất trong phạm vi 50 m |
| s05f28 | −y | Có cảnh báo điểm sát ngoài mép và đáy thấp hơn nền |
| s05f48 | −y | Có 4 hộp. Mình không nhận ra thêm vật đủ cơ sở để vẽ. Ảnh điểm nghi vấn lấy từ frame này |

## 3. Ba cuboid mẫu

Ảnh gồm `t2-a-car.png`, `t2-b-bus-truck.png` và `t2-c-ped-cyclist.png`. Hộp trong ảnh được đối chiếu với CSV bằng kích thước, vì ID giao diện có thể khác `idx`.

| Mẫu | Frame, idx trong Bảng A | Class, yaw | Kiểm đáy | Kích thước và điều cần rà lại |
|---|---|---|---|---|
| Xe con đã chụp | s01f06, #1 | Car, 272,5° | `ground_gap_m` +0,15 m | 4,687 × 1,951 × 1,445 m. Rộng hơn mốc sedan 1,90 m khoảng 0,051 m, cần kiểm cạnh hộp và phần gương |
| Xe tải đã chụp | s01f06, #6 | Truck, 90,6° | `ground_gap_m` +0,15 m | 4,393 × 1,784 × 2,060 m. Nhỏ hơn khoảng xe tải nhẹ trong bảng 4.3, cần kiểm loại xe và class |
| Xe hai bánh có người | s01f53, #7 | Cyclist, 266,0° | `ground_gap_m` +0,12 m | 1,796 × 0,764 × 1,810 m. So với xe máy có người, dài thấp hơn cận 1,80 m khoảng 0,004 m và cao hơn cận 1,80 m khoảng 0,010 m |


## 4. Bốn hiện tượng chất lượng

Các ảnh dưới đây lấy từ task cá nhân. Nguyên nhân được ghi là **giả thuyết**, cần đối chiếu khi đọc lại ảnh và CVAT.

| Hiện tượng | Frame và ảnh | Điều quan sát, giả thuyết | Hệ quả khi gán nhãn |
|---|---|---|---|
| Bóng che | s02f10, `t4-1-bong-che.png` | Có xe buýt và vùng thiếu điểm phía sau nó theo hướng quét. Giả thuyết là vật lớn che tia LiDAR | Đối chiếu camera. Nếu vật bị che vẫn đủ cơ sở nhận dạng thì vẽ phần suy luận được và kiểm occluded |
| Thưa ở xa | Ảnh sơ bộ s02f10, `t4-2-thua-o-xa.png`. Ví dụ định lượng xa lấy ở s01f06 #9 | Có cụm chỉ còn các dải điểm thưa. Car ở 42,80 m có 76 điểm, thấp hơn các Car gần. Giả thuyết là số tia trúng vật giảm theo khoảng cách. Vị trí hộp trong ảnh sơ bộ cần đối chiếu lại trước khi dùng làm bằng chứng cho vật xa | Áp quy tắc thấy khoảng 15–20% thể tích hoặc có cơ sở suy luận. Thiếu cơ sở thì bỏ qua và ghi LOW_VISIBILITY |
| Phản xạ thấp hoặc ít điểm bất thường | s02f10, `t4-3-phan-xa-thap.png` | Ảnh chọn xe tối màu để đối chiếu vùng ít điểm. Giả thuyết là vật liệu, kính hoặc góc tới làm giảm tín hiệu. Ảnh chưa tách được ảnh hưởng phản xạ khỏi che khuất và góc nhìn | Kiểm camera và các mặt hộp. Hạn chế suy rộng kích thước từ vài điểm rời |
| Điểm lạ | s05f48, `t4-4-diem-la.png` | Mình chọn chấm sáng đứng riêng trong vùng đen bên phải. Vùng đã khoanh được ghi là “điểm nghi vấn”. Giả thuyết là nhiễu đo hoặc phản xạ, nhưng vẫn cần kiểm khả năng đó là vật thật ở xa | Đối chiếu vị trí 3D, mặt đường và camera trước khi gán thành vật. Giữ ghi nhận nghi vấn khi chưa đủ cơ sở kết luận |

## 5. Số từ cuboid của tôi

Các số đo lấy từ hai CSV hiện có. Khoảng cách 3D tính bằng `r = √(x² + y² + z²)` từ tọa độ đã làm tròn trong CSV.

| Frame, idx, class | z (m) | h (m) | z đáy = z − h/2 (m) | dist_m (m) | r (m) | yaw (°) | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| s01f06, #3, Car | −0,951 | 1,824 | −1,863 | 17,13 | 17,155 | 269,7 | Phiếu ghi đáy thấp hơn nền địa phương 0,03 m, gần mức tiếp xúc đường |
| s01f06, #6, Truck | −1,168 | 2,060 | −2,198 | 19,45 | 19,481 | 90,6 | Thấp hơn mốc chung −1,7 m khoảng 0,498 m, nhưng phiếu ghi cao hơn nền địa phương 0,15 m. Cần kiểm nền quanh hộp |
| s01f53, #7, Cyclist | −0,388 | 1,810 | −1,293 | 9,97 | 9,974 | 266,0 | Cao hơn mốc chung −1,7 m khoảng 0,407 m. Khoảng hở với nền địa phương +0,12 m, cần kiểm Side và Front |

Hộp gần nhất toàn bộ bảng là Truck `s03f15 #4`, với `dist_m` **3,16 m** và r **3,240 m**. Hộp xa nhất là Bus `s03f50 #8`, với `dist_m` **59,68 m** và r **59,681 m**, ngoài vùng 50 m. Trong phạm vi 50 m, hộp xa nhất là Car `s04f26 #16`, với `dist_m` **48,18 m** và r **48,185 m**.

Car `s01f06 #2` dài 4,682 m, rộng 1,773 m và cao 1,450 m, đều trong khoảng sedan của bảng 4.3. Cyclist `s01f53 #7` được so với khoảng xe máy có người ở mục 3. Sai lệch nhỏ có thể liên quan tư thế người và cách căn mặt hộp.

Car phía trước `s01f06 #3` có y = −16,979 m và yaw **269,7°**, gần hướng cùng chiều 270°. Car `s01f06 #7` có yaw **91,4°**, gần hướng ngược chiều 90°. Nhãn hướng trong bảng phù hợp hai hướng tham chiếu này, còn đầu xe thực tế cần đối chiếu camera.

| Car dùng so mật độ | dist_m (m) | points_in |
|---|---:|---:|
| s01f06 #2 | 4,38 | 3004 |
| s01f06 #3 | 17,13 | 517 |
| s01f06 #7 | 25,28 | 214 |
| s01f06 #9 | 42,80 | 76 |

Trong bốn ví dụ cùng frame, khoảng cách tăng khoảng 9,77 lần từ 4,38 lên 42,80 m thì số điểm còn khoảng 2,53%, tức giảm khoảng 39,5 lần. Xu hướng giảm phù hợp đường cong minh họa trong bài giảng, gồm khoảng 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m và 19 ở 60–80 m. Số điểm cụ thể còn phụ thuộc kích thước, hướng bề mặt và che khuất.


Với cảnh báo sát ngưỡng như Truck `s03f15 #4` dài 3,992 m so với cận 4,0 m, tôi đã kiểm loại xe và điểm biên và quyết định giữ. Các cảnh báo chiều rộng lớn hoặc chiều cao bất thường đã rà lại và quyết định giữ.

**Kiểm chéo** — Người kiểm chéo có MSSV **2A202603114**. Tôi đã đối chiếu hai con số dưới đây trên màn hình với Bảng A và phiếu tự kiểm. Cả hai đều khớp, nên giữ nguyên số liệu.

| Cuboid | Số được đối chiếu | File đối chiếu | Kết quả |
|---|---|---|---|
| s01f06, idx 2, Car | `dist_m = 4,38 m` | `D13-2A202603243-bangA.csv` | Khớp |
| s01f06, idx 9, Car | `points_in = 76 điểm` | `D13-2A202603243-tukiem.csv` | Khớp |

Nhận xét: một số hộp hơi rộng hơn vật, nhưng nhìn chung nhãn chính xác.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Model cần nhãn 3D theo mét để học vị trí, kích thước và khoảng cách của vật trong không gian thực. Với vật thưa điểm, quy tắc riêng giúp tạo nhãn nhất quán hơn và hạn chế việc model học từ những hộp suy đoán thiếu cơ sở.

Mình có dùng AI trong quá trình làm bài và chuẩn bị bản báo cáo này.

- **Công cụ** — Codex.
- **Nội dung hỗ trợ** — Hướng dẫn CVAT, tổng hợp yêu cầu lab, thêm dấu chỉ dẫn, tính số từ CSV.

Những phần tôi còn chưa chắc là kích thước một số hộp bị cờ, đáy so với nền địa phương, nguyên nhân ít điểm ở xe tối màu và bản chất chấm đứng lẻ trong ảnh điểm nghi vấn. Kết quả vòng hiệu chuẩn, nhật ký hai hộp đã sửa.
