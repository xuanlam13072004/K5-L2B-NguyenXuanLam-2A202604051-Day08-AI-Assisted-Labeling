# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 520; đối tượng sau duyệt: 511; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 449 | 449/520 = 86.3% | 520 box máy |
| edited (sửa) | 21 | 21/520 = 4.0% | 520 box máy |
| deleted (xoá) | 47 | 47/520 = 9.0% | 520 box máy |
| added (vẽ thêm) | 41 | 41/511 = 8.0% | 511 đối tượng sau duyệt |

Trong số deleted + added, **6 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 35.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 257 | 3 | 9 | 10 |
| bicycle | 5 | 1 | 2 | 1 |
| motorbike | 121 | 10 | 5 | 10 |
| car | 49 | 0 | 4 | 13 |
| bus | 3 | 0 | 0 | 3 |
| truck | 1 | 0 | 4 | 2 |
| three_wheeler | 13 | 7 | 23 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 38 | 36 | 10 | 51.4% (KTC 95%: 40–62%) | 79.2% (KTC 95%: 66–88%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 1/3
  - wrong_class: 2/3
  - loose: 2/3
  - lọt: img_003.jpg (missing), img_028.jpg (loose), img_009.jpg (wrong_class), img_027.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
