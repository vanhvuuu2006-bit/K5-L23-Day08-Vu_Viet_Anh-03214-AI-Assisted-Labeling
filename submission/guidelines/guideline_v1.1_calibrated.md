# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603214 - Vũ Việt Anh

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Người lái và hành khách ngồi trên xe máy / xe ba bánh có tách riêng khỏi xe hay gộp chung? | Tách riêng từng `person` (cả người điều khiển và hành khách ngồi rõ hình dáng) khỏi khung bao của phương tiện; riêng hàng hóa, thùng chở và mái che cố định gắn trên xe được gộp chung vào khung phương tiện. | `img_013.jpg` | `c02` |
| 2 | Các dãy xe máy đậu sát nhau hoặc di chuyển dày đặc ở hậu cảnh kích thước nhỏ (`< 20 px`) có vẽ từng box lẻ không? | Không vẽ từng box lẻ cho cụm đối tượng quá nhỏ (`< 20 px`) bị che khuất chồng chéo; gom toàn bộ cụm đó vào một khung `ignore_region` duy nhất để tránh nhiễu nhãn. | `img_018.jpg` | `c03` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002.jpg` | Xe tuk-tuk / xe ba gác có thùng kín hoặc mui bạt và xe máy chở hàng cồng kềnh dễ bị nhận diện chéo giữa `car`/`truck`, `three_wheeler` và `motorbike` | `yolo26-s` gọi nhầm xe tuk-tuk thành `car`/`truck`; `yoloe-26s` (ở ngưỡng `0.15`) bắt nhầm xe máy chở hàng cồng kềnh thành `three_wheeler` | Phân loại dựa trên số bánh xe và kết cấu khung gầm (phải có 3 bánh độc lập và khoang chở khách/hàng phía sau mới gán `three_wheeler`); nâng ngưỡng YOLOE lên `0.30–0.35`. | Bổ sung vào **Mục 3.2 (Phân biệt lớp phương tiện)**: Căn cứ vào kết cấu 3 bánh độc lập và khoang chở để phân biệt `three_wheeler` với `motorbike` chở hàng và `car`/`truck` (`c01`). |
| `img_018.jpg` | Ảnh giao thông đông đúc có trên 20 phương tiện đan xen khiến người duyệt dễ bỏ sót các khung bao vẽ lỏng biên (`loose`) | Bẫy `loose` kéo rộng cạnh khung ra ngoài nền đường xung quanh xe máy | Bắt buộc phóng to màn hình (`> 150%`) khi duyệt các ảnh có trên 15 vật thể và rà soát 4 cạnh khung box ôm sát biên ngoài cùng của vật thể (sai số dưới 3 px). | Bổ sung vào **Mục 5.1 (Tiêu chuẩn độ khít khung bao)**: Mọi cạnh khung chữ nhật phải chạm sát biên ngoài vật thể, bắt buộc zoom > 150% với ảnh mật độ cao (`c04`). |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt quy tắc tách riêng người lái/hành khách (`person`) khỏi xe và gom cụm xe nhỏ `< 20 px` vào `ignore_region`; bổ sung tiêu chí phân biệt `three_wheeler` qua kết cấu 3 bánh và quy chuẩn siết chặt độ khít khung box (`< 3 px`) ở ảnh đông đúc. | Khắc phục lỗi nhận diện chéo của `yolo26-s` / `yoloe-26s` (`c01`), chốt câu hỏi mở về ranh giới đối tượng (`c02`, `c03`), và chống bỏ lọt bẫy khung lỏng ở ảnh mật độ cao (`c04`). |
