# Label contract — Bổ sung v1.1 — 2A202603289 - Nguyen Tien Dat

> Chỉ ghi phần làm rõ hoặc thay đổi so với `guideline_v1.0_label_contract.md`.

## 1. Câu hỏi mở đã chốt và còn mở

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 3 | Hàng rời nhô khỏi xe đạp chở hàng có thuộc box/mask xe không? | Không. Khung xe và giá/giỏ gắn cố định thuộc xe; hàng rời nhô khỏi xe quá 20% chiều dài xe để ngoài box/mask. Không tạo lớp riêng cho hàng hóa. | `img_031.jpg` | CAL-01 |
| 5 | Kính chắn gió/cửa sổ có thuộc mask xe khi nhìn thấy người phía sau không? | Kính thuộc mask xe; người nhìn thấy qua kính hoặc cạnh hở vẫn là `person` riêng, không nhập người vào mask xe. | `img_013.jpg` | CAL-02 |

Câu 1 (xe đạp điện), 2 (cụm xe máy gần camera), 4 (ngưỡng phần người nhìn thấy), 6 (box chồng trong ảnh đêm) và 7 (confidence tối thiểu) chưa được chốt: lô hiện tại không có đủ ví dụ phân biệt để đặt quy tắc đáng tin. Với ca gặp ngoài guideline, đánh dấu `can_review`; không suy diễn từ confidence.

## 2. Ca khó mới

| Ảnh | Tình huống | Máy làm gì | Quyết định | Bổ sung |
|---|---|---|---|---|
| `img_003.jpg` | Hai box `three_wheeler` #13 và #18 cùng phủ một xe; #13 chỉ phủ mái, #18 bao xe nhìn thấy. | Pre-label tạo box bộ phận trùng với box toàn xe. | Giữ một instance/xe; bỏ box trùng hoặc box chỉ phủ bộ phận, chọn box ôm phần xe nhìn thấy. Confidence không thay thế kiểm tra trùng. | Mục 4.1 |
| `img_031.jpg` | Xe đạp bán hàng có giỏ cố định và hàng rời nhô cao/rộng. | Box #409 chưa bao hết phần trên của xe/giá giỏ, trong khi hàng rời làm biên khó phân biệt. | Áp dụng CAL-01: phân biệt cấu kiện gắn cố định với hàng rời; rà lại box #409 trước khi phát hành. | Mục 4.1.5, 4.2 |
| `img_013.jpg` | SAM click trên tuk-tuk vẫn đưa người lái/người trong khoang vào mask xe. | Click âm không tách sạch người khỏi xe. | Người và xe là hai đối tượng riêng; mask thử nghiệm dính người phải được click bổ sung hoặc sửa tay, không chấp nhận tự động. | Mục 4.2 |
| `img_034.jpg`, `img_028.jpg`, `img_033.jpg` | Lượt review đầu lọt cả ba bẫy box lỏng. | Người review giữ box máy dù biên còn lỏng. | Quét ảnh trước khi hiện nhãn; sau đó rà từng ảnh theo thứ tự bỏ sót → thừa/trùng → sai lớp → box lỏng. | Quy trình review |

### Quy trình review pre-label bổ sung

Trước khi hiện nhãn máy, quét ảnh theo góc phần tư và ghi ước lượng đối tượng theo lớp. Sau đó đối chiếu lần lượt bỏ sót, thừa/trùng, sai lớp và box lỏng; ghi một kết quả QC cho từng ảnh. Kết quả bắt bẫy 12/12 sau khi đã mở key không được xem là lượt review mù hay bằng chứng độc lập.

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Làm rõ hàng rời/giỏ cố định, kính và người trong xe, box trùng; bổ sung quy trình review pre-label. | CAL-01–CAL-04; ví dụ `img_031`, `img_013`, `img_003`, `img_034` |
