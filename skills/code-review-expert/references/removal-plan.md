# Mẫu kế hoạch xoá code & lặp lại

## Mức ưu tiên

- [ ] **P0**: Xoá ngay (rủi ro bảo mật, chi phí lớn, chặn việc khác)
- [ ] **P1**: Xoá trong sprint hiện tại
- [ ] **P2**: Backlog / vòng lặp sau

---

## Xoá được ngay

### Mục: [Tên/Mô tả]

| Trường | Chi tiết |
|---|---|
| **Vị trí** | `path/to/file.ts:line` |
| **Lý do** | Vì sao nên xoá |
| **Bằng chứng** | Không còn tham chiếu, feature flag đã tắt, API đã deprecated |
| **Tác động** | Không có / Thấp — không còn ai dùng |
| **Các bước xoá** | 1. Xoá code 2. Xoá test 3. Xoá config |
| **Kiểm chứng** | Chạy test, không lỗi runtime, theo dõi log |

---

## Hoãn xoá (cần kế hoạch)

### Mục: [Tên/Mô tả]

| Trường | Chi tiết |
|---|---|
| **Vị trí** | `path/to/file.ts:line` |
| **Vì sao hoãn** | Còn người dùng, cần migrate, cần người liên quan đồng ý |
| **Điều kiện trước** | Feature flag tắt 2 tuần, telemetry cho thấy 0 lượt dùng |
| **Thay đổi phá vỡ** | Liệt kê thay đổi API/contract |
| **Kế hoạch migrate** | Các bước để bên dùng chuyển đổi |
| **Thời hạn** | Ngày hoặc sprint mục tiêu |
| **Người phụ trách** | Người/team chịu trách nhiệm |
| **Kiểm chứng** | Chỉ số xác nhận xoá an toàn (tỉ lệ lỗi, lượt dùng) |
| **Rollback** | Cách khôi phục nếu có sự cố |

---

## Checklist trước khi xoá

- [ ] Đã tìm mọi tham chiếu trong codebase (`rg`, `grep`)
- [ ] Đã kiểm tra cách dùng động/reflection (tên ghép chuỗi, gọi qua config)
- [ ] Đã xác nhận không có bên ngoài dùng (API, SDK, docs)
- [ ] Đã xem telemetry của feature flag (nếu có)
- [ ] Đã cập nhật/xoá test
- [ ] Đã cập nhật tài liệu
- [ ] Đã báo team (nếu là code dùng chung)
