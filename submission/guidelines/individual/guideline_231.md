# Annotation Guideline v1.0: Bounding Box cho xe và người trong ảnh đường phố đa dạng

**Dữ liệu:** images.zip, 40 ảnh (img_001 – img_040)
**Công cụ:** CVAT
**Lưu ý:** Các quyết định ghi "đề xuất" cần mentor xác nhận trước khi gán hàng loạt.

## 1. Mục tiêu

Mô hình nhận diện xe và người tham gia giao thông thật, kèm trạng thái bị che hoặc bị cắt, để xe tự lái dự đoán và tránh va chạm. Dataset gồm nhiều cảnh khác nhau, không chỉ đường phố. Vì vậy quy tắc quan trọng nhất là phân biệt đối tượng thật với hình ảnh, mô hình và vật trang trí.

## 2. Nhóm cảnh trong dataset

| Nhóm cảnh | Mô tả | Ảnh |
|---|---|---|
| Đường đông xe máy | Nhiều xe máy, người lái và người ngồi sau, xếp sát nhau | 001, 005, 006, 008, 010, 021, 026, 030 |
| Tuk-tuk và xe công cộng | Tuk-tuk, minibus, xe đạp kéo (cyclo) | 004, 009, 016, 023, 035, 040 |
| Xe tải và xe bán tải | Xe tải chở hàng, xe bán tải chở hàng hoặc chở xe khác | 002, 007, 012, 024, 028, 036, 037 |
| Người đi bộ và xe đạp | Người băng qua đường, người đẩy xe, xe đạp | 003, 014, 029, 034 |
| Cảnh đêm và mưa | Ánh đèn, mặt đường ướt phản chiếu xe | 013, 015, 031 |
| Không phải xe thật | Tượng, đồ chơi, mannequin, billboard, hình trên tường, phản chiếu trong gương | 011, 017, 018, 020, 027, 033, 038 |

## 3. Lớp nhãn

| Lớp | Định nghĩa | Ghi chú |
|---|---|---|
| car | Ô tô con và xe taxi chở người. | Xe taxi xếp vào car. |
| van | Xe van chở người (xe khách nhỏ tư nhân, xe van gia đình). | Phân biệt với bus theo mục 6. |
| bus | Xe buýt và minibus chở khách công cộng. | Đề xuất: minibus là bus nếu có dấu hiệu chở khách công cộng. |
| truck | Xe tải, xe bán tải có thùng hàng, xe chở xe khác. | Pickup và xe tải nhỏ đều là truck. |
| motorcycle | Thân xe máy, không tính người. | Sidecar gắn kèm được tính vào motorcycle. |
| bicycle | Xe đạp, không tính người. | Xe đạp không có người vẫn annotate. |
| rider | Người điều khiển hoặc ngồi trên xe máy, xe đạp, kể cả người ngồi sau. | Một người một box. |
| pedestrian | Người đi bộ, đứng, ngồi không trên phương tiện. | Người đẩy xe không có động cơ vẫn là pedestrian. |
| tuktuk | Xe ba bánh tuk-tuk. | Không gán car hay motorcycle. |
| jeepney | Xe jeepney. | Nếu gặp trong dataset. |
| other_vehicle | Phương tiện khác: xe đạp kéo (cyclo), xe kéo, xe thồ. | Không được bỏ qua vì vẫn là vật cản. |
| traffic_light | Đèn giao thông. | Đề xuất, xem mục 8. |

## 4. Thuộc tính

| Thuộc tính | Giá trị | Quy tắc |
|---|---|---|
| occluded | true / false | true khi đối tượng bị vật khác che một phần. |
| truncated | true / false | true khi đối tượng bị cắt bởi biên ảnh. |

Có thể bật cả occluded và truncated cùng lúc. Không dùng thêm thuộc tính khác.

## 5. Quy tắc Bounding Box

### 5.1. Vẽ box
- Vẽ hình chữ nhật bao sát đối tượng, không rộng hơn và không cắt vào đối tượng.
- Mỗi đối tượng một box. Box được phép chồng lấn khi đối tượng gần nhau hoặc che nhau.
- Người trên xe máy hoặc xe đạp có hai box: rider và motorcycle (hoặc bicycle). Mỗi người ngồi trên xe là một rider riêng.
- Xe đậu thành hàng, chồng sát nhau: vẽ từng xe, không gộp.

### 5.2. Đối tượng bị che, bị cắt, xa hoặc nhỏ
- Đối tượng bị che hoặc bị cắt dưới 80% kích thước ước tính phải annotate.
- Đối tượng bị che trên 80% vẫn annotate nếu còn đủ bằng chứng để xác định lớp và vị trí. Không đủ thì bỏ qua và ghi Issue.
- Đối tượng quá nhỏ hoặc ở xa: annotate nếu xác định được lớp. Không xác định được thì ghi Issue LOW_VISIBILITY.

### 5.3. Phạm vi
- Annotate mọi xe và người thật trong khung hình, kể cả xe đậu bên lề và xe đang chở hàng.
- Xe nằm trên xe khác (ví dụ xe con trên xe chở xe) là đối tượng thật. Annotate và đánh dấu occluded nếu bị che.
- Không gán tượng, đồ chơi, mô hình, mannequin, hình vẽ trên tường, billboard hoặc quảng cáo có hình xe.
- Không gán vật thể nhìn thấy trong gương, kính hoặc phản chiếu trên mặt đường ướt.
- Xe thật nhưng đã được trang trí nhiều (ví dụ xe trong sự kiện) vẫn annotate nếu còn nhận ra là xe thật.

### 5.4. Kích thước ảnh
- Annotate trên ảnh gốc, không resize hoặc crop. Ảnh có nhiều kích thước khác nhau, giữ tọa độ gốc.

## 6. Phân biệt các lớp dễ nhầm

| Cặp | Cách phân biệt |
|---|---|
| car và van | car chở người, kích thước nhỏ. van cao hơn, nhiều chỗ, thường chở người tư nhân. |
| van và bus | bus chở khách công cộng, có logo hoặc tuyến cố định. Không chắc thì ghi UNCERTAIN_CLASS. |
| truck và pickup | Pickup có thùng hàng hở là truck. Pickup có khoang kín vẫn là truck. |
| tuktuk và motorcycle | tuktuk có ba bánh và khoang chở khách phía sau. motorcycle có hai bánh. |
| tuktuk và other_vehicle | cyclo là xe đạp có khoang chở khách phía trước hoặc sau, không có động cơ, gán other_vehicle. |
| motorcycle và bicycle | Xe có động cơ là motorcycle. Xe đạp không có động cơ là bicycle. |

## 7. Ca khó và quyết định

| Ảnh | Tình huống | Quyết định |
|---|---|---|
| 022 | Ảnh đen trắng, xe máy có sidecar trên đường đất. | Annotate motorcycle bao cả sidecar. Không tách sidecar thành box riêng. Người trên xe là rider. |
| 017, 027, 018, 033, 038 | Tượng ngựa và người cưỡi, xe tượng đài, billboard có hình xe, mannequin, kệ đồ chơi xe. | Không gán nhãn. Đây không phải đối tượng thật. |
| 020 | Trong gương chiếu hậu có hình xe và đường. | Không gán. Đây là phản chiếu. |
| 015, 031 | Mặt đường ướt phản chiếu xe và đèn. | Chỉ annotate xe thật. Không gán phần phản chiếu xuống mặt đường. |
| 023 | Cyclo (xe đạp kéo) cùng xe ô tô và xe máy. | Cyclo gán other_vehicle. Người đạp cyclo là rider vì đang điều khiển xe. Ô tô và xe máy gán theo lớp tương ứng. |
| 002 | Xe tải chở nhiều ô tô trên thùng. | Xe tải gán truck. Từng ô tô trên thùng gán car, đánh dấu occluded true nếu bị che. Không gán một box cho cả đoàn xe. |
| 001, 030 | Đường đông, xe máy chồng sát nhau, một phần bị che. | Vẽ từng motorcycle và rider. Đối tượng bị che trên 80% thì xử lý theo mục 5.2. |
| 024 | Xe con màu đen chở một xe nhỏ màu vàng trên nóc. | Xe nhỏ trên nóc gán car, đánh dấu occluded true nếu bị che. Xe chở gán car theo loại xe thật. |
| 014, 034 | Người băng qua đường ở vạch kẻ. | Gán pedestrian. Người đi bộ bị che bởi người khác vẫn annotate, đánh dấu occluded. |

## 8. Quyết định cần xác nhận
- **Đèn giao thông:** dataset này hầu như không có đèn tín hiệu rõ ràng. Đề xuất không gán traffic_light trong phiên bản này, để giảm nhiễu.
- **Lớp jeepney và tuktuk:** đề xuất giữ đúng như trong dataset trước để thống nhất.
- **Minibus:** đề xuất gán bus khi có dấu hiệu chở khách công cộng. Xác nhận cách xử lý với mentor.
- **Xe trong sự kiện có trang trí nặng (ảnh 019):** đề xuất annotate car. Cần mentor xác nhận vì ranh giới giữa xe thật và đạo cụ không rõ.

## 9. Issue và xuất dữ liệu
Định dạng Issue: số ảnh | lớp liên quan | issue type | mô tả ngắn. Issue type: UNCERTAIN_CLASS, UNCERTAIN_BOUNDARY, UNCERTAIN_SCOPE, ATTRIBUTE_CHECK, LOW_VISIBILITY.

Ví dụ: `023 | other_vehicle | UNCERTAIN_CLASS | cyclo phía xa, không chắc có phải xe đạp kéo.`

- Tên lớp và thuộc tính viết đúng như mục 3 và 4. Không viết hoa hay thêm dấu cách.
- Giữ tên file gốc img_001 đến img_040 để đối chiếu với mục 2 và 7.
