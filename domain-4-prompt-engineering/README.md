# CCA-F: D4 Domain Summary — Prompt Engineering & Structured Output

**Exam weight:** 20%. **Numbering:** course D4 = official blueprint Domain 3 (Prompt Engineering & Structured Output). Some study websites call it Domain 4 — always go by the domain name.

**Coverage:** my three D4 note files — Lectures 84–104, plus two unnumbered evaluation lectures after L104 (probably L105–106), labeled Eval 1 and Eval 2. One exam-guide topic missing from my notes was added as a gap fill (Batch).

**Correction to my notes:** structured outputs **do** support simple regex `pattern`s (with limits). Numeric ranges and string lengths are **not** enforced (see L98).

**Legend:** ❌ wrong idea (exam trap) → ✅ right idea · ⚠️ GAP FILL = not from my notes · ➕ = exam-guide or docs check (added)

| Part | Source notes | Lectures | Gap fills |
|---|---|---|---|
| 1 · Effective Prompts | D4-1 | L84–89 | — |
| 2 · Advanced Prompting Techniques | D4-2 | L90–97 | — |
| 3 · Structured Output & Evaluation | D4-3 | L98–104, Eval 1–2 | Batch ⚠️ |


---

## Part 1 · Effective Prompts (L84–89)

*From the D4-1 summary.*

### L84 · Clarity, Specificity, and the Cost of Ambiguity

**Example:** "Give me something to eat" could get you anything. "One medium chicken biryani, no onions, raita on the side" gets exactly that. Saying why ("for a guest who can't eat spicy food") helps even more.

**Key facts**
- **Specificity is the #1 quality lever.** A vague prompt gives vague, inconsistent output.
- State the **exact task** (not just "analyze"), the **output format** (length, structure, tone), and the **audience and purpose** (who it is for, and why).
- Prefer **positive** instructions ("write formally") over a stack of negatives ("don't be casual").
- Break complex tasks into **numbered steps**.
- Golden rule: show the prompt to a colleague who has no context. If they would be confused, Claude will be too.
- ➕ Exam guide (precision): vague instructions like **"be conservative"** or **"only report high-confidence issues"** do not improve precision. Write **explicit criteria** for what to report and what to skip (for example, report bugs and security issues; skip minor style), with **examples for each severity level**. If one category gives many false positives, **turn it off for now** to restore trust while you fix the prompt.

**Exam traps**
- ❌ "Analyze this feedback" gives a summary, sentiment, or themes at random, so raise the temperature or use a bigger model → ✅ Say exactly which analysis you want, and in what format.
- ❌ Add more "don't" rules → ✅ Rewrite them as positive, measurable instructions.

### L85 · Instruction Decomposition and Constraint Design

**Example:** Directions for a driver. The turns are the decomposition ("straight 2 km, then left at the temple"). The driving rules are the constraints ("under 40 km/h, no highway").

**Key facts**
- **Decomposition** = ordered, single-purpose steps. It controls the **process** and improves completeness and consistency.
- **Constraints** = rules the output must obey. They control the **result**: format, length, what to include, what to exclude, tone, scope.
- Good constraints are **positive and measurable** ("under 50 words", not "short").
- Put a constraint next to the step it controls, or repeat a critical one at the end.
- **Avoid conflicting** constraints ("extremely detailed" + "exactly one sentence") and **over-constraining** (rules get dropped, or the output becomes rigid).

**Exam traps**
- ❌ "Be complete but very brief" → ✅ One measurable limit: "no more than 60 words".
- ❌ "Review this résumé" gives jumpy, partial output, so add more examples → ✅ Decompose it into ordered steps.

### L86 · The System Prompt

**Example:** A job description ("You are a support agent for a bank; always be polite; never share account numbers") vs. today's task ("Help this customer reset their PIN").

| | System prompt | User message |
|---|---|---|
| Purpose | Role, rules, stable context | The task right now |
| Lifespan | Every turn | This turn only |
| Changes | Rarely | Every turn |

**Key facts**
- In the Messages API, the system prompt is the **top-level `system` parameter** — **not** a `{"role": "system"}` item inside `messages`.
- Three jobs: **role** · **rules and guardrails** · **stable context** (policies, style guide, product facts).
- Put lasting instructions in the system prompt. Don't repeat them in every turn.

**Exam traps**
- ❌ Put `{"role": "system", ...}` as the first message → ✅ Use the separate `system` parameter.
- ❌ A chatbot drifts off-topic, so resend the rules every turn → ✅ Define the role, tone, and an off-topic rule in the system prompt.

### L87 · Role, Persona, and Context Setting

**Example:** Casting an actor. "Play a doctor" is the role. "A warm, senior pediatrician who explains things simply to worried parents" is the persona. "A busy clinic during flu season" is the context.

**Key facts**
- **Role** = who Claude is. **Persona** = the role plus expertise level, audience, and tone. **Context** = the background facts that apply.
- Role prompting improves **domain accuracy** and keeps **tone and vocabulary** consistent.
- Match the persona to the **audience**, not only the topic ("explain to a new patient" vs. "explain to an expert").
- A specific persona beats a generic one ("senior pediatrician" > "medical person"). These usually live in the system prompt.

**Exam traps**
- ❌ Answers are correct but too technical for beginners, so lower the temperature → ✅ Add a persona and audience ("explain to readers with no technical background").

### L88 · Using XML Tags to Structure Prompts

**Example:** Labeled folders — "Instructions", "Data", "Examples" — instead of one messy pile of papers.

**Key facts**
- XML tags (`<instructions>`, `<document>`, `<examples>`) **separate instructions from data**, so Claude doesn't mix them up.
- **Tag names are your choice.** There is no fixed list; just keep opening and closing tags consistent. Tags can be nested.
- Refer to sections by name ("Using the rules in `<guidelines>`, review `<contract>`").
- Ask for the answer inside a tag (`<answer>`) for **easy extraction**. Tags help most in long, multi-part prompts.

**Exam traps**
- ❌ Claude treats part of a pasted document as an instruction, so shorten the document → ✅ Wrap the instructions and the document in separate tags.
- ❌ "Only `<system>` and `<user>` tags are allowed" → ✅ Any names, used consistently.

### L89 · Structure Patterns for Multi-Part Tasks

**Example:** An exam paper: first the reading passage, then the numbered questions, then a labeled space for each answer.

**Key facts**
- **Long documents first, the task last.** Instructions near the end are followed better.
- **One tag per document** (`<doc1>`, `<doc2>`), so Claude can tell sources apart and cite them.
- **Number the steps** of a multi-part task, so none get skipped.
- Give an **output slot** (a tag or heading) for each answer, for organized, parseable results.

**Exam traps**
- ❌ The instruction comes before a 5-page contract, and risks get missed → ✅ Contract first, instruction last.
- ❌ Three articles in one block, and facts get mixed up → ✅ One tag per article.


---

## Part 2 · Advanced Prompting Techniques (L90–97)

*From the D4-2 summary.*

### L90 · Why Examples Work: Pattern Matching and Format Anchoring

**Example:** Copying a sample form. Give someone one correctly filled form, and they copy its layout exactly — same fields, same order.

**Key facts**
- Claude is a **pattern matcher**: input → output examples teach the rule without you explaining it.
- **Format anchoring:** examples fix the **output shape** (keys, labels, style) more reliably than written format rules. **Showing beats telling.**
- Claude copies **everything** in the examples, including mistakes and inconsistencies.
- The **last example** has a strong pull on the shape of the output.

**Exam traps**
- ❌ Detailed format instructions still give inconsistent output, so repeat the instruction → ✅ Add 3–4 examples in the exact format.

### L91 · Designing High-Quality Few-Shot Examples

**Example:** Training a new support agent with sample tickets. Show only easy, happy customers, and they freeze on an angry one. Show easy, angry, and confusing tickets, and they can handle anything.

**Key facts**
- **Quality and consistency beat quantity.** The usual sweet spot is **3–5** examples.
- **Cover the variety**, including edge cases (sarcasm, mixed feelings, ambiguous text).
- **Balance the labels.** 4 Positive + 1 Negative pushes Claude toward Positive.
- **Use one format** in every example ("Positive" every time, not "positive" or "POS").
- Wrap the examples in `<examples>` / `<example>` and the real input in a separate tag (for example, `<task>`), so Claude doesn't treat the real input as one more example.
- ➕ Exam guide: for **ambiguous cases**, use **2–4 targeted examples** that show **why** one choice beats the other likely choices. Good examples help Claude **apply its judgment to new cases**, not just copy. In extraction, they also reduce empty or invented fields when documents come in **different formats**.

**Exam traps**
- ❌ The classifier fails on sarcasm, so add more easy examples → ✅ Add correctly labeled sarcastic examples.
- ❌ Claude continues your real input as if it were another example → ✅ Separate `<examples>` from `<task>`.

### L92 · Chain-of-Thought (Visible Reasoning)

**Example:** "Show your work" on a maths exam. Writing down each step helps you catch your own mistakes.

**Key facts**
- **Chain-of-thought (CoT)** = ask Claude to reason step by step **before** the final answer. The biggest gains are on maths, logic, and multi-step decisions.
- Ways to trigger it: simple ("Think step by step") · guided ("First list the costs, then compare, then decide") · structured (`<thinking>`, then `<answer>`).
- **Reasoning first, answer second** — the order matters.
- Put the reasoning in `<thinking>` and show users only the `<answer>`.
- Cost: more tokens and more time. Skip it for simple lookups.
- CoT is **requested in the prompt**; extended thinking is an **API feature** (L93).

**Exam traps**
- ❌ Ask for the answer first and the reasoning after → ✅ Reasoning first.
- ❌ Use CoT for "What is the capital of Japan?" → ✅ Skip it — no accuracy gain.

### L93 · Extended Thinking

**Example:** CoT is asking a student to "show your working". Extended thinking gives the student a scratchpad and quiet time before speaking — built into the exam itself.

**Key facts**
- Turned on with a **`thinking` setting in the API call** (not in the prompt). The reasoning comes back in separate **thinking blocks**, followed by a text block with the answer.
- Classic mode: `thinking: {type: "enabled", budget_tokens: N}` — **minimum 1,024**, and it must be **less than `max_tokens`**.
- Newer models (4.6 and later) use **adaptive thinking** (`type: "adaptive"` plus an **effort** setting); `budget_tokens` is deprecated there.
- You pay for the **full** thinking tokens, but the API returns only a **summary** of the thinking.
- **Interleaved thinking** lets Claude think between tool calls.
- Use it for hard reasoning (debugging, maths, logic, reviewing non-trivial code). Skip it for lookups, extraction, classification, and format conversion.

**Exam traps**
- ❌ `budget_tokens: 500` → ✅ The minimum is 1,024.
- ❌ Extended thinking for simple ticket classification → ✅ Turn it off; it only adds cost and delay.

### L94 · Steering Model Output

**Example:** A TV host cannot change what a guest knows, but can say "answer in one sentence", start the guest's sentence, or cut to a break at the right moment.

| Lever | What it controls | Status |
|---|---|---|
| Direct instruction | Format, tone, no preamble | Now the main lever |
| Format anchoring (examples) | The output's shape | Works |
| Prefilling | The first words of the reply | ⚠️ Not on Claude 4.6+ (400 error) |
| Stop sequences | Where the output ends | Works |

**Key facts**
- On current models, **direct instructions** ("Respond with JSON only. No preamble.") replace most of the old tricks.
- **Prefill breaking change:** from Claude 4.6, a prefilled **last assistant turn** returns a **400 error**. Assistant messages **earlier** in the conversation (for example, few-shot examples) still work.
- Replacements: to force JSON → **structured outputs** or a tool with an enum; to remove "Here is…" → a system-prompt instruction; to continue a cut-off reply → put the partial text in a **new user turn** and ask Claude to continue.
- A replayed history that **ends on an assistant turn** is read as a prefill → **always end on a user turn**.
- **Stop sequences** end the output at a chosen string (for example, `</answer>`).

**Exam traps**
- ❌ Prefill `{` to force JSON on a 4.6 model → ✅ Structured outputs, or "JSON only, no preamble".
- ❌ Continue a cut-off reply by prefilling it → ✅ A new user turn: "Continue from where you left off".

### L95 · Temperature, top_p, top_k

**Example:** Ordering at your usual restaurant. Temperature is your mood (low = always your favourite dish; high = adventurous). top_k = "I only look at the top 5 dishes". top_p = "I only look at the dishes that make up 80% of all orders".

| | Temperature | top_p | top_k |
|---|---|---|---|
| Controls | Randomness of the whole list | Pool by total probability (flexible size) | Pool by a fixed count |
| Anthropic range | **0.0–1.0, default 1.0** | 0.0–1.0 | 1 and up |
| When to touch it | The main dial | Advanced use only | Rarely |

**Key facts**
- Claude picks each next token from a probability list. Temperature **divides the logits before softmax**: low = sharper (focused), high = flatter (creative).
- Low (near 0) for extraction, classification, code, and facts; high (near 1) for brainstorming and creative writing.
- top_p example: with cat 0.40, dog 0.25, bird 0.15 → **top_p 0.80 keeps {cat, dog, bird}**. **top_k 1** = always the top token (greedy).
- Golden rule: **change temperature or top_p, not both** — usually just temperature.
- Temperature 0 is **not fully deterministic**, and the Anthropic API has **no seed** parameter.
- Newer models increasingly use an **effort** setting instead of hand-tuned sampling.

**Exam traps**
- ❌ Temperature 1.8 → ✅ Anthropic's range is 0.0–1.0.
- ❌ Data extraction at temperature 0.9 → ✅ Lower it toward 0.

### L96 · Multi-Turn Interactions

**Example:** A helpful shopkeeper. "I need running shoes" → "What size?" → "Size 9" → they narrow it down. Each exchange builds on the last.

**Key facts**
- Messages alternate between **user** and **assistant**.
- **The API is stateless.** Claude only "remembers" earlier turns because **you resend the whole history** with every call.
- Resend only **relevant** context to control cost. Refer to earlier turns ("using the itinerary above…").
- Use multi-turn for clarification and step-by-step refinement; use a single turn for simple one-shot tasks.
- **End the array on a user turn** (see L94).

**Exam traps**
- ❌ "Claude should remember the last message automatically" → ✅ You must resend the history.

### L97 · Conversation State and Turn Management

**Example:** A meeting whiteboard. Every point goes on the board. When it fills up, someone wipes old notes or writes a short summary at the top. The board is the meeting's memory.

**Key facts**
- **State lives in the message array you resend.** It grows every turn, so the **cost rises** every turn and the history eventually hits the **context window**. Long histories also **bury details in the middle**.
- Strategies: **trim** old turns · **sliding window** (keep the last N turns; forgets early information) · **summarize / compact** old turns · **keep anchors** (system prompt + key facts).
- Typical pattern: **system prompt + a summary of old turns + the recent turns word for word**.

**Exam traps**
- ❌ A 100-turn chat is near the limit, so raise max_tokens → ✅ Summarize older turns and keep recent ones word for word.


---

## Part 3 · Structured Output & Evaluation (L98–104, Eval 1–2, Batch)

*From the D4-3 summary.*

### L98 · Structured Outputs: Schema-Constrained JSON

**Example:** A passport application form vs. a blank sheet of paper. With exact boxes, people can only fill in that shape.

| Level | Method | Guarantee |
|---|---|---|
| 1 | Prompting ("return JSON") | Weakest — may add markdown, a preamble, or bad commas |
| 2 | **JSON outputs** (`output_config.format`) | Strong — constrains Claude's **response** |
| 3 | **Strict tool use** (`strict: true`) | Strong — constrains the **tool-call arguments** |

**Key facts**
- Structured outputs work by **constrained decoding**: the schema is compiled into a **grammar** that limits which tokens can be generated, so invalid JSON becomes impossible.
- **`additionalProperties: false`** is required on every object (no extra fields).
- The first use of a new schema is slower (the grammar compiles). Compiled grammars are **cached for 24 hours**; changing the schema clears the cache.
- It is the modern replacement for the old prefill-`{` trick. Don't use it for **prose** (text people read).
- The grammar enforces structure, field names, types, required fields, and enums. **Numeric ranges and string lengths are not enforced** → check them in your own code.
- ➕ Docs check: simple regex **`pattern`**s and common string **`format`**s (date, email, uri, uuid) **are** supported, with limits (no backreferences or lookahead). *(My notes say regex is not enforced.)*

**Exam traps**
- ❌ "Return JSON" in the prompt keeps breaking the parser → ✅ Use structured outputs.
- ❌ A score must be 1–5, but Claude returns 7 with structured outputs on, so "the feature is broken" → ✅ Ranges are not enforced. Validate them in code.

### L99 · Schema Design for Reliable Extraction

**Example:** Designing a form for a data-entry clerk: clear boxes ("First name", "Date DD/MM/YYYY", "Status: ☐ Active ☐ Closed") and a note saying "leave blank if unknown".

**Key facts**
- **Valid ≠ correct.** Structured outputs guarantee a valid **shape**, not correct **values**. Correct values depend on good schema design.
- **Clear field names** work as instructions (`invoice_date`, `total_amount_usd` — not `date`, `num`).
- Use the **right types** (`is_paid: true`, not "yes") and **enums** for fixed sets (the grammar enforces them).
- **Missing data is the #1 extraction challenge.** Give Claude an honest way to say "not there": **nullable** fields (`["string", "null"]`) or a **"not_stated"** enum value. Otherwise it may **invent** a value to fill a required field.
- **Descriptions** guide what to extract (units, format). Prefer **flat** schemas, and **one field = one value**.
- ➕ Exam guide: for categories that may grow, use an enum with **"other" + a free-text detail field**, and add **"unclear"** for ambiguous cases. Put **format-normalization rules** in the prompt (for example, all dates as YYYY-MM-DD) alongside the schema.

**Exam traps**
- ❌ A required phone field gets a fake number when none exists, so raise max_tokens → ✅ Make the field nullable.
- ❌ "priority" comes back as "urgent", "High", "P1" → ✅ Use an enum.

### L100 · Structured Output Limits

**Example:** A vending machine reliably gives you what is in its slots, but it can't make fresh coffee or hold unlimited items.

**Key facts**
- **Limit 1 — only a subset of JSON Schema:** no numeric ranges and no string-length limits (for regex, see L98). Check those in code after generation.
- **Limit 2 — complexity caps** (number of properties, nesting, grammar size) are counted **across all strict schemas in one request together**. For example, 4 strict tools with 6 optional parameters each can already hit a limit.
- **Limit 3 — performance:** compile delay on first use; a 24-hour cache; a schema change makes it slow again. The list of supported models changes — check it before pinning a model.
- **Limit 4 — not for prose.**

**Exam traps**
- ❌ Each tool looks simple, yet the request is rejected → ✅ The limits count across all strict schemas together.
- ❌ The next call is slow after a schema edit → ✅ The cached grammar was cleared.

### L101 · Using Tool Definitions to Extract Data

**Example:** You hand a waiter an "order form" even though you have no kitchen — you only wanted the order written in exact boxes. The "tool" is a shape, not an action.

**Key facts**
- Define an **extraction tool** whose `input_schema` is the structure you want → **force it** with `tool_choice` → read the data from the **`input` of the `tool_use` block**. The tool is **never executed**.
- `tool_choice`: `auto` (may answer in prose — not good for extraction) · `any` (must use some tool) · **`{"type": "tool", "name": …}`** (must use that tool — best for one extraction shape) · `none`.
- Add **`strict: true`** for guaranteed-valid arguments (`additionalProperties: false` is still required).
- **JSON outputs vs. tool extraction:** a pure "text → JSON" job → JSON outputs (simpler; the result is in `content[0].text`). Inside an agent or tool workflow → strict tool use.
- ➕ Exam guide: **document type unknown, with several extraction schemas** → `tool_choice: "any"` (Claude picks the right one). To make one extraction run **first** (for example, `extract_metadata` before enrichment) → force that tool, then do the next steps in follow-up turns.

**Exam traps**
- ❌ With `auto`, Claude sometimes answers in prose → ✅ Force the specific tool.
- ❌ strict: true is on, but "score" is 500 (it should be 0–100) → ✅ Ranges still need validation in code.

### L102 · Validation Loops and Hallucination Prevention

**Example:** An editor checks a draft and sends it back with notes ("this figure is not in the source"). A writer who only uses what is in the source needs fewer rounds.

| Type of check | Question | Handled by |
|---|---|---|
| Structural | Is it valid JSON in the right shape? | Structured outputs (the grammar) |
| Semantic | Are the values correct, in range, and found in the source? | **Your** validation code |

**Key facts**
- **Validation loop:** generate → validate → on failure, send back the **specific error** ("age was 250; it must be 0–120") → retry. Blind retries fix very little.
- Your code checks what the grammar can't: ranges, lengths, **cross-field logic** (end date after start date), and **grounding** (is the value really in the source?).
- Anti-hallucination: use **only the provided text** · give a legal **"not present"** option (the #1 move) · ask for a **source quote** for each value · add a **confidence** field · use a **low temperature**.
- ➕ Exam guide: in the retry, include the **original document + the failed output + the specific errors**. For self-checks, extract **`stated_total` and `calculated_total`** (plus a **`conflict_detected`** flag), so mismatches show up automatically.

**Exam traps**
- ❌ The JSON is valid but the numbers are wrong, so raise max_tokens → ✅ Add semantic validation in code.
- ❌ Retry the same request with no feedback → ✅ Send the specific error back.

### L103 · Validation Gates and Retry Strategy

**Example:** Airport security. You don't reach the plane unless you pass the scan. You get a couple of tries, then you are taken aside for a manual check.

**Key facts**
- A **validation gate** sits between Claude's output and the next system. Only output that passes the **structural + semantic** checks goes through.
- A retry strategy has: a **maximum number of retries** (for example, 2–3) · **better-informed** retries (send the error back; maybe lower the temperature) · a defined **fallback** when the retries run out.
- Fallbacks: **human review** (high stakes: medical, legal, finance) · a safe default value · flag and skip · return an error.
- **Exponential backoff** (wait 1 s, 2 s, 4 s…) is for **temporary** errors (rate limits, timeouts), **not** for bad content.
- ➕ Exam guide: retries fix **format or structure** problems, but **not missing information**. If the data is simply not in the source (for example, it is in another document you didn't provide), retrying won't help — mark it as missing or escalate.

**Exam traps**
- ❌ A retry loop with no limit → ✅ A maximum number of retries + a fallback.
- ❌ Use exponential backoff for bad content → ✅ Backoff is for rate limits and timeouts.

### L104 · Multi-Pass Review

**Example:** Writing an important letter: you draft it, re-read it critically the next morning, then revise the weak parts.

**Key facts**
- **Multi-pass review** = separate passes with **different jobs**: **produce → critique → revise**.
- It is **proactive** (it runs every time). A validation loop is **reactive** (it runs only on failure).
- A dedicated critique pass catches errors that a single "do it perfectly" pass misses.
- Cost: more tokens and more time → use it for complex, high-stakes, or long tasks.
- ➕ Exam guide: do the review with an **independent instance** — a fresh session **without the generator's reasoning**. It catches more than **self-review in the same session** or extended thinking. For large multi-file reviews: **per-file passes** for local issues + a separate **cross-file integration pass** (this avoids diluted attention and contradictions).

**Exam traps**
- ❌ "Analyze it and make sure you missed nothing" in one prompt → ✅ Produce → critique → revise.
- ❌ Ask the same session to double-check its own code → ✅ Use an independent review instance.

### Eval 1 · Building a Prompt Evaluation Workflow

**Example:** Unit tests for code. You don't ship code because it "seemed to work once". An eval is unit tests for prompts.

| Grading method | Best for | Limits |
|---|---|---|
| Code / exact match | One right answer (classification, extraction) | Can't judge quality or tone |
| Model-based (LLM-as-judge) | Open-ended quality (summaries, tone) | Costs more; the judge must be validated |
| Human | The gold standard; high stakes | Slow, expensive, doesn't scale |

**Key facts**
- An **eval** runs a prompt on a **test dataset**, **grades** each output, and produces a **score**: dataset → run → grade → metrics. It replaces "vibes" with **numbers**.
- A good dataset covers **typical, edge, and known-failure** cases, each with a clear definition of success, and has enough cases to trust the number.
- **Regression testing:** re-run the **whole** suite after every prompt change.
- Keep a change **only if the overall score goes up**.

**Exam traps**
- ❌ Ship whatever "looks better" on one example → ✅ Run the eval suite.
- ❌ An eval of only easy cases shows 100% → ✅ Add edge and known-failure cases.

### Eval 2 · Test Datasets and Model-Based Grading

**Example:** An expert drafts a large question bank (you review it), and a second examiner marks essays with a rubric (you spot-check that examiner against a senior marker).

**Key facts**
- Test cases come from: **manual** writing (high quality, slow) · **synthetic** generation by Claude (fast and full of edge cases, but must be **curated**) · **real production data** (most realistic; clean it and protect privacy).
- Ask for a specific spread: "5 clear, 5 ambiguous, 5 with missing info, 5 adversarial".
- **Self-reference trap:** if the same model writes **and** answers the test cases, the eval is too easy. Use human-checked answers.
- A good **LLM judge** has a clear rubric · **reasoning before the score** · structured output (`{reasoning, score}`) · a fixed scale · a **low temperature**.
- **Validate the judge** against human grades before you trust it.
- Judge biases: **verbosity** (longer looks better) · **position** (order matters — swap and average) · **self-preference**.
- **Reference-based** grading (a gold answer is given) vs. **reference-free** grading (rubric only).

**Exam traps**
- ❌ Trust a new LLM judge immediately → ✅ Check it against human grades first.
- ❌ Scores look perfect when one model both made and answered the test set → ✅ That is the self-reference trap.

### Batch · Batch Processing with the Message Batches API ⚠️ GAP FILL

*Not from my notes (only a short mention in my D3-4 notes). Based on the official exam guide's Prompt Engineering & Structured Output topics.*

**Example:** Sending a big pile of letters by cheap overnight post instead of a courier. It costs half as much, but you can't stand at the door waiting for an answer.

**Key facts**
- **Message Batches API:** about **50% cheaper**, processed **within up to 24 hours**, with **no latency guarantee**.
- Good for **non-blocking** work: overnight reports, weekly audits, nightly test generation. Wrong for **blocking** work — a pre-merge check that someone is waiting for → use the normal (synchronous) API.
- Each request in a batch is **one independent Messages call**. There is **no back-and-forth tool loop** inside a request (you can't run a tool and send the result back mid-request).
- Give every request a **`custom_id`** to match each result to its request.
- On failures, **resubmit only the failed items** (found by `custom_id`), with a fix if needed (for example, split a document that was too long).
- **Test and refine the prompt on a small sample first**, then batch the full volume — fewer costly resubmissions.
- Plan submission times around your deadline, because processing can take up to 24 hours.

**Exam traps**
- ❌ Use the Batch API for a blocking pre-merge review → ✅ Use the synchronous API (or `claude -p`).
- ❌ Resubmit the whole batch because 3 items failed → ✅ Resubmit only those items, by `custom_id`.

---

## Section notes in this folder

- [D4-1 Summary — Effective Prompts](4.1-effective-prompts.md)
- [D4-2 Summary — Advanced Prompting Techniques](4.2-advanced-prompting-techniques.md)
- [D4-3 Summary — Structured Output & Evaluation](4.3-structured-output-and-evaluation.md)

[← Back to home](../README.md) · [Cheat sheet](../cheat-sheet.md)
