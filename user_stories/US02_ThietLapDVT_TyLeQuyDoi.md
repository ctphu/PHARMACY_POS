# USER STORY: US02 - THIẾT LẬP ĐƠN VỊ TÍNH & TỶ LỆ QUY ĐỔI (UNIT CONVERSION CONFIGURATION)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US02** |
| **Tên User Story** | Thiết lập Đơn vị tính Quy đổi & Tỷ lệ Quy đổi |
| **Phân hệ (Module)** | Phân hệ 1: Quản lý Danh mục & Quy đổi Đơn vị tính (Master Data) |
| **Use Case liên quan** | **UC02: Thiết lập ĐVT & Tỷ lệ Quy đổi** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-MD-02] Thiết lập Đơn vị tính Quy đổi & Tỷ lệ Quy đổi** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Cốt lõi hệ thống)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Hệ thống Tự động (Tính toán và kiểm tra tính nhất quán) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Quản lý / Chủ nhà thuốc,
* **Tôi muốn (I want to):** Cấu hình các Đơn vị tính cấp cao hơn (ĐVT Quy đổi như Vỉ, Hộp, Chai, Thùng...) và tỷ lệ quy đổi tương ứng về Đơn vị tính Gốc (Base Unit) cho từng mặt hàng thuốc,
* **Để (So that):** Hệ thống có thể linh hoạt hỗ trợ nhập hàng theo đơn vị đóng gói lớn, bán lẻ theo bất kỳ đơn vị nào mà khách hàng mong muốn, đồng thời tự động chuẩn hóa mọi tính toán tồn kho về ĐVT Gốc mà không sợ thất thoát hay sai lệch.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Đặc thù của ngành bán lẻ dược phẩm là đóng gói đa cấp: Nhà thuốc nhập hàng từ Nhà phân phối theo Thùng hoặc Hộp lớn, nhưng khi phân phối cho bệnh nhân có thể bán theo từng Hộp, từng Vỉ, hoặc cắt liều theo từng Viên.
Hệ thống giải quyết bài toán này bằng cơ chế **"Cây Quy đổi Quy Nhất Về Gốc"**:
* Mọi đơn vị tính quy đổi đều có một hệ số nhân $k$ so với ĐVT Gốc.
* Ví dụ: Thuốc *Paracetamol 500mg*:
  * ĐVT Gốc: `Viên` ($k = 1$)
  * Cấp quy đổi 1: `Vỉ` ($1 \text{ Vỉ} = 10 \text{ Viên} \rightarrow k = 10$)
  * Cấp quy đổi 2: `Hộp` ($1 \text{ Hộp} = 10 \text{ Vỉ} = 100 \text{ Viên} \rightarrow k = 100$)
Dù người dùng khai báo bắc cầu (1 Hộp = 10 Vỉ), hệ thống lưu trữ trực tiếp tỷ lệ về ĐVT Gốc ($k = 100$) để tăng tốc độ truy vấn và tránh độ trễ đệ quy khi xử lý POS.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-MD-02.1] Công thức toán học:**
  $$1 \text{ ĐVT Quy đổi} = k \times \text{ĐVT Gốc} \quad (k \in \mathbb{N}^*, k > 1)$$
  Hệ số quy đổi $k$ bắt buộc phải là số nguyên dương lớn hơn 1 (không chấp nhận số thập phân để tránh lỗi làm tròn trong CSDL).
* **[BR-MD-02.2] Tính duy nhất của ĐVT trong cùng thuốc:** Trong cùng một mặt hàng thuốc, tên ĐVT quy đổi không được trùng với ĐVT Gốc và không được trùng với các ĐVT quy đổi khác đã cấu hình.
* **[BR-MD-02.3] Hỗ trợ khai báo trực tiếp hoặc bắc cầu:**
  * Khai báo trực tiếp: 1 Hộp = 100 Viên ($k = 100$).
  * Khai báo bắc cầu: 1 Hộp = 10 Vỉ. Hệ thống tự động tính: $k_{Hộp} = 10 \times k_{Vỉ} = 10 \times 10 = 100 \text{ Viên}$.
* **[BR-MD-02.4] Ràng buộc khi sửa/xóa ĐVT quy đổi:**
  * Nếu ĐVT quy đổi đã phát sinh giao dịch nhập kho hoặc bán hàng theo ĐVT đó, hệ thống không cho phép xóa, chỉ cho phép chỉnh sửa nếu không làm thay đổi lịch sử chứng từ đã chốt, hoặc cảnh báo nghiêm ngặt.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Cấu hình ĐVT quy đổi thành công (Happy Path)
* **Given:** Quản lý đã chọn thuốc `Paracetamol 500mg` có ĐVT Gốc là `Viên`.
* **When:** Quản lý thực hiện:
  1. Thêm ĐVT: Tên `Vỉ`, Tỷ lệ quy đổi: `10` so với `Viên`.
  2. Thêm ĐVT: Tên `Hộp`, Tỷ lệ quy đổi: `10` so với `Vỉ` (hoặc nhập `100` so với `Viên`).
  3. Nhấn nút "Lưu cấu hình quy đổi".
* **Then:** Hệ thống kiểm tra hợp lệ, lưu vào bảng `QUY_DOI_DVT` với $k_{Vỉ} = 10$ và $k_{Hộp} = 100$. Màn hình hiển thị danh sách trực quan:
  * 1 Vỉ = 10 Viên
  * 1 Hộp = 10 Vỉ = 100 Viên

### Kịch bản 2: Báo lỗi khi tỷ lệ quy đổi không hợp lệ (Validation)
* **Given:** Quản lý đang cấu hình ĐVT quy đổi cho thuốc.
* **When:** Quản lý nhập Tỷ lệ quy đổi là `0`, số âm `-5`, số thập phân `2.5` hoặc bằng `1`.
* **Then:** Hệ thống báo lỗi ngay tại trường nhập: "Tỷ lệ quy đổi phải là số nguyên dương lớn hơn 1". Nút "Lưu" bị vô hiệu hóa.

### Kịch bản 3: Báo lỗi khi trùng lặp đơn vị tính (Duplicate Unit)
* **Given:** Thuốc đã có ĐVT Gốc là `Viên` và đã có ĐVT quy đổi là `Vỉ`.
* **When:** Quản lý cố gắng thêm một ĐVT quy đổi mới có tên là `Viên` hoặc `Vỉ`.
* **Then:** Hệ thống hiển thị thông báo lỗi: "Đơn vị tính '[Tên ĐVT]' đã tồn tại cho thuốc này. Vui lòng chọn hoặc nhập đơn vị khác".

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `QUY_DOI_DVT`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaThuoc` (VARCHAR(50), Foreign Key $\rightarrow$ `THUOC.MaThuoc`)
  * `MaDVT` (VARCHAR(20), Foreign Key $\rightarrow$ `DON_VI_TINH.MaDVT`)
  * `TyLeQuyDoi` (INT, Not Null, Check Constraint: `TyLeQuyDoi > 1`)
  * `GiaBan` (DECIMAL(18, 2), Nullable - cấu hình tại US03)
  * Unique Constraint: `UNIQUE(MaThuoc, MaDVT)`

### 6.2. Kiểm tra ràng buộc CSDL
```sql
ALTER TABLE QUY_DOI_DVT 
ADD CONSTRAINT CK_TyLeQuyDoi_GreaterThanOne CHECK (TyLeQuyDoi > 1);

ALTER TABLE QUY_DOI_DVT 
ADD CONSTRAINT UQ_Thuoc_DVT UNIQUE (MaThuoc, MaDVT);
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:** Mặt hàng thuốc đã được tạo và có ĐVT Gốc (**US01**).
* **Là tiền đề cho các User Story:**
  * **US03:** Thiết lập Bảng giá theo ĐVT (cần có các ĐVT để đặt giá).
  * **US05:** Quy đổi Số lượng Nhập về ĐVT Gốc (sử dụng `TyLeQuyDoi`).
  * **US08 / US09:** Bán lẻ và Phân bổ FEFO (sử dụng `TyLeQuyDoi` để quy đổi số lượng bán về ĐVT gốc $S_{Cần\_Xuất}$).
  * **US13:** Báo cáo Tồn kho Đa quy đổi (sử dụng các cấp quy đổi để hiển thị dạng Hộp - Vỉ - Viên).
