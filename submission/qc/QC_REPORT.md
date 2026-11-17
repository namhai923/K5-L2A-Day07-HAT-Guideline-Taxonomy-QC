# Báo cáo QC — HAT

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: thực hành đặc tả và QC bbox 2D (người, phương tiện thật) trong Day 07; 40 ảnh ca biên, chưa đánh giá độ đại diện. **Không dùng để huấn luyện** (Rework, mục 5).
- Nguồn và phép sử dụng dữ liệu: theo `data/ATTRIBUTION.csv`.
- Manifest và checksum nhãn cuối: `batch_manifest.json` (SHA-256 `4af05c80a92d226aa3efd4b8a7e12e0c13b04028cfa7a3ade7a1558ebf2610d4`); `labels_calibrated.json` (SHA-256 `5c762123f9971b8db0b0b247594a73df99ac0aa170f0e188a6f5dd108cc19858`), 40 ảnh, 647 annotation.
- Guideline và taxonomy khi gán / khi bàn giao: gán theo v1.0; bàn giao theo `guideline_v1.1_calibrated.md` (`semantic`). Taxonomy giữ v1.0, `rules` không đổi.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80. **Lệch quy trình:** không dùng `split_assign.py`; 101 gán 40 ảnh, 102 gán img_001–020, 103 gán img_021–040 (xem `TEAM.md`).
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 336 cặp; IoU 0.800, cùng lớp 94.9% → chưa đạt (≥ 0.85, ≥ 95%). 101|102: 126 cặp, 0.844, 90.5%. 101|103: 210 cặp, 0.773, 97.6%.
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: chưa ghép 345 (101) / 85 (102, 103); 0 loại do ignore; 65 ca bất đồng ignore (img_038: 37, img_019: 12).
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: img_038 — 101 gán đồ chơi, 103 vẽ `ignore_region` cả ảnh; theo R-06 cả hai sai. Kiểm qua `work/viz/img_038.png`.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 766/766 ca trong `calibration_decisions.csv`, người chốt 102; R-01 (415), R-12 (147), R-05 (111), R-06 (50). `kiem_lai`: kiểm bằng script, **chưa có thành viên kiểm mắt trên CVAT**.
- Tác động v1.1 lên nhãn và ảnh: img_011 (tượng), img_033 (phản chiếu) → không gán; R-06 thêm tủ đồ chơi. Phạm vi `lo` (C188, C618, C619) → rà cả 40 ảnh. 766 → 647 annotation.
- Tham chiếu bằng chứng khi Rework: vòng 1 = `submission/qc/reviews/round-01/`; chưa làm vòng 2.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 102, 2026-11-17
- Đã duyệt / cả lô: 40/40
- Ảnh lỗi / ảnh đã duyệt: 26/40 = 65.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 42/646 = 6.5%
- Vật bỏ sót, ảnh và vùng: 3 — img_010 (xe đạp, x 380–620, y 230–515, R-01); img_014 (người lái mờ, x 0–210, y 180–690, R-10); img_021 (cụm xa, x 150–900, y 195–270, thiếu `ignore_region`, R-05).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: lặp nhiều nhất `sai_thuoc_tinh` R-12 (29), ví dụ img_001 #7 `occluded_level` để 0. Tiếp theo `thua` (12), ví dụ img_005 #77 box trùng. Phiếu: `reviews/round-01/qc_review.csv`.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: máy — box < 20 px (`validate_labels.py`), box vượt mép ảnh (`build_final.py`, img_040 #764). Người — img_014 người lái mờ chuyển động (R-10).

## 5. Quyết định bàn giao

- Quyết định: Rework
- Lý do, người nhận và phạm vi sử dụng: 65.0% ảnh lỗi (ngưỡng 2%), lỗi lặp R-12.
- Nếu còn việc: (1) 101 gán tay lại 40 ảnh, chạy lại từ `validate_labels.py --all` — 101; (2) đưa 46 lỗi vòng 1 vào nhật ký (`Q01`…), sửa trên CVAT, kiểm mắt, sửa `phieu_sua` của C494 (img_029 #431) và cập nhật nhãn cuối cho img_031 #486 — 102, 103; (3) vòng 2 (`qc_sample.py`, `qc_report.py --round 2`), giữ vòng 1 — 102.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: img_029 #431 — `qc_review.csv` dòng 46 (`thua`, R-03) → C494 quyết định bỏ box ôm hàng chở, giữ box [332,210,225,161] → `labels_calibrated.json` id 431 vẫn còn bbox [73.58, 117.9, 837.42, 257.0] (`work/review/round-01/img_029.jpg.png`). Kết quả: **lệch** quyết định C494, sửa ở vòng 2.
