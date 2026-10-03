# USER STORY: US08 - LẬP HÓA ĐƠN BÁN LẺ TẠI QUẦY POS (POINT OF SALE CHECKOUT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US08** |
| **Tên User Story** | Lập Hóa đơn Bán lẻ tại Điểm bán (POS) |
| **Phân hệ (Module)** | Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO |
| **Use Case liên quan** | **UC08: Lập Hóa đơn Bán lẻ (POS)** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-POS-01] Chọn Thuốc & Đơn vị tính Bán**, **[UI 5.1/5.2] Giao diện POS & Thiết bị ngoại vi** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Cốt lõi hoạt động kinh doanh)** |
| **Tác nhân chính (Primary Actor)** | Dược sĩ / Nhân viên bán hàng (*Pharmacist/Sales Clerk*) |
| **Tác nhân phụ (Secondary Actor)** | Khách hàng (*Customer*), Cổng thanh toán Dynamic QR (*VietQR*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Dược sĩ / Nhân viên bán hàng tại quầy POS,
* **Tôi muốn (I want to):** Tra cứu thuốc nhanh bằng phím tắt hoặc máy quét mã vạch, linh hoạt chọn Đơn vị tính bán (Viên/Vỉ/Hộp) và nhập số lượng, hệ thống tự động tính thành tiền theo bảng giá tương ứng và in hóa đơn nhiệt/sinh mã thanh toán VietQR cho khách hàng,
* **Để (So that):** Thao tác bán hàng diễn ra nhanh chóng, chính xác trong vòng vài giây, giảm thiểu thời gian chờ đợi của khách hàng và hạn chế tối đa nhầm lẫn tính toán tiền bạc thủ công.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Tại quầy thuốc bán lẻ, tốc độ phục vụ là yếu tố sống còn, đặc biệt trong các giờ cao điểm. Dược sĩ cần:
1. Thao tác hoàn toàn bằng bàn phím (không cần dùng chuột) thông qua các phím tắt chuyên dụng (`F2` Tìm kiếm thuốc, `F8` Đổi ĐVT, `F9` Thanh toán).
2. Khi khách mua 2 Vỉ hoặc 1 Hộp, Dược sĩ chỉ cần bấm chọn ĐVT tương ứng, hệ thống sẽ lấy đúng đơn giá của ĐVT đó (được thiết lập ở US03) để nhân số lượng, thay vì bắt nhân viên tính nhẩm.
3. Hỗ trợ thanh toán đa kênh: Tiền mặt (tự tính tiền thối) hoặc quét mã VietQR tự động điền số tiền và nội dung chuyển khoản.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-POS-01.1] Công thức tính thành tiền từng dòng:**
  $$ThanhTien = SoLuongBan \times DonGiaBan(MaThuoc, MaDVT\_DuocChon)$$
* **[BR-POS-01.2] Tổng tiền hóa đơn:**
  $$TongTienHoaDon = \sum ThanhTien - ChietKhau + ThueVAT$$
* **[BR-POS-01.3] Không cho sửa giá bán tự do (NFR-SEC-01):** Dược sĩ bán lẻ không được phép can thiệp chỉnh sửa đơn giá bán trên màn hình POS nhằm ngăn ngừa gian lận giá. Mọi chiết khấu/giảm giá phải tuân theo chương trình khuyến mãi hoặc cần mã PIN của Quản lý.
* **[BR-POS-01.4] Tối ưu hiệu năng bàn phím (UI Shortcut):**
  * `F2`: Đặt con trỏ vào ô tìm kiếm thuốc / barcode.
  * `F4`: Nhập thông tin khách hàng (SĐT/Tên để tích điểm).
  * `F8`: Di chuyển con trỏ vào ô chọn ĐVT của dòng thuốc đang chọn.
  * `F9`: Mở hộp thoại thanh toán.
  * `Enter` tại ô thanh toán: Hoàn tất đơn hàng và kích hoạt lệnh in hóa đơn.
* **[BR-POS-01.5] Phản hồi tốc độ cao (NFR-PER-01):** Thời gian tính tiền và nạp hóa đơn phải dưới **1 giây** ($< 1s$).

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Thêm sản phẩm và tính tiền theo ĐVT quy đổi (Happy Path)
* **Given:** Thuốc `Paracetamol 500mg` có giá: 1 Viên = 2.000đ, 1 Vỉ = 18.000đ, 1 Hộp = 170.000đ.
* **When:**
  1. Dược sĩ nhấn `F2`, gõ `Para`, chọn thuốc `Paracetamol 500mg`.
  2. Dược sĩ nhấn `F8`, chọn ĐVT là `Vỉ`, nhập số lượng là `2`.
  3. Dược sĩ thêm tiếp sản phẩm 2: `Berberin`, ĐVT: `Hộp`, số lượng: `1`, giá `25.000đ`.
* **Then:**
  * Dòng 1 thành tiền hiển thị: $2 \times 18.000 = 36.000 \text{ VNĐ}$.
  * Dòng 2 thành tiền hiển thị: $1 \times 25.000 = 25.000 \text{ VNĐ}$.
  * Tổng tiền hóa đơn hiển thị chính xác: $36.000 + 25.000 = 61.000 \text{ VNĐ}$.

### Kịch bản 2: Thanh toán bằng mã Dynamic VietQR
* **Given:** Hóa đơn có tổng tiền là `61.000 VNĐ`.
* **When:** Dược sĩ nhấn `F9`, chọn phương thức "Chuyển khoản QR".
* **Then:** Màn hình POS (và màn hình phụ hướng về phía khách) hiển thị ngay lập tức một mã QR thanh toán chuẩn VietQR có sẵn số tiền `61000` và nội dung chuyển khoản chứa `MaHD`. Khi khách chuyển khoản thành công, hệ thống nhận tín hiệu webhook và tự động chốt hóa đơn.

### Kịch bản 3: Xử lý lưu tạm hóa đơn (Hold Bill / Multi-cart)
* **Given:** Dược sĩ đang nhập dở hóa đơn cho Khách A thì Khách A muốn đi lấy thêm sản phẩm khác.
* **When:** Dược sĩ bấm phím tắt `F6` "Lưu tạm đơn hàng" và mở tab đơn hàng mới phục vụ Khách B.
* **Then:** Đơn của Khách A được lưu trong danh sách hàng đợi; sau khi phục vụ Khách B xong, Dược sĩ bấm nạp lại đơn Khách A với đầy đủ các mặt hàng đã chọn trước đó.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `HOA_DON`:
  * `MaHD` (VARCHAR(50), Primary Key)
  * `NgayLap` (DATETIME, Default GETDATE())
  * `MaNhanVien` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `MaKhachHang` (VARCHAR(50), Nullable)
  * `TongTienHang` (DECIMAL(18, 2), Not Null)
  * `ChietKhau` (DECIMAL(18, 2), Default 0)
  * `TongThanhToan` (DECIMAL(18, 2), Not Null)
  * `PhuongThucTT` (VARCHAR(20): `TIEN_MAT`, `CHUYEN_KHOAN`, `THE`)
  * `TrangThai` (VARCHAR(20): `DA_THANH_TOAN`, `CHO_THANH_TOAN`, `HUY`)
* Bảng `CHI_TIET_HOA_DON`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaHD` (VARCHAR(50), Foreign Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `MaLo` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo` - Xác định bởi US09/US10)
  * `MaDVT` (VARCHAR(20), Foreign Key $\rightarrow$ `DON_VI_TINH.MaDVT`)
  * `SoLuongBan` (INT, Not Null)
  * `DonGiaBan` (DECIMAL(18, 2), Not Null)
  * `ThanhTien` (DECIMAL(18, 2), Not Null)
  * `SoLuongGocXuat` (INT, Not Null - Phục vụ trừ kho)

### 6.2. Tích hợp phần cứng
* Driver máy in hóa đơn ESC/POS khổ 80mm hoặc 58mm.
* Tương thích chuẩn máy quét mã vạch HID Keyboard Emulation.

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US01, US02, US03:** Đã có thuốc, đơn vị quy đổi và bảng giá.
* **Là tiền đề phối hợp chặt chẽ với:**
  * **US09:** Tự động Gợi ý & Trừ Kho theo FEFO (kích hoạt ngay trong lúc thêm sản phẩm hoặc khi bấm F9 Thanh toán).
  * **US10:** Cho phép Override chọn Lô thủ công ngay trên dòng sản phẩm POS.
  * **US11:** Đổi / Trả hàng (dựa trên MaHD đã tạo tại US08).
  * **US14:** Báo cáo Doanh thu & Lợi nhuận (tổng hợp từ bảng `HOA_DON`).
