# DANH MỤC CÁC USER STORY (AGILE USER STORIES)
## HỆ THỐNG QUẢN LÝ HIỆU THUỐC BÁN LẺ (PHARMACY POS & INVENTORY)
**Chủ đề trọng tâm:** Quản lý Lô - Hạn dùng (FEFO) & Quy đổi Đơn vị tính (ĐVT)  
**Tài liệu cơ sở:** [DacTa.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/DacTa.md) và [SRS.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/SRS.md)  
**Phiên bản:** 1.0  
**Ngày lập:** 03/10/2026  

---

## 1. TỔNG QUAN HỆ THỐNG USER STORIES

Hệ thống User Story được thiết kế chuẩn mực theo phương pháp Agile/Scrum, đóng vai trò làm cầu nối giữa các yêu cầu nghiệp vụ chuyên sâu trong tài liệu Đặc tả (DacTa.md / SRS.md) với đội ngũ phát triển phần mềm (Developers) và đội ngũ kiểm thử chất lượng (QA/QC).

Mỗi User Story được tổ chức thành một file Markdown độc lập trong thư mục `user_stories/`, tuân thủ cấu trúc chuẩn:
1. **Thông tin chung (Metadata):** Mã US, Tên, Phân hệ, Mức độ ưu tiên (MoSCoW), Tác nhân, Trạng thái.
2. **Nội dung User Story (Story Statement):** Định dạng *As a... / I want to... / So that...*
3. **Bối cảnh & Nghiệp vụ thực tế (Business Context).**
4. **Quy tắc nghiệp vụ & Thuật toán (Business Rules & Algorithms).**
5. **Tiêu chí nghiệm thu (Acceptance Criteria - AC):** Kịch bản kiểm thử chuẩn BDD/Gherkin (*Given - When - Then*).
6. **Đặc tả kỹ thuật & CSDL liên quan (Technical & Data Specs):** Schema CSDL, Pseudo-code, NFR, UI/UX shortcuts.
7. **Truy vết & Điều kiện phụ thuộc (Traceability & Dependencies).**

---

## 2. MA TRẬN TRUY VẾT YÊU CẦU (TRACEABILITY MATRIX)

Bảng đối chiếu 1-1 giữa User Stories với Use Case ([DacTa.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/DacTa.md)) và Yêu cầu chức năng ([SRS.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/SRS.md)):

| Mã US | Tên User Story | Phân hệ | Use Case (DacTa.md) | Yêu cầu SRS (SRS.md) | Tác nhân chính | File chi tiết |
| :---: | :--- | :---: | :---: | :---: | :---: | :--- |
| **US01** | Quản lý Danh mục Thuốc | Phân hệ 1 | **UC01** | **[FR-MD-01]** | Quản lý | [US01_QuanLyDanhMucThuoc.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US01_QuanLyDanhMucThuoc.md) |
| **US02** | Thiết lập ĐVT & Tỷ lệ Quy đổi | Phân hệ 1 | **UC02** | **[FR-MD-02]** | Quản lý | [US02_ThietLapDVT_TyLeQuyDoi.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US02_ThietLapDVT_TyLeQuyDoi.md) |
| **US03** | Thiết lập Bảng giá theo ĐVT | Phân hệ 1 | **UC03** | **[FR-MD-03]** | Quản lý | [US03_ThietLapBangGiaTheoDVT.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US03_ThietLapBangGiaTheoDVT.md) |
| **US04** | Lập Phiếu Nhập kho theo Lô/HSD | Phân hệ 2 | **UC04** | **[FR-INV-01]** | Thủ kho | [US04_LapPhieuNhapKhoTheoLoHSD.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US04_LapPhieuNhapKhoTheoLoHSD.md) |
| **US05** | Tự động Quy đổi Nhập về ĐVT Gốc | Phân hệ 2 | **UC05** | **[FR-INV-02]** | Hệ thống / Thủ kho | [US05_QuyDoiSoLuongNhapVeDVTGoc.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US05_QuyDoiSoLuongNhapVeDVTGoc.md) |
| **US06** | Kiểm kê Kho & Điều chỉnh Tồn Lô | Phân hệ 2 | **UC06** | **[FR-INV-03]** | Thủ kho / Quản lý | [US06_KiemKeKho_DieuChinhTonLo.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US06_KiemKeKho_DieuChinhTonLo.md) |
| **US07** | Xử lý Thuốc Hết hạn & Mất phẩm chất | Phân hệ 2 | **UC07** | **[Mức Đen]** | Thủ kho / Quản lý | [US07_XuLyThuocHetHan_MatPhamChat.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US07_XuLyThuocHetHan_MatPhamChat.md) |
| **US08** | Lập Hóa đơn Bán lẻ tại POS | Phân hệ 3 | **UC08** | **[FR-POS-01]** | Dược sĩ | [US08_LapHoaDonBanLePOS.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US08_LapHoaDonBanLePOS.md) |
| **US09** | Tự động Gợi ý & Trừ Kho theo FEFO | Phân hệ 3 | **UC09** | **[FR-POS-02]** | Dược sĩ / Hệ thống | [US09_TuDongGoiY_TruKhoFEFO.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US09_TuDongGoiY_TruKhoFEFO.md) |
| **US10** | Chọn Lô Thủ công (Override FEFO) | Phân hệ 3 | **UC10** | **[FR-POS-03]** | Dược sĩ | [US10_ChonLoThuCongOverrideFEFO.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US10_ChonLoThuCongOverrideFEFO.md) |
| **US11** | Xử lý Đổi / Trả Hàng Bán lẻ | Phân hệ 3 | **UC11** | **[Phân hệ 3.3]** | Dược sĩ | [US11_DoiTraHangBanLe.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US11_DoiTraHangBanLe.md) |
| **US12** | Quét & Cảnh báo Thuốc Cận hạn / Hết hạn | Phân hệ 4 | **UC12** | **[FR-REP-01]** | Hệ thống / Thủ kho | [US12_Quet_CanhBaoThuocSapHetHan.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US12_Quet_CanhBaoThuocSapHetHan.md) |
| **US13** | Báo cáo Tồn kho Chi tiết theo Lô & ĐVT | Phân hệ 4 | **UC13** | **[FR-REP-02]** | Quản lý / Thủ kho | [US13_XemBaoCaoTonKhoChiTietTheoLoVaDVT.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US13_XemBaoCaoTonKhoChiTietTheoLoVaDVT.md) |
| **US14** | Báo cáo Doanh thu & Lợi nhuận Bán lẻ | Phân hệ 4 | **UC14** | **[Báo cáo TC]** | Quản lý | [US14_XemBaoCaoDoanhThuLoiNhuan.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US14_XemBaoCaoDoanhThuLoiNhuan.md) |
| **US15** | Quản lý Tài khoản & Phân quyền RBAC | Phân hệ 4 | **UC15** | **[NFR-SEC-01]** | Quản lý | [US15_QuanLyTaiKhoan_PhanQuyen.md](file:///d:/0%20-%20Client/VIDO/PTTKHTTH/PHARMACY_POS/user_stories/US15_QuanLyTaiKhoan_PhanQuyen.md) |

---

## 3. PHÂN BỔ THEO PHÂN HỆ CHỨC NĂNG

```
HỆ THỐNG QUẢN LÝ HIỆU THUỐC BÁN LẺ (PHARMACY POS)
├── Phân hệ 1: Quản lý Danh mục & Quy đổi ĐVT (Master Data)
│   ├── US01: Quản lý Danh mục Thuốc
│   ├── US02: Thiết lập ĐVT & Tỷ lệ Quy đổi
│   └── US03: Thiết lập Bảng giá theo ĐVT
├── Phân hệ 2: Quản lý Nhập kho & Lô - Hạn dùng (Inventory & Batch Management)
│   ├── US04: Lập Phiếu Nhập kho theo Lô/HSD
│   ├── US05: Tự động Quy đổi Nhập về ĐVT Gốc
│   ├── US06: Kiểm kê Kho & Điều chỉnh Tồn Lô
│   └── US07: Xử lý Thuốc Hết hạn & Mất phẩm chất
├── Phân hệ 3: Bán hàng tại Điểm bán (POS) & Phân bổ FEFO
│   ├── US08: Lập Hóa đơn Bán lẻ tại POS
│   ├── US09: Tự động Gợi ý & Trừ Kho theo FEFO (Core Logic)
│   ├── US10: Chọn Lô Thủ công (Override FEFO)
│   └── US11: Xử lý Đổi / Trả Hàng Bán lẻ
└── Phân hệ 4: Cảnh báo, Báo cáo & Quản trị (Alerts, Reports & Admin)
    ├── US12: Quét & Cảnh báo Thuốc Cận hạn / Hết hạn
    ├── US13: Báo cáo Tồn kho Chi tiết theo Lô & Đa Quy đổi
    ├── US14: Báo cáo Doanh thu & Lợi nhuận Bán lẻ
    └── US15: Quản lý Tài khoản & Phân quyền RBAC
```

---

## 4. MA TRẬN TÁC NHÂN VS USER STORY

| User Story | Dược sĩ bán hàng | Thủ kho | Quản lý / Chủ nhà thuốc | Hệ thống Tự động (Worker) |
| :--- | :---: | :---: | :---: | :---: |
| **US01** | Tra cứu | Tra cứu | **Chính (CRUD)** | |
| **US02** | Xem | Xem | **Chính (CRUD)** | Kiểm tra quy đổi |
| **US03** | Áp dụng giá | | **Chính (CRUD)** | |
| **US04** | | **Chính (Lập phiếu)** | Phê duyệt (nếu cần) | |
| **US05** | | Sử dụng | | **Chính (Quy đổi ngầm)** |
| **US06** | | **Chính (Đếm thực tế)** | **Chính (Duyệt cân bằng)** | |
| **US07** | | **Chính (Lập biên bản)** | **Chính (Duyệt xuất hủy)** | Khóa bán tự động |
| **US08** | **Chính (Bán hàng)** | | Xem | |
| **US09** | **Chính (Thực hiện)** | | Xem | **Chính (Thuật toán FEFO)** |
| **US10** | **Chính (Override)** | | Giám sát Audit | |
| **US11** | **Chính (Đổi trả)** | | Giám sát | Hoàn tồn Lô |
| **US12** | Xem | **Chính (Xử lý)** | **Chính (Kế hoạch)** | **Chính (Cron job 00:00)** |
| **US13** | Tra cứu | **Chính (Kiểm đếm)** | **Chính (Báo cáo)** | Phân rã chuỗi ĐVT |
| **US14** | | | **Chính (Quản trị)** | Tính giá vốn theo Lô |
| **US15** | | | **Chính (Quản trị)** | Kiểm soát phiên & JWT |
