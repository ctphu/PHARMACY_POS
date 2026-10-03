# USER STORY: US14 - BÁO CÁO DOANH THU & LỢI NHUẬN BÁN LẺ (SALES & PROFIT REPORT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US14** |
| **Tên User Story** | Xem Báo cáo Doanh thu & Lợi nhuận Bán lẻ |
| **Phân hệ (Module)** | Phân hệ 4: Cảnh báo, Báo cáo & Quản trị (Alerts, Reports & Admin) |
| **Use Case liên quan** | **UC14: Xem Báo cáo Doanh thu & Lợi nhuận** (DacTa.md) |
| **Yêu cầu SRS** | **[Phân hệ 4.1 & Báo cáo Tài chính] Doanh thu, Giá vốn & Lợi nhuận gộp** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Hiệu quả kinh doanh & Quyết định tài chính)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Kế toán nhà thuốc (*Accountant*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Chủ nhà thuốc / Quản lý,
* **Tôi muốn (I want to):** Xem các báo cáo tổng hợp và trực quan về Doanh số bán hàng, Giá vốn hàng bán (COGS - tính chính xác theo từng Lô được xuất bán) và Lợi nhuận gộp theo từng ca làm việc, từng ngày, tháng hoặc theo từng mặt hàng thuốc,
* **Để (So that):** Tôi nắm bắt tức thì tình hình sức khỏe tài chính của nhà thuốc, đánh giá hiệu quả kinh doanh của từng nhân viên và từng nhóm sản phẩm, phát hiện kịp thời các mặt hàng mang lại biên lợi nhuận cao hoặc thua lỗ để tối ưu chiến lược nhập hàng và định giá.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Tại các hiệu thuốc, giá nhập của cùng một loại thuốc giữa các lô sản xuất khác nhau thường có sự biến động (do lạm phát, thay đổi tỷ giá hoặc chính sách khuyến mãi của nhà sản xuất).
Hầu hết các phần mềm thông thường chỉ tính giá vốn theo phương pháp bình quân gia quyền sơ sài, dẫn đến sai lệch biên lợi nhuận thực tế.
Hệ thống Pharmacy POS này khắc phục triệt để bằng cơ chế **"Tính Giá Vốn Theo Đúng Lô Thực Xuất" (Specific Identification Method via FEFO Traceability)**:
* Khi US09 xuất bán 12 viên từ Lô A (giá nhập 1.000đ/viên) và 8 viên từ Lô B (giá nhập 1.200đ/viên), hệ thống lưu rõ `MaLo` vào `CHI_TIET_HOA_DON`.
* Doanh thu = $20 \times 2.000đ = 40.000đ$.
* Giá vốn = $(12 \times 1.000đ) + (8 \times 1.200đ) = 12.000đ + 9.600đ = 21.600đ$.
* Lợi nhuận gộp chính xác từng đồng = $40.000đ - 21.600đ = 18.400đ$.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-REP-14.1] Công thức tính toán tài chính:**
  * **Doanh thu thuần:**
    $$\text{DoanhThuThuan} = \sum \text{TongTienHoaDon} - \sum \text{TienTraHang}$$
  * **Giá vốn hàng bán (COGS):**
    $$\text{GiaVon} = \sum_{i} \left( SoLuongGocXuat_i \times DonGiaNhapGoc(MaLo_i) \right) - \text{GiaVonHangTraLai}$$
  * **Lợi nhuận gộp (Gross Profit):**
    $$\text{LoiNhuanGop} = \text{DoanhThuThuan} - \text{GiaVon} - \text{ThietHaiXuatHuy}$$
  * **Tỷ suất lợi nhuận (Margin %):**
    $$\text{TySuatLoiNhuan} = \frac{\text{LoiNhuanGop}}{\text{DoanhThuThuan}} \times 100\%$$
* **[BR-REP-14.2] Phân quyền bảo mật cao (NFR-SEC-01):** Chỉ có tài khoản Quản lý / Chủ nhà thuốc mới có quyền truy cập module này. Dược sĩ bán hàng tại quầy POS hoàn toàn bị chặn và không nhìn thấy doanh số tổng hay giá vốn nhập hàng.
* **[BR-REP-14.3] Bộ lọc đa chiều:** Hỗ trợ lọc theo:
  * Khoảng thời gian (Hôm nay, Hôm qua, 7 ngày qua, Tháng này, Tùy chọn).
  * Ca làm việc / Nhân viên lập hóa đơn.
  * Phương thức thanh toán (Tiền mặt, Chuyển khoản QR, Thẻ).
  * Nhóm thuốc / Danh mục.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Xem báo cáo tổng quan doanh thu và lợi nhuận trong ngày (Happy Path)
* **Given:** Trong ngày hôm nay, nhà thuốc phát sinh:
  * 50 hóa đơn bán lẻ với tổng tiền bán: `15.000.000 VNĐ`.
  * Tổng giá vốn của các Lô xuất bán: `10.500.000 VNĐ`.
  * 1 phiếu trả hàng với số tiền hoàn: `200.000 VNĐ` (giá vốn hàng trả: `140.000 VNĐ`).
  * Không phát sinh phiếu xuất hủy.
* **When:** Quản lý mở "Báo cáo Doanh thu & Lợi nhuận", chọn khoảng thời gian "Hôm nay".
* **Then:** Hệ thống hiển thị:
  * Doanh thu thuần: $15.000.000 - 200.000 = 14.800.000 \text{ VNĐ}$.
  * Giá vốn hàng bán: $10.500.000 - 140.000 = 10.360.000 \text{ VNĐ}$.
  * Lợi nhuận gộp: $14.800.000 - 10.360.000 = 4.440.000 \text{ VNĐ}$.
  * Tỷ suất biên lợi nhuận: $\approx 30.0\%$.

### Kịch bản 2: Báo cáo chi tiết theo từng dòng thuốc và Lô đã bán
* **Given:** Quản lý muốn kiểm tra hiệu quả của thuốc `Panadol Extra`.
* **When:** Quản lý nhấp vào xem chi tiết của thuốc này trong báo cáo.
* **Then:** Hệ thống liệt kê chi tiết: số lượng đã bán (theo từng ĐVT và quy đổi về Viên), danh sách các Lô đã được xuất bán, đơn giá nhập của từng Lô, doanh thu mang lại và lợi nhuận gộp cụ thể.

### Kịch bản 3: Biểu đồ trực quan và xuất báo cáo
* **Given:** Quản lý đang xem báo cáo tháng.
* **When:** Quản lý xem phần biểu đồ (Chart).
* **Then:** Hệ thống hiển thị biểu đồ cột so sánh Doanh thu vs Giá vốn theo từng ngày trong tháng, và biểu đồ tròn thể hiện cơ cấu doanh thu theo nhóm thuốc; có nút tải file Excel báo cáo chi tiết.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Câu lệnh truy vấn mẫu tính Doanh thu & Giá vốn theo Lô
```sql
SELECT 
    T.MaThuoc,
    T.TenThuoc,
    SUM(CTHD.SoLuongGocXuat) AS TongSoLuongGocBan,
    SUM(CTHD.ThanhTien) AS TongDoanhThu,
    SUM(CTHD.SoLuongGocXuat * (CTPN.DonGiaNhap / CTPN.SoLuongGocNhap)) AS TongGiaVon,
    SUM(CTHD.ThanhTien) - SUM(CTHD.SoLuongGocXuat * (CTPN.DonGiaNhap / CTPN.SoLuongGocNhap)) AS LoiNhuanGop
FROM CHI_TIET_HOA_DON CTHD
INNER JOIN HOA_DON HD ON CTHD.MaHD = HD.MaHD
INNER JOIN THUOC T ON CTHD.MaThuoc = T.MaThuoc
INNER JOIN CHI_TIET_PHIEU_NHAP CTPN ON CTHD.MaLo = CTPN.MaLo AND CTHD.MaThuoc = CTPN.MaThuoc
WHERE HD.TrangThai = 'DA_THANH_TOAN'
  AND HD.NgayLap BETWEEN @TuNgay AND @DenNgay
GROUP BY T.MaThuoc, T.TenThuoc
ORDER BY TongDoanhThu DESC;
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US04:** Có dữ liệu đơn giá nhập kho của từng Lô.
  * **US08, US09:** Có hóa đơn bán lẻ đã thanh toán và gắn kết đúng `MaLo`.
  * **US11:** Có dữ liệu đổi trả hàng để cấn trừ doanh thu.
  * **US07:** Có dữ liệu xuất hủy thuốc để tính hao hụt chi phí.
