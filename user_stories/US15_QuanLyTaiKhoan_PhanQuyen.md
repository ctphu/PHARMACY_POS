# USER STORY: US15 - QUẢN LÝ TÀI KHOẢN & PHÂN QUYỀN TRUY CẬP (USER MANAGEMENT & RBAC)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US15** |
| **Tên User Story** | Quản lý Tài khoản Người dùng & Phân quyền Truy cập (RBAC) |
| **Phân hệ (Module)** | Phân hệ 4: Cảnh báo, Báo cáo & Quản trị (Alerts, Reports & Admin) |
| **Use Case liên quan** | **UC15: Quản lý Tài khoản & Phân quyền** (DacTa.md) |
| **Yêu cầu SRS** | **[NFR-SEC-01] Phân quyền truy cập (RBAC)**, **[Mục 2.2] Tác nhân người dùng** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Bảo mật & Kiểm soát nội bộ)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Nhân viên (Dược sĩ bán hàng, Thủ kho) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Chủ nhà thuốc / Quản lý hệ thống,
* **Tôi muốn (I want to):** Khởi tạo, chỉnh sửa thông tin, đặt lại mật khẩu, khóa tài khoản nhân viên và gán vai trò quyền hạn cụ thể (Dược sĩ bán hàng, Thủ kho, Quản lý) cho từng tài khoản,
* **Để (So that):** Phân định ranh giới trách nhiệm rõ ràng giữa các bộ phận, bảo vệ dữ liệu tài chính mật của nhà thuốc, ngăn ngừa tình trạng nhân viên quầy can thiệp tùy tiện vào giá bán hoặc gian lận số liệu tồn kho.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Tại một hiệu thuốc, nhân sự có các chuyên môn và trách nhiệm khác nhau:
* **Dược sĩ bán lẻ:** Cần tập trung tư vấn đơn thuốc và thao tác bán hàng nhanh tại POS. Nhân viên này tuyệt đối không được phép chỉnh sửa đơn giá thuốc đã niêm yết, không được điều chỉnh số lượng tồn kho tự do, và không được xem doanh số/lợi nhuận của nhà thuốc.
* **Thủ kho:** Chịu trách nhiệm nhập hàng, xếp kho, gắn số Lô/HSD và kiểm kê hàng hóa. Thủ kho không có quyền bán lẻ tại POS và không được xem báo cáo tài chính kinh doanh.
* **Chủ nhà thuốc / Quản lý:** Có toàn quyền quản trị danh mục, định giá, phê duyệt xuất hủy thuốc và theo dõi toàn bộ báo cáo doanh thu, lãi lỗ.

Hệ thống cần áp dụng mô hình phân quyền theo vai trò **RBAC (Role-Based Access Control)** để đảm bảo an ninh thông tin chặt chẽ.

---

## 4. QUY TẮC NGHIỆP VỤ & MA TRẬN PHÂN QUYỀN (BUSINESS RULES & RBAC MATRIX)

### 4.1. Ma trận Phân quyền Chức năng (RBAC Permission Matrix)

| Chức năng / User Story | Dược sĩ bán lẻ (Role: `PHARMACIST`) | Thủ kho (Role: `WAREHOUSE`) | Quản lý / Chủ (Role: `MANAGER`) |
| :--- | :---: | :---: | :---: |
| **US01:** Quản lý Danh mục Thuốc | Chỉ xem | Chỉ xem | **Toàn quyền (CRUD)** |
| **US02:** Thiết lập ĐVT & Quy đổi | Chỉ xem | Chỉ xem | **Toàn quyền (CRUD)** |
| **US03:** Thiết lập Bảng giá theo ĐVT | Không truy cập | Không truy cập | **Toàn quyền (CRUD)** |
| **US04, US05:** Nhập kho theo Lô/HSD | Không truy cập | **Toàn quyền (CRUD)** | Toàn quyền / Xem |
| **US06:** Kiểm kê & Cân bằng tồn Lô | Không truy cập | Lập phiếu kiểm | **Phê duyệt cân bằng** |
| **US07:** Xử lý Hủy thuốc Hết hạn | Không truy cập | Lập phiếu hủy | **Phê duyệt xuất hủy** |
| **US08, US09:** Bán lẻ POS & Trừ FEFO | **Toàn quyền** | Không truy cập | Toàn quyền |
| **US10:** Chọn Lô Thủ công (Override) | Thực hiện (có Audit) | Không truy cập | Toàn quyền / Cấu hình PIN |
| **US11:** Đổi / Trả hàng bán lẻ | **Thực hiện** | Không truy cập | Xem / Phê duyệt quá hạn |
| **US12:** Cảnh báo Thuốc Cận hạn | Xem cảnh báo | **Xử lý cảnh báo** | **Xử lý cảnh báo** |
| **US13:** Báo cáo Tồn kho Đa quy đổi | Tra cứu cơ bản | **Toàn quyền xem/xuất** | **Toàn quyền xem/xuất** |
| **US14:** Báo cáo Doanh thu & Lãi | **Bị cấm truy cập** | **Bị cấm truy cập** | **Toàn quyền xem/xuất** |
| **US15:** Quản lý Tài khoản & Phân quyền | **Bị cấm truy cập** | **Bị cấm truy cập** | **Toàn quyền (CRUD)** |

### 4.2. Quy tắc an toàn & xác thực
* **[BR-SEC-15.1] Mã hóa mật khẩu:** Mật khẩu người dùng bắt buộc được băm bằng thuật toán an toàn tiêu chuẩn ngành (**BCrypt** với Work Factor $\ge 12$ hoặc **Argon2id**), tuyệt đối không lưu Plain text.
* **[BR-SEC-15.2] Khóa tài khoản lập tức:** Khi nhân viên nghỉ việc hoặc có dấu hiệu vi phạm, Quản lý đổi trạng thái tài khoản sang `Locked/Inactive`. Mọi phiên làm việc (Session/Token JWT) của tài khoản đó phải bị vô hiệu hóa ngay lập tức.
* **[BR-SEC-15.3] Độ phức tạp mật khẩu:** Tối thiểu 8 ký tự, bao gồm chữ hoa, chữ thường, chữ số.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Tạo mới tài khoản Dược sĩ và kiểm tra phân quyền màn hình (Happy Path)
* **Given:** Quản lý đang ở màn hình "Quản lý Tài khoản".
* **When:** Quản lý tạo tài khoản mới:
  * Username: `duocsi_mai`
  * Họ tên: `Nguyễn Thị Mai`
  * Vai trò: `Dược sĩ bán hàng (PHARMACIST)`
  * Mật khẩu tạm thời: `Mai@2026!`
* **Then:**
  * Hệ thống tạo tài khoản thành công.
  * Khi Dược sĩ Mai đăng nhập, menu hệ thống chỉ hiển thị: "Màn hình POS Bán hàng", "Đổi trả hàng", "Tra cứu danh mục thuốc".
  * Các menu: "Bảng giá", "Nhập kho", "Kiểm kê kho", "Báo cáo Doanh thu Lợi nhuận" hoàn toàn ẩn đi.
  * Nếu Dược sĩ Mai cố tình gõ trực tiếp URL `/reports/revenue`, hệ thống chặn lại và trả về lỗi `403 Forbidden` cùng thông báo: "Bạn không có quyền truy cập chức năng này".

### Kịch bản 2: Khóa tài khoản nhân viên đã nghỉ việc
* **Given:** Tài khoản `thukho_tuan` đang có quyền đăng nhập vào hệ thống.
* **When:** Quản lý chuyển trạng thái của `thukho_tuan` sang `Đã khóa`.
* **Then:** Nếu tài khoản đang đăng nhập sẽ bị tự động đăng xuất ra màn hình Login; các lần thử đăng nhập tiếp theo đều bị chặn với thông báo: "Tài khoản của bạn đã bị khóa. Vui lòng liên hệ Quản lý!".

### Kịch bản 3: Đặt lại mật khẩu (Reset Password)
* **Given:** Nhân viên bán hàng quên mật khẩu đăng nhập.
* **When:** Quản lý chọn tài khoản của nhân viên đó và bấm nút "Cấp lại mật khẩu ngẫu nhiên".
* **Then:** Hệ thống sinh mật khẩu ngẫu nhiên an toàn, hiển thị cho Quản lý bàn giao cho nhân viên và đánh dấu cờ `MustChangePasswordOnNextLogin = true` (yêu cầu đổi mật khẩu ngay lần đăng nhập đầu tiên).

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `TAI_KHOAN`:
  * `MaNV` (VARCHAR(50), Primary Key)
  * `TenDangNhap` (VARCHAR(50), Unique, Not Null)
  * `MatKhauHash` (VARCHAR(255), Not Null)
  * `HoTen` (NVARCHAR(100), Not Null)
  * `SoDienThoai` (VARCHAR(20))
  * `Email` (VARCHAR(100))
  * `VaiTro` (VARCHAR(20): `PHARMACIST`, `WAREHOUSE`, `MANAGER`)
  * `TrangThai` (TINYINT: 1 - Hoạt động, 0 - Đã khóa)
  * `NgayTao` (DATETIME, Default GETDATE())
  * `YeuCauDoiMK` (BIT, Default 1)
* Bảng `NHAT_KY_DANG_NHAP`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaNV` (VARCHAR(50), Foreign Key)
  * `ThoiGian` (DATETIME, Default GETDATE())
  * `IPAddress` (VARCHAR(50))
  * `UserAgent` (NVARCHAR(255))
  * `TrangThai` (VARCHAR(20): `THANH_CONG`, `THAT_BAI`)

### 6.2. Cơ chế bảo vệ API (Backend Middleware / Guard)
```csharp
[Authorize(Roles = "MANAGER")]
[HttpGet("api/reports/profit")]
public async Task<IActionResult> GetProfitReport([FromQuery] ReportFilterDto filter)
{
    var data = await _reportService.CalculateProfitAsync(filter);
    return Ok(data);
}
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:** Khởi tạo tài khoản Quản trị viên mặc định (`admin` / `manager`) khi triển khai hệ thống (Seeding database).
* **Bảo vệ toàn diện cho:**
  * Tất cả các User Stories từ **US01** đến **US14** đều được kiểm soát quyền truy cập dựa trên cấu hình vai trò của **US15**.
