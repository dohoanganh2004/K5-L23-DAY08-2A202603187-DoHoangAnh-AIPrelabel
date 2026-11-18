# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603187 - Đỗ Hoành Anh

> **Chỉ viết phần thay đổi so với v1.0.** Mỗi ca có ảnh ví dụ và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy dày đặc: từng xe hay `ignore_region`? | Quyết theo **chiều cao** từng vật: vật cao ≥ 20 px vẽ từng xe kể cả khi đông; cụm từ 5 vật trở lên, mỗi vật cao < 20 px, sát nhau thì một `ignore_region`. Không dùng `ignore_region` cho vật rõ chỉ vì đông | img_018 | CAL-03 |
| 4, 5 (một phần) | Người ngồi trong xe ba bánh và chỗ ngồi: thuộc mask xe không? | Mask xe **không gồm người** (người là vật riêng). Ghế, chỗ ngồi, buồng lái gắn liền xe **vẫn thuộc mask xe** dù có người ngồi lên | img_013 | CAL-04 |
| 1, 3, 6, 7 | Xe đạp điện, hàng nhô ra, box chồng ở ảnh đêm, ngưỡng conf tối thiểu | **Chưa chốt:** chưa gặp đủ ca trong 24 ảnh batch để quyết định có bằng chứng | — | — |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_004, img_013, img_032 | Tuk-tuk, xích lô | YOLO26 gán `car` / `bicycle` hoặc sót; ba bánh là lớp bị sửa nhiều nhất | Luôn gán `three_wheeler`; kiểm lại nhãn mọi box xe có 3 bánh | Mục 2: thêm ví dụ máy gọi `car`, `bicycle`, `truck` (CAL-01) |
| img_014 | Gánh hàng, thúng treo | YOLOE gán `three_wheeler` cho gánh hàng (3 box, conf 0,19 đến 0,63) | Không gán: chỉ gán `three_wheeler` cho xe có bánh | Mục 2, cột "Không gồm": thêm gánh hàng, thúng treo, vật mang vác (CAL-05) |
| img_002 | Xe tải bị gọi ba bánh | YOLOE gán `three_wheeler` cho `truck` | Đếm bánh: 3 bánh là `three_wheeler`, ≥ 4 bánh có buồng lái và thùng là `truck`; không đếm được thì `can_review` | Mục 4.3: đếm bánh là bước bắt buộc (CAL-02) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ lớp `three_wheeler`, đếm bánh, ranh giới `ignore_region`, mask xe không gồm người | CAL-01 đến CAL-05 |
