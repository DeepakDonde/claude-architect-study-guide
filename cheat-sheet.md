# CCA-F: Exam Cheat Sheet — All 5 Domains

**Domains · weight · my notes** (websites number 2–4 differently — go by the name):
1. Agentic Architecture & Orchestration · **27%** · D1
2. Claude Code Configuration & Workflows · **20%** · D3
3. Prompt Engineering & Structured Output · **20%** · D4
4. Tool Design & MCP Integration · **18%** · D2
5. Context Management & Reliability · **15%** · D5

## 1 · Top 20 Exam Traps

1. ❌ End the loop when Claude's text says "done", or rely on an iteration cap → ✅ Loop while `stop_reason` is `"tool_use"`; stop at `"end_turn"`. A cap is only a safety net. *(D1 L4, L6)*
2. ❌ Enforce a hard rule (refund limit, PII, delete) with a stronger prompt or a bigger model → ✅ Enforce it in **code** (hook, tool check, permission rule). Prompts guide; code enforces. *(D1 L29)*
3. ❌ Escalate because the customer is angry or the model is "unsure" → ✅ Escalate on: a request for a human, a policy gap, no progress, or beyond its authority. *(D1 L31)*
4. ❌ The customer asks for a human, so investigate first → ✅ Hand off immediately. *(D1 L31)*
5. ❌ The subagent knows the parent's history → ✅ It starts fresh; pass all it needs. *(D1 L17)*
6. ❌ Report misses whole topics, so blame the subagents → ✅ Check the coordinator's task split. *(D1 L16)*
7. ❌ Give one agent 18 tools → ✅ 4–5 tools per agent, matched to its role. *(D2 T6)*
8. ❌ Wrong tool picked, so change temperature or tool order → ✅ Fix the descriptions: what · when · when-not · input. *(D2 T2)*
9. ❌ A failed tool returns an empty list → ✅ A structured error (type, cause, retryable, partial results); "found 0" ≠ "failed". *(D5 L118–119)*
10. ❌ Retry every error → ✅ Classify first. Retry only transient errors (backoff + a limit); otherwise fall back, fix, or escalate. *(D1 L25–27)*
11. ❌ Retry until a missing field appears → ✅ Retries can't add data the source doesn't have. *(D4 L103)*
12. ❌ Schema-valid JSON = correct data → ✅ The schema checks shape only; check sums, ranges, and logic in code. *(D4 L102)*
13. ❌ A required field with no "not found" option → ✅ Nullable, or "not_stated" / "unclear" — or Claude invents data. *(D4 L99)*
14. ❌ The same session reviews its own work → ✅ An independent review instance; per-file + cross-file passes. *(D4 L104)*
15. ❌ Team rules in `~/.claude/CLAUDE.md` → ✅ The project `CLAUDE.md` (committed); cross-folder rules → `.claude/rules/` with `paths:`. *(D3 2.1, 2.3)*
16. ❌ Block a command with PostToolUse, or with exit 1 → ✅ PreToolUse + **exit 2**. *(D3 3.7, 4.2)*
17. ❌ Run `claude` in CI without `-p` (it hangs) → ✅ `-p` + `--output-format json` (+ `--json-schema`, `--max-turns`). *(D3 4.5)*
18. ❌ Batch API for a blocking pre-merge check → ✅ A synchronous call. Batch = ~50% cheaper, up to 24 h, overnight jobs. *(D4 Batch)*
19. ❌ Trust summaries to keep exact IDs → ✅ A case-facts block or durable memory. *(D5 L110–111)*
20. ❌ Sources disagree, so silently pick one → ✅ Show the conflict with sources + dates; log it. *(D5 L133)*

### ⚠️ Four corrections to my original notes
1. **Strict mode** does **not** enforce min/max or string length — check them in code. *(D2 T4)*
2. **PostToolUse** can't undo, but **can** add context or **replace** the tool output Claude sees. *(D3 3.7)*
3. **Structured outputs do support** simple regex `pattern`s (no lookahead). *(D4 L98)*
4. **Self-reported confidence** is **not** a valid escalation trigger (a calibrated score is). *(D5 L115)*

## 2 · D1 — Agentic Architecture & Orchestration (27%)

- **Who decides the next step?** Developer (design time) = **workflow**; model (runtime) = **agent**.
- **5 workflow patterns:** prompt chaining · routing · parallelization · orchestrator–workers · evaluator–optimizer (needs "good enough" criteria + a round limit).
- **The loop:** `tool_use` → run the tool → add `tool_result` → call again → stop at `end_turn`. The runtime counts turns.
- **Decomposition:** dependencies decide sequential vs. parallel; **false parallelism** = silent errors.
- **Planning:** replan only when a core assumption breaks. Ambiguity: cheap to undo → assume and note it; irreversible → ask first.
- **Orchestrator:** decompose → assign (success, inputs, format) → aggregate (collect → **validate** → synthesize). The orchestrator decides on errors.
- **Topology:** hub-and-spoke (default; auditable) · pipeline (strict order) · peer-to-peer (rare; never when a full decision log is needed).
- **Handoffs:** task + context + format + constraints. Checkpoint **before** irreversible steps.
- **Multi-agent cost:** chat 1× · agent ~4× · multi-agent ~15× tokens → only for parallel, independent, high-value work.
- **Human review:** pause **before** irreversible steps: context + one decision + deadline.
- **Errors — classify first:** transient (timeout, 503, 429, 408) → retry with backoff + jitter, ~3 tries · permanent (400, 403, 404) → fallback or escalate · reasoning → fix the prompt or context · environment (database or network down) → wait and monitor.

## 3 · D2 — Tool Design & MCP Integration (18%)

- **Tool = name + description + schema.** One tool = one job; enums; describe every parameter.
- **`strict: true`** guarantees the tool name and argument shape (needs `additionalProperties: false`).
- **Errors as data:** `is_error` (API) / `isError` (MCP) — what failed, why, what next. Idempotency keys make retries safe.
- **Least privilege:** few, read-only tools. Huge catalogs → tool search (`defer_loading`).
- **MCP:** host → client (one per server) → server. **Tools** = model decides · **resources** = app decides (read-only) · **prompts** = user picks.
- **Lifecycle:** `initialize` → `notifications/initialized` → work → shutdown.
- **Transports:** stdio = local, logs to **stderr**. Streamable HTTP = remote, OAuth 2.1, `Mcp-Session-Id`, `Last-Event-ID`. HTTP+SSE is deprecated.
- **Client side:** sampling (client's model) · roots (allowed folders) · elicitation (ask the user).
- **Built-in:** server-run (web search, code execution) vs. client-run (bash, text editor, memory).
- **Claude Code MCP:** `.mcp.json` (team) · `~/.claude.json` (you) · secrets as `${TOKEN}`.

| `tool_choice` | Effect |
|---|---|
| `auto` | Claude decides (may reply with text) |
| `any` | Must call some tool |
| `{"type": "tool", "name": …}` | Must call that exact tool |
| `none` | No tools |

## 4 · D3 — Claude Code Configuration & Workflows (20%)

- **Tools:** Grep = inside files · Glob = file names · Edit needs a prior Read + unique match.
- **Context:** `/compact` = same task · `/clear` = new task (CLAUDE.md reloads).
- **Permissions:** default · acceptEdits · plan (read-only) · bypass (isolated only). Rules: **deny → ask → allow**; deny wins.
- **CLAUDE.md:** user (only you) · project (team) · subdirectory (lazy). No sure winner on conflicts → must-hold rules go in settings or hooks.
- **Secrets:** `permissions.deny` with `Read(./.env)`, or a PreToolUse hook. `.claudeignore` is not native.
- **Commands & skills:** `.claude/commands/` = team; skill description = trigger; `context: fork` isolates output.
- **Subagents:** fresh context; scoped tools; never more permissive than the parent; no nesting.
- **Plan mode** for big multi-file work; direct for small fixes; **Explore** subagent for verbose discovery.
- **Iterating:** 2–3 I/O examples · tests first · interview pattern · interacting bugs together.
- **SDK sessions:** continue · resume `<id>` · fork. Stale results → new session + summary.

| Hook event | Can block? | Typical use |
|---|---|---|
| PreToolUse | Yes (exit 2) | Block dangerous actions |
| PostToolUse | No | Lint, log, add context, replace output |
| UserPromptSubmit | Yes | Check or reject a prompt |
| Stop | Yes | "Not done until tests pass" |

*Exit 0 = proceed · exit 2 = block · any other code (including 1) = does not block.*

## 5 · D4 — Prompt Engineering & Structured Output (20%)

- **Be specific:** exact task, format, audience; measurable constraints. For precision, explicit report/skip criteria — not "be conservative".
- **System prompt** = top-level `system` parameter. XML tags separate instructions from data; long documents first, task last.
- **Few-shot:** 3–5 consistent, balanced examples; for ambiguous cases, 2–4 that show *why*.
- **CoT** = asked in the prompt; **extended thinking** = API setting (`budget_tokens` ≥ 1,024, < `max_tokens`).
- **Prefill** of the last assistant turn → 400 on Claude 4.6+. Stateless API: resend history; end on a user turn.
- **Sampling:** temperature 0–1 (default 1); change temperature **or** top_p, not both.
- **Structured outputs:** JSON outputs (`output_config.format`) or strict tool use; `additionalProperties: false`.
- **Validation loop:** retry with the specific error; max retries + a fallback (human for high stakes).
- **Evals:** typical + edge + known-failure cases; grade by code, LLM judge, or human; re-run after changes.
- **Batch API:** ~50% cheaper, up to 24 h, no tool loop inside a request; `custom_id`; resubmit only failures.

## 6 · D5 — Context Management & Reliability (15%)

- **Context window** = one conversation's working memory; output counts too. Key facts at the **start or end**.
- **Memory tool** is client-side: your app stores files under `/memories` and blocks path traversal.
- **Tool-output bloat** → filter · summarize · reference (store it, keep a pointer).
- **Errors:** fix at the lowest level that can; send the rest up with partial results. Wrap **every** tool.
- **Notes:** active context → scratchpad → durable memory; prune and overwrite.
- **Delegate discovery** (3+ files) with a precise question and a return format.
- **Compaction** is lossy — never for code review or legal text; not the same as context editing.
- **Confidence:** per-field scores are self-reports → calibrate on a labeled set.
- **Review sampling:** stratify by confidence, category, stakes; 97% overall can hide a weak type.
- **Provenance:** every claim → its sources; inline citations for high stakes; verify they exist.

## 7 · Memory Hooks

"Who decides the next step?" · "Prompts guide; code enforces." · "Classify first, then recover." · "Pre-tool guards; post-tool reacts." · "Tools = model · resources = app · prompts = user." · "Valid ≠ correct." · "Summarize the story; keep the facts." · "Found 0 ≠ failed."
