# USER STORY: US10 - CHỌN LÔ THỦ CÔNG (OVERRIDE FEFO MANUAL BATCH SELECTION)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US10** |
| **Tên User Story** | Chọn Lô Thủ công (Ghi đè gợi ý phân bổ FEFO) |
| **Phân hệ (Module)** | Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO |
| **Use Case liên quan** | **UC10: Chọn Lô Thủ công (Override FEFO)** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-POS-03] Chọn Lô Thủ công (Override FEFO)**, **[NFR-SEC-02] Audit Trail** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Should-Have (Tính năng linh hoạt thực tế)** |
| **Tác nhân chính (Primary Actor)** | Dược sĩ / Nhân viên bán hàng (*Pharmacist/Sales Clerk*) |
| **Tác nhân phụ (Secondary Actor)** | Quản lý (*Theo dõi báo cáo can thiệp Lô*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Dược sĩ bán hàng tại quầy POS,
* **Tôi muốn (I want to):** Có khả năng chủ động can thiệp ghi đè gợi ý FEFO mặc định của hệ thống để tự tay chỉ định Lô thuốc cụ thể cần xuất bán và nhập lý do giải trình,
* **Để (So that):** Đáp ứng các yêu cầu đặc thù và chính đáng của khách hàng (như khách mua nguyên hộp yêu cầu cùng 1 số Lô đồng nhất, hoặc khách mua gửi đi xa yêu cầu Lô có hạn sử dụng dài hơn), đồng thời vẫn lưu vết kiểm toán minh bạch để Quản lý giám sát.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Mặc dù FEFO là nguyên tắc tối ưu cho nhà thuốc, thực tế bán hàng phát sinh các tình huống ngoại lệ:
1. **Khách hàng từ chối nhận 2 vỉ thuốc thuộc 2 số Lô khác nhau:** Hệ thống FEFO tự động tách bán 12 viên Lô A (hạn gần) và 8 viên Lô B (hạn xa). Khách mua cảm thấy lo ngại vì khác số Lô trên vỉ thuốc và yêu cầu lấy trọn vẹn 20 viên từ cùng Lô B.
2. **Khách mua thuốc đem ra nước ngoài / công tác dài ngày:** Khách yêu cầu mua Lô có HSD xa nhất thay vì Lô cận hạn.
Nếu phần mềm quá cứng nhắc sẽ làm mất lòng khách hoặc làm chậm trễ giao dịch. Dược sĩ cần được trao quyền Override, nhưng thao tác này phải được kiểm soát chặt chẽ bằng việc bắt buộc nhập lý do và lưu vào nhật ký kiểm toán (Audit Log).

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-POS-03.1] Giới hạn Lô được phép chọn thủ công:**
  * Dược sĩ chỉ được chọn các Lô còn hạn sử dụng ($HSD > NgayHienTai$).
  * Lô được chọn phải có số lượng tồn khả dụng $\ge$ số lượng Dược sĩ muốn xuất từ Lô đó.
  * Tuyệt đối không cho phép chọn Lô đã hết hạn hoặc Lô đang ở trạng thái Khóa/Chờ hủy.
* **[BR-POS-03.2] Bắt buộc ghi nhận lý do giải trình:** Khi kích hoạt Override FEFO, giao diện bắt buộc Dược sĩ chọn một lý do có sẵn (ví dụ: *Khách yêu cầu nguyên hộp đồng nhất Lô, Khách yêu cầu HSD xa, Lô cận hạn trên kệ bị lỗi bao bì đang chờ kiểm tra*) hoặc nhập lý do chi tiết.
* **[BR-POS-03.3] Nhật ký can thiệp (Audit Trail - NFR-SEC-02):** Hệ thống tự động lưu vào bảng `NHAT_KY_OVERRIDE_FEFO` gồm: `MaHD`, `MaThuoc`, `MaLoFEFOGoiY`, `MaLoThucTeChon`, `SoLuong`, `MaNhanVien`, `ThoiGian`, `LyDo`.
* **[BR-POS-03.4] Phân quyền:** Quản lý có thể cấu hình chính sách: Cho phép Dược sĩ tự Override HOẶC yêu cầu Quản trị viên nhập mã xác nhận (PIN) nếu muốn hạn chế việc tự ý bán vượt Lô.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Chọn lại Lô thủ công thành công kèm lý do (Happy Path)
* **Given:** Dược sĩ bán 20 Viên thuốc A. Hệ thống gợi ý FEFO lấy 12 viên từ Lô `LOT-01` (HSD: 11/2026) và 8 viên từ Lô `LOT-02` (HSD: 08/2027). Khách yêu cầu lấy toàn bộ 20 viên cùng Lô `LOT-02` (Lô `LOT-02` đang tồn 100 viên).
* **When:**
  1. Dược sĩ bấm vào biểu tượng "Chọn lại Lô" (hoặc phím tắt `Ctrl + L`).
  2. Bảng danh sách các Lô khả dụng hiển thị. Dược sĩ chọn Lô `LOT-02`, nhập số lượng: `20`.
  3. Chọn lý do: "Khách yêu cầu đồng nhất số Lô".
  4. Bấm "Xác nhận đổi Lô".
* **Then:**
  * Màn hình POS cập nhật chỉ dẫn phân bổ thành: `[Lô LOT-02: 20 viên]`.
  * Hóa đơn được thanh toán bình thường và trừ đủ 20 viên từ Lô `LOT-02`.
  * Bảng `NHAT_KY_OVERRIDE_FEFO` ghi nhận 1 bản ghi mới ghi rõ việc bỏ qua `LOT-01` để chọn `LOT-02`.

### Kịch bản 2: Báo lỗi khi cố tình chọn Lô không đủ số lượng tồn
* **Given:** Lô `LOT-03` chỉ còn tồn `10 Viên`.
* **When:** Dược sĩ cố gắng chỉ định xuất `20 Viên` hoàn toàn từ Lô `LOT-03`.
* **Then:** Hệ thống cảnh báo: "Lô được chọn chỉ còn tồn 10 Viên, không đủ số lượng yêu cầu (20 Viên). Vui lòng chọn Lô khác hoặc phân bổ thêm Lô!".

### Kịch bản 3: Không cho phép chọn Lô đã hết hạn sử dụng
* **Given:** Trong danh sách Lô của thuốc có Lô `LOT-EXPIRED` có HSD nhỏ hơn ngày hiện tại.
* **When:** Màn hình danh sách chọn Lô thủ công hiển thị.
* **Then:** Lô `LOT-EXPIRED` hoàn toàn bị ẩn hoặc bị vô hiệu hóa (disabled) với nhãn cảnh báo đỏ "Đã hết hạn", Dược sĩ không thể nhấp chọn.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `NHAT_KY_OVERRIDE_FEFO`:
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaHD` (VARCHAR(50), Foreign Key $\rightarrow$ `HOA_DON.MaHD`)
  * `MaThuoc` (VARCHAR(50), Foreign Key $\rightarrow$ `THUOC.MaThuoc`)
  * `MaLoGoiY` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `MaLoChon` (VARCHAR(50), Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `SoLuongGoc` (INT, Not Null)
  * `MaNhanVien` (VARCHAR(50), Foreign Key $\rightarrow$ `TAI_KHOAN.MaNV`)
  * `LyDo` (NVARCHAR(500), Not Null)
  * `ThoiGian` (DATETIME, Default GETDATE())

### 6.2. Yêu cầu giao diện (UI)
* Modal "Chọn Lô Xuất Hàng" hiển thị dạng bảng:
  * Cột 1: Mã Lô / Số Lô NSX
  * Cột 2: Hạn sử dụng (kèm tag màu: Đỏ cận hạn, Xanh an toàn)
  * Cột 3: Số lượng tồn gốc khả dụng
  * Cột 4: Số lượng muốn lấy từ Lô này
  * Cột 5: Lý do Override (Dropdown + Text field)

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US08, US09:** Đang trong luồng bán hàng POS và sau khi US09 đã sinh kết quả gợi ý phân bổ FEFO ban đầu.
* **Là tiền đề cho:**
  * **US14:** Báo cáo chi tiết can thiệp Lô giúp Quản lý kiểm tra nhân viên có tuân thủ quy chế bán hàng hay không.
