# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603171 - Đặng Minh Quân

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 và
> thêm quy tắc mới rút ra từ lỗi của máy / lỗi của người gán. Mỗi ca khó có ảnh ví dụ và khớp một dòng
> trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc **gần camera**: từng xe hay `ignore_region`? | Nếu ≥5 vật sát nhau **và** chiều cao < 20 px: một `ignore_region`. Vật cao ≥ 20 px vẫn vẽ từng box, kể cả khi đứng trong khối dày. | `img_018`, `img_005` | CAL-04 |
| 3 | Hàng nhô ra (rổ trái cây trên xe đạp, xe ba gác chở rau): box/mask có gồm hàng không? | Box và mask chỉ ôm **thân xe**. Hàng nhô ra quá ~20% chiều dài thân xe không tính vào box/mask; phần này cắt tay sau khi SAM trả mask. | `img_031`, `img_009` | CAL-03 |

Chưa chốt: câu 1 (xe đạp điện nhỏ) — không gặp ca rõ trong 24 ảnh batch nên để nguyên v1.0; câu 4, 5, 6, 7 để nguyên.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002`, `img_011`, `img_032` | Tuk-tuk đứng rõ, 3 bánh, có mui | **YOLO26** (COCO) gọi là `car`/`truck`; **YOLOE** với prompt `three_wheeler` tìm đúng cùng vị trí (4 / 1 / 3 box) | Xe 3 bánh có mui luôn là `three_wheeler`, không dùng `car`/`truck` | Mục 4.2: `three_wheeler` = tuk-tuk + xích lô + xe ba gác; `car`/`truck` **không** bao gồm xe 3 bánh |
| `img_014`, `img_018`, `img_003` | Rổ hàng, mái hiên, biển hiệu, và cả xe buýt | **YOLOE** `conf 0.05` gán `three_wheeler` lên các vật này (21/43 box `three_wheeler` của lô bị xoá khi duyệt; `img_018` box 549,318 là xe buýt) | Chỉ giữ khi thấy **thân xe + 3 bánh**; box trên rổ hàng / biển hiệu phải xoá, không đổi lớp | Mục 4.1.6: trước khi Accepted một box `three_wheeler` của máy, phải kiểm bằng mắt là có thân xe và 3 bánh |
| `img_031` | Xe đạp bán trái cây, rổ treo hai bên | Box máy phủ cả rổ, rộng hơn thân xe ~2 lần (bẫy `loose` để lọt) | Kéo 4 cạnh về đúng thân xe trước khi Accepted | Mục 4.1.5: box ôm sát vật; lệch kích thước >10% diện tích phải sửa trước khi Accepted |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2 và câu 3 mục 6; thêm 3 quy tắc kiểm box máy (xe 3 bánh, box rác của YOLOE, box phình) | CAL-01 (bổ sung 3 bánh có mui), CAL-02 (YOLOE gán sai trên rổ hàng), CAL-03 (hàng nhô ra), CAL-04 (khối gần camera), CAL-05 (bẫy loose) |
