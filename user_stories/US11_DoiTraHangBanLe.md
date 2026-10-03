# USER STORY: US11 - ĐỔI TRẢ HÀNG BÁN LẺ VÀ HOÀN TỒN THEO LÔ (SALES RETURN MANAGEMENT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US11** |
| **Tên User Story** | Xử lý Đổi / Trả Hàng Bán lẻ và Hoàn Tồn Kho Đúng Lô |
| **Phân hệ (Module)** | Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO |
| **Use Case liên quan** | **UC11: Đổi / Trả hàng Bán lẻ** (DacTa.md) |
| **Yêu cầu SRS** | **[Phân hệ 3.3 & Mô hình Dữ liệu] Xử lý trả hàng đối soát theo Lô** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Should-Have (Cần thiết cho vận hành POS thực tế)** |
| **Tác nhân chính (Primary Actor)** | Dược sĩ / Nhân viên bán hàng (*Pharmacist/Sales Clerk*) |
| **Tác nhân phụ (Secondary Actor)** | Khách hàng (*Customer*), Quản lý |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Dược sĩ bán hàng tại quầy POS,
* **Tôi muốn (I want to):** Tra cứu hóa đơn mua hàng cũ của khách, thực hiện lập phiếu trả hàng hoặc đổi hàng cho một phần hoặc toàn bộ số thuốc đã mua, và hệ thống tự động hoàn lại số lượng tồn kho đúng vào chính Lô thuốc đã xuất bán trước đó,
* **Để (So that):** Đảm bảo quyền lợi chính đáng của người mua thuốc khi có sai sót đơn hoặc dị ứng, đồng thời số liệu tồn kho theo Lô được phục hồi chính xác 100%, tránh trường hợp nhập bù nhầm sang Lô khác gây sai lệch hạn dùng.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Trong bán lẻ dược phẩm, việc trả hàng đòi hỏi độ khắt khe cao hơn các mặt hàng thông thường:
1. Thuốc phải còn nguyên bao bì vỉ/hộp, không bị biến dạng và còn trong thời gian cho phép đổi trả theo chính sách nhà thuốc (thường trong vòng 24h - 48h kèm hóa đơn).
2. Khi khách trả lại 1 vỉ thuốc, Dược sĩ không thể cộng bừa vào kho chung, mà hệ thống phải truy vấn lại hóa đơn gốc: *Vỉ thuốc này ngày hôm qua được xuất từ Lô nào?*.
3. Hệ thống phải hoàn đúng số lượng gốc vào Lô đó trong bảng `TON_KHO_LO` để đảm bảo khi lô đó được bán tiếp cho khách sau, thông tin Lô và HSD trên bao bì thực tế vẫn khớp với dữ liệu phần mềm.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-POS-11.1] Bắt buộc chứng từ gốc:** Giao dịch trả hàng bắt buộc phải gắn liền với một Hóa đơn bán lẻ hợp lệ (`MaHD`) đã thanh toán thành công.
* **[BR-POS-11.2] Hoàn đúng Lô xuất gốc:** Số lượng thuốc trả lại phải được quy đổi về ĐVT Gốc và cộng hoàn trả chính xác vào `MaLo` đã được ghi nhận trong `CHI_TIET_HOA_DON` của hóa đơn gốc.
* **[BR-POS-11.3] Giới hạn số lượng trả:**
  $$0 < SoLuongTra \le (SoLuongDaMua - SoLuongDaTraTruocDo)$$
  Hệ thống chặn không cho phép nhập số lượng trả vượt quá số lượng khách thực tế đã mua.
* **[BR-POS-11.4] Kiểm soát thời hạn trả hàng:** Cho phép cấu hình số ngày tối đa được đổi trả (ví dụ: 3 ngày kể từ ngày in hóa đơn). Nếu quá hạn, hệ thống yêu cầu Quản lý phê duyệt mới được tiếp tục.
* **[BR-POS-11.5] Cấn trừ tài chính:** Tiền hoàn trả cho khách được tính theo đơn giá thực tế khách đã trả trên hóa đơn cũ (đã trừ chiết khấu nếu có).

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Trả lại thuốc và hoàn đúng Lô tồn kho (Happy Path)
* **Given:** Hóa đơn `HD00123` ngày hôm qua thể hiện khách mua:
  * Thuốc `Panadol`: 1 Vỉ (10 viên), xuất từ Lô `LOT-01` (HSD: 10/2026), đơn giá 15.000đ.
  * Hiện tại Lô `LOT-01` đang có tồn kho là `40 Viên`.
* **When:**
  1. Dược sĩ tra cứu hóa đơn `HD00123`, chọn trả hàng cho sản phẩm `Panadol`.
  2. Chọn số lượng trả: `1 Vỉ`, lý do "Bác sĩ đổi đơn".
  3. Bấm "Xác nhận trả hàng và hoàn tiền".
* **Then:**
  * Hệ thống tạo phiếu trả hàng `TH00045` với số tiền hoàn `15.000 VNĐ`.
  * Tồn kho của Lô `LOT-01` trong `TON_KHO_LO` tự động tăng thêm `10 Viên` (từ 40 thành `50 Viên`).
  * Ghi vào sổ nhật ký kho 1 dòng hoàn hàng gắn với phiếu `TH00045`.
  * In biên lai trả tiền cho khách.

### Kịch bản 2: Trả một phần thuốc thuộc hóa đơn được phân bổ nhiều Lô
* **Given:** Hóa đơn `HD00456` bán 20 viên thuốc X, được FEFO phân bổ:
  * 12 viên từ Lô A.
  * 8 viên từ Lô B.
* **When:** Khách mang trả lại `15 Viên`.
* **Then:** Hệ thống hiển thị giao diện đối soát cho phép Dược sĩ kiểm tra thực tế trên vỉ thuốc trả lại xem thuộc Lô nào (ví dụ: trả 12 viên Lô A và 3 viên Lô B) để hoàn đúng số lượng vào từng Lô tương ứng.

### Kịch bản 3: Báo lỗi khi trả vượt quá số lượng đã mua
* **Given:** Khách mua 1 Hộp (100 viên).
* **When:** Dược sĩ nhập số lượng trả là `150 Viên` (hoặc khách đã trả 50 viên trước đó, nay cố trả tiếp 60 viên).
* **Then:** Hệ thống cảnh báo đỏ: "Số lượng trả vượt quá số lượng khả dụng của hóa đơn gốc. Số lượng tối đa có thể trả: 50 viên!".

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `PHIEU_TRA_HANG`:
  * `MaPhieuTra` (VARCHAR(50), Primary Key)
  * `MaHDGoc` (VARCHAR(50), Foreign Key $\rightarrow$ `HOA_DON.MaHD`)
  * `NgayTra` (DATETIME, Default GETDATE())
  * `NhanVienNhan` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `TongTienHoan` (DECIMAL(18, 2), Not Null)
  * `LyDoTra` (NVARCHAR(500))
* Bảng `CHI_TIET_PHIEU_TRA`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaPhieuTra` (VARCHAR(50), Foreign Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `MaLo` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `MaDVT` (VARCHAR(20), Foreign Key)
  * `SoLuongTra` (INT, Not Null)
  * `DonGiaHoan` (DECIMAL(18, 2), Not Null)
  * `SoLuongGocHoan` (INT, Not Null)

### 6.2. Cập nhật tồn kho (Stock Restoration Query)
```sql
UPDATE TON_KHO_LO 
SET SoLuongTonGoc = SoLuongTonGoc + @SoLuongGocHoan,
    NgayCapNhat = GETDATE()
WHERE MaLo = @MaLoGoc;
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US08, US09:** Đã có hóa đơn bán lẻ phát sinh trước đó kèm chi tiết `MaLo`.
* **Là tiền đề cho:**
  * **US14:** Báo cáo Doanh thu & Lợi nhuận (tự động trừ khoản tiền và số lượng hàng bị trả lại).
