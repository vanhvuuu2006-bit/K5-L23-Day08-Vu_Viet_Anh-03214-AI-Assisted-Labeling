# Thông tin học viên

- Họ và tên: Vũ Việt Anh
- MSHV: 2A202603214
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: vanhvuuu2006-bit
- Repo bài nộp (URL): https://github.com/vanhvuuu2006-bit/K5-L23-Day08-Vu_Viet_Anh-03214-AI-Assisted-Labeling
- Commit chốt bài (hash): c25a3b4
- Máy: Windows 10 + WSL2 (Ubuntu 24.04), 16 GB RAM, 16 luồng CPU, GPU: không có (chạy CPU)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: **311 (84.7%) / 16 (4.4%) / 40 (10.9%) (mẫu số: 367 box máy)**; added: **74 (18.5%, gồm 60 sót thật và 14 đổi lớp) (mẫu số: 401 đối tượng)**
- So với gold: precision / recall của **máy** **58.0% (29/50) / 93.5% (29/31)**
- Bắt bẫy: **10/11 = 90.9%**
- Thời gian: gán tay **4.5 phút/ảnh**; duyệt có pre-label **2.0 phút/ảnh**
- Lỗi QC: **0.0% theo ảnh (0/5)**, **0.0% theo đối tượng (0/82)**
- **Quyết định:** **Release** — lý do một dòng: Tỷ lệ ảnh lỗi QC đạt 0.0% (≤ 2%), bắt bẫy đạt 90.9% (> 80%), hiệu quả hỗ trợ của máy cao (accepted 84.7%) và đã chốt guideline v1.1 đầy đủ.
