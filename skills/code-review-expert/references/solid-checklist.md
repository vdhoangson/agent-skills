# Checklist SOLID & code smell

## SRP (Single Responsibility — Một trách nhiệm)

- File gánh nhiều mối quan tâm không liên quan (vd: HTTP + DB + nghiệp vụ trong một file)
- Class/module lớn, gắn kết thấp, có nhiều lý do để thay đổi
- Hàm điều phối nhiều bước không liên quan
- God object biết quá nhiều về hệ thống
- **Hỏi**: "Module này thay đổi vì một lý do duy nhất nào?"

## OCP (Open/Closed — Mở để mở rộng, đóng để sửa đổi)

- Thêm hành vi mới phải sửa nhiều khối switch/if
- Tính năng mới đòi sửa logic lõi thay vì mở rộng
- Không có điểm strategy/hook/plugin cho các biến thể
- **Hỏi**: "Thêm biến thể mới có cần đụng code cũ không?"

## LSP (Liskov Substitution — Thay thế được)

- Lớp con kiểm tra kiểu cụ thể, hoặc throw ở method của lớp cha
- Method override nới lỏng điều kiện đầu vào hoặc siết điều kiện đầu ra
- Lớp con bỏ qua hoặc để trống hành vi của lớp cha
- **Hỏi**: "Thay bất kỳ lớp con nào mà caller không cần biết được không?"

## ISP (Interface Segregation — Tách interface)

- Interface nhiều method, phần lớn không được implementer dùng
- Caller phụ thuộc interface rộng cho nhu cầu hẹp
- Implementation rỗng/stub cho method của interface
- **Hỏi**: "Mọi implementer có dùng hết các method không?"

## DIP (Dependency Inversion — Đảo ngược phụ thuộc)

- Logic cấp cao phụ thuộc trực tiếp kiểu IO, storage, network cụ thể
- Hard-code implementation thay vì inject
- Chuỗi import gắn chặt nghiệp vụ với hạ tầng
- **Hỏi**: "Đổi implementation có phải sửa nghiệp vụ không?"

---

## Code smell phổ biến (ngoài SOLID)

| Smell | Dấu hiệu |
|---|---|
| **Hàm quá dài** | > 30 dòng, lồng nhiều cấp |
| **Feature envy** | Method dùng dữ liệu của class khác nhiều hơn của chính nó |
| **Data clumps** | Cùng một nhóm tham số luôn đi cùng nhau |
| **Primitive obsession** | Dùng string/number thay vì kiểu domain |
| **Shotgun surgery** | Một thay đổi phải sửa nhiều file |
| **Divergent change** | Một file thay đổi vì nhiều lý do không liên quan |
| **Dead code** | Code không bao giờ chạy tới / không ai gọi |
| **Speculative generality** | Abstraction cho nhu cầu giả định trong tương lai |
| **Magic number/string** | Giá trị hard-code không có tên hằng |

---

## Nguyên tắc refactor

1. **Tách theo trách nhiệm, không theo kích thước** — file nhỏ vẫn có thể vi phạm SRP
2. **Chỉ thêm abstraction khi cần** — đợi trường hợp dùng thứ hai
3. **Refactor từng bước** — cô lập hành vi trước khi di chuyển
4. **Giữ nguyên hành vi trước** — thêm test trước khi tái cấu trúc
5. **Đặt tên theo ý định** — khó đặt tên thường là dấu hiệu abstraction sai
6. **Ưu tiên composition hơn inheritance** — kế thừa tạo phụ thuộc chặt
7. **Làm trạng thái sai không thể biểu diễn** — dùng kiểu dữ liệu để đảm bảo bất biến
