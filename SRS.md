# TÀI LIỆU ĐẶC TẢ YÊU CẦU PHẦN MỀM (SRS)

## HỆ THỐNG QUẢN LÝ HIỆU THUỐC BÁN LẺ (PHARMACY POS & INVENTORY)
**Dự án:** Hệ thống Quản lý Hiệu thuốc Bán lẻ  
**Tập trung giải pháp:** Quản lý Lô - Hạn dùng (FEFO) & Quy đổi Đơn vị tính (ĐVT)  
**Phiên bản:** 1.0  
**Ngày lập:** 03/10/2026  
**Vai trò soạn thảo:** Senior Business Analyst (BA)

---

## 1. GIỚI THIỆU (INTRODUCTION)

### 1.1. Mục đích
Tài liệu SRS này mô tả chi tiết các yêu cầu chức năng, phi chức năng, quy tắc nghiệp vụ và giao diện hệ thống cho **Hệ thống Quản lý Hiệu thuốc Bán lẻ**. Tài liệu đóng vai trò làm căn cứ cho đội ngũ phát triển (Developers), kiểm thử (QA/QC), quản lý dự án (PM) và là sản phẩm phân tích chính trong môn Phân tích Thiết kế Hệ thống Thông tin.

### 1.2. Phạm vi hệ thống
Hệ thống là một phần mềm quản lý bán hàng (POS) và kho hàng dành cho các nhà thuốc bán lẻ, chuỗi nhà thuốc quy mô vừa và nhỏ. Phạm vi tập trung vào:
*   Quản lý danh mục thuốc và cấu hình quy đổi đơn vị tính đa cấp.
*   Quản lý kho chi tiết theo Lô (Batch/Lot) và Hạn sử dụng (HSD).
*   Gợi ý và tự động trừ kho xuất bán theo nguyên tắc **FEFO (First Expired, First Out)**.
*   Cảnh báo thuốc cận hạn/hết hạn và báo cáo doanh thu, tồn kho.

### 1.3. Định nghĩa và Viết tắt
*   **FEFO (First Expired, First Out):** Nguyên tắc ưu tiên xuất các lô hàng có hạn sử dụng gần nhất trước.
*   **ĐVT Gốc (Base Unit):** Đơn vị tính nhỏ nhất không thể chia nhỏ hơn của thuốc (ví dụ: Viên, Gói, Chai). Tất cả tính toán tồn kho trong CSDL đều quy về ĐVT Gốc.
*   **ĐVT Quy đổi (Conversion Unit):** Đơn vị tính đóng gói cấp cao hơn ĐVT gốc (ví dụ: Vỉ, Hộp, Thùng).
*   **POS (Point of Sale):** Điểm bán hàng lẻ tại quầy.
*   **SKU (Stock Keeping Unit):** Mã đơn vị quản lý kho.

---

## 2. MÔ TẢ TỔNG QUAN (OVERALL DESCRIPTION)

### 2.1. Góc nhìn sản phẩm (Product Perspective)
Hệ thống vận hành theo kiến trúc Web-based hoặc Desktop/Cloud client-server. Hệ thống tương tác trực tiếp với người dùng tại quầy, thủ kho tại kho và đồng bộ dữ liệu tập trung.

```
[Màn hình POS / Quản lý] <---> [Backend API / Business Logic] <---> [Database Server]
                                          |
                                          +---> [External: QR Payment / E-Invoice]
```

### 2.2. Đặc điểm Tác nhân Người dùng (User Characteristics)

| Tác nhân | Mức độ am hiểu CNTT | Tần suất sử dụng | Mục tiêu chính |
| :--- | :--- | :--- | :--- |
| **Dược sĩ / Nhân viên POS** | Trung bình | Liền tục | Thao tác bán hàng nhanh, tìm kiếm chính xác, không cần tính toán thủ công về quy đổi/HSD. |
| **Thủ kho** | Trung bình | Thường xuyên | Nhập kho chính xác số Lô/HSD, kiểm kê kho dễ dàng, nắm rõ hàng tồn cận hạn. |
| **Quản lý / Chủ nhà thuốc** | Khá | Thường xuyên | Quản lý giá, xem báo cáo doanh thu, cảnh báo lỗ do hết hạn, quản lý nhân sự. |

### 2.3. Ràng buộc Thiết kế và Cài đặt (Design & Implementation Constraints)
1.  **Ràng buộc toàn vẹn tồn kho:** Số lượng tồn kho trong cơ sở dữ liệu **bắt buộc** phải lưu trữ theo **ĐVT Gốc** và kiểu số nguyên dương ($SoLuong \in \mathbb{Z}^+$) để tránh sai số do chia thập phân.
2.  **Ràng buộc FEFO:** Hệ thống không cho phép xuất bán thuốc đã hết hạn sử dụng ($HanSuDung \le NgayHienTai$).
3.  **Tuân thủ Dược điển / GPP:** Đảm bảo khả năng truy xuất nguồn gốc Lô sản xuất khi có sự cố từ Nhà cung cấp hoặc Bộ Y tế yêu cầu thu hồi.

---

## 3. YÊU CẦU CHỨC NĂNG (FUNCTIONAL REQUIREMENTS)

### 3.1. Phân hệ 1: Quản lý Danh mục & Quy đổi Đơn vị tính (Master Data)

#### [FR-MD-01] Quản lý Danh mục Thuốc gốc
*   **Mô tả:** Hệ thống cho phép Quản lý thêm/sửa/xóa thông tin thuốc.
*   **Dữ liệu bắt buộc:** Mã thuốc, Tên thuốc, Tên hoạt chất, Hàm lượng, Nhà sản xuất, ĐVT Gốc.
*   **Quy tắc nghiệp vụ (BR-MD-01):**
    *   ĐVT Gốc là đơn vị nhỏ nhất (Ví dụ: "Viên"). Khi đã phát sinh giao dịch, **không được sửa** ĐVT Gốc.

#### [FR-MD-02] Thiết lập Đơn vị tính Quy đổi & Tỷ lệ Quy đổi
*   **Mô tả:** Khai báo các đơn vị tính cấp lớn hơn và tỷ lệ quy đổi tương ứng về ĐVT Gốc.
*   **Công thức:**
    $$1 \text{ ĐVT Quy đổi} = k \times \text{ĐVT Gốc} \quad (k \in \mathbb{N}^*, k > 1)$$
*   **Ví dụ:**
    *   ĐVT Gốc: `Viên` ($k=1$)
    *   ĐVT Quy đổi 1: `Vỉ` ($k=10 \rightarrow 1 \text{ Vỉ} = 10 \text{ Viên}$)
    *   ĐVT Quy đổi 2: `Hộp` ($k=100 \rightarrow 1 \text{ Hộp} = 100 \text{ Viên}$)
*   **Cấu trúc dữ liệu quy đổi:** Hệ thống hỗ trợ đa cấp nhưng khi lưu trữ sẽ tự động quy đổi đồng nhất về ĐVT Gốc.

#### [FR-MD-03] Thiết lập Giá bán theo từng ĐVT
*   **Mô tả:** Cho phép cài đặt Bảng giá linh hoạt cho từng ĐVT.
*   **Quy tắc nghiệp vụ (BR-MD-02):** Giá bán theo ĐVT lớn có thể thấp hơn tổng giá bán lẻ các ĐVT nhỏ cộng lại (Chính sách chiết khấu mua nguyên hộp/thùng).
    *   *Ví dụ:* 1 Viên = 2.000đ; 1 Vỉ (10 viên) = 18.000đ (Rẻ hơn 2.000đ).

---

### 3.2. Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management)

#### [FR-INV-01] Lập Phiếu Nhập kho theo Lô & HSD
*   **Mô tả:** Thủ kho nhập thông tin phiếu nhập từ Nhà cung cấp.
*   **Dữ liệu nhập từng dòng thuốc:** Mã Thuốc, Số Lô (Batch No), Ngày sản xuất, Hạn sử dụng, Số lượng nhập, ĐVT Nhập, Đơn giá nhập.
*   **Quy tắc nghiệp vụ (BR-INV-01):**
    *   Cùng 1 Mã Thuốc có thể nhập nhiều Lô khác nhau trong 1 hoặc nhiều lần nhập.
    *   Nếu Số Lô đã tồn tại cho Thuốc đó $\rightarrow$ Hệ thống cập nhật cộng dồn số lượng.
    *   Nếu Số Lô chưa tồn tại $\rightarrow$ Hệ thống tự động khởi tạo Lô mới.
    *   Hạn sử dụng bắt buộc phải lớn hơn Ngày hiện tại ít nhất 30 ngày (có thể cấu hình ngưỡng cảnh báo).

#### [FR-INV-02] Tự động Quy đổi Tồn kho khi Nhập
*   **Mô tả:** Khi Thủ kho nhập số lượng theo ĐVT Quy đổi (VD: Nhập 5 Hộp), hệ thống tự động quy đổi ra ĐVT Gốc để cộng vào kho.
*   **Công thức:**
    $$SoLuongGocTang = SoLuongNhap \times TyLeQuyDoi$$
    *(Ví dụ: 5 Hộp $\times$ 100 = 500 Viên).*

#### [FR-INV-03] Kiểm kê & Điều chỉnh Kho theo Lô
*   **Mô tả:** Cho phép kiểm kê số lượng thực tế của từng Lô cụ thể và cập nhật chênh lệch (Thừa/Thiếu/Hư hỏng).

---

### 3.3. Phân hệ 3: Bán hàng tại Điểm bán (POS & FEFO Logic)

#### [FR-POS-01] Chọn Thuốc & Đơn vị tính Bán
*   **Mô tả:** Dược sĩ gõ tên hoặc quét mã vạch chọn Thuốc. Chọn ĐVT muốn bán (Viên/Vỉ/Hộp) và nhập số lượng bán.
*   **Tính tiền:**
    $$ThanhTien = SoLuongBan \times GiaBan(DVT\_DuocChon)$$

#### [FR-POS-02] Tự động Trừ kho theo Thuật toán FEFO (Core Functional Requirement)
*   **Mô tả:** Khi bấm Thanh toán, hệ thống tự động xác định các Lô cần trừ tồn kho dựa trên nguyên tắc FEFO.
*   **Quy trình xử lý của Thuật toán FEFO:**
    1.  Chuyển đổi Tổng số lượng bán về ĐVT Gốc: $S_{Cần\_Xuất} = SoLuongBan \times TyLeQuyDoi$.
    2.  Truy vấn danh sách các Lô khả dụng của mặt hàng:
        $$\text{Danh sách Lô} = \{ L_i \mid L_i.MãThuốc = X \land L_i.TồnGốc > 0 \land L_i.HSD > NgàyHiệnTại \}$$
    3.  Sắp xếp danh sách Lô theo chiều tăng dần của Hạn sử dụng (`ORDER BY HSD ASC`).
    4.  Lặp qua từng Lô $L_i$ trong danh sách:
        *   Nếu $L_i.TồnGốc \ge S_{Cần\_Xuất}$:
            *   Trừ $L_i.TồnGốc = L_i.TồnGốc - S_{Cần\_Xuất}$.
            *   Ghi nhận $S_{Cần\_Xuất}$ xuất từ Lô $L_i$.
            *   Hoàn tất phân bổ ($S_{Cần\_Xuất} = 0$).
        *   Nếu $L_i.TồnGốc < S_{Cần\_Xuất}$:
            *   Trừ hết tồn Lô $L_i$: Ghi nhận $L_i.TồnGốc$ xuất từ Lô $L_i$.
            *   Cập nhật số lượng còn thiếu: $S_{Cần\_Xuất} = S_{Cần\_Xuất} - L_i.TồnGốc$.
            *   Gán $L_i.TồnGốc = 0$.
            *   Chuyển sang Lô kế tiếp $L_{i+1}$.
    5.  Nếu duyệt hết danh sách Lô mà $S_{Cần\_Xuất} > 0 \rightarrow$ Báo lỗi "Không đủ tồn kho khả dụng theo Lô".

#### [FR-POS-03] Chọn Lô Thủ công (Override FEFO)
*   **Mô tả:** Cho phép Dược sĩ can thiệp chọn Lô thủ công nếu khách hàng có yêu cầu đặc biệt (Ví dụ: Khách muốn mua đúng 1 Lô cụ thể để lấy nguyên hộp).
*   **Ràng buộc:** Phải có quyền cấu hình hoặc ghi lại Nhật ký can thiệp (Audit Log).

---

### 3.4. Phân hệ 4: Cảnh báo, Báo cáo & Quản trị

#### [FR-REP-01] Quét & Cảnh báo Thuốc Cận hạn / Hết hạn
*   **Mô tả:** Hệ thống chạy tiến trình quét ngầm hàng ngày (Cron job) để thống kê danh sách Lô sắp hết hạn.
*   **Phân loại ngưỡng cảnh báo:**
    *   *Mức Đỏ:* $HSD - NgayHienTai \le 30 \text{ ngày}$ (Cần xử lý gấp / Trả nhà cung cấp).
    *   *Mức Vàng:* $30 \text{ ngày} < HSD - NgayHienTai \le 90 \text{ ngày}$ (Ưu tiên đẩy bán / Khuyến mãi).
    *   *Mức Đen:* $HSD \le NgayHienTai$ (Khóa không cho bán, chuyển sang trạng thái chờ hủy).

#### [FR-REP-02] Báo cáo Tồn kho Đa quy đổi
*   **Mô tả:** Hiển thị tồn kho thông minh dưới dạng ĐVT Quy đổi để Quản lý dễ đọc.
*   **Ví dụ:** Thuốc Y tồn $255$ Viên gốc ($1 \text{ Hộp} = 100 \text{ Viên}$, $1 \text{ Vỉ} = 10 \text{ Viên}$).
    *   *Hiển thị:* **2 Hộp - 5 Vỉ - 5 Viên**.

---

## 4. YÊU CẦU PHI CHỨC NĂNG (NON-FUNCTIONAL REQUIREMENTS)

### 4.1. Hiệu năng (Performance)
*   **[NFR-PER-01] Thời gian phản hồi POS:** Thuật toán phân bổ FEFO và tính tiền tại màn hình POS phải hoàn thành trong thời gian dưới **1 giây** ($< 1s$) cho hóa đơn chứa tối đa 20 mặt hàng.
*   **[NFR-PER-02] Khả năng xử lý đồng thời:** Hệ thống đáp ứng tối thiểu 50 giao dịch POS cùng lúc mà không gây tắc nghẽn dữ liệu (Locking Issue) trên bảng Tồn Kho Lô.

### 4.2. An toàn & Bảo mật (Security & Audit)
*   **[NFR-SEC-01] Phân quyền truy cập (RBAC):** Nhân viên POS không có quyền sửa đơn giá, không được chỉnh sửa số lượng tồn kho thủ công.
*   **[NFR-SEC-02] Nhật ký giao dịch (Audit Trail):** Mọi thao tác xuất/nhập/điều chỉnh Lô hàng và Override FEFO đều phải ghi lại `User_ID`, `Timestamp`, `Số Lô`, `Lý do`.

### 4.3. Độ tin cậy & Lỗi (Reliability)
*   **[NFR-REL-01] Bàn giao giao dịch (Database Transaction):** Thao tác thanh toán hóa đơn bao gồm [Tạo Hóa đơn + Trừ Kho Lô + Ghi nhật ký] phải nằm trong 1 **ACID Transaction**. Nếu có lỗi ở bất kỳ bước nào, toàn bộ giao dịch phải Rollback.

---

## 5. YÊU CẦU GIAO DIỆN VÀ TÍCH HỢP (EXTERNAL INTERFACE REQUIREMENTS)

### 5.1. Giao diện Người dùng (UI Requirements)
*   **Màn hình POS Bán hàng:**
    *   Thiết kế tối ưu cho thao tác bàn phím (Phím tắt: `F2` Tìm kiếm, `F8` Chọn ĐVT, `F9` Thanh toán).
    *   Hiển thị rõ ràng tên Lô và HSD được gợi ý bên cạnh dòng sản phẩm bán.
*   **Màn hình Quản lý Lô:** Hiển thị mã màu cảnh báo HSD (Đỏ, Vàng, Xanh).

### 5.2. Giao diện Tích hợp (Hardware / External Integration)
*   Tích hợp Đầu đọc Mã vạch (Barcode Scanner) qua cổng USB/Bluetooth.
*   Tích hợp Máy in Hóa đơn nhiệt (Khổ 80mm hoặc 58mm).
*   Tích hợp Cổng thanh toán Dynamic QR (VietQR) để tự động sinh mã QR đúng số tiền.

---

## 6. MÔ HÌNH DỮ LIỆU CỐT LÕI (CONCEPTUAL DATA MODEL)

Để phục vụ bài toán FEFO và ĐVT, kiến trúc CSDL tối thiểu phải chứa các thực thể và quan hệ sau:

```
[THUOC] 1 --- * [QUY_DOI_DVT]
   1
   |
   *
[LO_HANG] 1 --- 1 [TON_KHO_LO]
   1
   |
   *
[CHI_TIET_HOA_DON]
```

### Chi tiết thuộc tính chính:
1.  **`THUOC`** (`MaThuoc`, `TenThuoc`, `MaDVTGoc`, `MoTa`)
2.  **`DON_VI_TINH`** (`MaDVT`, `TenDVT`)
3.  **`QUY_DOI_DVT`** (`MaThuoc`, `MaDVT`, `TyLeQuyDoi`, `GiaBan`)
4.  **`LO_HANG`** (`MaLo`, `MaThuoc`, `SoLoNSX`, `NgaySanXuat`, `HanSuDung`)
5.  **`TON_KHO_LO`** (`MaLo`, `SoLuongTonGoc`)
6.  **`HOA_DON`** (`MaHD`, `NgayLap`, `TongTien`, `MaNhanVien`)
7.  **`CHI_TIET_HOA_DON`** (`MaHD`, `MaThuoc`, `MaLo`, `MaDVT`, `SoLuongBan`, `DonGiaBan`, `SoLuongGocXuat`)

---

## 7. TIÊU CHÍ NGHỆM THU (ACCEPTANCE CRITERIA)

| STT | Kịch bản kiểm thử | Kết quả mong đợi |
| :--- | :--- | :--- |
| **TC01** | Bán 1 Vỉ (10 viên) của Thuốc A. Kho có Lô 1 (Tồn 5 viên, HSD: 05/2026) và Lô 2 (Tồn 50 viên, HSD: 12/2026). | Hệ thống tự động trừ: 5 viên từ Lô 1 (Lô 1 về 0) và 5 viên từ Lô 2 (Lô 2 còn 45 viên). |
| **TC02** | Thêm mới ĐVT "Hộp" cho Thuốc B với Tỷ lệ quy đổi = 100. Nhập kho 2 Hộp. | Tồn kho gốc của Thuốc B tăng chính xác **200** đơn vị gốc. |
| **TC03** | Chọn bán một Lô đã hết hạn sử dụng. | Hệ thống chặn giao dịch, báo lỗi "Thuốc trong Lô đã hết hạn không thể bán". |