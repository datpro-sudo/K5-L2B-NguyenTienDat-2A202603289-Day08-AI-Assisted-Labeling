# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 490; đối tượng sau duyệt: 464; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 435 | 435/490 = 88.8% | 490 box máy |
| edited (sửa) | 9 | 9/490 = 1.8% | 490 box máy |
| deleted (xoá) | 46 | 46/490 = 9.4% | 490 box máy |
| added (vẽ thêm) | 20 | 20/464 = 4.3% | 464 đối tượng sau duyệt |

Trong số deleted + added, **7 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 13.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 242 | 1 | 5 | 4 |
| bicycle | 4 | 0 | 3 | 2 |
| motorbike | 113 | 5 | 10 | 7 |
| car | 50 | 0 | 5 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 4 | 0 | 2 | 2 |
| three_wheeler | 19 | 3 | 21 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 40 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 37 | 28 | 3 | 56.9% (KTC 95%: 45–68%) | 92.5% (KTC 95%: 80–97%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - missing: 3/3
  - loose: 3/3
  - wrong_class: 3/3
  - extra: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
