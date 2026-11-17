# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 336 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 336 | 0.800 | 94.9% | 61.0% | 90.5% | 345/85 | 0 | CHƯA ĐẠT |
| 101|102 | 20 | 126 | 0.844 | 90.5% | 79.4% | 92.1% | 174/13 | 0 | CHƯA ĐẠT |
| 101|103 | 20 | 210 | 0.773 | 97.6% | 50.0% | 89.5% | 171/72 | 0 | CHƯA ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 766 ({'iou_thap': 166, 'sai_thuoc_tinh': 146, 'bo_sot_thua': 372, 'sai_lop': 17, 'bat_dong_ignore': 65}), trên 40 ảnh cần calibration.
