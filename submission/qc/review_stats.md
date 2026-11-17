# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 328; đối tượng sau duyệt: 440; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 317 | 317/328 = 96.6% | 328 box máy |
| edited (sửa) | 5 | 5/328 = 1.5% | 328 box máy |
| deleted (xoá) | 6 | 6/328 = 1.8% | 328 box máy |
| added (vẽ thêm) | 118 | 118/440 = 26.8% | 440 đối tượng sau duyệt |

Trong số deleted + added, **2 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 116.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 164 | 1 | 1 | 67 |
| bicycle | 3 | 0 | 2 | 4 |
| motorbike | 77 | 3 | 0 | 26 |
| car | 29 | 1 | 1 | 19 |
| bus | 1 | 0 | 1 | 1 |
| truck | 2 | 0 | 0 | 0 |
| three_wheeler | 41 | 0 | 1 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 50 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 33 | 9 | 17 | 78.6% (KTC 95%: 64–88%) | 66.0% (KTC 95%: 52–78%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 1/3
  - loose: 0/3
  - wrong_class: 2/3
  - missing: 2/3
  - lọt: img_021.jpg (extra), img_013.jpg (extra), img_029.jpg (loose), img_008.jpg (loose), img_017.jpg (wrong_class), img_006.jpg (loose), img_014.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
