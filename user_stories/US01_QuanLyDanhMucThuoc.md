# USER STORY: US01 - QUẢN LÝ DANH MỤC THUỐC (MEDICINE CATALOG MANAGEMENT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US01** |
| **Tên User Story** | Quản lý Danh mục Thuốc gốc |
| **Phân hệ (Module)** | Phân hệ 1: Quản lý Danh mục & Quy đổi Đơn vị tính (Master Data) |
| **Use Case liên quan** | **UC01: Quản lý Danh mục Thuốc** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-MD-01] Quản lý Danh mục Thuốc gốc** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Cao nhất)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Dược sĩ / Nhân viên bán lẻ (Tra cứu thông tin) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Quản lý / Chủ nhà thuốc,
* **Tôi muốn (I want to):** Tạo mới, cập nhật, tra cứu và quản lý danh mục thông tin các mặt hàng thuốc (Mã thuốc, Tên biệt dược, Tên hoạt chất, Hàm lượng, Nhà sản xuất, Nhóm thuốc, ĐVT Gốc),
* **Để (So that):** Thiết lập dữ liệu định danh chuẩn xác làm cơ sở cho toàn bộ quy trình vận hành nhập kho, thiết lập quy đổi đơn vị tính, quản lý Lô - HSD và bán hàng tại quầy POS.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Tại nhà thuốc, mỗi mặt hàng thuốc cần được định danh rõ ràng theo quy định của Dược điển và Bộ Y tế. Dữ liệu cốt lõi bao gồm tên thuốc, hoạt chất chính (để phục vụ tư vấn thay thế), hàm lượng, nhà sản xuất và đặc biệt là **Đơn vị tính Gốc (Base Unit)** – đơn vị nhỏ nhất không thể chia nhỏ hơn nữa trong kinh doanh (ví dụ: Viên, Gói, Chai, Ống). ĐVT Gốc là mỏ neo dữ liệu cho mọi phép tính tồn kho và quy đổi sau này.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-MD-01.1] Bắt buộc ĐVT Gốc:** Mỗi mặt hàng thuốc bắt buộc phải có đúng một ĐVT Gốc ngay khi tạo mới.
* **[BR-MD-01.2] Khóa chỉnh sửa ĐVT Gốc:** Khi mặt hàng thuốc đã phát sinh bất kỳ giao dịch nào (đã có phiếu nhập kho, hóa đơn bán lẻ hoặc điều chỉnh kiểm kê), hệ thống **tuyệt đối không cho phép sửa đổi hoặc xóa ĐVT Gốc** nhằm bảo toàn tính toàn vẹn dữ liệu tồn kho.
* **[BR-MD-01.3] Tính duy nhất của Mã thuốc:** Mã thuốc (`MaThuoc`) là duy nhất trên toàn hệ thống. Có thể sinh tự động theo quy tắc (`TH000001`, `TH000002`...) hoặc nhập mã vạch/mã nội bộ.
* **[BR-MD-01.4] Xóa mềm (Soft Delete):** Không hỗ trợ xóa cứng (Hard delete) các mặt hàng thuốc đã có lịch sử giao dịch. Chỉ cho phép chuyển trạng thái sang `Ngừng kinh doanh` (Inactive).

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Thêm mới thuốc thành công (Happy Path)
* **Given:** Quản lý đã đăng nhập và đang ở màn hình "Quản lý Danh mục Thuốc".
* **When:** Quản lý bấm "Thêm thuốc mới", điền đầy đủ các thông tin bắt buộc:
  * Mã thuốc: `PARA500`
  * Tên thuốc: `Paracetamol 500mg`
  * Tên hoạt chất: `Paracetamol`
  * Hàm lượng: `500mg`
  * Nhà sản xuất: `Dược Hậu Giang`
  * ĐVT Gốc: `Viên`
  Và bấm "Lưu".
* **Then:** Hệ thống kiểm tra hợp lệ, lưu thuốc vào CSDL với trạng thái `Đang kinh doanh`, hiển thị thông báo "Thêm mới thuốc thành công" và hiển thị thuốc mới trên danh sách.

### Kịch bản 2: Báo lỗi khi thiếu dữ liệu bắt buộc hoặc trùng Mã thuốc (Validation)
* **Given:** Quản lý đang ở form thêm mới thuốc.
* **When:** Quản lý nhập `Mã thuốc` đã tồn tại trong hệ thống HOẶC bỏ trống các trường bắt buộc (`Tên thuốc`, `ĐVT Gốc`) và bấm "Lưu".
* **Then:** Hệ thống không lưu dữ liệu, giữ nguyên form nhập và hiển thị thông báo lỗi cụ thể tương ứng (ví dụ: "Mã thuốc đã tồn tại trong hệ thống", "Đơn vị tính gốc không được để trống").

### Kịch bản 3: Chỉnh sửa thông tin thuốc đã có giao dịch (Boundary / Constraint)
* **Given:** Thuốc `Paracetamol 500mg` đã có ít nhất một phiếu nhập kho hoặc hóa đơn bán lẻ.
* **When:** Quản lý mở form "Chỉnh sửa thuốc".
* **Then:**
  * Trường `ĐVT Gốc` bị làm mờ (disabled/read-only) và có tooltip giải thích: "Không thể thay đổi ĐVT Gốc vì thuốc đã phát sinh giao dịch tồn kho".
  * Các trường khác (`Tên thuốc`, `Hàm lượng`, `Nhà sản xuất`, `Ghi chú`) vẫn cho phép chỉnh sửa và lưu bình thường.

### Kịch bản 4: Tìm kiếm và lọc danh mục thuốc
* **Given:** Danh mục thuốc có nhiều sản phẩm.
* **When:** Quản lý/Dược sĩ nhập từ khóa tìm kiếm (theo Tên thuốc, Tên hoạt chất hoặc Mã thuốc) tại ô tìm kiếm.
* **Then:** Bảng danh sách cập nhật kết quả theo thời gian thực (debounce 300ms) trả về đúng các mặt hàng thuốc khớp với từ khóa tìm kiếm.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `THUOC`:
  * `MaThuoc` (VARCHAR(50), Primary Key)
  * `TenThuoc` (NVARCHAR(255), Not Null)
  * `HoatChat` (NVARCHAR(255))
  * `HamLuong` (NVARCHAR(100))
  * `NhaSanXuat` (NVARCHAR(255))
  * `MaDVTGoc` (VARCHAR(20), Foreign Key $\rightarrow$ `DON_VI_TINH.MaDVT`)
  * `TrangThai` (TINYINT: 1 - Đang kinh doanh, 0 - Ngừng kinh doanh)
  * `MoTa` (NVARCHAR(MAX))
* Bảng `DON_VI_TINH`:
  * `MaDVT` (VARCHAR(20), Primary Key)
  * `TenDVT` (NVARCHAR(50), Not Null)

### 6.2. Kiểm tra giao dịch tồn tại (Query Constraint)
```sql
-- Kiểm tra xem thuốc đã phát sinh giao dịch chưa trước khi cho phép sửa ĐVT gốc hoặc xóa
SELECT COUNT(1) FROM (
    SELECT MaThuoc FROM LO_HANG WHERE MaThuoc = @MaThuoc
    UNION ALL
    SELECT MaThuoc FROM CHI_TIET_HOA_DON WHERE MaThuoc = @MaThuoc
) AS Transactions;
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:** Đã khởi tạo bảng danh mục Đơn vị tính chung (`DON_VI_TINH`).
* **Là tiền đề cho các User Story:**
  * **US02:** Thiết lập ĐVT & Tỷ lệ Quy đổi (phụ thuộc vào `MaDVTGoc` của US01).
  * **US04:** Lập Phiếu Nhập kho theo Lô/HSD (cần có danh mục thuốc để chọn mặt hàng nhập).
  * **US08:** Lập Hóa đơn Bán lẻ (cần có danh mục thuốc để bán).
