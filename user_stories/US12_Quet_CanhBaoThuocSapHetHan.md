# USER STORY: US12 - QUÉT & CẢNH BÁO THUỐC CẬN HẠN VÀ HẾT HẠN (EXPIRY MONITORING & ALERT)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US12** |
| **Tên User Story** | Quét & Cảnh báo Thuốc Cận hạn / Hết hạn |
| **Phân hệ (Module)** | Phân hệ 4: Cảnh báo, Báo cáo & Quản trị (Alerts, Reports & Admin) |
| **Use Case liên quan** | **UC12: Quét & Cảnh báo Thuốc Sắp Hết hạn** (DacTa.md) |
| **Yêu cầu SRS** | **[FR-REP-01] Quét & Cảnh báo Thuốc Cận hạn / Hết hạn** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Quản trị rủi ro & chất lượng dược phẩm)** |
| **Tác nhân chính (Primary Actor)** | Quản lý / Chủ nhà thuốc (*Pharmacy Owner/Manager*), Thủ kho |
| **Tác nhân phụ (Secondary Actor)** | Hệ thống Tự động (*Background Worker / Cron Job*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Quản lý nhà thuốc và Thủ kho,
* **Tôi muốn (I want to):** Hệ thống có tiến trình tự động quét định kỳ kho hàng và phân loại cảnh báo các Lô thuốc sắp hết hạn theo các cấp độ màu sắc trực quan (Đỏ, Vàng, Đen) trên Dashboard và danh sách cảnh báo,
* **Để (So that):** Kịp thời chủ động lên kế hoạch xử lý như: đàm phán đổi trả lại cho Nhà cung cấp trước thời hạn cam kết, thiết lập chương trình khuyến mãi/giảm giá để kích cầu đẩy bán, hoặc chủ động cách ly tiêu hủy thuốc đã hết hạn nhằm bảo vệ sức khỏe người dùng và tối thiểu hóa thiệt hại tài chính.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Hàng tồn kho hết hạn là một trong những nguyên nhân gây thâm hụt lợi nhuận lớn nhất của các nhà thuốc bán lẻ. Hầu hết các nhà phân phối dược phẩm cho phép nhà thuốc đổi trả hàng cận hạn nếu thông báo trước từ 3 đến 6 tháng.
Nếu nhân viên phải đi lật từng hộp thuốc trên kệ để ghi chép ngày hết hạn vào sổ tay thì vừa tốn công, vừa dễ bỏ sót.
Hệ thống giải quyết triệt để vấn đề này bằng một tiến trình tự động quét ngầm (Cron Job) chạy mỗi ngày lúc 00:00:
* Tự động tính toán số ngày còn lại đến hạn: $\text{SoNgayConLai} = HSD - NgayHienTai$.
* Phân loại Lô vào các nhóm cảnh báo chuẩn màu sắc.
* Hiển thị thông báo nổi bật ngay khi Quản lý hoặc Thủ kho đăng nhập vào hệ thống.

---

## 4. QUY TẮC NGHIỆP VỤ (BUSINESS RULES)

* **[BR-REP-01.1] Phân loại ngưỡng cảnh báo:**
  * **Mức Đen ($HSD \le NgayHienTai$):** Thuốc đã hết hạn sử dụng. Hệ thống tự động khóa bán trên POS và chuyển trạng thái Lô thành "Chờ hủy".
  * **Mức Đỏ ($0 < HSD - NgayHienTai \le 30 \text{ ngày}$):** Cảnh báo nguy cấp. Cần đàm phán trả Nhà cung cấp ngay lập tức hoặc thanh lý gấp.
  * **Mức Vàng ($30 < HSD - NgayHienTai \le 90 \text{ ngày}$):** Cảnh báo cận hạn. Cần ưu tiên FEFO đẩy bán, tư vấn thay thế hoặc giảm giá.
  * **Mức Xanh ($HSD - NgayHienTai > 90 \text{ ngày}$):** Trạng thái an toàn bình thường.
* **[BR-REP-01.2] Khả năng tùy biến cấu hình:** Cho phép Quản lý tùy chỉnh số ngày của ngưỡng Vàng (ví dụ: điều chỉnh thành 60 ngày hoặc 120 ngày tùy theo quy mô nhà thuốc).
* **[BR-REP-01.3] Chu kỳ quét:** Tiến trình ngầm tự động kích hoạt hàng ngày vào lúc `00:00:00` và có thể bấm nút "Quét lại ngay" thủ công bất cứ lúc nào trên giao diện.

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Cảnh báo tự động hiển thị trên Dashboard (Happy Path)
* **Given:** Trong kho đang có:
  * 2 Lô có HSD còn 15 ngày (Mức Đỏ).
  * 5 Lô có HSD còn 60 ngày (Mức Vàng).
  * 1 Lô có HSD đã qua 2 ngày (Mức Đen).
* **When:** Quản lý hoặc Thủ kho đăng nhập vào hệ thống vào buổi sáng.
* **Then:**
  * Widget trên Dashboard hiển thị thẻ tóm tắt:
    * `1 Lô Hết hạn` (Màu đen/đỏ thẫm, nhấp nháy cảnh báo).
    * `2 Lô Cận hạn khẩn cấp <= 30 ngày` (Màu đỏ).
    * `5 Lô Sắp cận hạn <= 90 ngày` (Màu vàng).
  * Nhấp vào từng thẻ sẽ chuyển sang màn hình danh sách chi tiết các Lô đó.

### Kịch bản 2: Tự động khóa bán đối với Lô Mức Đen
* **Given:** Lô `LOT-BLACK-99` có HSD là ngày hôm qua (`02/10/2026`).
* **When:** Tiến trình Cron job 00:00 chạy qua ngày mới (`03/10/2026`).
* **Then:** Trạng thái của Lô `LOT-BLACK-99` tự động chuyển sang `3 - Hết hạn/Khóa bán`. Khi Dược sĩ bán thuốc đó ở quầy POS, hệ thống không bao giờ phân bổ Lô này.

### Kịch bản 3: Xuất danh sách Lô cận hạn ra file Excel
* **Given:** Quản lý đang ở màn hình "Báo cáo Cảnh báo Cận hạn".
* **When:** Quản lý lọc danh sách Mức Đỏ ($\le 30$ ngày) và bấm nút "Xuất file Excel".
* **Then:** Hệ thống tải về file Excel được định dạng chuẩn, chứa đầy đủ các cột: Tên thuốc, Tên hoạt chất, Số Lô NSX, Hạn sử dụng, Số lượng tồn gốc, Tồn quy đổi (Hộp/Vỉ/Viên), Tên Nhà cung cấp và Số điện thoại liên hệ để gửi khiếu nại trả hàng.

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thiết kế Worker / Background Job
* Sử dụng **Quartz.NET**, **Hangfire** hoặc **Cron Job** chạy định kỳ mỗi 24 giờ.
```csharp
public async Task ScanExpiryBatchesJob()
{
    var today = DateTime.Today;
    
    // 1. Tự động khóa các lô đã hết hạn hôm nay
    await _dbContext.LoHang
        .Where(l => l.HanSuDung <= today && l.TrangThai == 1)
        .ExecuteUpdateAsync(s => s.SetProperty(b => b.TrangThai, 3)); // 3: Khóa bán

    // 2. Gửi thông báo Push Notification / Email cảnh báo nếu có lô Mức Đỏ
    var redBatches = await _dbContext.LoHang
        .Include(l => l.Thuoc)
        .Where(l => l.HanSuDung > today && l.HanSuDung <= today.AddDays(30) && l.TrangThai == 1)
        .ToListAsync();

    if (redBatches.Any())
    {
        await _notificationService.SendManagerAlertAsync("CẢNH BÁO THUỐC CẬN HẠN", 
            $"Hiện có {redBatches.Count} lô thuốc sắp hết hạn trong 30 ngày tới.");
    }
}
```

### 6.2. Hiển thị UI
* Sử dụng bảng màu chuẩn:
  * Mức Đen: Badge `#212529` Text `#FFFFFF`
  * Mức Đỏ: Badge `#DC3545` Text `#FFFFFF`
  * Mức Vàng: Badge `#FFC107` Text `#000000`
  * Mức Xanh: Badge `#28A745` Text `#FFFFFF`

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US04:** Đã có thông tin Lô và HSD khi nhập kho.
* **Là tiền đề cho:**
  * **US07:** Cung cấp danh sách Lô Mức Đen để lập phiếu xuất hủy.
  * **US09:** Bảo đảm các Lô Mức Đen bị loại khỏi thuật toán FEFO.
