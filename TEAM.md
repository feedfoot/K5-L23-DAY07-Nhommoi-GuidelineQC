# Thông tin nhóm

- Tên nhóm: Nhommoi
- Mã thành viên (ba chữ số): 055, 043, 056
- Người điều phối: Cao Văn Dương
- Người review bản cuối: Nguyễn Viết Minh
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80
- IAA trước calibration (kèm mẫu số): IoU TB 0.946, cùng lớp 98.2% trên 383 cặp box ghép (đạt ngưỡng 0.85 / 95%); cặp có 056 chỉ 52–57% cùng occluded_level
- Số ảnh phải calibration: 29 / 40 (sau QC vòng 1 thêm img_001 → phạm vi sửa 30 / 40)
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1 40/40 (REWORK) → vòng 2 40/40 (RELEASE)
- Tỉ lệ lỗi: 0.0 % theo ảnh (0/40), 0.0 % theo đối tượng (0/478) ở vòng 2; vòng 1 là 60.0 % (24/40) và 11.9 % (60/506)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: Release — vòng 2 duyệt toàn lô, 0/40 ảnh lỗi (≤ 2%), không lỗi lặp, bằng chứng khớp checksum
