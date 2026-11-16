# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 383 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 383 | 0.946 | 98.2% | 73.1% | 93.5% | 60/74 | 0 | ĐẠT |
| 043|055 | 13 | 158 | 0.998 | 99.4% | 98.7% | 100.0% | 3/2 | 0 | ĐẠT |
| 043|056 | 13 | 154 | 0.912 | 96.8% | 56.5% | 87.7% | 39/41 | 0 | ĐẠT |
| 055|056 | 14 | 71 | 0.905 | 98.6% | 52.1% | 91.5% | 18/31 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 286 ({'bo_sot_thua': 132, 'sai_thuoc_tinh': 118, 'iou_thap': 25, 'sai_lop': 7, 'bat_dong_ignore': 4}), trên 29 ảnh cần calibration.
