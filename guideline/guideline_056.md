# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: …
- Ngày: …
- Người viết / duyệt: …

## 1. Mục tiêu
Mô hình huấn luyện cho xe tự hành dùng nhãn này, để làm traning và evaluation, sai ở việc gán nhầm box người lên ảnh, poster, hình nộm hoặc bỏ sót, sai lớp.

## 2. Lớp và thuộc tính
### Lớp
| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| car | Ô tô con ≤ 8 chỗ (sedan, SUV, hatchback, pickup không nắp thùng cao) | 8 xe con trên sàn xe chở, xe đậu sau hàng rào | Tượng xe bán tải ; xe mô hình đồ chơi ; pickup nắp thùng cao kín → xét van |
| van | Xe thùng kín cỡ nhỏ–trung, khoang hàng liền cabin, gồm minivan và pickup có nắp thùng kín cao | bán tải gắn nắp thùng cao | Xe buýt nhỏ (> 15 chỗ) → other_vehicle; xe tải thùng rời cabin → truck |
| truck | Xe tải có cabin và thùng hàng tách rời, gồm xe chở ô tô, xe ben, đầu kéo | xe tải Suzuki, đầu xe chở ô tô | Tượng xe tải ; van thùng kín liền cabin → van |
| other_vehicle | Phương tiện có động cơ ≥ 3 bánh không thuộc car/van/truck, gồm xe buýt, tuk-tuk, xe cứu hỏa | tuk-tuk 3 bánh; xe buýt | Xích lô đạp (không động cơ → bicycle); xe đạp điện rõ bàn đạp → bicycle |
| bicycle | Xe 2–3 bánh không động cơ, gồm xe đạp, xích lô đạp | xe đạp, xích lô | Xe máy/scooter (có động cơ → motorbike); xe đạp điện có rõ động cơ → motorbike |
| motorbike | Xe 2 bánh có động cơ, gồm xe máy, mô tô, scooter | xe máy trên đường, | Xe đạp (không động cơ → bicycle); tuk-tuk 3 bánh → other_vehicle |
| traffic_sign | Biển báo giao thông cố định do cơ quan giao thông lắp đặt (cấm, chỉ dẫn, cảnh báo) | Biển báo tốc độ, biển cấm rẽ bên đường | Biển quảng cáo  — KHÔNG gán; bảng tên cửa hàng — KHÔNG gán |
| traffic_light | Cụm đèn tín hiệu giao thông điều khiển xe hoặc người đi bộ | Đèn đỏ-vàng-xanh tại ngã tư | Đèn trang trí đèn màu quán ăn — KHÔNG gán; đèn chiếu sáng đường — KHÔNG gán |
| pedestrian | Người đi bộ hoặc đứng yên, không đang ngồi trên/trong phương tiện |người đứng bên quán,người đi bộ nền xa | Tượng người cưỡi ngựa  — KHÔNG gán; ma-nơ-canh  — KHÔNG gán; chân dung trên biển quảng cáo  — KHÔNG gán; người ngồi trên xe máy → gán riêng pedestrian cho phần người |
| goods | Hàng hóa/kiện hàng rời chất trên phương tiện, nhô ra ngoài thân xe rõ rệt |bao tải chất cao trên nóc tuk-tuk | Hàng trong thùng kín — KHÔNG gán; ô tô trên sàn xe chở  → gán car riêng, không gán goods |

Vật **không thuộc lớp nào** thì không gán.

### Thuộc tính

| Thuộc tính | Loại | Giá trị | Ngưỡng cụ thể | Mặc định |
|---|---|---|---|---|
| occluded_level | select | `0`, `1`, `2`, `3` | `0`: vật không bị che, thấy ≥ 95% diện tích. `1`: bị che 5–30% (ví dụ: nửa bánh xe khuất sau gốc cây). `2`: bị che 31–70% (ví dụ: nửa thân xe khuất sau xe khác). `3`: bị che > 70% nhưng vẫn nhận dạng được lớp (nếu không nhận dạng được → không gán box). | `0` |
| truncated | select | `no`, `yes` | `no`: toàn bộ vật nằm trong khung ảnh. `yes`: vật bị cắt bởi mép ảnh (bất kỳ cạnh nào của bbox chạm mép ảnh ± 2 px). | `no` |

## 3. Quy tắc vẽ box

**R-01 — Ôm sát phần nhìn thấy**: Box bao đúng phần vật thể hiện ra trong ảnh, không thêm phần bị che hoặc phần đoán ngoài mép ảnh. Khoảng trống giữa cạnh box và vật ≤ 3 px mỗi phía.

**R-02 — Vật bị che khuất**: Vẽ box ôm phần nhìn thấy của vật, chọn `occluded_level` tương ứng. Nếu phần nhìn thấy < 30% và không đủ để nhận dạng chắc chắn lớp → không gán box.

**R-03 — Vật bị cắt mép ảnh**: Vẫn gán box, cạnh box trùng mép ảnh, đánh `truncated = yes`. Nếu phần còn lại trong ảnh < 20% diện tích ước lượng của vật → không gán.

**R-04 — Phản chiếu / hình in / tượng / đồ chơi / ma-nơ-canh**: KHÔNG gán box. Chỉ gán vật thật, đang tồn tại vật lý trên đường hoặc vỉa hè. Cụ thể: tượng đồng/đá, ma-nơ-canh trong tủ kính, xe mô hình đồ chơi, chân dung trên biển quảng cáo, phương tiện phản chiếu qua gương chiếu hậu/kính cửa sổ — đều KHÔNG gán.

**R-05 — Xe chủ (ego vehicle)**: KHÔNG gán box cho xe đang mang camera chụp. Nếu thấy nắp ca-pô hoặc gương chiếu hậu của xe chủ trong ảnh → bỏ qua.

**R-06 — Vật quá nhỏ**: Nếu chiều cao box < 20 px → không gán. Nếu chiều cao 20–30 px → gán nhưng tick `can_review`.

**R-07 — Hàng hóa trên xe**: Box phương tiện chỉ ôm thân xe (cabin + thùng/khung), KHÔNG bao hàng hóa nhô ra ngoài thân xe. Hàng hóa nhô rõ rệt → gán box riêng lớp `goods`.

**R-08 — Xe chở xe**: Gán 1 box `truck` cho xe đầu kéo/xe chở. Mỗi xe con trên sàn gán riêng 1 box `car` với `occluded_level` phù hợp (xe bị che bởi xe khác hoặc khung sàn).

**R-09 — Người đi xe máy**: Gán 2 box tách rời: 1 box `motorbike` ôm toàn bộ xe, 1 box `pedestrian` ôm phần thân người (từ đầu đến chân/yên xe). Hai box được phép chồng nhau.

**R-10 — Đám đông / cụm vật dày đặc**: Nếu có thể tách từng vật riêng biệt → gán từng box. Nếu cụm vật quá dày (> 10 vật, mỗi vật < 20 px) không tách được → vẽ 1 box `ignore_region` bao cả cụm.

**R-11 — Motion blur / ánh sáng yếu**: Nếu vẫn nhận dạng được lớp → gán box bình thường, tick `can_review` nếu không chắc chắn. Nếu nhòe hoàn toàn không nhận dạng được → không gán.

## 4. Ca khó
| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_019 | Xe diễu hành Danger Mouse (xe vàng biển YR04 NUM) — xe giả tưởng biến dạng có phải car không? | R-04 | Gán `car` | Xe thật đang chạy trên đường, có biển số đăng ký, có 4 bánh ≤ 8 chỗ. Dù hình dáng biến dạng, nó vẫn là phương tiện thật tham gia giao thông → gán bình thường. |
| img_019 | Hình nộm chuột Danger Mouse ngồi ghế phụ — có gán pedestrian không? | R-04 | KHÔNG gán | Là mascot/hình nộm, không phải người thật → R-04 loại tượng/hình nộm/ma-nơ-canh. |
| img_019 | Người lái xe ngồi trong xe diễu hành — có gán pedestrian không? | R-09 | KHÔNG gán pedestrian | Định nghĩa pedestrian: người đi bộ hoặc đứng yên, KHÔNG đang ngồi trên/trong phương tiện. Người ngồi trong ô tô không gán pedestrian (R-09 chỉ tách box người khi đi xe máy, vì trên xe máy người lộ rõ toàn thân). |
| img_019 | Đám đông khán giả sau hàng rào — nhiều người nhỏ, chen chúc | R-06, R-10 | Người ≥ 20 px → gán `pedestrian` từng người. Cụm người < 20 px không tách được → 1 box `ignore_region` | Ưu tiên gán từng người nếu đủ lớn. Hàng sau xa, mỗi người < 20 px và dính nhau → gộp ignore. |
| img_017 | Tượng đồng người cưỡi ngựa (Duke of Wellington, Glasgow) — trông rất giống người thật | R-04 | KHÔNG gán | Là tượng đồng cố định, không phải người/phương tiện thật. Dù kích thước lớn và chi tiết giống thật, R-04 loại rõ tượng đồng/đá. Ảnh này có 0 annotation. |
| img_017 | Cọc giao thông (traffic cone) đặt trên đầu tượng | — | KHÔNG gán | Cọc giao thông không thuộc lớp nào trong taxonomy (không phải traffic_sign, không phải phương tiện, không phải người). |
| img_011 | Tượng kim loại Predator cỡ lớn ở tiền cảnh — hình dáng giống người | R-04 | KHÔNG gán | Tượng/tác phẩm điêu khắc kim loại, không phải người thật → R-04. Lỗi nguy hiểm nếu gán nhầm: mô hình học feature tượng = người. |
| img_011 | Mô hình xe máy bằng kim loại (tác phẩm nghệ thuật) bên cạnh tượng | R-04 | KHÔNG gán | Là tác phẩm điêu khắc xe máy, không phải xe thật đang lưu thông → R-04 loại mô hình/đồ chơi. |
| img_011 | Người thật đi bộ ở nền giữa (bên trái, cầm gậy) | — | Gán `pedestrian` | Người thật, đứng/đi bộ, đủ lớn (> 20 px) → gán bình thường. Cần phân biệt rõ với tượng ở tiền cảnh. |
| img_011 | Xe ô tô/van thật ở nền xa (giữa ảnh, sau lều trắng) | R-06 | Gán `car` nếu ≥ 20 px, tick `can_review` nếu 20–30 px | Xe thật nhưng rất xa và nhỏ. Đo chiều cao box: nếu < 20 px → bỏ theo R-06. |

Nếu chưa chốt, đánh dấu `can_review` trên box hoặc ghi vấn đề cấp ảnh. Dùng mã quy tắc trong nhật ký QC.

## 5. QC và phiên bản

### Ngưỡng QC

| Chỉ số | Ngưỡng | Ý nghĩa |
|---|---|---|
| Chiều cao box tối thiểu | ≥ 20 px | Vật < 20 px không gán (R-06) |
| IoU ghép cặp box | ≥ 0.5 | Hai box cùng ảnh ghép 1–1 khi IoU ≥ 0.5 |
| IoU trung bình (IAA) | ≥ 0.85 | Đo độ khớp vị trí/kích thước giữa 2 người gán |
| Đồng thuận lớp (IAA) | ≥ 95% | Tỉ lệ cặp box ghép có cùng lớp |
| Tỉ lệ ảnh lỗi tối đa | ≤ 2% | Vượt → Rework; đạt + đủ bằng chứng → Release |

### Quy trình xử lý ca mơ hồ

1. Trong lúc gán: tick `can_review` trên box hoặc ghi vào `image_issues.csv` nếu chưa vẽ được box.
2. Sau khi đo IAA: các ca bất đồng xuất hiện trong `calibration_candidates.csv`.
3. Họp calibration: nhóm chốt từng ca, gắn mã quy tắc (R-01…R-11), ghi vào `calibration_decisions.csv` (9 cột).
4. Nếu quyết định đổi cách chọn vật/lớp/box/thuộc tính (`kind: "semantic"`) → kiểm lại toàn bộ 40 ảnh.
5. Nếu chỉ sửa câu chữ (`kind: "editorial"`) → chỉ sửa ảnh bị cờ.

### Cây quyết định bàn giao

- **Hold**: thiếu bằng chứng, còn box `can_review`, guideline có lỗ hổng, hoặc chỉ duyệt mẫu (không duyệt toàn lô).
- **Rework**: tỉ lệ ảnh lỗi > 2%, hoặc có lỗi lặp lại (≥ 3 lỗi cùng loại chiếm ≥ 50% tổng lỗi). Sửa xong → mở vòng QC mới, giữ nguyên vòng cũ để truy vết.
- **Release**: duyệt toàn lô 40/40, ảnh lỗi ≤ 2%, không lỗi lặp lại, đủ bằng chứng hợp lệ.

### Lịch sử phiên bản

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên, chỉ có 5 lớp (car, truck, van, motorcycle, person), thuộc tính mô tả định tính, chưa có ca khó | Bản gốc sơ sài, cố ý thiếu để nhóm tự phát hiện lỗ hổng |
| v1.0 | Thêm 5 lớp (other_vehicle, bicycle, traffic_sign, traffic_light, goods), đổi motorcycle → motorbike, person → pedestrian. Thêm ngưỡng % cho occluded_level. Viết 11 quy tắc R-01…R-11. Bổ sung bảng ca khó 3 ảnh | Phát hiện tuk-tuk/bus/xe đạp không có lớp (img_004, img_009, img_006); tượng/ma-nơ-canh gây nhầm (img_017, img_033); đám đông img_019 cần ignore_region |
| v1.1 | *(Dành cho sau calibration — cập nhật quy tắc theo kết quả IAA và họp nhóm)* | *(Ghi sau khi hoàn thành Pha 5)* |
