# Thông tin nhóm

- Tên nhóm: HAT
- Mã thành viên (ba chữ số): 
  - Nguyễn Minh Nam Hải (2A202603076/101)
  - Kiều Quang Trường (2A202603205/102)
  - Vũ Việt Anh (2A202603214/103)
- Người điều phối: Nguyễn Minh Nam Hải (2A202603076)
- Người review bản cuối: Kiều Quang Trường (2A202603205)
- Commit chốt bài (điền ở kênh nộp sau khi commit):

## Kết quả tóm tắt (điền cuối bài)

- Lượt gán hoàn tất / được giao: 80/80
- IAA trước calibration (kèm mẫu số): IoU TB 0.800, cùng lớp 94.9% trên 336 cặp box ghép (chưa đạt ≥ 0.85 / ≥ 95%).
- Số ảnh phải calibration: 40 / 40
- Vòng QC và số ảnh đã duyệt / cả lô: vòng 1, 40/40
- Tỉ lệ lỗi: 65.0 % theo ảnh (26/40), 6.5 % theo đối tượng (42/646)
- **Quyết định** (Release / Hold / Rework) và lý do một dòng: **Rework** — 65% ảnh có lỗi (ngưỡng 2%), lỗi lặp thuộc tính R-12.

## Ghi chú sai lệch quy trình chia ảnh

- Nhóm không dùng phân công của `split_assign.py` (seed 42). Thực tế: 101 gán cả 40 ảnh, 102 gán img_001–020, 103 gán img_021–040; mỗi ảnh vẫn đúng 2 người gán độc lập (80/80 lượt).
- `submission/qc/assignment.csv` và `assignments/*.txt` được ghi lại theo phân công thực tế để `validate_labels.py` và `qc_compare.py` chạy được. Bản seed 42 lưu ở `work/backup/qc_seed42/`.
- `103.json`: bỏ 2 box nằm ngoài phần được giao (img_002, img_020). Bản gốc ở `work/backup/103_orig.json`.
- Hệ quả: 102 và 103 không có ảnh chung, nên IAA chỉ đo cặp 101–102 và 101–103.

## Sửa bug script

- `scripts/common.py` `xyxy_to_xywh`: làm tròn `x` và `w` riêng nên `x + w` có thể vượt mép ảnh (img_040 #764 trong `consensus_draft.json`: y 169.78 + h 470.23 = 640.01 > 640), làm `build_final.py` báo "bbox không hợp lệ hoặc vượt ảnh". Sửa: làm tròn từng cạnh trước rồi mới tính `w`, `h`.
- Bằng chứng chạy lại: `python -m unittest discover -s tests` 20/20 OK; chạy lại `qc_compare.py`, `calibration_candidates.csv` giống hệt bản trước (766 ca, cùng ID); tạo lại `calibration_decisions.csv` và chép lại quyết định đã điền theo `case_id`; `build_final.py` qua bước kiểm quyết định và guideline.
