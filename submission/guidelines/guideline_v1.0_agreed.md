# Guideline gán nhãn — v1.0 (thống nhất) — Nhóm Nhommoi

- Phiên bản: **v1.0** — khóa cho lượt gán độc lập (Pha 3)
- Ngày: 2026-11-16
- Tổng hợp từ: `guideline_055.md`, `guideline_043.md`, `guideline_056.md`
- Duyệt: 055 · 043 · 056 (xác nhận ở mục 7)
- Taxonomy đi kèm: `submission/taxonomy.json` (version `v1.0`)

---

## 1. Mục tiêu

Nhãn dùng để huấn luyện và đánh giá mô hình **phát hiện người và phương tiện giao thông thật** trên ảnh 2D đường phố (camera giao thông / xe tự lái), kèm mức bị che và bị cắt mép.

- **Sai nguy hiểm nhất:** bỏ sót người hoặc phương tiện thật → mô hình không phanh / không cảnh báo.
- **Sai thứ hai:** gán vật không thật (tượng, đồ chơi, ma-nơ-canh, ảnh in, phản chiếu) → báo động giả, mô hình học sai đặc trưng.
- **Ngoài phạm vi v1.0:** biển báo, đèn giao thông, hàng hóa, cọc giao thông (không gán).

**Phép thử cho mọi quy tắc:** *quyết định có dựa trên thứ nhìn thấy được trong ảnh không?* Nếu phải đoán công dụng hay ý định (xe chở người hay chở hàng?) thì quy tắc chưa đủ tốt.

---

## 2. Lớp và thuộc tính

### 2.1 Lớp (9 lớp)

| Lớp | Định nghĩa (dựa trên hình dáng nhìn thấy) | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Xe 4 bánh cỡ nhỏ chở người: sedan, hatchback, SUV, wagon, coupe, taxi, xe mui trần, xe diễu hành chạy được. | img_005 (xe con bạc bên trái), img_002 (ô tô trên xe chở xe), img_019 (xe diễu hành), img_024 (xe đen) | Pickup → `truck`; thân hộp cao → `van`; tượng xe (img_027), đồ chơi (img_038) → không gán |
| `van` | Xe 4 bánh **thân hộp cao, khoang kín liền cabin**, chở khách ≤ 15 chỗ hoặc chở hàng (minivan, xe van). | img_005 (van bạc biển 51LD-70.65), img_011 (van trắng nền xa) | Xe > 15 chỗ → `bus`; cabin tách thùng → `truck` |
| `bus` | Xe chở khách **> 15 chỗ**: thân dài, hàng cửa sổ dọc thân (xe buýt, minibus lớn). | img_037 (xe buýt xanh bên trái) | Xe van nhỏ → `van` |
| `truck` | Xe có **cabin tách khỏi thùng / sàn hàng**: xe tải, đầu kéo, xe ben, xe chở ô tô, **mọi xe pickup** (kể cả có nắp thùng). | img_002 (xe chở xe), img_001, img_012, img_028, img_036, img_037 (pickup) | Thân hộp liền cabin → `van` |
| `motorcycle` | Xe **2 bánh có động cơ**: xe máy, mô tô, scooter, xe đạp điện. Xe máy có **sidecar** → 1 box bao cả sidecar. | img_005, img_026, img_030, img_022 (sidecar), img_035 (xe máy phải) | Không động cơ → `bicycle`; 3 bánh → `other_vehicle` |
| `bicycle` | Xe **2 bánh không động cơ** (xe đạp), có người hay không. | img_003 (xe đạp có giỏ bên trái) | Xích lô 3 bánh → `other_vehicle` |
| `other_vehicle` | Phương tiện thật **không thuộc các lớp trên**, chủ yếu xe **3 bánh**: tuk-tuk, auto rickshaw, xích lô, xe ba gác, xe kéo / xe thồ có bánh. | img_004 (tuk-tuk), img_035 (auto rickshaw vàng), img_006 / img_009 / img_023 / img_034 / img_039 (xích lô) | Xe máy + sidecar → `motorcycle` |
| `person` | **Người thật ở ngoài khoang xe kín**: đi bộ, đứng, ngồi, người lái / ngồi sau xe máy, xe đạp, xích lô, tuk-tuk, người trên thùng pickup. Người hóa trang vẫn là `person`. | img_005, img_026, img_014, img_011 (người đứng bên trái) | Tượng (img_017, img_011 Predator), ma-nơ-canh (img_033), mascot / hình nộm (chuột trong img_019), người trên billboard (img_018) → không gán; người ngồi trong ô tô → R-05 |
| `ignore_region` | Vùng **đám đông** không gán từng vật (R-10). Không có thuộc tính. | img_019 (khán giả sau rào), img_032 (người chờ ở trạm), img_025, img_040 | Một nhóm nhỏ tách được từng người → gán từng `person` |

**Vật không thuộc lớp nào** (biển báo, đèn, cọc giao thông, hàng hóa, động vật): **không gán**.

### 2.2 Thuộc tính (cho mọi lớp trừ `ignore_region`)

| Thuộc tính | Kiểu | Giá trị | Ngưỡng |
|---|---|---|---|
| `occluded_level` | select | `0` | bị vật khác che **< 10%** diện tích ước lượng |
| | | `1` | bị che **10% – 50%** |
| | | `2` | bị che **> 50%** (vẫn thấy ≥ 20%, xem R-02) |
| | | `3` | **không ước lượng được** phần bị che: vật trong bóng tối, nhòe, hoặc chồng chéo dày đặc tới mức không thấy ranh giới |
| `truncated` | select | `yes` | vật bị **mép ảnh** cắt (cạnh box chạm mép ảnh ± 2 px **và** vật còn tiếp ra ngoài) — dù cắt ít hay nhiều |
| | | `no` | vật nằm trọn trong ảnh |
| `can_review` | checkbox | tick | xem R-13 |

Quy ước đo:
- `occluded_level` **chỉ tính phần bị vật khác che**; phần nằm ngoài mép ảnh tính vào `truncated`.
- **Người ngồi trên xe không tính là che chiếc xe đó** (R-04). Người khác, xe khác, cột, cây… thì tính.
- Không dùng cờ occluded có sẵn của CVAT (phím `Q`), chỉ dùng `occluded_level`.

---

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Box ôm sát phần nhìn thấy.** Không ước lượng phần bị che, không đoán phần ngoài mép ảnh. Khe hở giữa cạnh box và vật ≤ 3 px mỗi phía. |
| **R-02** | **Ngưỡng vẽ — phải đủ cả 3:** (a) box cao **≥ 20 px**; (b) thấy **≥ 20%** vật (ước lượng); (c) **nhận ra được lớp**. Thiếu một điều kiện → không vẽ (trừ khi nằm trong vùng R-10). |
| **R-03** | **Cắt mép ảnh:** vẫn vẽ nếu đạt R-02; cạnh box trùng mép ảnh; `truncated = yes`. |
| **R-04** | **Người trên xe hở → box tách riêng.** Mỗi người trên xe máy, xe đạp, xích lô, tuk-tuk = **một box `person`** (cả người lái, người ngồi sau, trẻ em); xe = **một box** theo lớp xe. Box `person` từ đỉnh đầu (gồm mũ bảo hiểm) tới điểm thấp nhất nhìn thấy của người. Box xe bao toàn bộ phần xe nhìn thấy. Hai box được chồng nhau. Người trên xe **không** tính vào `occluded_level` của xe đó. |
| **R-05** | **Người trong khoang xe** (ô tô, van, bus, cabin xe tải — kể cả xe mui trần) **không gán `person`**. Lý do: không phải người tham gia giao thông độc lập, thường chỉ thấy qua kính, dễ bất đồng. |
| **R-06** | **Vật không thật → không gán:** tượng, tượng đài, tác phẩm điêu khắc, mô hình, đồ chơi, ma-nơ-canh, mascot / hình nộm, người hoặc xe trên billboard / poster / tranh tường, **mọi hình phản chiếu** (gương chiếu hậu, kính, mặt đường ướt). |
| **R-07** | **Xe chở xe:** đầu kéo + rơ-moóc nối liền = **một box `truck`**. Mỗi xe chở trên sàn = một box riêng theo lớp của nó, `occluded_level` theo phần bị khung sàn / xe khác che. |
| **R-08** | **Hàng chở không thuộc box xe:** box xe chỉ ôm thân xe; hàng nhô ra ngoài thân xe không bao vào và **không gán**. |
| **R-09** | **Xe thật đang đỗ, trưng bày, trong nhà, được trang trí hoặc diễu hành → vẫn gán** theo hình dáng. |
| **R-10** | **Đám đông → `ignore_region`.** Cụm **≥ 10** người / xe chồng lấn nhau mà phần lớn bị che > 50% hoặc cao < 20 px → vẽ **một** box `ignore_region` phủ cả cụm. Vật ở rìa cụm thấy rõ (đạt R-02, bị che ≤ 50%) vẫn gán riêng. |
| **R-11** | **Xe chủ (ego vehicle)** mang camera: không gán (nắp ca-pô, gương, khung xe chủ). |
| **R-12** | **Ảnh tối / nhòe / mưa:** nhận ra được lớp → gán bình thường; không nhận ra → không gán. |
| **R-13** | **`can_review`:** tick khi người gán **không chắc** về (a) lớp, (b) vật thật hay giả, hoặc (c) ranh giới box. Không tick theo kích thước. Ca chưa vẽ được box → ghi vấn đề cấp ảnh `ảnh, vùng, lý do` (vào `image_issues.csv`). |

---

## 4. Ca khó
| Ảnh | Tình huống | Rule | Quyết định v1.0 |
|---|---|---|---|
| img_005 | Nhiều người đi xe máy chồng lấp; người phía sau chỉ thấy mũ | R-04, R-02, R-01 | Mỗi người đạt R-02 = 1 `person`; người chỉ thấy mũ (< 20%) → không gán; xe máy chỉ thấy tay lái → kiểm R-02 |
| img_026 | 2 người trên 1 xe máy, nền là tranh in | R-04, R-06 | 2 `person` + 1 `motorcycle`; tranh nền không gán |
| img_022 | Xe máy có sidecar | R-04 | 1 `motorcycle` bao cả sidecar; mỗi người 1 `person` |
| img_004 | Hai tuk-tuk đỗ | R-09 | 2 `other_vehicle` |
| img_035 | Auto rickshaw chở hàng cao; xe máy; xe con | R-08, R-04 | `other_vehicle` chỉ ôm thân xe, hàng không gán; người lái rickshaw (khoang hở) → `person`; xe máy + người → R-04; xe trắng → `car` |
| img_006, 009, 023, 034, 039 | Xích lô | R-04 | `other_vehicle`; người đạp và khách → `person` |
| img_002 | Xe chở 8 ô tô | R-07 | 1 `truck` (đầu kéo + sàn); mỗi ô tô 1 `car` |
| img_024 | Bảo tàng xe: xe vàng trên nóc xe đen | R-09, R-13 | Xe đen, các xe trưng bày → `car`. Xe vàng trên nóc: nếu là xe thật cỡ nhỏ → `car`; không chắc là xe thật hay mô hình → `car` + tick `can_review` |
| img_001, 012, 028, 036, 037 | Xe pickup (có / không nắp thùng) | lớp `truck` | `truck` |
| img_037 | Pickup nắp thùng cao + xe buýt nền + người đi bộ | lớp `truck`, `bus` | pickup `truck`; xe buýt `bus` (`truncated` nếu chạm mép); người `person` |
| img_019 | Xe diễu hành, tài xế, mascot chuột, khán giả | R-09, R-05, R-06, R-10 | xe → `car`; tài xế trong xe → không gán; mascot → không gán; khán giả dày sau rào → `ignore_region`, người ở rìa thấy rõ → `person` |
| img_011 | Tượng Predator, xe máy nghệ thuật bằng phế liệu, người, van | R-06, R-13 | Tượng → không gán; xe máy nghệ thuật: không chắc là xe chạy được → `motorcycle` + `can_review`; người bên trái → `person`; van trắng → `van` |
| img_017 | Tượng người cưỡi ngựa, cọc giao thông | R-06 | Không gán gì (0 box) |
| img_027 | Tượng đài hình pickup | R-06 | Không gán |
| img_033 | Ma-nơ-canh | R-06 | Không gán |
| img_038 | Tủ kính đồ chơi, mô hình xe | R-06 | Không gán |
| img_018 | Billboard có hình người | R-06 | Không gán phần trong billboard |
| img_020 | Hình trong gương xe | R-06, R-11 | Không gán phản chiếu và xe chủ |
| img_015, 031 | Đường ướt phản chiếu xe | R-06, R-12 | Chỉ gán xe thật |
| img_029 | Xe chở hàng cồng kềnh | R-08 | Box xe chỉ ôm thân xe |
| img_032 | Đám đông chờ ở trạm xe buýt, cọc giao thông, xe con | R-10, R-03 | Cụm người chồng lấn dày → `ignore_region`; người ở rìa thấy rõ → `person`; xe xanh bên trái → `car`, `truncated = yes`; cọc giao thông không gán |
| img_003 | Người bán hàng gánh, xe đạp, xe máy dựng | R-04, R-02 | Mỗi người 1 `person`; xe đạp `bicycle`; xe máy đỗ đạt R-02 → `motorcycle`; quang gánh, hàng không gán |
| img_025, 040 | Đám đông / nhiều vật nhỏ | R-10, R-02 | Cụm dày → `ignore_region`; vật rõ → gán riêng |

---

## 5. Bất đồng giữa 3 bản cá nhân và cách chốt

| # | Chủ đề | 055 | 043 | 056 | **v1.0 chốt** | Lý do |
|---|---|---|---|---|---|---|
| D1 | Bộ lớp | 5 lớp v0 | 12 lớp (rider, pedestrian, tuktuk, jeepney, traffic_light…) | 10 lớp (traffic_sign, traffic_light, goods…) | 9 lớp: thêm `bus`, `bicycle`, `other_vehicle`; bỏ biển báo, đèn, goods, jeepney | Mục tiêu là người và phương tiện; dataset không có jeepney; biển, đèn, hàng ngoài phạm vi; ít lớp hơn → ít bất đồng hơn |
| D2 | Lớp người | `person` | `rider` + `pedestrian` | `pedestrian` cho mọi người (mâu thuẫn định nghĩa) | Một lớp `person` | Phân biệt rider / pedestrian phải đoán (người đứng cạnh xe, người đẩy xe); vai trò suy ra từ box chồng với xe |
| D3 | Tuk-tuk / rickshaw | `motorcycle` | `tuktuk` | `other_vehicle` | `other_vehicle` | 2/3 tách khỏi xe máy; giữ `motorcycle` = đúng 2 bánh, quy tắc theo số bánh dễ kiểm |
| D4 | Xích lô | `motorcycle` | `other_vehicle` | `bicycle` | `other_vehicle` | Xích lô không động cơ (≠ xe máy) và 3 bánh (≠ xe đạp 2 bánh) |
| D5 | Pickup | `car` (img_001) | `truck` | `car`; nắp cao → `van` | `truck` (mọi pickup) | Cabin tách thùng = định nghĩa `truck`; một quy tắc cho mọi pickup, không phải xét nắp |
| D6 | `van` | chở **vật** | chở **người** | thùng kín liền cabin | theo **hình dáng**: thân hộp liền cabin | 055 và 043 trái ngược nhau vì dựa vào công dụng — mô hình không nhìn thấy công dụng |
| D7 | Xe buýt | không có | `bus` | `other_vehicle` | `bus` (> 15 chỗ) | Có trong dataset (img_037), dễ nhận, quan trọng cho giao thông |
| D8 | Box bị che | ước lượng cả phần bị che | "bao sát" (không rõ) | phần nhìn thấy | **phần nhìn thấy** (R-01) | Ước lượng phần bị che khác nhau giữa người gán → khó đạt IoU 0.85; phần nhìn thấy kiểm được bằng mắt |
| D9 | Ngưỡng vẽ | ≥ 20% **và** ≥ 20 px | che < 80% gán; > 80% nếu nhận ra | < 30% & không nhận ra → bỏ; < 20 px bỏ | ≥ 20 px **và** thấy ≥ 20% **và** nhận ra lớp (R-02) | Gộp cả ba; 20 px khớp `min_box_height_px` của lab |
| D10 | `occluded_level` | 0 < 10, 1 10–50, 2 > 50, 3 không rõ | true / false | 0 < 5, 1 5–30, 2 31–70, 3 > 70 | theo 055; mức 3 = không ước lượng được | Lab cố định 4 mức; ước lượng 5% khó lặp lại; > 80% đã bị R-02 loại |
| D11 | Người trong ô tô | gán | (không nói) | không gán | **không gán** (R-05) | Thấy qua kính, thường < 20%, dễ bất đồng; không phải người tham gia giao thông độc lập |
| D12 | Hàng hóa | không bao vào box | (không nói) | lớp `goods` riêng | không bao, không gán (R-08) | Hàng ngoài mục tiêu; thêm lớp làm tăng bất đồng |
| D13 | Đám đông | `ignore_region` | ghi Issue LOW_VISIBILITY | `ignore_region` khi > 10 vật mỗi vật < 20 px | `ignore_region` khi ≥ 10 vật chồng lấn, đa số che > 50% hoặc < 20 px (R-10) | Theo 056 nhưng mở rộng cho đám đông lớn ảnh gần (img_019) — gán từng người ở đó gây bất đồng nhiều |
| D14 | `can_review` | (chưa chốt) | Issue types | tick khi box 20–30 px | tick khi không chắc lớp / thật-giả / ranh giới (R-13) | Tick theo kích thước sinh quá nhiều ca calibration không cần thiết |
| D15 | Thuộc tính | 3 thuộc tính lab | `occluded` bool, bỏ `can_review` | 3 thuộc tính lab | giữ 3 thuộc tính lab | Script bắt buộc `can_review` kiểu checkbox; lab dùng `occluded_level` 4 mức |

**Đồng thuận sẵn cả 3 bản:** người đi xe máy = 2 box tách; không gán tượng, ma-nơ-canh, đồ chơi, billboard, phản chiếu; xe chở xe = `truck` + từng `car`; xe diễu hành img_019 = `car`; `truncated = yes` khi bị mép ảnh cắt.

**Bổ sung từ một bản:** R-11 xe chủ, R-12 ảnh nhòe (056); phản chiếu mặt đường ướt, xe trên nóc img_024 (043).

---

## 6. QC và phiên bản

- Ngưỡng theo mục `rules` của `taxonomy.json` (không sửa): IoU ≥ 0.85 (box ≥ 50 px) / 0.7 (nhỏ hơn); IAA lớp ≥ 95%; ghép cặp IoU ≥ 0.5; lỗi theo ảnh ≤ 2%.
- Ca mơ hồ khi gán: `can_review` (R-13) hoặc vấn đề cấp ảnh. Nhóm chốt ở calibration (Pha 5), ghi mã R-xx vào `calibration_decisions.csv`.
- Người chốt ca mơ hồ: cả nhóm, đa số 2/3; người review cuối xác nhận.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v1.0 | 9 lớp, ngưỡng thuộc tính, R-01 → R-13, bảng ca khó mục 4 | Tổng hợp 3 bản cá nhân, bảng D1–D15 |
| v1.1 | *(sau calibration)* | |

---

## 7. Xác nhận khóa v1.0

Phép thử trước khi khóa: mỗi người đếm box trên **img_005** và **img_035** theo v1.0 — ba kết quả phải trùng.

| Mã | Đồng ý D1–D15 | Số box img_005 | Số box img_035 | Ghi chú |
|---|---|---|---|---|
| 055 | ☐ | | | |
| 043 | ☐ | | | |
| 056 | ☐ | | | |
