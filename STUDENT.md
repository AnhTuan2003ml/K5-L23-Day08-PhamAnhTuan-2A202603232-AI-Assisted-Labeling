# Thông tin học viên

- Họ và tên: Phạm Anh Tuấn
- MSHV: 2A202603232
- Lớp (L2 / L3 / khác): K5-L2
- Tài khoản GitHub: AnhTuan2003ml
- Repo bài nộp (URL): https://github.com/AnhTuan2003ml/K5-L23-Day08-PhamAnhTuan-2A202603232-AI-Assisted-Labeling
- Commit chốt bài (hash): (cập nhật ở commit cuối)
- Máy: Windows, GPU CUDA (NVIDIA GeForce RTX 4060 8GB)
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 469 / 6 / 30 (mẫu số: 505 box máy); added: 14 (mẫu số: 489 đối tượng)
- So với gold: precision / recall của **máy** 41.1% (30/73) / 62.5% (30/48)
- Bắt bẫy: 12/12 = 100.0% (sau rework; lượt đầu 4/12 = 33.3%)
- Thời gian: gán tay 2.0 phút/ảnh (8 phút / 4 ảnh); duyệt có pre-label 1.21 phút/ảnh (29 phút / 24 ảnh)
- Lỗi QC: 20.0% theo ảnh (1/5), 1.0% theo đối tượng (1/99)
- **Quyết định:** Rework — lý do một dòng: Đã sửa 8 bẫy lọt (đạt 12/12), nhưng mẫu QC vẫn còn 1/5 = 20.0% ảnh lỗi (#426 trên img_031.jpg > 2%).
