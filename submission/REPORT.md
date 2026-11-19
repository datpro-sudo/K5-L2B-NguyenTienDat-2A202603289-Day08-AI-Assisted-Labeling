# Báo cáo Day 08 — 2A202603289 - Nguyen Tien Dat

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.35 | `r06`; 15 vật gold trên 4 ảnh. Ngưỡng cao nhất ≤ 0.5 vẫn đạt recall mục tiêu: recall 93%, precision 48%. |
| bicycle | YOLO26m | mặc định 0.3 | `r06`; chỉ 4 vật gold (< 5), không chọn ngưỡng riêng vì mẫu quá ít. |
| motorbike | YOLO26m | 0.35 | `r06`; 9 vật gold. Ngưỡng cao nhất ≤ 0.5 còn đạt recall mục tiêu: recall 100%, precision 82%. |
| car | YOLO26m | mặc định 0.3 | `r06`; 6 vật gold, nhưng không ngưỡng ≤ 0.5 đạt recall 90% (tối đa 83%), nên không ép chọn ngưỡng riêng. |
| bus | YOLO26m | mặc định 0.3 | `r06`; không có vật bus trong 4 ảnh gold, không đủ dữ liệu để chọn ngưỡng. |
| truck | YOLO26m | mặc định 0.3 | `r06`; không có vật truck trong 4 ảnh gold, không đủ dữ liệu để chọn ngưỡng. |
| three_wheeler | YOLOE-26s, prompt từ taxonomy | 0.15 | `r07`; 6 vật gold. Ngưỡng cao nhất ≤ 0.5 đạt recall mục tiêu: recall 100%, precision 67%; pilot `r04` cho thấy cần review box. |

- Ngưỡng chuyển sang Part 4: `--thr "person=0.35,motorbike=0.35,three_wheeler=0.15"`; các lớp không liệt kê dùng `--conf 0.3`. Chưa chạy lệnh merge trong Part 3.
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`), vì weights đã có sẵn và chạy được trên RTX 3050; ba lượt click mất trung bình 7.57 giây/ảnh. Đây là mask nháp: ở `img_013.jpg` mask vẫn dính người lái dù có click âm.
- Lỗi **không sửa được bằng đổi ngưỡng:** YOLO26 không có lớp tuk-tuk/`three_wheeler` trong taxonomy COCO nên có thể gọi nhầm thành `car` hoặc `truck`. Dùng YOLOE làm nguồn ứng viên rồi đối chiếu định nghĩa và sửa lớp bằng mắt; đổi confidence không đổi được tên lớp.
- Box **điểm cao mà sai / điểm thấp mà sai:** YOLOE `img_002.jpg` có box score 0.9267 nhưng box lỏng/chồng lên vùng xe khác (`r04`); ở `img_011.jpg`, score 0.2834 đặt nhầm vào mép xe xanh tiền cảnh và bỏ xe ba bánh chở hàng màu cam (`r04`). Điểm confidence không thay thế kiểm tra vị trí và đối tượng.
- Giới hạn: chỉ có 4 ảnh gold; chọn ngưỡng và tính precision/recall trên cùng dữ liệu nên số đo trên gold lạc quan. Ngưỡng chỉ là ước lượng thô, đặc biệt với lớp ít vật hoặc không có mẫu như bus/truck.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 435 | 88.8% | 490 box máy trên 24 ảnh batch, sau khi loại vùng bẫy |
| edited | 9 | 1.8% | 490 box máy trên 24 ảnh batch, sau khi loại vùng bẫy |
| deleted | 46 | 9.4% | 490 box máy trên 24 ảnh batch, sau khi loại vùng bẫy |
| added | 20 | 4.3% | 464 đối tượng sau duyệt trên 24 ảnh batch, sau khi loại vùng bẫy |

- Lớp máy tệ nhất: `three_wheeler` có 21 deleted, 3 edited, 3 added; cao nhất về số box bị xóa. `img_013.jpg` và `img_014.jpg` mỗi ảnh có 3 box `three_wheeler` không ghép được với box cuối (IoU/class); log tổng hợp không phân biệt được box sai lớp với box sai vị trí. Pilot `r04` cũng ghi nhận YOLOE có box lỏng/nhầm và bỏ sót xe ba bánh chở hàng ở `img_011.jpg`.
- So với gold: pre-label đạt precision 56.9% (37/65 dự đoán) và recall 92.5% (37/40 vật gold), trên 4 ảnh. Đây là số đo in-sample, lạc quan.
- `added=20` không phải tổng số máy bỏ sót: có 7 cặp deleted+added thực chất là đổi lớp, nên khoảng 13 là vật được thêm thật. Vật mà cả máy lẫn người review cùng bỏ sót sẽ không xuất hiện trong added; cần đối chiếu gold/QC để phát hiện.
- Kiểm tra nhãn: 24 ảnh, 477 đối tượng, không có lỗi cấu trúc; còn 1 cảnh báo: box `car` cao 17.2 px trong `img_027.jpg`, dưới ngưỡng 20 px của guideline.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Export trước khi AI sửa bắt 5/12 bẫy (41.7%, dưới ngưỡng 80%): extra 3/3, missing 1/3, wrong_class 1/3, loose 0/3. Bị lọt: missing `img_035.jpg`, `img_013.jpg`; loose `img_034.jpg`, `img_028.jpg`, `img_033.jpg`; wrong_class `img_026.jpg`, `img_031.jpg`. Đây là điểm đo ban đầu; xem `qc/review_stats_first_pass.md`.
- Theo yêu cầu người học, AI đã dùng key để sửa đúng 7 bẫy lọt: khôi phục 2 box person bị thiếu, chỉnh 3 box lỏng về box gốc, sửa 2 nhãn bicycle thành person. File cuối chạy lại cho kết quả 12/12, nhưng đây là **kết quả sau khi biết key và sửa có hướng dẫn**, không phải lượt review mù hay bằng chứng người review tự bắt đủ bẫy.
- Kết quả cho thấy accepted cao không bảo đảm bắt lỗi cài chủ đích: lượt đầu giữ nhiều box bình thường nhưng lọt cả ba bẫy box lỏng. Sau lần AI sửa, cần kiểm tra thủ công box `car` cao 17.2 px ở `img_027.jpg`; không nên coi việc đạt 12/12 sau sửa là thay thế QC độc lập.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 5 phút/ảnh (20/4) vs lần review đã ghi 0.83 phút/ảnh (20/24). Thời gian lần review lại chưa được cộng vì chưa có số phút thực tế. So sánh chỉ mang tính tham khảo: ảnh gold và batch khác độ khó, và người review đã quen kiểu cảnh sau khi gán gold.
- Kết luận: Có dùng pre-label làm điểm xuất phát, nhưng không tự động chấp nhận. Ở bản review trước AI sửa, 435/490 box được giữ nguyên; có 46 deleted và 20 added. `three_wheeler` cần review đặc biệt (21 deleted). AI chỉ sửa các bẫy đã biết; không rà lại toàn bộ nội dung ảnh hoặc thay thế một lượt review mù.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy, quét và ghi số lượng ước tính theo lớp trước khi hiện nhãn ở cả 24/24 ảnh.
  2. Dùng checklist theo đúng thứ tự bỏ sót → thừa → sai lớp → box lỏng cho từng ảnh; đánh dấu hoàn tất đủ 24/24 ảnh.
  3. Rà theo góc phần tư ở 6 ảnh dày xe máy (005, 018, 021, 024, 028, 029); cụm vật dưới 20 px dùng `ignore_region`, nghỉ 3–5 phút sau khoảng 10 ảnh.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` (tuk-tuk có người lái) | SAM 2.1 base (`sam2.1_b.pt`) | 9 (4 dương, 5 âm) | Có: mask vẫn gồm phần lớn người lái và một phần người trong khoang; không có hàng hoá riêng. | Click âm chưa tách người khỏi xe; một số nan/chi tiết khung bị lẫn với vùng nền trong khoang. Mask phủ 38.2% ảnh. | `tuktuk_img013_sam2.jpg` |
| `img_032.jpg` (xích lô tiền cảnh) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Không thấy người/hàng hoá trong mask mục tiêu; click âm loại được người đứng cạnh. | Mask phủ thân/ghế và hai bánh; các khe hở lớn của khung vẫn cần rà theo quy tắc lỗ. Mask phủ 8.5% ảnh. | `cyclo_img032_sam2.jpg` |
| `img_004.jpg` (tuk-tuk xanh dương bị xe khác che một phần) | SAM 2.1 base (`sam2.1_b.pt`) | 4 (2 dương, 2 âm) | Không gồm người đàn ông đứng cạnh; không thấy hàng hoá. | Biên ở chỗ hai xe sát nhau còn gần đúng, nhưng mask dính ít nền/bóng dưới bánh trước. Mask phủ 13.1% ảnh. | `occluded_tuktuk_img004_sam2.jpg` |

- Chọn SAM 2.1 base vì weights đã có sẵn và cho mask click nhanh trên ba ảnh (~7.57 giây/ảnh trung bình; `model_log.csv`, r08). Click dương trên mui/thân/bánh giúp mở rộng mask; click âm giúp loại người cạnh xích lô và xe lân cận. Trường hợp `img_013.jpg` cho thấy nhiều click âm vẫn không bảo đảm tách người lái: cần xem mask, thêm điểm hoặc sửa thủ công, không coi đầu ra là nhãn cuối.
- Đối chiếu guideline mục 4.2: mask nên bám phần nhìn thấy, không lấy bóng đổ/nền; các khoảng rỗng lớn xuyên khung cần để ngoài polygon, còn lỗ nhỏ dưới khoảng 5% có thể bỏ qua. Ảnh `img_013.jpg` còn vi phạm tách riêng người lái/người ngồi; `img_004.jpg` cần bỏ phần nền dưới bánh và rà biên xe đang bị che. Đây là overlay thử nghiệm; chưa đo sai lệch biên theo ngưỡng 3 px.
- Interactor SAM trong CVAT: chưa thử; các kết quả trên chạy offline bằng `sam2_points.py`, không phải review hay sửa mask trong CVAT.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`qc/calibration_decisions.csv`); quan trọng nhất: `CAL-03` loại box bộ phận trùng trên cùng xe, và `CAL-04` ghi rõ lượt bắt bẫy 5/12 ban đầu là phép đo độc lập còn lượt 12/12 sau khi biết key thì không.
- Quy tắc mới trong `guidelines/guideline_v1.1_calibrated.md`: một vật một instance; tách giá/giỏ cố định khỏi hàng rời; kính thuộc mask xe nhưng người vẫn là instance riêng; quét ảnh theo góc phần tư trước khi xem nhãn máy và kiểm theo checklist sau đó.
- Nguyên nhân gốc: vừa có lỗi máy (box trùng/lỏng), vừa có thiên kiến tự động hóa ở lượt review đầu; câu hỏi về hàng rời và kính cần được diễn giải cụ thể hơn. Các câu hỏi chưa có ví dụ đủ rõ vẫn để mở và cần `can_review`.

## 7. QC và quyết định (Part 6)

- Mẫu: 5 ảnh (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42); kiểm bằng Copilot visual QC trên ảnh overlay, không phải xác nhận độc lập của học viên.
- Lỗi theo **đối tượng**: 3/98 = 3.1% · theo **ảnh**: 3/5 = 60.0% (ngưỡng Release ≤ 2%).
- Lỗi quan sát: `thua` 2 (`img_003.jpg` #13 là box mái trùng xe #18; `img_035.jpg` #469 là box bộ phận cạnh #468); `sai_vi_tri_kich_thuoc` 1 (`img_031.jpg` #409 cắt mất phần trên của xe/giá chở hàng).
- **Giới hạn mẫu nhỏ:** `qc_report_auto.md` cho cận trên Wilson 95% là 88%; 5 ảnh không đủ chứng minh ngưỡng lỗi thật ≤ 2%. QC bằng AI cũng không thay thế xác nhận trực tiếp của người nộp.
- Script kiểm cấu trúc, lớp, số lượng, matching và thống kê; không tự phát hiện vật bị cả máy và người bỏ sót, cũng không xác nhận box có ôm sát thực tế. Ảnh “không lỗi rõ” trong mẫu chỉ là kết quả spot-check, không chứng minh recall đầy đủ.

**Quyết định: REWORK**, ngày 2026-11-18. Không có lỗi dữ liệu khiến phải Hold, nhưng 3/5 ảnh mẫu có lỗi, vượt ngưỡng 2%; thêm nữa lượt review mù ban đầu chỉ bắt 5/12 bẫy (41.7%). Số 12/12 hiện tại là sau khi dùng key sửa bẫy, không phải kết quả độc lập.
Việc cần làm tiếp: sửa ba đối tượng nêu trên trong CVAT, export phần sửa thành `labels/qc_fixed.json`, chạy lại `build_release.py` và QC với mẫu mới; người nộp cần tự xác nhận các overlay trước khi quyết định Release.

## 8. Ghi chú cá nhân

- **Dùng AI:** Copilot hỗ trợ chạy model, tạo mask và điền bản nháp QC/calibration; ảnh overlay được xem trực tiếp.
- Gold: theo trình tự đã ghi nhận, gán `gold_manual` hoàn tất trước các lượt model; không thấy bằng chứng pre-label được dùng để tạo gold.
- Máy / GPU / CVAT: Windows, NVIDIA RTX 3050 Laptop GPU, CUDA (PyTorch 2.6.0+cu124), CVAT Docker v2.81.0. Lỗi thiếu `ultralytics` đã xử lý bằng cài requirements vào venv.
- Cache `data/cache/`: không ghi nhận sử dụng; các lượt model chạy từ weights local.
- Lỗi / bug gặp: thiếu `ultralytics` khi chạy ngoài đúng venv; lệnh kích hoạt PowerShell từng bị dính vào lệnh Python do thiếu dấu xuống dòng/dấu phân cách.
- Lần sau: giữ bản review mù riêng, không dùng key để sửa trước khi ghi kết quả bắt bẫy; làm QC trực quan độc lập trước khi chốt Release.
