# The Control Plane — Configuration Files in an LLM Agent

How files like `AGENTS.md`, `CLAUDE.md`, `MEMORY.md`, skills, slash commands, hooks, MCP servers, and `settings.json` plug into the agent architecture.

> Companion files:
> - [`agent-architecture.md`](./agent-architecture.md) — the underlying architecture this configures
> - [`control-plane.svg`](./control-plane.svg) — visual diagram with the configuration plane overlaid
> - [`formal-model.md`](./formal-model.md) — formal model
> - [`study-guide.md`](./study-guide.md) — where to learn more
> - [`sources.md`](./sources.md) — every source cited here, in one place

---

## 1. The big idea

Configuration files are **not a new architectural layer**. They're a *control plane*, a parallel set of artifacts that **parameterize** the existing layers (interface, orchestrator, LLM, memory, tools, environment) without changing the agent's code.

The terminology is borrowed from networking and distributed systems: the **data plane** processes traffic at runtime, while the **control plane** is everything that *configures* what the data plane does (routes, policies, ACLs) without participating in the per-packet path. The split is standard in software-defined networking and Kubernetes. ([§7](#7-why-control-plane) explains the analogy in more detail.)

A useful teaching framing — *not* a standard literature term, just a way these notes organize the same material — is that the agent has **three knobs you can turn from outside**:

1. **What it knows** → the *memory* layer (facts, history, docs).
2. **How it acts** → the *reasoning* layer (instructions, conventions, style).
3. **What it can do** → the *tool* and *orchestrator* layers (tools, permissions, hooks, runtime limits).

Every config file fits one of these knobs.

---

## 2. The mapping — file → layer

### → Reasoning layer (becomes part of the system prompt)

| File | Purpose |
|---|---|
| [`AGENTS.md`][agentsmd] / `CLAUDE.md` / `.cursorrules` | Project-level conventions, dos/don'ts, output style. Loaded once at session start and merged into the system prompt. |
| `.claude/agents/*.md` | Subagent definitions — system prompt, tool allow-list, model. Loaded only when that subagent is spawned. See [Claude Code subagent docs][cc-subagents]. |
| `commands/*.md` (slash commands) | User-message templates. Expand into the user turn when typed. See [Claude Code slash commands][cc-slash]. |
| `skills/*` (body) | Capability instructions. Lazily injected when the skill is invoked. See [Claude Code skills][cc-skills]. |

These shape *how the LLM thinks* before it sees a single user message.

> `AGENTS.md` is a small, opinionated, cross-vendor spec ([agents.md][agentsmd]) originated by OpenAI Codex, Amp, Jules (Google), Cursor, and Factory, now stewarded by the Agentic AI Foundation under the Linux Foundation. `CLAUDE.md` is the Claude-Code-specific equivalent, predates `AGENTS.md`, and is read by Claude Code in addition to (not instead of) any `AGENTS.md` it finds.

### → Memory layer (persistent context the LLM reads)

| File | Purpose |
|---|---|
| `MEMORY.md` | Always-loaded index of persistent memories. |
| `memory/*.md` | Individual memory files. Loaded on reference or retrieval. |
| `docs/`, `READMEs`, knowledge bases | Semantic memory available for RAG. |
| User profile / preference files | Long-term user-level state. |

> Distinction vs system-prompt files: memory is *factual* (what's true, what happened); `AGENTS.md`-style files are *behavioral* (how to act).

### → Tool layer

| File | Purpose |
|---|---|
| `.mcp.json` (MCP server registry) | Declares external [MCP][mcp] tool servers; their tools become callable. |
| Permission allow / deny lists | Which tools run freely, which need approval. |
| Custom tool definitions | User-defined functions exposed to the LLM. |
| `skills/*` (metadata) | The frontmatter (`name`, `description`) appears in the tool catalog so the LLM knows skills exist. |

### → Orchestrator layer (the loop, not the LLM)

These are read by the **harness** (the orchestrator runtime). The LLM never sees them.

| File | Purpose |
|---|---|
| `settings.json` | Model selection, permission mode, max steps, env vars, caching, cost limits. See [Claude Code settings][cc-settings]. |
| `hooks/` | Shell commands the harness runs at specific points: `PreToolUse`, `PostToolUse`, `UserPromptSubmit`, `SessionStart`, `Stop`, `Notification`. See [Claude Code hooks][cc-hooks]. |
| Output style / status line / keybindings | Interface and runtime config. |

> **Hooks are special.** They let you inject deterministic behavior into the loop without trusting the LLM to do it. The orchestrator triggers them on events; the LLM never sees or invokes them. This is the "harness-only" loading strategy in [§5](#5-three-loading-strategies).

### → Environment layer

| File | Purpose |
|---|---|
| Sandbox config | Containers, jails, ephemeral worktrees. |
| `.env` | Secrets and environment variables for tool execution. |
| Working dir / worktree settings | Where tool calls actually run. |
| Container settings | Isolation, resource limits. |

---

## 3. Skills — the interesting case

**Skills span the boundary between Reasoning and Tools.** They're neither pure prompt nor pure tool — they're **prompt expansions triggered on demand**.

- The **metadata** (YAML frontmatter: `name`, `description`, `when_to_use`) is **always in the tool catalog**, so the LLM knows the skill exists.
- The **body** (the actual instructions) is **only injected into context when the skill is invoked** (via the `Skill` tool or auto-matched).

This is the clever bit: **lazy loading**. You can have 50 skills available and only ~50 lines of metadata in context. Full bodies enter only when relevant. That's how capability scales without blowing the prompt budget.

> **Skills are now a cross-vendor open standard.** Originally a Claude Code feature, the `SKILL.md` format was released by Anthropic as the open [Agent Skills][agentskills] spec ([agentskills.io][agentskills]) and has been adopted by Cursor, GitHub Copilot, VS Code, Gemini CLI, OpenAI Codex, OpenHands, Goose, Letta, JetBrains Junie, Factory, Amp, Kiro, OpenCode, Spring AI, Mistral Vibe, Snowflake Cortex, Databricks Genie, Laravel Boost, and others. So while the rest of this section uses Claude Code's terminology, the *primitive* — lazy-loaded prompt expansions discoverable through tool-catalog metadata — is portable across the ecosystem in the same way that [MCP][mcp] tool servers are. The spec lives at [agentskills.io/specification][agentskills-spec], the source at [github.com/agentskills/agentskills][agentskills-gh], and Anthropic's reference example skills at [github.com/anthropics/skills][agentskills-examples].

Slash commands work similarly but expand into the *user message* rather than being invoked as a tool call. The official descriptions live at [Claude Code skills][cc-skills] and [Claude Code slash commands][cc-slash].

---

## 4. Loading model — when each artifact enters context

| Artifact | LLM sees it? | When loaded |
|---|---|---|
| `AGENTS.md` / `CLAUDE.md` | Yes | Session start, always |
| `MEMORY.md` (index) | Yes | Session start, always |
| Individual memory files | Yes | On reference / retrieval |
| Skill metadata (frontmatter) | Yes | Session start, always |
| Skill body | Yes | Only when invoked |
| Slash command body | Yes | Only when typed |
| Subagent prompt | Yes (in subagent's own context) | When spawned |
| MCP tool schemas | Yes | Session start, always |
| RAG retrieved chunks | Yes | On retrieval |
| `settings.json` | **No** | Process start, used by harness |
| Hooks | **No** | Executed by harness on events |
| `.mcp.json` (server registry) | **No** | Read by harness at startup |
| `keybindings.json`, statusline | **No** | Used by interface layer |
| `.env`, sandbox config | **No** | Used by environment layer |

The bottom half is the important distinction: **the harness reads it, the LLM never does.** Hooks especially — they're how you make things happen *around* the LLM without trusting the LLM to do them.

---

## 5. Three loading strategies

1. **Always loaded** — sits in the system prompt or context every turn. Cost: context-window space. Use for: rules that must never be skipped.
2. **Lazy / on-demand** — only enters context when invoked, retrieved, or referenced. Cost: a tool call or retrieval step. Use for: large bodies of capability that aren't always needed.
3. **Harness-only** — never enters the LLM context at all. The runtime executes it deterministically. Cost: none on context. Use for: anything you don't want the LLM to be able to skip, override, or hallucinate around.

The art of agent configuration is choosing the right strategy for each piece. **Critical safety policy → harness-only (hook). Skill the LLM picks when relevant → lazy. Project conventions → always loaded.**

---

## 6. Mental model: three knobs

(Teaching framing, not standard terminology — see [§1](#1-the-big-idea).)

| Knob | Layer it tunes | Files |
|---|---|---|
| **What it knows** | Memory | `MEMORY.md`, memory files, docs, RAG sources |
| **How it acts** | Reasoning | `AGENTS.md`, `CLAUDE.md`, subagent prompts, skills, slash commands |
| **What it can do** | Tools + Orchestrator | MCP servers, permissions, hooks, `settings.json` |

Skills are special: they tune the **how it acts** knob *lazily*, so capability doesn't cost context until you need it.

---

## 7. Why "control plane"?

Borrowing the term from networking and systems: the **data plane** is the runtime that processes requests turn by turn (the architecture in [`agent-architecture.md`](./agent-architecture.md)). The **control plane** is everything that *configures* the data plane without participating in it directly — policies, capabilities, permissions, prompts, hooks. This is the same split used in software-defined networking, in Kubernetes (`kube-apiserver` + controllers vs `kubelet` + CNI), and in service meshes.

A well-designed agent has a clean separation:

- The **data plane** is generic: same loop, same LLM, same orchestrator regardless of project.
- The **control plane** is project-specific: `AGENTS.md`, the MCP servers you've enabled, the hooks you've installed, the skills you've added.

This is why a single tool like Claude Code can become wildly different agents in different projects — same data plane, different control planes.

---

## 8. Practical implications

- **Don't put behavior in code that belongs in the control plane.** If you find yourself forking the harness to change how the agent behaves in your project, you probably want a hook, an `AGENTS.md` rule, or a skill instead.
- **Don't trust the LLM to follow rules that matter.** Critical policies belong in hooks (harness-enforced), not in the system prompt (LLM-suggested).
- **Use lazy loading aggressively.** Putting everything in `AGENTS.md` blows your context. Move large or situational content to skills or memory.
- **Treat the control plane as code.** Version it, review it, test it. Hooks especially can have outsized effects.

---

## 9. TL;DR

The runtime architecture is **how an agent works**. The control plane is **how you tell it to work in your context**. Each config file plugs into a specific layer, with one of three loading strategies (always / lazy / harness-only). The art is choosing the right strategy for each piece.

---

## References

The full annotated index lives in [`sources.md`](./sources.md). The references below are cited specifically by this document.

[agentsmd]: https://agents.md/ "AGENTS.md — cross-vendor spec for coding-agent instruction files"
[agentskills]: https://agentskills.io/ "Agent Skills — open SKILL.md spec"
[agentskills-spec]: https://agentskills.io/specification "Agent Skills — specification"
[agentskills-gh]: https://github.com/agentskills/agentskills "Agent Skills — GitHub"
[agentskills-examples]: https://github.com/anthropics/skills "Anthropic — example skills"
[mcp]: https://modelcontextprotocol.io/ "Model Context Protocol"
[cc-subagents]: https://code.claude.com/docs/en/sub-agents "Claude Code — subagents"
[cc-slash]: https://code.claude.com/docs/en/slash-commands "Claude Code — slash commands"
[cc-skills]: https://code.claude.com/docs/en/skills "Claude Code — skills"
[cc-settings]: https://code.claude.com/docs/en/settings "Claude Code — settings"
[cc-hooks]: https://code.claude.com/docs/en/hooks "Claude Code — hooks"

### Notes on framing

A few framings used above are the author's teaching devices, not standard literature terms; they're flagged here so the reader doesn't go looking for them in a textbook:

- **"The harness"** — common in industry usage for "the orchestrator runtime that wraps the LLM" (Claude Code, Aider, OpenHands, Devin), not in academic papers.
- **"Three knobs"** (what it knows / how it acts / what it can do) — a teaching device specific to these notes. The categories themselves correspond to standard architectural layers; only the *triplet framing* is original.
- **"Control plane / data plane"** — borrowed from networking and Kubernetes; an analogy, not a deep claim about agent internals.
- **"Always loaded / lazy / harness-only"** — descriptive labels; the underlying mechanisms are standard but the three-way split is a notes-internal taxonomy.
