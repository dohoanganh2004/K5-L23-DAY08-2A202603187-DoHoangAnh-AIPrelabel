# Thông tin học viên

- Họ và tên: Đỗ Hoành Anh
- MSHV: 2A202603187
- Lớp : L2A
- Tài khoản GitHub: https://github.com/dohoanganh2004
- Repo bài nộp (URL): https://github.com/dohoanganh2004/K5-L23-DAY08-2A202603187-DoHoangAnh-AIPrelabel.git
- Commit chốt bài (hash): 
- Máy: Windows 11, không có GPU (CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 512 / 37 / 73 (mẫu số: 622 box máy); added: 60 (mẫu số: 609 đối tượng)
- So với gold: precision / recall của **máy** 65,2% / 84,3% (51 đối tượng gold)
- Bắt bẫy: 8/12 = 66,7%
- Thời gian: gán tay 3,75 phút/ảnh (15/4); duyệt có pre-label 1,71 phút/ảnh (41/24)
- Lỗi QC: 40,0% theo ảnh (2/5), 3,4% theo đối tượng (4/116)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 40% > 2%, lỗi `thua` lặp lại ở `three_wheeler`, bắt bẫy 66,7% < 80%
