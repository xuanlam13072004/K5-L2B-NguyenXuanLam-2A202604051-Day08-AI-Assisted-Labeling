# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 3 ảnh (0 ca khó + 3 ngẫu nhiên), 53 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 1/53 = 1.9% | 53 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 1/3 = 33.3% | 3 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 449/520 = 86.3% / 21/520 = 4.0% / 47/520 = 9.0% | 520 box máy |
| Máy: added | 41/511 = 8.0% | 511 đối tượng sau duyệt |
| Bắt lỗi bẫy | 8/12 = 66.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 1 | img_003.jpg |

## Giới hạn thống kê
- 1 ảnh lỗi trong 3 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **79%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 33.3% > 2%
- bắt bẫy 67% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
