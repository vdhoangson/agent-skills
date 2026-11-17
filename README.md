# agent-skills

Bộ sưu tập [Agent Skills](https://docs.claude.com/en/docs/agents-and-tools/agent-skills/overview) cá nhân dùng cho **Claude Code** và **claude.ai**.

| Skill | Mô tả |
|---|---|
| [`dev-workflow`](skills/dev-workflow/SKILL.md) | Quy trình chuẩn cho mọi task code: đánh giá độ lớn yêu cầu, deep research, phân công model (Opus / Haiku / Sonnet), định tuyến sang plugin `ponytail` và `ui-ux-pro-max`, giới hạn tài nguyên khi chạy nhiều subagent. |
| [`code-review-expert`](skills/code-review-expert/SKILL.md) | Review code góc nhìn senior engineer: bảo mật, race condition, xử lý lỗi, hiệu năng, điều kiện biên, SOLID, code nên xoá. Phân mức P0–P3, kiểm chứng từng lỗi, review song song bằng subagent khi diff lớn. Nhận diff / PR / commit / branch. |

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
4. Review & báo cáo ........... session chính + test
        ├─ Vừa → ponytail-review
        └─ Lớn / auth / thanh toán / dữ liệu → code-review-expert
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
| "Thêm OAuth Google + phân quyền theo role" | Lớn | Deep research → phân tích phương án → hỏi điểm mơ hồ → Sonnet → `code-review-expert` |

### Tuỳ biến

Sửa trực tiếp [`skills/dev-workflow/SKILL.md`](skills/dev-workflow/SKILL.md):

- Đổi model/effort ở bước 1–3.
- Đổi ngưỡng RAM/CPU ở mục 5.
- Đổi tiêu chí Nhỏ/Vừa/Lớn ở mục 0.
- Phần `description` trong frontmatter quyết định khi nào skill tự kích hoạt.

Nếu bạn cũng đang ghi quy trình tương tự trong `~/.claude/CLAUDE.md`, nên giữ một nơi duy nhất để tránh hai bản lệch nhau.

---

## code-review-expert

Chuyển thể (Việt hoá + tuỳ biến) từ [sanyuan0704/sanyuan-skills — code-review-expert](https://github.com/sanyuan0704/sanyuan-skills/tree/main/skills/code-review-expert), giấy phép MIT.

### Khác gì bản gốc

| | Bản gốc | Bản này |
|---|---|---|
| Ngôn ngữ | Tiếng Anh | Tiếng Việt (cả checklist) |
| Phạm vi review | Diff hiện tại | Diff hiện tại, PR (`gh`), commit, commit range, branch |
| Diff lớn | Review theo lô | Chia 3 subagent Sonnet song song (bảo mật / chất lượng / SOLID + xoá code), có kiểm tra RAM/CPU, gộp và loại trùng |
| Độ tin cậy | — | Mỗi finding phải có **kịch bản lỗi cụ thể** và nhãn *Đã xác nhận* / *Có thể*; không chứng minh được thì hạ mức hoặc bỏ |
| Khi sửa | Session tự sửa | Theo `dev-workflow`: Sonnet + `ponytail`, rồi review lại phần vừa sửa |

### Quy trình

```
1. Xác định phạm vi ── diff / PR / commit / range / branch
        │
2. Chọn chế độ ──► diff nhỏ: tự review
        │      └─► diff lớn: kiểm tra RAM/CPU → 3 subagent song song
        │                     ├─ Bảo mật & độ tin cậy
        │                     ├─ Chất lượng code
        │                     └─ SOLID + code nên xoá
3. Review theo checklist (references/)
        │
4. Kiểm chứng từng finding → loại false positive
        │
5. Báo cáo P0–P3
        │
6. Hỏi bạn: sửa tất cả / chỉ P0-P1 / mục cụ thể / không sửa
        │
7. Sửa (Sonnet + ponytail) → review lại phần vừa sửa
```

### Mức độ

| Mức | Ý nghĩa | Hành động |
|---|---|---|
| **P0** | Lỗ hổng bảo mật, mất dữ liệu, bug sai kết quả | Chặn merge |
| **P1** | Lỗi logic, race condition, hồi quy hiệu năng | Sửa trước merge |
| **P2** | Code smell, khó bảo trì, thiếu xử lý biên | Sửa luôn hoặc follow-up |
| **P3** | Style, đặt tên | Tuỳ chọn |

### Sử dụng

```
/code-review-expert
/code-review-expert PR 42
/code-review-expert main...feature/login
review giúp commit abc1234
```

Skill **không sửa gì** cho tới khi bạn chọn cách sửa.

> **Phân vai với các skill review khác:** `code-review-expert` cho review kỹ (PR/branch trước merge, thay đổi lớn, đụng auth/thanh toán/dữ liệu); `ponytail-review` cho kiểm tra nhanh thay đổi nhỏ; `ponytail-audit` cho rà cả repo. `dev-workflow` tự chọn theo mức task. Câu chung chung như "review code" vẫn có thể kích hoạt skill khác — gõ `/code-review-expert` khi muốn chắc chắn.

### Cài đặt

Giống `dev-workflow`, thay tên thư mục:

```bash
ln -s ~/agent-skills/skills/code-review-expert ~/.claude/skills/code-review-expert
```

---

## Cấu trúc repo

```
agent-skills/
├── README.md
├── LICENSE
└── skills/
    ├── dev-workflow/
    │   └── SKILL.md
    └── code-review-expert/
        ├── SKILL.md
        ├── LICENSE              # MIT của bản gốc
        └── references/
            ├── security-checklist.md
            ├── code-quality-checklist.md
            ├── solid-checklist.md
            └── removal-plan.md
```

Thêm skill mới: tạo thư mục `skills/<tên-skill>/SKILL.md` và thêm một dòng vào bảng ở đầu README.

## Đóng gói file `.skill`

```bash
cd skills && zip -r ../<tên-skill>.skill <tên-skill>
```

## License

[MIT](LICENSE). `code-review-expert` chuyển thể từ công trình của [sanyuan0704](https://github.com/sanyuan0704/sanyuan-skills) (MIT, xem [LICENSE](skills/code-review-expert/LICENSE)).
