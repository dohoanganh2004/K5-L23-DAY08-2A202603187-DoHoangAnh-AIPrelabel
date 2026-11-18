# Báo cáo Day 08 — 2A202603187 - Đỗ Hoành Anh

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

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 base (`sam2.1_b.pt`): mask bám tốt vật rõ (img_004, img_032), ~8,9 giây/ảnh trên CPU, không cần token HuggingFace (SAM 3 cần đăng nhập), RAM đủ cho máy không GPU.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 không có lớp ba bánh nên bỏ sót hoặc gọi sai (xích lô img_032 thành `bicycle`, tuk-tuk img_004 / img_013 thành `car` khi chạy SAM, r05): hạ ngưỡng không cứu được. Sửa bằng YOLOE cho `three_wheeler` rồi người duyệt lại nhãn; YOLOE lại nhầm truck img_002 thành ba bánh và sót 1 xích lô ở img_032 nên vẫn phải duyệt tay.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm cao mà sai: img_032, YOLO26 gán `bicycle` 0.70 cho phần khung / bánh xích lô thay vì cả xe. Điểm thấp mà đúng: img_032, YOLOE bắt xích lô xa / bị che chỉ ở 0.27–0.48 (chiếc 0.92 là xích lô gần nhất). img_030: motorbike bắt được ở 0.25 nhưng box lỏng, mất ở 0.5.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh, 61 đối tượng) nên số đo máy trên gold là lạc quan; ngưỡng chỉ là ước lượng thô, nhất là lớp ít vật (bicycle 5, bus 1, truck 0). Chỉ motorbike (0.1) và three_wheeler (0.15) có ngưỡng riêng; ngưỡng thấp như 0.1 tăng recall nhưng giảm precision (67%) nên tăng việc xoá box thừa khi duyệt.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 512 | 82,3% | 622 box máy |
| edited | 37 | 5,9% | 622 box máy |
| deleted | 73 | 11,7% | 622 box máy |
| added | 60 | 9,9% | 609 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? `three_wheeler`: trong 41 box máy chỉ 10 giữ nguyên (24%), 10 sửa, 21 xoá, và thêm 4 box vẽ tay. Nguyên nhân: YOLO26 không có lớp ba bánh nên gán `car`/`bicycle`, YOLOE nhầm truck thành ba bánh (img_002) và sót xích lô (img_032) (xem r01–r05 trong `model_log.csv`). `motorbike` cũng đáng chú ý: 32 box bị xoá / 236 box máy (13,6%), nhiều ở ảnh đông xe (ví dụ img_018) vì ngưỡng thấp 0.1. Trong deleted + added có 11 ca đổi lớp (box đúng chỗ, sai lớp), nên số vật máy sót thật ≈ 60 − 11 = 49.
- So với gold: precision / recall của **máy** 65,2% (KTC 95%: 53–76%) / 84,3% (KTC 95%: 72–92%) (mẫu số: 51 đối tượng gold trong 4 ảnh). Con số lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 8/12 = 66,7% (KTC 95%: 39–86%, dưới ngưỡng 80%); mỗi loại bắt 2/3 (wrong_class, loose, extra, missing); loại nào lọt: 4 bẫy, mỗi loại lọt 1: img_035 `wrong_class` (motorbike nhỏ ở mép phải bị đổi thành bicycle), img_004 `loose` (box three_wheeler bị kéo to), img_027 `missing` (sót motorbike sát mép trái), img_005 `extra` (box truck ma sát mép trên); vì sao (bạn đã bỏ bước nào của checklist)? tôi chỉ chú ý các box lớn, dễ thấy và bỏ qua các box nhỏ (vật nhỏ, sát mép ảnh), nên không thực hiện kỹ bước 0 (quét toàn ảnh trước khi xem nhãn máy) và bước 1 (tìm vật bị sót); 3/4 bẫy lọt (img_035, img_027, img_005) là vật nhỏ ở mép ảnh. Quyết định: giữ nguyên số đo 8/12 để báo cáo trung thực, không duyệt lại để "chữa" con số (đã biết đáp án sẽ làm phép đo mất giá trị).
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 82,3% nhưng bắt bẫy chỉ 66,7%: bạn tin nhãn máy khá nhiều và lỗi lọt nằm ở chỗ mắt ít nhìn (mép ảnh, vật nhỏ, box nhìn hợp lý). Lỗi của máy không hiện trên điểm số (vật bị sót, box lỏng, nhầm lớp) nên chỉ lộ ra khi quét ảnh trước.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15 phút / 4 ảnh = 3,75 phút/ảnh vs duyệt 41 phút / 24 ảnh ≈ 1,71 phút/ảnh (nhanh hơn ~54%). Thiên lệch: gold gán trước nên đã quen cảnh; ảnh batch có thể dễ / khó khác ảnh gold; thời gian duyệt chưa gồm việc sửa lại ở ảnh bẫy lọt.
- Kết luận: **chỉ cho một số lớp**: dùng pre-label cho `person`, `motorbike`, `car` (accepted tổng 82,3% = 512/622, duyệt nhanh hơn ~54% so với gán tay); **không tin** `three_wheeler` (chỉ 10/41 box máy giữ nguyên, QC phát hiện 3 box thừa ở img_014) và cần duyệt kỹ mép ảnh, vật nhỏ (bắt bẫy 8/12 = 66,7%), dựa trên `review_stats.md`, `time_log.csv`, `qc_report_auto.md`.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Trước khi hiện nhãn máy, quét riêng **4 mép ảnh và vật nhỏ** của mỗi ảnh. Kiểm bằng bắt bẫy: mục tiêu ≥ 10/12 (lần này 8/12, 3 trong 4 bẫy lọt nằm ở mép / vật nhỏ).
  2. Mọi box `three_wheeler` có conf < 0.5 phải đối chiếu bằng cách **đếm bánh** trước khi giữ. Kiểm bằng QC: số lỗi `thua` ở lớp `three_wheeler` về 0 (lần này 3 box ở img_014).
  3. Sau lượt duyệt đầu, duyệt **vòng hai theo lớp** cho các lớp máy hay sai (`three_wheeler`, `truck`, `bicycle`), không chỉ duyệt theo từng ảnh. Kiểm bằng số lỗi `sai_lop` trong QC (lần này 1 lỗi ở img_035).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk) | SAM 2.1 base (`sam2.1_b.pt`) | 5 (3 pos + 2 neg) | Có lấn: mask dính người lái và khách ngồi trong xe (loang, vỡ hạt ở áo xanh / áo sọc) | Mask bám mui, thân, bánh tốt nhưng lấn sang người dù đã có 2 click `neg` trên người; vi phạm guideline 4.2 (người là vật riêng) | `masks/tuktuk_img013.jpg` |
| img_013 v2 (cùng tuk-tuk, thêm click) | SAM 2.1 base | 8 (3 pos + 5 neg) | Không: người lái và khách đã bị loại khỏi mask | Đánh đổi: thêm 3 click `neg` loại được người nhưng xe bị mất nhiều mảng (mái trái, ghế dài bên trái, cản sau, bánh sau lốm đốm, một mảng buồng lái); thêm `neg` làm SAM nghi ngờ cả vùng xe có màu giống người; ghế ngồi (băng ghế đen bên trái) và chỗ ngồi của người lái chưa được mask | `masks/tuktuk_img013_v2.jpg` |
| img_013 v3 (thêm click pos vào ghế, mái, bánh) | SAM 2.1 base | 12 (7 pos + 5 neg) | Không: người lái và khách vẫn bị loại | Tốt hơn v2: ghế dài bên trái, bánh sau và cản được mask lại; còn lốm đốm ở mái và vài mảng buồng lái. Cần 12 click (so với 5 ở lần đầu) để loại người mà vẫn giữ xe | `masks/tuktuk_img013_v3.jpg` |
| img_004 (tuk-tuk xanh cam) | SAM 2.1 base | 6 (4 pos + 2 neg) | Không có người; gồm cả ghế, thùng đỏ trước xe | Sạch: không lấn sang người đàn ông bên phải hay tuk-tuk xanh lá sát bên (nhờ 2 `neg`); mui và bánh trước còn hơi lỏng mép | `masks/tuktuk_img004.jpg` |
| img_032 (xích lô) | SAM 2.1 base | 5 (3 pos + 2 neg) | Không có người; gồm đệm ghế, mui hoa, hai bánh | Ôm khá sát, không dính người đứng cạnh; khung / nan bánh bị mờ ở phần mỏng | `masks/cyclo_img032.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Mục 4.2.4 (hai vật chạm nhau: mỗi vật một polygon, không chồng) giúp quyết định loại người khỏi mask xe ở img_013. Thiếu quy tắc cho **người ngồi trong xe ba bánh** (mask xe có gồm không khi người nằm trong khung?) và cho phần khung / nan mảnh (câu hỏi mở 5 về kính / khoảng trống); SAM 2.1 cần nhiều `neg` hơn khi người mặc màu giống thân xe.

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`: 3 `loi_may`, 2 `cau_hoi_mo`); quan trọng nhất: CAL-01 (tuk-tuk / xích lô luôn là `three_wheeler`, lớp máy sai nhiều nhất) và CAL-05 (gánh hàng không gán, lỗi QC bắt được).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: ví dụ máy gọi `car` / `bicycle` cho xe ba bánh; đếm bánh để phân biệt `truck` và `three_wheeler`; ranh giới `ignore_region` theo chiều cao (< 20 px, từ 5 vật); mask xe không gồm người nhưng gồm ghế và buồng lái; gánh hàng không gán.
- Nguyên nhân gốc: chủ yếu **máy** (YOLO26 không có lớp ba bánh, YOLOE ngưỡng thấp cho dương tính giả) và việc **tôi duyệt sót**; guideline mơ hồ chỉ ở mask người ngồi trong xe và ranh giới `ignore_region`.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42): img_003, 006, 014, 031, 035; 116 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 4/116 = 3,4% · theo **ảnh**: 2/5 = 40,0% (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: thừa 3/4 lỗi (img_014: 3 box `three_wheeler` #169, #170, #171 là gánh hàng và vật không thuộc lớp nào, do YOLOE ở ngưỡng 0.15); sai lớp 1/4 lỗi (img_035: #609 motorbike nhỏ bị gán bicycle, chính là bẫy `wrong_class` đã lọt ở mục 3). Lỗi `thua` lặp lại ở `three_wheeler`: lỗi hệ thống của recipe, không chỉ lỗi một ảnh.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 2 ảnh lỗi trong 5 ảnh mẫu, cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 77%; mẫu 5/24 ảnh không thể chứng minh ngưỡng 2%, và ngược lại cũng chưa đủ để nói chính xác tỉ lệ lỗi cả lô.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script chỉ đếm theo phiếu kiểm bằng mắt của chính tôi, không tự phát hiện lỗi; tôi vừa duyệt vừa kiểm nên có thể cùng bỏ sót với lúc duyệt (ví dụ vật bị sót mà máy cũng không vẽ). Ba ảnh còn lại (003, 006, 031) tôi không thấy lỗi bằng mắt, nhưng đó là đánh giá của một người, không phải bằng chứng là sạch.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release): không phải Hold vì guideline đủ rõ để gán (lỗi chủ yếu do máy và do tôi duyệt sót, không phải do quy tắc mơ hồ); là Rework vì ảnh lỗi 2/5 = 40% > 2%, lỗi `thua` lặp lại (3/4 lỗi), và bắt bẫy 8/12 = 66,7% < 80% nên lô duyệt chưa đáng tin.
Nếu Rework / Hold, việc cần làm tiếp: (1) xoá các box `three_wheeler` thừa ở img_014 (#169, #170, #171) và sửa #609 ở img_035 về `motorbike`; (2) rà lại mọi box `three_wheeler` của lô (26 đối tượng) vì ngưỡng 0.15 cho nhiều dương tính giả; (3) cân nhắc nâng ngưỡng `three_wheeler`; (4) export `labels/qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, ghi kết quả vòng sau.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code: giải thích hướng dẫn lab, chạy / sửa script và ghi `model_log.csv`, `time_log.csv`, soạn nháp báo cáo, `calibration_decisions.csv` và guideline v1.1, gợi ý toạ độ click SAM. Tôi tự gán gold, tự duyệt 24 ảnh trong CVAT, tự chạy các lệnh model và xem lại ảnh `viz/` / mask để xác nhận nhận xét.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không nhìn pre-label và không chạy model lên 4 ảnh gold trước khi gán xong. Có hỏi AI về guideline (cách dùng `ignore_region`, `can_review`) trong lúc gán, AI không gán hộ.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, Python 3.11.9, không có GPU (torch CPU); CVAT chạy bằng Docker. Vướng: lần đầu chạy YOLOE phải tải thêm gói CLIP và `mobileclip2_b.ts` (~242 MB) nên chờ lâu.
- Dùng cache `data/cache/` thay vì tự chạy? Không, đã tự chạy YOLO26 và YOLOE trên cả 36 ảnh (r06, r07).
- Lỗi / bug script đã gặp (nếu có): không có lỗi script; `validate_labels.py` chỉ cảnh báo 18 box cao dưới 20 px (img_018, 025, 027) chưa gộp thành `ignore_region`.
- Một điều bạn làm khác lần sau: duyệt kỹ mép ảnh và các box `three_wheeler` điểm thấp, và chọn ngưỡng `three_wheeler` cao hơn 0.15 để bớt dương tính giả (img_014).
