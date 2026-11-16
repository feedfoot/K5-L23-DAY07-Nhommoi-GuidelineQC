# Guideline gán nhãn — v0.1-cá nhân — 055

- Phiên bản: v0.1 (bản cá nhân, sửa từ v0)
- Ngày: 2026-11-16
- Người viết: 055 · Người duyệt: (nhóm, Pha 2)

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **phát hiện người và phương tiện** trên ảnh 2D đường phố (camera giao thông / xe tự lái).

- Sai nguy hiểm nhất: **bỏ sót người hoặc xe thật** đang tham gia giao thông (mô hình không thấy → không phanh / không cảnh báo).
- Sai thứ hai: **gán vật không thật** (tượng, đồ chơi, ảnh in) → mô hình báo động giả.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe 4 bánh chủ yếu để **chở người** (ô tô con, SUV, pickup chở người). | img_001 (pickup đỗ trước quán), img_005 (xe con bạc bên trái), img_002 (ô tô trên xe chở xe), img_019 (xe diễu hành) | `van` nếu thùng kín dùng chở hàng; tượng xe (img_027) → **không gán** |
| `truck` | Xe tải. ⚠ *Chưa chốt dấu hiệu phân biệt với `car`/`van`* — xem mục 6. | | |
| `van` | Xe thùng kín cỡ nhỏ chủ yếu để **chở vật / hàng**. | | `car` nếu dùng chở người |
| `motorcycle` | Xe 2–3 bánh có động cơ hoặc **xe chở khách dạng 3 bánh**: xe máy, mô tô, tuk-tuk, auto rickshaw, xích lô, xe có sidecar. | img_005, img_026, img_004 (tuk-tuk), img_035 (auto rickshaw), img_006 / img_009 / img_023 (xích lô), img_022 (sidecar) | Đồ chơi / mô hình xe → không gán |
| `person` | Người thật: đi bộ, lái xe, ngồi sau, **ngồi trong ô tô**. | img_005, img_026 | Tượng (img_017), ma-nơ-canh (img_033), người trên billboard (img_018), hình phản chiếu (img_020) → **không gán** |
| `ignore_region` | Vùng **đám đông** (nhiều người / xe chen nhau, không tách được từng vật). Một box phủ cả vùng; vật trong vùng không gán riêng. | img_025, img_032, img_040 | Vật nhỏ **đứng riêng lẻ** dưới ngưỡng R-03 → không gán, không dùng ignore |

**Vật không thuộc lớp nào / không phải vật thật** (R-05): **không gán**.

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | `0` | bị vật khác che **< 10%** |
| | `1` | bị che **10% – 50%** |
| | `2` | bị che **> 50%** |
| | `3` | **không xác định** được mức che ⚠ *(cần ví dụ cụ thể — mục 6)* |
| `truncated` | `yes` | vật bị **mép ảnh cắt, dù ít hay nhiều** |
| | `no` | vật nằm trọn trong ảnh |
| `can_review` | tick | ⚠ *chưa chốt — mục 6* |

- `occluded_level` chỉ tính phần bị **vật khác** che; phần nằm ngoài mép ảnh tính vào `truncated`, không tính vào `occluded_level`.
- Dùng thuộc tính `occluded_level` của nhóm, **không** dùng cờ occluded (phím `Q`) của CVAT.

## 3. Quy tắc vẽ box

| Mã | Quy tắc |
|---|---|
| **R-01** | **Người đi xe máy → 2 box tách riêng**: một box `person` cho người, một box `motorcycle` cho xe. Người ngồi sau (và mọi người trên xe) mỗi người **một box `person` riêng**. Hai box được phép chồng nhau. |
| **R-02** | **Box ôm cả phần bị che (ước lượng)**: khi vật bị vật khác che, vẽ box theo **kích thước ước lượng của cả vật**, không chỉ phần nhìn thấy. Phần bị mép ảnh cắt thì box dừng ở mép ảnh. |
| **R-03** | **Ngưỡng vẽ**: chỉ vẽ box khi **cả hai** điều kiện đúng: nhìn thấy **≥ 20% vật** **và** box cao **≥ 20 px**. Thiếu một trong hai → không vẽ (trừ khi nằm trong vùng R-10). |
| **R-04** | **Hàng chở không thuộc box xe**: box `motorcycle` / `car` chỉ ôm thân xe, không mở rộng để bao hàng hóa chở trên xe. |
| **R-05** | **Vật không thật không gán**: tượng, đài tưởng niệm, ma-nơ-canh, đồ chơi, mô hình, người/xe trên billboard hay hình in, hình phản chiếu. |
| **R-06** | **Xe 3 bánh chở khách = `motorcycle`**: tuk-tuk, auto rickshaw, xích lô, xe sidecar. |
| **R-07** | **Xe được chở trên xe khác vẫn gán**: mỗi ô tô trên xe chở xe là một box `car`. |
| **R-08** | **Xe giả tưởng / trang trí chạy được thì gán theo công dụng**: xe diễu hành chở người → `car`. |
| **R-09** | **Người trong ô tô vẫn gán `person`** nếu nhìn thấy qua kính và đạt R-03. |
| **R-10** | **Đám đông → `ignore_region`**: vùng nhiều người / xe chen nhau không tách được từng vật thì vẽ **một** box `ignore_region` phủ cả vùng, không gán từng vật bên trong. Vật nhỏ **riêng lẻ** dưới ngưỡng R-03 → không gán. |

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| img_005 | Nhiều người đi xe máy, chồng lấp nhau | R-01, R-02 | Mỗi người 1 `person`, mỗi xe 1 `motorcycle`; box ước lượng cả phần bị che | Mô hình cần đếm đúng từng người tham gia giao thông |
| img_026 | 2 người trên 1 xe máy | R-01 | 2 `person` + 1 `motorcycle` | Người ngồi sau cũng là người tham gia giao thông |
| img_004 | Tuk-tuk | R-06 | `motorcycle` | Taxonomy không có lớp 3 bánh; gần với xe máy nhất về động cơ và cách di chuyển |
| img_035 | Auto rickshaw chở quá tải | R-06, R-01 | `motorcycle` + mỗi người 1 `person` | như trên |
| img_006 / img_009 / img_023 | Xích lô | R-06 | `motorcycle` | như trên |
| img_022 | Xe máy có sidecar | R-06 | `motorcycle` | như trên |
| img_001 | Pickup đỗ trước quán | — | `car` | Dùng chở người |
| img_002 | Ô tô trên xe chở xe | R-07 | Mỗi ô tô 1 box `car` | Vẫn là ô tô thật, mô hình cần nhận ra |
| img_017 | Tượng người cưỡi ngựa | R-05 | Không gán | Không phải người thật |
| img_027 | Tượng đài hình xe pickup | R-05 | Không gán | Không phải xe thật |
| img_033 | Ma-nơ-canh | R-05 | Không gán | Không phải người thật |
| img_038 | Tủ kính đồ chơi, mô hình xe | R-05 | Không gán | Không phải vật thật |
| img_018 | Billboard có hình người | R-05 | Không gán | Hình in |
| img_020 | Hình trong gương xe | R-05 | Không gán | Hình phản chiếu |
| img_019 | Xe Dangermouse trong diễu hành | R-08 | `car` | Xe chạy được, chở người |
| img_029 | Xe máy chở hàng cồng kềnh | R-04 | Box `motorcycle` chỉ ôm xe | Hàng không phải phương tiện |
| img_025 / img_032 / img_040 | Đám đông, vật nhỏ ở xa | R-10, R-03 | Vùng đám đông → 1 box `ignore_region`; vật lẻ < 20 px hoặc thấy < 20% → không gán | Không tách được từng vật; dùng ignore để box lỡ vẽ / bỏ sót trong vùng không bị tính lỗi |

Ca chưa chốt: tick `can_review` trên box, hoặc ghi vấn đề cấp ảnh (`ảnh, vùng, lý do`).

## 5. QC và phiên bản

- Ngưỡng QC: theo mục `rules` trong `taxonomy.json` (IoU ≥ 0.85 với box cao ≥ 50 px, 0.7 với box nhỏ hơn; đồng thuận lớp ≥ 95%; lỗi theo ảnh ≤ 2%).
- Ca mơ hồ: người gán tick `can_review`; nhóm quyết ở họp calibration (Pha 5), ghi mã R-xx vào `calibration_decisions.csv`.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên | — |
| v0.1-055 | Thêm R-01 → R-10, ngưỡng `occluded_level`, `truncated`, định nghĩa `car`/`van` theo công dụng | img_005, 026, 004, 035, 002, 017, 027, 033, 019, 029 |

## 6. Điểm còn mở (cần 055 chốt trước khi gửi)

1. **R-03**: bộ phận tối thiểu (ví dụ "thấy đầu + vai") — để trống hay bổ sung?
2. **`truck`**: dấu hiệu phân biệt với `car` / `van` là gì? Pickup **chở hàng** (img_012, img_028, img_036, img_037) thuộc lớp nào?
3. **img_002**: đầu kéo của xe chở xe là lớp gì?
4. **`ignore_region`**: bao nhiêu vật chen nhau thì tính là "đám đông"?
5. **`occluded_level = 3`**: ví dụ một tình huống cụ thể.
6. **`can_review`**: khi nào tick?
7. **R-06**: xích lô không có động cơ → định nghĩa `motorcycle` dựa trên "chở khách" hay "có động cơ"? Xe đạp thường thì sao?
