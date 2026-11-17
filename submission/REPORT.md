# Báo cáo Day 08 — 2A202603214 - Vũ Việt Anh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Ngưỡng gợi ý | Lý do |
|---|---|---|
| person | 0.5 | recall 100% (>= 90%), precision 62% (10 vật gold) |
| bicycle | (mặc định) | chỉ 2 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| motorbike | (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 80%, 5 vật): dùng ngưỡng mặc định, nói rõ trong báo cáo |
| car | 0.45 | recall 100% (>= 90%), precision 64% (7 vật gold) |
| bus | (mặc định) | chỉ 1 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| truck | (mặc định) | chỉ 0 vật trong ảnh gold (< 5): mẫu quá ít, KHÔNG chọn ngưỡng theo lớp này, dùng ngưỡng mặc định |
| three_wheeler | 0.15 | recall 100% (>= 90%), precision 67% (6 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): "person=0.5,car=0.45,three_wheeler=0.15"
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: chất lượng mask, tốc độ, RAM, quyền truy cập Chọn **SAM 2.1 (`sam2.1_b.pt`)** (đã thử ở `r04`). Phiên bản này cho chất lượng đường bao (mask) sắc nét, bám sát chi tiết phương tiện hơn hẳn SAM 1 (`mobile_sam.pt`), chạy ổn định trên cấu hình 16 GB RAM (mức 8 GB cấp cho WSL) với tốc độ chấp nhận được trên CPU (~4.03 giây/ảnh) và tải mở trực tiếp không cần xin quyền truy cập phức tạp như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi gán sai tên lớp (class confusion) do lệch phân phối dữ liệu huấn luyện (ví dụ: YOLO26 liên tục gọi xe tuk-tuk/ba gác là `car` hoặc `truck` với điểm tự tin rất cao) và lỗi hai vật thể dính sát nhau bị gộp chung một khung box không thể sửa bằng cách tăng hay giảm ngưỡng. Cách sửa: kết hợp mô hình open-vocabulary (**YOLOE**) chuyên bắt lớp `three_wheeler`, loại bỏ khung trùng lặp bằng NMS khi gộp nhãn (`merge_prelabels.py`), và người review sửa trực tiếp trên CVAT (đổi nhãn lớp, xóa box nhầm hoặc tách/vẽ lại box chuẩn)
Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào):
  - Điểm cao mà sai: Trên các ảnh pilot có xe tuk-tuk/ba gác (như `img_002.jpg`, `img_004.jpg`), `yolo26-s` gán nhầm phần đầu hoặc thùng xe tuk-tuk thành `car`/`truck` với điểm tự tin trên `0.6–0.8`, hoặc bắt nhầm ô dù/biển quảng cáo thành `person` ở ngưỡng `0.5` (`r02`).
  - Điểm thấp mà đúng: Ở ngưỡng thấp (`0.15 – 0.25`), `yoloe-26s` bắt đúng các xe tuk-tuk/ba gác nằm ở góc nghiêng hoặc bị che khuất một phần (`r03`), và `yolo26-s` bắt đúng các `motorbike` / `person` ở xa phía cuối đường mà khi nâng lên `0.5` (`r02`) đã bị lọc mất.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Tập Gold chỉ có **4 ảnh** (32 đối tượng) và việc tìm ngưỡng lẫn chấm điểm đều thực hiện trên cùng một tập dữ liệu này (không có tập validation tách biệt). Do đó, các chỉ số precision (62%–67%) và recall (100%) đo được mang tính lạc quan (bị quá khớp vào 4 ảnh mẫu) và thực tế sẽ giảm nhẹ khi áp dụng sang lô 24 ảnh mới ở Part 4.
## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 311 | 84.7% | 367 box máy |
| edited | 16 | 4.4% | 367 box máy |
| deleted | 40 | 10.9% | 367 box máy |
| added | 74 (60 sót thật, 14 đổi lớp) | 18.5% | 401 đối tượng sau duyệt |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? Lớp **`three_wheeler`** tệ nhất về tỷ lệ khung giả (`deleted`: **24 box** và `edited`: **7 box** trên tổng 41 box máy vẽ, chỉ giữ nguyên được 10 box). Nguyên nhân là do ở Part 3 ta chọn ngưỡng quá thấp (`0.15`) trên tập Gold nhỏ khiến `yoloe-26s` sinh nhiều box rác (nhận nhầm xe máy chở hàng cồng kềnh hoặc ô tô góc nghiêng thành xe ba bánh) và vẽ box lỏng; ngoài ra lớp **`person`** có số lượng bỏ sót (`added`: **38 box**) nhiều nhất ở các ảnh đông đúc (như `img_005.jpg`, `img_018.jpg`, `img_024.jpg`) do ngưỡng `0.5` hơi cao làm mất người ngồi sau xe máy hoặc người đứng ở xa.
- So với gold: precision / recall của **máy** **Precision = 58.0% (29/50, KTC 95%: 44–71%) / Recall = 93.5% (29/31, KTC 95%: 79–98%)** (mẫu số: **31 đối tượng gold** trên 4 ảnh).

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **10/11 (90.9%, KTC 95%: 62–98%)** *(hệ thống sinh 11 bẫy hợp lệ theo MSHV 03214)*; loại nào lọt: **1 bẫy loại `loose` (khung vẽ lỏng/rộng hơn vật thể) tại ảnh `img_018.jpg`** (các loại `wrong_class`: 3/3, `missing`: 2/2, `extra`: 3/3 đều bắt đạt 100%); vì sao (bạn đã bỏ bước nào của checklist)? Ảnh `img_018.jpg` thuộc nhóm 6 ảnh xe máy dày đặc, tôi đã làm tốt bước 1–3 (ẩn nhãn quét trước, bắt đủ vật sót và sai lớp) nhưng ở **bước 4 (kiểm tra độ ôm khít của cạnh khung — lỏng)** đã duyệt lướt hơi nhanh dưới áp lực số lượng box chồng chéo lớn nên bỏ qua một box bị kéo rộng biên.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Ở đây **cả `accepted` đều cao (84.7%) lẫn `bắt bẫy` đều rất cao (90.9% > ngưỡng 80%)**: điều này chứng minh tỷ lệ giữ nguyên 84.7% đến từ việc mô hình thực sự dự đoán tốt và đã được thẩm định kỹ lưỡng qua quy trình quét ảnh độc lập trước khi nhìn nhãn máy, chứ không phải do người duyệt lười biếng bấm Accept theo quán tính (**automation bias**).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **8p/ 4 phút/ảnh** vs duyệt **50p/ 24 phút/ảnh**. Thiên lệch (bạn gán gold trước nên đã quen cảnh): Việc gán 4 ảnh Gold trước giúp tôi nắm vững ranh giới các lớp đặc thù đường phố châu Á (tuk-tuk, xe ba gác, cụm `ignore_region`) và quen tay phím tắt CVAT nên tốc độ duyệt ở lô 24 ảnh sau đó trôi chảy hơn; ngược lại, do không phải ngồi vẽ hộp chữ nhật từ đầu cho 311 đối tượng (84.7%) trên một lô ảnh có mật độ dày đặc hơn hẳn tập Gold (trung bình ~16.7 vật/ảnh so với ~8 vật/ảnh của Gold), thời gian thao tác trên mỗi đối tượng giảm đi rõ rệt.
- Kết luận: **Có — rất đáng dùng cho toàn bộ quy trình (đặc biệt hiệu quả cao nhất cho `person`, `motorbike`, `car`, còn `three_wheeler` nên nâng ngưỡng lên `0.30–0.35` trước khi nạp vào CVAT)**, dựa trên **tỷ lệ `accepted` đạt 84.7% (vượt xa ngưỡng hòa vốn 50%), tỷ lệ chỉnh sửa/xóa thấp (15.3%), và chỉ mất trung bình vài phút/ảnh cho lô 401 đối tượng**.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quét mù ảnh gốc 3 giây trước khi bật nhãn AI:** Khi chuyển sang ảnh mới, luôn nhấn phím tắt ẩn toàn bộ lớp nhãn máy trong tối thiểu 3 giây để mắt tự đếm và định vị vật thể trước, đảm bảo không bỏ sót lỗi `missing` (đã giúp bắt 2/2 bẫy `missing`).
  2. **Duyệt tách biệt theo 4 lượt lỗi thay vì duyệt từng box:** Không bấm Accept tuần tự từng khung mà quét toàn ảnh theo 4 vòng riêng biệt (sót → thừa → sai tên lớp → độ khít biên), đặc biệt phóng to >150% ở các ảnh có trên 15 vật thể (như `img_005`, `img_018`, `img_021`, `img_024`, `img_028`, `img_029`) để kiểm tra lỗi `loose`.
  3. **Nghỉ giải lao bắt buộc 2 phút sau mỗi 10 ảnh:** Đặt mốc nghỉ mắt sau mỗi 10 ảnh duyệt xong (hoặc mỗi 25–30 phút) để duy trì mức độ tập trung và giữ tỷ lệ bắt bẫy luôn duy trì trên ngưỡng **80%**.


## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không gồm người lái (đã tách bằng điểm âm); gồm phần mái che và thùng xe | Lem nhẹ một chút ở vùng bóng râm bánh xe và thanh chắn khung xe | `tuktuk_img013.jpg` |
| `img_004.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 pos, 1 neg) | Không gồm người đứng cạnh; gồm trọn vẹn mái che chữ TAXI và thùng xe tuk-tuk đỏ | Mask ôm rất sắc nét toàn thân và bánh xe; chỉ khuyết rất nhỏ ở góc khuất phía sau người đàn ông áo đen | `tuktuk_img004.jpg` |
| `img_002.jpg` | SAM 2.1 (`sam2.1_b.pt`) | 4 (2 pos, 2 neg) | Không gồm tài xế áo xanh (tách sạch nhờ điểm neg trên lưng áo); gồm hành lý/hàng hóa bên trong khoang xe | Ở lần thử 3 click đầu, điểm pos sát gầm xe làm mask tràn xuống mặt đường nhựa; sau khi thêm 1 điểm neg dưới mặt đường thì cắt sạch lỗi tràn | `vehicle_img002.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - *Quy tắc giúp ích:* Quy định tách riêng người điều khiển (`person`) khỏi phương tiện (`three_wheeler` / `motorbike`) và gộp phần hàng hóa/mái che gắn liền trên xe vào chung mask phương tiện giúp đặt đúng vị trí điểm âm (`--neg` lên người lái) và điểm dương (`--pos` lên thùng hàng/mái che) nhất quán trên cả 3 ảnh.
  - *Quy tắc còn thiếu / chưa rõ:* Guideline mục 4.2 chưa quy định rõ cách xử lý các khoảng trống xuyên thấu bên trong khung xe (ví dụ: phần khung sắt hở của xe tuk-tuk/ba gác nhìn xuyên qua thấy người đi bộ hoặc nền phố phía sau — nên để lỗ rỗng *hole* trong mask hay lấp đầy theo bao lồi).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Chưa kích hoạt serverless Nuclio trên Docker nên chỉ chạy SAM offline qua `sam2_points.py`. Khác biệt chính: chạy offline phải tự nhập tọa độ pixel `(x, y)` thủ công qua dòng lệnh và chỉ xuất ảnh kiểm tra trực quan, còn dùng Interactor SAM trực tiếp trong CVAT cho phép click chuột trực quan lên ảnh, thấy mask cập nhật tức thì theo từng cú click và lưu thẳng thành polygon/mask vào dataset.


- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? 
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: 

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** (`calibration_decisions.csv`: `c01`, `c02`, `c03`, `c04`); quan trọng nhất (dẫn `CAL-xx`): **`c01` (`img_002.jpg`)** — chuẩn hóa tiêu chí phân biệt `three_wheeler` (tuk-tuk/ba gác 3 bánh độc lập và khoang chở) với `motorbike` chở hàng và `car`/`truck`, đồng thời nâng ngưỡng YOLOE từ `0.15` lên `0.30–0.35` để cắt giảm mạnh lượng khung giả (`deleted` 24 box ở Part 4).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - Chốt câu hỏi mở mục 6 của v1.0: tách riêng từng `person` (người lái và hành khách ngồi rõ hình dáng) khỏi phương tiện, gộp hàng hóa và mái che gắn liền trên xe vào khung phương tiện (`c02`, `img_013.jpg`); gom cụm xe/người kích thước `< 20 px` chồng chéo dày đặc vào một khung `ignore_region` duy nhất (`c03`, `img_018.jpg`).
  - Bổ sung quy tắc từ lỗi máy và bẫy lọt: phân biệt `three_wheeler` dựa trên kết cấu 3 bánh và khoang chở (`c01`, `img_002.jpg`); bắt buộc phóng to `> 150%` và siết cạnh khung box cách biên vật thể `< 3 px` ở các ảnh mật độ cao (`c04`, `img_018.jpg`).
- Nguyên nhân gốc: **Kết hợp cả 3 yếu tố**: (1) **máy** bị lệch miền dữ liệu (YOLO26 không có lớp tuk-tuk) và ngưỡng `0.15` từ tập Gold nhỏ quá thấp nên sinh nhiều nhiễu; (2) **guideline v1.0** còn mở ở ranh giới người ngồi trên xe nhiều khoang và cụm vật thể nhỏ; (3) **chưa siết chặt bước cuối của checklist** (độ khít biên khung) khi gặp ảnh đông xe máy nên để lọt 1 bẫy `loose` ở `img_018.jpg`.

## 7. QC và quyết định (Part 6)

- Mẫu: **5** ảnh (**0** ảnh `can_review` + **5** ngẫu nhiên, seed 42: `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`), tổng cộng **82 đối tượng**; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt nên có thể tồn tại điểm mù chủ quan giống lúc duyệt ban đầu).
- Lỗi theo **đối tượng**: **0/82 = 0.0%** · theo **ảnh**: **0/5 = 0.0%** (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: **Không ghi nhận lỗi nào (`0/82` đối tượng)** trên cả 5 ảnh mẫu QC (các đối tượng trong `img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg` đều bám sát biên, phân loại đúng người/xe và tách bạch người lái khỏi phương tiện).
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): Với **0 ảnh lỗi trong 5 ảnh mẫu**, cận trên khoảng tin cậy 95% (Wilson) của tỷ lệ ảnh lỗi thực tế lên tới **43%**. Do đó, ngưỡng chất lượng `≤ 2%` chưa thể được chứng minh tuyệt đối bằng mẫu nhỏ 5 ảnh mà cần tiếp tục lấy mẫu lớn hơn nếu triển khai thực tế.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật):
  - *Script kiểm được:* Tính hợp lệ cấu trúc COCO JSON, mã lớp hợp lệ, kích thước khung dương, tỷ lệ trùng lặp/lỗi tự động và tổng hợp phiếu kiểm QC.
  - *Script không kiểm được:* Độ ôm sát thực tế của cạnh khung quanh vật thể trên ảnh, việc người duyệt có cùng bỏ sót một đối tượng bị che khuất giống máy hay không, và việc người duyệt có thực sự ẩn nhãn máy quét ảnh trước theo đúng checklist hay không.

**Quyết định: Release**, ngày **2026-11-17**. Lý do (đi theo Hold → Rework → Release):
1. *Không Hold:* Bộ guideline v1.1 đã chốt rõ mọi ca nhập nhằng (`c01`–`c04`), dữ liệu 36 ảnh và file nhãn xuất bản (`release/`) toàn vẹn, hợp lệ qua `validate_labels.py`.
2. *Không Rework:* Tỷ lệ ảnh lỗi trên mẫu QC đạt `0.0%` (`≤ 2%`), không phát hiện lỗi hệ thống lặp lại, và tỷ lệ bắt bẫy ở bước review đạt `10/11 = 90.9%` (vượt xa ngưỡng yêu cầu `80%`).
3. *Chọn Release:* Đạt đầy đủ mọi tiêu chí chất lượng, nhất quán nhãn và kiểm soát thiên kiến tự động hóa để phát hành tập dữ liệu.
Nếu Rework / Hold, việc cần làm tiếp: Không áp dụng (quyết định **Release**); nếu mở rộng bộ dữ liệu ở giai đoạn sau sẽ áp dụng ngay ngưỡng `three_wheeler = 0.30–0.35` và lấy mẫu QC lớn hơn (`≥ 30–50 ảnh`) để thu hẹp khoảng tin cậy Wilson.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Có sử dụng trợ lý AI (Gemini) để hỗ trợ giải thích lỗi môi trường WSL2/Docker/Git, rà soát cú pháp lệnh Python/CSV và chuẩn hóa cấu trúc trình bày báo cáo; mọi số liệu chạy mô hình, thao tác gán nhãn tay, duyệt ảnh trên CVAT và kiểm tra ảnh mask đều tự thực hiện trực tiếp và đối chiếu bằng các script kiểm thử của bài (`validate_labels.py`, `review_stats.py`, `qc_report.py`, `check_submission.py`).
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Không** — 4 ảnh tập Gold (`001`, `012`, `020`, `036`) được gán thủ công hoàn toàn trên CVAT ở Part 1 trước khi chạy bất kỳ mô hình pre-label nào (Part 2–4), đảm bảo tính độc lập tuyệt đối để đo lường máy.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Máy cá nhân chạy Windows (15.7 GB RAM, 16 luồng CPU, không có GPU rời nên chạy hoàn toàn trên **CPU**), cài Ubuntu 24.04 qua **WSL2** (cấu hình giới hạn `memory=8GB`, `processors=8`, `swap=8GB`) và chạy **CVAT v2.81.0** cục bộ qua **Docker Desktop**. Vướng mắc cài đặt đã xử lý: lỗi phân quyền Docker trên WSL (`usermod -aG docker`), thiếu thư viện `transformers`/`huggingface_hub`, và đồng bộ Git từ bản tải ZIP lên GitHub.
- Dùng cache `data/cache/` thay vì tự chạy? **Không** — đã tự chạy trực tiếp cả `yolo26-s`, `yoloe-26s` và `sam2.1_b.pt` trên CPU của máy ở Part 2 và Part 5 để lấy dữ liệu thực tế cho toàn bộ quy trình.
- Lỗi / bug script đã gặp (nếu có): Trong file hướng dẫn cài đặt có ký tự toán học `⩾` thay vì `>=` khi cài `ultralytics`; khi chạy script từ Terminal mặc định của VS Code bị gọi nhầm Python của Windows thay vì Python trong WSL2; và cột `error_type` trong `qc_review.csv` cần điền đúng từ khóa `ok` để `qc_report.py` nhận diện ảnh đã kiểm.
- Một điều bạn làm khác lần sau: Chạy `threshold_sweep.py` trên một tập validation độc lập lớn hơn (khoảng 10–15 ảnh thay vì chỉ 4 ảnh Gold) và đặt ngưỡng khởi điểm cho `three_wheeler` từ `0.30` trở lên trước khi nạp vào CVAT để giảm bớt công sức xóa box giả ở bước review.

