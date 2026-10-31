# CCA-F: D1 Domain Summary — Agentic Architecture & Orchestration

**Exam weight:** 27% — the biggest domain. **Numbering:** course D1 = official blueprint Domain 1 (same name).

**Course coverage:** Sections 2–5 · 30 lectures + 4 quizzes (about 3 hr 24 min of video).

**Legend:** ❌ wrong idea (exam trap) → ✅ right idea · ⚠️ GAP FILL = not from my notes · ➕ = exam-guide check (added)

| Part | Course section | Lectures | Gap fills |
|---|---|---|---|
| 1 · Foundations | Section 2 | L3–9 (L7 = 1-min check-in) | L9 ⚠️ |
| 2 · Task Decomposition & Planning | Section 3 | L10–15 | — |
| 3 · Multi-Agent Orchestration | Section 4 | L16–23 | L19 ⚠️ |
| 4 · Reliability & Human Oversight | Section 5 | L24–32 | L31 ⚠️ |

*Quizzes 1A–1D are not included.*


---

## Part 1 · Foundations (Section 2 · L3–9)

*From the D1-1 summary.*

### L3 · Agents vs. Workflows vs. Chatbots

**Example:** Chatbot = info desk clerk (only answers). Workflow = fast-food assembly line (same steps every time). Agent = personal chef (decides the next step while cooking).

**Key facts**
- Main question: **who decides the next step?** Chatbot: no steps, it only replies. Workflow: the **developer**, at design time. Agent: the **model**, at runtime.
- An LLM alone is not an agent. It only outputs text. The **action-environment gap** = the space between "knows what to do" and "did it". **Tool use** closes this gap.
- **Augmented LLM** (Anthropic's building block) = model + **retrieval + tools + memory**.
- **Tool schema** = name + description + parameters. Claude chooses tools by reading the **description**.
- Autonomy is a spectrum. Most real systems sit in the middle. More autonomy = more risk.
- Build agents less often than you think. If you can write the steps in advance, use a workflow. (Anthropic says "simplicity". The course says "minimal footprint".)

**Exam traps**
- ❌ "Many LLM calls, so it is an agent" → ✅ If the developer fixed the order, it is a workflow (for example, prompt chaining).
- ❌ Fix wrong tool choice by renaming parameters, reordering tools, or changing temperature → ✅ Fix the tool **description** first.

### L4 · Tool Use as the Foundation of Agency

**Example:** A manager who cannot leave the desk writes a note (tool call). A runner goes and does the job (runtime). The answer comes back on the note (observation). The manager reads it and writes the next note (the loop).

**Key facts**
- Every agent step has 3 parts: **tool definition → tool call → observation**.
- API round trip: the response has `stop_reason: "tool_use"` and one or more `tool_use` blocks (tool name + JSON arguments). Your code runs the tool and sends back a `tool_result` block with the same tool-use ID.
- **Client tools:** your application runs them (for example, your database). **Server tools:** Anthropic runs them (for example, web search).
- Workflow: the developer chooses the tools and the order. Agent: the model chooses tools at runtime. **Runtime tool choice is the defining trait of an agent.**
- A good tool description says **what it does, what input it needs, and what it returns**. Not how the code works inside.
- Tool types: **data access** (read), **computation** (run code), **external API** (change things in the world). Each has different speed, failures, and security risk.
- After each result: success → move on. Error → replan (retry differently, try another tool, or escalate). Partial result → make a follow-up call.
- **Parallel tool calls:** many independent calls in one response → lower latency.

**Exam traps**
- ❌ "The model fills in the parameters, so it is an agent" → ✅ The developer fixed the tools and the order, so it is a workflow.
- ➕ ❌ Decide the loop is finished by reading the model's text ("I'm done") or by checking for text content → ✅ Use `stop_reason`: continue while it is `"tool_use"`, stop when it is `"end_turn"`. Add each tool result to the conversation before the next call.

### L5 · Perception, Reasoning, Action, Observation

**Example:** A detective reads the case file (perception), thinks about what is missing (reasoning), interviews a witness (action), and adds the new clue to the file (observation). Then the detective reads the file again.

**Key facts**
- The agent loop: **Perception → Reasoning → Action → Observation**, then repeat. It is a feedback loop, not a straight line.
- **Perception** reads everything in context: instructions, history, documents, and all tool results so far.
- **Reasoning** picks the next action that moves toward the goal. The plan can change every turn (dynamic planning).
- **Action** = a structured tool call. **Observation** = the tool result added back into context. The tool result is what turns an action into an observation.
- The context window is the agent's working memory. Calls, results, and reasoning pile up. Long runs can run out of space unless you summarize or trim (see D5).
- In-context memory is fast but **lost** when the session ends or the context is cut.
- Same loop, three names: L3 = Perception → Selection → Execution → Iteration. L5 = Perception → Reasoning → Action → Observation. L6 (Anthropic's words) = **gather context → take action → verify work**.

**Exam traps**
- ❌ "Reading the returned search summaries is Observation" → ✅ Reading = **Perception**. Observation = the result arriving in context.
- ❌ "Deciding to refine a search query is Perception" → ✅ Any decision = **Reasoning**.

### L6 · Loop Control: Stopping, Iterating, Escalating

**Example:** Rules for a new employee: "You are done when X is finished" (termination condition). "Stop after 2 hours and show me" — the manager tracks the time, not the employee (turn budget). "Come and get me if you are stuck" (escalation).

**Key facts**
- **Termination condition** tells the agent it is done. **Turn budget** = a hard iteration limit, enforced by the **runtime/orchestrator**, not by the model. **Escalation** = a planned handoff to a human or a fallback system.
- Four reasons to stop: **task complete**, **error threshold reached**, **budget used up**, **stop signal**. Handle all four.
- Infinite loop causes: no stop condition, a tool that always fails but is retried forever, circular reasoning. Fix: clear stop conditions + hard caps, set **before** the agent runs.
- **Spinning:** the same tool call with the same parameters, the same errors, no new data, and the gap to the goal does not shrink. Detect it by comparing results and counting repeats.
- **Iterate** when there is new data, a new approach, or a recoverable error. **Escalate** at the authorization boundary, when information is missing, or when human judgment is needed.
- Choose the cap on purpose (simple lookup: 3–5; research task: about 20). Never leave it open.
- Escalation is a designed safety valve, not a failure.

**Exam traps**
- ❌ "Tell the model in the prompt to count its turns" → ✅ The runtime counts the turns and stops the loop.
- ❌ The agent reaches an approval it is not allowed to give, so it retries → ✅ Escalate.
- ➕ ❌ "Raise the iteration cap" to fix an agent that stops too early → ✅ The main stop signal is `stop_reason` (`"end_turn"`). A cap is only a safety net against runaway loops.

### L8 · When Workflows Outperform Agents

**Example:** You cook the same dish every day → use a recipe card (workflow). Hire an improvising chef (agent) only when the job changes every time.

**Key facts**
- The deciding question: **does the task need judgment at runtime?** No → workflow. Yes → maybe an agent.
- Three checks: Is the input predictable? Is the output known and testable? Can every step be written at design time? **All yes → workflow.**
- Workflow: fixed flow, predictable, auditable, cheaper, faster. A failure points to one step. Agent: flexible, higher ceiling, but more cost, more latency, more failure points, harder to debug.
- Anthropic's **5 workflow patterns:** prompt chaining · routing · parallelization · orchestrator–workers (course: orchestrator–subagent) · evaluator–optimizer. All five are still **workflows**.
- **Routing:** a classifier sends each input to the right path. The classifier can be an LLM, but the paths are fixed.
- **Parallelization:** calls run at the same time and join at a fan-in step. **Evaluator–optimizer:** one call writes, another grades, repeat until good enough.
- Agents raise token cost on **every** run, not only on hard tasks.
- Rule: use an agent only when a workflow **clearly** falls short.

**Exam traps**
- ❌ "Agents are more capable, so choose an agent" → ✅ If all three checks pass, choose a workflow.
- ❌ "Routing with an LLM classifier is an agent" → ✅ Still a workflow, because the paths are fixed.

### L9 · Choosing the Right Architecture ⚠️ GAP FILL

*Not from my notes. Based on Anthropic's "Building effective agents" guidance and my L3 + L8 notes.*

**Example:** To get across town you can walk (one simple LLM call), take a bus on a fixed route (workflow), take a taxi that picks its own route (agent), or hire a moving company with many workers (multi-agent). Pick the simplest option that does the job.

**Key facts**
- Start with the simplest design. Add complexity only when it clearly improves the results.
- Decision ladder:
  1. One call with a good prompt (and retrieval) is enough → **single LLM call**.
  2. The steps are known in advance → **workflow**. Pick the pattern: chaining, routing, parallelization, orchestrator–workers, or evaluator–optimizer.
  3. Open-ended, the number of steps is unknown, and the path must adapt to results → **single agent**.
  4. Large breadth-first work that splits into independent parts, and the value is worth the cost → **multi-agent** (use the L24 fit test).
- Agents trade cost and latency for better results on open-ended tasks. Accept this trade only when it pays off.
- Every agent needs clear tool descriptions, stop conditions, guardrails, and testing in a safe sandbox.
- Mixed designs are normal. A workflow can contain one agentic step.

**Exam traps**
- ❌ Choose the most powerful or most impressive design → ✅ Choose the simplest design that meets the need.
- ❌ Multi-agent for tightly coupled steps → ✅ A single agent or a workflow.


---

## Part 2 · Task Decomposition & Planning (Section 3 · L10–15)

*From the D1-2 summary.*

### L10 · Breaking Complex Goals into Subtasks

**Example:** A building contractor splits "build a house" into foundation, framing, plumbing, and roof. If the plumbing fails inspection, you fix only the plumbing, not the foundation.

**Key facts**
- **Orchestrator** = owns the plan, assigns subtasks, controls the order, and combines the results.
- Main benefit: each subtask can be **retried, delegated, and tested on its own**. If step 4 fails, retry only step 4.
- Too big (**monolithic**): fragile, no parallel work, one failure means a full retry. Too small (**over-decomposed**): coordination costs more than the work. Aim for meaningful chunks.
- Three questions before you split: Can parts run independently? (→ parallel) Does one output feed the next? (→ sequential) Are different skills needed, like retrieve → analyze → write? (→ natural split points)
- A good subtask has **one responsibility · defined inputs · bounded scope · clean structured output**.
- Handoff rules: pass **only what is needed** · use a **structured schema** · **write down what is NOT passed**.
- Every boundary adds logic, latency, and a new failure point. It must "earn its keep".

**Exam traps**
- ❌ Run "retrieve reviews → summarize → write report" in parallel → ✅ Each output feeds the next step, so it must be sequential.
- ❌ Pass the full context in every handoff "to be safe" → ✅ Extra context is noise. Pass only what is needed.
- ❌ A subagent quietly assumes data it never received → ✅ This causes silent wrong output. Document what is not passed.

### L11 · Decomposition Patterns: Hierarchical, Sequential, Parallel

**Example:** A family dinner. Head chef → station chefs → line cooks (hierarchical). Reduce the stock → add cream → season (sequential). Grill the steak while the salad is plated (parallel).

**Key facts**
- **Hierarchical:** a tree of goals. Each level handles only its own layer. More levels = a bigger **error propagation surface** (an error must pass up through every layer). Merge levels that run the same agent with almost the same prompt.
- **Sequential:** data dependency forces the order. Easy to debug. Total time = the sum of all steps.
- **Parallel:** independent subtasks run at once (five 10-second tasks take 10 s, not 50 s). Needs no shared data that branches change, a **fan-in** step, and a **partial-failure policy**.
- **Data dependency decides.** If B and C both need A, but not each other → run A first, then B and C in parallel.
- **Race condition:** two branches write to the same resource without coordination.
- Three anti-patterns: over-decomposition · under-decomposition · **false parallelism** (the most dangerous, because it gives silent wrong answers).
- A pattern is a **reading of the dependencies**, not a preference. Real systems mix all three.

**Exam traps**
- ❌ "Deeper trees are cleaner, so add a level" → ✅ Depth must earn its cost through real specialization.
- ❌ Mixing up the two mistakes → ✅ Unneeded sequential order = **missed optimization** (only slower). Dependent steps run in parallel = **data hazard** (wrong output).

### L12 · Sequential Pipelines: Design and Trade-offs

**Example:** A car assembly line: frame → engine → doors → paint → inspect. If the engine robot jams, every later station waits. Radio and floor mats installed one after the other only because "that is the layout" = an artificial dependency.

**Key facts**
- **Prompt chaining** = one LLM call's output is the next call's input. It is a workflow technique, not agentic.
- A pipeline stops when a step fails. All later steps have no input (**cascade**). The risk grows with the number of steps.
- **Dependency graph:** draw an arrow A → B if B needs A's output. No incoming arrows → can run first or in parallel. Many incoming arrows → a **synchronization point**.
- Sequential is right when there is a **true data dependency**, state **builds up**, the order is **required** (parse → validate → enrich), or **auditability** matters more than speed.
- **Handoff schema:** step N's output schema = step N+1's input contract (required fields, types, nullability). It catches mismatches at the boundary.
- **Validate at each step's entry.** **Fail loudly.** Never pass bad data forward.
- Removing **artificial dependencies** is the biggest latency win, with no cost to correctness.

**Exam traps**
- ❌ Validate only the final output → ✅ Validate at every step boundary.
- ❌ A missing field is quietly set to null and the pipeline continues → ✅ Use an explicit schema, so it fails clearly at the boundary.

### L13 · Parallel Execution: Fan-Out, Fan-In, Synchronization

**Example:** A teacher splits 100 exam papers among 4 assistants (fan-out), then collects them and adds up the scores (fan-in). If one assistant returns an empty pile, the teacher must decide out loud what to do — never report those papers as graded.

**Key facts**
- **Fan-out** = send independent branches at the same time. A branch can be a function, a tool call, a sub-agent, or a nested workflow. **Fan-in** = collect results, handle failures, put them back in order if needed, and merge.
- **Independence test (before sending):** (1) Can B start without A's output? (2) Do the branches avoid writing to shared data? (3) Does any order give the same result? Any "no" → not parallel.
- Partial failure — choose one, explicitly: **fail all** (every branch is required) · **continue with partial results + flag the missing ones** · **retry only the failed branches**.
- Merge strategies: **voting** (same question, majority wins) · **concatenation** (separate pieces joined into one list) · **structured aggregation** (each branch fills one named field). **Choose the merge strategy before designing the branches.**
- Cost: parallel cuts waiting time, but **total tokens = the sum of all branches**. Rate limits can cap real concurrency.

**Exam traps**
- ❌ Silently drop failed branches and continue → ✅ Always wrong. It hides errors.
- ❌ "Token cost = the cost of the longest branch" → ✅ It is the sum of all branches.
- ❌ "The fan-in step will fix conflicting writes" → ✅ That is a race condition. Fix it before sending the branches.

### L14 · Adaptive Planning: Updating the Plan Mid-Execution

**Example:** Paper directions written before the drive (static plan) vs. a GPS that reroutes live (dynamic planning). A reroute after a road closure = replanning. A GPS that, after 20 reroutes, drops you at *a* restaurant instead of *your* restaurant = goal drift.

**Key facts**
- **Static plan:** all steps decided before the run (predictable, auditable, cheap). **Dynamic planning:** the next step is decided at runtime (adaptable, less predictable). **Replanning:** revising the plan after something fails or changes.
- Most production systems mix both: fixed where possible, dynamic only where needed.
- Four events that force a rethink: **tool call failure** · **unexpected output** (the tool worked, but the result breaks an assumption) · **state change** (the world changed) · **constraint violation** (the most critical).
- Replan only when a **core assumption** is broken. For a minor problem, continue with small adjustments.
- Dynamic planning is less predictable. Compensate with **monitoring, logging, and explicit fallback paths**.
- **Goal drift** = the agent solves each obstacle but slowly stops serving the real goal. A quiet failure, not a crash. Prevent it with a **maximum replanning cap** + a **goal check before resuming**. Save the original goal and log each revision against it.

**Exam traps**
- ❌ Replan on every small problem → ✅ It wastes tokens, adds latency, and risks drift.
- ❌ Remove the iteration limit to give the agent more freedom → ✅ Never. The cap prevents endless loops.

### L15 · Handling Ambiguity and Incomplete Specs

**Example:** "Book me a table." A normal table you can cancel for free → assume 2 people at 7 pm and book it. A non-refundable $500 private room → message the person and ask first.

**Key facts**
- Two strategies: **clarify first** or **assume and proceed**. Neither is always right.
- Clarify first when the action is **irreversible**, **high-cost**, or the **scope is unclear**.
- Assume and proceed when the stakes are **low and reversible**, the task is **time-sensitive**, or the intent is **clear enough**.
- Decision criteria: **reversibility (the most important)** · stake level · scope clarity.
- Core rule: cheap to undo → go ahead. Expensive or permanent → stop and ask.
- **Write down assumptions** in the output or log. A reviewer can check them without rerunning the task.
- **Evaluator–optimizer loop:** a generator writes → an evaluator scores it against a rubric and gives specific feedback (not just pass/fail) → the generator revises. It needs **"good enough" criteria + a maximum number of rounds**. Best for writing and analysis.
- Long-term fix: write clearer task specs at the source.

**Exam traps**
- ❌ "Clarifying is always safer" → ✅ Match the strategy to the real risk.
- ❌ An evaluator–optimizer loop with no round limit → ✅ It can run forever when the evaluator is hard to satisfy.


---

## Part 3 · Multi-Agent Orchestration (Section 4 · L16–23)

*From the D1-3 summary.*

### L16 · The Orchestrator Role

**Example:** A travel agency team lead splits a Dubai trip into flights, hotel, visa, and tours. Each staff member gets a precise brief ("3 return flights, Delhi–Dubai, 10–17 March, 4 people, under budget X; send back airline, time, price"). The lead checks every reply before building the final plan.

**Key facts**
- Three jobs: **Decompose → Assign → Aggregate**. (Lecture title words: Directing ≈ Decompose, Delegating ≈ Assign.) Each job must be designed.
- A good task brief answers: What does success look like? What information and tools are needed? What is the output format?
- A bad subagent result usually comes from a **badly scoped task**, not from a bad subagent.
- The orchestrator keeps the master plan, execution state, error handling, and aggregation rules. A subagent gets only its instructions, its context, and its tools. **Nothing is passed automatically.**
- Aggregate = **collect → validate → synthesize**. Skipping validation causes errors that are hard to trace.
- When a subagent fails: **retry** (temporary problem) · **substitute** (agent not available → give the task to another) · **escalate** (neither helps). The subagent only reports; **the orchestrator decides**.
- Hub-and-spoke is easy to audit, but the hub is a single point of failure and a bottleneck.
- Log every assignment (with the routing reason), the progress, and every error (with context). This is the audit trail.

**Exam traps**
- ❌ Join subagent outputs without checking them → ✅ Validate each output before synthesis.
- ❌ Put retry and fallback decisions inside every subagent → ✅ The orchestrator decides. This keeps the system predictable and auditable.
- ❌ The final result misses whole topics, so blame the subagents → ✅ Check the orchestrator's task split and briefs first.

### L17 · Subagent Design: Scope, Authority, Isolation

**Example:** A visa specialist knows only what the team lead tells her. She can read passports and check visa rules. She cannot charge cards or change bookings (least privilege).

**Key facts**
- A subagent = a separate instance with its own context window and its own tool permissions. It **starts blank** (context isolation). This is by design.
- Benefits of isolation: no context bloat · no information leakage · lower token cost · easy to test alone.
- Pass: task instructions, needed data, output format, constraints. Hold back: full history, other subagents' outputs, unrelated system prompts, unneeded credentials. **Start minimal and add on purpose.**
- **Least privilege:** only the tools and data the task needs. Also write what is in scope and out of scope in the instructions.
- Avoid **side effects** on shared state by default.
- Good instructions = **narrow scope + precise output format + explicit success criteria**.
- "Testable alone" and "swappable" both need **no hidden dependencies**.
- Anti-patterns: **overly broad scope** (the most common) · underpowered context · unclear success criteria.
- Claude Code fact: each subagent has its own system prompt, tools, and context window. Subagents **cannot start other subagents**.

**Exam traps**
- ❌ If you must choose one: write a prompt rule instead of removing the tool → ✅ Remove the tool. That is real enforcement (see L29).
- ❌ A subagent sometimes stops halfway and sometimes adds extra work → ✅ The success criteria are missing.
- ❌ Wrong refund because the policy was never passed → ✅ Underpowered context.

### L18 · Hub-and-Spoke vs. Pipeline vs. Peer-to-Peer

**Example:** Three ways to run a travel agency: one team lead hands out and collects all work (hub-and-spoke); a line from Sales → Documents → Billing (pipeline); staff talk directly to each other with no lead (peer-to-peer).

| Topology | Wins | Breaks |
|---|---|---|
| Hub-and-spoke | One place for rules, state, and audit. The hub handles recovery. | Single point of failure. Bottleneck. Hub context grows. |
| Pipeline | Narrow stages. Test or swap one stage alone. | Time = sum of all stages. A middle failure blocks everything after it. |
| Peer-to-peer | Low latency. No single point of failure. | Hard to audit (no central log). Unexpected (emergent) behaviour. |

**Key facts**
- Choose by mapping the data flow: a natural boss that splits work → **hub-and-spoke**. Each step needs the previous output → **pipeline**. Loosely coupled, no shared state, no required order → **peer-to-peer**.
- **Override rule:** if you need a full decision log, peer-to-peer is out.
- An evaluator–optimizer loop needs clear "good enough" criteria **and** a maximum number of rounds.
- Anti-patterns: **hub without hierarchy** (a hub with no real coordination work = a bottleneck with no benefit) · **P2P without an audit trail** when a log is required.
- Hybrids are fine (for example, a hub that runs a pipeline inside). Each seam needs an explicit handoff: format, failure handling, state.

**Exam traps**
- ❌ Regulated system with loosely coupled agents → choose peer-to-peer → ✅ Avoid P2P when a complete decision log is required.
- ❌ All messages go through a coordinator, but nobody splits or delegates work → ✅ This is "hub without hierarchy": a bottleneck with no benefit.

### L19 · Selecting the Right Topology for the Problem ⚠️ GAP FILL

*Not from my notes. Based on my L18 and L24 notes and the exam guide's focus on the coordinator (hub-and-spoke) pattern.*

**Example:** Planning a group trip. Get quotes from 10 hotel suppliers at once → one lead + many searchers (hub-and-spoke). Visa → ticket → invoice in strict order → a line (pipeline). Two planners who freely negotiate with each other → only if no record is needed (peer-to-peer, rare).

**Key facts**
- **Step 0:** Do you need many agents at all? Run the L24 fit test. Default = one agent.
- **Step 1:** Map the data flow and dependencies first.
- **Step 2:** Match the shape:
  - Independent parts that need central control and merging → **hub-and-spoke** (the usual multi-agent design; the exam calls it "coordinator + subagents").
  - Strict order where each step needs the last one → **pipeline**.
  - Loosely coupled, no shared state, no order, no audit need → **peer-to-peer** (rare).
- **Step 3:** Check constraints. Audit or compliance needed → no P2P. Need speed → run spokes in parallel. Hub as a single point of failure → add checkpoints and recovery.
- In hub-and-spoke, all subagent communication goes **through the coordinator**: one place for observability, error handling, and control of what each agent sees.
- The coordinator picks which subagents to use for each request. It does not need to call every subagent every time.
- Hybrids are fine (a hub can run a pipeline inside). Design every seam.

**Exam traps**
- ❌ Pick a topology because it looks clean or the team likes it → ✅ Pick it from the dependencies and the audit needs.
- ❌ "Let spokes talk directly to save time" in a system that needs a full log → ✅ Keep all communication through the coordinator.

### L20 · Designing Agent-to-Agent Message Schemas

**Example:** Bad: "Book something nice for the Sharma family." Good: task (book 1 hotel) + context (4 guests, Dubai, 10–17 March, budget X) + output format (JSON: hotel name, price, confirmation ID) + constraints (free cancellation only, do not charge the card) + `schema_version: 2`.

**Key facts**
- The receiver knows **only what is in the message**. Nothing else crosses over.
- Four parts of a handoff message: **task · relevant context · output format · constraints**.
- Freeform failures: **missing context** (the most common) · inconsistent format · **implicit assumptions** (the most dangerous — the sender thinks the receiver already knows).
- Leave out: full history, the sender's reasoning logs, internal state, repeated context. Target: **minimal but sufficient**.
- Under-specified → the receiver guesses (hallucination, silent failure). Over-specified → more cost, slower, more noise.
- Versioning: a version field · handle missing optional fields without crashing · a new **required** field needs a **migration path** (update all senders) · forward compatibility.
- Test at development time: **contract tests** (required fields present), simulate the receiver with the schema only, test with missing fields.
- Design against an **interface**, not one agent. Then receivers can be swapped.

**Exam traps**
- ❌ Catch handoff mismatches with more production logging or runtime questions → ✅ Contract tests before production.
- ❌ Add a new required field and update only the receiver → ✅ Plan a migration path for all senders.

### L21 · Handoff Protocols and Continuity Across Agents

**Example:** Bad: the lead emails the hotel team and moves on. The email went to spam, and the guests arrive with no room. Good: the hotel team replies "Got it, and I have everything I need" — only then does the lead move on.

**Key facts**
- Handoff verification: **send → acknowledge receipt → confirm context → then release control**.
- **Silent failure** = looks successful, but data was lost and no error was raised. The worst kind of failure.
- Subagents return **structured error payloads** (error type + context + recovery hints). Never null or empty.
- Anti-patterns: **swallowing errors** (returns an empty result, so the orchestrator thinks it worked) · **undifferentiated errors** (one generic error code, so the orchestrator cannot choose retry, reroute, escalate, or stop).
- **Checkpoint:** save task state before each handoff, at the **last confirmed good state**. On failure, resume from the checkpoint, not from the start. This avoids repeating side effects (a second charge, a second email).
- Define the protocol by behaviour (inputs, acknowledgement, success output, error output), so any receiver that follows it can be swapped in.
- Audit log per handoff: sender, receiver, time, payload summary, verification result, errors.

**Exam traps**
- ❌ Release control right after sending → ✅ Only after the receiver confirms receipt and context.
- ❌ An API error is returned as an empty list ("no results") → ✅ That is a swallowed error. Return a typed error.

### L22 · In-Context State vs. External Memory

**Example:** On a phone call, a travel agent keeps what the customer just said in mind (in-context). The confirmed booking goes into the booking system (external). The hotel price can be checked again from the supplier (reconstructable — no need to store it).

**Key facts**
- **In-context:** instant, but limited by the window size, can be cut (truncated), and is lost when the session ends. **External:** durable and searchable, but every read or write is a tool call (more latency).
- Store types: **key-value** (exact ID lookup: profiles, task status, config) · **relational** (queries and joins: audit logs, records) · **vector index** (search by meaning). Match the store to how you search.
- **Hot** state → keep in context. **Cold** state → external storage. **Reconstructable** state → often do not store it.
- Default = in context. Persist only when it truly must survive.
- Four questions: Could it be recovered if the session ended now? Do you search it by meaning? Will another session or user need it? Can it be rebuilt cheaply?
- Hybrid (most production agents): a hot tier in context + a cold tier outside. On resume, load **only the minimal state** needed.

**Exam traps**
- ❌ Key-value store to "find complaints with a similar meaning" → ✅ Vector index.
- ❌ Save every tool result, even live prices → ✅ Over-persisting reconstructable state makes the agent slow.
- ❌ On resume, load the full history → ✅ Load only the minimal state.

### L23 · Session Continuity Across Turns and Failures

**Example:** A trip booking crashes at step 3 (charge the card). With a checkpoint, the agent sees "steps 1 and 2 done" and continues at step 3. Without one, it books the flights again.

**Key facts**
- To resume you need: the completed steps · the inputs for the next step · earlier results that later steps need.
- Checkpoint at the **end of each phase** and **BEFORE any irreversible action** (never after).
- A checkpoint holds: phase marker · step outputs · original task inputs · timestamp.
- **Coarse** (per phase) = default. **Fine** (per step) only for expensive steps, steps with side effects, or steps that are not safe to repeat (not idempotent).
- Recovery: check for a checkpoint → **validate its integrity** → load → skip completed phases → continue. A corrupt checkpoint is worse than none. A partial checkpoint → resume from it and flag the gap.
- **State versioning:** save a timestamp + environment hash. On resume, if the world changed, **flag the conflict and wait**. Never continue on stale state.
- Memory hygiene: remove stale state, but archive it first if it may be needed later.

**Exam traps**
- ❌ Save the checkpoint after charging the card → ✅ Save it before, or a restart may charge twice.
- ❌ Checkpoint after every tool call for cheap, safe steps → ✅ Coarse checkpoints at phase boundaries.


---

## Part 4 · Reliability & Human Oversight (Section 5 · L24–32)

*From the D1-4 summary.*

### L24 · Multi-Agent Cost and Fit

**Example:** Searching 10 hotel suppliers at once = good fit (independent searches, one lead merges them). Booking one package where the flight date sets the hotel date, and the hotel sets the transfer = poor fit (use one agent).

**Key facts**
- Token cost (Anthropic research): chat **1×** · one agent with tools **about 4×** · multi-agent **about 15×**.
- Costs stack up: orchestrator planning · each worker repeats setup context · merging all results (result stitching).
- Good fit = **breadth-first** work that splits into independent directions. Poor fit = **tightly coupled** work with shared state.
- **Fit test:** (1) Does it split into parallel parts that barely touch? (2) Does each part fit in its own context without shared state? (3) Is it valuable enough to pay several times more tokens? **All YES → multi-agent may be worth it. Any NO → one agent.**
- Default = one agent. Multi-agent must be justified.
- Save cost: the strongest model as the orchestrator; lighter, faster models for narrow subtasks.

**Exam traps**
- ❌ "The codebase is huge, so use 10 sub-agents" → ✅ Files depend on each other. Size is not the same as parallelism.
- ❌ "Two YES answers out of three is enough" → ✅ Any NO → one agent.

### L25 · Tool Errors vs. Reasoning Errors vs. Environment Errors

**Example:** A travel booking agent — the table shows one case of each error type.

| Error type | Where it comes from | Example | Fix |
|---|---|---|---|
| Tool – transient | Outside the model; temporary | Supplier API timeout, 503 | Retry with backoff + a retry limit |
| Tool – permanent | Outside the model; fails every time | 400 invalid date, 403 denied | Do not retry. Fall back or escalate. |
| Reasoning | Inside the model | Calls cancel_booking when asked to change it | Fix the prompt or context. Never retry blindly. |
| Environment | Under the tool | Tool code is fine, but its database is down | Wait, retry after a delay, monitor |

**Key facts**
- Golden rule: **classify first, then recover**. Ask: "Where did the failure come from?"
- HTTP hint: 5xx is usually transient; 4xx is usually permanent. **But 429 (rate limit) and 408 (timeout) are retryable.**
- Reasoning errors repeat with the same input. Causes: unclear instructions, missing state context, made-up facts (hallucination).

**Exam traps**
- ❌ Retry a permanent error or a reasoning error → ✅ It wastes tokens and delays the real fix.
- ❌ Escalate an environment error to a human → ✅ A short wait and retry usually fixes it. Escalating creates noise.

### L26 · Error Detection — Catch Failures Before They Cascade

**Example:** The supplier API returns 200 OK with valid JSON, but a 5-star hotel costs 0.00 → a range check catches it. `update_booking` says "success", but the date never changed → only reading the record back catches it.

**Key facts**
- **Silent failure** = looks successful, the output is wrong, and there is no error signal. Retry logic cannot help (there is no error code).
- **Validation gate** = a check between steps. The next step starts only if the output passes.
- Three strategies: (1) **output validation** — schema, range, completeness (not cut off); (2) **state verification** — read back after a write, confirm the side effect really happened; (3) **sanity checks** — reasonable length, key input items present, no contradictions.
- You need both: **schema validation** (structure — fast and easy) and **semantic validation** (does it make sense for this input?).
- Gate steps in order: inspect → retry with corrected input or prompt → fallback or degrade gracefully → escalate or alert.
- Log inputs and outputs at **every** step (structured logs, execution traces, anomaly detection), not only on errors.
- A check catches only what it was written to check. One check is rarely enough.

**Exam traps**
- ❌ The gate checks only status codes → ✅ It must check the meaning of the output.
- ❌ Trust the write API's "success" → ✅ Read the record back (state verification).

### L27 · Retry Logic — When to Retry, When to Abort

**Example:** 429 "try again in 30 seconds" → wait 30 s, then retry with backoff + jitter. 400 "invalid date format" → abort now. 500 for an unknown reason → one retry after a delay; if it fails again, treat it as permanent.

**Key facts**
- First question: **"Will this error go away if I try again?"** Classify before you choose a policy.
- Policies: immediate retry (rare) · fixed delay · **exponential backoff with jitter** (the standard; jitter stops all clients from retrying together — a "retry storm").
- Match the policy to the error: rate limit → backoff + jitter, respect the **Retry-After** header · timeout → short delay, 1–3 tries · invalid input → abort at once · reasoning error → abort and escalate.
- **Retry budget:** about 3 attempts with backoff, based on acceptable cost and latency — not on "maybe it will work later".
- Abort when the budget is used up, or on a permanent error at the first try. **Abort must escalate. Never fail silently.**

**Exam traps**
- ❌ Retry a 400 invalid input 5 times with backoff → ✅ Abort on the first failure and report it.
- ❌ When the budget runs out, return an empty result → ✅ Escalate the error.

### L28 · Fallback Chains and Graceful Degradation

**Example:** Hotel price: live supplier API → second supplier → 2-hour-old cached price (marked `"price_source": "cache"`) → human agent. The payment step sees the flag and re-checks the live price before charging.

**Key facts**
- **Fallback chain** = an ordered, pre-planned list: primary tool → alternative tool → cached result → escalate. It must always end with a clear result or an escalation.
- **Pre-validate** every fallback before production: it works when the primary fails, meets the quality bar, and works under load. An untested fallback is a second point of failure.
- **Graceful degradation** = a useful but lower-quality result **with a visible signal** (flag, confidence, status) inside the output. Every downstream step must check the signal.
- **Silent degradation** = a weaker result with no signal. It is a failure, not a recovery.
- A partial result is OK only if it cannot cause harm downstream. A partial summary is fine; a partially validated payment is not.
- Low-risk, reversible action → accept the degraded result (with the signal). High-risk, irreversible action → escalate first.

**Exam traps**
- ❌ Return a cached price with no flag → ✅ Add a degradation flag.
- ❌ The chain loops A → B → A → B → ✅ It needs a final (terminal) step.

### L29 · Why Prompts Are Not Enough for High-Stakes Paths

**Example:** The prompt says "Never refund more than $500." After a long chat, an angry customer gets a $2,000 refund. Fix: code inside `issue_refund` blocks amounts over $500 and sends them to a manager.

**Key facts**
- A prompt rule is a **request** (probabilistic). Code is a **guarantee** (deterministic). **Prompts guide; code enforces.**
- The right question for high-stakes paths: not "How often does the model obey?" but "**Is a violation blocked by the design?**"
- Why prompt-only rules fail: **context drift** (early rules fade in long runs — the most common) · adversarial inputs · edge cases.
- A prompt alone is fine for tone, format, and low-stakes, reversible defaults. Code is required for money, personal data (PII), safety-critical steps, and irreversible actions (send, pay, delete).
- Where to enforce: money → hard limits + permission checks **before** execution · PII → masking/output filter **after** generation, before saving or sending · safety → hard blocks **before** execution.
- Defense in depth: prompt → validation code → runtime monitoring. Prompts still carry intent; they just cannot be the last line of defense.

**Exam traps**
- ❌ Fix the rule with CAPITAL letters, repetition, few-shot examples, or a stronger model → ✅ Add a code check in the tool.
- ❌ "We tested the prompt to 99.9% compliance" → ✅ On a high-stakes path, that is still a gap.

### L30 · Programmatic Validation and Enforcement Layers

**Example:** A refund agent. **Before:** Right format? Amount between $0 and $500? Is the user allowed to request refunds? **After:** Valid JSON? Refund ≤ booking total? Unusual compared with history? **During the run:** token budget, max 20 loops, only this customer's bookings.

**Key facts**
- **Pre-execution gate** (before acting): schema, range, permission. The cheapest place to stop bad input.
- **Post-execution gate** (after output, before saving or sending): output schema, business rules, anomaly detection.
- **Runtime checks** (while running): cost limits, iteration caps, scope enforcement. They catch overspending, loops, and slow scope creep.
- Pre and post gates catch different failures. Use both when errors have real-world impact.
- Response modes: **hard block** (non-negotiable rules: money limits, permissions, safety) · **soft warning** (log and continue; advisory) · **escalation trigger** (pause and ask a human; genuinely unclear cases).
- Four layers: prompt → pre-execution → post-execution → runtime.
- Calibration: too strict blocks good work (that is also a failure); too loose misses risky cases.

**Exam traps**
- ❌ Hard-block every booking over $1,000, even valid group bookings → ✅ Over-strict. Use a soft warning or an escalation.
- ❌ A $480 refund (limit $500) for a customer whose refunds are usually $25 → approve or block → ✅ Escalate to a human.

### L31 · Escalation Trigger Design — Valid vs. Invalid Patterns ⚠️ GAP FILL

*Not from my notes. Based on my D5 Lecture 115 notes and the official exam guide's escalation topics.*

**Example:** A help-desk rep. The customer says "I want to talk to a manager" → transfer now. The customer is angry but asks a simple question the rep can solve → solve it, and transfer only if they ask for a manager again.

**Valid triggers (observable facts)**
- The customer **explicitly asks for a human** → escalate immediately. Do not investigate first.
- A **policy gap or exception**: the policy is silent or unclear on this case (for example, a competitor price-match request when the policy covers only your own prices).
- **No progress:** repeated failure on the same subtask after real attempts.
- **Beyond authority:** a defined limit is crossed (refund over $500), or a high-impact, irreversible action needs approval.
- A **calibrated** score (checked against labeled data) below a set threshold → send to human review (see D5 L127–130).

**Invalid triggers (unreliable signals)**
- Negative **sentiment or frustration alone** — mood is not the same as case difficulty.
- The model's **self-reported confidence** ("rate yourself 1–10", "escalate if unsure") — the model can be confidently wrong.
- Complexity alone, time passed, or the number of tool calls.
- Vague rules like "use good judgment".

**Key facts**
- Too little escalation → risky wrong actions. Too much → the automation becomes useless.
- Good triggers are **specific and measurable**. Enforce critical ones in **code**, not only in the prompt (see L29).

**Exam traps**
- ❌ Angry customer → escalate at once → ✅ Acknowledge and try to solve. Escalate if they ask for a human or a valid trigger fires.
- ❌ Customer demands a human → investigate first → ✅ Escalate right away.

### L32 · Interruption Points and Review Workflows

**Example:** Wrong: the agent cancels 20 hotel rooms, then asks a manager to review. Right: the agent pauses **before** cancelling and sends: context (what and why) · one decision ("Approve cancelling 20 rooms with a $2,400 penalty? Yes/No") · what happens after yes and after no · a deadline (4 hours, then the duty manager). Progress is saved in the database, so the agent continues after approval.

**Key facts**
- The pause point decides whether a review adds any safety. Valid places: **before irreversible actions** (the most important) · at phase boundaries (where scope or authority changes) · when an anomaly is detected. Invalid: **after** the action.
- A good review message has 4 parts: **context · one specific decision · consequences (approve/reject) · deadline**. Raw JSON + "please review" → slow reviews and rubber-stamp approvals.
- **Save state outside the model** (database) before pausing: steps done, outputs, decisions. In-context state can be lost.
- Resume from the interruption point with the reviewer's decision. Do not rerun finished steps.
- **Synchronous** escalation = stop and wait (all remaining work needs the decision). **Async** escalation = queue the review and keep working on independent parts (needs more state tracking).
- Async needs a **timeout threshold + timeout action** (escalate further, abort, or apply a safe default). An unhandled timeout is a silent failure — the run shows "running" forever.

**Exam traps**
- ❌ Review after the cancellation or email → ✅ Pause before it.
- ❌ After approval, rerun all steps "to get fresh data" → ✅ Resume from the saved state.
- ❌ Stop the whole package for one approval when the other parts are independent → ✅ Async escalation.

---

**Note:** My D1-4 notes also include a Claude Code hooks lecture (PreToolUse, exit code 2, and more). That is Claude Code content (course D3), so it will be summarized with D3-4, not here.

---

## Section notes in this folder

- [D1-1 Summary — Agentic Architecture & Orchestration (Foundations)](1.1-agentic-architecture-and-orchestration.md)
- [D1-2 Summary — Task Decomposition & Planning](1.2-task-decomposition-and-planning.md)
- [D1-3 Summary — Multi-Agent Orchestration](1.3-multi-agent-orchestration.md)
- [D1-4 Summary — Reliability & Human Oversight](1.4-reliability-and-human-oversight.md)

[← Back to home](../README.md) · [Cheat sheet](../cheat-sheet.md)
