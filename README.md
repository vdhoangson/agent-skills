# agent-skills

Bộ sưu tập [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) cá nhân dùng cho **Claude Code** và **claude.ai**.

| Skill | Mô tả |
|---|---|
| [`dev-workflow`](skills/dev-workflow/SKILL.md) | Quy trình chuẩn cho mọi task code: đánh giá độ lớn yêu cầu, deep research, phân công model (Opus / Haiku / Sonnet), định tuyến sang plugin `ponytail` và `ui-ux-pro-max`, giới hạn tài nguyên khi chạy nhiều subagent. |

---

## dev-workflow

### Vấn đề nó giải quyết

- Dùng một model đắt cho mọi việc (kể cả đọc file, tìm kiếm) → tốn token.
- Task lớn bắt tay code ngay, thiếu nghiên cứu → sai hướng, phải làm lại.
- Task nhỏ lại bị "làm quá" với nhiều subagent → chậm và phí.
- Code sinh ra thừa abstraction; UI thiếu nguyên tắc thiết kế.
- Chạy nhiều subagent song song làm máy treo.

### Cách hoạt động

```
Yêu cầu
  │
  ▼
0. Đánh giá độ lớn ──► Nhỏ ──► làm thẳng, không subagent
  │
  ├─► Vừa ──► research nhẹ (Explore + Context7)
  │
  └─► Lớn ──► DEEP RESEARCH song song
                ├─ Explore (Haiku): rà toàn bộ codebase liên quan
                ├─ Context7: docs thư viện hiện hành
                └─ WebSearch extended: best practice, issue đã biết
                    → bản phân tích: phạm vi, phương án, rủi ro, điểm mơ hồ
  │
  ▼
1. Phân tích & hỏi lại ........ Opus 5.5 · xhigh
  │
  ▼
2. Tìm kiếm & đọc file ........ Haiku 5.5 · high  (subagent Explore)
  │
  ▼
3. Viết code .................. Sonnet 5.5 · xhigh (subagent general-purpose)
  │     ├─ Code thường → ponytail
  │     └─ UI          → ui-ux-pro-max + ponytail
  ▼
4. Review & báo cáo ........... session chính + ponytail-review + test
```

**Mục 5 — Giới hạn tài nguyên:** trước khi tạo nhiều subagent, skill kiểm tra `free -m; nproc; uptime`. RAM không vượt **90%**, CPU không vượt **80%**; gần ngưỡng thì giảm số subagent song song hoặc chạy tuần tự.

### Đánh giá độ lớn

| Mức | Dấu hiệu | Cách làm |
|---|---|---|
| **Nhỏ** | Vài tool call, biết chính xác file/symbol | Làm thẳng |
| **Vừa** | 1 tính năng/bug, vài file, phạm vi rõ | Explore + Context7 → bước 1–4 |
| **Lớn** | Nhiều module, kiến trúc mới, bên thứ ba, yêu cầu mơ hồ, rủi ro dữ liệu/bảo mật | Deep research → bước 1–4 |

Phân vân giữa hai mức thì chọn mức lớn hơn.

### Yêu cầu

| Thành phần | Bắt buộc? | Ghi chú |
|---|---|---|
| [Claude Code](https://docs.claude.com/en/docs/claude-code/overview) hoặc claude.ai | Có | Phân công model + subagent đầy đủ nhất trên Claude Code |
| Plugin [`ponytail`](https://github.com/DietrichGebert/ponytail) | Nên có | Dùng cho bước viết code và review |
| Plugin [`ui-ux-pro-max`](https://github.com/nextlevelbuilder/ui-ux-pro-max-skill) | Nên có | Dùng cho task UI |
| MCP [Context7](https://github.com/upstash/context7) | Nên có | Tra docs thư viện hiện hành |
| Linux/macOS (`free`, `nproc`, `uptime`) | Cho mục 5 | Trên macOS dùng `vm_stat` / `sysctl hw.ncpu` thay `free` / `nproc` |

Thiếu plugin nào thì skill vẫn chạy, chỉ bỏ qua bước định tuyến tương ứng.

Cài hai plugin trong Claude Code:

```
/plugin marketplace add https://github.com/DietrichGebert/ponytail.git
/plugin install ponytail@ponytail
/plugin marketplace add https://github.com/nextlevelbuilder/ui-ux-pro-max-skill.git
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

### Cài đặt

**Claude Code — cho mọi project (khuyên dùng):**

```bash
git clone https://github.com/vdhoangson/agent-skills.git ~/agent-skills
```

```bash
ln -s ~/agent-skills/skills/dev-workflow ~/.claude/skills/dev-workflow
```

Dùng symlink để `git pull` là cập nhật luôn. Muốn chỉ áp dụng cho một project thì symlink/copy vào `<project>/.claude/skills/` thay vì `~/.claude/skills/`.

**claude.ai:** tải `dev-workflow.skill` ở trang [Releases](https://github.com/vdhoangson/agent-skills/releases), rồi vào *Settings → Capabilities → Skills* để tải lên.

### Sử dụng

Skill tự kích hoạt khi bạn yêu cầu viết, sửa, refactor, review code hoặc làm UI. Gọi thủ công:

```
/dev-workflow thêm đăng nhập OAuth Google cho app
```

Ví dụ định tuyến:

| Yêu cầu | Mức | Luồng xử lý |
|---|---|---|
| "Đổi tên biến `usr` thành `user` trong utils.ts" | Nhỏ | Làm thẳng |
| "Thêm nút export CSV vào trang báo cáo" | Vừa | Explore → Sonnet + `ui-ux-pro-max` + `ponytail` → review |
| "Sửa lỗi Prisma query trả null khi có relation" | Vừa | Context7 (Prisma) → Explore caller → Sonnet + `ponytail` → `ponytail-review` |
| "Thêm OAuth Google + phân quyền theo role" | Lớn | Deep research → phân tích phương án → hỏi điểm mơ hồ → Sonnet → `ponytail-review` |

### Tuỳ biến

Sửa trực tiếp [`skills/dev-workflow/SKILL.md`](skills/dev-workflow/SKILL.md):

- Đổi model/effort ở bước 1–3.
- Đổi ngưỡng RAM/CPU ở mục 5.
- Đổi tiêu chí Nhỏ/Vừa/Lớn ở mục 0.
- Phần `description` trong frontmatter quyết định khi nào skill tự kích hoạt.

Nếu bạn cũng đang ghi quy trình tương tự trong `~/.claude/CLAUDE.md`, nên giữ một nơi duy nhất để tránh hai bản lệch nhau.

---

## Cấu trúc repo

```
agent-skills/
├── README.md
├── LICENSE
└── skills/
    └── dev-workflow/
        └── SKILL.md
```

Thêm skill mới: tạo thư mục `skills/<tên-skill>/SKILL.md` và thêm một dòng vào bảng ở đầu README.

## Đóng gói file `.skill`

```bash
cd skills && zip -r ../dev-workflow.skill dev-workflow
```

## License

[MIT](LICENSE)
