# BẢN ĐẶC TẢ TÁC NHÂN VÀ USE CASE
## HỆ THỐNG QUẢN LÝ HIỆU THUỐC BÁN LẺ (PHARMACY POS & INVENTORY)
**Chủ đề tập trung:** Quản lý Lô - Hạn dùng (FEFO) & Quy đổi Đơn vị tính (ĐVT)

---

## 1. TỔNG QUAN HỆ THỐNG (SYSTEM OVERVIEW)

Hệ thống Quản lý Hiệu thuốc Bán lẻ hỗ trợ toàn bộ quá trình vận hành kinh doanh tại nhà thuốc, từ nhập hàng, quản lý kho, thiết lập giá bán đến bán lẻ tại điểm bán (POS). Hệ thống giải quyết hai bài toán nghiệp vụ quan trọng:
1. **Quy đổi Đơn vị tính linh hoạt:** Thuốc có thể nhập theo Hộp, bán theo Vỉ hoặc Viên. Hệ thống tự động quy đổi về **ĐVT gốc** (đơn vị nhỏ nhất) để quản lý tồn kho chính xác.
2. **Quản lý Lô – Hạn dùng theo nguyên tắc FEFO (First Expired, First Out):** Tự động ưu tiên xuất bán các lô thuốc có hạn sử dụng gần nhất, giúp giảm thiểu rủi ro thuốc hết hạn trong kho.

---

## 2. DANH SÁCH TÁC NHÂN (ACTORS)

| STT | Tên Tác nhân (Actor) | Loại Tác nhân | Mô tả vai trò & Trách nhiệm |
| :--- | :--- | :--- | :--- |
| **1** | **Dược sĩ / Nhân viên Bán hàng** *(Pharmacist/Sales Clerk)* | Human (Primary) | Thực hiện bán thuốc tại điểm bán (POS), tư vấn đơn thuốc, tra cứu thông tin Lô/HSD, lập hóa đơn, xử lý đổi trả thuốc cho khách hàng. |
| **2** | **Thủ kho / Quản lý Kho** *(Warehouse Specialist)* | Human (Primary) | Lập phiếu nhập kho, kiểm tra lô sản phẩm, gắn Ngày sản xuất/Hạn sử dụng, kiểm kê kho, xử lý thanh lý/hủy thuốc hết hạn. |
| **3** | **Chủ nhà thuốc / Quản lý** *(Pharmacy Owner/Manager)* | Human (Primary) | Quản lý danh mục thuốc, cấu hình ĐVT & Tỷ lệ quy đổi, cài đặt bảng giá, xem báo cáo doanh thu, báo cáo tồn kho/HSD, quản lý tài khoản. |
| **4** | **Khách hàng** *(Customer)* | Human (Secondary) | Người mua thuốc (tác nhân gián tiếp tương tác thông qua Dược sĩ hoặc nhận hóa đơn/thanh toán). |
| **5** | **Hệ thống Tự động (Cron Job / System Worker)** | System (Secondary) | Tiến hành chạy tự động ngầm: quét cảnh báo HSD sắp hết hạn, tự động phân bổ lô theo thuật toán FEFO khi bán hàng, tự động tính toán quy đổi tồn kho. |
| **6** | **Cổng Thanh toán / Hóa đơn điện tử** *(External System)* | System (Secondary) | Hệ thống bên thứ 3 tích hợp để xử lý thanh toán chuyển khoản/QR code và xuất hóa đơn điện tử (nếu có). |

---

## 3. MA TRẬN TÁC NHÂN VS USE CASE (ACTOR - USE CASE MATRIX)

| Mã UC | Tên Use Case | Dược sĩ | Thủ kho | Quản lý | Hệ thống Tự động |
| :--- | :--- | :---: | :---: | :---: | :---: |
| **UC01** | Quản lý Danh mục Thuốc | | | **X** | |
| **UC02** | Thiết lập ĐVT & Tỷ lệ Quy đổi | | | **X** | |
| **UC03** | Thiết lập Bảng giá theo ĐVT | | | **X** | |
| **UC04** | Lập Phiếu Nhập kho theo Lô/HSD | | **X** | | |
| **UC05** | Quy đổi Số lượng Nhập về ĐVT Gốc | | **X** | | **X** |
| **UC06** | Kiểm kê Kho & Điều chỉnh Tồn Lô | | **X** | **X** | |
| **UC07** | Xử lý Thuốc Hết hạn / Mất phẩm chất | | **X** | **X** | |
| **UC08** | Lập Hóa đơn Bán lẻ (POS) | **X** | | | |
| **UC09** | Tự động Gợi ý & Trừ Kho theo FEFO | **X** | | | **X** |
| **UC10** | Chọn Lô Thủ công (Override FEFO) | **X** | | | |
| **UC11** | Đổi / Trả hàng Bán lẻ | **X** | | | |
| **UC12** | Quét & Cảnh báo Thuốc Sắp Hết hạn | | **X** | **X** | **X** |
| **UC13** | Xem Báo cáo Tồn kho chi tiết theo Lô & ĐVT | | **X** | **X** | |
| **UC14** | Xem Báo cáo Doanh thu & Lợi nhuận | | | **X** | |
| **UC15** | Quản lý Tài khoản & Phân quyền | | | **X** | |

---

## 4. DANH SÁCH USE CASE CHI TIẾT PHÂN THEO PHÂN HỆ

### 4.1. Phân hệ 1: Quản lý Danh mục & Quy đổi Đơn vị tính (Master Data)
* **UC01: Quản lý Danh mục Thuốc**
  * *Mô tả:* Quản lý thông tin chi tiết của thuốc (Mã thuốc, Tên hoạt chất, Hàm lượng, Nhà sản xuất, ĐVT gốc).
* **UC02: Thiết lập ĐVT & Tỷ lệ Quy đổi**
  * *Mô tả:* Cho phép cấu hình các đơn vị tính cấp cao hơn dựa trên ĐVT gốc.
  * *Công thức cấu hình:* $1 \text{ ĐVT Quy Đổi} = N \times \text{ĐVT Gốc}$.
  * *Ví dụ:* ĐVT gốc = "Viên". Thiết lập: 1 "Vỉ" = 10 "Viên"; 1 "Hộp" = 10 "Vỉ" (tương đương 100 "Viên").
* **UC03: Thiết lập Bảng giá theo ĐVT**
  * *Mô tả:* Cấu hình giá bán riêng cho từng ĐVT. Khách mua lẻ 1 Hộp có thể rẻ hơn mua 10 Vỉ riêng lẻ.

### 4.2. Phân hệ 2: Quản lý Nhập kho & Quản lý Lô – Hạn dùng
* **UC04: Lập Phiếu Nhập kho theo Lô/HSD**
  * *Mô tả:* Thủ kho ghi nhận thông tin nhập từ Nhà cung cấp, bắt buộc nhập: Mã Lô (Batch/Lot No), Ngày sản xuất, Hạn sử dụng, Số lượng nhập và ĐVT nhập.
* **UC05: Quy đổi Số lượng Nhập về ĐVT Gốc**
  * *Mô tả:* Khi nhập kho theo "Hộp", hệ thống tự động nhân tỷ lệ quy đổi để ghi nhận chính xác số lượng tăng thêm trong cơ sở dữ liệu dưới ĐVT gốc.
* **UC06: Kiểm kê Kho & Điều chỉnh Tồn Lô**
  * *Mô tả:* Đối soát số lượng thực tế với số lượng trên hệ thống theo từng Lô cụ thể. Ghi nhận chênh lệch và cập nhật lý do.
* **UC07: Xử lý Thuốc Hết hạn / Mất phẩm chất**
  * *Mô tả:* Xuất hủy các Lô thuốc đã hết hạn sử dụng hoặc bị hư hỏng, loại bỏ khỏi kho khả dụng.

### 4.3. Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO
* **UC08: Lập Hóa đơn Bán lẻ**
  * *Mô tả:* Dược sĩ chọn Thuốc $\rightarrow$ Chọn ĐVT bán (Viên/Vỉ/Hộp) $\rightarrow$ Nhập số lượng. Hệ thống tự động tính tiền dựa theo Bảng giá của ĐVT tương ứng.
* **UC09: Tự động Gợi ý & Trừ Kho theo FEFO**
  * *Mô tả (Core Logic):*
    1. Thuốc được chọn mua quy đổi ra $S_{gốc}$ (Số lượng theo ĐVT gốc).
    2. Hệ thống truy vấn các Lô còn hàng ($Tồn > 0$) của thuốc đó, sắp xếp tăng dần theo Hạn sử dụng (`ORDER BY HanSuDung ASC`).
    3. Hệ thống tự động phân bổ $S_{gốc}$ trừ từ Lô có hạn gần nhất trước. Nếu Lô đó không đủ, tiếp tục trừ sang Lô kế tiếp.
* **UC10: Chọn Lô Thủ công (Override FEFO)**
  * *Mô tả:* Trong trường hợp đặc biệt (ví dụ: khách muốn mua nguyên hộp cùng một lô), Dược sĩ có quyền chọn lại Lô xuất hàng khác với gợi ý mặc định của hệ thống.
* **UC11: Đổi / Trả hàng Bán lẻ**
  * *Mô tả:* Xử lý trả lại thuốc từ khách hàng. Hệ thống yêu cầu xác định đúng Lô đã bán trên hóa đơn cũ để cộng lại tồn kho Lô tương ứng.

### 4.4. Phân hệ 4: Cảnh báo, Báo cáo & Hệ thống
* **UC12: Quét & Cảnh báo Thuốc Sắp Hết hạn**
  * *Mô tả:* Hệ thống định kỳ quét kho và đưa ra danh sách các Lô thuốc còn tồn nhưng có HSD nằm trong ngưỡng cảnh báo (ví dụ: còn dưới 30 ngày, 60 ngày, 90 ngày) để Quản lý có chính sách khuyến mãi hoặc trả nhà cung cấp.
* **UC13: Xem Báo cáo Tồn kho chi tiết theo Lô & ĐVT**
  * *Mô tả:* Cho phép lọc tồn kho theo từng Lô, từng HSD và hiển thị quy đổi đa dạng (Ví dụ: Tồn 250 viên $\rightarrow$ Hiển thị: 2 Hộp, 5 Vỉ).
* **UC14: Xem Báo cáo Doanh thu & Lợi nhuận**
  * *Mô tả:* Thống kê doanh số, chi phí nhập hàng, lợi nhuận gộp theo ca, theo ngày/tháng.
* **UC15: Quản lý Tài khoản & Phân quyền**
  * *Mô tả:* Quản lý nhân viên, cấp quyền truy cập các chức năng bán hàng, nhập kho hoặc xem báo cáo.

---

## 5. ĐẶC TẢ CHI TIẾT USE CASE TRỌNG TÂM (DETAILED USE CASE SPECIFICATION)

### 5.1. UC09: Tự động Gợi ý & Trừ Kho theo FEFO

* **Primary Actor:** Dược sĩ / Nhân viên Bán hàng
* **Secondary Actor:** Hệ thống Tự động
* **Pre-conditions (Điều kiện tiên quyết):**
  * Dược sĩ đã đăng nhập vào màn hình POS.
  * Sản phẩm còn tồn kho ($Tồn_{tổng} > 0$).
* **Post-conditions (Điều kiện sau khi thực hiện):**
  * Hóa đơn được tạo thành công.
  * Tồn kho của từng Lô bị trừ chính xác theo ĐVT gốc.
  * Nhật ký giao dịch Lô (Batch Transaction Log) được ghi nhận.

#### Luồng sự kiện chính (Main Flow):
1. Dược sĩ gõ tên hoặc quét mã vạch tìm kiếm Thuốc X.
2. Dược sĩ chọn ĐVT bán (Ví dụ: "Vỉ") và nhập Số lượng bán (Ví dụ: $2$).
3. Hệ thống tra cứu Bảng quy đổi ĐVT: $1 \text{ Vỉ} = 10 \text{ Viên}$. Tính tổng số lượng gốc cần trừ: $2 \times 10 = 20 \text{ Viên}$.
4. Hệ thống quét danh sách các Lô còn tồn kho của Thuốc X, lọc `SoLuongTonGoc > 0`, sắp xếp `HanSuDung ASC`.
5. Hệ thống thực hiện thuật toán phân bổ FEFO:
   * *Lô A (HSD: 10/2026, Tồn: 12 viên)* $\rightarrow$ Lấy hết 12 viên. Số lượng còn thiếu: $20 - 12 = 8 \text{ viên}$.
   * *Lô B (HSD: 12/2026, Tồn: 50 viên)* $\rightarrow$ Lấy tiếp 8 viên. Số lượng còn thiếu: $0$.
6. Hệ thống hiển thị trên màn hình POS thông tin phân bổ Lô chi tiết để Dược sĩ thực hiện lấy thuốc thực tế trên kệ:
   * 12 viên Lô A
   * 8 viên Lô B
7. Dược sĩ xác nhận và thực hiện thanh toán hóa đơn.
8. Hệ thống trừ $12$ viên ở Lô A và $8$ viên ở Lô B trong CSDL.

#### Luồng ngoại lệ (Alternative / Exception Flows):
* **A1: Không đủ tồn kho tổng:**
  * Tại bước 3, nếu $Tồn_{tổng} < S_{gốc}$ cần bán, hệ thống hiển thị thông báo "Không đủ hàng tồn kho. Tồn hiện tại: Y viên, Cần bán: X viên". Dược sĩ có thể điều chỉnh lại số lượng bán.
* **A2: Dược sĩ muốn thay đổi Lô (Override):**
  * Tại bước 6, Dược sĩ nhấn "Chọn lại Lô" $\rightarrow$ Kích hoạt **UC10 (Chọn Lô Thủ công)**. Dược sĩ tự tích chọn Lô mong muốn và nhập lý do điều chỉnh.

---

### 5.2. UC02: Thiết lập ĐVT & Tỷ lệ Quy đổi

* **Primary Actor:** Chủ nhà thuốc / Quản lý
* **Pre-conditions:** Quản lý đã đăng nhập và mở màn hình Danh mục Thuốc.

#### Luồng sự kiện chính (Main Flow):
1. Quản lý chọn một mặt hàng Thuốc (Ví dụ: *Paracetamol 500mg*).
2. Quản lý xác định **ĐVT Gốc** (Đơn vị nhỏ nhất không thể chia nhỏ hơn nữa trong kinh doanh, ví dụ: `Viên`).
3. Quản lý bấm "Thêm ĐVT Quy đổi".
4. Quản lý nhập thông tin ĐVT mới:
   * Tên ĐVT: `Vỉ`
   * Tỷ lệ quy đổi so với ĐVT gốc: `10` (Hiểu là: 1 Vỉ = 10 Viên)
5. Quản lý bấm "Thêm ĐVT Quy đổi" tiếp theo:
   * Tên ĐVT: `Hộp`
   * Tỷ lệ quy đổi so với ĐVT gốc: `100` (Hiểu là: 1 Hộp = 100 Viên)
6. Hệ thống kiểm tra tính hợp lệ của tỷ lệ quy đổi ($Tỷ\ lệ > 1$, các ĐVT không trùng tên).
7. Quản lý nhấn "Lưu". Hệ thống lưu cấu hình quy đổi vào CSDL.