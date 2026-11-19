# Báo cáo Day 08 — 2A202604051 - Nguyễn Xuân Lam

> Mọi con số kèm mẫu số và nguồn. Máy: RTX 3050 Laptop (CUDA), CVAT 2.81.0 Docker.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | 0.3 (mặc định, `--conf 0.3`) | r02/r03: gán tốt person, motorbike, car ở ảnh rõ nét; conf 0.5 làm mất nhiều vật nhỏ (img_007 person 15→6). `threshold_sweep`: person / motorbike / car không ngưỡng nào đạt recall 90% (tối đa 14/18, 10/12, 7/8); bicycle (3 vật), bus, truck (0 vật) < 5 vật nên giữ mặc định |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | r04: YOLOE tìm được xe ba bánh mà YOLO26 gọi sai; `threshold_sweep`: ở 0.15 recall 7/7 = 100%, precision 7/9 = 78%; lên 0.3 recall còn 5/7 |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM:** SAM 2.1 base (`sam2.1_b.pt`). Máy có GPU RTX 3050 nên chạy được bản base (~3.1 s/ảnh, r05); chất lượng mask tốt hơn SAM 1 (`mobile_sam`), không cần quyền truy cập như SAM 3.
- **Lỗi không sửa được bằng đổi ngưỡng:** YOLO26 gọi tuk-tuk là `car` (img_013, img_004), xích lô quay đuôi là `bicycle` và bỏ sót xích lô quay đầu (img_011). COCO không có lớp xe ba bánh nên tăng / giảm ngưỡng chỉ đổi số box, không đổi lớp. Cách sửa: lấy lớp `three_wheeler` từ YOLOE (gõ chữ) và gộp bằng `merge_prelabels`, rồi người duyệt kiểm mọi box `car` / `bicycle` / `truck` chồng lên xe ba bánh.
- **Box điểm cao mà sai:** box `car` của YOLO26 trên tuk-tuk; `truck` trên bán tải / ba gác (4/5 box truck bị xoá khi duyệt). **Điểm thấp mà đúng:** YOLOE cho xe ba bánh đúng điểm thấp (0.15–0.3), nên lên 0.3 mất 2/7 xe trên gold.
- **Giới hạn:** ngưỡng chọn và chấm trên chính 4 ảnh gold (48 vật) nên precision / recall của máy trên gold là lạc quan; lớp ít vật (bicycle, bus, truck) không có kết luận.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 449 | 86.3% | 520 box máy |
| edited | 21 | 4.0% | 520 box máy |
| deleted | 47 | 9.0% | 520 box máy |
| added | 41 | 8.0% | 511 đối tượng sau duyệt |

Đã loại 22 đối tượng quanh bẫy khỏi bảng (lỗi nhân tạo, không phản ánh máy). Trong deleted + added có 6 ca đổi lớp, nên máy sót thật ≈ 35 vật.

- **Lớp tệ nhất: `three_wheeler`**: 43 box máy, xoá 23 (23/43 = 53%), chỉ giữ nguyên 13. Ngưỡng 0.15 ưu tiên recall nên kéo theo nhiều box sai; ngưỡng chọn trên 4 ảnh gold không khớp với 24 ảnh batch. `truck`: 4/5 box bị xoá (bán tải / ba gác). `car`: 13 box vẽ thêm. `person` (257/269) và `motorbike` (121/136) giữ nguyên trên 89%.
- **So với gold:** precision của máy 38/74 = 51.4% (KTC 95% 40–62%), recall 38/48 = 79.2% (KTC 95% 66–88%); mẫu số 48 đối tượng gold. Máy sót 10/48 trên gold nhưng `added` trên batch chỉ 8%: có thể một số vật máy sót tôi cũng sót theo, nên `added` thấp không chứng minh máy ít sót.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **8/12 = 66.7%** (KTC 95% 39–86%), dưới ngưỡng 80%. Theo loại: extra 3/3, loose 2/3, wrong_class 2/3, **missing 1/3**. Lọt: img_003 và img_027 (missing), img_009 (wrong_class), img_028 (loose, ảnh dày xe máy).
- Nguyên nhân: tôi bắt tốt lỗi **nhìn thấy được trên nhãn máy** (box thừa, box lỏng) nhưng yếu với vật **không có box** (missing). Tôi chưa làm đúng bước 0 (ẩn nhãn, tự quét ảnh trước), nên mắt chỉ kiểm những gì máy đưa ra. img_009: bỏ bước kiểm nhãn từng box; img_028: box nhỏ trong ảnh dày, khó thấy độ lỏng.
- Accepted 86.3% cùng bắt bẫy 66.7% và tốc độ 0.83 phút/ảnh: tôi giữ phần lớn box máy với tốc độ nhanh hơn gợi ý (~3 phút/ảnh) và để lọt 1/3 số bẫy. Accepted cao không có nghĩa máy tốt, một phần là tôi chấp nhận mà chưa kiểm hết.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay 15 phút / 4 ảnh = 3.75 phút/ảnh; duyệt có pre-label 20 phút / 24 ảnh = 0.83 phút/ảnh. Thiên lệch: tôi gán gold trước nên đã quen cảnh và CVAT; phần tốc độ còn đến từ duyệt chưa kỹ (bẫy 8/12).
- Kết luận: **có, cho person và motorbike** (accepted trên 89%); **không tin trực tiếp cho `three_wheeler`, `truck`** (xoá 53% và 4/5), phải kiểm từng box hoặc tăng ngưỡng.
- **Ba quy tắc giảm thiên kiến:**
  1. Mỗi ảnh ẩn nhãn máy, ghi số vật tự đếm trước khi hiện nhãn; kiểm bằng cách so số tự đếm với số box sau duyệt, và bắt bẫy missing phải ≥ 2/3.
  2. Một lượt riêng chỉ kiểm box `three_wheeler` / `car` / `bicycle` / `truck` (lớp máy hay gọi sai); kiểm bằng tỉ lệ wrong_class bắt được.
  3. Không duyệt nhanh hơn 2 phút/ảnh và bấm Ctrl+S sau mỗi ảnh; kiểm bằng `time_log.csv` và đếm box ở một ảnh đã sửa trước khi export.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (v1) | SAM 2.1 base | 2 dương | Mask lan sang vật bên cạnh | Không có click âm nên mask tràn sang vật khác | tuktuk_img013_v1 |
| img_013 | SAM 2.1 base | 4 (2 dương + 2 âm: xe máy bên cạnh, người lái) | Vẫn lan sang vật khác dù đã thêm click âm | 2 click âm chưa đủ để tách xe khỏi vật chạm vào | tuktuk_img013 |
| img_004 | SAM 2.1 base | 2 dương | Mask lan sang vật khác | Nhiều xe ba bánh đỗ sát nhau, chỉ click dương nên mask gộp vật lân cận | tuktuk_img004 |
| img_008 | SAM 2.1 base | 2 (1 dương + 1 âm: xe ba bánh sát bên) | Mask lan sang vật khác | Hai xe ba bánh chồng box, 1 click âm chưa đủ | tw_img008 |

- Kết luận: với xe ba bánh đứng sát vật khác (xe máy, người, xe ba bánh khác), SAM 2.1 cần nhiều click âm hơn 1–2 điểm; số click tối thiểu để có mask sạch phụ thuộc số vật chạm vào. Lần sau thêm click âm trên từng vật chạm, hoặc dùng box làm prompt kèm click âm.
- Đối chiếu guideline 4.2: người là vật riêng nên mask xe không gồm người lái; quy tắc 4.2.4 (hai vật chạm nhau không chồng mask) cần click âm khi xe sát nhau, như ở Part 2 mask img_013 tràn sang xe máy bên cạnh (r05).
- Interactor SAM trong CVAT: không thử (thiếu thời gian, cần cài Nuclio).

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`): 3 `loi_may`, 1 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất: CAL-01 / CAL-02 (xe ba bánh bị gọi `car` / `bicycle` / bỏ sót theo góc nhìn).
- Quy tắc mới vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 2 (ngưỡng 20 px cho khối xe gần camera, img_018); tuk-tuk, xích lô mọi góc nhìn là `three_wheeler`; bán tải → `car`, `truck` chỉ cho xe tải ≥ 4 bánh.
- Nguyên nhân gốc: CAL-01, 02, 04 do **máy** (COCO không có lớp xe ba bánh); CAL-03 do **guideline mơ hồ**; CAL-05 do **tôi** (bỏ bước quét ảnh), không đổi guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 3 ảnh (img_003, img_006, img_031; 53 đối tượng) (`qc_sample.py --n 3 --seed 42`; không có box `can_review` nên chỉ gồm ảnh ngẫu nhiên); người kiểm: tôi (**hạn chế**: vừa là người duyệt).
- Lỗi theo **đối tượng**: 1/53 = 1.9% · theo **ảnh**: 1/3 = 33.3% (ngưỡng Release ≤ 2%).
- Top lỗi: bỏ sót (1 lỗi, img_003: một person nhỏ bên trái không có box; là bẫy missing lọt ở Part 4). Không thấy lỗi thừa, sai lớp, sai vị trí trong mẫu. Lỗi bỏ sót không hiện trên ảnh QC (chỉ vẽ box đã gán) nên dễ bị đánh giá là sạch.
- **Giới hạn mẫu nhỏ:** 1 ảnh lỗi / 3 ảnh mẫu, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 79%; với vài ảnh, ngưỡng 2% không thể chứng minh.
- Script kiểm được số box, lớp, accepted / edited / deleted / added; không kiểm được box có ôm sát thật, vật mà cả máy và tôi cùng bỏ sót, hay tôi có làm đúng checklist không.

**Quyết định: Rework**, ngày 2026-11-19. Lý do (Hold → Rework → Release): không Hold vì guideline đã chốt các ca chính ở v1.1 và dữ liệu không hỏng; Rework vì ảnh có lỗi 1/3 = 33.3% > 2% và bắt bẫy 8/12 = 66.7% < 80%, nên lô duyệt chưa đáng tin.
Việc cần làm tiếp: duyệt lại 24 ảnh theo checklist, ưu tiên bước 0 (ẩn nhãn, tự đếm vật) để tìm vật bị sót; sửa 4 bẫy lọt (img_003, img_027 missing; img_009 wrong_class; img_028 loose) và các box < 20 px (img_018…) vào `labels/qc_fixed.json`; chạy lại `build_release.py`, `qc_sample.py --force` với mẫu lớn hơn (≥ 10 ảnh) và ghi vòng sau vào báo cáo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude (trợ lý chat) để giải thích lệnh, sửa lỗi cài đặt (CVAT migration, torch CUDA), định dạng `model_log.csv` / `time_log.csv` từ quan sát của tôi, đọc output `threshold_sweep` / `review_stats` và soạn nháp các mục báo cáo, guideline v1.1, calibration từ quan sát và số liệu của tôi. Toạ độ click SAM lấy từ box `three_wheeler` trong nhãn tôi đã duyệt. Tôi đã đọc lại, đối chiếu số với file `qc/*` và sửa. Không gửi ảnh nào cho AI.
- Gold không bị nhiễm: gán 4 ảnh gold trước khi chạy bất kỳ model nào, không nhìn pre-label, không hỏi AI về ảnh gold.
- Máy: Windows, RTX 3050 Laptop GPU (torch 2.11.0+cu128; ban đầu cài nhầm bản CPU, đã cài lại); CVAT 2.81.0 Docker (gặp cảnh báo migration lúc khởi động, chạy `manage.py migrate`).
- Không dùng cache `data/cache/`.
- Bug script: không gặp.
- Lần sau: quét ảnh trước khi hiện nhãn máy, và lưu (Ctrl+S) sau mỗi ảnh.
