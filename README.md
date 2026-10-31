# Claude Certified Architect – Foundations (CCA-F) Study Notes

Study notes, domain summaries and a one-page cheat sheet for the **Claude Certified Architect – Foundations (CCA-F)** exam by Anthropic.

I used these notes to prepare for the exam, and I passed it in October 2026. I am sharing them so other people preparing for CCA-F can use them too.

> **Disclaimer:** These are my personal study notes. They are not official Anthropic material, and they do not contain any real exam questions. Always check the official Anthropic documentation for the latest details.

---

## How to use these notes

1. **Start with the [cheat sheet](cheat-sheet.md).** It covers all 5 domains on one page, including the top exam traps.
2. **Read the domain summary** (the `README.md` inside each folder) for a full picture of one domain.
3. **Go to the section notes** when you need more depth on a single topic.

**Legend used in the notes:**

| Mark | Meaning |
|---|---|
| ❌ → ✅ | A common wrong idea (exam trap) and the right idea |
| ⚠️ GAP FILL | Added from official docs or the exam guide, not from the course |
| ➕ | Checked against the official exam guide or docs |

---

## Domains

The notes follow the course order (D1 to D5). The official exam guide numbers some domains differently, so always go by the **domain name**.

| Folder | Domain | Official exam domain | Exam weight |
|---|---|---|---|
| [domain-1-agentic-architecture](domain-1-agentic-architecture/) | Agentic Architecture & Orchestration | Domain 1 | 27% |
| [domain-2-tool-design-and-mcp](domain-2-tool-design-and-mcp/) | Tool Design & MCP Integration | Domain 4 | 18% |
| [domain-3-claude-code-configuration](domain-3-claude-code-configuration/) | Claude Code Configuration & Workflows | Domain 2 | 20% |
| [domain-4-prompt-engineering](domain-4-prompt-engineering/) | Prompt Engineering & Structured Output | Domain 3 | 20% |
| [domain-5-context-and-reliability](domain-5-context-and-reliability/) | Context Management & Reliability | Domain 5 | 15% |

---

## Repository structure

```
claude-architect-cca-f-notes/
├── README.md
├── cheat-sheet.md
├── domain-1-agentic-architecture/
│   ├── README.md
│   ├── 1.1-agentic-architecture-and-orchestration.md
│   ├── 1.2-task-decomposition-and-planning.md
│   ├── 1.3-multi-agent-orchestration.md
│   └── 1.4-reliability-and-human-oversight.md
├── domain-2-tool-design-and-mcp/
│   ├── README.md
│   ├── 2.1-tool-design-fundamentals.md
│   ├── 2.2-building-mcp-servers.md
│   └── 2.3-mcp-production-and-built-in-tools.md
├── domain-3-claude-code-configuration/
│   ├── README.md
│   ├── 3.1-architecture-and-setup.md
│   ├── 3.2-claude-md-configuration.md
│   ├── 3.3-slash-commands-skills-and-subagents.md
│   └── 3.4-hooks-sdk-and-ci-cd.md
├── domain-4-prompt-engineering/
│   ├── README.md
│   ├── 4.1-effective-prompts.md
│   ├── 4.2-advanced-prompting-techniques.md
│   └── 4.3-structured-output-and-evaluation.md
└── domain-5-context-and-reliability/
    ├── README.md
    ├── 5.1-the-context-window.md
    ├── 5.2-multi-agent-failure-modes.md
    ├── 5.3-cross-session-knowledge-and-coherence.md
    └── 5.4-confidence-sampling-and-provenance.md
```

---

## Useful links

- [Anthropic documentation](https://docs.claude.com)
- [Claude Code documentation](https://docs.claude.com/en/docs/claude-code/overview)
- [Model Context Protocol](https://modelcontextprotocol.io)

---

## Contributing

Found a mistake or something out of date? Please open an issue or a pull request. Corrections are welcome.

## Author

**Deepak Donde** · Claude Certified Architect – Foundations
[LinkedIn](https://www.linkedin.com/in/deepak-donde-2710463a/)

If these notes helped you, please give the repo a ⭐.
