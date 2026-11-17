# Guideline gán nhãn — <phiên bản> — <mã hoặc nhóm>

> Copy file này (hoặc `guideline_v0_starter.md`) rồi viết. Một guideline đủ gồm **5 phần**. Câu nào của v0
> làm hai người đọc ra hai cách khác nhau thì phải viết lại cho một cách hiểu duy nhất.

- Phiên bản: …
- Ngày: …
- Người viết / duyệt: 103

# 1. Mục tiêu
- **Mô hình sử dụng:** Mô hình phát hiện đối tượng 2D (2D Object Detection như YOLO, Faster R-CNN, RT-DETR) huấn luyện trên dữ liệu giao thông đô thị hỗn hợp.
- **Mục đích:** Nhận diện chính xác vị trí và phân loại người đi bộ cùng các phương tiện cơ giới trên đường phố để phục vụ hệ thống hỗ trợ lái xe nâng cao (ADAS), xe tự hành hoặc giám sát lưu lượng giao thông.
- **Sai sót gây nguy hiểm:**
  - **Bỏ sót (False Negative):** Bỏ sót người đi bộ, người đi xe máy hoặc xe đang băng qua đường khiến hệ thống không kịp phanh/né tránh, dẫn đến tai nạn va chạm trực tiếp.
  - **Nhận diện nhầm vật giả (False Positive):** Gán nhãn nhầm cho tượng, ma-nơ-canh, poster quảng cáo hoặc ảnh phản chiếu qua gương làm hệ thống phanh gấp vô cớ (phantom braking).
  - **Nhầm lẫn kích thước phương tiện:** Phân loại nhầm `bus` / `truck` cỡ lớn thành `car` nhỏ làm mô hình dự đoán sai quỹ đạo rẽ và khoảng cách an toàn.

---

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `person` | Người thật đang đi bộ, đứng, ngồi hoặc điều khiển/ngồi trên phương tiện giao thông. | Người đi bộ (`img_006.jpg`), người lái xe máy (`img_005.jpg`), người đẩy xe đạp/gánh hàng (`img_003.jpg`). | Tượng đồng/đá (`img_011.jpg`, `img_017.jpg`), ma-nơ-canh (`img_033.jpg`), người in trên biển quảng cáo (`img_018.jpg`). |
| `car` | Xe ô tô con chở khách dưới 9 chỗ (sedan, hatchback, SUV, crossover, taxi). | Xe taxi trắng (`img_006.jpg`), 8 ô tô con nằm trên sàn xe tải chuyên dụng (`img_002.jpg`). | Xe mô hình đồ chơi (`img_038.jpg`), tượng xe bán tải (`img_027.jpg`), xe minivan cỡ lớn (`van`). |
| `truck` | Xe tải chở hàng có thùng tách biệt hoặc liền khối cỡ lớn, xe đầu kéo và xe chuyên dụng chở ô tô. | Xe tải thùng `ISUZU` (`img_007.jpg`), xe tải 2 sàn chở ô tô (`img_002.jpg`), xe bán tải thùng lửng (`img_001.jpg`). | Xe chở khách (`bus`), xe minivan chở khách 12–16 chỗ (`van`). |
| `van` | Xe tải nhẹ hoặc xe chở khách cỡ nhỏ (12–16 chỗ) có thiết kế khoang liền khối. | Xe minivan bạc `51LD-70-65` (`img_005.jpg`), xe tải van giao hàng cỡ nhỏ. | Xe buýt thân dài trên 24 chỗ (`bus`), xe bán tải có thùng rời phía sau (`truck`). |
| `bus` | Xe buýt công cộng hoặc xe khách đường dài thân dài cỡ lớn (từ 24 chỗ trở lên). | Xe khách thân dài màu kem/nâu qua ngã tư (`img_009.jpg`), xe buýt đô thị (`img_031.jpg`, `img_040.jpg`). | Xe minivan 16 chỗ (`van`), xe tải thùng kín (`truck`). |
| `motorcycle` | Phương tiện cơ giới hai bánh có gắn động cơ (xe máy, xe tay ga, mô tô). | Xe tay ga, xe số lưu thông trên đường (`img_006.jpg`, `img_010.jpg`). | Xe đạp thường không động cơ (`img_003.jpg`, `img_006.jpg`), xe 3 bánh (`tuk_tuk`). |
| `ignore_region` | Vùng chứa đám đông người hoặc cụm phương tiện san sát nhau không thể tách lẻ từng box, hoặc cụm vật thể quá nhỏ (`< 20 px`). | Hàng xe máy đỗ chồng lấn dày đặc ở trung/hậu cảnh (`img_008.jpg`), đám đông xa (`img_006.jpg`). | Các đối tượng đứng tách biệt và có chiều cao `>= 20 px` (bắt buộc vẽ box riêng). |

- **Xử lý vật không thuộc lớp nào:**
  - **Không gán (bỏ qua hoàn toàn):** Đối với xe đạp thường, xe xích lô không động cơ (`img_003.jpg`, `img_006.jpg`, `img_029.jpg`), động vật (`img_009.jpg`), đồ vật/quang gánh rời, tượng, tranh quảng cáo, ma-nơ-canh và hình phản chiếu.
  - **Dùng vùng `ignore_region`:** Khi có đám đông người hoặc bãi xe dày đặc chồng chéo ở xa không thể phân tách từng khung box riêng lẻ.
  - **Đề xuất lớp mới:** Bổ sung `bus` và `tuk_tuk` vào `taxonomy.json` (v1.0) vì đây là hai nhóm phương tiện xuất hiện lặp lại nhiều lần nhưng bản `v0` chưa bao phủ.

---

### Thuộc tính
1. **`occluded_level`** *(Mức độ bị vật thể khác che khuất)*:
   - **`0`**: Hiển thị rõ ràng **100%** diện tích thực tế (không bị che khuất).
   - **`1`**: Bị che khuất nhẹ từ **1% đến 30%** diện tích (ví dụ: người ngồi sau quầy bàn trong `img_001.jpg`).
   - **`2`**: Bị che khuất trung bình từ **31% đến 70%** diện tích (ví dụ: tài xế ngồi bên trong cabin xe tuk-tuk ở `img_004.jpg`).
   - **`3`**: Bị che khuất nặng **trên 70%** diện tích nhưng vẫn còn đủ bộ phận đặc trưng để nhận diện chắc chắn đúng lớp.
2. **`truncated`** *(Bị cắt bởi viền khung hình)*:
   - **`true`**: Khung box chạm sát ít nhất một cạnh biên của bức ảnh (vật thể bị cắt mất một phần do góc chụp, ví dụ: người đi xe máy sát mép dưới trong `img_005.jpg`).
   - **`false`**: Toàn bộ vật thể nằm trọn vẹn bên trong khung ảnh.
3. **`can_review`** *(Cờ yêu cầu reviewer kiểm tra lại)*:
   - **`true`**: Đánh dấu vào các đối tượng thuộc ca biên gây tranh cãi hoặc chưa chắc chắn ranh giới lớp (ví dụ: xe bán tải độ thùng kín cao ở `img_007.jpg`).
   - **`false`**: Các đối tượng rõ ràng, tuân thủ hoàn toàn quy tắc chuẩn.

---

## 3. Quy tắc vẽ box
- **R-01 (Ôm sát phần nhìn thấy):** Vẽ khung chữ nhật bám khít đường viền ngoài cùng của **phần diện tích thực tế nhìn thấy được**, bao gồm cả gương chiếu hậu, bánh xe và hàng hóa chất cố định trên xe. Tuyệt đối không vẽ đoán mò xuyên qua vật cản.
- **R-02 (Vật bị che khuất):** Chỉ bao khung phần nhìn thấy và chọn mức `occluded_level` (1, 2 hoặc 3) tương ứng với tỷ lệ diện tích bị che. Nếu vật bị che gần như hoàn toàn (> 90%) không thể nhận diện chắc chắn thì bỏ qua.
- **R-03 (Vật bị cắt mép ảnh):** Vẽ box chạm đúng đường biên bức ảnh và bật thuộc tính `truncated = true`.
- **R-04 (Loại bỏ vật giả & phản chiếu):** Không gán nhãn cho tượng đồng/đá (`img_011.jpg`, `img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), mô hình đồ chơi (`img_038.jpg`), chân dung trên poster quảng cáo (`img_018.jpg`) và hình ảnh phản chiếu trong gương chiếu hậu (`img_020.jpg`).
- **R-05 (Xe chủ - Ego vehicle):** Không gán nhãn cho phần nắp capo, khung kính hoặc nội thất của chính chiếc xe gắn camera đang chụp ảnh.
- **R-06 (Vật quá nhỏ & đám đông):** Bỏ qua các đối tượng đơn lẻ có chiều cao **nhỏ hơn 20 pixel (`< 20 px`)**. Nếu nhiều đối tượng nhỏ hoặc xe đỗ san sát nhau tạo thành cụm không thể tách rời (`img_008.jpg`), vẽ một khung `ignore_region` bao trọn vùng đó.
- **R-07 (Một vật có nhiều phần - Người & Xe):** Với người đi xe máy (`img_006.jpg`), tách riêng **1 box `motorcycle`** cho chiếc xe và **1 box `person`** riêng cho từng người ngồi trên xe. Với xe đạp không động cơ (`img_003.jpg`), chỉ vẽ box `person` cho người điều khiển và bỏ qua khung xe đạp.
- **R-08 (Xe chở xe):** Với xe tải chuyên dụng chở ô tô (`img_002.jpg`), vẽ **1 box `truck`** lớn cho toàn bộ xe đầu kéo chuyên dụng và vẽ thêm **các box `car` riêng lẻ** cho từng chiếc ô tô con nằm trên sàn chở (nếu cao `>= 20 px`).

---

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002` | Xe tải chuyên dụng 2 tầng chở 8 chiếc ô tô con phía sau. | `R-08` | Vẽ 1 box `truck` bao trọn xe chở + 8 box `car` riêng cho từng ô tô con trên sàn. | Các ô tô con hiển thị rõ hình dáng đặc trưng và có chiều cao `>= 20 px`, mô hình 2D cần nhận diện đầy đủ. |
| `img_003`, `img_006`, `img_010` | Người đẩy xe đạp, người đi xe đạp, người gánh hàng rong cồng kềnh. | `R-01`, `R-07` | Chỉ vẽ box `person` ôm sát cơ thể người; **bỏ qua** khung xe đạp và sạp hàng/quang gánh. | Bộ nhãn chỉ định nghĩa xe cơ giới có động cơ (`motorcycle`, `tuk_tuk`), không gán nhãn cho xe đạp thô sơ và hàng hóa rời. |
| `img_004`, `img_016`, `img_035` | Xe ba bánh tuk-tuk chở khách và hàng hóa chất cao. | `R-01`, `R-02` | Bổ sung lớp mới `tuk_tuk` (box ôm cả hàng hóa cố định trên xe); gán `person` cho tài xế với `occluded_level = 2`. | Xe 3 bánh không thuộc `car` hay `motorcycle`; hàng hóa gắn cố định tạo thành khối di chuyển chung của phương tiện. |
| `img_007`, `img_016`, `img_037` | Xe bán tải cải tạo gắn nắp thùng kín cao hoặc khung mui bạt phía sau. | `R-01` | Xếp vào lớp `truck` và bật cờ `can_review = true`. | Thiết kế thùng chở hàng tách biệt/nâng cao dễ gây tranh cãi ranh giới giữa `truck` và `van`, cần đánh dấu để nhóm chốt chung ở Pha 2. |
| `img_008`, `img_019` | Hàng xe máy đỗ san sát chồng lấn ban đêm và đám đông khán giả kín sau rào. | `R-06` | Tách box lẻ cho các đối tượng rõ ở tiền cảnh; gom cụm đối tượng chồng chéo/quá nhỏ (`< 20 px`) vào `ignore_region`. | Các đối tượng ở trung/hậu cảnh bị che khuất đan xen dày đặc, không thể phân tách ranh giới từng box đơn lẻ chính xác. |
| `img_009`, `img_031`, `img_040` | Xe khách/xe buýt thân dài cỡ lớn lưu thông trên đường phố. | `R-01` | Bổ sung lớp mới `bus` và gán nhãn cho các phương tiện này. | Kích thước thân dài (từ 24 chỗ trở lên) vượt tiêu chuẩn `van` 16 chỗ, cần tách lớp riêng để mô hình nhận diện đúng kích thước. |
| `img_011`, `img_013`, `img_017`, `img_027`, `img_033`, `img_038` | Tượng Predator, tượng đồng cưỡi ngựa, tượng gà/sư tử, tượng bán tải, ma-nơ-canh, đồ chơi. | `R-04` | **Không gán nhãn** (bỏ qua hoàn toàn; ảnh chỉ chứa vật giả như `img_017` giữ nguyên 0 annotation). | Đây là các vật thể tĩnh trang trí hoặc mô hình giả lập, không tham gia giao thông thật, gán nhãn sẽ gây nhiễu cho mô hình. |
| `img_014`, `img_015` | Xe máy và ô tô chạy tốc độ cao bị nhòe chuyển động (motion blur) tạo vệt bóng ma. | `R-01`, `R-02` | Chỉ vẽ box ôm sát **phần thân đặc nhìn thấy rõ**, tuyệt đối không kéo box theo vệt nhòe mờ. | Kéo box trùm cả vệt bóng ma sẽ làm sai lệch kích thước thực tế và tâm tọa độ của phương tiện. |
| `img_018`, `img_020` | Chân dung người trên biển quảng cáo (`img_018`) và ô tô phản chiếu trong gương chiếu hậu (`img_020`). | `R-04`, `R-05` | **Không gán nhãn** chân dung quảng cáo, ảnh phản chiếu qua gương và phần xe chủ (ego vehicle). | Tranh in và ảnh ảo qua gương gây sai lệch tọa độ không gian 2D thực tế; các ô tô đỗ sau hàng rào sắt ở `img_018` gán `car` với `occluded_level = 1`. |
| `img_019` | Xe diễu hành màu vàng chạy trên đường có tài xế thật và mô hình chuột khổng lồ ngồi cạnh. | `R-01`, `R-04` | Gán `car` (`can_review = true`) cho xe diễu hành và `person` cho tài xế; **bỏ qua** mô hình chuột. | Xe có biển số thật và đang lưu thông trên đường nên vẫn cần nhận diện như phương tiện, còn linh vật hoạt hình là vật giả. |


---

## 5. QC và phiên bản
- **Ngưỡng QC áp dụng:**
  - Tỷ lệ hoàn thành lượt gán chéo: **100%** số ảnh được phân công theo `assignment.csv`.
  - Độ đồng thuận tọa độ khung (IoU trung bình trên các cặp ghép 1–1): 
  - Độ đồng thuận phân loại lớp (Class Agreement): 
  - Kích thước đối tượng tối thiểu: Chiều cao box 
- **Người quyết định ca mơ hồ:**
  - Trong **Pha 3** (gán độc lập): Người gán tự đánh dấu `can_review = true` hoặc ghi vào `submission/qc/image_issues.csv`, tuyệt đối không tự ý đoán mò hay hỏi chéo đồng đội.
  - Trong **Pha 5** (hiệu chỉnh sau đo IAA): Cả nhóm họp dưới sự điều phối của **Leader / Reviewer** để chốt phương án cuối cùng, cập nhật quy tắc vào guideline và đối chiếu ca tham chiếu của Coach nếu vẫn còn tranh cãi.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản khởi tạo ban đầu với 5 lớp cơ bản (`car`, `truck`, `van`, `motorcycle`, `person`) và mô tả định tính. | Tài liệu khung ban đầu của bài lab (`guideline_v0_starter.md`). |

