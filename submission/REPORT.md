# Báo cáo Day 08 — 2A202603171 - Đặng Minh Quân

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.15 | r01: recall 131 box ở 0.25 nhưng nhiều box chồng trên người xa; sweep cho recall 100% ở 0.15 trên gold (`qc/threshold_sweep.md`) |
| bicycle | YOLO26m | 0.2 | r02: 8 -> 3 box khi lên 0.5, mất xe đạp xa; 0.2 giữ recall 100%, precision 71% (5 vật gold) |
| motorbike | YOLO26m | mặc định 0.3 | r01/r02: **không ngưỡng nào ≤ 0.5 đạt recall 90%, trần 77%** (13 vật gold) — ngưỡng không cứu được, xem mục 2 |
| car | YOLO26m | 0.5 | r02: recall 90%, precision 69% (10 vật gold); 0.25 sinh 106 box car dưới ngưỡng trong lô |
| bus, truck | YOLO26m | mặc định 0.3 | 0 và 1 vật trong gold — quá ít mẫu, sweep in ra "KHÔNG chọn ngưỡng" |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | 0.15 | r03: YOLO26 không có lớp này (0 box trong 8 ảnh pilot) nên phải thêm một lượt open-vocab; 0.15 cho recall 100%, precision 78% (7 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):

```
python scripts\merge_prelabels.py work\prelabel\yoloe_all\pre.coco.json work\prelabel\y26_all\pre.coco.json --thr "person=0.15,bicycle=0.2,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work\prelabel\merged_review\pre.coco.json
```

- **Phiên bản SAM bạn chọn**: **SAM 2.1** (`sam2.1_b.pt`, 154 MB) làm đường chính — 3.78 s/ảnh trên RTX 3050 Laptop (CUDA 12.8), 1024 px, không cần token.
  **SAM 3 đã thử thật** (r05): cài `transformers 5.20.2` (bản 5.0+ có `Sam3Model`) và tải `facebook/sam3` (~3.3 GB, gated — tài khoản HF của tôi đã được duyệt quyền). Kết quả: **271.65 s/ảnh (nạp 32.8 s)**, tức **chậm gấp ~60 lần** SAM 2.1, và chất lượng **không tốt hơn**: `img_013` mask **53.2%** ảnh (nuốt cả khách ngồi sau mui, tệ hơn 44.9% của SAM 2.1), `img_009` **7.3%** vẫn cắt mất thùng rau — đúng lỗi CAL-03. Vì vậy vẫn giữ SAM 2.1 + cắt tay mask, SAM 3 chỉ dùng để đối chiếu.
- Lỗi **không sửa được bằng đổi ngưỡng**: tuk-tuk bị gọi `car`/`truck`. Đây là lỗi **không gian nhãn** (COCO không có `three_wheeler`), không phải lỗi điểm số: hạ ngưỡng chỉ sinh thêm box `car` sai, tăng ngưỡng thì mất luôn xe. Cách sửa: thêm lượt YOLOE với prompt `three_wheeler` (r03) rồi merge.
- Box **điểm cao mà sai**: `img_018` box `three_wheeler` (549,318) — YOLOE gán vào một **xe buýt**; 21/43 box `three_wheeler` của lô bị xoá khi duyệt (17 rổ hàng / mái hiên / biển hiệu, 1 xe buýt). **Điểm thấp mà đúng**: các tuk-tuk thật chỉ xuất hiện ở conf 0.15, ở 0.25–0.5 YOLOE không thấy.
- Giới hạn: ngưỡng được chọn **trên chính 4 ảnh gold** rồi cũng chấm trên 4 ảnh đó (in-sample, chỉ 61 vật) nên precision/recall của máy ở mục 2 là số **lạc quan**; đọc kèm khoảng tin cậy 95%.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 524 | 85.9% | 610 box máy |
| edited | 22 | 3.6% | 610 box máy |
| deleted | 63 | 10.3% | 610 box máy |
| added | 31 | 5.4% | 577 đối tượng sau duyệt |

- Lớp máy tệ nhất: **`three_wheeler`** — 25/43 box bị xoá (58%), chỉ 13 accepted, 5 edited, 5 added. Nguyên nhân: YOLOE `conf 0.05` bắn box vào rổ hàng, mái hiên, biển hiệu, xe buýt (`img_014`, `img_018`, `img_003`). Lớp tệ thứ hai là `person`: 30 box bị xoá (chủ yếu box chồng lên người khác trong đám đông `img_018`, `img_024`).
- So với gold: precision **67.9%** (KTC 95%: 57–77%), recall **86.9%** (KTC 95%: 76–93%) — TP 53 / FP 25 / FN 8 trên **61 đối tượng gold**. Trong 31 `added`, có 11 ca là **đổi lớp** (box đúng chỗ, sai lớp) → số vật máy **sót thật** ≈ 12, không phải 31.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **10/12 = 83.3%** (ngưỡng đạt ≥ 80%). Theo loại: `extra` 3/3, `wrong_class` 3/3, `loose` **2/3**, `missing` **2/3**.
- Lọt: `img_031` (loose) và `img_018` (missing). Vì sao: ở `img_031` tôi so cạnh box với **thân xe** nhưng bỏ qua việc rổ hàng hai bên làm box rộng gấp đôi — tức là bỏ bước "kiểm 4 cạnh theo kích thước" trong checklist. Ở `img_018` tôi quét ảnh nhưng chỉ nhìn **vùng giữa** (nơi có 32 person + 23 motorbike), không quét mép phải và bóng cây.
- Accepted cao (85.9%) **không** đi kèm bắt bẫy thấp (83.3%) — nghĩa là phần lớn box máy thật sự ổn, và 12 bẫy là loại lỗi hiếm chứ không phải lỗi điển hình. Ngược lại, điểm yếu thật của tôi là **loose/missing ở mép ảnh**, đúng hai loại lọt.

## 4. Máy có đáng dùng không?

- Thời gian (`time_log.csv`): gán tay **40 phút / 4 ảnh = 10 phút/ảnh**; duyệt có pre-label **<<... phút/ảnh>>** (24 ảnh, 610 box máy). Thiên lệch: tôi gán gold **trước** khi chạy model nên đã quen cảnh và biết cảnh nào khó → 10 phút/ảnh của gold là số **nhanh hơn thực tế** của người mới.
- Kết luận: **dùng được, nhưng chỉ cho `person`, `car`, `motorbike`** — 3 lớp này cho accepted 337+37+125 = 499/610 box máy (81.8%). **Không dùng cho `three_wheeler`**: 25/43 box phải xoá, tức là duyệt lại còn tốn hơn vẽ tay; và **không dùng YOLO26 cho ba bánh** vì lớp không tồn tại.
- Ba quy tắc giảm thiên kiến (kiểm được bằng số):
  1. **Ẩn nhãn trước, quét ảnh sau** — bật/tắt label bằng phím tắt, chỉ hiện box khi đã quét xong. Kiểm: số `missing` bắt được (2/3) và số `added` (31/577).
  2. **Kiểm 4 cạnh mọi box ở mép ảnh** trước khi Accepted. Kiểm: sau khi thêm bước này, `loose` bắt được 2/3 (trước đó 1/3).
  3. **Mọi box `three_wheeler` của máy phải nhìn thấy thân xe + 3 bánh** (CAL-02). Kiểm: 25 box rác bị xoá, trong đó 1 là xe buýt.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013` (tuk-tuk Thái) | SAM 2.1 (`sam2.1_b.pt`) | 2 dương + 1 âm | **Có người lái** (thân + chân), gồm cả mui | 44.9% ảnh; mask tràn qua người lái — vi phạm mục 4.2 (người là `person` riêng) | `tuktuk_img013.jpg` |
| `img_009` (xe ba gác chở rau) | SAM 2.1 | 1 dương + 1 âm | **Không** — chỉ được đầu xe máy | 2.8% ảnh: chỉ lấy phần nhựa đỏ phía trước, bỏ toàn bộ thùng hàng | `cargo_img009.jpg` |
| `img_010` (xích lô) | SAM 2.1 | 1 dương + 1 âm | Không | 2.3% ảnh: bám vào **dè sau** thành một dải mảnh, không phải xe | `cyclo_img010.jpg` |
| `img_009` (lặp, có box gợi ý) | SAM 2.1 | 1 dương + box | **Có hàng hoá** (cả bó rau) | 14.1% ảnh; box hint cứu được thân xe nhưng mask nuốt luôn hàng nhô ra | `cargo_img009_box.jpg` |
| `img_010` (lặp, có box gợi ý) | SAM 2.1 | 1 dương + box | Có mảng áo người lái / khách | 14.2% ảnh, **3 polygon** rời nhau (thân, mui, áo) | `cyclo_img010_box.jpg` |
| `img_013` (đối chiếu) | **SAM 3** (`facebook/sam3`, prompt `three_wheeler`) | 0 click (concept prompt) | Nhiều hơn SAM 2.1 | 53.2% ảnh, score 0.97; mask lan qua cả khách ngồi sau mui → **tệ hơn** SAM 2.1 (44.9%) | `work/prelabel/t_sam3/pre.coco.json` |
| `img_009` (đối chiếu) | **SAM 3** | 0 click | Không (vẫn cắt thùng rau) | 7.3% ảnh, score 0.65; chỉ ôm đầu xe máy, thùng rau vẫn ngoài mask — vi phạm CAL-03 y như SAM 2.1 | `work/prelabel/t_sam3/pre.coco.json` |

- Đối chiếu guideline mục 4.2: quy tắc "box/mask chỉ ôm thân xe, hàng nhô ra > 20% không tính" **giúp** giải thích lỗi của `cargo_img009_box`. Quy tắc **thiếu**: guideline chưa nói mask phải **trừ người** khi người nằm trong lòng mask (tuk-tuk `img_013`), và chưa nói mask nhiều polygon rời nhau thì lấy polygon nào → đã bổ sung ở v1.1 (CAL-03).
- Interactor SAM trong CVAT: không thử trong bài này (đã dùng script offline để có file lặp lại được); nói rõ đây là **tuỳ chọn** chưa làm.

## 6. Calibration (Part 6)

- Số quyết định: **5** (`qc/calibration_decisions.csv`), gồm 2 `loi_may`, 2 `cau_hoi_mo`, 1 `bay_lot`.
- Quan trọng nhất: **CAL-01** — YOLO26 gọi tuk-tuk là `car`/`truck` (không gian nhãn), sửa bằng **lượt YOLOE có prompt** chứ không bằng ngưỡng. CAL-02 mạnh thứ hai: YOLOE `conf 0.05` gán `three_wheeler` lên rổ hàng / biển hiệu / xe buýt (25/43 box bị xoá).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu 2 và câu 3 mục 6; 3 quy tắc kiểm box máy (xe 3 bánh, box rác YOLOE, box phình).
- Nguyên nhân gốc: **guideline mơ hồ** (câu 2: khối gần camera; câu 3: hàng nhô ra) **cộng với** lỗi **không gian nhãn** của máy — hai loại khác nhau, phải sửa bằng hai cách khác nhau.

## 7. QC và quyết định (Part 6)

- Mẫu: **3** ảnh (0 ảnh `can_review` + 3 ảnh ngẫu nhiên, seed 42), **57 đối tượng**; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt, nên đây là tự kiểm không độc lập).
- Lỗi theo **đối tượng**: **2/57 = 3.5%** · theo **ảnh**: **2/3 = 66.7%** (ngưỡng Release ≤ 2%).
- Top lỗi: `sai_vi_tri_kich_thuoc` 2/2 — `img_006` #85 (`bicycle` phủ cả giỏ hàng và chân người đẩy xe), `img_031` #513 (`bicycle` rộng gấp đôi thân xe, gồm 2 rổ trái cây — chính là bẫy `loose` chưa sửa).
- **Giới hạn mẫu nhỏ** (`qc_report_auto.md`): với 2/3 ảnh lỗi, **cận trên 95% (Wilson) là 94%** → ngưỡng 2% **không thể chứng minh** bằng mẫu 3 ảnh; con số 3.5% theo đối tượng cũng chỉ là ước lượng thô.
- Script kiểm được / không kiểm được: `validate_labels.py` bắt được cấu trúc, lớp, toạ độ ngoài ảnh, box < 20 px (**WARN**, 16 box). Script **không** kiểm được "box có ôm sát không", "có bỏ sót vật không", hay "người duyệt có cùng mù với máy không" (ví dụ tôi Accepted 524 box máy mà không soát lại từng cạnh).

**Quyết định: Rework**, ngày 2026-11-18. Lý do (theo cây Hold → Rework → Release): bắt bẫy 10/12 = 83.3% **đạt**, nhưng lỗi theo ảnh 2/3 = 66.7% **vượt xa** ngưỡng 2% và cả hai lỗi đều là lỗi lặp (`sai_vi_tri_kich_thuoc`) chứ không phải ca hiếm → không thể Release. Chọn **Rework** chứ không **Hold** vì guideline không còn mơ hồ ở hai ca này (đã chốt ở CAL-03/CAL-05): đây là lỗi thực thi, sửa được.
Nếu Rework: (1) kéo lại #85 và #513 về đúng thân xe; (2) khôi phục xe ba gác ở `img_009` thành `three_wheeler`; (3) export `labels/qc_fixed.json` nếu nhãn đổi, chạy lại `build_release.py` và `qc_sample.py --force`, ghi vòng 2 vào báo cáo.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: có. Dùng cho (a) đọc `GUIDE/RULES/CHECKPOINTS` và tóm tắt luồng, (b) viết và chạy các lệnh Part 2–5 (`prelabel.py`, `threshold_sweep.py`, `merge_prelabels.py`, `sam2_points.py`), (c) soát số và viết nháp mục 1–7 của báo cáo này. Kiểm tra bằng cách đối chiếu từng con số với file gốc (`qc/review_stats.json`, `qc/threshold_sweep.csv`, `qc/qc_report_auto.md`) và tự chạy lại lệnh để xem output. Phần **gán nhãn** (gold, duyệt 24 ảnh, quyết định calibration) do tôi làm bằng tay.
- Gold có bị nhiễm không: **không**. 4 ảnh gold gán tay trong CVAT trước khi chạy model lần đầu, export `gold_manual.json` và commit `bed876f` **trước** mọi lệnh `prelabel.py`; không xem pre-label trước khi gán xong.
- **Máy / GPU / CVAT**: Windows, RTX 3050 Laptop GPU (CUDA 12.8, torch 2.10.0+cu128), CVAT **v2.74.1** (Docker, `localhost:8080`).
  **Vướng mắc cài đặt (theo thứ tự gặp):**
  1. Python hệ thống 3.12 thiếu 7 gói (`check_env.py` FAIL numpy, cv2, torch, ultralytics...) → tạo `venv` Python 3.10.6, cài lại toàn bộ.
  2. **Phân biệt hai thứ dễ nhầm**: gói `sam3 0.1.0` (repo Meta, cài editable) **đã có trong venv từ 11/11**, nhưng `scripts/prelabel.py --model sam3` không dùng gói đó — nó import `Sam3Model`/`Sam3Processor` **từ `transformers`**, và `transformers` thì **chưa có** ở cả venv lẫn Python hệ thống 3.12 (`ModuleNotFoundError`). Vì vậy 3 backend `gdino`, `owlv2`, `sam3` đều không import được.
  3. Ngày **18/11 09:10** đã cài `transformers 5.20.2` (bản 5.0+ có `Sam3Model`) và tải `facebook/sam3` (3.3 GB, gated; tài khoản HF `Darksider1403` đã được duyệt) → cả 3 backend chạy được, `check_env.py` giờ **PASS** toàn bộ.
  4. Chạy thử SAM 3 thật ở **r05**: 271.65 s/ảnh (nạp 32.8 s), chậm ~60 lần SAM 2.1, mask không tốt hơn → vẫn giữ SAM 2.1 làm đường chính (chi tiết ở mục 1 và mục 5).
- Dùng cache `data/cache/`: **không** — mọi file `pre.coco.json` trong `work/prelabel/` đều do máy này chạy ra (log thời gian trong `model_log.csv`).
- Lỗi / bug script đã gặp: (1) `prelabel.py` **bỏ im lặng** lớp COCO không có trong taxonomy (umbrella 57, banana 105, chair 29...) — phải đọc dòng "Tên thô KHÔNG có trong ánh xạ"; (2) `sam2_points.py` với 1 click trả mask rỗng nếu click vào khoảng trống trong xe → phải click vào thân xe hoặc thêm `--box`.
- Một điều làm khác lần sau: **duyệt theo thứ tự mép ảnh trước, giữa ảnh sau**, và chốt "kiểm 4 cạnh" thành một bước bắt buộc trước khi Accepted — hai bẫy lọt của tôi đều ở mép ảnh.
