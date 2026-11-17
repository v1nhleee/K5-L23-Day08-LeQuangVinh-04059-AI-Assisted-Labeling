# Báo cáo Day 08 — 04059 - Lê Quang Vinh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26-n | --conf 0.25 (r01) / Threshold sweep Part 3 | r01 bắt tốt các đối tượng phổ biến rõ nét trên tập pilot. |
| three_wheeler | YOLOE-26s (prompt: three_wheeler) | --conf 0.25 (r02) | r02 tìm chuyên biệt được các xe ba bánh / tuk-tuk mà YOLO26 bỏ sót. |

- Lệnh `merge_prelabels.py` đã dùng: `python scripts/merge_prelabels.py --inputs work/prelabel/t_y26 work/prelabel/t_yoloe --weights "1.0,1.0" --out submission/prelabel/pre_traps.coco.json`
- **Phiên bản SAM bạn chọn**: SAM 2.1 (`sam2.1_b.pt`) vì cân bằng tốt giữa chất lượng tạo mask chi tiết, tốc độ chạy trên CPU và tính tương thích sẵn có trong thư viện lab.
- Lỗi nào **không sửa được bằng đổi ngưỡng**: Tuk-tuk bị model nhầm lẫn nhận diện thành ô tô (`car`) hoặc xe tải (`truck`). Cần sửa thủ công trên CVAT hoặc dùng model chuyên biệt như YOLOE.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**: Các vùng phản chiếu ánh sáng mạnh hoặc vệt sáng chuyển động ban đêm dễ làm model tự tin nhận diện nhầm box điểm cao.
- Giới hạn: Chọn ngưỡng trên chính 4 ảnh gold nên số đo máy trên gold có tính chất lạc quan (in-sample).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 317 | 317/328 = 96.6% | 328 box máy |
| edited (sửa) | 5 | 5/328 = 1.5% | 328 box máy |
| deleted (xoá) | 6 | 6/328 = 1.8% | 328 box máy |
| added (vẽ thêm) | 118 | 118/440 = 26.8% | 440 đối tượng sau duyệt |

- Lớp nào máy tệ nhất: Lớp `bicycle` và `person` ở các khu vực đông đúc do dễ bị che khuất hoặc nhập nhèm với vệt sáng.
- So với gold: Precision của máy là **78.6%** (KTC 95%: 64–88%), Recall của máy là **66.0%** (KTC 95%: 52–78%).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **5/12 = 41.7%** (KTC 95%: 19–68%); loại bẫy lọt chủ yếu ở dạng `loose` và `extra`.
- Accepted cao (96.6%) mà bắt bẫy thấp (41.7%) cho thấy quá trình duyệt có xu hướng cả tin vào khung pre-label của máy, lướt nhanh qua các lỗi tinh vi do model cài cậy sẵn.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): Gán tay **20 phút / 4 ảnh** (5 phút/ảnh) vs Duyệt **30 phút / 24 ảnh** (1.25 phút/ảnh). 
- Kết luận: **Có đáng dùng** (đặc biệt hữu ích cho các lớp đối tượng phổ biến như `person`, `motorbike`, giúp tiết kiệm thời gian đáng kể so với việc vẽ thủ công hoàn toàn).
- **Ba quy tắc giảm thiên kiến**:
  1. Quét tổng thể toàn bộ ảnh trước khi bật lớp pre-label của máy lên xem.
  2. Kiểm tra kỹ các khu vực rìa ảnh và góc khuất thay vì chỉ nhìn vào trung tâm.
  3. Tuân thủ nghiêm ngặt checklist xử lý đối tượng nhỏ/dày đặc bằng `ignore_region`.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_004.jpg | sam2.1_b.pt | 1 (`--pos "350,420"`) | Có | Ôm sát thân xe tốt | mask_img004.jpg[cite: 5] |
| img_013.jpg | sam2.1_b.pt | 1 (`--pos "400,480"`) | Có | Đôi khi dính nền nhẹ | mask_img013.jpg[cite: 5] |
| img_015.jpg | sam2.1_b.pt | 1 (`--pos "300,350"`) | Có | Rất sắc nét | mask_img015.jpg[cite: 5] |

- Đối chiếu guideline: Quy tắc click điểm dương trong thân xe giúp SAM bắt vùng rất nhanh; tuy nhiên thiếu điểm âm (`--neg`) dễ làm mask bị loang ra ngoài khi vật thể có màu trùng với nền.

## 6. Calibration (Part 6)

- Số quyết định: Các điều chỉnh trong `calibration_decisions.csv`.
- Quy tắc mới trong `guideline_v1.1_calibrated.md`: Quy chuẩn rõ cách xử lý các vệt sáng chuyển động ban đêm và vùng `ignore_region` cho các cụm xe máy dày đặc dưới 20px.
- Nguyên nhân gốc: Do guideline ban đầu còn mơ hồ ở các góc chụp thiếu sáng và mật độ giao thông cao.

## 7. QC và quyết định (Part 6)

- Mẫu: 24 ảnh review.
- Top lỗi: Bỏ sót (`missing`) và nhận sai lớp (`wrong_class`).
- **Quyết định: Hold**, ngày 2026-06-07. Lý do: Tỷ lệ bắt bẫy đạt 41.7% (thấp hơn ngưỡng tiêu chuẩn 80%), cho thấy cần rà soát và kiểm tra lại chất lượng lô dữ liệu kỹ hơn trước khi đưa vào Release chính thức.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Có sử dụng trợ lý AI để hỗ trợ giải thích cú pháp, kiểm tra lỗi script và định dạng báo cáo.
- Gold không bị nhiễm (tuân thủ quy trình gán tay độc lập trước khi chạy pre-label).
- Môi trường: Chạy trên CPU local với Python venv.
- Một điều làm khác lần sau: Tăng cường tập trung kiểm tra các bẫy tinh vi do hệ thống cài sẵn thay vì quá tin tưởng vào độ chính xác của pre-label.
