# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 367; đối tượng sau duyệt: 401; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 311 | 311/367 = 84.7% | 367 box máy |
| edited (sửa) | 16 | 16/367 = 4.4% | 367 box máy |
| deleted (xoá) | 40 | 40/367 = 10.9% | 367 box máy |
| added (vẽ thêm) | 74 | 74/401 = 18.5% | 401 đối tượng sau duyệt |

Trong số deleted + added, **14 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 60.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 148 | 1 | 2 | 38 |
| bicycle | 4 | 2 | 0 | 4 |
| motorbike | 110 | 5 | 8 | 13 |
| car | 35 | 1 | 3 | 11 |
| bus | 2 | 0 | 1 | 3 |
| truck | 2 | 0 | 2 | 2 |
| three_wheeler | 10 | 7 | 24 | 3 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 31 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 29 | 21 | 2 | 58.0% (KTC 95%: 44–71%) | 93.5% (KTC 95%: 79–98%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 10/11 = 90.9% (KTC 95%: 62–98%) (ngưỡng đạt: 80%). Chỉ 11 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - missing: 2/2
  - loose: 2/3
  - extra: 3/3
  - lọt: img_018.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
