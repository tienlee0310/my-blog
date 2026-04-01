---
title: Source code Claude Code bị leak qua sourcemap trên npm - cùng mổ xẻ
draft: false
tags:
  - AI
  - ChatGPT
  - Social-Media
  - Artificial-Intelligence
  - GenerativeAI
  - Internet
  - OpenAI
  - Claude
  - "#Anthropic"
created: 2026-03-31
modified: 2026-03-31
---
Hôm nay (31/03/2026) — Chaofan Shou trên X phát hiện một thứ mà Anthropic chắc chắn không muốn cả thế giới nhìn thấy: **toàn bộ source code** của Claude Code (CLI coding chính thức của Anthropic) đang “lộ thiên” trên npm registry thông qua một file **sourcemap** được bundle vào package đã publish.

[![The tweet announcing the leak](https://raw.githubusercontent.com/kuberwastaken/claude-code/main/public/leak-tweet.png)](https://raw.githubusercontent.com/kuberwastaken/claude-code/main/public/leak-tweet.png)

[Mình đã giữ một bản backup code đó trên GitHub ở đây](https://github.com/Kuberwastaken/claude-code) nhưng đó chưa phải phần vui nhất.

Hãy đào sâu: trong đó có gì, leak xảy ra thế nào, và quan trọng nhất là những thứ giờ ta biết — vốn chưa từng được định để public.

## Vì sao chuyện này xảy ra được?

Đây là đoạn khiến mình phải kiểu “…thiệt luôn?”

Khi bạn publish package JavaScript/TypeScript lên npm, build toolchain thường sinh ra **source map files** (file `.map`). Chúng là cây cầu giữa code production đã minify/bundle và source gốc — để khi production crash, stack trace chỉ đúng vào *dòng code thật* trong *file gốc*, chứ không phải “line 1, column 48293” của một cục minified blob.

Nhưng “phần vui” là: **sourcemap chứa source code gốc**. Đúng nghĩa đen. Source thô, nhét thành string trong một file JSON.

Cấu trúc `.map` trông như thế này:

```json
{
  "version": 3,
  "sources": ["../src/main.tsx", "../src/tools/BashTool.ts", "..."],
  "sourcesContent": ["// The ENTIRE original source code of each file", "..."],
  "mappings": "AAAA,SAAS,OAAO..."
}
```

Cái mảng `sourcesContent` kia? Đó là tất cả.
Mọi file. Mọi comment. Mọi hằng số nội bộ. Mọi system prompt. Tất cả. Nằm trơ trọi trong JSON mà npm phục vụ cho bất kỳ ai chạy `npm pack` hoặc thậm chí chỉ cần xem nội dung package.

Đây không phải vector tấn công mới. Nó từng xảy ra và thật ra sẽ còn xảy ra.

Sai lầm thường giống nhau: ai đó quên thêm `*.map` vào `.npmignore` hoặc không cấu hình bundler để bỏ sourcemap ở build production. Với bundler của Bun (Claude Code dùng), sourcemap được tạo mặc định trừ khi bạn tắt rõ ràng.

[![Claude Code source files exposed in npm package](https://raw.githubusercontent.com/kuberwastaken/claude-code/main/public/claude-files.png)](https://raw.githubusercontent.com/kuberwastaken/claude-code/main/public/claude-files.png)

Điều hài nhất là trong code có hẳn một hệ thống tên ["Undercover Mode"](#undercover-mode--do-not-blow-your-cover) để ngăn lộ thông tin nội bộ của Anthropic.

Họ xây cả một subsystem để AI không vô tình lộ codename nội bộ trong git commit… rồi lại ship toàn bộ source trong một file `.map` — có khi cũng do Claude.

---
## Claude Code “bên trong” trông ra sao?

Nếu bạn đang sống dưới một tảng đá, Claude Code là CLI chính thức của Anthropic để code với Claude và cũng là AI coding agent phổ biến nhất.

Nhìn bên ngoài, nó như một CLI bóng bẩy nhưng có vẻ “đơn giản”.

Nhìn bên trong: một entry point **785KB [`main.tsx`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/cli/src/main.rs)**, một React terminal renderer tuỳ biến, 40+ tool, hệ orchestration đa-agent, một engine “gom trí nhớ” chạy nền tên "dream", và nhiều thứ khác.

Enough yapping — đây là vài thứ trong source mà mình thấy thật sự cool sau một buổi chiều đào:

---
## BUDDY - Tamagotchi trong terminal của bạn

Mình không bịa.

Claude Code có hẳn một **hệ pet kiểu Tamagotchi** tên "Buddy". Một **deterministic gacha system** với độ hiếm loài, shiny variant, chỉ số sinh theo thủ tục, và “mô tả linh hồn” do Claude viết lúc nở lần đầu kiểu OpenClaw.

Toàn bộ nằm trong [`buddy/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates) và bị khoá bởi compile-time feature flag `BUDDY`.

### Hệ gacha

Loài của buddy được quyết định bởi **Mulberry32 PRNG**, một bộ sinh số giả ngẫu nhiên 32-bit nhanh, seed từ hash `userId` (kèm salt `'friend-2026-401'`):

```typescript
// Mulberry32 PRNG - deterministic, reproducible per-user
function mulberry32(seed: number): () => number {
  return function() {
    seed |= 0; seed = seed + 0x6D2B79F5 | 0;
    var t = Math.imul(seed ^ seed >>> 15, 1 | seed);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }
}
```

Cùng user → luôn ra cùng buddy.

### 18 loài (được che trong code)

Tên loài bị giấu bằng mảng `String.fromCharCode()` — rõ ràng Anthropic không muốn nó bị search string ra. Decode ra, danh sách đầy đủ là:

| Rarity | Species |
|--------|---------|
| **Common** (60%) | Pebblecrab, Dustbunny, Mossfrog, Twigling, Dewdrop, Puddlefish |
| **Uncommon** (25%) | Cloudferret, Gustowl, Bramblebear, Thornfox |
| **Rare** (10%) | Crystaldrake, Deepstag, Lavapup |
| **Epic** (4%) | Stormwyrm, Voidcat, Aetherling |
| **Legendary** (1%) | Cosmoshale, Nebulynx |

Ngoài ra còn có **1% shiny chance** độc lập với rarity. Nên Shiny Legendary Nebulynx có xác suất **0,01%**. Dang.

### Stats, mắt, mũ, và “linh hồn”

Mỗi buddy được generate:
- **5 stats**: `DEBUGGING`, `PATIENCE`, `CHAOS`, `WISDOM`, `SNARK` (0-100 mỗi stat)
- **6 kiểu mắt** và **8 lựa chọn mũ** (một số bị khoá theo rarity)
- **Một "soul"**: personality do Claude tạo khi nở lần đầu, viết “đúng vai”

Sprite được render dưới dạng **ASCII art** cao 5 dòng, rộng 12 ký tự, có nhiều frame animation. Có idle animation, reaction animation, và nó ngồi cạnh prompt input.

### Lore

Code nhắc 1–7/4/2026 như một **teaser window**, với launch đầy đủ khoá cho tháng 5/2026. Pet có system prompt hướng dẫn Claude:

```
A small {species} named {name} sits beside the user's input box and 
occasionally comments in a speech bubble. You're not {name} - it's a 
separate watcher.
```

Nên không chỉ cosmetic — buddy có personality riêng và có thể phản hồi khi bạn gọi tên. Mình thật sự mong họ ship nó.

---
## KAIROS - "Always-On Claude"

Trong [`assistant/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates) có một mode tên **KAIROS**: một trợ lý Claude “luôn chạy”, không đợi bạn gõ. Nó quan sát, log, và **chủ động** làm gì đó khi thấy cần.

Cái này bị khoá bởi feature flags `PROACTIVE` / `KAIROS` và hoàn toàn không có trong build public.

### Cách nó hoạt động

KAIROS duy trì **append-only daily log files** — ghi quan sát, quyết định, hành động xuyên ngày. Định kỳ nó nhận prompt `<tick>` để quyết định có chủ động làm gì hay im lặng.

Hệ có **15-second blocking budget**: hành động chủ động nào chặn workflow người dùng quá 15 giây sẽ bị hoãn. Đây là cách Claude cố hữu ích mà không gây phiền.

### Brief mode

Khi KAIROS bật, có output mode đặc biệt tên **Brief**: phản hồi cực ngắn, phù hợp trợ lý luôn chạy mà không spam terminal. Kiểu khác nhau giữa bạn bè nói nhiều và trợ lý chuyên nghiệp chỉ nói khi có thứ đáng nói.

### Tool “độc quyền”

KAIROS có tool mà Claude Code thường không có:

| Tool | What It Does |
|------|-------------|
| **SendUserFile** | Push files directly to the user (notifications, summaries) |
| **PushNotification** | Send push notifications to the user's device |
| **SubscribePR** | Subscribe to and monitor pull request activity |

 ---
## ULTRAPLAN - phiên planning remote 30 phút

Cái này “wild” về mặt hạ tầng.

**ULTRAPLAN** là mode nơi Claude Code đẩy một task planning phức tạp lên **remote Cloud Container Runtime (CCR) session** chạy **Opus 4.6**, cho tối đa **30 phút** để nghĩ, và bạn duyệt kết quả trên browser.

Luồng cơ bản:

1. Claude Code phát hiện task cần planning sâu
2. Nó spin up remote CCR session qua config `tengu_ultraplan_model`
3. Terminal hiển thị trạng thái polling — check mỗi **3 giây** để lấy kết quả
4. UI trên browser cho phép bạn xem planning diễn ra và approve/reject
5. Khi approve, có sentinel `__ULTRAPLAN_TELEPORT_LOCAL__` để “teleport” kết quả về terminal local

---
## "Dream" System - Claude… mơ thật

Đây là một trong những thứ cool nhất trong này.

Claude Code có hệ **autoDream** ([`services/autoDream/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates)) — engine gom trí nhớ chạy nền dưới dạng **forked subagent**. Tên gọi rất có chủ ý. Claude đang… mơ.

Điều này buồn cười vì [tuần trước mình cũng có ý tưởng tương tự cho LITMUS - OpenClaw subagents “leisure time” để đi tìm paper mới](https://github.com/Kuberwastaken/litmus)

### Three-gate trigger

Dream không chạy bừa. Nó có hệ trigger 3 cổng:

1. **Time gate**: 24 giờ kể từ lần mơ trước
2. **Session gate**: ít nhất 5 session kể từ lần mơ trước  
3. **Lock gate**: lấy consolidation lock (tránh mơ song song)

Phải qua cả 3. Tránh over-dreaming lẫn under-dreaming.

### 4 phase

Khi chạy, dream đi qua 4 phase chặt theo prompt ở [`consolidationPrompt.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/query/src/compact.rs):

**Phase 1 - Orient**: `ls` thư mục memory, đọc `MEMORY.md`, skim topic files cũ để biết cần cải thiện gì.

**Phase 2 - Gather Recent Signal**: tìm tín hiệu mới đáng lưu. Nguồn ưu tiên: daily logs → drifted memories → transcript search.

**Phase 3 - Consolidate**: viết/update memory files. Đổi ngày tương đối thành tuyệt đối. Xoá fact bị mâu thuẫn.

**Phase 4 - Prune and Index**: giữ `MEMORY.md` dưới 200 dòng và ~25KB. Bỏ pointer cũ. Giải quyết mâu thuẫn.

Prompt còn ghi rõ:

> *"You are performing a dream - a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly."*

Dream subagent chỉ có **read-only bash** — xem project được nhưng không sửa. Nó chỉ để consolidate memory.

---
## Undercover Mode - "Do Not Blow Your Cover"

Cái này rất hay nếu nhìn từ góc chiến lược công ty.

Nhân viên Anthropic (nhận diện bằng `USER_TYPE === 'ant'`) dùng Claude Code trên repo public/open-source. **Undercover Mode** ([`utils/undercover.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/core/src/lib.rs)) ngăn AI vô tình lộ info nội bộ trong commit/PR.

Khi bật, nó inject vào system prompt:

```
## UNDERCOVER MODE - CRITICAL

You are operating UNDERCOVER in a PUBLIC/OPEN-SOURCE repository. Your commit
messages, PR titles, and PR bodies MUST NOT contain ANY Anthropic-internal
information. Do not blow your cover.

NEVER include in commit messages or PR descriptions:
- Internal model codenames (animal names like Capybara, Tengu, etc.)
- Unreleased model version numbers (e.g., opus-4-7, sonnet-4-8)
- Internal repo or project names
- Internal tooling, Slack channels, or short links (e.g., go/cc, #claude-code-…)
- The phrase "Claude Code" or any mention that you are an AI
- Co-Authored-By lines or any other attribution
```

Logic kích hoạt:
- `CLAUDE_CODE_UNDERCOVER=1` ép ON (kể cả internal repo)
- Nếu không thì **tự động**: active UNLESS remote của repo match allowlist nội bộ
- Không có force-OFF — *“nếu không chắc là internal repo, cứ undercover”*

Từ đây xác nhận:
1. **Nhân viên Anthropic dùng Claude Code để đóng góp open-source** — và AI được dặn giấu việc nó là AI
2. **Codename model nội bộ là tên động vật** — Capybara, Tengu, v.v.
3. **"Tengu"** xuất hiện rất nhiều như prefix feature flag và analytics event — gần như chắc chắn là **codename nội bộ của dự án Claude Code**

Tất cả bị dead-code-eliminate khỏi build public. Nhưng sourcemap thì không quan tâm dead code.

Nó làm mình tự hỏi nội bộ họ đang “quậy” open source repo tới mức nào.

---
## Multi-Agent Orchestration - "Coordinator Mode"

Claude Code có hẳn hệ orchestration đa-agent trong [`coordinator/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/query/src), bật qua `CLAUDE_CODE_COORDINATOR_MODE=1`.

Khi bật, Claude Code chuyển từ single agent thành **coordinator** spawn và quản nhiều worker agent song song. Prompt coordinator ở [`coordinatorMode.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/query/src/agent_tool.rs) là một bài mẫu thiết kế multi-agent:

| Phase | Who | Purpose |
|-------|-----|---------|
| **Research** | Workers (parallel) | Investigate codebase, find files, understand problem |
| **Synthesis** | **Coordinator** | Read findings, understand the problem, craft specs |
| **Implementation** | Workers | Make targeted changes per spec, commit |
| **Verification** | Workers | Test changes work |

Prompt dạy parallelism rất trực diện:

> *"Parallelism is your superpower. Workers are async. Launch independent workers concurrently whenever possible - don't serialize work that can run simultaneously."*

Workers nói chuyện bằng XML `<task-notification>`. Có **scratchpad directory** (gated `tengu_scratch`) để chia sẻ kiến thức bền giữa worker. Và prompt có viên ngọc cấm “lười”:

> *Do NOT say "based on your findings" - read the actual findings and specify exactly what to do.*

Hệ còn có khả năng **Agent Teams/Swarm** (`tengu_amber_flint`) với teammate in-process dùng `AsyncLocalStorage` để cô lập context, teammate theo process dùng tmux/iTerm2 panes, đồng bộ memory team, và gán màu để phân biệt trực quan.

---
## Fast Mode trong nội bộ gọi là "Penguin Mode"

Ừ, họ gọi là Penguin Mode thật. API endpoint trong [`utils/fastMode.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/core/src/lib.rs) là:

```typescript
const endpoint = `${getOauthConfig().BASE_API_URL}/api/claude_code_penguin_mode`
```

Config key là `penguinModeOrgEnabled`. Kill-switch là `tengu_penguins_off`. Analytics event fail là `tengu_org_penguin_mode_fetch_failed`. Penguin all the way down.

---
## Kiến trúc system prompt

System prompt không phải một string đơn như đa số app — nó được build từ **các section mô-đun, cache được** và compose runtime trong [`constants/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/core/src).

Kiến trúc dùng marker `SYSTEM_PROMPT_DYNAMIC_BOUNDARY` để tách prompt thành:
- **Static sections** - cache được giữa org (không đổi theo user)
- **Dynamic sections** - nội dung theo user/session, làm “vỡ cache” khi đổi

Có function tên `DANGEROUS_uncachedSystemPromptSection()` cho phần volatile bạn muốn phá cache có chủ đích. Tên gọi đã nói lên việc ai đó từng “ăn hành” vì cache prompt.

### Hướng dẫn về rủi ro an ninh

Một section đáng chú ý là `CYBER_RISK_INSTRUCTION` trong [`constants/cyberRiskInstruction.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/core/src/lib.rs), có header cảnh báo lớn:

```
IMPORTANT: DO NOT MODIFY THIS INSTRUCTION WITHOUT SAFEGUARDS TEAM REVIEW
This instruction is owned by the Safeguards team (David Forsythe, Kyla Guru)
```

Vậy là ta biết rõ ai ở Anthropic sở hữu các quyết định “ranh giới an ninh” và nó được quản trị bởi team cụ thể, có tên người. Nội dung hướng dẫn đặt ranh: security testing được phép thì ok, nhưng kỹ thuật phá hoại và compromise supply chain thì không.

---
## Registry tool đầy đủ - 40+ tool

Hệ tool của Claude Code nằm ở [`tools/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/tools/src). Đây là danh sách:

| Tool | What It Does |
|------|-------------|
| **AgentTool** | Spawn child agents/subagents |
| **BashTool** / **PowerShellTool** | Shell execution (with optional sandboxing) |
| **FileReadTool** / **FileEditTool** / **FileWriteTool** | File operations |
| **GlobTool** / **GrepTool** | File search (uses native `bfs`/`ugrep` when available) |
| **WebFetchTool** / **WebSearchTool** / **WebBrowserTool** | Web access |
| **NotebookEditTool** | Jupyter notebook editing |
| **SkillTool** | Invoke user-defined skills |
| **REPLTool** | Interactive VM shell (bare mode) |
| **LSPTool** | Language Server Protocol communication |
| **AskUserQuestionTool** | Prompt user for input |
| **EnterPlanModeTool** / **ExitPlanModeV2Tool** | Plan mode control |
| **BriefTool** | Upload/summarize files to claude.ai |
| **SendMessageTool** / **TeamCreateTool** / **TeamDeleteTool** | Agent swarm management |
| **TaskCreateTool** / **TaskGetTool** / **TaskListTool** / **TaskUpdateTool** / **TaskOutputTool** / **TaskStopTool** | Background task management |
| **TodoWriteTool** | Write todos (legacy) |
| **ListMcpResourcesTool** / **ReadMcpResourceTool** | MCP resource access |
| **SleepTool** | Async delays |
| **SnipTool** | History snippet extraction |
| **ToolSearchTool** | Tool discovery |
| **ListPeersTool** | List peer agents (UDS inbox) |
| **MonitorTool** | Monitor MCP servers |
| **EnterWorktreeTool** / **ExitWorktreeTool** | Git worktree management |
| **ScheduleCronTool** | Schedule cron jobs |
| **RemoteTriggerTool** | Trigger remote agents |
| **WorkflowTool** | Execute workflow scripts |
| **ConfigTool** | Modify settings (**internal only**) |
| **TungstenTool** | Advanced features (**internal only**) |
| **MCPTool** | Generic MCP tool execution |
| **McpAuthTool** | MCP server authentication |
| **SyntheticOutputTool** | Structured output via dynamic JSON schemas |
| **SuggestBackgroundPRTool** | Suggest background PRs (**internal only**) |
| **VerifyPlanExecutionTool** | Verify plan execution (gated by `CLAUDE_CODE_VERIFY_PLAN`) |
| **CtxInspectTool** | Context window inspection (gated by `CONTEXT_COLLAPSE`) |
| **TerminalCaptureTool** | Terminal panel capture (gated by `TERMINAL_PANEL`) |
| **CronCreateTool** / **CronDeleteTool** / **CronListTool** | Granular cron job management (under `ScheduleCronTool/`) |
| **SendUserFile** / **PushNotification** / **SubscribePR** | KAIROS-exclusive tools |

Tool được đăng ký qua `getAllBaseTools()` rồi filter theo feature gate, user type, env flag, và luật deny permission. Có **tool schema cache** ([`toolSchemaCache.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/tools/src/lib.rs)) cache JSON schema để tiết kiệm token.

---
## Hệ permission và security

Permission system của Claude Code trong [`tools/permissions/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/core/src) phức tạp hơn “allow/deny” rất nhiều:

**Permission Modes**: `default` (hỏi tương tác), `auto` (auto-approve bằng ML classifier dựa trên transcript), `bypass` (bỏ check), `yolo` (deny all — đặt tên mỉa mai)

**Risk Classification**: mọi action của tool được phân loại **LOW**, **MEDIUM**, hoặc **HIGH** risk. Có **YOLO classifier** — ML nhanh để quyết định permission tự động.

**Protected Files**: `.gitconfig`, `.bashrc`, `.zshrc`, `.mcp.json`, `.claude.json` và nhiều file khác bị chặn khỏi auto-edit.

**Path Traversal Prevention**: URL-encoded traversal, Unicode normalization, backslash injection, case-insensitive path manipulation — đều được xử lý.

**Permission Explainer**: một LLM call riêng giải thích rủi ro tool cho user trước khi approve. Khi Claude nói “lệnh này sẽ sửa git config”, phần giải thích đó cũng do Claude generate.

---
## Beta headers ẩn và API feature chưa release

File [`constants/betas.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/api/src/lib.rs) lộ ra mọi beta feature Claude Code “đàm phán” với API:

```typescript
'interleaved-thinking-2025-05-14'      // Extended thinking
'context-1m-2025-08-07'                // 1M token context window
'structured-outputs-2025-12-15'        // Structured output format
'web-search-2025-03-05'                // Web search
'advanced-tool-use-2025-11-20'         // Advanced tool use
'effort-2025-11-24'                    // Effort level control
'task-budgets-2026-03-13'              // Task budget management
'prompt-caching-scope-2026-01-05'      // Prompt cache scoping
'fast-mode-2026-02-01'                 // Fast mode (Penguin)
'redact-thinking-2026-02-12'           // Redacted thinking
'token-efficient-tools-2026-03-28'     // Token-efficient tool schemas
'afk-mode-2026-01-31'                  // AFK mode
'cli-internal-2026-02-09'             // Internal-only (ant)
'advisor-tool-2026-03-01'              // Advisor tool
'summarize-connector-text-2026-03-13'  // Connector text summarization
```

`redact-thinking`, `afk-mode`, và `advisor-tool` cũng chưa release.

---
## Model sắp tới - Capybara, Opus 4.7, và Sonnet 4.8

Codebase có reference tới các model Anthropic chưa công bố:

- **Claude "Capybara"** - họ model mới đã v2, có biến thể `capybara-v2-fast` chuẩn bị với **1M context window**
- Capybara có tier "fast" và tier thinking thường
- **Opus 4.7** và **Sonnet 4.8** đã được reference trong code

### Kỹ thuật production quanh Capybara

Code cho thấy Anthropic gặp một failure mode production thật: Capybara có thể dừng sinh sớm khi prompt “giống” turn boundary sau tool results. Thay vì chờ model fix, họ giảm thiểu bằng **prompt-shape surgery**:

1. **Force marker an toàn** (`Tool loaded.`) để tránh boundary mơ hồ
2. **Relocate các block “rủi ro”** có thể kích stop sớm
3. **Smoosh reminder text vào tool results** để giữ dòng generation
4. **Thêm marker không rỗng cho output tool rỗng** để tránh làm model rối

Tất cả bọc bằng **kill-switch gates** (`tengu_*`) để rollout theo giai đoạn và revert nhanh.

- Comment có **bằng chứng A/B test cụ thể** (không chung chung) → vùng này “launch-critical”
- Comment kiểu *"un-gate once validated on external via A/B"* xác nhận **ant/internal users là canary** trước rollout rộng
- Diễn giải mạnh nhất: Anthropic đang chuẩn bị **họ model Capybara** với biến thể fast (`capybara-v2-fast`), hỗ trợ tới **1M context**

Không có gì xác nhận ngày launch hay SKU chính thức, nhưng chữ ký implement hợp một model family đang chuẩn bị release ;)

---
## Feature gating - internal vs external build

Đây là phần kiến trúc thú vị nhất.

Claude Code dùng **compile-time feature flags** qua `feature()` của Bun (`bun:bundle`). Bundler sẽ **constant-fold** và **dead-code-eliminate** các nhánh bị gate khỏi build public. Danh sách flag:

| Flag | What It Gates |
|------|--------------|
| `PROACTIVE` / `KAIROS` | Always-on assistant mode |
| `KAIROS_BRIEF` | Brief command |
| `BRIDGE_MODE` | Remote control via claude.ai |
| `DAEMON` | Background daemon mode |
| `VOICE_MODE` | Voice input |
| `WORKFLOW_SCRIPTS` | Workflow automation |
| `COORDINATOR_MODE` | Multi-agent orchestration |
| `TRANSCRIPT_CLASSIFIER` | AFK mode (ML auto-approval) |
| `BUDDY` | Companion pet system |
| `NATIVE_CLIENT_ATTESTATION` | Client attestation |
| `HISTORY_SNIP` | History snipping |
| `EXPERIMENTAL_SKILL_SEARCH` | Skill discovery |

Ngoài ra, `USER_TYPE === 'ant'` gate các feature nội bộ: staging API (`claude-ai.staging.ant.dev`), internal beta headers, Undercover mode, command `/security-review`, `ConfigTool`, `TungstenTool`, và debug dump prompt ra `~/.config/claude/dump-prompts/`.

**GrowthBook** xử lý runtime feature gating với cache rất “gắt”. Flag prefix `tengu_` điều khiển từ fast mode tới memory consolidation. Nhiều check dùng `getFeatureValue_CACHED_MAY_BE_STALE()` để tránh block main loop — dữ liệu stale được xem là chấp nhận được cho gate.

---
## Một vài phát hiện đáng chú ý khác

### Upstream proxy
Thư mục [`upstreamproxy/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/bridge/src) có proxy relay “container-aware”, dùng **`prctl(PR_SET_DUMPABLE, 0)`** để ngăn ptrace heap memory với cùng UID. Nó đọc session token từ `/run/ccr/session_token` trong CCR container, tải CA cert, và chạy relay CONNECT→WebSocket local. Anthropic API, GitHub, npmjs.org, và pypi.org được loại khỏi proxy.

### Bridge mode
Một bridge JWT-auth trong [`bridge/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/bridge/src) để tích hợp với claude.ai. Hỗ trợ work mode: `'single-session'` | `'worktree'` | `'same-dir'`. Có trusted device token cho tier bảo mật cao hơn.

### Codenames lộ trong migrations
Thư mục [`migrations/`](https://github.com/kuberwastaken/claude-code/tree/main/src-rust/crates/core/src) lộ lịch sử codename:
- `migrateFennecToOpus` - **"Fennec"** là codename Opus
- `migrateSonnet1mToSonnet45` - Sonnet 1M context thành Sonnet 4.5
- `migrateSonnet45ToSonnet46` - Sonnet 4.5 → Sonnet 4.6
- `resetProToOpusDefault` - Pro user từng bị reset về Opus

### Attribution header
Mọi API request gồm:
```
x-anthropic-billing-header: cc_version={VERSION}.{FINGERPRINT}; 
  cc_entrypoint={ENTRYPOINT}; cch={ATTESTATION_PLACEHOLDER}; cc_workload={WORKLOAD};
```
Feature `NATIVE_CLIENT_ATTESTATION` cho Bun HTTP stack overwrite placeholder `cch=00000` bằng hash tính toán — kiểu kiểm chứng request đến từ Claude Code “thật”.

### Computer Use - "Chicago"
Claude Code có full Computer Use implementation, codename nội bộ **"Chicago"**, build trên `@ant/computer-use-mcp`. Nó cung cấp screenshot, click/keyboard input, và coordinate transform. Bị gate theo gói Max/Pro (internal user có bypass).

### Pricing
Giá trong [`utils/modelCost.ts`](https://github.com/kuberwastaken/claude-code/blob/main/src-rust/crates/api/src/lib.rs) khớp [pricing công khai của Anthropic](https://docs.anthropic.com/en/docs/about-claude/models). Không có gì mới.

---
## Lời kết

Không phóng đại: đây là một trong những cái nhìn toàn diện nhất mà chúng ta từng có về cách một AI coding assistant production hoạt động “dưới nắp capo”. Bằng chính source code.

Vài điểm nổi bật:

**Engineering thật sự ấn tượng.** Đây không phải project cuối tuần bọc trong CLI. Multi-agent coordination, dream system, three-gate trigger, compile-time feature elimination — đều là những hệ thống được nghĩ kỹ.

**Còn rất nhiều thứ sắp tới.** KAIROS (always-on Claude), ULTRAPLAN (planning remote 30 phút), Buddy companion, coordinator mode, agent swarms, workflow scripts… codebase đi trước bản public một đoạn xa. Phần lớn bị feature-gated và “vô hình” với build ngoài.

**Văn hoá nội bộ lộ rõ.** Codename động vật (Tengu, Fennec, Capybara), tên feature vui (Penguin Mode, Dream System), pet Tamagotchi kèm gacha. Có vẻ một số người ở Anthropic đang “enjoy”.

Nếu có một takeaway: security khó. Nhưng `.npmignore` còn khó hơn, apparently :P

---

Một bài viết bởi [Kuber Mehta](https://kuber.studio/)

