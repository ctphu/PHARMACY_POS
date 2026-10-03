# USER STORY: US03 - THIẾT LẬP BẢNG GIÁ THEO ĐƠN VỊ TÍNH (PRICE PER UNIT CONFIGURATION)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US03** |
| **Tên User Story** | Thiết lập Bảng giá theo từng Đơn vị tính |
| **Phân hệ (Module)** | Phân hệ 1: Quản lý Danh mục & Quy đổi Đơn vị tính (Master Data) |
| **Use Case liên quan** | **UC03: Thiết lập Bảng giá theo ĐVT** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-MD-03] Thiết lập Giá bán theo từng ĐVT** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Quan trọng)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Dược sĩ bán hàng (Áp dụng giá tại POS) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Quản lý / Chủ nhà thuốc,
* **Tôi muốn (I want to):** Thiết lập và điều chỉnh giá bán riêng biệt cho từng Đơn vị tính (ĐVT Gốc và từng ĐVT Quy đổi) của mỗi mặt hàng thuốc,
* **Để (So that):** Áp dụng chính sách giá linh hoạt, tự động khuyến khích khách hàng mua theo đơn vị đóng gói lớn (nguyên vỉ, nguyên hộp) với mức giá ưu đãi hơn so với mua lẻ từng viên, đồng thời giúp màn hình POS tự động tính tiền chính xác theo đơn vị khách chọn.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Trong kinh doanh nhà thuốc, giá bán theo đơn vị lớn thường không đơn thuần bằng phép nhân giá bán lẻ đơn vị nhỏ:
* Mua lẻ 1 Viên: `2.000 VNĐ`.
* Mua 1 Vỉ (10 viên): Nếu nhân trực tiếp sẽ là $10 \times 2.000 = 20.000 \text{ VNĐ}$, nhưng nhà thuốc muốn bán với giá ưu đãi `18.000 VNĐ` (chiết khấu 2.000 VNĐ cho khách mua nguyên vỉ).
* Mua 1 Hộp (10 vỉ = 100 viên): Bán với giá `170.000 VNĐ` (tiết kiệm thêm 10.000 VNĐ so với mua 10 vỉ lẻ).

Hệ thống cần cho phép thiết lập giá bán cụ thể cho từng ĐVT của thuốc mà không bị gò bó bởi phép nhân cứng, và tự động áp giá này khi dược sĩ chọn ĐVT tương ứng trên màn hình bán hàng POS.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-MD-03.1] Bảng giá độc lập theo từng ĐVT:** Mỗi ĐVT được kích hoạt cho thuốc (bao gồm ĐVT Gốc và các ĐVT Quy đổi) phải có một mức đơn giá bán xác định ($\text{GiaBan} \ge 0$).
* **[BR-MD-03.2] Chính sách giá khuyến khích mua số lượng lớn:**
  $$\frac{GiaBan(ĐVT_{Lớn})}{TyLeQuyDoi(ĐVT_{Lớn})} \le GiaBan(ĐVT_{Gốc})$$
  Hệ thống hiển thị cảnh báo (Warning) nếu Quản lý vô tình nhập giá đơn vị lớn đắt hơn mua lẻ nhiều đơn vị nhỏ, nhằm tránh sai sót trong định giá.
* **[BR-MD-03.3] Hiệu lực tức thì:** Thay đổi giá bán sẽ có hiệu lực ngay lập tức cho các giao dịch POS lập mới. Các hóa đơn đã thanh toán hoặc đang lưu tạm trước thời điểm đổi giá giữ nguyên giá tại thời điểm tạo.
* **[BR-MD-03.4] Phân quyền:** Chỉ Quản lý / Chủ nhà thuốc mới có quyền sửa đổi giá bán. Dược sĩ tại màn hình POS không được tự ý sửa đơn giá bán.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Cài đặt bảng giá thành công (Happy Path)
* **Given:** Thuốc `Paracetamol 500mg` có ĐVT Gốc là `Viên`, các ĐVT Quy đổi là `Vỉ` (10 viên) và `Hộp` (100 viên).
* **When:** Quản lý mở màn hình "Cấu hình Giá bán", nhập:
  * Giá bán 1 Viên: `2.000 VNĐ`
  * Giá bán 1 Vỉ: `18.000 VNĐ`
  * Giá bán 1 Hộp: `170.000 VNĐ`
  Và nhấn "Lưu bảng giá".
* **Then:** Hệ thống lưu bảng giá thành công. Khi mở màn hình POS chọn bán 1 Hộp, thành tiền hiển thị đúng `170.000 VNĐ`; chọn 1 Vỉ hiển thị `18.000 VNĐ`; chọn 1 Viên hiển thị `2.000 VNĐ`.

### Kịch bản 2: Cảnh báo định giá bất thường (Warning)
* **Given:** Quản lý đang cấu hình giá cho thuốc `Paracetamol 500mg` (Giá 1 Viên = `2.000 VNĐ`, 1 Vỉ = 10 Viên).
* **When:** Quản lý nhập giá 1 Vỉ = `25.000 VNĐ` (cao hơn mua 10 viên lẻ = 20.000 VNĐ).
* **Then:** Hệ thống hiển thị hộp thoại cảnh báo: "Giá bán của ĐVT 'Vỉ' (25.000đ) đang cao hơn tổng giá bán lẻ theo Viên (20.000đ). Bạn có chắc chắn muốn áp dụng mức giá này?". Nếu Quản lý bấm "Xác nhận", hệ thống mới tiến hành lưu.

### Kịch bản 3: Báo lỗi khi nhập giá âm hoặc không hợp lệ (Validation)
* **Given:** Quản lý đang ở form nhập giá bán.
* **When:** Quản lý nhập giá bán là số âm `-5000` hoặc để trống ô giá bán của một ĐVT.
* **Then:** Hệ thống hiển thị thông báo lỗi: "Giá bán không được để trống và phải lớn hơn hoặc bằng 0". Không cho phép lưu.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bổ sung trường giá vào bảng `THUOC` (cho ĐVT Gốc) và `QUY_DOI_DVT` (cho các ĐVT Quy đổi):
  * `THUOC.GiaBanGoc` (DECIMAL(18, 2), Not Null, Default 0)
  * `QUY_DOI_DVT.GiaBan` (DECIMAL(18, 2), Not Null, Check Constraint: `GiaBan >= 0`)
* Hoặc quản lý qua bảng lịch sử bảng giá:
  * `BANG_GIA_CHI_TIET` (`MaBangGia`, `MaThuoc`, `MaDVT`, `DonGiaBan`, `NgayHieuLuc`)

### 6.2. Ràng buộc bảo mật (Security Constraint)
* Phân quyền (RBAC): Chỉ người dùng có Role = `MANAGER` hoặc `ADMIN` mới có quyền gọi API `PUT /api/medicines/{id}/prices`. Người dùng Role = `SALES_CLERK` (Dược sĩ) chỉ có quyền `GET`.

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US01:** Đã có thuốc và ĐVT Gốc.
  * **US02:** Đã cấu hình các ĐVT Quy đổi cho thuốc.
* **Là tiền đề cho các User Story:**
  * **US08:** Lập Hóa đơn Bán lẻ tại POS (hệ thống tự lấy giá bán theo ĐVT đã chọn).
  * **US14:** Xem Báo cáo Doanh thu & Lợi nhuận (tính doanh thu dựa trên giá bán thực tế).
