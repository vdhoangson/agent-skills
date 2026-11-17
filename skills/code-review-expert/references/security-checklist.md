# Checklist bảo mật & độ tin cậy

## An toàn input/output

- **XSS**: chèn HTML không an toàn, `dangerouslySetInnerHTML`, template không escape, gán `innerHTML`
- **Injection**: SQL/NoSQL/command/GraphQL injection qua nối chuỗi hoặc template literal
- **SSRF**: URL do người dùng kiểm soát gọi tới dịch vụ nội bộ mà không có allowlist
- **Path traversal**: input người dùng nằm trong đường dẫn file mà không làm sạch (`../`)
- **Prototype pollution**: merge object không an toàn trong JavaScript (`Object.assign`, spread với input người dùng)

## Xác thực / phân quyền (AuthN/AuthZ)

- Thiếu kiểm tra tenant hoặc quyền sở hữu khi đọc/ghi
- Endpoint mới không có auth guard hoặc RBAC
- Tin vào role/flag/ID do client gửi lên
- Lỗi kiểm soát truy cập (IDOR — truy cập trực tiếp object của người khác)
- Session fixation hoặc quản lý session yếu

## JWT & token

- Tấn công nhầm thuật toán (chấp nhận `none` hoặc `HS256` khi đáng lẽ là `RS256`)
- Secret yếu hoặc hard-code
- Thiếu hoặc không kiểm tra hạn (`exp`)
- Dữ liệu nhạy cảm trong payload JWT (token chỉ base64, không mã hoá)
- Không kiểm tra `iss` (issuer) hoặc `aud` (audience)

## Secret & dữ liệu cá nhân (PII)

- API key, token, credential trong code/config/log
- Secret trong lịch sử git hoặc biến môi trường lộ ra client
- Log quá nhiều PII hoặc payload nhạy cảm
- Thiếu che dữ liệu trong thông báo lỗi

## Chuỗi cung ứng & dependency

- Dependency không ghim phiên bản, cho phép cập nhật độc hại
- Dependency confusion (trùng tên package private)
- Import từ nguồn/CDN không tin cậy mà không kiểm tra integrity
- Dependency cũ có CVE đã biết

## CORS & header

- CORS quá rộng (`Access-Control-Allow-Origin: *` kèm credentials)
- Thiếu security header (CSP, X-Frame-Options, X-Content-Type-Options)
- Lộ header nội bộ hoặc stack trace

## Rủi ro runtime

- Vòng lặp, đệ quy không giới hạn, buffer lớn trong bộ nhớ
- Thiếu timeout, retry, rate limit khi gọi dịch vụ ngoài
- Thao tác chặn trên request path (I/O đồng bộ trong ngữ cảnh async)
- Cạn tài nguyên (file handle, connection, bộ nhớ)
- ReDoS (regex có thể bị khai thác gây treo)

## Mật mã

- Thuật toán yếu (MD5, SHA1 cho mục đích bảo mật)
- IV hoặc salt hard-code
- Mã hoá không xác thực (chế độ ECB, không có HMAC)
- Độ dài khoá không đủ

## Race condition

Race condition gây lỗi chập chờn và lỗ hổng bảo mật, rất khó tái hiện. Chú ý đặc biệt:

### Truy cập trạng thái dùng chung
- Nhiều thread/goroutine/async task truy cập biến chung không đồng bộ
- Trạng thái global hoặc singleton bị sửa đồng thời
- Khởi tạo lười không có lock (lỗi double-checked locking)
- Collection không thread-safe dùng trong ngữ cảnh đồng thời

### Check-then-act (TOCTOU)
- Pattern `if (tồn tại) thì dùng` không nguyên tử
- `if (có quyền) thì thực hiện` khi quyền có thể đổi giữa chừng
- Kiểm tra file tồn tại rồi mới thao tác file
- Kiểm tra số dư rồi mới trừ tiền (thao tác tài chính)
- Kiểm tra tồn kho rồi mới đặt hàng

### Đồng thời trong database
- Thiếu optimistic locking (cột `version`, kiểm tra `updated_at`)
- Thiếu pessimistic locking (`SELECT FOR UPDATE`)
- Đọc-sửa-ghi không có transaction isolation
- Tăng bộ đếm không nguyên tử (nên dùng `UPDATE SET count = count + 1`)
- Vi phạm unique constraint khi insert đồng thời

### Hệ phân tán
- Thiếu distributed lock cho tài nguyên chung
- Race khi bầu leader
- Race khi invalidate cache (đọc dữ liệu cũ sau khi ghi)
- Phụ thuộc thứ tự event mà không có cơ chế sắp thứ tự
- Split-brain trong cluster

### Pattern nguy hiểm cần báo
```
# Nguy hiểm:
if not exists(key):       # TOCTOU
    create(key)

value = get(key)          # Đọc-sửa-ghi
value += 1
set(key, value)

if user.balance >= amount:  # Check-then-act
    user.balance -= amount
```

### Câu hỏi nên đặt
- "Hai request chạy đoạn này cùng lúc thì sao?"
- "Thao tác này có nguyên tử không, có bị ngắt giữa chừng không?"
- "Đoạn code này truy cập trạng thái chung nào?"
- "Dưới tải đồng thời cao thì hành vi ra sao?"

## Toàn vẹn dữ liệu

- Thiếu transaction, ghi dở dang, cập nhật trạng thái không nhất quán
- Validate yếu trước khi lưu (lỗi ép kiểu)
- Thiếu idempotency cho thao tác có thể retry
- Mất cập nhật do sửa đồng thời
