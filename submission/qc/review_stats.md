# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 622; đối tượng sau duyệt: 609; ngưỡng accepted: IoU >= 0.9
- Đã **loại 24 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 512 | 512/622 = 82.3% | 622 box máy |
| edited (sửa) | 37 | 37/622 = 5.9% | 622 box máy |
| deleted (xoá) | 73 | 73/622 = 11.7% | 622 box máy |
| added (vẽ thêm) | 60 | 60/609 = 9.9% | 609 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 49.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 256 | 10 | 7 | 17 |
| bicycle | 5 | 1 | 2 | 4 |
| motorbike | 189 | 15 | 32 | 13 |
| car | 49 | 1 | 5 | 15 |
| bus | 2 | 0 | 1 | 3 |
| truck | 1 | 0 | 5 | 4 |
| three_wheeler | 10 | 10 | 21 | 4 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 51 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 23 | 8 | 65.2% (KTC 95%: 53–76%) | 84.3% (KTC 95%: 72–92%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 8/12 = 66.7% (KTC 95%: 39–86%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 2/3
  - loose: 2/3
  - extra: 2/3
  - missing: 2/3
  - lọt: img_035.jpg (wrong_class), img_004.jpg (loose), img_027.jpg (missing), img_005.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
