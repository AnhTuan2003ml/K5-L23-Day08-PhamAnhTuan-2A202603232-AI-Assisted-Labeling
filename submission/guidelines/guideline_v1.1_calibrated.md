# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603232 - Phạm Anh Tuấn

> **Chỉ viết phần thay đổi so với v1.0**. Chốt các câu hỏi mở ở mục 6 của v1.0 và thêm quy tắc mới rút ra từ lỗi của máy và quá trình review.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng hoá nhô ra ngoài xe (> 20% chiều dài xe) | Không gộp phần hàng nhô > 20% vào box xe hoặc mask xe | `img_031.jpg` | `CAL-01` |
| 4 | Người ngồi trong xe ba bánh có mui che | Vẫn tách `person` riêng nếu nhìn thấy >= 20% diện tích người; không gộp người lái vào mask `three_wheeler` | `img_013.jpg` | `CAL-02` |
| 5 | Kính chắn gió / cửa sổ xe | Kính chắn gió tính thuộc mask xe | `img_013.jpg` | `CAL-02` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_031.jpg` | Rổ hàng nhô ra phía sau xe đạp | `yolo26-m` vẽ box `bicycle` (#425) cắt ngang rổ hàng nhô xa | Giữ nguyên quy tắc không gộp phần hàng nhô > 20% chiều dài xe vào box/mask xe | Mục 4.1.5 / 4.2 |
| `img_013.jpg` | Người lái ngồi bên trong tuk-tuk | `sam2.1_b.pt` khi tách mask bằng điểm click trên xe đã nuốt luôn cả người lái | Kính chắn gió thuộc mask xe, nhưng người lái phải tách thành `person` riêng | Mục 4.2 |
| `img_003.jpg` | Hai box chồng nhau trên cùng xe ba gác và nhầm `bicycle` / `motorbike` | `yoloe-s` + `yolo26-m` tạo 2 box `three_wheeler` chồng nhau (#14, #15); bẫy `sai_lop` #11 bị lọt | Kiểm tra trùng box trên `three_wheeler` và đối chiếu kiểu xe (xe đạp vs xe máy) khi review | Checklist review + Mục 4.3 |
| `img_031.jpg` | Mũ cam treo trên kệ hàng bị nhận nhầm thành người | `yolo26-m` gán box `person` (#427, conf 0.34) lên chiếc mũ | Xoá bỏ, không gán nhãn cho quần áo/mũ trưng bày trên kệ | Mục 4.1.7 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc hàng nhô > 20%, tách người lái khỏi mask `three_wheeler`, tính kính chắn gió vào mask xe, loại bỏ đồ vật trưng bày (mũ/quần áo) và kiểm tra trùng box `three_wheeler` | `CAL-01`, `CAL-02`, `CAL-03`, `CAL-04` (`img_031.jpg`, `img_013.jpg`, `img_003.jpg`) |
