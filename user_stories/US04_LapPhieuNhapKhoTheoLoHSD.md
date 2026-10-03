# USER STORY: US04 - LẬP PHIẾU NHẬP KHO THEO LÔ VÀ HẠN DÙNG (BATCH GOODS RECEIPT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US04** |
| **Tên User Story** | Lập Phiếu Nhập kho theo Lô & Hạn sử dụng |
| **Phân hệ (Module)** | Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management) |
| **Use Case liên quan** | **UC04: Lập Phiếu Nhập kho theo Lô/HSD** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-INV-01] Lập Phiếu Nhập kho theo Lô & HSD** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Cốt lõi kho hàng)** |
| **Tác nhân chính (Primary Actor)** | Thủ kho / Quản lý kho (*Warehouse Specialist*) |
| **Tác nhân phụ (Secondary Actor)** | Nhà cung cấp (*Supplier - Đối tác giao hàng*), Quản lý |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Thủ kho,
* **Tôi muốn (I want to):** Lập phiếu nhập kho ghi nhận chi tiết từng dòng thuốc kèm theo Số Lô sản xuất (Batch/Lot No), Ngày sản xuất (NSX), Hạn sử dụng (HSD), Đơn vị tính nhập và Đơn giá nhập từ Nhà cung cấp,
* **Để (So that):** Nhập hàng chính xác vào kho, thiết lập dữ liệu vòng đời sản phẩm phục vụ thuật toán phân bổ FEFO khi bán hàng, đồng thời đảm bảo khả năng truy xuất nguồn gốc lô thuốc khi có yêu cầu kiểm tra hoặc thu hồi của Bộ Y tế theo chuẩn GPP.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Khác với các ngành hàng thông thường (quản lý theo mã vạch SKU chung), ngành dược phẩm bắt buộc quản lý chi tiết đến từng Lô sản xuất. Mỗi lần nhập hàng từ nhà phân phối, một loại thuốc có thể có một hoặc nhiều số Lô khác nhau, mỗi Lô có một Hạn sử dụng riêng biệt.
Thủ kho khi nhận hàng thực tế phải đối soát số Lô và HSD in trên vỏ hộp/vỏ thùng với hóa đơn chứng từ giao hàng. Nếu số Lô đã tồn tại (lần trước nhập chưa hết, lần này nhập thêm cùng Lô), hệ thống sẽ ghi nhận cộng dồn. Nếu là Lô mới, hệ thống tự động sinh bản ghi Lô mới trong hệ thống.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-INV-01.1] Bắt buộc thông tin Lô:** Mọi dòng thuốc trong phiếu nhập bắt buộc phải có `Số Lô` và `Hạn sử dụng` hợp lệ.
* **[BR-INV-01.2] Quy tắc hạn sử dụng khi nhập kho:**
  * Thuốc đã hết hạn ($HSD \le NgayHienTai$): **Tuyệt đối chặn không cho nhập kho**.
  * Thuốc cận hạn ($NgayHienTai < HSD \le NgayHienTai + 30 \text{ ngày}$): Hệ thống cảnh báo đỏ và yêu cầu Quản lý xác nhận phê duyệt đặc biệt mới cho phép lưu.
  * Hạn sử dụng phải lớn hơn Ngày sản xuất ($HSD > NSX$).
* **[BR-INV-01.3] Xử lý trùng Số Lô:**
  * Nếu `SoLoNSX` của mặt hàng thuốc đó đã tồn tại trong bảng `LO_HANG` và trùng khớp thông tin `HSD`: Hệ thống liên kết dòng nhập với `MaLo` hiện có để cộng dồn tồn kho.
  * Nếu trùng `SoLoNSX` nhưng khác `HSD`: Hệ thống cảnh báo xung đột dữ liệu Lô để Thủ kho kiểm tra lại chứng từ thực tế.
* **[BR-INV-01.4] Đơn giá nhập:** Đơn giá nhập được ghi nhận theo ĐVT Nhập, dùng để tính toán giá trị kho và phục vụ tính giá vốn hàng bán (COGS) tại US14.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Nhập kho Lô mới thành công (Happy Path)
* **Given:** Thủ kho đang mở màn hình "Lập Phiếu Nhập Kho", đã chọn Nhà cung cấp "Công ty Dược Hậu Giang".
* **When:** Thủ kho thêm dòng hàng:
  * Thuốc: `Paracetamol 500mg`
  * Số Lô: `LOT-202610-01`
  * Ngày sản xuất: `01/10/2026`
  * Hạn sử dụng: `01/10/2028` (còn 2 năm)
  * Số lượng nhập: `50`
  * ĐVT Nhập: `Hộp` (1 Hộp = 100 Viên)
  * Đơn giá nhập: `120.000 VNĐ / Hộp`
  Và nhấn "Hoàn tất phiếu nhập".
* **Then:** Hệ thống tạo phiếu nhập kho, sinh mã Lô mới `LO_PARA500_01`, tạo bản ghi tồn kho Lô với số lượng tồn gốc quy đổi là `5.000 Viên` (thực hiện qua US05), trạng thái phiếu nhập chuyển sang "Đã hoàn thành".

### Kịch bản 2: Chặn nhập thuốc đã hết hạn sử dụng (Error Handling)
* **Given:** Thủ kho đang nhập thông tin dòng thuốc.
* **When:** Thủ kho nhập HSD là `01/01/2026` (nhỏ hơn ngày hiện tại `03/10/2026`).
* **Then:** Hệ thống hiển thị thông báo lỗi màu đỏ: "Hạn sử dụng không hợp lệ. Không thể nhập thuốc đã hết hạn vào kho!". Nút lưu phiếu bị vô hiệu hóa.

### Kịch bản 3: Nhập bổ sung số lượng vào Lô đã có trong kho
* **Given:** Trong kho đã có Lô `LOT-202605-A` của thuốc `Amoxicillin 500mg` với HSD `15/05/2027`, tồn kho hiện tại là `200 Viên`.
* **When:** Thủ kho lập phiếu nhập hàng mới và nhập đúng Số Lô `LOT-202605-A` và HSD `15/05/2027` với số lượng `300 Viên`.
* **Then:** Hệ thống nhận diện đây là Lô cũ, không tạo mã Lô mới mà cập nhật số tồn kho của Lô đó thành $200 + 300 = 500 \text{ Viên}$.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `PHIEU_NHAP_KHO`:
  * `MaPhieuNhap` (VARCHAR(50), Primary Key)
  * `NgayNhap` (DATETIME, Not Null, Default GETDATE())
  * `MaNhaCungCap` (VARCHAR(50), Foreign Key)
  * `NguoiNhap` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `TongTien` (DECIMAL(18, 2))
  * `GhiChu` (NVARCHAR(500))
* Bảng `CHI_TIET_PHIEU_NHAP`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaPhieuNhap` (VARCHAR(50), Foreign Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `MaLo` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `SoLuongNhap` (INT, Not Null, Check > 0)
  * `MaDVTNhap` (VARCHAR(20), Foreign Key)
  * `DonGiaNhap` (DECIMAL(18, 2), Not Null)
  * `SoLuongGocNhap` (INT, Not Null)
* Bảng `LO_HANG`:
  * `MaLo` (VARCHAR(50), Primary Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `SoLoNSX` (VARCHAR(100), Not Null)
  * `NgaySanXuat` (DATE)
  * `HanSuDung` (DATE, Not Null)
  * `TrangThai` (TINYINT: 1 - Khả dụng, 2 - Cận hạn, 3 - Hết hạn/Khóa, 4 - Đã hủy)

### 6.2. Toàn vẹn giao dịch
Việc tạo phiếu nhập, tạo/cập nhật thông tin Lô và cộng tồn kho phải được thực thi trong một Database Transaction duy nhất (ACID).

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US01:** Đã có danh mục thuốc.
  * **US02:** Đã cấu hình ĐVT quy đổi nếu nhập bằng đơn vị lớn hơn ĐVT Gốc.
* **Là tiền đề cho các User Story:**
  * **US05:** Quy đổi Số lượng Nhập về ĐVT Gốc.
  * **US09:** Tự động Gợi ý & Trừ Kho theo FEFO (cần dữ liệu Lô và HSD).
  * **US12:** Quét & Cảnh báo Thuốc Cận hạn / Hết hạn.
  * **US13:** Báo cáo Tồn kho chi tiết theo Lô & ĐVT.
