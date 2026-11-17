# Guideline gán nhãn — v1.0 — 102

- Phiên bản: v1.0
- Ngày: 16/11/2026
- Người viết: 102

## 1. Mục tiêu
Xây dựng tập dữ liệu bounding box 2D cho các đối tượng giao thông đường phố. Nhãn này phục vụ mô hình phát hiện vật thể (Object Detection) thời gian thực.
- **Quan trọng:** Mô hình cần học ranh giới vật thể thực trong không gian 3D. Sai sót trong việc gán nhãn các đối tượng gây nhiễu (đồ chơi, phản chiếu) hoặc gán sai ranh giới vật thể sẽ trực tiếp làm giảm độ an toàn của mô hình.

### 2Thuộc tính
- **occluded_level:** 
  - 0: Không bị che (0%)
  - 1: Che nhẹ (1–30%)
  - 2: Che vừa (31–70%)
  - 3: Che nặng (>70% diện tích vật thể)
- **truncated:** [0, 1] - 1 nếu vật bị cắt bởi mép ảnh.
- **can_review:** [0, 1] - 1 nếu cần reviewer xem lại box này do không chắc chắn.

## 3. Quy tắc vẽ box
- **Nguyên tắc chung:** Box phải ôm sát phần nhìn thấy của vật thể.
- **Xe chở xe:** Gán 1 box cho xe tải và gán box cho từng xe con trên thùng nếu nhìn rõ hình dạng.
- **Vật bị che:** Gán box bao quanh phần nhìn thấy; nếu che >70%, gán mức occluded_level=3.
- **Vật nhỏ:** Không gán vật thể/vùng nhìn thấy có kích thước nhỏ hơn 20x20 pixel.
- **Người đi xe máy:** Gán cả người lái và xe vào chung 1 box.
- **Đồ chơi/Tượng/Phản chiếu:** Không gán nhãn cho xe đồ chơi, tượng trong cửa kính, hình ảnh phản chiếu trong gương hoặc bề mặt kim loại bóng.
- **Nhòe chuyển động (Motion blur):** Chỉ gán đối tượng nhìn rõ hình dáng cốt lõi. Bỏ qua các vệt nhòe nặng không rõ cấu trúc.
- **Mật độ cao (Crowded/Parking lot):** Chỉ gán những đối tượng ở hàng đầu tiên (tiền cảnh) hoặc những đối tượng nhìn thấy rõ >50% diện tích. Đối với các xe xếp hàng dài ở xa hoặc bị che khuất quá 70% bởi xe khác, bỏ qua để tránh nhiễu dữ liệu.
- **Độ tương phản thấp (Low contrast):** Chỉ gán nhãn cho những đối tượng có thể nhận diện được hình dáng cốt lõi. Nếu ảnh quá mờ, nhiễu hoặc tương phản quá kém đến mức không thể phân biệt ranh giới đối tượng với môi trường, hãy bỏ qua.

## 4. QC và phiên bản
- **Ngưỡng QC:** Sai lệch IoU > 0.3 giữa hai người gán là ca cần recalibration.
- **Lịch sử thay đổi:**
  - v1.0: Thiết lập quy tắc gán nhãn ban đầu dựa trên khảo sát 40 ảnh.
