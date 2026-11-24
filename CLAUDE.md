<!-- gitnexus:start -->
# GitNexus — Code Intelligence

This project is indexed by GitNexus as **Dormitory-Manager** (1076 symbols, 1652 relationships, 53 execution flows).

> Index stale? Run `node .gitnexus/run.cjs analyze --index-only` from the project root — it auto-selects an available runner. No `.gitnexus/run.cjs` yet? Bootstrap with `npx`, `bunx`, or `pnpm dlx` — e.g. `bunx gitnexus@latest analyze` (npm 11 npx crash; #1939).

## Always Do

- **MUST run impact before editing.** Use `impact({target: "symbolName", direction: "upstream"})` or `node .gitnexus/run.cjs impact "symbolName" --direction upstream --repo .`; report callers, processes, and risk. Never substitute grep for graph analysis.
- **MUST analyze graph changes before committing.** Use `detect_changes({scope: "all"})` (MCP) or `node .gitnexus/run.cjs detect-changes --scope all --repo .` (CLI fallback). `partial: true` or `truncated: true` is not a clean check — a zero means unseen, not unaffected; re-run it. For regression review: `detect_changes({scope: "compare", base_ref: "master"})` or `node .gitnexus/run.cjs detect-changes --scope compare --base-ref "master" --repo .`.
- MUST warn on HIGH/CRITICAL `risk` pre-edit; never use `riskSharedAxes` to waive a HIGH/CRITICAL `risk` warning. Compare File/symbol: MCP File omits axes; Graph-RAG expands File.
- **MUST treat `risk: UNKNOWN` as unresolved, not as low.** An empty caller set is not evidence the symbol is unused — it can also mean the callers are not resolvable by the index (plain-object property access, dynamic dispatch, cross-language calls). `impact` pairs `UNKNOWN` with a `riskNote` saying so. Confirm with a text search before treating the symbol as safe to change or delete; do not proceed on the strength of a zero.
- **MUST use `query({search_query: "concept"})` for concepts/flows, `context({name: "symbolName"})` for a named symbol, or `impact` for blast radius, on read-only callers, dependencies, imports, or execution flow.** Graph first; text search only for empty/`UNKNOWN`/literals.
- For security review, `explain({target: "fileOrSymbol"})` lists taint findings (source→sink flows; needs `analyze --pdg`).

## Never Do

- NEVER edit a function, class, or method before MCP/CLI impact analysis.
- NEVER ignore HIGH or CRITICAL risk warnings from impact analysis, and never read `UNKNOWN` as an all-clear — it means the walk could not answer, which is the one verdict that requires confirming by other means.
- NEVER rename symbols with find-and-replace — use `rename` which understands the call graph.
- NEVER commit before MCP/CLI graph change analysis.

## Resources

| Resource | Use for |
| --- | --- |
| `gitnexus://repo/Dormitory-Manager/context` | Codebase overview, check index freshness |
| `gitnexus://repo/Dormitory-Manager/clusters` | All functional areas |
| `gitnexus://repo/Dormitory-Manager/processes` | All execution flows |
| `gitnexus://repo/Dormitory-Manager/process/{name}` | Step-by-step execution trace |

## CLI

| Task | Read this skill file |
| --- | --- |
| Understand architecture / "How does X work?" | `.claude/skills/gitnexus-exploring/SKILL.md` |
| Blast radius / "What breaks if I change X?" | `.claude/skills/gitnexus-impact-analysis/SKILL.md` |
| Trace bugs / "Why is X failing?" | `.claude/skills/gitnexus-debugging/SKILL.md` |
| Rename / extract / split / refactor | `.claude/skills/gitnexus-refactoring/SKILL.md` |
| Tools, resources, schema reference | `.claude/skills/gitnexus-guide/SKILL.md` |
| Index, status, clean, wiki CLI commands | `.claude/skills/gitnexus-cli/SKILL.md` |

<!-- gitnexus:end -->

---

# Dormitory Manager — Hướng Dẫn Kỹ Thuật Cho Các Tác Tử AI (AI Agent Guidelines)

Dự án này được phát triển phối hợp bởi nhiều mô hình AI (**Antigravity**, **Claude Code**, **OpenAI Codex**).  
Mọi tác tử AI khi tham gia đọc, viết mã, kiểm thử hoặc review BẮT BUỘC tuân thủ các nguyên tắc dưới đây.

## 1. Tổng Quan Dự Án & Công Nghệ (Tech Stack)

- **Ngôn ngữ & Runtime**: C# 12 / .NET 8.0 LTS.
- **Mô hình kiến trúc**: Clean Architecture 4 tầng + MVVM (`CommunityToolkit.Mvvm`).
  - `src/Dormitory.Core`: Domain Entities, Enums, Value Objects (hoàn toàn POCO, 0 dependencies).
  - `src/Dormitory.Application`: Interfaces (`IService`), DTOs, Request/Response models, constants.
  - `src/Dormitory.Infrastructure`: Data Access (`DormitoryDbContext` EF Core SQLite), External Engines (ClosedXML, QuestPDF, MailKit, BCrypt, HttpListener Webhook, VietQR).
  - `src/Dormitory.Desktop`: Avalonia UI 11.0, Views, ViewModels, Dialogs, Converters, Assets.
  - `tests/Dormitory.UnitTests`: Bộ kiểm thử đơn vị xUnit + FluentAssertions + Moq.
  - `tests/Dormitory.E2ETests`: Bộ kiểm thử tự động Avalonia Headless E2E.
- **Trạng thái hiện tại**: Đã hoàn thành 100% Phase 1 – 10 (Phiên bản phát hành **`v2.6.0`**, **498/498 tests PASS 100%**).
- **Mục tiêu kế tiếp**: **Phase 11: Nhập Chỉ Số Điện Nước Theo Lô & Biểu Giá Bậc Thang** (Xem [docs/phase11/spec-metering-tariffs.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/phase11/spec-metering-tariffs.md)).

---

## 2. Các Quy Tắc Bất Biến (Immutable Invariants)

1. **Quy ước đặt tên & Ngôn ngữ (Strict Language Policy)**:
   - **Mã nguồn C#**: 100% tiếng Anh cho Class, Method, Variable, Property, File, Namespace, Enum.
   - **TỪ KHÓA BẮT BUỘC**: Luôn sử dụng `Dormitory`, **TUYỆT ĐỐI KHÔNG DÙNG `KTX`** trong code identifiers và namespaces.
   - **Giao diện & Giao tiếp người dùng**: 100% tiếng Việt cho Text UI, Thông báo lỗi, Hộp thoại, Comments code, XML documentation, README và tài liệu trong `docs/`.
2. **Nguyên tắc phát triển qua Subagent (Subagent-Driven Development)**:
   - Tách biệt rõ ràng giữa vai trò **Implementer** (viết code và unit tests) và **Reviewer** (kiểm tra spec, security, quality gates).
   - Không xác nhận hoàn thành khi chưa chạy kiểm thử xác thực.
3. **Tiêu chuẩn chất lượng (Quality Gate)**:
   - Mọi thay đổi mã nguồn trước khi commit phải đảm bảo lệnh `dotnet test` trả về **100% Passed** (không được làm gãy 498 tests hiện có).
   - Phải chạy `node .gitnexus/run.cjs detect-changes --scope all --repo .` trước khi commit.

---

## 3. Phân Công Vai Trò Giữa Các AI (Multi-Agent Division of Labor)

Dự án áp dụng mô hình phối hợp nhằm **tối ưu hóa token** và phát huy điểm mạnh của từng AI (Chi tiết tại [docs/agent-collaboration-guide.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/agent-collaboration-guide.md)):

```
┌─────────────────────────────────────────────────────────────┐
│                 Antigravity (Orchestrator)                 │
│  - Điều phối Roadmap, quản lý Phase & Releases              │
│  - Viết tài liệu đặc tả, phân rã task cho Claude/Codex       │
│  - Tích hợp liên phân hệ, CI/CD đa nền tảng (macOS/Win/Linux)│
└──────────────┬──────────────────────────────┬───────────────┘
               │                              │
               ▼                              ▼
┌──────────────────────────────┐┌──────────────────────────────┐
│  Claude Code (Architect/UI)  ││    OpenAI Codex (Worker)     │
│ - Dựng Avalonia AXAML & UX   ││ - Sinh mã lặp lại, boilerplate│
│ - Quản lý ViewModel MVVM     ││ - Sinh Entities, DTOs, Enums │
│ - Tái cấu trúc phức tạp      ││ - Sinh hàng loạt Unit Tests   │
│ - Độc lập Code Review PR     ││ - Viết EF Core Migrations    │
└──────────────────────────────┘└──────────────────────────────┘
```

- **OpenAI Codex**: Chuyên thực hiện các tác vụ lặp lại, có quy chuẩn rõ ràng để **tiết kiệm token cho Claude Code**. Nhận việc qua mẫu Handoff Ticket kèm ràng buộc an toàn cụ thể.
- **Claude Code**: Chuyên sâu về tầng Presentation Avalonia UI, ViewModel bindings, kiến trúc tổng thể và đảm nhận vai trò **Reviewer** khắt khe trước khi tích hợp.
- **Antigravity**: Giữ vai trò nhạc trưởng điều phối toàn cục, nghiệm thu cuối cùng và phát hành.

---

## 4. Danh Mục Lệnh Thường Dùng (Commands Cheat Sheet)

```bash
# Biên dịch dự án
dotnet build

# Chạy toàn bộ 498+ bài kiểm thử
dotnet test

# Chạy ứng dụng giao diện Desktop
dotnet run --project src/Dormitory.Desktop

# GitNexus: Phân tích ảnh hưởng trước khi sửa hàm
node .gitnexus/run.cjs impact "SymbolName" --direction upstream --repo .

# GitNexus: Kiểm tra thay đổi đồ thị trước khi commit
node .gitnexus/run.cjs detect-changes --scope all --repo .

# Đóng gói ứng dụng (macOS / Windows / Linux)
bash scripts/package/build-macos.sh
bash scripts/package/build-windows.sh
bash scripts/package/build-linux.sh
```

---

## 5. Tài Liệu Kỹ Thuật Tham Chiếu (Documentation Index)

- 📐 **Kiến trúc hệ thống**: [docs/architecture.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/architecture.md)
- 🤝 **Hướng dẫn phối hợp đa AI**: [docs/agent-collaboration-guide.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/agent-collaboration-guide.md)
- 📖 **Hướng dẫn sử dụng & vận hành**: [docs/user-guide.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/user-guide.md)
- 📦 **Đóng gói & Phát hành CI/CD**: [docs/packaging-and-deployment.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/packaging-and-deployment.md)
- 📋 **Đặc tả Phase 10 (Đã hoàn thành - v2.6.0)**: [docs/phase10/visitor-and-attendance-spec.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/phase10/visitor-and-attendance-spec.md)
- ⚡ **Đặc tả Phase 11 (Kế tiếp - v2.7.0)**: [docs/phase11/spec-metering-tariffs.md](file:///Volumes/AI_Models/source/Dormitory-Manager/docs/phase11/spec-metering-tariffs.md)
