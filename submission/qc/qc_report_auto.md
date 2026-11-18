# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 3 ảnh (0 ca khó + 3 ngẫu nhiên), 57 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 2/57 = 3.5% | 57 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/3 = 66.7% | 3 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 524/610 = 85.9% / 22/610 = 3.6% / 63/610 = 10.3% | 610 box máy |
| Máy: added | 31/577 = 5.4% | 577 đối tượng sau duyệt |
| Bắt lỗi bẫy | 10/12 = 83.3% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| sai_vi_tri_kich_thuoc | 2 | img_006.jpg, img_031.jpg |

## Giới hạn thống kê
- 2 ảnh lỗi trong 3 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **94%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 66.7% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
