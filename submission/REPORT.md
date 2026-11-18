# Báo cáo Day 08 — 2A202603232 - Phạm Anh Tuấn

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | `yolo26-m` | `0.3` (mặc định) | `r01` (conf `0.25`, 214 box trên `pilot`) nhiều box nhiễu; `r02` (conf `0.50`, 141 box) làm rớt nhiều `person` (`88` vs `131`) và `motorbike` (`33` vs `55`). Dùng `0.3` làm mức trung bình khi merge. |
| three_wheeler | `yoloe-s` (`yoloe-26s-seg.pt`, prompt: `tuk-tuk`, `cyclo`, `cargo tricycle`) | `0.3` (mặc định) | `r03` (`pilot`, 8 box, `0.13s/ảnh`) bắt được tuk-tuk/xích lô mà COCO 80 lớp của YOLO26 không có. |

- Lệnh `merge_prelabels.py` đã dùng:
  `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM đã chọn**: `sam2.1_b.pt` (SAM 2.1 Base). Lý do: chạy nhanh trên RTX 4060 (`1.31s/ảnh` ở `r04` kèm YOLO26), không cần xin quyền gated như SAM 3 và tách mask đủ dùng cho Part 2 và Part 5.
- **Lỗi không sửa được bằng đổi ngưỡng**: Tuk-tuk hoặc xe ba gác bị `yolo26-m` gọi nhầm thành `car`/`truck`, hoặc ngược lại `yoloe-s` bắt trùng trên cùng vật (`img_003.jpg`). Đổi ngưỡng conf không giải quyết được lỗi sai lớp; phải ưu tiên `yoloe` cho lớp `three_wheeler` khi merge và sửa thủ công khi review.
- **Box điểm cao mà sai / điểm thấp mà đúng**: Ở `img_031.jpg`, `yolo26-m` gán nhầm mũ cam trên kệ thành `person` (`#427`, conf `0.34`); ở `img_003.jpg`, hai box `three_wheeler` (`#14` conf `0.90` và `#15` conf `0.80`) cùng chồng lên một xe kéo.
- **Giới hạn `threshold_sweep`**: Tập `gold` chỉ có 4 ảnh (`48` đối tượng). `bicycle` (4), `bus` (0), `truck` (1) đều `< 5` mẫu nên không thể chọn ngưỡng riêng; các lớp còn lại không đạt recall 90% ở `<= 0.5`, đồng thời việc chọn ngưỡng và đánh giá trên cùng 4 ảnh `gold` sẽ cho số đo lạc quan hơn thực tế.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 468 | 468/505 = 92.7% | 505 box máy |
| edited | 7 | 7/505 = 1.4% | 505 box máy |
| deleted | 30 | 30/505 = 5.9% | 505 box máy |
| added | 14 | 14/489 = 2.9% | 489 đối tượng sau duyệt |

*(Trong số deleted + added có 12 trường hợp đổi lớp; số vật máy sót thật khoảng `14 - 12 = 2`.)*

- **Lớp máy tệ nhất**: `three_wheeler` (`9` accepted, `3` edited, `12` deleted, `3` added) và `truck` (`2` accepted, `5` deleted, `2` added) do dễ bị nhận nhầm giữa xe ba gác, xe tải nhỏ và ô tô.
- **So với gold (4 ảnh, 48 đối tượng)**: Precision máy = `30/73 = 41.1%` (KTC 95%: `31–53%`), Recall máy = `30/48 = 62.5%` (KTC 95%: `48–75%`).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy**: Ở lượt review đầu tiên chỉ bắt được `4/12 = 33.3%` (`extra: 3/3`, `wrong_class: 1/3`, `loose: 0/3`, `missing: 0/3`; lọt `img_003`, `img_004`, `img_009`, `img_021`, `img_022`, `img_025`, `img_027`, `img_029`). Sau vòng Rework sửa 8 bẫy lọt, tỉ lệ bắt bẫy cuối đạt `12/12 = 100.0%` (KTC 95%: `76–100%`).
- **Nguyên nhân**: Ở lượt đầu mình bị thiên kiến tự động hóa — tin vào box có sẵn của máy nên chỉ xoá box thừa rõ ràng (`extra: 3/3`), nhưng bỏ qua bước ẩn nhãn máy để quét vật bị sót (`missing: 0/3`) và không kiểm tra kỹ độ ôm sát của từng cạnh box (`loose: 0/3`).
- **Accepted cao nhưng bắt bẫy thấp**: `accepted = 92.7%` trong khi lượt đầu bắt bẫy chỉ `33.3%` cho thấy phần lớn box giữ nguyên là do duyệt lướt theo đề xuất của máy, không đồng nghĩa với chất lượng nhãn sạch lỗi.

## 4. Máy có đáng dùng không?

- **Thời gian (từ `time_log.csv`)**: Gán tay `gold_manual` mất `8 phút / 4 ảnh = 2.0 phút/ảnh`; duyệt `batch_review` mất `29 phút / 24 ảnh = 1.21 phút/ảnh`. (Có thiên lệch nhẹ do đã quen cảnh khi gán 4 ảnh gold trước, nhưng bù lại 6 ảnh batch dày xe máy mất nhiều thời gian hơn.)
- **Kết luận**: Máy **đáng dùng để tạo khung ban đầu** cho `person`, `motorbike`, `car` vì giảm thời gian từ `2.0` xuống `1.21 phút/ảnh` với `93.3%` box giữ lại, nhưng **không được duyệt lướt** và với `three_wheeler` vẫn cần kiểm tra kỹ từng ảnh.
- **Ba quy tắc giảm thiên kiến**:
  1. Bắt buộc nhấn phím ẩn toàn bộ box máy trên CVAT để tự quét đếm nhanh các vật trước khi bật lại nhãn (kiểm tra được qua việc giảm bẫy `missing` lọt từ `3/3` xuống `0/3`).
  2. Với các ảnh có `three_wheeler` hoặc xe máy dày (`img_005`, `018`, `021`, `024`, `028`, `029`), kiểm tra từng cạnh box xem có hở > 2 px hay hai box chồng lên nhau không (kiểm tra được qua tỷ lệ bắt bẫy `loose` >= `80%`).
  3. So sánh chéo mọi box `bicycle` và `motorbike` xem có bị gán nhầm lớp không (kiểm tra được qua tỷ lệ bắt bẫy `wrong_class` = `3/3`).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | Có, mask ôm luôn cả người lái ngồi trong khoang tuk-tuk | Dính thêm phần ghế sau qua khung hở và một phần bánh xe máy phía sau | `tuktuk_img013.jpg` |
| `img_004.jpg` | `sam2.1_b.pt` | 2 (1 pos, 1 neg) | Không dính người đứng cạnh | Hơi sót biển TAXI trắng trên nóc và lem nhẹ sang áo người đứng bên phải | `tuktuk_img004.jpg` |
| `img_008.jpg` | `sam2.1_b.pt` | 3 (2 pos, 1 neg) | Không gồm hàng hoá | Chỉ tách được phần bánh xe và khung gầm dưới, bỏ sót toàn bộ thùng chuối phía trên | `cyclo_img008.jpg` |

- **Đối chiếu guideline mục 4.2**: Quy tắc tách riêng `person` và `three_wheeler` giúp nhận ra ngay lỗi ở `img_013.jpg` (SAM gộp người lái vào xe); quy tắc 4.2 còn thiếu hướng dẫn rõ cho trường hợp hàng hoá khác màu/vật liệu đặt trên xe ba gác như ở `img_008.jpg` (cần click thêm điểm lên thùng hàng nếu hàng nằm gọn trong phần gắn cố định của xe).
- **Interactor SAM trong CVAT**: Không dùng Interactor trực tiếp trong CVAT ở bước này (chạy offline qua `scripts/sam2_points.py` để tái lập chính xác toạ độ click).

## 6. Calibration (Part 6)

- **Số quyết định**: `4` quyết định trong [calibration_decisions.csv](qc/calibration_decisions.csv) (`CAL-01` đến `CAL-04`); quan trọng nhất là `CAL-03` (`img_003.jpg` — bổ sung kiểm tra trùng box `three_wheeler` và kiểm tra chéo `bicycle`/`motorbike`) và `CAL-02` (`img_013.jpg` — chốt tách người lái và kính chắn gió).
- **Quy tắc mới vào `guideline_v1.1_calibrated.md`**: Chốt câu hỏi mở mục 6 (câu 3, 4, 5), mở rộng mục 4.1.7 không gán nhãn cho mũ/quần áo trưng bày trên kệ (`CAL-04`, `img_031.jpg`), và thêm bước kiểm tra trùng box `three_wheeler`.
- **Nguyên nhân gốc**: Kết hợp cả hai phía — guideline v1.0 còn câu hỏi mở về hàng nhô/người trong khoang xe/kính xe, và lỗi của máy + duyệt nhanh gây lọt lỗi trên `img_003.jpg`, `img_031.jpg`.

## 7. QC và quyết định (Part 6)

- **Mẫu**: `5/24` ảnh batch (`0` ca `can_review` + `5` ảnh ngẫu nhiên seed 42: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`), tổng `99` đối tượng. Hạn chế: người kiểm QC cũng chính là người review nên không độc lập hoàn toàn.
- **Lỗi theo đối tượng**: `0/99 = 0.0%` · **Lỗi theo ảnh**: `0/5 = 0.0%` (đạt ngưỡng Release `<= 2%`, sau khi đã sửa các lỗi vòng 1: `#11` sai lớp và `#15` trùng trên `img_003.jpg`; `#427` mũ trên kệ và `#426` box người nửa thân trên `img_031.jpg`).
- **Top lỗi**: Không còn lỗi nào trên mẫu 5 ảnh sau vòng Rework (vòng 1 ghi nhận `thua` x2, `sai_lop` x1, `sai_vi_tri_kich_thuoc` x1).
- **Giới hạn mẫu nhỏ**: Với `0/5` ảnh lỗi, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật vẫn là `43%` (`qc_report_auto.md`); mẫu 5 ảnh không đủ để chứng minh tuyệt đối ngưỡng `<= 2%` cho toàn bộ 24 ảnh.
- **Script kiểm được / không kiểm được**: Script (`validate_labels.py`) kiểm được schema, nhãn hợp lệ và cảnh báo box `< 20px`, nhưng không tự phát hiện được box gắn nhầm lên đồ vật (`#427`), box sai lớp (`#11`), hay box ôm thiếu thân người (`#426`).

**Quyết định: RELEASE**, ngày `2026-11-18`. Lý do (theo cây Hold -> Rework -> Release): guideline v1.1 đã chốt đủ câu hỏi mở liên quan (không Hold), tỉ lệ bắt bẫy sau rework đạt `12/12 = 100.0%` (>= `80%`) và tỉ lệ ảnh lỗi trên mẫu QC sau rework là `0/5 = 0.0%` (<= `2%`).
Việc cần làm tiếp: nếu đưa vào sử dụng thực tế trên tập lớn hơn, cần lấy mẫu QC độc lập thêm để giảm cận trên Wilson từ `43%`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Có dùng Antigravity hỗ trợ cài đặt `.venv`, upload/export dữ liệu qua CVAT local, chạy các script `prelabel.py` / `threshold_sweep.py` / `review_stats.py` / `sam2_points.py` / `qc_sample.py` / `qc_report.py` và tổng hợp số liệu từ output thật vào báo cáo; tự kiểm tra trực tiếp bằng mắt trên CVAT và ảnh trong `work/qc/review/`, `submission/masks/`.
- **Gold có bị nhiễm không**: Không, 4 ảnh Gold (`img_001`, `img_012`, `img_020`, `img_036`) được gán tay trên CVAT trước khi chạy model.
- **Máy / GPU / CVAT dùng**: Windows, NVIDIA GeForce RTX 4060 (8GB), CVAT Docker v2.82.1 tại `http://100.104.206.55:8080`.
- **Dùng cache `data/cache/` thay vì tự chạy?**: Không, tự chạy toàn bộ model trên GPU RTX 4060.
- **Lỗi / bug script đã gặp**: Khi chạy `merge_prelabels.py --thr ""` trong PowerShell bị lỗi thiếu tham số vì chuỗi rỗng bị nuốt (xử lý bằng cách bỏ cờ `--thr` để dùng `--conf 0.3`).
- **Một điều làm khác lần sau**: Khi review sẽ ẩn toàn bộ box máy ngay từ đầu mỗi ảnh để tự quét hết vật trước, tránh bị lọt các bẫy `missing` và `loose`.
