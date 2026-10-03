# USER STORY: US06 - KIỂM KÊ KHO & ĐIỀU CHỈNH TỒN KHO THEO LÔ (STOCKTAKING & LOT ADJUSTMENT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US06** |
| **Tên User Story** | Kiểm kê Kho & Điều chỉnh Tồn Lô |
| **Phân hệ (Module)** | Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management) |
| **Use Case liên quan** | **UC06: Kiểm kê Kho & Điều chỉnh Tồn Lô** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-INV-03] Kiểm kê & Điều chỉnh Kho theo Lô** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Should-Have (Quan trọng cho vận hành kho)** |
| **Tác nhân chính (Primary Actor)** | Thủ kho (*Warehouse Specialist*) |
| **Tác nhân phụ (Secondary Actor)** | Quản lý / Chủ nhà thuốc (*Manager - Phê duyệt điều chỉnh*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Thủ kho và Quản lý nhà thuốc,
* **Tôi muốn (I want to):** Lập phiếu kiểm kê định kỳ đối soát số lượng tồn thực tế trên kệ thuốc với số lượng ghi nhận trên phần mềm theo từng Lô cụ thể, và thực hiện cân bằng/điều chỉnh tồn kho kèm theo ghi chú lý do giải trình,
* **Để (So that):** Phát hiện kịp thời các chênh lệch thừa/thiếu, hao hụt, bể vỡ hoặc nhầm lẫn giữa các Lô, đưa số liệu phần mềm về đúng thực tế trên quầy kệ nhằm bảo đảm độ chính xác cho việc xuất bán và kiểm soát tài chính.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Trong quá trình bán lẻ, có thể xảy ra tình trạng:
* Khách lấy nhầm vỉ thuốc của Lô A thay vì Lô B (dẫn đến lệch số liệu giữa 2 Lô dù tổng tồn không đổi).
* Hao hụt do vỡ viên thuốc, rơi rớt, ẩm mốc hoặc thất thoát.
Khi kiểm kê (hàng tuần hoặc hàng tháng), Thủ kho đếm số lượng thực tế của từng Lô, nhập vào phiếu kiểm kê. Hệ thống tự động tính độ lệch:
$$\Delta = SoLuongThucTe - SoLuongHeThong$$
Nếu có chênh lệch, Thủ kho ghi rõ lý do. Quản lý xem xét và bấm "Cân bằng kho" để hệ thống điều chỉnh lại `TON_KHO_LO.SoLuongTonGoc`.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-INV-03.1] Kiểm kê chi tiết theo từng Lô:** Không kiểm kê chung theo mã thuốc, bắt buộc kiểm kê chi tiết đến từng Mã Lô (`MaLo`).
* **[BR-INV-03.2] Bắt buộc giải trình chênh lệch:** Nếu $\Delta \ne 0$ (thừa hoặc thiếu), hệ thống bắt buộc người dùng chọn hoặc nhập `Lý do điều chỉnh` (ví dụ: *Hao hụt tự nhiên, Nhầm lẫn khi bán Lô, Rơi vỡ mất phẩm chất, Thất thoát chưa rõ nguyên nhân*).
* **[BR-INV-03.3] Phân quyền phê duyệt:**
  * Thủ kho có quyền nhập số liệu đếm thực tế và lưu tạm phiếu kiểm kê.
  * Chỉ Quản lý / Chủ nhà thuốc mới có thẩm quyền bấm "Xác nhận Điều chỉnh Cân bằng kho".
* **[BR-INV-03.4] Ghi nhật ký bắt buộc (Audit Trail):** Mọi thao tác điều chỉnh tồn kho phải được ghi vào bảng nhật ký kiểm toán với đầy đủ thông tin: `User_ID`, `Thời gian`, `Mã Lô`, `Số lượng trước`, `Số lượng sau`, `Độ lệch`, `Lý do`.
* **[BR-INV-03.5] Không tồn âm:** Số lượng thực tế kiểm kê không được phép là số âm ($\text{SoLuongThucTe} \ge 0$).

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Kiểm kê khớp 100% số lượng (Không có chênh lệch)
* **Given:** Lô `LOT-A` của thuốc `Panadol Extra` trên hệ thống có tồn kho là `100 Viên`.
* **When:** Thủ kho đếm thực tế và nhập vào phiếu kiểm kê: `100 Viên`.
* **Then:** Hệ thống ghi nhận độ lệch $\Delta = 0$, trạng thái "Khớp số liệu". Quản lý bấm "Hoàn tất kiểm kê" mà không cần điều chỉnh tồn kho.

### Kịch bản 2: Kiểm kê phát hiện thiếu và điều chỉnh cân bằng kho (Happy Path điều chỉnh)
* **Given:** Lô `LOT-B` của thuốc `Decolgen` trên hệ thống có tồn kho là `50 Viên`.
* **When:**
  1. Thủ kho đếm thực tế chỉ còn `46 Viên` (thiếu 4 viên do ẩm mốc làm hỏng).
  2. Thủ kho nhập số thực tế `46`, chọn lý do "Ẩm mốc hư hỏng", bấm "Gửi phê duyệt".
  3. Quản lý đăng nhập vào duyệt phiếu kiểm kê và bấm "Cân bằng kho".
* **Then:**
  * Hệ thống cập nhật `TON_KHO_LO.SoLuongTonGoc` của `LOT-B` thành `46 Viên`.
  * Ghi vào sổ nhật ký kho 1 dòng điều chỉnh giảm `4 Viên` kèm lý do "Ẩm mốc hư hỏng".
  * Trạng thái phiếu kiểm kê chuyển thành "Đã cân bằng".

### Kịch bản 3: Báo lỗi khi không nhập lý do điều chỉnh khi có chênh lệch
* **Given:** Thủ kho nhập số lượng thực tế là `80 Viên` trong khi hệ thống đang ghi nhận `100 Viên` ($\Delta = -20$).
* **When:** Thủ kho để trống ô "Lý do điều chỉnh" và cố gắng bấm "Lưu phiếu".
* **Then:** Hệ thống báo lỗi và làm nổi bật trường lý do: "Bắt buộc nhập lý do điều chỉnh khi số lượng thực tế có chênh lệch so với hệ thống!".

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `PHIEU_KIEM_KE`:
  * `MaPhieuKiemKe` (VARCHAR(50), Primary Key)
  * `NgayKiemKe` (DATETIME, Default GETDATE())
  * `NguoiKiemKe` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `NguoiPheDuyet` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`, Nullable)
  * `TrangThai` (VARCHAR(20): `CHO_DUYET`, `DA_CAN_BANG`, `HUY`)
* Bảng `CHI_TIET_PHIEU_KIEM_KE`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaPhieuKiemKe` (VARCHAR(50), Foreign Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `MaLo` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `SoLuongHeThong` (INT, Not Null)
  * `SoLuongThucTe` (INT, Not Null)
  * `SoLuongLech` (INT, Computed Column: `SoLuongThucTe - SoLuongHeThong`)
  * `LyDoDieuChinh` (NVARCHAR(500))

### 6.2. Yêu cầu phi chức năng (NFR)
* **[NFR-SEC-02] Audit Trail:** Ghi nhật ký đầy đủ và không cho phép bất kỳ ai (kể cả Admin) xóa lịch sử phiếu kiểm kê đã cân bằng kho.

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US04, US05:** Đã có danh sách Lô và số lượng tồn gốc trong kho.
* **Là tiền đề cho các User Story:**
  * **US09:** Giúp thuật toán FEFO lấy đúng số lượng tồn thực tế trên kệ.
  * **US13:** Báo cáo Tồn kho chi tiết theo Lô.
