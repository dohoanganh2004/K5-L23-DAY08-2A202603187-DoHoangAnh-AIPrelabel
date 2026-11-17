# Báo cáo Day 08 — <<MSHV - Họ và tên>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.3 (mặc định) | Không ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 88%, 17 vật gold); r01–r03 cho thấy hạ ngưỡng thì tăng recall nhưng thêm box chồng ở cảnh đông |
| bicycle | YOLO26m | 0.3 (mặc định) | Recall tối đa 60% (5 vật gold), mẫu ít nên không chọn riêng |
| motorbike | YOLO26m | 0.1 | Recall 92%, precision 67% (13 vật gold); r02 (conf 0.5) làm mất xe máy ở img_030 |
| car | YOLO26m | 0.3 (mặc định) | Recall tối đa 88% (8 vật gold), không đạt 90% |
| bus / truck | YOLO26m | 0.3 (mặc định) | Gold chỉ có 1 bus và 0 truck (< 5 vật): mẫu quá ít, không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | Recall 100%, precision 78% (7 vật gold); YOLO26 không có lớp ba bánh (r01–r03), YOLOE bắt được (r04, r07) nhưng còn sót / nhầm truck (img_002) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp ba bánh nên bỏ sót hoặc gọi sai (xích lô img_032 thành `bicycle`, tuk-tuk img_004 / img_013 thành `car` khi chạy SAM, r05): hạ ngưỡng không cứu được. Sửa bằng YOLOE cho `three_wheeler` rồi người duyệt lại nhãn; YOLOE lại nhầm truck img_002 thành ba bánh và sót 1 xích lô ở img_032 nên vẫn phải duyệt tay.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_032, YOLO26 gán `bicycle` 0.70 cho phần khung / bánh xích lô thay vì cả xe. Điểm thấp mà đúng: img_032, YOLOE bắt xích lô xa / bị che chỉ ở 0.27–0.48 (chiếc 0.92 là xích lô gần nhất). img_030: motorbike bắt được ở 0.25 nhưng box lỏng, mất ở 0.5.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh, 61 đối tượng) nên số đo máy trên gold là lạc quan; ngưỡng chỉ là ước lượng thô, nhất là lớp ít vật (bicycle 5, bus 1, truck 0). Chỉ motorbike (0.1) và three_wheeler (0.15) có ngưỡng riêng; ngưỡng thấp như 0.1 tăng recall nhưng giảm precision (67%) nên tăng việc xoá box thừa khi duyệt.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
