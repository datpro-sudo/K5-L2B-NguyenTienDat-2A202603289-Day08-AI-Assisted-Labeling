# Thông tin học viên

- Họ và tên: Nguyen Tien Dat
- MSHV: 2A202603289
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: datpro-sudo
- Repo bài nộp (URL): https://github.com/datpro-sudo/K5-L2B-NguyenTienDat-2A202603289-Day08-AI-Assisted-Labeling
- Commit chốt bài (hash): Chưa commit Part 6
- Máy: Windows, NVIDIA RTX 3050 Laptop GPU (CUDA; PyTorch 2.6.0+cu124)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 435 / 9 / 46 (mẫu số: 490 box máy); added: 20 (mẫu số: 464 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 56.9% (37/65 dự đoán) / 92.5% (37/40 vật gold), 4 ảnh
- Bắt bẫy: 12/12 = 100% sau sửa có dùng key; lượt đầu 5/12 = 41.7% (đo độc lập ban đầu)
- Thời gian: gán tay 5 phút/ảnh (20/4); duyệt có pre-label 0.83 phút/ảnh (20/24)
- Lỗi QC: 60.0% theo ảnh (3/5), 3.1% theo đối tượng (3/98)
- **Quyết định:** REWORK — lỗi ở 3/5 ảnh mẫu vượt ngưỡng ảnh lỗi 2%; cần sửa và QC lại.
