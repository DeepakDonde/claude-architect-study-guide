# CCA-F: D2 Domain Summary — Tool Design & MCP Integration

**Exam weight:** 18%. **Numbering:** course D2 = official blueprint Domain 4 (Tool Design & MCP Integration). Some study websites call it Domain 2 — always go by the domain name.

**Coverage:** my three D2 note files. There is no D2 index file, so missing lectures could not be checked. One exam-guide topic that is missing from my notes was added as a gap fill (T26).

**Correction to my notes:** strict tool use does **not** enforce minimum/maximum (see T4).

**Legend:** ❌ wrong idea (exam trap) → ✅ right idea · ⚠️ GAP FILL = not from my notes · ➕ = exam-guide or docs check (added)

| Part | Source notes | Topics | Gap fills |
|---|---|---|---|
| 1 · Tool Design Fundamentals | D2-1 | T1–T8 | — |
| 2 · Building MCP Servers & Clients | D2-2 | T9–T15 | — |
| 3 · MCP Production & Built-in Tools | D2-3 | T16–T26 | T26 ⚠️ |


---

## Part 1 · Tool Design Fundamentals (T1–T8)

*From the D2-1 summary.*

### T1 · What a Tool Is

**Example:** A smart friend locked in a room can only give advice. Give them a phone, a calculator, and a computer, and they can really book the table and check the weather. The tools don't make the friend smarter — they let the friend act.

**Key facts**
- A tool lets Claude **take an action** or get **live data** (today's date, the weather, an order status). Text alone cannot do this.
- A tool definition has 3 parts: **name · description · input schema** (JSON Schema for the parameters).
- The **description is the most important part**. Claude reads it to decide whether and when to call the tool. Claude cannot see your code.
- Flow: you give Claude the tool list → Claude asks to call one, with inputs → **your code runs it** → the result goes back → Claude writes the answer.
- For high-risk tools (pay, delete, book), add a human confirmation step before the tool runs.

**Exam traps**
- ❌ The assistant doesn't know today's date, so add examples or a longer prompt → ✅ Give it a tool that returns the current date.
- ❌ Claude calls cancel_order when the user only wanted a status check, so change the temperature → ✅ Fix the unclear tool descriptions.

### T2 · Writing Tool Descriptions That Route Correctly

**Example:** Two waiters both described as "helps people" — a customer cannot choose. "Takes food orders at the table" vs. "handles the bill" — now it is clear.

**Key facts**
- A good description says **What · When · When-NOT · Which input**.
- Be specific: "Search products by name, category, or price range", not "search stuff".
- Draw borders between similar tools: "Do NOT use this to cancel — use cancel_order." Saying what a tool does NOT do is the strongest fix for wrong routing.
- Include the words users really say ("refund", "money back", "return").
- The name and the description must match.
- **One tool = one job.** A tool that searches, cancels, and refunds confuses routing. Split it.

**Exam traps**
- ❌ Merge two overlapping tools into one big tool → ✅ Keep them separate and add "use the other tool for X" to each.
- ❌ Make the descriptions shorter → ✅ Make them clearer and more specific.

### T3 · Designing Input Schemas (Required, Optional, Constraints)

**Example:** A pizza order with one empty box ("big cheese one, the usual") vs. a form with size (small/medium/large), toppings from a list, quantity 1–10, and a required address.

**Key facts**
- Memory trick: **Type · Enum · Required · Describe · Constrain**.
- Use the right **type** (string, number/integer, boolean, array, object). Money is a **number**, not "$100" text.
- **enum** locks a value to a fixed list (["low", "medium", "high"]). One of the strongest tools for precision.
- **Required** = the tool cannot run without it. **Optional** = nice to have; give it a **default** (for example, newsletter = false).
- Give **every parameter a description** with its format and an example ("date in YYYY-MM-DD, e.g. 2026-08-01"). Claude reads these.
- Constraints: minimum/maximum for numbers, pattern for text.
- Prefer **separate structured fields** over one big free-text "details" field. **One parameter = one meaning.**
- Separate optional filters (category, min_price, max_price) let Claude fill only what the user asked for.

**Exam traps**
- ❌ Values like "high", "urgent", "Critical!!" arrive, so add prompt examples → ✅ Add an enum.
- ❌ Keep one free-text "details" field (the date is often missing) → ✅ Split it into typed fields (date, time, service) and mark the needed ones required.
- ❌ Make every field required → ✅ Only the true must-haves. Optional fields get defaults.

### T4 · Strict Tool Use (strict: true)

**Example:** A paper form vs. an airport check-in screen that accepts only a real number and will not submit until every required box is filled.

**Key facts**
- Add `"strict": true` to the tool definition. The tool **name and inputs are then guaranteed** to match your schema (right types, required fields present). It works by grammar-constrained decoding.
- Every object in the schema needs `"additionalProperties": false`.
- Strict tool use controls **tool inputs**. Its sister feature, **JSON outputs** (`output_config.format`), controls Claude's **text answer**. To extract data as clean JSON with no tool call → JSON outputs.
- Use strict mode where a wrong type would crash your code (billing, chains of agent tools).
- Strict mode guarantees a **correct call**. The description still decides **which tool** is chosen.

**Exam traps**
- ❌ Claude sometimes sends amount: "50" as text, so add prompt examples or retries → ✅ strict: true guarantees the type.
- ➕ ❌ "Strict mode enforces minimum and maximum" → ✅ Not true. Official docs: numeric and length limits (minimum, maximum, minLength, maxLength) are **not enforced** by structured outputs (the SDKs strip them). Check ranges in your own code. *(My notes say the opposite.)*

### T5 · Error Messages, Idempotency, Partial Success

**Example:** A delivery driver drops off 8 of 10 parcels. A useful report says which 2 failed and why — not just "delivery done". And pressing an elevator button twice does not call two elevators (idempotency).

**Key facts**
- **Return errors as data, don't throw.** Send a `tool_result` with `is_error: true` and a clear message. A thrown exception can stop the whole agent.
- A useful error says **What failed · Why · What next**: "Order ID 12345 not found. Ask the user to confirm the ID, or use search_orders."
- **Idempotency:** the same call twice has the same effect as once (no double charge). Use a **unique request key**. Idempotent tools are safe to retry.
- **Partial success:** report exactly what worked and what failed: "Sent 8 of 10. Failed 2: invalid addresses [a@x, b@y]."
- Protect the loop: a hard iteration cap (as a safety net) + **per-tool timeouts**, so one slow or broken tool cannot block everything.
- ➕ In **MCP**, a failed tool returns a normal result with `isError: true` (the Claude API's `tool_result` uses `is_error`). Also include an **error type** and a **retryable yes/no**, so the agent doesn't waste retries (see D5 L118).

**Exam traps**
- ❌ Return "Error" or an empty string → ✅ A specific, actionable message with is_error: true.
- ❌ Report "success" when 3 of 10 invoices failed → ✅ Report a partial success with the names and reasons.
- ❌ Remove retries so customers are never charged twice → ✅ Make the tool idempotent.

### T6 · Tool Scoping (Least Privilege)

**Example:** A hotel key card opens your room and the gym, but not other guests' rooms or the safe. If you lose it, the damage stays small.

**Key facts**
- **Least privilege:** give each agent and each tool only the access its job needs. Smaller access = smaller damage (**blast radius**).
- Two levels: **agent-level** (which tools an agent can see — a support agent gets search_orders, not delete_database) and **permission-level** (what a tool may touch: read-only vs. write, only this user's data, limits like refunds ≤ $100 without approval).
- Each subagent gets a narrow tool set for its role (links to D1 L17).
- In MCP: **access scoping** = which client may reach which tools and resources, tied to **authentication + authorization**.
- Why it matters: safety (limits prompt-injection damage), security, **clearer routing** (fewer tools = better choices), and trust.
- ➕ Exam-guide example: an agent with **18 tools** chooses less reliably than one with **4–5 tools** for its role. Give each agent only its role's tools; put other tools on other subagents.

**Exam traps**
- ❌ A support agent keeps a delete_user_account tool "just in case" → ✅ Remove it.
- ❌ A web-research subagent also gets update_database → ✅ Remove it. Untrusted web content could trick it (prompt injection).
- ❌ Unlimited automatic refunds → ✅ Auto-approve small amounts; above a limit, require human approval.

### T7 · Controlling Tool Selection (tool_choice)

**Example:** Instructions to an employee: "Use your judgment" (auto). "You must use one of your tools — you pick" (any). "Use the phone, specifically" (tool). "Don't touch any tools, just answer" (none).

| Option | What it does | Use it when |
|---|---|---|
| `auto` | Claude decides if and which tool (the default when tools exist) | Normal agents |
| `any` | Claude must call a tool, but picks which one | Every turn must take an action |
| `tool` + name | Forces one specific tool | Guaranteed structured output through one tool |
| `none` | No tools this turn | A text-only step, or testing |

**Key facts**
- With `any` or `tool`, the API **prefills** the response to force a tool call, so Claude writes **no text before** the tool_use block. If you want an explanation first, use `auto`.
- Changing `tool_choice` **invalidates cached message blocks** (prompt caching).
- `disable_parallel_tool_use` stops Claude from calling several tools at once.
- Three levers: **descriptions guide** the choice (under auto) · **tool scoping** decides what can be picked · **tool_choice overrides** the choice.

**Exam traps**
- ❌ Use `any` to guarantee that record_sentiment runs → ✅ `any` forces *some* tool. Use `tool` with the name.
- ❌ Force `tool` on every request to fix wrong picks between two similar tools → ✅ Fix the descriptions first.

### T8 · Tool Search for Large Catalogs

**Example:** A library. You don't pile every book on your desk. You use the catalog and bring only the 2–3 books you need.

**Key facts**
- Two problems with huge tool lists: **context bloat** (5 servers like GitHub, Slack, Sentry, Grafana, and Splunk ≈ 55k tokens of definitions) and **worse selection** (accuracy drops past about 30–50 tools).
- Tool search: mark tools with `defer_loading: true`. Claude **searches the catalog and loads only the 3–5 tools** it needs. Token use drops by 85% or more.
- Two search methods: **regex** (exact pattern) and **BM25** (keywords / natural language).

**Exam traps**
- ❌ 500 tools across many MCP servers, so delete most of them or write longer descriptions → ✅ Enable tool search with defer_loading.


---

## Part 2 · Building MCP Servers & Clients (T9–T15)

*From the D2-2 summary.*

### T9 · MCP Basics: Host, Client, Server

**Example:** MCP is like a standard wall socket: every device has the same plug shape. In a restaurant, you (the user) sit in the restaurant (host app). The manager (Claude) decides what is needed. The waiter (MCP client) carries messages to the kitchen (MCP server), which has the stove (tools), the pantry (resources), and the menu (prompts).

**Key facts**
- **MCP (Model Context Protocol)** = one shared way for AI apps to connect to outside tools and data. Build once, plug in anywhere.
- **Host** = the app you use (Claude Desktop, an IDE, your chatbot). It holds the clients and talks to Claude.
- **MCP client** = lives inside the host. **One client per server.** It is the only part that talks to the server.
- **MCP server** = a separate program (local or remote) that **exposes** tools, resources, and prompts.
- Round trip: user → host → Claude decides → client → server runs a tool or returns a resource → back to Claude → answer. It can **loop** many times (the agent loop).
- MCP itself is the set of rules (JSON-RPC 2.0 messages) on the connection between client and server.

**Exam traps**
- ❌ "The user talks directly to the MCP server" → ✅ The user talks to the host. Only the client talks to the server.
- ❌ "Tools live inside Claude" → ✅ They live on the MCP server.

### T10 · The Three Primitives — Who Controls Each?

**Example:** A code review. The user clicks a /code-review template (prompt). The app attaches the open code files (resources). Claude decides to call lint_file and post_comment (tools).

| Primitive | What it is | Who controls it | Discover → use |
|---|---|---|---|
| Tool | An action (can change things) | **Model** — Claude decides | tools/list → tools/call |
| Resource | Read-only data, found by a URI | **Application** — the host decides | resources/list → resources/read |
| Prompt | A reusable template, often a slash command | **User** — the user picks | prompts/list → prompts/get |

**Key facts**
- Memory line: **Tools = model picks · Resources = app picks · Prompts = user picks.**
- Tool vs. resource test: does it **take an action or change something** (tool), or is it **read-only data** (resource)?
- Resources **never have side effects**. Anything that writes, sends, or deletes must be a tool.
- Each primitive can be used alone. Claude can use a tool with no prompt or resource.

**Exam traps**
- ❌ delete_user exposed as a resource, so loading it as context deleted data → ✅ Actions with side effects belong in tools.
- ❌ "Prompts are model-controlled" → ✅ Prompts are user-controlled.

### T11 · Building a Server in Python (FastMCP)

**Example:** An office assistant robot. Tools = what it can do ("book a meeting room"). Resources = documents it can read to you (the holiday list). Prompts = request forms it offers ("write a leave request").

**Key facts**
- Setup: Python 3.10+, `pip install "mcp[cli]"`, create `FastMCP("name")`, then call `mcp.run()`.
- Decorators map to the primitives: `@mcp.tool()` · `@mcp.resource("uri")` · `@mcp.prompt()`.
- For tools, the **docstring becomes the description** (the routing signal) and the **type hints become the JSON Schema**.
- Resources use URIs: **static** (`config://company-holidays`) or **templated** (`employees://{emp_id}/profile`). One templated definition serves many items.
- Prompts can take arguments and return a filled-in message.

**Exam traps**
- ❌ Create one resource per employee by hand → ✅ Use a templated resource URI.
- ❌ A tool docstring that says "does stuff" → ✅ A clear docstring: what it does and when to use it.

### T12 · Building a Client: Lifecycle, Transports, Routing

**Example:** A universal TV remote. First it does a handshake to learn which buttons work, then it presses them. One remote can control many TVs (multi-server routing).

**Key facts**
- Lifecycle: **Initialization** (client sends `initialize` → server replies → client sends `notifications/initialized`; they agree on version and capabilities) → **Operation** (list, call, read, get) → **Shutdown**. Skip `initialize` and every call fails.
- Transports: **stdio** = local subprocess · **Streamable HTTP** = remote / production · **SSE** = old and deprecated.
- Python SDK: stdio gives `(read, write)`; Streamable HTTP gives `(read, write, _)` — the extra item is a session-ID helper.
- Use `async with` for the transport and the session, so both **close cleanly** (no leaks).
- **Discover tools at runtime** with `list_tools()`. Never hardcode the tool list.
- Many servers: one session per server, and the **client owns the routing** (a map from tool name → server). If two servers have the same tool name, **add a prefix** (for example, `hr__search`).

**Exam traps**
- ❌ list_tools() fails with a protocol error, so switch the transport → ✅ You forgot `session.initialize()`.
- ❌ Hardcode the tools "so it starts faster" → ✅ Discover them. The server can change.
- ❌ "Too many values to unpack" with `(read, write)` on HTTP → ✅ Streamable HTTP returns three values.

### T13 · Two Common Server Bugs

**Example:** A notice pinned on the shop door after the shop has closed — nobody sees it (code after `run()`). One worker takes a 10-minute phone call at the only till, and the whole queue stops (a blocking call).

**Key facts**
- `run()` **blocks**: code after it never runs. **Register every tool before `run()`.**
- To add tools while the server runs: add them inside the running server and send **`notifications/tools/list_changed`**, so clients re-fetch the list.
- An MCP server runs on **one async event loop**. One **blocking (synchronous) call**, like a big file read, freezes **all** client connections, not just one.
- Fix: run the blocking work in a thread (`asyncio.to_thread` / `run_in_executor`) or use async I/O (for example, `aiofiles`).

**Exam traps**
- ❌ Fix the stall by making the file smaller or adding more clients → ✅ Move the blocking call off the event loop.
- ❌ Re-run `initialize` so the client sees new tools → ✅ The server sends tools/list_changed; the client lists the tools again.

### T14 · Resource Subscriptions (Notify, Then Fetch)

**Example:** Waiting for a parcel. You can call the courier every 5 minutes (polling), or sign up for a text alert and then check the door (subscription).

**Key facts**
- The client asks to be told when **one resource's content changes**, instead of polling.
- The server must declare `"resources": { "subscribe": true }` at the handshake.
- Flow: `resources/subscribe` (with the URI) → a change happens → the server sends `notifications/resources/updated` (**URI only, no content**) → the client calls `resources/read` → later, `resources/unsubscribe` (to avoid leaks).
- Best for data that changes: logs, live metrics, shared documents.

| Notification | Meaning |
|---|---|
| `notifications/resources/updated` | One subscribed resource's **content** changed → read it again |
| `notifications/resources/list_changed` | The **list** of resources changed (added or removed) → list again |

**Exam traps**
- ❌ Expect the new content inside the notification → ✅ It only says "changed". The client must read it.
- ❌ Re-read the resource every 2 seconds → ✅ Subscribe.

### T15 · System Prompt vs. MCP Prompt

**Example:** The system prompt is an employee's job description and standing rules from day one (always on). An MCP prompt is a ready-made form at the front desk that a customer picks to make one request.

| | System prompt | MCP prompt |
|---|---|---|
| Controlled by | Developer / app | **User** (often a slash command) |
| Lifespan | The whole conversation, always on | Inserted once when picked; then it is a normal message |
| Lives in | The API `system` field | An MCP server (prompts/list, prompts/get) |
| Takes arguments? | No | Yes (for example, order_id) |

**Key facts**
- Standing rules (tone, safety, "never reveal internal data") → **system prompt**.
- A reusable task starter with parameters → **MCP prompt**.

**Exam traps**
- ❌ Put "never reveal internal data" in an MCP prompt → ✅ It only applies when the user picks it. Put standing rules in the system prompt.


---

## Part 3 · MCP Production & Built-in Tools (T16–T26)

*From the D2-3 summary.*

### T16 · STDIO Transport (Local)

**Example:** You and an assistant at the same desk, passing paper notes. No phone, no internet. You pass a note (stdin); they pass an answer back (stdout).

**Key facts**
- MCP has **two official transports**: **STDIO** (local) and **Streamable HTTP** (remote).
- STDIO: the client **starts the server as a child process** on the same machine. Messages are JSON-RPC 2.0 lines over stdin/stdout.
- **One process per client**, so it does not scale to many users.
- **No network, no login** (same-machine trust). Very low latency and easy to set up.
- Best for local developer tools: file system, Git, a local database.
- **stdout is only for protocol messages. Logs go to stderr.**

**Exam traps**
- ❌ `print("debug")` in a STDIO server, and now messages look corrupted → ✅ Write logs to stderr.
- ❌ STDIO for a cloud server shared by 200 people → ✅ Streamable HTTP.

### T17 · Streamable HTTP Transport (Remote / Production)

**Example:** A call centre with one phone number. A quick question gets one answer. A long request keeps the line open with live updates.

**Key facts**
- **One endpoint** (for example, `/mcp`): **POST** to send messages, **GET** to open a stream for messages from the server.
- The server chooses the reply: a short task → one reply (`application/json`); a long task → an **SSE stream** (`text/event-stream`) with progress updates.
- **Scales to many users** and **needs authentication** (OAuth 2.1, bearer token).
- **`Mcp-Session-Id`**: the server issues it at initialize; the client sends it on every later request. It is **not** the auth token.
- **`Last-Event-ID`**: resumes a dropped stream without losing messages.
- Send the `MCP-Protocol-Version` header on every request (from spec 2025-06-18).
- It replaced the old **HTTP+SSE** transport (spec 2024-11-05; deprecated in 2025-03-26 — two endpoints, no resume). **WebSocket is not in the MCP spec.**
- In a stream, progress messages are notifications (no id). The message with the request's **id + result** completes the request.

**Exam traps**
- ❌ HTTP+SSE or WebSocket for a new remote server → ✅ Streamable HTTP.
- ❌ "Mcp-Session-Id is the password / auth token" → ✅ It only ties requests to one session.
- ❌ The stream dropped, so restart the whole task → ✅ Reconnect with Last-Event-ID (and the Mcp-Session-Id).

### T18 · Sampling — The Server Asks the Client's Model

**Example:** A junior worker (the server) needs an expensive machine he doesn't own. He asks the manager (the client). She reads the request, may edit or reject it, runs it on her machine (the model), and checks the result before handing it back.

**Key facts**
- **Sampling** = an LLM call **started by the server** and run by the client: `sampling/createMessage`. The direction is flipped compared with `tools/call`.
- The server needs **no model and no API key**. The client owns the model and pays for it.
- **Human-in-the-loop safety:** the host can show the request for approval and review the output.
- The client must declare `"sampling": {}` at initialize, or the server cannot use it.
- `modelPreferences` is only a **hint**. The **client chooses** the model.
- Three client features (the server asks): **Sampling** (use the client's model) · **Roots** (which folders am I allowed to use?) · **Elicitation** (ask the **user** for missing input).

**Exam traps**
- ❌ Ship the server with its own hard-coded API key → ✅ Use sampling.
- ❌ "The server forces the exact model" → ✅ The client decides; the server only hints.

### T19 · Notifications, Roots, Progress

**Example:** A notification is a doorbell — it expects no reply. Roots are the rooms a cleaner is allowed to enter. Progress is the food-app tracker: received → cooking → out for delivery.

**Key facts**
- **Notification** = a one-way message with **no id and no reply**. It can go in both directions.
- Common ones: `notifications/initialized` (client: handshake done) · `notifications/tools/list_changed` (server: re-fetch my tools) · `notifications/progress` · `notifications/message` (a log line) · `notifications/cancelled`.
- **Roots** = the folders the **client** allows the server to work in (a file-system boundary; least privilege). When they change → `notifications/roots/list_changed`.
- **Progress:** the client adds a `progressToken` (inside `_meta`). The server sends `notifications/progress` with the same token + progress (+ total). The token links each update to its request.
- Progress updates and root changes are both **delivered as notifications**.

**Exam traps**
- ❌ A file server gets access to the whole disk ("/") → ✅ Set roots to the project folder only.
- ❌ "The progressToken authenticates the user" → ✅ It links each update to the right request.

### T20 · Log Levels

**Example:** Hospital triage: a routine check-up (info), "slightly high temperature, watch it" (warning), "needs treatment now" (error).

**Key facts**
- MCP uses syslog levels: debug < info < notice < warning < error < critical < alert < emergency. **Know at least: debug < info < warning < error.**
- The client sets the minimum level with **`logging/setLevel`**. The server then sends **that level and everything more serious**.
- Each log line arrives as `notifications/message` with a `level` field.

**Exam traps**
- ❌ setLevel "warning" means only warnings → ✅ Warning, error, and everything above.
- ❌ Flooded with debug lines, so turn off notifications → ✅ Raise the minimum level.

### T21 · Tool Boundaries, Access Scoping, Least Privilege (MCP)

**Example:** Hotel staff keys. The cleaner's card opens guest rooms, but not the safe. Nobody gets a master key "just in case".

**Key facts**
- **Boundary** = the edge of one tool (what it can and cannot do). **Scope** = how wide the access is. **Least privilege** = the smallest access that works, **by default** (start narrow; widen only for a real need).
- Split big tools: `read_file`, `write_file`, `delete_file` instead of one `manage_files` that also runs shell commands. Then grant only what is needed.
- MCP ways to scope: **roots** (one project folder) · **read-only database accounts** (only the needed tables) · **scoped OAuth tokens** ("read calendar", not "delete events").
- Payoff: a **small blast radius**, less prompt-injection damage, easier audits.
- It is a **design choice** (built into the tools, roots, and tokens), not just a runtime check. (See T6.)

**Exam traps**
- ❌ "Give broad access now, tighten it later" → ✅ Least privilege by default.
- ❌ A reporting tool uses a full admin database account → ✅ A read-only account, scoped to the tables it needs.

### T22 · Production: Authentication, Versioning, Reliability

**Example:** An office with a security desk. You show your badge (the token), and it lists the floors you may enter (the scopes). Two people agree on one language before they talk (version negotiation).

**Key facts**
- **Authentication:** remote = required (**OAuth 2.1**, `Authorization: Bearer …`); STDIO = none. Use **short-lived, scoped tokens per user**, never one shared hard-coded admin key.
- **Versioning:** the spec has dated versions (for example, 2025-06-18, 2025-11-25). The client proposes a `protocolVersion` in initialize, and the server replies with the one it will use (**version negotiation**).
- Send `MCP-Protocol-Version` on each request. Unknown value → the server returns **HTTP 400**. No header → the server assumes 2025-03-26.
- **Reliability toolkit:** resumable streams (Last-Event-ID) · retries with backoff · **idempotency keys** (no double invoice) · timeouts · clear errors (`is_error`) · sessions (Mcp-Session-Id).

**Exam traps**
- ❌ Unknown protocol version, so silently use the newest one → ✅ Return HTTP 400.
- ❌ Retry charge_card after a network glitch and charge twice → ✅ Use an idempotency key.

### T23 · The Built-in Toolset — Server Tools vs. Client Tools

**Example:** Water and electricity come already plumbed in — you just open the tap (server tools). A standard socket has a fixed shape, but you plug in your own appliance and run it in your kitchen (client tools).

**Key facts**
- The key question: **where does the code run?**
- **Server tools** — Anthropic runs them, and the results come back in the same response: **web search, web fetch, code execution** (Python/Bash in a sandbox), **tool search**.
- **Client tools** — Anthropic defines the schema, but **your app runs them**: **bash, text editor, computer use, memory**.
- Pattern: tools that touch **your files or your machine** are client tools, so **you** control what they can reach (least privilege).
- Tool `type` names carry a date version (for example, `text_editor_20250728`). A new version appears when behaviour or schema changes; old versions stay available.
- ➕ On the exam, "built-in tools" can also mean **Claude Code's** Read, Write, Edit, Bash, Grep, Glob. **Grep = search inside files; Glob = find files by name or path** (see the D3-1 summary).

**Exam traps**
- ❌ "bash runs on Anthropic's servers" → ✅ It is a client tool — your app runs it.
- ❌ Build your own sandbox to analyze a CSV → ✅ Use the code execution server tool.

### T24 · Built-in vs. Custom Tools

**Example:** You use the city's electricity (built-in), but you build custom shelves for your own alcove (custom).

**Key facts**
- **Built-in** for common, general needs (web search, fetch a page, run code, edit files): fast, low effort, maintained by Anthropic (server tools), but less control.
- **Custom** for **your own data, systems, and business rules** (your order database, CRM, loyalty discount): full control and an exact fit, but you build and maintain it.
- Quick test: **"Could any company use this tool as-is?"** Yes → built-in. Only us → custom.
- Real systems **mix both**, each tool with a clear boundary (web_search for public info + a scoped, read-only lookup_order for private data).

**Exam traps**
- ❌ Spend two weeks building your own web search → ✅ Use the built-in one.
- ❌ Use web_search to read your private orders → ✅ It can't. Build a scoped custom tool.

### T25 · Stateless vs. Session-Based HTTP Servers

**Example:** A café where the barista remembers your usual order (session-based) vs. one where you show a full written order every time, so any barista can serve you (stateless).

**Key facts**
- **Session-based:** the server remembers you through `Mcp-Session-Id`. Good for continuity (resumable streams, subscriptions, long multi-step work), but harder to scale.
- **Stateless:** each request carries everything it needs, and any instance can serve it. Easy to scale and resilient. **The safe default for production at scale.**
- The danger is **where the session lives**. In one instance's memory + **sticky sessions** = uneven load, and sessions are lost on a crash. If you need sessions at scale, keep them in a **shared store** (for example, Redis).

**Exam traps**
- ❌ In-memory sessions with sticky routing behind a load balancer → ✅ Stateless by default, or keep session state in a shared store.

### T26 · MCP Servers in Claude Code — Config Scopes ⚠️ GAP FILL

*Not from my notes (there is no D2 index to check for missing lectures). Based on the official exam guide's Tool Design & MCP topics and the Claude Code MCP documentation.*

**Example:** House rules pinned on the shared kitchen wall for everyone (project scope), your personal notebook that you carry to every kitchen (user scope), and a sticky note just for you in this one kitchen (local scope).

| Scope | Stored in | Who gets it |
|---|---|---|
| local (default) | `~/.claude.json` (for this project) | Only you, only this project |
| project | `.mcp.json` at the repo root (commit it to git) | The whole team |
| user | `~/.claude.json` (global) | Only you, in all your projects |

**Key facts**
- Team-shared servers → **project `.mcp.json`**. Personal or experimental servers → **user** (or local) scope.
- **Never commit secrets.** Use environment-variable expansion in `.mcp.json`: `${GITHUB_TOKEN}` (or `${VAR:-default}`).
- If the same server name is in several scopes: **local > project > user**.
- Tools from all configured servers are discovered when Claude Code connects at startup, and they are all available together. Check them with `/mcp`.
- Prefer existing **community MCP servers** for standard integrations (Jira, GitHub). Build **custom** servers only for team-specific workflows.
- Expose **content catalogs** (document lists, schemas, issue summaries) as **MCP resources**, so the agent doesn't need exploratory tool calls.
- If the agent keeps using built-in **Grep** instead of a more capable MCP tool → improve the MCP tool's **description** (what it can do and what it returns).

**Exam traps**
- ❌ Put the API token directly in `.mcp.json` → ✅ Use `${TOKEN}` environment-variable expansion.
- ❌ Put a team server in your personal `~/.claude.json` → ✅ Put it in the project `.mcp.json`, so everyone gets it.

---

## Section notes in this folder

- [D2-1 Summary — Tool Design Fundamentals](2.1-tool-design-fundamentals.md)
- [D2-2 Summary — Building MCP Servers (and Clients)](2.2-building-mcp-servers.md)
- [D2-3 Summary — MCP Production & Built-in Tools](2.3-mcp-production-and-built-in-tools.md)

[← Back to home](../README.md) · [Cheat sheet](../cheat-sheet.md)
