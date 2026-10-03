# USER STORY: US13 - BÁO CÁO TỒN KHO CHI TIẾT THEO LÔ & ĐA QUY ĐỔI (MULTI-UNIT & BATCH INVENTORY REPORT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US13** |
| **Tên User Story** | Xem Báo cáo Tồn kho chi tiết theo Lô & Đơn vị tính Đa quy đổi |
| **Phân hệ (Module)** | Phân hệ 4: Cảnh báo, Báo cáo & Quản trị (Alerts, Reports & Admin) |
| **Use Case liên quan** | **UC13: Xem Báo cáo Tồn kho chi tiết theo Lô & ĐVT** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-REP-02] Báo cáo Tồn kho Đa quy đổi** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Hỗ trợ ra quyết định & Kiểm soát kho)** |
| **Tác nhân chính (Primary Actor)** | Chủ nhà thuốc / Quản lý (*Pharmacy Owner/Manager*) |
| **Tác nhân phụ (Secondary Actor)** | Thủ kho (*Warehouse Specialist*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Quản lý nhà thuốc và Thủ kho,
* **Tôi muốn (I want to):** Xem báo cáo tồn kho chi tiết phân rã theo từng Lô, từng Hạn sử dụng, và được hệ thống tự động tính toán diễn giải số lượng tồn gốc thành chuỗi Đơn vị tính Quy đổi trực quan (ví dụ: *2 Hộp - 5 Vỉ - 5 Viên*),
* **Để (So that):** Tôi có thể dễ dàng kiểm đếm số lượng hàng hóa vật lý trên quầy kệ theo thói quen đóng gói thực tế mà không cần phải mang máy tính cầm tay đi chia nhẩm từ số viên lẻ ra số hộp và số vỉ, đồng thời kiểm soát chính xác tuổi thọ của từng lô hàng.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Nếu báo cáo tồn kho chỉ hiển thị: *"Paracetamol: Tồn 255 Viên"*, khi Thủ kho đi kiểm đếm thực tế trên kệ sẽ rất bối rối vì trên kệ thuốc được xếp theo từng Hộp và từng Vỉ. Thủ kho sẽ phải tự tính nhẩm:
* $255 \text{ viên} \div 100 = 2 \text{ Hộp}$ (còn dư 55 viên).
* $55 \text{ viên} \div 10 = 5 \text{ Vỉ}$ (còn dư 5 viên).
* $5 \text{ viên lẻ}$.
Do đó, báo cáo của hệ thống cần cung cấp tính năng **"Đa Quy Đổi Thông Minh" (Smart Multi-Unit Decomposition)**: Tự động phân rã số tồn nguyên gốc thành chuỗi ĐVT đóng gói lớn dần theo đúng cấu hình phân cấp của thuốc, giúp báo cáo trực quan, thân thiện và chính xác tuyệt đối.

---

## 4. QUY TẮC NGHIỆP VỤ & THUẬT TOÁN (BUSINESS RULES & ALGORITHM)

### 4.1. Thuật toán Phân rã Đa quy đổi (Multi-Unit Decomposition Algorithm)
Giả sử một mặt hàng thuốc có ĐVT Gốc $U_0$ và danh sách các ĐVT Quy đổi $\{ U_1, U_2, \dots, U_m \}$ với tỷ lệ quy đổi tương ứng $\{ k_1, k_2, \dots, k_m \}$ đã được sắp xếp giảm dần theo tỷ lệ quy đổi ($k_m > k_{m-1} > \dots > k_1 > 1$).
Với số lượng tồn gốc hiện tại là $Q$:
1. Khởi tạo số dư còn lại: $R = Q$.
2. Lặp từ đơn vị lớn nhất $U_m$ xuống đơn vị nhỏ nhất $U_1$:
   * Số lượng của đơn vị $U_i$: $q_i = \lfloor R / k_i \rfloor$.
   * Số dư cập nhật: $R = R \pmod{k_i}$.
3. Số lượng của ĐVT Gốc $U_0$: $q_0 = R$.
4. Chuỗi hiển thị kết quả: Ghép các $q_i > 0$ thành chuỗi định dạng (ví dụ: `$q_2$ Hộp - $q_1$ Vỉ - $q_0$ Viên`).

### 4.2. Quy tắc nghiệp vụ bổ sung
* **[BR-REP-02.1] Tính hai chiều:** Báo cáo luôn cung cấp 2 cột số lượng song song:
  * Cột 1: `Tồn kho nguyên gốc` (Ví dụ: `255 Viên` - chuẩn số liệu kế toán).
  * Cột 2: `Tồn quy đổi trực quan` (Ví dụ: `2 Hộp, 5 Vỉ, 5 Viên` - phục vụ kiểm đếm).
* **[BR-REP-02.2] Phân nhóm đa chiều:** Cho phép người dùng chuyển đổi qua lại giữa:
  * Chế độ gom nhóm theo Mặt hàng thuốc (tổng tồn của tất cả các Lô).
  * Chế độ chi tiết theo từng Lô (Mỗi dòng là 1 Lô kèm Số Lô NSX, HSD, Ngày nhập, Nhà cung cấp).

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Phân rã tồn kho đa cấp chính xác (Happy Path - Khớp SRS FR-REP-02)
* **Given:** Thuốc `Paracetamol 500mg` có:
  * ĐVT Gốc: `Viên` ($k=1$)
  * ĐVT 1: `Vỉ` ($k=10$)
  * ĐVT 2: `Hộp` ($k=100$)
  * Số lượng tồn kho thực tế trong `TON_KHO_LO` là `255 Viên`.
* **When:** Quản lý mở Báo cáo Tồn kho.
* **Then:** Tại dòng thuốc `Paracetamol 500mg`, cột "Tồn Quy Đổi" hiển thị chính xác chuỗi: **"2 Hộp, 5 Vỉ, 5 Viên"**, cột "Tồn Gốc" hiển thị **"255 Viên"**.

### Kịch bản 2: Hiển thị thuốc chỉ có ĐVT Gốc hoặc chỉ có 1 cấp quy đổi
* **Given:** Thuốc `Siro Ho Prospan` chỉ có ĐVT Gốc là `Chai` (không có quy đổi). Tồn kho là `18 Chai`.
* **When:** Xem báo cáo tồn kho.
* **Then:** Cột "Tồn Quy Đổi" hiển thị gọn gàng là **"18 Chai"**, không phát sinh các tiền tố rỗng.

### Kịch bản 3: Lọc báo cáo theo tình trạng Hạn sử dụng của Lô
* **Given:** Trong kho có 100 Lô thuốc với nhiều HSD khác nhau.
* **When:** Thủ kho chọn bộ lọc "Tình trạng HSD": chọn "Cận hạn <= 30 ngày (Mức Đỏ)".
* **Then:** Báo cáo chỉ hiển thị danh sách các Lô có $HSD - NgàyHiệnTại \le 30 \text{ ngày}$, có bôi đỏ cột HSD và hiển thị số ngày còn lại đến hạn.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Hàm xử lý logic phân rã chuỗi (C# Service)
```csharp
public string FormatMultiUnitDisplay(int soLuongTonGoc, string maThuoc)
{
    if (soLuongTonGoc <= 0) return "0";

    var thuoc = _dbContext.Thuoc.Find(maThuoc);
    var quyDoiList = _dbContext.QuyDoiDVT
        .Where(q => q.MaThuoc == maThuoc)
        .OrderByDescending(q => q.TyLeQuyDoi)
        .ToList();

    List<string> parts = new List<string>();
    int remainder = soLuongTonGoc;

    foreach (var qd in quyDoiList)
    {
        int count = remainder / qd.TyLeQuyDoi;
        if (count > 0)
        {
            parts.Add($"{count} {qd.MaDVT}");
            remainder %= qd.TyLeQuyDoi;
        }
    }

    if (remainder > 0 || !parts.Any())
    {
        parts.Add($"{remainder} {thuoc.MaDVTGoc}");
    }

    return string.Join(" - ", parts);
}
```

### 6.2. Xuất dữ liệu
* Hỗ trợ Export file định dạng Excel (.xlsx) với các sheet phân loại riêng biệt và PDF để in trực tiếp cho thủ kho đi kiểm kê.

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US01, US02:** Có danh mục thuốc và các cấp tỷ lệ quy đổi ĐVT.
  * **US04, US05:** Có dữ liệu tồn kho theo Lô `TON_KHO_LO`.
* **Hỗ trợ cho:**
  * **US06:** Cung cấp biểu mẫu chuẩn để Thủ kho in ra đi đối soát kiểm kê thực tế.
