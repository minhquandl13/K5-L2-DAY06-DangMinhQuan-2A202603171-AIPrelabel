# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 610; đối tượng sau duyệt: 577; ngưỡng accepted: IoU >= 0.9
- Đã **loại 27 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 524 | 524/610 = 85.9% | 610 box máy |
| edited (sửa) | 22 | 22/610 = 3.6% | 610 box máy |
| deleted (xoá) | 63 | 63/610 = 10.3% | 610 box máy |
| added (vẽ thêm) | 31 | 31/577 = 5.4% | 577 đối tượng sau duyệt |

Trong số deleted + added, **11 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 20.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 337 | 6 | 30 | 9 |
| bicycle | 6 | 2 | 0 | 4 |
| motorbike | 125 | 8 | 5 | 7 |
| car | 37 | 1 | 1 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 3 | 0 | 2 | 4 |
| three_wheeler | 13 | 5 | 25 | 5 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 61 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 53 | 25 | 8 | 67.9% (KTC 95%: 57–77%) | 86.9% (KTC 95%: 76–93%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/12 = 83.3% (KTC 95%: 55–95%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - wrong_class: 3/3
  - loose: 2/3
  - missing: 2/3
  - lọt: img_018.jpg (missing), img_031.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
