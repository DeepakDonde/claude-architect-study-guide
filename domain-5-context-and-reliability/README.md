# CCA-F: D5 Domain Summary — Context Management & Reliability

**Exam weight:** 15%. **Numbering:** course D5 = official blueprint Domain 5 (same name).

**Course coverage:** Sections 16–19 · 26 lectures + 4 quizzes (about 2 hr 57 min of video). Every content lecture is in my notes, so no gap fills were needed. L131 is a 1-minute check-in; quizzes 5A–5D are not included.

**Legend:** ❌ wrong idea (exam trap) → ✅ right idea · ⚠️ GAP FILL = not from my notes · ➕ = exam-guide or docs check (added)

| Part | Course section | Lectures | Gap fills |
|---|---|---|---|
| 1 · The Context Window | Section 16 | L108–114 | — |
| 2 · Multi-Agent Failure Modes | Section 17 | L115–120 | — |
| 3 · Cross-Session Knowledge & Coherence | Section 18 | L121–126 | — |
| 4 · Confidence, Sampling & Provenance | Section 19 | L127–133 (L131 = check-in) | — |


---

## Part 1 · The Context Window (Section 16 · L108–114)

*From the D5-1 summary.*

### L108 · How the Context Window Works

**Example:** A whiteboard in a meeting room. Everyone writes on one fixed-size board. When it is full, old writing must be wiped off. When the meeting ends, the board is cleaned for the next meeting.

**Key facts**
- The **context window** = the model's working memory for **one** conversation. It holds the system prompt, all turns, documents, and tool outputs. If something is not in the window, the model cannot use it. A new conversation starts empty.
- A token ≈ 4 characters ≈ ¾ of a word (1,000 tokens ≈ 750 words).
- The window size depends on the model (current large models go up to 1M tokens; some smaller ones have 200K). Check the models page — older course material often says "200K" for every model.
- **Output tokens count too.** If input + output won't fit, the API does not silently cut text — you get an error or a `model_context_window_exceeded` stop.
- **Bigger is not automatically better:** more tokens = more cost, slower replies, and the **lost-in-the-middle** problem.

**Exam traps**
- ❌ "The window only limits the input" → ✅ Output counts too.
- ❌ "Fill the 1M window to get better answers" → ✅ A clean, focused context usually works better.

### L109 · Context Window Economics: Cost, Speed, Quality

**Example:** A lawyer who charges per page, reads slower the more pages you hand over, and is more likely to miss the key clause in a 500-page dump. So you give a tight brief with the important parts first.

**Key facts**
- More context costs you three ways: **money** (you pay per token), **speed** (slower first token), and **quality** (key details missed, noise distracts).
- **Every turn re-sends the whole history**, so a long chat costs more with each turn.
- The **usable** context is smaller than the advertised window size.
- **Lost in the middle:** models pay most attention to the **start and end** of a long context. Put key information there.
- Best practice: **curated retrieval** — put in what is relevant, not everything.
- Prompt caching lowers cost (cache reads cost about 10% of the normal input price). Prices change — check the current pricing page.

**Exam traps**
- ❌ A critical instruction buried in the middle of a huge context → ✅ Move it to the start or end, and trim the noise.
- ❌ Send the whole 800K-token knowledge base with every request "to be safe" → ✅ Retrieve only the relevant slice.

### L110 · Why Summaries Lose Facts

**Example:** The minutes of a two-hour meeting say "the team approved the budget" — but not the exact number. Weeks later, nobody knows the figure.

**Key facts**
- A summary is **lossy compression**: it keeps the **gist** and drops the **specifics**.
- Most at risk: **exact numbers, IDs, prices, dates, names, addresses, config values, edge cases**. Paraphrasing can also change the meaning.
- The summarizer cannot know which detail you will need later.
- A fact that is summarized away is **gone from the window**. The model then says it doesn't know — or worse, it **guesses** (hallucination).
- Rule: **summarize the story; keep exact facts word for word.**

**Exam traps**
- ❌ Fix lost details with longer summaries or a bigger model → ✅ Keep critical facts word for word in a fact block (L111).
- ❌ "Claude ignored the value on purpose" → ✅ The value was summarized away, so Claude guessed.

### L111 · Designing a Persistent Fact Block

**Example:** A sticky note on your monitor with the exact account number and the deadline. The papers on your desk get shuffled and filed away; the sticky note stays put.

**Key facts**
- A **persistent fact block** = a small section with **critical facts word for word**, present **every turn**, and **never summarized or trimmed**. It is the fix for L110.
- ➕ The exam guide calls this a **"case facts" block**: amounts, dates, order numbers, and statuses kept outside the summarized history.
- Put in: exact IDs, order numbers, names, addresses, dates, key decisions, hard constraints, config values. Leave out: small talk, general chat, Claude's reasoning.
- Keep it **small and high-signal** (it is re-sent every turn), **word for word**, **structured** (`Order ID: 88231`), **current** (remove stale facts), and at the **start or end** of the context.
- A place compaction can't touch: the **system prompt** or **CLAUDE.md** (in Claude Code, /compact keeps them).

**Exam traps**
- ❌ A 40,000-token fact block full of "maybe useful" notes → ✅ Trim it to the must-keep facts.

### L112 · The Memory Tool for Persistence

**Example:** A doctor cannot memorize every patient, so each patient has a file in a cabinet. The doctor pulls your file when you visit, adds notes, and files it back.

**Key facts**
- The **memory tool** lets Claude keep **files outside the context window** that survive between conversations. Claude reads them **just in time**, only when needed.
- It is **client-side**: Claude only **requests** file operations; **your app** runs them, on storage **you** control.
- Setup: one tools entry, `{"type": "memory_20250818", "name": "memory"}`. All files live under **`/memories`**.
- Six commands: `view`, `create`, `str_replace`, `insert`, `delete`, `rename`.
- The API automatically adds a protocol: check memory first, save progress as you go, and "assume interruption".
- Security is **your** job: block **path traversal** (`/memories/../../`), strip sensitive data, cap file sizes.
- It pairs with **context editing** (clears old tool results) and **compaction** (summarizes): compaction keeps the window small; memory keeps what must survive.
- It is not the same as the "memory" feature in the Claude.ai app.

**Exam traps**
- ❌ "Anthropic stores the memory files" → ✅ Your app stores them.
- ❌ Run any path Claude requests → ✅ Check that the path stays inside /memories.

### L113 · The Tool-Output Bloat Problem

**Example:** You ask an assistant for today's temperature, and they drop a 500-page weather report on your desk — ten times a day.

**Key facts**
- Tool results come **back into the context window** and count as tokens.
- Raw results (full JSON, long logs, whole documents, 50 search articles) are **big and mostly irrelevant**.
- They **stay in the history and are re-sent every turn**, so cost and window use keep growing.
- Symptoms: higher cost, slower replies, and early instructions getting buried or forgotten.
- Fixes: filter / summarize / reference (L114), plus **context editing** (clears stale tool results).

**Exam traps**
- ❌ An agent forgets its early instructions after many tool calls, so blame the model → ✅ Tool output buried them. Trim or clear old results.

### L114 · Trim Strategies: Filter, Summarize, Reference

**Example:** For the 500-page weather report, you can highlight the one line you need (filter), write a one-paragraph summary (summarize), or file the report in a cabinet and hand over a note saying "Report A-12 is in the cabinet" (reference).

| Strategy | What stays in the window | Best when | Watch out |
|---|---|---|---|
| Filter | Only the needed fields | You know exactly what you need | May drop a field you need later |
| Summarize | A short gist | Long text, and the gist is enough | Lossy — drops exact values |
| Reference | A pointer or ID | Big data you may need in full later | You must be able to fetch it back |

**Key facts**
- Known fields → **filter** (the cheapest and most precise). The meaning of a long text → **summarize**. Big data you may need in full later → **reference** (store it outside, fetch it on demand).
- You can combine them (for example, filter + reference).
- Context editing can clear stale tool results automatically.

**Exam traps**
- ❌ Summarize a contract you must quote exactly later → ✅ Reference it.
- ❌ Summarize a weather JSON when you need one field → ✅ Filter it.


---

## Part 2 · Multi-Agent Failure Modes (Section 17 · L115–120)

*From the D5-2 summary.*

### L115 · When to Escalate — Trigger Design

**Example:** A junior help-desk rep has written rules: refunds over a set amount → the supervisor; legal questions → the legal team; tried 3 fixes with no luck → escalate; closing an account → get approval first.

**Key facts**
- **Escalation** = the agent hands off to a human (or a stronger system). A **trigger** = the exact condition that starts it.
- Two ways to fail: **too little** escalation (confident wrong actions on high-stakes tasks) vs. **too much** (humans do everything, so the automation is useless).
- Good triggers are **specific and measurable**: "refund over $500", "the same question asked 3 times without a fix", "the action permanently deletes data".
- Common trigger types: high risk or irreversible · ambiguity · repeated failure · out of scope or no permission.
- High-impact or irreversible actions (delete, pay, send) → require human approval by default. Enforce critical triggers **in code**.
- ➕ Exam guide: the model's **self-reported confidence** ("escalate if unsure") and **negative sentiment** are **unreliable** triggers — the model can be confidently wrong, and frustration is not the same as difficulty. Valid triggers: an explicit request for a human, a policy gap, no progress, beyond the agent's authority. A **calibrated** score (L128) can route work to review. (See D1 L31.)

**Exam traps**
- ❌ "Escalate if you feel unsure" → ✅ Specific, measurable conditions.
- ❌ Politely ask the agent in the prompt to be careful with big refunds → ✅ A hard-coded rule that blocks refunds over $500 and sends them to a human.
- ❌ List only the permitted actions, with no default rule → ✅ Block or escalate anything not explicitly permitted ("fail closed"), or a banned action slips through.

### L116 · The Handoff Payload

**Example:** Before you go on leave, you write a handover note: the goal, what is done, what is left, where the files are, and warnings ("don't touch the client folder, it is locked").

**Key facts**
- The **handoff payload** = the package passed to the next actor (an agent or a human) so they can continue **without guessing or redoing work**.
- Include: **the goal · what is done · what is left and the next step · needed inputs (IDs, files, links) · constraints and warnings · the expected output format**.
- **Just enough:** too little → the next actor guesses; too much (the whole raw transcript) → noise, wasted tokens, buried key points.
- Use the same fields every time (a **handoff schema**).
- If the next actor is a **human**, write in clear language, not internal codes.
- Anthropic's lesson from its own research system: vague subagent instructions caused misunderstood tasks and **duplicate searches**. The fix was a clear objective, an output format, and task boundaries.

**Exam traps**
- ❌ "Continue." as the whole handoff → ✅ Goal + progress + remaining work + warnings.
- ❌ Pass the full raw transcript "to be safe" → ✅ A trimmed summary of what the next actor needs.

### L117 · Where to Catch the Error: Local or Up the Stack

**Example:** A worker refills the printer paper themselves (local). A big client threatening to cancel goes to the manager (up the stack). Hiding the angry client is wrong — and so is running to the manager about printer paper.

**Key facts**
- Rule: handle an error at the **lowest level that has enough information and authority** to fix it — and no lower.
- **Locally:** temporary errors (retry), the wrong tool (use another one), a fixable bad input.
- **Up the stack** (the orchestrator or a human): it needs approval, data is missing, it is out of scope, or it still fails after retries.
- **Never swallow errors** (that creates a silent failure). **Don't over-escalate** (it floods humans with small problems).

**Exam traps**
- ❌ A subagent returns an empty result when its lookup fails → ✅ Report the failure.
- ❌ Send every retryable timeout to a human → ✅ Retry locally; escalate only what you can't fix.
- ❌ A $5,000 refund is over the limit, so quietly lower the amount → ✅ Send it up to a human who has the authority.

### L118 · Designing the Propagation Contract

**Example:** A company's standard incident-report form with fixed boxes: what happened, how serious it is, what you already tried, and whether it is still happening. Every manager knows where to look.

**Key facts**
- A **propagation contract** = one agreed format for reporting errors through the system (the error version of a handoff schema).
- Fields: **status** (ok / error) · **error type** (tool / reasoning / environment) · **cause** · **what was attempted** · **retryable?** · **severity**.
- The error type lets the receiver choose the right fix: retry a tool error, re-plan a reasoning error, wait or escalate an environment error.
- A clear `status: error` stops a real failure from looking like a normal empty result.
- ➕ Exam guide: also return any **partial results** and **possible alternatives**. Both **silently hiding** an error and **stopping the whole workflow** because one subagent failed are anti-patterns. In the final synthesis, add **coverage notes**: which findings are well supported, and which topics have gaps because a source was unavailable.

**Exam traps**
- ❌ Each subagent reports errors in its own style → ✅ One shared propagation contract.
- ❌ One subagent fails, so stop everything → ✅ Continue with partial results and report the gap.

### L119 · The Silent Bug: When "Nothing" Is Ambiguous

**Example:** Your phone says "0 new messages". Did nobody write to you, or did the app fail to sync? You can't tell just by looking.

**Key facts**
- A **silent bug** = a failure with **no error message**. It returns empty, null, or zero output that **looks normal**.
- "Nothing" is **ambiguous**: "searched and found zero" and "something broke" look exactly the same.
- Silent failures are **worse than loud errors**: there is no warning, so wrong results spread.
- Fix: **make emptiness explicit** — `{"status": "success", "count": 0}` vs. `{"status": "error", "cause": …}`. Trust an empty result only when status = success.
- Sanity-check suspicious emptiness, and have the agent report **what it did** ("searched the 2025 orders table; 0 matched").

**Exam traps**
- ❌ Treat an empty list as success by default → ✅ Separate "found 0" from "failed".

### L120 · Disambiguation Patterns and Tool-Wrapper Discipline

**Example:** A quality-control and packaging station after every factory machine. Every box gets a clear label: GOOD — 5 parts / EMPTY — 0 parts, machine ran fine / FAILED — machine jammed.

**Key facts**
- **Disambiguation:** never let one value (empty, null, or zero) mean several things. Separate at least **found something · found nothing (successfully) · failed** — sometimes also **not run**.
- A **tool wrapper** = code around a tool that runs it, turns raw errors into the propagation contract format, makes emptiness explicit, and returns **one standard shape** every time.
- **Wrapper discipline:** wrap **every** tool. Reliability is only as strong as the **weakest unwrapped** tool.
- On the input side, for an ambiguous **request** ("cancel my subscription" when there are two), **ask a clarifying question** or use a documented default — never guess.

**Exam traps**
- ❌ Skip wrappers on "simple" tools → ✅ Wrap every tool; one gap lets silent bugs back in.
- ❌ Cancel both subscriptions "to be safe" → ✅ Ask which one.


---

## Part 3 · Cross-Session Knowledge & Coherence (Section 18 · L121–126)

*From the D5-3 summary.*

### L121 · The Scratchpad Pattern

**Example:** Rough-work paper in a maths exam. You work through the steps first, then copy the clean answer onto the answer sheet.

| Type | What it is | Use it when |
|---|---|---|
| Manual scratchpad | `<scratchpad>` for rough work + `<answer>` for the final answer | You want visible, parseable reasoning, or extended thinking is off |
| Extended thinking | Built-in reasoning **before** the response (an API setting) | Deep reasoning; no need to add "think step by step" |
| "Think" tool | A tool Claude calls **during** a task, after tool results | Agents that must check rules before the next action |

**Key facts**
- A scratchpad = working space for rough reasoning, kept **separate from the final answer**.
- Extended thinking happens **before** the response; the think tool works **during** the task (after tool results). Before vs. during.
- Agents can use **temporary files** as a scratchpad, which keeps working notes out of the main context (clean them up afterwards).
- Trade-off: more room to reason = fewer errors, but more tokens and more context used.

**Exam traps**
- ❌ Add "think step by step" when extended thinking is already on → ✅ Usually unnecessary.

### L122 · Designing a Healthy Scratchpad

**Example:** An office: the papers on your desk (active context), the whiteboard on the wall (this session's scratchpad), and the filing cabinet (durable memory). A whiteboard that is never wiped becomes unreadable.

**Key facts**
- Three tiers: **active context** (now) → **scratchpad** (this session) → **durable memory** (across sessions). Put each fact in the right tier (for example, a user's long-term rule → durable memory).
- **Context rot:** recall gets worse as tokens grow, **even before** the limit.
- Hygiene: keep notes **outside** the main context and pull back only what you need · use **structured** sections · **prune** stale notes · **overwrite** instead of endlessly appending.
- Failure modes: too big (rot, cost, lost in the middle) · stale (acting on old information) · unstructured (can't find things) · carrying too much forward (almost as bad as forgetting).
- ➕ Exam guide (long codebase exploration): when answers start drifting to "typical patterns" instead of the specific classes found earlier, that is context degradation. Keep a **scratchpad file of key findings** and refer back to it, and **summarize each phase** before starting the next. For crash recovery, each agent **exports its state** to a known location, and the coordinator **loads a manifest** when it resumes.

**Exam traps**
- ❌ "Carrying extra notes forward is harmless" → ✅ It adds cost and context rot.

### L123 · Why Delegate Discovery to a Subagent

**Example:** A manager needs one fact from a huge archive. Instead of hauling 20 boxes onto a clean desk, they send an assistant. The assistant digs through the boxes in the archive room and comes back with one index card.

**Key facts**
- **Discovery** (finding the right files or docs in a big codebase) is **verbose**: many searches, dead ends, and irrelevant files.
- **Delegate** it to a subagent: the mess stays in the subagent's **disposable context**, and the parent receives only a **focused summary**.
- Four steps: frame a **precise question** + the answer shape → a subagent with **minimal tools** → a **structured summary** → the parent continues.
- **Return contract** = the answer shape, defined up front (otherwise the subagent pollutes the parent again).
- **Trust boundary** = treat the subagent's answer as a **hypothesis**; verify critical findings.
- Worth it for multi-file surveys, long documents, and multi-step lookups (rule of thumb: about **3 or more files** read in a row). Overkill when you already know the exact file.
- Cost: more time and tokens (multi-agent can use about 15× a normal chat), in exchange for a clean main context.

**Exam traps**
- ❌ Read 400 files into the main context "to be thorough" → ✅ Delegate the discovery.
- ❌ Start a subagent to read one known file → ✅ Do it in the main session.

### L124 · Delegation Mechanics: Prompts, Returns, Failure Modes

**Example:** Briefing the archive assistant: "Bring me the file path and the function that checks the login token. Look only in the auth room. Write it on one index card. Come back in 15 minutes with whatever you have."

**Key facts**
- A subagent prompt has four parts: **question** (one concrete ask) · **scope** (where to look, what to skip) · **return shape** · **stop condition**.
- Return contract: **structured** (file path, line number, short summary) + a **length cap**.
- Tools: **read and search only** by default — no write, delete, or network tools.
- Trust boundary: act directly on low-stakes answers; **spot-check** before critical changes.
- Three failure modes — **wall of text**, **scope creep**, **blind trust** — all caused by a **weak specification**.

**Exam traps**
- ❌ "Investigate the caching layer and tell me about it" → ✅ A precise question + return shape + length cap.
- ❌ Give a discovery subagent write access → ✅ Read and search only.

### L125 · Long-Conversation Coherence, Compaction, Session Memory

**Example:** An all-day meeting with one notepad. Two different problems: the notepad runs out of pages (window pressure), and people forget what was said mid-morning even when pages are left (lost in the middle).

| | Lost in the middle | Context window pressure |
|---|---|---|
| What it is | Less attention to the middle | The content no longer fits |
| When it happens | Even when everything fits | When you go over the limit |
| Fix | Move or prune; key facts at the edges | Compaction, external memory |

**Key facts**
- Don't use compaction to fix an **attention** problem.
- **Compaction** = summarize earlier turns. It is **lossy by design**. Strategies: full summary · selective (keep decisions and outcomes) · sliding window (last N turns word for word + a summary header).
- **Don't compact** where exact wording matters (code review, legal and compliance).
- Memory can be **in-context** (simple, capped) · **external** (scales, needs retrieval) · **hybrid** (the production default: a summary header + the last N turns word for word + the summary saved between sessions).
- Claude gets **context-awareness** signals automatically (token budget, usage warnings). They are a **signal, not a guarantee** — still enforce a budget in code.
- Coherence tools: a **session header**, a **JSON state object** (goals, facts, decisions), and **goal anchoring** (restate the original goal every turn — the simplest defence against scope creep).
- Compaction (and switching models) **invalidates the prompt cache**, which can cause a cost spike.
- Current practice: **server-side compaction** is the recommended primary strategy; client-side SDK compaction is being deprecated.

**Exam traps**
- ❌ A fact from 40 turns ago is ignored even though everything fits, so compact → ✅ It is lost in the middle. Move or prune.
- ❌ Compact a legal conversation → ✅ Keep it word for word.

### L126 · Server-Side Compaction

**Example:** The conference venue provides an automatic stenographer who condenses the old pages when your notepad is full. It saves you work, but you don't control what it keeps — so you write the truly critical things in your own private notebook.

**Key facts**
- **Server-side compaction:** when the conversation reaches a **trigger threshold**, the platform summarizes the older history into a compact block and continues from there.
- It is **opt-in** (a beta header + a context-management setting). Default trigger ≈ **150K input tokens**; minimum **50K** (lower gives an API error). Header and parameter names change — check the docs.
- By default, recent turns are **not** kept word for word (that is your client-side job).
- The trade: **less to build, less control, and it still costs a step**. You can steer the summary with an **instructions** parameter, and keep must-haves in the system prompt or recent messages.
- Summaries are **lossy** → **never treat the transcript as your only copy**. Keep durable facts in a file or database (or the memory tool), and re-read them.
- **Compaction ≠ context editing:** compaction **summarizes** history; context editing **clears old tool results** (and is cache-friendly).

**Exam traps**
- ❌ A deployment ID from 200 turns ago disappeared after compaction, so "it's a model bug" → ✅ The summary dropped it. Keep critical IDs in durable state.
- ❌ "You have zero control over the summary" → ✅ Use the instructions parameter, and keep must-haves outside the summarized part.


---

## Part 4 · Confidence, Sampling & Provenance (Section 19 · L127–133)

*From the D5-4 summary.*

### L127 · Per-Field Confidence

**Example:** A school report card. One overall grade, "B+ (85%)", hides a failing maths mark (40%). A grade for each subject shows exactly which subject needs a tutor.

**Key facts**
- **Per-field confidence** = a score for **each field**, not one score for the whole record — so you send **only the weak field** to a human.
- Claude has **no built-in confidence meter**. A score is a **self-report** written because you asked for it. It means nothing until it is **calibrated** (L128).
- You get it through **schema design**: each field has a `value` and a `confidence` (0–1).
- Threshold: at or above → auto-accept; below → the review queue (an optional borderline zone in between).
- **Per-field routing:** only weak fields go to review (efficient, but downstream systems must accept partly reviewed records). **Per-record routing:** any weak field sends the whole record (simpler to audit; preferred where compliance is strict).
- The main failure: trusting an **uncalibrated** score (it says 0.90, but it is right only 60% of the time).

**Exam traps**
- ❌ A record score of 0.85 auto-accepts a due date scored 0.41 → ✅ Use per-field scores.
- ❌ "Turn on a confidence parameter in the API" → ✅ Add value + confidence keys to the output schema.

### L128 · Calibrating with a Validation Set

**Example:** A student writes "90% sure" next to some answers. You grade the test: if only 60% of those answers are right, the student is overconfident, and you can't trust their number.

**Key facts**
- **Calibration** = checking whether the stated confidence matches the **real** accuracy.
- **Validation set** = labeled examples with correct answers. It must be **representative** of the real work and **kept separate** (the model never learns from it).
- Group outputs by stated confidence and measure the real accuracy of each group. **Calibrated:** says 0.90, right 90% of the time. **Overconfident:** says 0.90, right 60% (dangerous — errors slip through). **Underconfident:** says 0.60, right 90% (wasteful).
- **Reliability diagram:** stated confidence vs. real accuracy. The diagonal line = honest; sagging below it = overconfident.
- Set the threshold from the **measured** accuracy, not from the model's own number.
- **Recalibrate** after any model upgrade or prompt change, using the same fixed validation set.

**Exam traps**
- ❌ Set the auto-accept threshold from the model's self-reported number → ✅ Set it from measured accuracy.

### L129 · Stratified Sampling

**Example:** Airport security. Searching every 20th traveller finds almost nothing, so the staff stop looking carefully. Searching flagged travellers, unusual routes, and high-security flights keeps every search meaningful.

**Key facts**
- **Random sampling** mirrors the model's accuracy. With a 95%-accurate model, reviewers mostly confirm correct outputs, and the few errors are too scattered to show patterns.
- **Stratified sampling** takes more of the risky outputs. Three dimensions: **confidence** (below the threshold → sampled more; above it → still sampled, but rarely) · **category** (**per-category quotas**, so rare categories still get reviewed — this also catches drift) · **stakes** (high stakes → human review **no matter the confidence**).
- Combine them with a scoring function, so reviewers work where an error's **expected cost** is highest.
- **Reviewer burnout** is a **design failure**: a queue full of correct outputs leads to rubber-stamping. Stratification raises the share of real errors in the queue.
- ➕ Exam guide: a high **overall accuracy** (for example, 97%) can **hide** a weak document type or field. Check accuracy **per document type and per field** before reducing human review. Keep sampling **high-confidence** outputs too, to measure their real error rate and catch **new kinds of errors**.

**Exam traps**
- ❌ A rare category with a 20% error rate barely shows up in random review → ✅ Per-category quotas.
- ❌ A high-stakes decision at 0.96 confidence is auto-accepted → ✅ Human review, no matter the confidence.

### L130 · Routing Low-Confidence Output Through a Review Queue

**Example:** A hospital lab. Machines send clean results straight to the doctor. Odd results go onto a pathologist's worklist (strangest first), and the pathologist approves, edits, or rejects each one.

**Key facts**
- The **review queue** sits **between the model and its consumer**. A routing layer sends below-threshold or high-stakes items to the queue; everything else goes straight downstream.
- Priority order: lowest confidence and highest stakes on top. Track **queue depth** (how many wait) and **queue latency** (how long they wait) from day one.
- **One screen:** input + output + confidence (+ any suggested fix). Context first, no tab-switching.
- Exactly **three actions**, all logged with time and reviewer: **approve** (released) · **edit** (fixed and released; the original + correction pair is the **best training data**) · **reject** (nothing released; the input type is flagged).
- **Synchronous** review = the consumer waits (needed when a placeholder is not acceptable, like a medical sign-off). **Asynchronous** = a placeholder now, the real output later.
- **Bypass** only by policy (an emergency, an empirically validated high-confidence floor, or no reviewer available with a warning flag) — and **every bypass is logged**.

**Exam traps**
- ❌ Let "high-confidence" items skip the queue using a round-number threshold, with no logs → ✅ An empirically validated floor + log every bypass.

### L132 · Claim-to-Source Mappings

**Example:** A good student essay has a footnote after each fact naming the exact book. A teacher can pick any sentence, open that book, and check it.

**Key facts**
- **Provenance tracking:** attach a verifiable source to **every claim**. Fluent writing is not proof.
- **Claim-to-source mapping** (one per claim): **claim text · list of source IDs · optional confidence per source**. Each claim becomes a data object you can query, audit, and fix.
- **Inline citations** = accountability per claim → required for high stakes (regulatory, clinical, legal). A list at the end of a paragraph = paragraph-level only.
- Benefits: trust · auditability · compliance · fixing one sentence without re-running everything.
- Verify by **risk-weighted sampling**: pick a claim → follow its source → read the passage → judge.
- **Fabricated citations** (real-looking, but wrong or non-existent) are a top danger → verify citations before finalizing. Source IDs must be **stable** (they must not break when a document moves).
- ➕ Exam guide: subagents must output claim-source mappings, and the synthesis step must **keep and merge** them — attribution is often **lost during summarization**.

**Exam traps**
- ❌ A fluent 10-source report with no per-sentence sourcing → ✅ Inline claim-to-source mappings.
- ❌ Trust a citation because it looks credible → ✅ Check that it exists and really supports the claim.

### L133 · Conflict Annotation

**Example:** Two witnesses describe an accident differently. A bad report writes one version as fact. A good report says: "Witness A said the light was green; Witness B said red; this was not resolved."

**Key facts**
- **Conflict annotation** = showing disagreements in the output instead of hiding them.
- **Averaging anti-pattern:** silently blending conflicting sources, or picking one without logging it → false certainty that sounds confident.
- **Detect before writing:** semantic similarity (same topic) + **entailment checking** (contradicts / supports / unrelated). Same topic + contradiction = a conflict.
- Resolution rules: **prefer newer** (facts that change: regulations, versions, prices) · **prefer authoritative** (needs a predefined source ranking) · **escalate to a human** (high stakes, no clean rule).
- **Log every conflict**, even resolved ones (the claims, source IDs, rule used, and what was chosen vs. discarded). The log is part of the audit trail.
- Show it according to the stakes: high → **inline**; medium → a footnote or expandable section; full review → a conflict summary at the end.
- ➕ Exam guide: require **publication or data-collection dates** in subagent outputs — two numbers from different years may **both** be right (not a real conflict). Keep both values, label them with their sources, and let the coordinator decide.

**Exam traps**
- ❌ Silently pick one date and write it as fact → ✅ Annotate the conflict and log the choice.
- ❌ Put a high-stakes clinical conflict only in an end summary → ✅ Inline, next to the claim.

---

## Section notes in this folder

- [D5-1 Summary — The Context Window](5.1-the-context-window.md)
- [D5-2 Summary — Multi-Agent Failure Modes](5.2-multi-agent-failure-modes.md)
- [D5-3 Summary — Cross-Session Knowledge & Coherence](5.3-cross-session-knowledge-and-coherence.md)
- [D5-4 Summary — Confidence, Sampling & Provenance](5.4-confidence-sampling-and-provenance.md)

[← Back to home](../README.md) · [Cheat sheet](../cheat-sheet.md)
