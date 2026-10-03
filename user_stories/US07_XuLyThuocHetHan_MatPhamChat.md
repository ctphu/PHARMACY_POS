# USER STORY: US07 - XỬ LÝ THUỐC HẾT HẠN & MẤT PHẨM CHẤT (EXPIRED & DAMAGED DRUGS DISPOSAL)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US07** |
| **Tên User Story** | Xử lý Thuốc Hết hạn / Mất phẩm chất (Xuất hủy & Cách ly) |
| **Phân hệ (Module)** | Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management) |
| **Use Case liên quan** | **UC07: Xử lý Thuốc Hết hạn / Mất phẩm chất** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-REP-01 Mức Đen] Khóa thuốc hết hạn & Xử lý hủy** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Tuân thủ Pháp chế Y tế & GPP)** |
| **Tác nhân chính (Primary Actor)** | Thủ kho (*Warehouse Specialist*) |
| **Tác nhân phụ (Secondary Actor)** | Quản lý / Chủ nhà thuốc (*Manager - Phê duyệt biên bản hủy*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Thủ kho và Quản lý nhà thuốc,
* **Tôi muốn (I want to):** Lập biên bản và phiếu xuất hủy đối với các Lô thuốc đã quá hạn sử dụng hoặc bị hư hỏng, vỡ, biến chất, đồng thời đưa số lượng tồn của Lô đó ra khỏi danh mục hàng hóa khả dụng,
* **Để (So that):** Đảm bảo an toàn tuyệt đối cho người bệnh, ngăn ngừa 100% rủi ro vô tình bán thuốc hết hạn, tuân thủ đúng quy chế chuyên môn Dược và tiêu chuẩn Thực hành tốt nhà thuốc (GPP).

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Theo quy định của Bộ Y tế và tiêu chuẩn GPP, các loại thuốc đã hết hạn sử dụng hoặc bị ẩm mốc, đổi màu, nứt vỡ bao bì bắt buộc phải được đưa vào khu vực biệt trữ/cách ly (Quarantine Area) và tiến hành tiêu hủy theo quy trình có biên bản chứng thực.
Phần mềm cần hỗ trợ:
1. Tự động chuyển các Lô có $HSD \le NgayHienTai$ sang trạng thái "Khóa bán/Chờ xử lý".
2. Cho phép Thủ kho gom các Lô này vào "Phiếu Xuất Hủy Thuốc".
3. Sau khi Quản lý ký duyệt điện tử, tồn kho của Lô đó được trừ triệt để về 0, chuyển trạng thái Lô thành "Đã xuất hủy" và hạch toán vào chi phí thiệt hại do thuốc hết hạn.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-INV-07.1] Khóa bán tự động đối với Lô hết hạn:** Bất kỳ Lô thuốc nào có $HSD \le NgayHienTai$ phải bị loại bỏ ngay lập tức khỏi mọi luồng bán lẻ POS (kể cả chọn FEFO tự động lẫn can thiệp chọn Lô thủ công Override).
* **[BR-INV-07.2] Quy trình 2 bước (Lập phiếu $\rightarrow$ Phê duyệt):**
  * Thủ kho lập phiếu xuất hủy, chọn các Lô cần hủy, ghi rõ lý do chi tiết cho từng Lô.
  * Quản lý / Chủ nhà thuốc xem xét và bấm "Duyệt xuất hủy".
* **[BR-INV-07.3] Tác động tồn kho:** Khi duyệt phiếu xuất hủy, số lượng tồn gốc của Lô trong `TON_KHO_LO` được trừ giảm tương ứng, trạng thái Lô trong `LO_HANG` chuyển sang `4 - Đã xuất hủy` nếu số tồn về 0.
* **[BR-INV-07.4] Không thể hoàn tác (Irreversible):** Phiếu xuất hủy sau khi Quản lý đã duyệt không thể hoàn tác hay xóa bỏ nhằm bảo đảm tính pháp lý và kiểm toán tài chính.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Lập và duyệt phiếu xuất hủy thuốc hết hạn thành công (Happy Path)
* **Given:** Lô `LOT-EXPIRED-01` của thuốc `Siro Ho` có HSD là `01/09/2026` (đã hết hạn), số tồn hiện tại là `15 Chai`. Trạng thái Lô là "Khóa bán / Chờ hủy".
* **When:**
  1. Thủ kho mở màn hình "Xử lý Thuốc Hết hạn / Mất phẩm chất", chọn Lô `LOT-EXPIRED-01`.
  2. Nhập hình thức xử lý: "Xuất hủy rác thải y tế", số lượng hủy: `15 Chai`.
  3. Bấm "Tạo phiếu xuất hủy và gửi duyệt".
  4. Quản lý mở phiếu và bấm "Phê duyệt biên bản tiêu hủy".
* **Then:**
  * Hệ thống trừ tồn kho Lô `LOT-EXPIRED-01` từ `15` về `0`.
  * Trạng thái Lô chuyển thành `Đã xuất hủy`.
  * Ghi vào sổ nhật ký kho với loại giao dịch `XUAT_HUY`.
  * Có nút in "Biên bản Tiêu hủy Dược phẩm" đúng biểu mẫu chuẩn.

### Kịch bản 2: Xuất hủy thuốc bị hư hỏng/mất phẩm chất trước hạn dùng
* **Given:** Lô `LOT-VALID-02` của thuốc `Kháng sinh Amox` có HSD đến năm 2028 (còn hạn), nhưng bị nước ngấm ẩm mốc `30 Viên`.
* **When:** Thủ kho chọn Lô này, nhập số lượng hủy `30 Viên`, lý do "Ẩm mốc rách vỉ bao bì", gửi Quản lý duyệt.
* **Then:** Sau khi duyệt, tồn kho Lô giảm đúng `30 Viên`, số lượng còn lại trong Lô (nếu có) vẫn giữ nguyên trạng thái khả dụng.

### Kịch bản 3: Thử nghiệm bán Lô đã hết hạn trên màn hình POS (Chặn tuyệt đối)
* **Given:** Lô `LOT-EXPIRED-01` đã hết hạn sử dụng.
* **When:** Dược sĩ cố tình tìm kiếm Lô này tại màn hình Bán lẻ POS hoặc Override chọn Lô.
* **Then:** Hệ thống không hiển thị Lô này trong danh sách chọn, hoặc báo lỗi đỏ nếu cố tình nhập mã Lô: "Lô thuốc đã hết hạn sử dụng, hệ thống từ chối xuất bán!".

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `PHIEU_XUAT_HUY`:
  * `MaPhieuHuy` (VARCHAR(50), Primary Key)
  * `NgayHuy` (DATETIME, Default GETDATE())
  * `NguoiLap` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `NguoiPheDuyet` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `HinhThucTieuHuy` (NVARCHAR(255))
  * `TongGiaTriHuy` (DECIMAL(18, 2))
  * `TrangThai` (VARCHAR(20): `CHO_DUYET`, `DA_HUY`, `TU_CHOI`)
* Bảng `CHI_TIET_PHIEU_HUY`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaPhieuHuy` (VARCHAR(50), Foreign Key)
  * `MaThuoc` (VARCHAR(50), Foreign Key)
  * `MaLo` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `SoLuongHuyGoc` (INT, Not Null, Check > 0)
  * `DonGiaVon` (DECIMAL(18, 2))
  * `LyDoHuy` (NVARCHAR(500), Not Null)

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US04, US05:** Có tồn kho Lô.
  * **US12:** Nhận diện các Lô thuộc Mức Đen từ tính năng quét cảnh báo HSD.
* **Là tiền đề cho các User Story:**
  * **US09:** Bảo vệ thuật toán FEFO không bao giờ quét trúng thuốc hỏng/hết hạn.
  * **US13:** Báo cáo tồn kho cập nhật chính xác hàng khả dụng.
  * **US14:** Hạch toán chi phí thiệt hại do xuất hủy vào báo cáo tài chính.
