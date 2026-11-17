---
name: dev-workflow
description: Quy trình làm việc cá nhân cho MỌI task code — viết, sửa bug, refactor, review code, làm giao diện/UI. Phân công model (Opus phân tích, Haiku tìm kiếm, Sonnet viết code), định tuyến sang plugin ponytail (code tối giản + review) và ui-ux-pro-max (UI/UX), tra docs thư viện qua Context7, kiểm tra RAM/CPU trước khi tạo nhiều subagent. Dùng skill này bất cứ khi nào người dùng yêu cầu viết/sửa/review code hoặc làm UI, kể cả khi họ không nhắc tên skill.
---

# Dev Workflow

Quy trình chuẩn cho mọi task code. Mục tiêu: model đắt chỉ dùng cho việc cần suy nghĩ, model rẻ cho việc đọc/tìm, code luôn tối giản, UI luôn có hướng dẫn thiết kế.

Trả lời người dùng **ngắn gọn, đúng trọng tâm, bằng tiếng Việt**.

## 0. Đánh giá độ lớn yêu cầu

Ngay khi nhận yêu cầu, xếp nó vào một mức và nói ngắn cho người dùng biết mức đã chọn. Mức quyết định độ sâu nghiên cứu — nghiên cứu ít quá thì code sai hướng, nhiều quá thì phí token.

| Mức | Dấu hiệu | Cách làm |
|---|---|---|
| **Nhỏ** | Vài tool call, đã biết chính xác file/symbol (sửa 1 dòng, đổi tên, đọc 1 file) | Làm thẳng, bỏ qua deep research và subagent. |
| **Vừa** | 1 tính năng/bug, vài file, phạm vi rõ | Research nhẹ: Explore + Context7, rồi theo bước 1–4. |
| **Lớn** | Nhiều module, kiến trúc mới, tích hợp bên thứ ba, yêu cầu mơ hồ, hoặc có rủi ro dữ liệu/bảo mật | **Deep research** (mục 0.1) trước khi phân tích, rồi theo bước 1–4. |

Khi phân vân giữa hai mức, chọn mức lớn hơn.

### 0.1 Deep research (task Lớn)

Mục tiêu: hiểu đủ để phân tích chi tiết trước khi viết dòng code nào. Chạy các nhánh độc lập song song (kiểm tra tài nguyên ở mục 5 trước):

1. **Codebase** — subagent `Explore` (`model: "haiku"`, `effort: "high"`, độ rộng "very thorough"): kiến trúc hiện tại, module bị ảnh hưởng, pattern/helper có sẵn, caller, test, config.
2. **Docs thư viện** — Context7 cho từng thư viện/framework liên quan, mỗi khái niệm một query.
3. **Web** — `WebSearch` mode `extended` cho best practice, issue đã biết, so sánh giải pháp, thay đổi gần đây (dùng `standard` trước nếu câu hỏi đơn giản).

Tổng hợp thành bản phân tích ngắn gồm: phạm vi, các phương án (kèm đánh đổi) và phương án đề xuất, rủi ro, điểm còn mơ hồ. Điểm mơ hồ → hỏi người dùng bằng `AskUserQuestion` ở bước 1. Bản phân tích này là đầu vào cho brief bước 3.

## 1. Phân tích & hỏi lại — Opus 5.5, effort `xhigh`

- Session chính tự phân tích yêu cầu (dựa trên kết quả deep research nếu là task Lớn). Nếu session không chạy Opus 5.5 → giao cho subagent `Plan` với `model: "opus"`, `effort: "xhigh"`.
- Nếu chưa rõ phạm vi, hành vi mong muốn hoặc ràng buộc → hỏi bằng `AskUserQuestion` trước khi làm. Không đoán những gì chỉ người dùng mới quyết được.
- Xác định **loại task** để định tuyến ở bước 3:
  - **UI**: giao diện, component, layout, màu sắc, responsive, design system, landing page, dashboard.
  - **Code thường**: logic, API, bug fix, refactor, script.
  - **Review**: review diff/PR/commit.

## 2. Tìm kiếm & đọc file — Haiku 5.5, effort `high`

- Giao cho subagent `Explore` với `model: "haiku"`, `effort: "high"`. Yêu cầu nó trả về: file liên quan, symbol, caller, test, config cần đụng tới.
- Chỉ tự tìm trực tiếp khi đã biết chính xác file/symbol.
- **Context7**: nếu task dùng thư viện/framework/SDK/API/CLI/cloud service, tra docs hiện hành qua Context7 (`resolve-library-id` → `query-docs`, mỗi khái niệm một query) và đưa kết quả vào brief cho bước 3. Dùng cả với thư viện quen thuộc — kiến thức huấn luyện có thể đã cũ. Không dùng cho logic nghiệp vụ hay khái niệm lập trình chung.

## 3. Viết code — Sonnet 5.5, effort `xhigh`

Giao cho subagent `general-purpose` với `model: "sonnet"`, `effort: "xhigh"`. Brief phải tự đủ (subagent không thấy hội thoại), gồm:
- Kết quả phân tích bước 1 và danh sách file/caller/test bước 2.
- Docs Context7 liên quan (nếu có).
- Tiêu chí hoàn thành cụ thể (test nào phải pass, hành vi nào phải đúng).
- Chỉ dẫn plugin theo loại task:

| Loại task | Dặn subagent |
|---|---|
| Code thường | Gọi skill `ponytail:ponytail` trước khi code: thay đổi nhỏ nhất mà đầy đủ, không abstraction/option thừa, logic mới có test nhỏ. |
| UI | Gọi `ui-ux-pro-max:ui-ux-pro-max` (hoặc `ui-ux-pro-max:ui-styling` cho shadcn/Tailwind, `ui-ux-pro-max:design-system` cho token) để lấy hướng dẫn thiết kế, **và** `ponytail:ponytail` để giữ code gọn. Thiết kế đẹp không có nghĩa là thêm code thừa. |

Nếu task tách được thành nhiều phần độc lập (không đụng cùng file) → có thể chạy song song nhiều subagent, nhưng xem mục 5 trước.

## 4. Review & báo cáo — session chính

- Đọc diff subagent trả về, kiểm tra đúng tiêu chí hoàn thành. Kết quả của subagent là báo cáo, không phải bằng chứng.
- Với thay đổi không tầm thường: chạy `ponytail:ponytail-review` (hoặc `ponytail:ponytail-audit` nếu người dùng muốn rà cả repo).
- Chạy test/lint/build cần thiết.
- Báo người dùng ngắn gọn: đã đổi gì, đã kiểm tra gì, còn gì chưa kiểm tra hoặc rủi ro.

## 5. Giới hạn tài nguyên khi chạy nhiều subagent

Trước khi tạo nhiều subagent (song song hoặc workflow), kiểm tra máy:

```bash
free -m; nproc; uptime
```

- RAM dùng không vượt **90%**, CPU (load / số nhân) không vượt **80%**.
- Gần hoặc vượt ngưỡng → giảm số subagent chạy cùng lúc, chạy tuần tự, hoặc chờ tài nguyên giảm.
- Task chạy lâu → kiểm tra lại giữa các đợt tạo subagent.

## Ví dụ định tuyến

**Ví dụ 1:** "Thêm nút export CSV vào trang báo cáo"
→ UI + code. Explore (Haiku) tìm trang báo cáo + helper CSV có sẵn → Sonnet code với `ui-ux-pro-max` + `ponytail` → session chính review + chạy test.

**Ví dụ 2:** "Sửa lỗi Prisma query trả về null khi có relation"
→ Code thường + thư viện. Context7 tra docs Prisma relation → Explore tìm mọi caller → Sonnet sửa ở gốc với `ponytail` → `ponytail-review`.

**Ví dụ 3:** "Đổi tên biến `usr` thành `user` trong utils.ts"
→ Mức Nhỏ, biết chính xác file → làm thẳng, không subagent.

**Ví dụ 4:** "Thêm đăng nhập OAuth Google + phân quyền theo role cho toàn app"
→ Mức Lớn (nhiều module, bảo mật, bên thứ ba). Deep research: Explore toàn bộ auth/middleware/route + Context7 docs thư viện auth + WebSearch extended về best practice OAuth/RBAC → bản phân tích các phương án → hỏi người dùng điểm mơ hồ → Sonnet code từng phần → `ponytail-review`.
