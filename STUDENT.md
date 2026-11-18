# Thông tin học viên

- Họ và tên: Đặng Minh Quân
- MSHV: 2A202603171
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: minhquandl13
- Repo bài nộp (URL): https://github.com/minhquandl13/K5-L2-DAY06-DangMinhQuan-2A202603171-AIPrelabel
- Commit chốt bài (hash): (điền khi nộp)
- Máy: Windows 11, NVIDIA RTX 3050 Laptop GPU (CUDA 12.8, torch 2.10.0+cu128)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: **524 / 22 / 63** (mẫu số: 610 box máy); added: **31** (mẫu số: 577 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: **67.9% / 86.9%** (53 TP, 25 FP, 8 FN trên 61 đối tượng gold)
- Bắt bẫy: **10/12 = 83.3%** (extra 3/3, wrong_class 3/3, loose 2/3, missing 2/3)
- Thời gian: gán tay **10 phút/ảnh** (40 phút / 4 ảnh); duyệt có pre-label **<<... phút/ảnh>>** (24 ảnh)
- Lỗi QC: **66.7% theo ảnh (2/3)**, **3.5% theo đối tượng (2/57)** — cận trên 95% (Wilson) 94%
- **Quyết định:** Rework — 2/3 ảnh có lỗi (ngưỡng Release ≤ 2%) dù bắt bẫy đạt 83.3%; hai lỗi đều là `sai_vi_tri_kich_thuoc` lặp lại (`img_006` #85, `img_031` #513).
