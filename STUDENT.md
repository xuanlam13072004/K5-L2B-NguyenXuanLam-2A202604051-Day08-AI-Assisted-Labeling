# Thông tin học viên

- Họ và tên: Nguyễn Xuân Lam
- MSHV: 2A202604051
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: xuanlam13072004
- Repo bài nộp (URL): https://github.com/xuanlam13072004/K5-L2B-NguyenXuanLam-2A202604051-Day08-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Windows, NVIDIA GeForce RTX 3050 Laptop GPU (CUDA 12.8, torch 2.11.0+cu128)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 449 / 21 / 47 (mẫu số: 520 box máy); added: 41 (mẫu số: 511 đối tượng)
- So với gold: precision / recall của **máy** 38/74 = 51.4% / 38/48 = 79.2%
- Bắt bẫy: 8/12 = 66.7%
- Thời gian: gán tay 3.75 phút/ảnh (15/4); duyệt có pre-label 0.83 phút/ảnh (20/24)
- Lỗi QC: 33.3% theo ảnh (1/3), 1.9% theo đối tượng (1/53)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 1/3 > 2% và bắt bẫy 8/12 < 80%, lô duyệt chưa đáng tin
