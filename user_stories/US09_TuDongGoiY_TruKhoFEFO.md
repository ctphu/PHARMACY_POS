# USER STORY: US09 - TỰ ĐỘNG GỢI Ý & TRỪ KHO THEO NGUYÊN TẮC FEFO (CORE FEFO AUTO ALLOCATION)

---

## 1. THÔNG TIN CHUNG (METADATA)

| Thuộc tính | Chi tiết |
| :--- | :--- |
| **Mã User Story** | **US09** |
| **Tên User Story** | Tự động Gợi ý & Trừ Kho theo Nguyên tắc FEFO |
| **Phân hệ (Module)** | Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO |
| **Use Case liên quan** | **UC09: Tự động Gợi ý & Trừ Kho theo FEFO** (Đặc tả chi tiết Mục 5.1 trong DacTa.md) |
| **Yêu cầu SRS** | **[FR-POS-02] Tự động Trừ kho theo Thuật toán FEFO (Core)** (SRS.md) |
| **Độ ưu tiên (Priority)** | **Must-Have (Tính năng Trọng tâm Cốt lõi của Dự án)** |
| **Tác nhân chính (Primary Actor)** | Dược sĩ / Nhân viên bán hàng (*Pharmacist/Sales Clerk*) |
| **Tác nhân phụ (Secondary Actor)** | Hệ thống Tự động (*FEFO Allocation Engine / Worker*) |
| **Trạng thái (Status)** | Ready for Development |

---

## 2. NỘI DUNG USER STORY (STORY STATEMENT)

* **Là một (As a):** Dược sĩ bán hàng tại quầy POS,
* **Tôi muốn (I want to):** Hệ thống tự động phân tích các Lô thuốc còn hạn sử dụng trong kho, ưu tiên gợi ý và trừ kho tự động từ các Lô có Hạn sử dụng gần nhất trước (First Expired, First Out) khi tôi bán thuốc theo bất kỳ Đơn vị tính nào,
* **Để (So that):** Nhà thuốc giải phóng tối đa lượng thuốc cận hạn, ngăn chặn rủi ro thuốc hết hạn tồn ứ trong kho gây tổn thất tài chính, đảm bảo tuân thủ nghiêm ngặt chuẩn Thực hành tốt nhà thuốc (GPP) mà không đòi hỏi nhân viên phải nhớ hoặc kiểm tra thủ công từng hộp thuốc trên kệ.

---

## 3. BỐI CẢNH & MÔ TẢ NGHIỆP VỤ (BUSINESS CONTEXT)

Trong bán lẻ dược phẩm, nguyên tắc quản lý kho tối quan trọng không phải là FIFO (Nhập trước Xuất trước) mà là **FEFO (First Expired, First Out - Hết hạn trước Xuất trước)**:
* Một loại thuốc có thể có nhiều lô sản xuất khác nhau đang nằm trên kệ.
* Lô có hạn dùng gần hơn bắt buộc phải được ưu tiên bán trước, kể cả khi lô đó nhập về kho sau các lô khác.
* Khi khách hàng mua một số lượng lớn (ví dụ: mua 20 viên), nếu Lô có hạn gần nhất chỉ còn 12 viên, hệ thống phải tự động thông minh "tách dòng phân bổ": lấy hết 12 viên của Lô cận hạn, và lấy thêm 8 viên từ Lô tiếp theo.
* Màn hình POS phải chỉ dẫn rõ ràng cho Dược sĩ biết cần ra kệ nhặt thuốc từ Lô nào, số lượng bao nhiêu.

---

## 4. QUY TẮC NGHIỆP VỤ & THUẬT TOÁN (BUSINESS RULES & ALGORITHM)

### 4.1. Quy tắc nghiệp vụ (Business Rules)
* **[BR-POS-02.1] Điều kiện Lô khả dụng xuất bán:**
  $$\text{Danh sách Lô khả dụng} = \{ L_i \mid L_i.MaThuoc = X \land L_i.SoLuongTonGoc > 0 \land L_i.HanSuDung > NgayHienTai \land L_i.TrangThai = \text{Khả dụng} \}$$
  Bất kỳ Lô nào có $HSD \le NgayHienTai$ hoặc trạng thái bị khóa/chờ hủy đều bị loại trừ khỏi thuật toán.
* **[BR-POS-02.2] Quy đổi số lượng cần xuất về ĐVT Gốc:**
  $$S_{Cần\_Xuất} = SoLuongBan \times TyLeQuyDoi(MaThuoc, MaDVT)$$
* **[BR-POS-02.3] Bảo toàn giao dịch ACID (NFR-REL-01):** Thao tác [Ghi nhận Hóa đơn + Trừ tồn kho chi tiết từng Lô trong `TON_KHO_LO` + Ghi nhật ký kho `LICH_SU_KHO`] phải nằm trọn vẹn trong một Database Transaction. Nếu có bất kỳ lỗi nào xảy ra, toàn bộ giao dịch phải Rollback lập tức.
* **[BR-POS-02.4] Kiểm soát tương tranh (Concurrency Control - NFR-PER-02):** Khi nhiều quầy POS cùng bán một mặt hàng cùng lúc, hệ thống sử dụng cơ chế Khóa bi quan có thời hạn (`SELECT ... WITH (UPDLOCK, ROWLOCK)`) hoặc Khóa lạc quan (Optimistic Concurrency với `RowVersion`) để đảm bảo không bị bán âm (Oversell) cùng một Lô.

### 4.2. Mã giả Thuật toán Phân bổ FEFO (FEFO Pseudocode)
```text
FUNCTION AllocateFEFO(MaThuoc, SoLuongBan, MaDVT):
    SCanXuat = SoLuongBan * GetTyLeQuyDoi(MaThuoc, MaDVT)
    
    // Lấy danh sách lô khả dụng, ưu tiên hạn gần nhất
    DanhSachLo = SELECT * FROM TON_KHO_LO 
                 JOIN LO_HANG ON TON_KHO_LO.MaLo = LO_HANG.MaLo
                 WHERE LO_HANG.MaThuoc = MaThuoc 
                   AND TON_KHO_LO.SoLuongTonGoc > 0 
                   AND LO_HANG.HanSuDung > CURRENT_DATE()
                   AND LO_HANG.TrangThai = 1
                 ORDER BY LO_HANG.HanSuDung ASC

    TongTonKhaDung = SUM(DanhSachLo.SoLuongTonGoc)
    IF TongTonKhaDung < SCanXuat THEN:
        THROW BusinessException("Không đủ tồn kho khả dụng theo Lô. Tồn hiện tại: " + TongTonKhaDung + ", Cần bán: " + SCanXuat)
    
    KetQuaPhanBo = LIST()
    
    FOR EACH Lo IN DanhSachLo:
        IF SCanXuat <= 0 THEN BREAK
        
        IF Lo.SoLuongTonGoc >= SCanXuat THEN:
            KetQuaPhanBo.ADD({ MaLo: Lo.MaLo, SoLuongXuat: SCanXuat, HSD: Lo.HanSuDung })
            Lo.SoLuongTonGoc = Lo.SoLuongTonGoc - SCanXuat
            SCanXuat = 0
        ELSE:
            KetQuaPhanBo.ADD({ MaLo: Lo.MaLo, SoLuongXuat: Lo.SoLuongTonGoc, HSD: Lo.HanSuDung })
            SCanXuat = SCanXuat - Lo.SoLuongTonGoc
            Lo.SoLuongTonGoc = 0
            
    RETURN KetQuaPhanBo
END FUNCTION
```

---

## 5. TIÊU CHÍ CHẤP NHẬN (ACCEPTANCE CRITERIA - AC)

### Kịch bản 1: Phân bổ FEFO trừ qua nhiều Lô (TC01 trong SRS / DacTa 5.1)
* **Given:** Khách hàng mua `2 Vỉ` thuốc `Paracetamol 500mg` ($1 \text{ Vỉ} = 10 \text{ Viên} \rightarrow S_{Cần\_Xuất} = 20 \text{ Viên}$). Kho hiện có:
  * Lô A (`LOT-A`): HSD `10/2026`, tồn kho `12 Viên`.
  * Lô B (`LOT-B`): HSD `12/2026`, tồn kho `50 Viên`.
* **When:** Dược sĩ nhập bán 2 Vỉ và bấm thanh toán.
* **Then:**
  * Hệ thống tự động phân bổ:
    * Lấy `12 Viên` từ Lô A (Tồn Lô A giảm về `0 Viên`).
    * Lấy tiếp `8 Viên` từ Lô B (Tồn Lô B giảm từ `50` xuống `42 Viên`).
  * Màn hình POS hiển thị chỉ dẫn chi tiết: `[Lô A: 12 viên] + [Lô B: 8 viên]`.
  * Hóa đơn chi tiết `CHI_TIET_HOA_DON` lưu tách thành 2 dòng liên kết đúng `MaLo`.

### Kịch bản 2: Báo lỗi khi tổng tồn kho khả dụng không đủ (Exception Flow A1)
* **Given:** Thuốc `Panadol Cảm Cúm` tổng tồn kho khả dụng của tất cả các Lô còn hạn là `15 Viên`.
* **When:** Dược sĩ nhập bán `2 Vỉ` ($20 \text{ Viên}$).
* **Then:** Hệ thống chặn giao dịch, không trừ kho và hiển thị thông báo lỗi rõ ràng: "Không đủ hàng tồn kho khả dụng. Tồn hiện tại: 15 Viên, Cần bán: 20 Viên". Gợi ý Dược sĩ điều chỉnh số lượng bán hoặc kiểm tra kho.

### Kịch bản 3: Tự động loại trừ Lô đã hết hạn khỏi phân bổ (TC03 trong SRS)
* **Given:** Thuốc X có:
  * Lô 1: HSD `01/01/2026` (Đã hết hạn), tồn kho `100 Viên`.
  * Lô 2: HSD `15/12/2027` (Còn hạn), tồn kho `20 Viên`.
* **When:** Dược sĩ bán 10 Viên thuốc X.
* **Then:** Thuật toán FEFO tự động bỏ qua Lô 1 (dù hạn gần hơn nhưng đã quá hạn), và trừ đúng 10 viên từ Lô 2. Lô 2 còn lại `10 Viên`.

### Kịch bản 4: Hiệu năng thực thi (NFR-PER-01)
* **Given:** Một hóa đơn bán lẻ tại POS có 20 mặt hàng thuốc khác nhau, mỗi mặt hàng có trung bình 3-5 Lô tồn kho.
* **When:** Dược sĩ bấm "Thanh toán".
* **Then:** Thuật toán phân bổ FEFO và tính toán hoàn tất trong thời gian **dưới 1 giây** ($< 1s$).

---

## 6. ĐẶC TẢ KỸ THUẬT & DỮ LIỆU LIÊN QUAN (TECHNICAL & DATA SPECS)

### 6.1. Thực thể CSDL liên quan
* Bảng `TON_KHO_LO` (Cập nhật số lượng)
* Bảng `LO_HANG` (Truy vấn HSD và trạng thái)
* Bảng `CHI_TIET_HOA_DON` (Lưu thông tin Lô được trừ cho từng mặt hàng)
* Bảng `LICH_SU_KHO` (Ghi nhận giao dịch xuất kho theo Lô)

### 6.2. Truy vấn phân bổ có Lock chống tranh chấp đồng thời (SQL Server)
```sql
SELECT 
    TK.MaLo,
    LH.SoLoNSX,
    LH.HanSuDung,
    TK.SoLuongTonGoc
FROM TON_KHO_LO TK WITH (UPDLOCK, ROWLOCK)
INNER JOIN LO_HANG LH ON TK.MaLo = LH.MaLo
WHERE LH.MaThuoc = @MaThuoc
  AND TK.SoLuongTonGoc > 0
  AND LH.HanSuDung > CAST(GETDATE() AS DATE)
  AND LH.TrangThai = 1
ORDER BY LH.HanSuDung ASC;
```

---

## 7. MỐI QUAN HỆ & ĐIỀU KIỆN TIÊN QUYẾT (TRACEABILITY & DEPENDENCIES)

* **Điều kiện tiên quyết:**
  * **US01, US02:** Có dữ liệu thuốc và tỷ lệ quy đổi ĐVT.
  * **US04, US05:** Có dữ liệu Lô, HSD và số tồn kho gốc trong `TON_KHO_LO`.
  * **US08:** Được kích hoạt từ luồng bán hàng POS.
* **Tương tác với:**
  * **US10:** Dược sĩ có thể bấm "Override FEFO" để can thiệp lại kết quả phân bổ này.
  * **US11:** Khi đổi trả hàng, cần hoàn lại đúng các Lô mà US09 đã trừ.
  * **US14:** Dữ liệu trừ Lô của US09 được dùng để tính giá vốn chính xác theo từng Lô bán ra.
