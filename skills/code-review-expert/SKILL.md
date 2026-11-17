---
name: code-review-expert
description: Review code chuyên sâu với góc nhìn senior engineer — tìm bug logic, lỗ hổng bảo mật, race condition, lỗi xử lý ngoại lệ, vấn đề hiệu năng, vi phạm SOLID và code nên xoá; phân mức P0–P3, kiểm chứng từng lỗi để loại false positive. Nhận diff hiện tại, số PR, commit, commit range hoặc branch. Dùng skill này bất cứ khi nào người dùng nói "review", "review code", "review PR", "xem giúp diff", "check commit này", "code này có vấn đề gì không", "trước khi merge", kể cả khi không nhắc tên skill. Ưu tiên skill này hơn ponytail-review, code-review, security-review khi người dùng muốn review kỹ/chi tiết, review PR hoặc branch trước khi merge, hoặc thay đổi đụng auth, thanh toán, dữ liệu; ponytail-review chỉ dành cho kiểm tra nhanh thay đổi nhỏ.
---

# Code Review Expert

Review có cấu trúc với góc nhìn senior engineer. **Chỉ review, không sửa** cho tới khi người dùng xác nhận — người dùng cần thấy toàn cảnh trước khi quyết định sửa gì.

Trả lời bằng **tiếng Việt, ngắn gọn, đúng trọng tâm**.

## Mức độ nghiêm trọng

| Mức | Tên | Ví dụ | Hành động |
|---|---|---|---|
| **P0** | Nghiêm trọng | Lỗ hổng bảo mật, mất dữ liệu, bug sai kết quả | Chặn merge |
| **P1** | Cao | Lỗi logic, race condition, hồi quy hiệu năng, vi phạm SOLID nặng | Sửa trước khi merge |
| **P2** | Trung bình | Code smell, khó bảo trì, thiếu xử lý biên | Sửa trong PR này hoặc tạo follow-up |
| **P3** | Thấp | Đặt tên, style, gợi ý nhỏ | Tuỳ chọn |

## Quy trình

### 1) Xác định phạm vi review

Đoán mục tiêu từ lời người dùng:

| Người dùng đưa | Lệnh lấy diff |
|---|---|
| Không nói gì | `git status -sb`, `git diff --stat`, `git diff` (chưa stage) + `git diff --cached` (đã stage) |
| Số PR / link PR | `gh pr view <n>`, `gh pr diff <n>` |
| 1 commit | `git show <sha>` |
| Commit range | `git diff <a>..<b>` |
| Branch | `git diff <base>...<branch>` (base mặc định: nhánh chính của repo) |

- **Không có thay đổi**: báo người dùng, hỏi muốn review commit/branch/PR nào.
- Dùng `rg`/`grep` tìm module liên quan, caller, contract. Diff chỉ là một phần — bug thường nằm ở chỗ code mới gặp code cũ.
- Đánh dấu đường đi quan trọng: auth, thanh toán, ghi dữ liệu, network, migration.

### 2) Chọn chế độ: tự review hay song song

- **Diff nhỏ (≤ ~300 dòng, ít file)**: session chính tự review đủ các mảng ở bước 3.
- **Diff lớn**: review song song bằng subagent (bước 2a). Một agent đọc 2.000 dòng sẽ bỏ sót; chia theo mảng giúp mỗi agent tập trung.

#### 2a) Review song song

1. **Kiểm tra tài nguyên** trước khi tạo subagent: `free -m; nproc; uptime`. RAM ≤ 90%, CPU (load/số nhân) ≤ 80%. Gần ngưỡng → giảm số agent hoặc chạy tuần tự.
2. Tạo tối đa 3 subagent `general-purpose`, `model: "sonnet"`, `effort: "xhigh"`, chạy cùng lúc, mỗi agent một mảng:
   - **Bảo mật & độ tin cậy** — đọc `references/security-checklist.md`
   - **Chất lượng code** (xử lý lỗi, hiệu năng, điều kiện biên) — đọc `references/code-quality-checklist.md`
   - **Kiến trúc & SOLID + code nên xoá** — đọc `references/solid-checklist.md` và `references/removal-plan.md`
3. Brief cho mỗi agent phải tự đủ: lệnh lấy diff (bước 1), đường dẫn tuyệt đối tới file checklist, yêu cầu **chỉ đọc, không sửa file**, và trả về danh sách finding theo dạng: `file:line`, mức P0–P3, mô tả, **kịch bản lỗi cụ thể**, đề xuất sửa.
4. Session chính gộp kết quả, **loại trùng** (cùng gốc lỗi báo ở nhiều mảng → giữ một, lấy mức cao nhất), rồi chuyển sang bước 4.

### 3) Các mảng cần review

**Kiến trúc & SOLID** — `references/solid-checklist.md`
- SRP, OCP, LSP, ISP, DIP và các code smell phổ biến.
- Đề xuất refactor phải giải thích *vì sao* tăng tính gắn kết/giảm phụ thuộc, và đưa cách tách nhỏ, an toàn. Refactor lớn → kế hoạch từng bước, không viết lại toàn bộ.
- Không đề xuất abstraction "cho tương lai" — chỉ khi đã có trường hợp dùng thứ hai.

**Code nên xoá** — `references/removal-plan.md`
- Code không dùng, trùng lặp, feature flag đã tắt.
- Phân biệt **xoá được ngay** và **hoãn, cần kế hoạch**.

**Bảo mật & độ tin cậy** — `references/security-checklist.md`
- XSS, injection (SQL/NoSQL/command), SSRF, path traversal.
- Thiếu AuthN/AuthZ, thiếu kiểm tra tenant/quyền sở hữu.
- Lộ secret/API key trong log/env/file.
- Rate limit, vòng lặp không giới hạn, điểm nóng CPU/bộ nhớ.
- Deserialize không an toàn, mã hoá yếu, mặc định không an toàn.
- **Race condition**: truy cập đồng thời, check-then-act, TOCTOU, thiếu lock.
- Nêu rõ cả **khả năng khai thác** và **mức tác động**.

**Chất lượng code** — `references/code-quality-checklist.md`
- Xử lý lỗi: nuốt exception, catch quá rộng, thiếu xử lý, lỗi async.
- Hiệu năng: N+1 query, tính toán nặng trên hot path, thiếu cache, bộ nhớ không giới hạn.
- Điều kiện biên: null/undefined, collection rỗng, biên số, off-by-one.
- Ưu tiên lỗi có thể gây **hỏng âm thầm** hoặc sự cố production.

### 4) Kiểm chứng từng finding

Trước khi báo cáo, mỗi finding phải qua bước này — một báo cáo đầy false positive khiến người dùng bỏ qua cả lỗi thật.

- Đọc code liên quan quanh dòng đó (caller, validation phía trên, middleware, kiểu dữ liệu) để chắc lỗi không đã được xử lý ở chỗ khác.
- Viết được **kịch bản lỗi cụ thể**: input/trạng thái nào → kết quả sai/crash/lộ dữ liệu gì. Không viết được → hạ mức hoặc bỏ.
- Gắn nhãn:
  - **Đã xác nhận** — đã lần theo code, kịch bản chắc chắn xảy ra.
  - **Có thể** — hợp lý nhưng phụ thuộc điều kiện chưa kiểm được (config runtime, dữ liệu thật).
- Finding chỉ là style/ý thích cá nhân → tối đa P3.

### 5) Định dạng báo cáo

```markdown
## Tóm tắt review

**Phạm vi**: <diff hiện tại / PR #n / commit / branch> — X file, Y dòng thay đổi
**Đánh giá chung**: APPROVE / REQUEST_CHANGES / COMMENT

---

## Phát hiện

### P0 - Nghiêm trọng
(không có, hoặc danh sách)

### P1 - Cao
1. **[file:line]** Tiêu đề ngắn — *Đã xác nhận*
   - Vấn đề: ...
   - Kịch bản lỗi: ...
   - Đề xuất sửa: ...

### P2 - Trung bình
2. (đánh số liên tục qua các mục)

### P3 - Thấp
...

---

## Kế hoạch xoá / lặp lại
(nếu có)

## Gợi ý thêm
(không chặn merge)
```

Thay đổi trải trên nhiều tính năng → nhóm finding theo tính năng, không chỉ theo thứ tự file.

**Review sạch** (không có lỗi): nói rõ đã kiểm tra gì, phần nào **chưa** kiểm tra (vd: "chưa kiểm tra migration DB"), rủi ro còn lại và test nên bổ sung.

### 6) Hỏi bước tiếp theo

```markdown
---

## Bước tiếp theo

Tìm thấy X vấn đề (P0: _, P1: _, P2: _, P3: _).

1. **Sửa tất cả**
2. **Chỉ sửa P0/P1**
3. **Sửa mục cụ thể** — cho biết số thứ tự
4. **Không sửa** — kết thúc review
```

Dùng `AskUserQuestion` nếu có. **Không sửa gì cho tới khi người dùng xác nhận.**

### 7) Khi người dùng chọn sửa

Theo quy trình `dev-workflow` (nếu đã cài):
- Giao subagent `general-purpose`, `model: "sonnet"`, `effort: "xhigh"`, dặn gọi skill `ponytail:ponytail` (sửa tối thiểu mà đầy đủ, sửa ở gốc, cập nhật caller/test bị ảnh hưởng). Brief gồm danh sách finding được chọn kèm `file:line`, kịch bản lỗi, đề xuất sửa.
- Sau khi sửa: **review lại đúng phần vừa sửa** (chạy lại bước 3–4 trên diff mới), chạy test liên quan, rồi báo kết quả.

## Tài liệu tham khảo

| File | Nội dung |
|---|---|
| `references/solid-checklist.md` | Dấu hiệu vi phạm SOLID, code smell, nguyên tắc refactor |
| `references/security-checklist.md` | Bảo mật web/app, race condition, toàn vẹn dữ liệu |
| `references/code-quality-checklist.md` | Xử lý lỗi, hiệu năng, điều kiện biên |
| `references/removal-plan.md` | Mẫu kế hoạch xoá code |

---

Chuyển thể từ [sanyuan0704/sanyuan-skills — code-review-expert](https://github.com/sanyuan0704/sanyuan-skills/tree/main/skills/code-review-expert) (MIT, © 2025 sanyuan0704).
