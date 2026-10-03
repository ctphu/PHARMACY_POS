# PHARMACY POS & INVENTORY SYSTEM
## Hệ Thống Quản Lý Hiệu Thuốc Bán Lẻ & Kho Dược

Dự án phân tích, thiết kế và phát triển hệ thống POS & Quản lý Kho Dược phẩm tập trung vào bài toán cốt lõi:
- **Quản lý Đa Đơn vị tính (ĐVT) & Tỷ lệ Quy đổi** (Thùng, Hộp, Vỉ, Viên,... quy về ĐVT Gốc).
- **Quản lý Kho theo Lô (Batch/Lot) & Hạn sử dụng (Exp Date)**.
- **Tự động Gợi ý & Trừ Kho theo nguyên tắc FEFO (First Expired, First Out)**.
- **Cảnh báo thuốc cận date / hết hạn & Báo cáo doanh thu, tồn kho**.

---

## 📁 Cấu Trúc Tài Liệu & Dự Án

```text
PHARMACY_POS/
├── DacTa.md                 # Tài liệu Đặc tả Yêu cầu Nghiệp vụ (15 Use Cases chi tiết)
├── SRS.md                   # Software Requirements Specification (Đặc tả Yêu cầu Phần mềm)
├── docs/
│   └── feature_tracking.md  # Ma trận & Bảng theo dõi tiến độ phát triển tính năng
├── user_stories/            # Danh mục 15 User Stories chuẩn Agile / Scrum
│   ├── README.md            # Tổng quan & Ma trận truy vết Traceability Matrix
│   ├── US01_QuanLyDanhMucThuoc.md
│   ├── US02_ThietLapDVT_TyLeQuyDoi.md
│   ├── US03_ThietLapBangGiaTheoDVT.md
│   ├── US04_LapPhieuNhapKhoTheoLoHSD.md
│   ├── US05_QuyDoiSoLuongNhapVeDVTGoc.md
│   ├── US06_KiemKeKho_DieuChinhTonLo.md
│   ├── US07_XuLyThuocHetHan_MatPhamChat.md
│   ├── US08_LapHoaDonBanLePOS.md
│   ├── US09_TuDongGoiY_TruKhoFEFO.md
│   ├── US10_ChonLoThuCongOverrideFEFO.md
│   ├── US11_DoiTraHangBanLe.md
│   ├── US12_Quet_CanhBaoThuocSapHetHan.md
│   ├── US13_XemBaoCaoTonKhoChiTietTheoLoVaDVT.md
│   ├── US14_XemBaoCaoDoanhThuLoiNhuan.md
│   └── US15_QuanLyTaiKhoan_PhanQuyen.md
└── README.md
```

---

## 🎯 4 Phân Hệ Nghiệp Vụ Cốt Lõi

1. **Phân hệ 1: Quản lý Danh mục & Đơn vị tính (Master Data)**
   - Danh mục thuốc, hoạt chất, nhóm thuốc.
   - Cây quy đổi ĐVT đa cấp (ví dụ: 1 Hộp = 10 Vỉ = 100 Viên).
   - Thiết lập bảng giá bán và tỷ lệ chiết khấu linh hoạt theo ĐVT.

2. **Phân hệ 2: Quản lý Nhập kho & Tồn kho Lô/HSD (Inventory)**
   - Nhập kho theo số lô và hạn sử dụng từ nhà cung cấp.
   - Tự động quy đổi số lượng nhập về ĐVT gốc để theo dõi tồn kho chính xác tuyệt đối.
   - Kiểm kê, cân bằng kho và xuất hủy thuốc hỏng/hết hạn.

3. **Phân hệ 3: Bán lẻ tại Quầy POS & Thuật toán FEFO (POS)**
   - Giao diện bán hàng nhanh, hỗ trợ phím tắt, tìm kiếm thông minh.
   - Thuật toán FEFO tự động đề xuất xuất bán lô có hạn sử dụng gần nhất.
   - Cho phép Dược sĩ ghi đè (Override) chọn lô thủ công có kiểm soát và log vết.
   - Xử lý đổi trả hàng và hoàn trả kho chuẩn xác.

4. **Phân hệ 4: Cảnh báo, Báo cáo & Quản trị Hệ thống (Admin & Reports)**
   - Cron job / Scheduler tự động quét và cảnh báo thuốc cận hạn theo ngưỡng cấu hình.
   - Báo cáo tồn kho đa chiều (theo lô, hạn dùng, ĐVT).
   - Báo cáo doanh thu, giá vốn, lợi nhuận gộp theo thời gian thực.
   - Phân quyền người dùng dựa trên vai trò (RBAC).

---

## 📌 Liên Kết Tham Khảo
- [Đặc tả Hệ thống (DacTa.md)](DacTa.md)
- [Software Requirements Specification (SRS.md)](SRS.md)
- [Danh mục User Stories (user_stories/README.md)](user_stories/README.md)
- [Bảng theo dõi tiến độ (docs/feature_tracking.md)](docs/feature_tracking.md)
