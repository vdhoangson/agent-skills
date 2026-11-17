# Checklist chất lượng code

## Xử lý lỗi

### Anti-pattern cần báo

- **Nuốt exception**: catch rỗng hoặc chỉ log rồi bỏ qua
  ```javascript
  try { ... } catch (e) { }  // Lỗi âm thầm
  try { ... } catch (e) { console.log(e) }  // Log rồi quên
  ```
- **Catch quá rộng**: bắt lớp gốc `Exception`/`Error` thay vì kiểu cụ thể
- **Lộ thông tin lỗi**: stack trace hoặc chi tiết nội bộ trả về cho người dùng
- **Thiếu xử lý lỗi**: không bọc các thao tác có thể lỗi (I/O, network, parse)
- **Lỗi async**: promise rejection không được xử lý, thiếu `.catch()`, thiếu error boundary

### Thực hành tốt cần kiểm tra

- [ ] Lỗi được bắt ở đúng ranh giới
- [ ] Thông báo lỗi thân thiện, không lộ chi tiết nội bộ
- [ ] Log lỗi đủ ngữ cảnh để debug
- [ ] Lỗi async được truyền lên hoặc xử lý đúng
- [ ] Có hành vi fallback cho lỗi khôi phục được
- [ ] Lỗi nghiêm trọng kích hoạt cảnh báo/monitoring

### Câu hỏi nên đặt
- "Thao tác này lỗi thì chuyện gì xảy ra?"
- "Caller có biết đã có lỗi không?"
- "Có đủ ngữ cảnh để debug lỗi này không?"

---

## Hiệu năng & cache

### Thao tác tốn CPU

- **Thao tác nặng trên hot path**: compile regex, parse JSON, crypto trong vòng lặp
- **Chặn main thread**: I/O đồng bộ, tính toán nặng không dùng worker/async
- **Tính lại không cần thiết**: cùng một phép tính chạy nhiều lần
- **Thiếu memoization**: hàm thuần được gọi lặp lại với cùng input

### Database & I/O

- **N+1 query**: vòng lặp query từng phần tử thay vì gom lại
  ```javascript
  // Sai: N+1
  for (const id of ids) {
    const user = await db.query(`SELECT * FROM users WHERE id = ?`, id)
  }
  // Đúng: gom một lần
  const users = await db.query(`SELECT * FROM users WHERE id IN (?)`, ids)
  ```
- **Thiếu index**: query trên cột không có index
- **Lấy thừa dữ liệu**: `SELECT *` khi chỉ cần vài cột
- **Không phân trang**: nạp toàn bộ dataset vào bộ nhớ

### Vấn đề cache

- **Thiếu cache cho thao tác đắt**: gọi API, query DB, tính toán lặp lại
- **Cache không có TTL**: dữ liệu cũ được trả mãi
- **Không có chiến lược invalidate**: dữ liệu đổi nhưng cache không xoá
- **Trùng cache key**: key không đủ phân biệt
- **Cache dữ liệu riêng của user ở phạm vi global**: lỗi bảo mật/riêng tư

### Bộ nhớ

- **Collection không giới hạn**: mảng/map lớn dần không có trần
- **Giữ tham chiếu object lớn**: cản garbage collector
- **Nối chuỗi trong vòng lặp**: dùng StringBuilder/join
- **Đọc cả file lớn vào bộ nhớ**: dùng stream

### Câu hỏi nên đặt
- "Độ phức tạp thời gian của thao tác này là bao nhiêu?"
- "Dữ liệu gấp 10x/100x thì sao?"
- "Kết quả này có cache được không? Có nên cache không?"
- "Có gom lại xử lý một lần thay vì từng cái được không?"

---

## Điều kiện biên

### Null/undefined

- **Thiếu kiểm tra null**: truy cập thuộc tính của object có thể null
- **Nhầm truthy/falsy**: `if (value)` trong khi `0` hoặc `""` là giá trị hợp lệ
- **Lạm dụng optional chaining**: `a?.b?.c?.d` che giấu vấn đề cấu trúc
- **Lẫn lộn null và undefined**: dùng lẫn không có quy ước

### Collection rỗng

- **Không xử lý mảng rỗng**: code giả định mảng luôn có phần tử
- **Object rỗng**: `for...in` hoặc `Object.keys` trên object rỗng
- **Lấy phần tử đầu/cuối**: `arr[0]` hoặc `arr[arr.length-1]` không kiểm tra độ dài

### Biên số

- **Chia cho 0**: không kiểm tra trước khi chia
- **Tràn số nguyên**: số lớn vượt phạm vi an toàn
- **So sánh số thực**: dùng `===` thay vì so với sai số epsilon
- **Giá trị âm**: index hoặc số đếm không được âm
- **Off-by-one**: biên vòng lặp, cắt mảng, phân trang

### Biên chuỗi

- **Chuỗi rỗng**: không được xử lý như trường hợp biên
- **Chuỗi toàn khoảng trắng**: qua được kiểm tra truthy nhưng thực chất rỗng
- **Chuỗi rất dài**: không giới hạn độ dài, gây vấn đề bộ nhớ/hiển thị
- **Unicode**: emoji, chữ RTL, ký tự tổ hợp (dấu tiếng Việt dạng tổ hợp vs dựng sẵn)

### Pattern nguy hiểm cần báo

```javascript
// Nguy hiểm: không kiểm tra null
const name = user.profile.name

// Nguy hiểm: truy cập mảng không kiểm tra
const first = items[0]

// Nguy hiểm: chia không kiểm tra
const avg = total / count

// Nguy hiểm: truthy loại bỏ giá trị hợp lệ
if (value) { ... }  // sai với 0, "", false
```

### Câu hỏi nên đặt
- "Nếu giá trị này null/undefined thì sao?"
- "Nếu collection rỗng thì sao?"
- "Phạm vi hợp lệ của số này là gì?"
- "Ở biên (0, -1, MAX_INT) thì chuyện gì xảy ra?"
