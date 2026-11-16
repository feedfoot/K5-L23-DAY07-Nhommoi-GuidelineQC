# Báo cáo QC — Nhommoi

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D (9 lớp người và phương tiện) cho lô thực hành 40 ảnh đường phố của lab Day07; chỉ dùng cho học tập và thử nghiệm huấn luyện. Lô nhỏ và cố ý chứa nhiều ca khó (tuk-tuk, xích lô, tượng, phản chiếu, đám đông) nên không đại diện cho phân bố giao thông thực và không dùng làm benchmark đánh giá mô hình.
- Nguồn và phép sử dụng dữ liệu: 40 ảnh lấy từ Wikimedia Commons; tác giả, giấy phép và đường dẫn gốc của từng ảnh nằm trong `data/ATTRIBUTION.csv` (build_final.py đã kiểm đủ 40 ảnh). Khi dùng lại phải giữ ghi công theo giấy phép của từng ảnh.
- Manifest và checksum nhãn cuối: `submission/qc/batch_manifest.json`; `labels_calibrated.json` SHA-256 `6a794b96ab4bc748b1f4d84af968c4b8fa046ecca1725a4e18ae0d5f259e5059`; `labels_calibrated.csv` SHA-256 `0ada52e39c8ac914adc16e34b834f2ef4b4ecadf78bcebf10b990d2c0dea651b` (40 ảnh, 481 annotation = 478 đối tượng + 3 `ignore_region`).
- Guideline và taxonomy khi gán / khi bàn giao: Gán theo `guideline_v1.0_agreed.md` + `taxonomy.json` v1.0. Bàn giao theo `guideline_v1.1_calibrated.md` (thay đổi editorial: làm rõ ngưỡng occluded 0↔1, nhắc kiểm truncated khi chạm mép, bổ sung ca khó §4) + taxonomy v1.0 giữ nguyên (không đổi lớp, thuộc tính, rule nên không tạo `taxonomy_v1.1.json`).

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 lượt (40 ảnh × 2 người; 043: 26 ảnh, 055: 27 ảnh, 056: 27 ảnh; cặp 043|055: 13 ảnh, 043|056: 13 ảnh, 055|056: 14 ảnh)
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 383 cặp box ghép (IoU ≥ 0.5, ghép 1-1); IoU TB 0.946; cùng lớp 98.2%, cùng occluded_level 73.1%, cùng truncated 93.5% (mẫu số 383). Theo cặp: 043|055 158 cặp, IoU 0.998, lớp 99.4%; 043|056 154 cặp, IoU 0.912, lớp 96.8%, occluded 56.5%; 055|056 71 cặp, IoU 0.905, lớp 98.6%, occluded 52.1%. Đạt ngưỡng IoU ≥ 0.85 và lớp ≥ 95%, nhưng thuộc tính của hai cặp có 056 lệch mạnh.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: 60/74 box chưa ghép (A/B, cộng mọi cặp); 0 box bị loại do ignore; 4 ca bất đồng vùng ignore (C127, C128 ở img_019; C187, C188 ở img_031) vì chỉ 055 vẽ `ignore_region`.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_009 (043 và 056 cùng gán 4 hành khách trong xe buýt, trái R-05); img_005 (043 và 055 cùng gán xe van 51LD-70.65 là `car` và tách làm 2 box, trái §2.1 vốn lấy chính xe này làm ví dụ `van`); img_001 (043 và 055 cùng bỏ sót người áo hồng ngồi sau bàn, R-02). Cách kiểm: đối chiếu từng box với ví dụ §2.1/§4 và R-01…R-13 trên overlay của vòng QC, không coi việc hai người trùng nhau là bằng chứng đúng.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: `submission/qc/calibration_decisions.csv` có 310 dòng: 286 ca IAA C001–C286 (đủ quyết định, rule, phiếu sửa, người chốt 043/055/nhóm, kiểm lại OK, phạm vi `anh`) và 24 ca QC Q01–Q24 (người chốt 043, phiếu sửa ghi ID box của vòng 1, kiểm lại ở round-02).
- Tác động v1.1 lên nhãn và ảnh: v1.1 chỉ editorial và mọi quyết định có phạm vi `anh` → phạm vi sửa 30 ảnh (29 ảnh bị cờ IAA + img_001 do Q01); 10 ảnh còn lại giữ nhãn đồng thuận; không cần mapping lớp vì taxonomy không đổi.
- Tham chiếu bằng chứng khi Rework: Vòng 1 (`reviews/round-01`, nhãn SHA-256 `a94480a2…`): REWORK, 24/40 ảnh lỗi → thêm Q01–Q24 → chạy lại `prepare_calibration.py` → `labels_calibrated_779289695fefa165.json` (30 ảnh) → `build_final.py` → vòng 2 (`reviews/round-02`, nhãn SHA-256 `6a794b96…`): RELEASE. Thư mục round-01 giữ nguyên để truy vết.

## 4. Kết quả duyệt

- Vòng QC: 2
- Người review và ngày: 043, 2026-11-17 (khác người nhận phiếu sửa chính là 056; duyệt có hỗ trợ của Claude Code — đối chiếu tự động với nhãn gốc và đọc overlay/crop phóng to — 043 cần mở lại `work/review/round-02` để xác nhận)
- Đã duyệt / cả lô: 40/40 ảnh (mode full; ca_kho 29/29, con_lai 11/11)
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0% (vòng 1: 24/40 = 60.0%)
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/478 = 0.0% (vòng 1: 60/506 = 11.9%)
- Vật bỏ sót, ảnh và vùng: Vòng 2: 0. Vòng 1: 9 — img_001 (người áo hồng x113-200 y106-250), img_016 (tuk-tuk xanh dương x0-258), img_019 (đám đông y0-205 + 4 người ở rìa/tiền cảnh), img_031 (vùng ignore x362-485 y355-376), img_032 (vùng ignore x0-695 y250-368); đã bổ sung qua Q01, Q11, Q12, Q18, Q19.
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Vòng 1: ca_kho 23/29 ảnh lỗi, con_lai 1/11; theo loại: thua 34, sai_lop 12, bo_sot 9, sai_thuoc_tinh 9, sai_vi_tri_kich_thuoc 6 (không loại nào chiếm ≥ 50% tổng). Box trùng chủ yếu do bước gộp đồng thuận và áp quyết định bằng script ghép nhầm box (ví dụ img_034 #434/#437 trùng ô tô #427, img_026 #313 trùng xe máy #312); lỗi lớp do quyết định chưa được áp (img_004 tuk-tuk C043/C045, img_019 xe diễu hành C129). Vòng 2: 0/29 và 0/11.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy: box trùng IoU > 0.7 và box chạm mép ±2 px mà truncated = no (script đối chiếu bắt img_030 #332, #336; build_final/qc_report chặn bbox vượt biên và `can_review`). Người: hành khách trong xe buýt img_009 (R-05) và tựa đầu ghế trong xe van img_005 bị gán `person` — chỉ đọc ảnh mới nhận ra.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: Vòng 2 duyệt toàn bộ 40/40 ảnh, 0% ảnh lỗi (≤ 2%), không lỗi lặp, không còn `can_review`, bằng chứng khớp checksum (`qc_report.py --round 2` ra RELEASE). Người nhận: coach và lớp K5-L23 Day07. Phạm vi: lô nhãn thực hành 2D cho học tập và thử nghiệm, kèm giới hạn ở mục 1.
- Nếu còn việc: Không còn việc chặn bàn giao. Khuyến nghị: (1) bản guideline sau sửa danh sách ví dụ `ignore_region` ở §2.1/§4 vì img_025, img_040 không có cụm dày đạt R-10 (nhóm xử lý, bằng chứng là §4 mới); (2) các ảnh rất dày img_003, img_006, img_008, img_021 mới duyệt ở mức overlay, nên kiểm lại vật nhỏ ở xa nếu dùng cho huấn luyện thật; (3) nhờ người ngoài nhóm thử kiểm ngược một ca trước khi nộp.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca Q12 → `data/images/img_019.jpg` (overlay `work/review/round-02/img_019.jpg.png`) → R-05, R-10, §2.1 → `labels_calibrated.json`: #204 `car` [117,202,878,636] (không còn `truck`), #206 `ignore_region` [0,0,960,205], #207–#210 `person`, không còn box tài xế; khớp phiếu sửa Q12 và phiếu duyệt vòng 2.
