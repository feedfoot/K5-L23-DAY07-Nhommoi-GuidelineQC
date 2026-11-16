# Sổ tay nhóm Nhommoi — Day 07

Thành viên: `055` · `043` · `056` → mỗi ảnh **2 người** gán, 80 lượt, mỗi người ~26–27 ảnh.
Chạy mọi lệnh từ thư mục gốc repo. Chi tiết gốc: [GUIDE.md](GUIDE.md), chấm điểm: [RUBRIC.md](RUBRIC.md).

> Sổ tay này chỉ là **quy trình**. Mọi quyết định gán nhãn do nhóm tự viết trong guideline.

---

## 0. Phân vai (điền trước khi bắt đầu)

| Vai | Mã | Việc |
|---|---|---|
| Điều phối | … | giữ giờ, chạy script chung (`split_assign`, `qc_compare`, `build_final`), tập hợp file |
| Thư ký guideline | … | ghi biên bản họp Pha 2 & 5, viết `v1.0`, `v1.1`, `taxonomy.json` |
| Reviewer QC | … | duyệt lô ở Pha 6 — **không** là người sửa nhãn ở Pha 5 |

Ghi điều phối + reviewer vào [TEAM.md](TEAM.md).

---

## 1. Dòng thời gian

| Mốc | Pha | Ai | Ra file |
|---|---|---|---|
| 0:00–0:15 | 0 Chuẩn bị | cả nhóm | `TEAM.md` |
| 0:15–0:50 | 1 Guideline cá nhân — **độc lập** | mỗi người | `submission/guidelines/individual/guideline_<mã>.md` |
| 0:50–1:20 | 2 Họp chốt v1.0 | cả nhóm | `guideline_v1.0_agreed.md`, `taxonomy.json`, `cvat/labels_generated.json` |
| 1:20–1:30 | Nghỉ | | |
| 1:30–2:30 | 3 Gán — **độc lập** | mỗi người | `submission/labels/<mã>.json` |
| 2:30–2:50 | 4 IAA | điều phối chạy, cả nhóm đọc | `iaa_summary.md`, `calibration_candidates.csv` |
| 2:50–3:30 | 5 Calibration | cả nhóm | `calibration_decisions.csv`, `guideline_v1.1_calibrated.md`, `labels_calibrated.json` |
| 3:30–4:00 | 6 QC & nộp | reviewer + cả nhóm | `QC_REPORT.md`, `individual_notes/<mã>.md` |

---

## Pha 0 — Chuẩn bị

```bash
python -m pip install -r requirements.txt
```
- Đăng nhập CVAT (`localhost:8080`). Lỗi *Bad Gateway* → `docker ps -a`, xem container nào `Restarting`, đọc `docker logs <tên>`.
- Hỏi coach **kênh nộp riêng** (dùng ở Pha 1 và Pha 3).

## Pha 1 — Guideline cá nhân (KHÔNG bàn nhau)

1. Copy `guideline/TEMPLATE_guideline.md` → `submission/guidelines/individual/guideline_<mã>.md`.
2. Xem **cả 40 ảnh** trong `data/images/` (chưa cần CVAT).
3. Viết đủ 5 mục. Mỗi quyết định = **ảnh `img_0xx` → quyết định → lý do**.
4. Gửi riêng qua kênh coach. Chỉ mở bản của nhau khi **cả 3** đã gửi.

### Câu hỏi guideline bắt buộc phải trả lời (tự trả lời, kèm ID ảnh)

| # | Chủ đề | Câu hỏi | Ảnh gợi ý soi |
|---|---|---|---|
| Q1 | Người + xe máy | Mấy box? Box `person` bao tới đâu? Người ngồi sau, trẻ em ngồi trước? | 005, 026, 030 |
| Q2 | Phần nhìn thấy | Box ôm phần **nhìn thấy** hay **ước lượng** phần bị che? | 005, 026 |
| Q3 | Ngưỡng vẽ | Thấy bao nhiêu (%, chiều cao px, bộ phận nào) mới vẽ? Dưới ngưỡng thì sao? | 005, 014, 032 |
| Q4 | `occluded_level` | Ngưỡng % cho mức 0/1/2; mức 3 "không rõ" nghĩa là gì? | ảnh đông |
| Q5 | `truncated` | Khác `occluded` thế nào? Cắt mép bao nhiêu thì `yes`? | 005 |
| Q6 | Xe ngoài taxonomy | Tuk-tuk, auto rickshaw, xích lô, sidecar, xe đạp: lớp nào / ignore / bỏ? | 004, 035, 006, 009, 023, 022 |
| Q7 | `car`/`truck`/`van` | Dấu hiệu phân biệt? Pickup thuộc lớp nào? | 001, 012, 028, 036, 037 |
| Q8 | Xe chở xe | Vẽ xe trên rơ-moóc không? Đầu kéo lớp gì? | 002 |
| Q9 | Vật không thật | Tượng, đài tưởng niệm, ma-nơ-canh, đồ chơi, billboard, hình in, phản chiếu | 017, 027, 033, 038, 018, 020, 026 |
| Q10 | Xe giả tưởng / trang trí | Lớp gì? Người hóa trang? | 019, 011 |
| Q11 | Hàng chở | Hàng trên xe máy / thùng xe có nằm trong box xe? | 029, 012 |
| Q12 | Đám đông / vật nhỏ | Khi nào dùng `ignore_region`? Box ignore vẽ thế nào? | 025, 032, 040 |
| Q13 | Người trong xe | Người sau kính xe có vẽ không? | 005, 015 |
| Q14 | Xe đỗ / đỗ trong nhà | Có khác xe đang chạy không? | 007, 016 |
| Q15 | `can_review` | Khi nào tick? Ai quyết ca mơ hồ? | — |

**Mục tiêu điểm (15đ):** ≥ 8 ca thật có ID, mỗi lớp có ví dụ + phản ví dụ, thuộc tính có ngưỡng đo được.

## Pha 2 — Họp chốt v1.0 (30')

Quy trình họp:
1. Đặt 3 bản cạnh nhau, đi lần lượt Q1 → Q15. Câu nào **3 người khác nhau** → thảo luận, chốt, **ghi lý do**.
2. Gắn mã quy tắc `R-01`, `R-02`… Ghi trong v1.0 một bảng *"Bất đồng giữa bản cá nhân → quyết định → lý do"* (rubric chấm 6đ riêng phần này).
3. Lưu `submission/guidelines/guideline_v1.0_agreed.md`.
4. Sửa `submission/taxonomy.json`: `"version": "v1.0"`, cập nhật `definition` từng lớp cho khớp guideline. **KHÔNG đụng mục `rules`.**
5. Chạy:
   ```bash
   python scripts/make_cvat_labels.py
   ```
6. Phép thử khóa: mỗi người đếm box img_005 theo v1.0 → **ba con số phải bằng nhau**. Chưa bằng → sửa câu chữ.
7. Commit, coi v1.0 là **đã khóa**.

## Pha 3 — Chia ảnh & gán độc lập (60')

Điều phối chạy (một lần, commit `assignment.csv` cho cả nhóm):
```bash
python scripts/split_assign.py --members 055,043,056 --copy-images
```
Mỗi người:
1. CVAT: **một Project chung** tạo từ `cvat/labels_generated.json` (Labels → Raw). Kiểm mỗi lớp có `occluded_level`, `truncated`, `can_review`.
2. Tạo **Task riêng** tên `<mã>`, tải ảnh trong `work/<mã>/images/`. Số frame = số dòng `submission/qc/assignments/<mã>.txt`.
3. Gán theo v1.0. Dùng thuộc tính `occluded_level` của nhóm, **không** dùng cờ occluded (phím `Q`) của CVAT.
4. Chưa chắc → tick `can_review`. Không vẽ được → ghi `ảnh, vùng, lý do` ra giấy.
5. Export: Actions → Export task dataset → **COCO 1.0**, tắt Save images → lấy `annotations/instances_default.json` → lưu `submission/labels/<mã>.json`.
6. ```bash
   python scripts/validate_labels.py submission/labels/<mã>.json
   ```
7. Gửi riêng. **Không xem nhãn của nhau** trước khi cả 3 nộp.

Sau khi cả 3 nộp: gom ghi chú vào `submission/qc/image_issues.csv` (cột `image,annotator,region,note`).

## Pha 4 — IAA (20')

```bash
python scripts/validate_labels.py --all
python scripts/qc_compare.py
python scripts/viz_disagreement.py --images flagged
```
Đọc `submission/qc/iaa_summary.md`, ghi lại:
- Độ phủ `…/80` lượt.
- IoU TB và % cùng lớp **kèm mẫu số** (số cặp box ghép). Ngưỡng: IoU ≥ 0.85, cùng lớp ≥ 95%.
- Box chưa ghép, bất đồng `ignore_region`.

Mở `work/viz/`, mỗi ca bất thường phân loại:

| Dấu hiệu | Nguyên nhân | Xử lý |
|---|---|---|
| Cả hai lệch cùng kiểu | guideline mơ hồ | sửa guideline (v1.1) |
| Một người lệch | người gán chưa nắm | hướng dẫn lại, sửa nhãn |
| Hai người giống nhau nhưng **cùng sai** | cùng hiểu sai | đối chiếu ca tham chiếu của coach |

Tìm ít nhất **một ca cùng sai** — IAA không phát hiện được, rubric có điểm riêng.

## Pha 5 — Calibration (40')

1. ```bash
   python scripts/build_final.py
   ```
   → mẫu `submission/qc/calibration_decisions.csv`. Điền **mỗi ca một dòng**: `quyet_dinh`, `quy_tac_ghi_vao_guideline` (R-xx), `phien_ban_guideline` (v1.1), `nguoi_chot`, `phieu_sua`, `kiem_lai`, `pham_vi` (`anh`/`lo`).
2. Viết `submission/guidelines/guideline_v1.1_calibrated.md` (thêm bảng lịch sử thay đổi) và `submission/guideline_change.json`:
   - chỉ sửa câu chữ → `{"kind": "editorial", "summary": "..."}`
   - đổi cách chọn vật / lớp / box / thuộc tính → `"semantic"`, cần ≥ 1 dòng `pham_vi=lo` và **rà lại cả lô**.
   - Đổi taxonomy → `submission/taxonomy_v1.1.json` + `python scripts/make_cvat_labels.py --final`.
3. ```bash
   python scripts/prepare_calibration.py
   ```
   Tạo Task từ `work/calib/images/`, sửa đúng ảnh đó, export COCO lưu **đúng đường dẫn script in ra**.
4. ```bash
   python scripts/build_final.py
   ```
   → `labels_calibrated.json/.csv`, `batch_manifest.json`.

## Pha 6 — QC & bàn giao (30')

```bash
python scripts/qc_sample.py --round 1
# reviewer mở work/review/round-01/, điền submission/qc/reviews/round-01/qc_review.csv
python scripts/qc_report.py --round 1
```
Cách điền phiếu (cột `image, object_id, error_type, reviewer, rule_id, region, note`):
- Ảnh sạch: một dòng `error_type=khong`.
- Lỗi box: `thua` / `sai_lop` / `sai_vi_tri_kich_thuoc` / `sai_thuoc_tinh` + `object_id` + `rule_id`.
- Bỏ sót: `bo_sot`, **để trống** `object_id`, ghi `region`.
- `reviewer` = mã 3 số trên **mọi** dòng.

Quyết định theo thứ tự **Hold → Rework → Release**:
- Hold: thiếu bằng chứng / guideline chưa chốt.
- Rework: > 2% ảnh lỗi (40 ảnh → **1 ảnh lỗi đã là 2,5%**) hoặc lỗi lặp → sửa (Pha 5) → `--round 2`, giữ round-01.
- Release: số liệu đạt, reviewer xác nhận.

Hoàn tất:
1. Copy `docs/QC_REPORT_TEMPLATE.md` → `submission/qc/QC_REPORT.md`, điền đủ (mọi tỉ lệ dạng `tử/mẫu`, cả theo **ảnh** và theo **đối tượng**).
2. Mỗi người: `submission/individual_notes/<mã>.md` từ `docs/INDIVIDUAL_NOTE_TEMPLATE.md`.
3. Điền phần kết quả trong `TEAM.md`.
4. ```bash
   python scripts/check_submission.py --round <vòng-cuối>
   ```
5. Nhờ người ngoài nhóm lần một ca: báo cáo → ảnh → rule → nhãn.
6. Nộp repo riêng tư `K5-L23-DAY07-Nhommoi-GuidelineQC`, **không** nộp `work/`.

---

## Checklist nộp

- [ ] 3 guideline cá nhân, 3 file COCO, 3 ghi chú cá nhân (tên file = mã)
- [ ] `validate_labels.py --all` sạch, đủ 80/80 lượt
- [ ] `taxonomy.json` version v1.0, `rules` nguyên vẹn
- [ ] Mỗi ca IAA có một dòng `calibration_decisions.csv`
- [ ] `guideline_v1.1_calibrated.md` + `guideline_change.json`
- [ ] Nhãn cuối + `batch_manifest.json`
- [ ] `QC_REPORT.md` đủ trường, có mẫu số, có quyết định
- [ ] `check_submission.py` không lỗi
- [ ] Không họ tên / MSHV / email / mật khẩu trong repo

## Bẫy hay mất điểm

1. Sửa mục `rules` trong taxonomy.
2. Xem nhãn của nhau trước khi nộp riêng.
3. Tạo Project CVAT bằng nhãn v0 → `validate_labels` báo lớp lạ.
4. Task tải thừa/thiếu ảnh so với `assignments/<mã>.txt`.
5. Tỉ lệ không kèm mẫu số.
6. Bịa dòng QC / số liệu — Hold có lý do vẫn điểm tốt.
