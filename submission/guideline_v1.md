# Guideline gán nhãn — v1.0 (nháp) — nhóm HAT

- Phiên bản: v1.0 (nháp)
- Ngày: 2026-11-16
- Người viết / duyệt: 2A202603076
- Loại nhãn: bounding box trên ảnh 2D · Công cụ: CVAT, xuất COCO 1.0
- Taxonomy đi kèm: `submission/taxonomy.json` (version `v1.0`)

## 1. Mục tiêu

Phát hiện **người và phương tiện thật** trên ảnh đường phố (camera giao thông, hỗ trợ lái xe).
Lỗi nguy hiểm nhất: bỏ sót người thật, gán vật không thật (tượng, poster) thành người hoặc xe, gộp người và xe vào một box.

**Nguyên tắc:** chỉ gán vật **thật, nhìn thấy, cao ≥ 20 px**, mỗi vật đúng một lớp. Không chắc thì tick `can_review`.

## 2. Lớp và thuộc tính

| Lớp | Định nghĩa | Dễ nhầm |
|---|---|---|
| `car` | Xe 4 bánh chở người cỡ nhỏ: sedan, SUV, wagon, MPV, taxi | có thùng hở tách cabin → `truck` |
| `truck` | Xe có thùng hàng tách cabin: bán tải (kể cả có nắp thùng), xe tải, đầu kéo + rơ-moóc | SUV cốp kín → `car` |
| `van` | Thân hộp liền khối, mũi ngắn, nhỏ hơn xe buýt (img_005, img_016) | ≥ 16 chỗ → `bus` |
| `bus` | Xe khách từ ~16 chỗ, kể cả minibus (img_009) | |
| `motorcycle` | Xe 2 bánh có động cơ, kể cả có sidecar | 3 bánh có mui → `three_wheeler` |
| `bicycle` | Xe 2 bánh đạp chân, kể cả chở hàng | xích lô → `three_wheeler` |
| `three_wheeler` | Tuk-tuk, xe lôi, auto-rickshaw, xích lô | |
| `person` | Người thật, kể cả người lái và người ngồi sau | tượng, ma-nơ-canh, hình in → không gán |
| `ignore_region` | Vùng cụm vật thật không tách được (R-05) | không dùng cho vật không thật |

Vật ngoài các lớp trên (động vật, nón giao thông, xe đẩy…): **không gán**.

**Thuộc tính**
- `occluded_level`: 
  - 0: Không bị che (0%)
  - 1: Che nhẹ (1–30%)
  - 2: Che vừa (31–70%)
  - 3: Che nặng (>70% diện tích vật thể)
- `truncated` (chọn `no` / `yes`, mặc định `no`): `yes` nếu vật bị cắt bởi mép ảnh.
- `can_review` (checkbox, mặc định không tick): tick nếu cần reviewer xem lại box này do không chắc lớp, không chắc có nên gán, hoặc không chắc giới hạn box.

## 3. Quy tắc

| Mã | Quy tắc |
|---|---|
| **R-01** | Box ôm sát **phần nhìn thấy**, không đoán phần bị che. Vẫn gán khi bị che hoặc cắt mép, nếu nhận ra lớp và phần thấy cao ≥ 20 px. |
| **R-02** | **Người trên xe:** người và xe là 2 box riêng (được chồng nhau). Mỗi người ngồi sau là một box `person`. Box xe không gồm người. |
| **R-03** | **Hàng chở** thò ra ngoài hoặc chất cao trên xe không vào box xe (img_029, img_035). Sidecar, giỏ, thùng gắn liền thì thuộc xe. Đầu kéo + rơ-moóc là 1 box. |
| **R-04** | **Xe chở xe:** xe thật nằm trên xe khác vẫn gán theo lớp của nó (img_002, img_024). |
| **R-05** | **Đám đông:** cụm ≥ 5 vật cùng lớp không tách được box (hoặc đa số < 20 px) → 1 `ignore_region` ôm sát cụm. Không gán vật riêng bên trong. Vật ở rìa tách được thì gán riêng. |
| **R-06** | **Vật không thật → không gán, không ignore:** tượng, mô hình, đồ chơi, ma-nơ-canh, poster, tranh tường, phản chiếu (gương, kính, đường ướt). |
| **R-07** | **Xe chủ** (gương, capô, tay lái của người chụp) không gán (img_020, img_040). |
| **R-08** | **Người trong xe** thấy rõ đầu và vai qua kính hoặc cửa → `person`. |
| **R-09** | **Xe lạ nhưng thật** (xe diễu hành, xe độ, xe cổ) gán theo công năng. Nhân vật trang trí trên xe không gán. |
| **R-10** | **Mờ chuyển động:** gán nếu nhận ra lớp, box ôm thân vật, không ôm vệt mờ. |
| **R-11** | **Ảnh không có vật** vẫn có trong COCO với 0 annotation. |
| **R-12** | **Bị che:** `occluded_level` = % diện tích cả vật bị **vật khác** che (phần ngoài mép ảnh chỉ tính `truncated`). Nhiều mảnh nhìn thấy vẫn là 1 box. Hàng rào, kính tăng tối đa 1 mức. Phân vân thì chọn mức cao hơn. |

## 4. Ca khó

Chỉ ghi các ca dễ làm hai người gán khác nhau.

| Ảnh | Tình huống | Quyết định | Rule |
|---|---|---|---|
| img_002 | Xe chở ô tô 2 tầng | 1 `truck` + mỗi ô tô trên xe là 1 `car` | R-03, R-04 |
| img_017, 027, 038 | Tượng, đài tưởng niệm hình xe, tủ đồ chơi | Không gán (0 annotation) | R-06, R-11 |
| img_018, 026, 008 | Biển quảng cáo, tranh tường, poster hình người | Không gán hình. Xe thật sau hàng rào (018) vẫn gán | R-06 |
| img_020 | Xe phản chiếu trong gương xe chủ | Không gán | R-06, R-07 |
| img_021, 030, 032 | Dòng xe máy, bãi xe, đám đông chờ xe | Vật tách được gán riêng, phần còn lại → `ignore_region` | R-05 |
| img_023 | Xích lô có người đạp | `three_wheeler` + `person` | R-02 |
| img_028 | Tượng ngựa trên bán tải thật | `truck`, không gồm tượng | R-03, R-06 |
| img_014 | Xe máy mờ chuyển động | Gán nếu nhận ra, tick `can_review` | R-10 |
| img_011 | Nhân vật Predator: tượng hay người hóa trang? | **Cần reviewer** — tạm gán `person` + `can_review` | R-06 |
| img_019 | Mascot chuột trên xe diễu hành | **Cần reviewer** — tạm gán `person` + `can_review`. Xe → `car` | R-09 |
| img_033 | Ma-nơ-canh; cửa xe bên phải là xe thật hay phản chiếu? | Ma-nơ-canh không gán. Cửa xe: **cần reviewer** | R-06 |

## 5. QC và phiên bản

- Ngưỡng QC: theo mục `rules` của `taxonomy.json`, không đổi.
- Ca `can_review`: người điều phối gom lại, **người review bản cuối** chốt và ghi mã rule.
- Đổi cách chọn vật, lớp, box hoặc thuộc tính là thay đổi `semantic`: tăng phiên bản, kiểm lại cả lô. Chỉ sửa câu chữ là `editorial`.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0 | Bản của giảng viên | — |
| v1.0 | Thêm `bus`, `bicycle`, `three_wheeler`. Ngưỡng `occluded_level`. R-01 đến R-12. | Xe buýt (009, 037), xe đạp (003, 029), tuk-tuk và xích lô (004, 023, 035), vật không thật, đám đông |
