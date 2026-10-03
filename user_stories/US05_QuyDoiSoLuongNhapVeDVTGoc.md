# USER STORY: US05 - QUY ĐỔI SỐ LƯỢNG NHẬP VỀ ĐƠN VỊ TÍNH GỐC (INVENTORY CONVERSION ON RECEIPT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US05** |
| **Tên User Story** | Tự động Quy đổi Số lượng Nhập về ĐVT Gốc |
| **Phân hệ (Module)** | Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management) |
| **Use Case liên quan** | **UC05: Quy đổi Số lượng Nhập về ĐVT Gốc** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-INV-02] Tự động Quy đổi Tồn kho khi Nhập** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Cốt lõi tính toán tồn kho)** |
| **Tác nhân chính (Primary Actor)** | Hệ thống Tự động (*System Worker / Business Logic Engine*) |
| **Tác nhân phụ (Secondary Actor)** | Thủ kho (*Warehouse Specialist*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Thủ kho và Hệ thống Quản lý Kho,
* **Tôi muốn (I want to):** Hệ thống tự động chuyển đổi số lượng hàng nhập từ các Đơn vị tính Quy đổi (như Hộp, Thùng, Vỉ) về đúng Đơn vị tính Gốc (Base Unit) trước khi cộng dồn vào bảng tồn kho của từng Lô,
* **Để (So that):** Cơ sở dữ liệu duy trì một chuẩn lưu trữ tồn kho duy nhất, đồng nhất bằng số nguyên dương, loại bỏ hoàn toàn các sai số do tính toán số thập phân và đảm bảo trừ kho chính xác đến từng viên lẻ khi bán tại quầy POS.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Nếu hệ thống lưu trữ tồn kho theo đơn vị nhập (ví dụ: kho lưu "5 Hộp"), khi bán lẻ 3 vỉ hoặc 5 viên thuốc, hệ thống sẽ phải lưu số thập phân (ví dụ: $5 - \frac{3}{10} - \frac{5}{100} = 4.65 \text{ Hộp}$). Việc tính toán số thực trong máy tính gây ra sai số dấu phẩy động (floating-point precision error), dẫn đến hiện tượng lệch kho, thất thoát hoặc không thể đối soát.
Vì vậy, nguyên tắc kiến trúc bất biến của hệ thống là:
1. **Lớp giao diện (Presentation):** Cho phép Thủ kho nhập theo ĐVT thuận tiện nhất (Hộp, Thùng, Vỉ).
2. **Lớp nghiệp vụ (Business Logic):** Tự động tra cứu `TyLeQuyDoi` của ĐVT đó so với ĐVT Gốc để tính $SoLuongGocTang$.
3. **Lớp lưu trữ (Persistence):** Chỉ ghi nhận và cộng dồn số lượng nguyên vào trường `SoLuongTonGoc` trong bảng `TON_KHO_LO`.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-INV-02.1] Công thức quy đổi nhập kho:**
  $$SoLuongGocTang = SoLuongNhap \times TyLeQuyDoi(MaThuoc, MaDVTNhap)$$
  * Nếu ĐVT Nhập chính là ĐVT Gốc: $\text{TyLeQuyDoi} = 1 \rightarrow SoLuongGocTang = SoLuongNhap$.
  * Nếu ĐVT Nhập là ĐVT Quy đổi: $\text{TyLeQuyDoi} > 1$.
* **[BR-INV-02.2] Ràng buộc số nguyên dương ($\mathbb{Z}^+$):** Cả $SoLuongNhap$, $TyLeQuyDoi$ và $SoLuongGocTang$ bắt buộc là số nguyên dương $> 0$.
* **[BR-INV-02.3] Tính giá trị vốn theo ĐVT Gốc:**
  $$DonGiaGoc = \frac{DonGiaNhap}{TyLeQuyDoi}$$
  Hệ thống ghi nhận đơn giá vốn theo ĐVT Gốc để phục vụ tính giá vốn hàng bán chính xác khi xuất bán lẻ theo viên hoặc vỉ.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Tự động quy đổi từ ĐVT Hộp về ĐVT Viên (Happy Path)
* **Given:** Thuốc `Paracetamol 500mg` có ĐVT Gốc là `Viên`, ĐVT Quy đổi `Hộp` với tỷ lệ quy đổi là `100`. Lô `LOT-001` đang có tồn kho là `200 Viên`.
* **When:** Thủ kho nhập thêm vào phiếu nhập: Thuốc `Paracetamol 500mg`, Lô `LOT-001`, Số lượng nhập: `5`, ĐVT Nhập: `Hộp`.
* **Then:**
  * Trên giao diện form nhập hiển thị ngay dòng thông tin gợi ý: "Tương đương: 500 Viên".
  * Khi bấm "Lưu", hệ thống tính: $5 \times 100 = 500 \text{ Viên}$.
  * Cập nhật `TON_KHO_LO.SoLuongTonGoc` của Lô `LOT-001` thành: $200 + 500 = 700 \text{ Viên}$.

### Kịch bản 2: Nhập trực tiếp theo ĐVT Gốc
* **Given:** Thuốc `Vitamin C 500mg` có ĐVT Gốc là `Viên`.
* **When:** Thủ kho nhập số lượng `150`, ĐVT Nhập là `Viên`.
* **Then:** Hệ thống nhân tỷ lệ quy đổi là `1`, số lượng gốc tăng thêm đúng `150 Viên`.

### Kịch bản 3: Phiếu nhập có nhiều mặt hàng với các ĐVT khác nhau
* **Given:** Một phiếu nhập kho gồm:
  * Dòng 1: Thuốc A, nhập 2 `Thùng` (1 Thùng = 500 Viên) $\rightarrow$ quy đổi 1.000 Viên.
  * Dòng 2: Thuốc B, nhập 10 `Hộp` (1 Hộp = 50 Gói) $\rightarrow$ quy đổi 500 Gói.
  * Dòng 3: Thuốc C, nhập 30 `Chai` (ĐVT Gốc là Chai) $\rightarrow$ quy đổi 30 Chai.
* **When:** Thủ kho xác nhận hoàn tất phiếu nhập.
* **Then:** Hệ thống quy đổi chính xác từng dòng độc lập và cập nhật tồn kho từng Lô tương ứng đúng số lượng gốc trong 1 Transaction.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `TON_KHO_LO`:
  * `MaLo` (VARCHAR(50), Primary Key, Foreign Key $\rightarrow$ `LO_HANG.MaLo`)
  * `SoLuongTonGoc` (INT, Not Null, Check Constraint: `SoLuongTonGoc >= 0`)
  * `NgayCapNhat` (DATETIME, Default GETDATE())
* Bảng `LICH_SU_KHO` (Stock Ledger):
  * `ID` (BIGINT, Primary Key, Identity)
  * `MaLo` (VARCHAR(50))
  * `LoaiGiaoDich` (VARCHAR(20): `NHAP_KHO`, `XUAT_BAN`, `KIEM_KE`, `XUAT_HUY`)
  * `MaChungTu` (VARCHAR(50))
  * `SoLuongThayDoiGoc` (INT, Mang dấu `+` khi nhập, `-` khi xuất)
  * `SoLuongTonSauGD` (INT)
  * `ThoiGian` (DATETIME)

### 6.2. Đoạn mã mẫu Logic xử lý quy đổi
```csharp
public int CalculateBaseQuantity(string maThuoc, string maDVTNhap, int soLuongNhap)
{
    var thuoc = _dbContext.Thuoc.Find(maThuoc);
    if (thuoc.MaDVTGoc == maDVTNhap)
    {
        return soLuongNhap;
    }

    var quyDoi = _dbContext.QuyDoiDVT
        .FirstOrDefault(q => q.MaThuoc == maThuoc && q.MaDVT == maDVTNhap);

    if (quyDoi == null)
    {
        throw new BusinessException($"Chưa cấu hình tỷ lệ quy đổi cho ĐVT '{maDVTNhap}' của thuốc '{maThuoc}'.");
    }

    return soLuongNhap * quyDoi.TyLeQuyDoi;
}
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US02:** Đã thiết lập tỷ lệ quy đổi `TyLeQuyDoi` cho các ĐVT.
  * **US04:** Phiếu nhập kho được kích hoạt lưu.
* **Là tiền đề cho các User Story:**
  * **US06:** Kiểm kê Kho & Điều chỉnh Tồn Lô.
  * **US09:** Tự động Gợi ý & Trừ Kho theo FEFO (dựa trên `SoLuongTonGoc`).
  * **US13:** Báo cáo Tồn kho chi tiết theo Lô & ĐVT.
