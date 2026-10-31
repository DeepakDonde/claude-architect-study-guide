# CCA-F: D3 Domain Summary — Claude Code Configuration & Workflows

**Exam weight:** 20%. **Numbering:** course D3 = official blueprint Domain 2 (Claude Code Configuration & Workflows). Some study websites call it Domain 3 — always go by the domain name.

**Coverage:** my four D3 note files, plus the Claude Code hooks lecture from my D1-4 notes (merged into 3.7). There is no D3 index file, so missing lectures could not be checked. One exam-guide topic missing from my notes was added as a gap fill (3.10). "1.6" is an added label for the unnumbered "Protected Paths & Deny Rules" section in my D3-1 notes.

**Correction to my notes:** PostToolUse cannot undo an action, but it **can** change what Claude sees — add context or replace the tool's output (see 3.7).

**Legend:** ❌ wrong idea (exam trap) → ✅ right idea · ⚠️ GAP FILL = not from my notes · ➕ = exam-guide or docs check (added)

| Part | Source notes | Topics | Gap fills |
|---|---|---|---|
| 1 · Architecture & Setup | D3-1 | 1.1–1.6 | — |
| 2 · CLAUDE.md Configuration | D3-2 | 2.1–2.6 | — |
| 3 · Commands, Skills, Subagents, Hooks & Plan Mode | D3-3 (+ D1-4 hooks lecture) | 3.1–3.10 | 3.10 ⚠️ |
| 4 · Hooks, Agent SDK & CI/CD | D3-4 | 4.1–4.5 | — |


---

## Part 1 · Architecture & Setup (1.1–1.6)

*From the D3-1 summary.*

### 1.1 · What Claude Code Is

**Example:** A chatbot is a friend on the phone who tells you how to fix a leaking pipe. Claude Code is a contractor you let into the house: it inspects, plans, fixes, tests, and tells you it is done.

**Key facts**
- Claude Code is an **agentic coding tool**. It reads your codebase, edits files, runs commands and tests, and handles Git (commits, branches, pull requests).
- Its loop: **describe the task → read context → plan → execute → check → repeat**.
- It runs in the **terminal, IDE, desktop app, and web**.
- You set the task and review the result. No copy-pasting of snippets.
- It reads **CLAUDE.md** for project rules and connects to outside tools through **MCP**.

**Exam traps**
- ❌ "Claude Code is a chat window that suggests snippets" → ✅ It takes real actions on your project.

### 1.2 · The Tool System: Reads, Bash, Edits, Web

**Example:** A worker in your workshop: eyes and reading glasses (Read, Grep, Glob), hands on the machines (Bash), a pen and eraser (Edit, Write), and a phone to the outside world (WebSearch, WebFetch).

| Family | Tools | Job |
|---|---|---|
| Reads | Read, Grep, Glob | Look and search; never change files (auto-approved) |
| Bash | Bash | Run commands: tests, installs, Git (usually asks) |
| Edits | Edit, Write | Change or create files (asks in default mode) |
| Web | WebSearch, WebFetch | Look things up online |

**Key facts**
- **Grep = search inside files** (ripgrep, regex). **Glob = find files by name or path pattern.** **Read = open one file** (with line numbers; also images, PDFs, notebooks).
- **Edit** = an exact string replacement. It needs a **prior Read** of that file, and the match must be **unique**. **Write** = create or fully overwrite a file.
- **WebSearch finds links; WebFetch reads a page.** Two separate steps.
- Tool names are **exact, case-sensitive strings**. You reuse them in permission rules, subagent tool lists, and hook matchers.
- Prefer the built-in tools over Bash (Grep, not `grep`; Read, not `cat`): faster, with fewer permission prompts.
- ➕ Exam guide: when Edit cannot find a unique match, fall back to **Read + Write** (rewrite the file). Explore **step by step** (Grep or Glob first, then Read only what you need) instead of reading every file up front.

**Exam traps**
- ❌ Use Glob to find every caller of a function → ✅ Grep (the callers are inside files). Glob is for names like `**/*.test.tsx`.
- ❌ Edit a file Claude has not opened yet → ✅ Read it first.
- ❌ Expect full page text from WebSearch → ✅ Follow up with WebFetch.

### 1.3 · Starting a Project: Setup and /init

**Example:** A new developer joins. They get a desk and keys (install, log in), read the code, and receive a team handbook. `/init` lets Claude write that handbook (CLAUDE.md) for itself.

**Key facts**
- Setup: install, `cd` into the project, run `claude`, log in (paid plan or API credits). The native installer does not need Node.js (only the npm route does).
- **Explore first:** ask Claude to explain the codebase before big changes. Claude reads files **on demand** — you don't paste them in.
- **/init** analyzes the repo and writes a starter **CLAUDE.md** (commands, structure, conventions). If one already exists, it **suggests improvements** instead of overwriting.
- **Review and edit** the generated file, then **commit it** so the whole team shares it.
- CLAUDE.md loads at the **start of every session**. Keep it short — bloated files get ignored.
- Give location hints ("the bug is in src/auth/login.ts around line 45") and break big tasks into steps.

**Exam traps**
- ❌ Run /init and commit the file without reading it → ✅ Review and edit it first.
- ❌ Paste all the important files into the chat → ✅ Claude reads on demand. Just point to the area.

### 1.4 · Managing Context: /context, /compact, /clear

**Example:** Claude's desk. Everything it works with must fit on it. When it is too full, Claude knocks things over (context rot). `/compact` tidies the desk into a summary; `/clear` sweeps it clean.

| You want to… | Command |
|---|---|
| See how full the window is | `/context` |
| Keep working on the same task but free space | `/compact` (you can add a focus: "/compact keep the API decisions") |
| Start unrelated work | `/clear` (permanent — no recovery) |
| Go back to an earlier point | `/rewind` (or press Esc twice) |

**Key facts**
- The window holds the conversation, tool calls and outputs, files read, the system prompt, and CLAUDE.md. Near the limit, quality drops (**context rot**: repeats rejected ideas, forgets early constraints).
- **Auto-compact** fires near the limit, when the model is weakest. **Compact manually** at a natural break instead.
- After `/clear` or `/compact`, **CLAUDE.md still reloads**. Put lasting rules there, not in chat.
- Add context **precisely**: @-mention files, read big files by line range or one function, and disable unused MCP servers.

**Exam traps**
- ❌ "/clear deletes CLAUDE.md" → ✅ It only clears the conversation.
- ❌ Keep going until auto-compact fires → ✅ Run /compact earlier.

### 1.5 · Permission Modes

**Example:** How much freedom you give a contractor: "ask before anything" (default), "edit freely, but ask before using machines" (acceptEdits), "walk around and write a plan only" (plan), "work alone while a safety inspector watches" (auto), "here are all the keys" (bypassPermissions — test sites only).

| Mode | File edits | Bash commands | Notes |
|---|---|---|---|
| default | Ask | Ask | Reads are auto-approved. The safe baseline. |
| acceptEdits | Auto | Risky ones still ask (npm test, git push, curl) | Not "approve everything" |
| plan | None | None | Read-only; proposes a plan |
| auto | Auto | Auto, but a classifier blocks dangerous actions | Needs organization and model support |
| bypassPermissions | Auto | Auto, no checks | Isolated containers / VMs only |

**Key facts**
- Change the mode with **Shift+Tab** (default → acceptEdits → plan), the `--permission-mode` flag, or `defaultMode` in settings.json — **not** by asking Claude in chat.
- Modes are the baseline. **Permission rules** (deny / ask / allow) sit on top (see 1.6). Deny and ask rules apply in every mode; allow rules do nothing in bypass.

**Exam traps**
- ❌ "Prompts are annoying, so use bypassPermissions on my work laptop" → ✅ Use auto mode, or allow + deny rules. Bypass only in isolated environments.
- ❌ Explore an unfamiliar repo in acceptEdits → ✅ Use plan mode.

### 1.6 · Permission Rules: Deny Rules and Protected Paths

**Example:** The permission mode is the contractor's general freedom. Permission rules are specific locks: "never open the safe" (deny), "always ask before taking the car" (ask), "the garden tools are free to use" (allow).

**Key facts**
- Rules live in **settings.json** under `permissions`, in `allow`, `ask`, and `deny` lists, written as **Tool(specifier)**: `Bash(npm run lint:*)`, `Read(./.env)`, `Edit(*.env)`, `WebFetch(domain:example.com)`, `mcp__server__tool`.
- Order of checking: **deny → ask → allow**. The first match wins, and specificity does not matter. **Deny always wins** — it cannot have allow exceptions.
- Settings levels (strongest first): **managed (enterprise) > command-line flags > local (`.claude/settings.local.json`) > project (`.claude/settings.json`) > user (`~/.claude/settings.json`)**. A deny at any level wins.
- See and manage rules in a session with **/permissions**.
- For secrets, add **Read deny rules** (`Read(./.env)`, `Read(./.env.*)`). This also keeps them **out of Claude's context**.
- Protected paths (like `.git` and Claude's own config) are never auto-approved, except in bypass mode.
- Bash allow rules are best-effort pattern matching, not a security wall. Use deny rules plus a sandboxed environment (and hooks, see 4.2).

**Exam traps**
- ❌ "A narrow allow rule beats a broad deny rule" → ✅ Deny always wins.
- ❌ Rely on a Bash allow-list as your security boundary → ✅ Deny rules + a restricted environment + hooks.


---

## Part 2 · CLAUDE.md Configuration (2.1–2.6)

*From the D3-2 summary.*

### 2.1 · The CLAUDE.md Hierarchy

**Example:** Three rulebooks: your personal sticky notes that you carry to every job (user), the team handbook in the shared office (project), and a sign taped to one machine (directory).

| Level | File | Shared via git? | Use it for |
|---|---|---|---|
| User | `~/.claude/CLAUDE.md` | No — only you | Your personal preferences, in all projects |
| Project | `./CLAUDE.md` or `./.claude/CLAUDE.md` | Yes — the whole team | Team standards, build and test commands |
| Directory | for example `packages/api/CLAUDE.md` | Yes | Rules for one folder only |

*(An enterprise / managed policy level sits on top and cannot be overridden.)*

**Key facts**
- All files found are **joined together (concatenated)** in context, broad → specific: user → project → directory. It is not a strict override.
- Directory files load **lazily** — only when Claude reads files in that folder.
- ➕ Import other files with **`@path/to/file`** inside CLAUDE.md (the exam calls this "@import"). It keeps CLAUDE.md modular.
- **/memory** shows which memory files are loaded (a diagnostic). It does not "turn on" any file.
- CLAUDE.md loads automatically at the start of every session.

**Exam traps**
- ❌ The team lead puts team rules in her `~/.claude/CLAUDE.md`, and teammates don't get them → ✅ Move them to the project CLAUDE.md and commit it.
- ❌ "Run /memory to activate a file" → ✅ Files load by location. /memory only shows them.

### 2.2 · Precedence and Conflicts

**Example:** CLAUDE.md is like handing someone several instruction sheets at once — if two disagree, you can't be sure which one they follow. settings.json is a law book with a clear chain of command.

| | CLAUDE.md | settings.json |
|---|---|---|
| How files combine | Concatenated (all stacked together) | Strict order of authority |
| On a conflict | Soft weighting; may be picked at random | A guaranteed winner |
| Use it for | Guidance and conventions | Rules that MUST hold |

**Key facts**
- Later-read (more specific) instructions have only a **soft** advantage. The docs say that on a contradiction, Claude **may follow either one**.
- The right approach: **avoid conflicts**. Child files should **add** rules, not repeat a root rule with a different value.
- Rules that must be enforced (for example, "never read .env") → a **settings.json deny rule or a hook**, not CLAUDE.md.
- CLAUDE.md is sent on every turn, so keep it small.

**Exam traps**
- ❌ "User file says tabs, project file says spaces — the project file always wins" → ✅ Neither is guaranteed. Remove the conflict, or enforce the rule with settings or hooks.
- ❌ "CLAUDE.md works like CSS: the most specific rule always wins" → ✅ Only a soft weighting.

### 2.3 · Scoped Rules: Subdirectory CLAUDE.md vs. .claude/rules/

**Example:** A sign posted in one room (subdirectory CLAUDE.md) vs. a rule that follows a type of item anywhere: "wear gloves when you handle any chemical bottle, in any room" (a path-scoped rule).

**Key facts**
- **Subdirectory CLAUDE.md** = scope by **folder**. It loads lazily and adds to the root file.
- **`.claude/rules/*.md`** = one topic per file, found automatically (no import needed).
  - **No `paths:`** in the frontmatter → loads every session, like CLAUDE.md.
  - **With `paths:` globs** (for example `"**/*.test.tsx"` or `"terraform/**/*"`) → loads **only when Claude works on matching files**.
- Use a **folder file** for one package's conventions. Use a **path rule** for conventions spread over many folders (for example, all test files).
- Path rules save context and tokens, because they stay out during unrelated work.
- Split a huge root CLAUDE.md into topic files in `.claude/rules/` (testing.md, api-conventions.md, deployment.md).
- After /compact, the **root** CLAUDE.md is re-loaded. Nested files and path rules come back when a matching file is read again.

**Exam traps**
- ❌ Put "all test files use Jest" in `packages/api/CLAUDE.md` → ✅ A path rule like `**/*.test.*`, so it reaches test files everywhere.
- ❌ "My subdirectory CLAUDE.md is broken — it didn't load at the start" → ✅ It loads lazily. Open a file in that folder, then check with /memory.

### 2.4 · Excluding Files and Content

**Example:** Ways to keep someone out of a drawer: a "don't browse here" note (.gitignore), a polite sticky note that is often ignored (.claudeignore), a real lock (permissions.deny), and a guard at the door (a PreToolUse hook).

| Mechanism | Goal | Reliable? |
|---|---|---|
| Respect .gitignore | Less noise in the file picker and search | Not a security control |
| .claudeignore | — | ❌ Not a native feature; Claude can still read the file |
| permissions.deny (Read rules) | Block access and keep it out of context | ✅ Good (a few edge cases) |
| claudeMdExcludes | Skip other teams' CLAUDE.md files | ✅ For noise, not for secrets |
| PreToolUse hook | Hard block on paths | ✅ The strongest |
| Secrets manager | Keep secrets out of the repo | ✅ Removes the risk |

**Key facts**
- Two different goals: **clean context** vs. **protecting secrets**. Choose the right tool for each.
- Deny-rule edge cases: absolute-path bypass bugs, and the IDE can send an **open file's** content anyway — don't keep secret files open.

**Exam traps**
- ❌ Put .env in .claudeignore → ✅ Use permissions.deny (`Read(./.env)`) or a hook.
- ❌ "Respect .gitignore" protects .env → ✅ It only affects the file picker and search.

### 2.5 · Instruction Design: What Claude Actually Follows

**Example:** "Cook until done" can give you anything. "Bake at 180°C for 25 minutes" gives one result. Write the second kind.

**Key facts**
- Reading ≠ following. Claude may skip content it thinks is irrelevant, so **noise weakens** your real rules.
- **Specific and checkable** beats vague. Test: can you check it in the output? ("Max function length 30 lines", not "keep functions small".)
- Say what **to do**. Every prohibition needs a **replacement** ("never use `any`; define types or use `unknown`").
- Use **emphasis sparingly**. If everything is IMPORTANT, nothing is.
- Keep the root file **short** (well under ~300 lines). Move task-specific rules to `.claude/rules/`.
- Two useful kinds of rule: **output constraints** (binary, checkable) and **procedural steps** (do X before Y).
- When you correct Claude repeatedly, turn the correction into an explicit rule.
- CLAUDE.md is **guidance, not a guard**. Back hard rules with linters, hooks, or settings.

**Exam traps**
- ❌ Mark every rule CRITICAL → ✅ Keep emphasis for 2–3 rules.
- ❌ "More detail in the root file = better adherence" → ✅ Bloat lowers adherence.

### 2.6 · Common CLAUDE.md Patterns for Teams

**Example:** CLAUDE.md is the onboarding notes for a very fast junior developer who forgets everything between sessions.

**Key facts**
- Good team files are **short, blunt, and heavy on "Do NOT"** (the mistakes that already cost the team time).
- Common sections: tech stack · **build / test / lint commands** · architecture (where code goes) · style · testing conventions · gotchas.
- With the **exact CI commands** in the file, Claude can check its own work before saying "done".
- Keep a **tracked** project file (committed) and a **personal** file (gitignored, for example CLAUDE.local.md). No personal paths in the shared file.
- **Context cascade** for monorepos: the root file holds global rules and pointers; subfolder files hold local commands and conventions. Start Claude in the subfolder you work in, so its rules load.
- In multi-agent setups, CLAUDE.md is the shared "constitution" that every subagent reads.

**Exam traps**
- ❌ A long essay on clean-code principles → ✅ A blunt "Do NOT" list of the real repeated mistakes.
- ❌ Commit `/Users/yourname/...` paths in the project file → ✅ Put them in the personal (gitignored) file.


---

## Part 3 · Commands, Skills, Subagents, Hooks & Plan Mode (3.1–3.10)

*From the D3-3 summary.*

### 3.1 · Custom Slash Commands

**Example:** A rubber stamp with a fill-in blank. You carve the long instructions once, then just press `/pr`. `$ARGUMENTS` is the blank.

**Key facts**
- A command = a **Markdown file**. The filename is the command name (`commit.md` → `/commit`); the body is the prompt.
- Scopes: **`.claude/commands/`** (project — committed, for the whole team) · **`~/.claude/commands/`** (personal — all your projects).
- Frontmatter: `description` (shown in the / menu; helps matching) · `argument-hint` (shows what to type) · `allowed-tools` (limits tools — a review command with Read, Grep, Glob can never write) · `model` (for example, haiku for cheap tasks).
- Arguments: **`$ARGUMENTS`** = everything typed after the command; **`$1`, `$2`** = positional values.
- Subfolders create namespaces (`frontend/component.md` → `/frontend:component`).
- A line starting with **`!`** runs a shell command and inserts its output (for example, `` !`git diff --cached` ``).
- A command is run explicitly with `/name`. A skill can also be started by Claude itself (3.3). In newer Claude Code, commands and skills are merged — both create `/name` (3.4).

**Exam traps**
- ❌ A team command kept in `~/.claude/commands/` → ✅ Move it to `.claude/commands/` and commit it.
- ❌ A prompt line saying "don't edit files" in a review command → ✅ Restrict it with `allowed-tools`.

### 3.2 · Command Design and Team Distribution

**Example:** A tool starts in your own toolbox (personal), moves to the shared workbench (project), and finally ships as a boxed kit with automatic updates (plugin).

**Key facts**
- **Rule of two:** done the same task more than twice → make it a command.
- Design: **one purpose per command**, scoped `allowed-tools`, reusable arguments, a cheap model for simple tasks, and your project's own conventions.
- Distribution ladder: **personal → project (committed) → plugin**. Each step adds durability *and* friction.
- **Plugin** = one versioned, installable bundle of commands, skills, agents, hooks, and MCP servers, shared through **marketplaces**.
- One-liner: **skills are the unit of capability; plugins are the unit of distribution.**
- Keep experiments **local** (`.claude/settings.local.json`, gitignored) until they are proven.

**Exam traps**
- ❌ Copy the `.claude/` folder by hand into 12 repos → ✅ A plugin (one versioned source, automatic updates).
- ❌ One mega-command that does everything → ✅ One purpose per command.

### 3.3 · What Skills Are and When to Build Them

**Example:** A shelf of playbook binders. The worker reads only the label on each spine. When a matching job comes, they open that one binder — and the appendix only if needed.

**Key facts**
- A skill = a **folder with SKILL.md** (+ optional scripts and reference files). Scopes: `.claude/skills/` (project) · `~/.claude/skills/` (personal).
- SKILL.md = frontmatter (**name + description = when to use it**) + body (**how to do it**).
- **Progressive disclosure:** (1) at startup only the name + description load → (2) the full body loads when the skill is relevant → (3) bundled files load only when used. So many installed skills cost almost nothing.
- Triggering: **Claude invokes it automatically** by matching the description, or you type `/skill-name`. The **description is load-bearing** — a vague one never fires.
- Two kinds: **capability uplift** (new abilities, usually with scripts — for example, making PDFs) · **encoded preference** (your team's way of doing something — for example, a commit format; usually just text).
- **Skills vs. CLAUDE.md:** skills load **on demand** for specific tasks; CLAUDE.md is **always loaded** for universal standards.

**Exam traps**
- ❌ A skill never fires; its description says "helps with reports" → ✅ Rewrite it as a routing rule ("Use when the user asks for X").
- ❌ "A skill can only run if Claude auto-detects it" → ✅ You can always type `/skill-name`.

### 3.4 · Authoring Skills

**Example:** Writing a good playbook binder: a clear spine label (frontmatter), a lean main page (body), and the heavy reference material in the appendix (bundled files).

**Key facts**
- Folder: `SKILL.md` (required) + `scripts/` + `reference/` (optional). No build step, no manifest.
- Key frontmatter: `name` · **`description`** (write it as "Use when…") · **`allowed-tools`** (limits tools *and* pre-approves them, so no permission prompts) · `argument-hint` (asks for missing input) · **`disable-model-invocation: true`** (runs only when you type /name) · `paths` (path-scoped).
- ➕ **`context: fork`** (with `agent:`, for example Explore) runs the skill in an **isolated subagent context**. Verbose output, like a codebase analysis, stays out of the main conversation.
- Bundle large references freely — **no token cost until a file is read**. Scripts cost only their **output**.
- Keep the body lean (**under ~500 lines**) and move detail into bundled files. Use forward slashes and descriptive file names.
- **skill-creator** builds a skill with you. Commands and skills are merged: `.claude/commands/deploy.md` and `.claude/skills/deploy/SKILL.md` both give `/deploy`.

**Exam traps**
- ❌ A deploy skill runs on its own when you didn't want it → ✅ `disable-model-invocation: true`.
- ❌ One 1,200-line SKILL.md → ✅ A ~200-line body + reference files.
- ❌ An analysis skill floods the main chat with output → ✅ `context: fork`.

### 3.5 · Creating Custom Subagents

**Example:** The main session is the project lead; a subagent is a contractor for one job. The contractor works in a separate room with only the tools you give them, and returns a one-page report — not their scratch notes.

**Key facts**
- A subagent = a Claude instance with its **own context window, system prompt, tools, and permissions**. The parent gets back **only the summary**.
- It starts **fresh**: its system prompt + the delegation message + the project CLAUDE.md + preloaded skills. **No parent history**, files, or earlier tool calls.
- Defined in **`.claude/agents/<name>.md`** (project) or `~/.claude/agents/` (user). The body is the system prompt; only `name` and `description` are required. Manage them with **/agents**.
- Key fields: `tools` / `disallowedTools` · `model` · `permissionMode` · `skills` (preload) · `mcpServers` · `memory` (opt-in persistence) · `maxTurns` · `isolation: worktree` (its own git worktree, for parallel edits).
- A subagent can be **more restricted** than the parent, **never more permissive** (a parent in acceptEdits or bypass wins).
- Subagents are **heavy** (a full context each). Use them when isolation pays off; **3–5 at once** is the sweet spot.
- A skill runs in the main context; a subagent runs in a separate, isolated context.

**Exam traps**
- ❌ "The subagent knows what we decided 40 messages ago" → ✅ Put it in the delegation message.
- ❌ Two subagents editing the same repo in parallel keep clashing → ✅ `isolation: worktree`.

### 3.6 · Designing Subagents for Delegation

**Example:** A good engineering manager writes a clear job posting (description), a precise work order, and a required report format, gives only the access needed, and runs independent specialists in parallel before combining their reports.

**Key facts**
- **Auto-delegation runs on the description.** Write a trigger: "Use after any code change", "Use proactively for security scans".
- Three ways to invoke: automatic (description match), explicit ("use the code-reviewer agent"), or @-mention.
- Best use: **high-output, focused work** where only the result matters (test runs, log crawls, big searches).
- Define an **output contract** (numbered, structured). It is the biggest quality lever.
- **One responsibility** per subagent. Start with a code reviewer.
- A tight tool scope also protects against **prompt injection** in content the subagent reads.
- Parallel fan-out = **hub-and-spoke**: the parent combines the results; the spokes don't talk to each other.
- Limits: **no nested subagents** · it forgets everything unless `memory` is set · parallel runs raise total tokens. **Fork** is the exception: it inherits the current conversation.
- Subagents = workers that don't need to talk; **agent teams** = workers that must coordinate.

**Exam traps**
- ❌ "A subagent can start its own sub-subagent" → ✅ No nesting. Chain from the main conversation.
- ❌ The reviewer returns a different format every time → ✅ Add an output contract.

### 3.7 · Hook Lifecycle Events

**Example:** Sensors at fixed doorways that run your script: the front door (SessionStart), the work-order box (UserPromptSubmit), a gate before each machine (PreToolUse), an inspector after it (PostToolUse), the exit checkpoint (Stop), and the back door (SessionEnd).

| How often it fires | Events | Can it block? |
|---|---|---|
| Once per session | SessionStart, SessionEnd | No |
| Once per turn | UserPromptSubmit, Stop | Yes (reject the prompt / refuse to stop) |
| Every tool call — before | PreToolUse | **Yes — the gate** |
| Every tool call — after | PostToolUse | No — it cannot undo |
| Special | SubagentStop (yes), PreCompact (yes, in newer versions), Notification (no) | — |

**Key facts**
- A hook = **your own code** that always runs at an event, so it is **deterministic**. CLAUDE.md is only guidance.
- Configured in `.claude/settings.json` (team) or `.claude/settings.local.json` (personal). Hooks fire on every surface.
- The **matcher** is the exact tool name for tool events (`Bash`, `Edit|Write`).
- The core five: PreToolUse, PostToolUse, UserPromptSubmit, SessionStart, Stop.
- Memory line: **Pre-tool guards. Post-tool reacts.**
- ➕ PostToolUse cannot undo an action, but it **can change what Claude sees**: add context, or **replace the tool's output** (for example, normalize dates and status codes from different MCP tools, or redact secrets). The exam guide uses this for data normalization.
- From my D1-4 hooks notes: `PermissionRequest` denies with a **decision object** (not exit 2), and a `UserPromptSubmit` block message is shown to the **user only**.

**Exam traps**
- ❌ Block `rm -rf` with a PostToolUse hook → ✅ PreToolUse (by PostToolUse time, it already ran).
- ❌ Load sprint notes at the start with PreToolUse → ✅ SessionStart.

### 3.8 · Plan Mode: When to Plan First

**Example:** A builder walks the site and hands you a written plan before touching a single brick.

**Key facts**
- Plan mode is **read-only**: Claude explores and proposes a plan, with **no edits or commands** until you approve. Enter it with **Shift+Tab** or `--permission-mode plan`.
- Use it for big, unfamiliar, or risky work: multi-file changes, refactors, architecture choices, several possible approaches. **Skip it** for small, obvious edits.
- Its value: you catch a wrong approach **on paper**, while it is cheap to fix.
- ➕ Exam-guide examples: plan mode for a library migration touching 45+ files or a microservice restructure; direct execution for a one-file bug fix with a clear stack trace. The built-in **Explore subagent** (read-only) does verbose discovery and returns a summary, so your main context stays clean. Common combo: **investigate with plan mode / Explore, then implement directly**.

**Exam traps**
- ❌ Plan mode for a one-character typo → ✅ Just make the edit.
- ❌ "In plan mode, Claude can edit if it is sure" → ✅ Never — read-only until you approve.

### 3.9 · Iterating on a Plan: Approve, Edit, Re-plan

**Example:** Reviewing a contractor's proposal: you sign it, cross out step 3 and fix it, or say "rethink the whole approach".

| Choice | Use when |
|---|---|
| Approve | The plan is right → Claude leaves plan mode and executes it |
| Edit | Mostly right → adjust some steps, then approve |
| Re-plan | The whole approach is wrong → say why |

**Key facts**
- A plan is a **draft**. Steering it is the **cheapest** moment to add constraints ("don't touch the API layer", "use the existing helper").
- When you ask for a re-plan, give **specific feedback**.

**Exam traps**
- ❌ Approve a flawed plan and fix the mess later → ✅ Edit or re-plan first.

### 3.10 · Iterative Refinement Techniques ⚠️ GAP FILL

*Not from my notes (there is no D3 index to check for missing lectures). Based on the official exam guide's Claude Code topics.*

**Example:** Ordering a custom cake. You show the baker 2–3 photos of cakes you like (examples). The baker asks about allergies, guests, and flavours before baking (interview). A taste test decides when it is right (tests).

**Key facts**
- When prose instructions give inconsistent results, show **2–3 concrete input → output examples**.
- **Test-driven iteration:** write the tests first (expected behaviour, edge cases, performance), then share the **test failures** to guide each fix.
- **Interview pattern:** ask Claude to **ask you questions first**, to surface things you didn't think of (cache invalidation, failure modes). Best in unfamiliar areas.
- To fix an edge case, give one **specific test case**: an example input and the expected output (for example, null values in a migration script).
- Problems that **interact** → describe them all in **one detailed message**. **Independent** problems → fix them **one at a time**.

**Exam traps**
- ❌ Keep rewording the prose description → ✅ Give concrete input/output examples.
- ❌ Fix interacting bugs one by one in separate messages → ✅ One message that covers all of them.


---

## Part 4 · Hooks, Agent SDK & CI/CD (4.1–4.5)

*From the D3-4 summary.*

### 4.1 · Four Common Hook Events

**Example:** Checkpoints around a worker: one before each tool, one after each tool, a pager for when the worker needs you, and the exit door.

| Event | Typical job | Can it block? |
|---|---|---|
| PreToolUse | Safety gate: stop dangerous actions | Yes |
| PostToolUse | Format, lint, or log after each edit | No — cannot undo |
| Notification | Alert you when Claude waits for permission or goes idle | No |
| Stop | "Not done until the tests pass" | Yes — refuses to end the turn |

**Key facts**
- To **intervene** → PreToolUse or Stop. To **observe or react** → PostToolUse or Notification.
- Notification's matcher is a sub-trigger, not a tool name: `permission_prompt`, `idle_prompt`, `auth_success`.

**Exam traps**
- ❌ Get a phone alert when Claude waits for permission with PostToolUse → ✅ Notification (matcher `permission_prompt`).

### 4.2 · Implementing Hooks: Exit Codes and Gotchas

**Example:** A bouncer gets a note about each guest (JSON on stdin) and answers with a hand signal: thumbs up (exit 0) = let them in; fist (exit 2) = block and say why (stderr). Any other gesture (exit 1) is just mumbling — nobody is stopped.

| Exit code | Effect |
|---|---|
| 0 | Proceed. stdout can carry a JSON decision. |
| **2** | **Block** (on blocking events). stderr goes to Claude as the reason. |
| Any other (including 1) | Non-blocking error. The action still happens. |

**Key facts**
- **Exit 1 blocks nothing.** To enforce a rule, exit 2.
- Use **exit codes OR JSON**, not both (on exit 2, stdout is ignored). JSON with exit 0 can return `permissionDecision` (allow / deny / ask) and `updatedInput` (rewrite the tool's arguments).
- Config nests **event → matcher → handler**. The matcher is a **case-sensitive** tool name (regex allowed; empty = all tools).
- Handler types: command (default), http, prompt, mcp_tool, agent.
- Use **`$CLAUDE_PROJECT_DIR`** for paths, because the hook's working folder can differ.
- A PreToolUse **deny holds even in bypassPermissions**. Hooks only tighten permissions; they never loosen them.
- Gotchas: a **Stop hook that exits 2 can loop forever** (check `stop_hook_active`) · an `echo` in your shell profile can break JSON output · `bash` ≠ `Bash` · the script must be executable (`chmod +x`) · no secrets in hooks · slow hooks block the session.

**Exam traps**
- ❌ exit 1 to block .env edits → ✅ exit 2.
- ❌ JSON on stdout plus exit 2 → ✅ Choose one channel.

### 4.3 · Real-World Hook Applications

**Example:** A security camera (logging — it watches), a cleaner after each job (linting — it reacts), and a locked gate (safety net — it blocks).

| Goal | Event | Blocks? |
|---|---|---|
| Audit log of everything Claude does | PostToolUse / Stop | No |
| Format and lint every edit | PostToolUse (Edit / Write) | No |
| Prevent dangerous actions (rm -rf, force push, reading .env) | PreToolUse (Bash, file tools) | Yes |
| Alert you when Claude needs you | Notification | No |
| "Tests must pass before done" | Stop | Yes |

**Key facts**
- The deciding question: **prevent it → PreToolUse; react to it → PostToolUse; just watch it → logging.**
- Put team guardrails in the committed **`.claude/settings.json`**, so everyone gets them.
- ➕ Exam guide (Agent SDK): use a tool-call interception hook to **block policy breaks** (for example, refunds over $500) and **redirect to a human**. Hooks give guarantees; prompts don't.

**Exam traps**
- ❌ Auto-format code with a PreToolUse hook → ✅ PostToolUse (after the edit).

### 4.4 · The Claude Agent SDK: Session Control

**Example:** Save files in a video game. Continue = load the latest save. Resume = load a specific named slot. Fork = copy a slot to try another path without losing the original.

| Mechanism | Finds the session by | Use when |
|---|---|---|
| Continue (`continue: true`) | The most recent session in this folder | One conversation at a time |
| Resume (`resume: <id>`) | A specific session ID | Many conversations |
| Fork (`resume` + `forkSession: true`) | A resumed session → a new ID | Try another approach; the original stays |

**Key facts**
- The Agent SDK runs **Claude Code's agent loop from your own code** (Python or TypeScript). `query()` streams back messages. (It was formerly the Claude Code SDK.)
- One `query()` already runs **many turns** and handles permission prompts inside the loop.
- Get the ID from `session_id` on the **result message** (present on success and on error).
- Sessions are saved as `.jsonl` files under `~/.claude/projects/`.
- In unattended runs, always set **`max_turns`** and an explicit **permission mode**.
- ➕ CLI versions: `claude -c` (continue) · `claude --resume <id or name>` · `--fork-session`. Exam guide: if files changed a lot, **a new session with a short structured summary** can work better than resuming with **stale tool results**. When you resume, **tell Claude which files changed**.

**Exam traps**
- ❌ 50 user conversations, so use continue → ✅ Resume each one by ID.
- ❌ Overwrite the session to test a risky idea → ✅ Fork it.

### 4.5 · Claude Code in CI/CD: the -p Flag

**Example:** Interactive mode is a chef who keeps asking "taste this? okay to plate?". `-p` is a vending machine: one request in, one result out, no questions.

| Flag | Why it matters in CI |
|---|---|
| `-p` / `--print` | **Required.** Non-interactive: runs to completion, prints the result, exits. Without it, the job hangs on the first prompt. |
| `--output-format json` | A machine-readable result for the next pipeline step |
| `--max-turns N` | Stops endless loops (print mode only; no default) |
| `--allowedTools` / a permission policy | Limits what an unattended run can do |
| ➕ `--json-schema '{…}'` | Validated JSON that matches your schema (for example, findings to post as PR comments) |

**Key facts**
- `--dangerously-skip-permissions` only in **isolated containers**. Permission-prompt hooks don't fire in `-p` mode — use PreToolUse hooks or `--allowedTools`.
- **`-p` vs. the Batch API:** a blocking pre-merge check needs `-p` (it answers now). The Batch API is 50% cheaper but asynchronous (up to 24 hours) — only for offline jobs like overnight audits.
- The official GitHub Action wraps `claude -p`.
- ➕ Exam guide: put CI context in **CLAUDE.md** (testing standards, fixtures, review criteria). Review with an **independent instance** — the session that wrote the code is weaker at reviewing its own changes. When re-reviewing after new commits, include the **earlier findings** and ask only for **new or still-open** issues. Give the **existing test files**, so new tests don't duplicate them.

**Exam traps**
- ❌ A CI job hangs forever, so add retries → ✅ Add `-p`.
- ❌ Use the Batch API for a blocking PR gate → ✅ Use `-p`.

---

## Section notes in this folder

- [D3-1 Summary — Claude Code Architecture & Setup](3.1-architecture-and-setup.md)
- [D3-2 Summary — CLAUDE.md Configuration](3.2-claude-md-configuration.md)
- [D3-3 Summary — Slash Commands, Skills, Subagents, Hooks & Plan Mode](3.3-slash-commands-skills-and-subagents.md)
- [D3-4 Summary — Hooks, Agent SDK & CI/CD](3.4-hooks-sdk-and-ci-cd.md)

[← Back to home](../README.md) · [Cheat sheet](../cheat-sheet.md)
