# Label contract — Phố châu Á — BỔ SUNG v1.1 — Le Quang Vinh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Phân biệt xe máy chở hàng cồng kềnh với xe ba bánh | Dựa vào số lượng bánh xe chính xác và kết cấu khung xe ba gác | img_021.jpg | CAL-02 |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013.jpg | Xe tuk-tuk chở khách có mui che | YOLO26-n gọi nhầm là car | Gán nhãn three_wheeler và giữ nguyên | Mục 2: Xe ba bánh có mui vẫn định nghĩa là three_wheeler |
| img_008.jpg | Xe máy nằm ở vùng khuất ánh sáng | Bỏ sót (missing) | Thêm box thủ công | Mục 4: Quét kỹ vùng biên và góc khuất trước khi bật pre-label |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc phân loại xe ba bánh và quy trình quét vùng khuất | CAL-01, CAL-02, CAL-03 |