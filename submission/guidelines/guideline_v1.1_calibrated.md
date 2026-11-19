# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202604051 - Nguyễn Xuân Lam

> Chỉ ghi phần thay đổi so với v1.0. Mỗi ca khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Xe cao ≥ 20 px gán từng xe; cụm ≥ 5 vật cao < 20 px vẽ **một** `ignore_region`; không giữ box lẻ cao < 20 px. Ngưỡng 20 px áp dụng cả khi khối xe ở gần camera | img_018 | CAL-03 |

Các câu 1, 3, 4, 5, 6, 7 **chưa chốt**: lô ảnh này chưa đủ ca để quyết, giữ nguyên v1.0 và ghi `can_review` khi gặp.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004 | Tuk-tuk có mui, khoang khách | YOLO26 gán `car` | `three_wheeler` (lấy từ YOLOE) | Mục 2: xe ba bánh có mui vẫn là `three_wheeler` dù máy gọi `car`; khi duyệt kiểm mọi box `car` chồng lên xe ba bánh (CAL-01) |
| img_011 | Xích lô quay đầu / quay đuôi về camera | YOLO26 bỏ sót xích lô quay đầu, gán `bicycle` cho xích lô quay đuôi | `three_wheeler` ở mọi góc nhìn | Mục 2 + 4.3: lớp không phụ thuộc hướng xe; kiểm mọi box `bicycle` (CAL-02) |
| img_002, img_011, img_019 | Bán tải / ba gác | YOLO26 gán `truck` (4/5 box truck bị xoá khi duyệt) | Bán tải → `car`; ba gác → `three_wheeler` | Mục 2: `truck` chỉ cho xe tải ≥ 4 bánh có thùng hàng; thêm ví dụ bán tải → `car` (CAL-04) |

Ca `CAL-05` (bẫy missing lọt ở img_003, img_027) là lỗi của người duyệt, không phải guideline mơ hồ: **không đổi guideline**, chỉ siết checklist review.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2 (ngưỡng 20 px cho khối xe gần camera); thêm 3 ca khó: tuk-tuk bị gọi car, xích lô theo góc nhìn, bán tải / ba gác bị gọi truck | CAL-01, CAL-02, CAL-03, CAL-04 |
