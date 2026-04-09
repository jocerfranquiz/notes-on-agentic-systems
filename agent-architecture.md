# LLM Agent Architecture

A reference for the architecture of LLM-powered agents — components, how they fit together, and how the runtime loop works.

> Companion files:
> - [`agent-architecture.svg`](./agent-architecture.svg) — visual diagram of the architecture
> - [`control-plane.md`](./control-plane.md) — how configuration files plug into each layer
> - [`formal-model.md`](./formal-model.md) — formal model
> - [`study-guide.md`](./study-guide.md) — where to learn more
> - [`sources.md`](./sources.md) — every source cited here, in one place

---

## 1. Components

An LLM agent has these core components:

1. **LLM (the reasoning engine)** — the model that interprets input, plans, and decides actions.
2. **Instructions / system prompt** — role, goals, constraints, behavioral rules.
3. **Tools** — external functions the agent can call (APIs, code execution, file I/O, search, databases). Defined with schemas so the LLM knows how to invoke them.
4. **Memory**
   - *Short-term*: the current conversation / context window.
   - *Long-term*: persistent storage (vector databases, files, knowledge bases) recalled across sessions.
5. **Planning / reasoning loop** — control flow that lets the agent decompose tasks, reflect, and iterate (ReAct, Plan-and-Execute, Reflexion, …).
6. **Orchestration / runtime (the harness)** — the loop that feeds tool results back to the LLM, manages state, handles retries, enforces limits, and decides when to stop.
7. **Environment interface** — the world the agent acts on: filesystem, browser, shell, OS, other agents, users.
8. **Guardrails** — safety, validation, permissions, and policy checks on inputs, outputs, and tool calls.
9. **Observability** — logging, tracing, and evaluation of steps, tool calls, and outcomes.

> In short: **LLM + prompt + tools + memory + planning loop + runtime + guardrails**.

This decomposition follows the framing in Lilian Weng's [*LLM Powered Autonomous Agents*][weng] and the practical taxonomy in Anthropic's [*Building effective agents*][bea]; both are listed under [References](#references).

---

## 2. The architecture, at a glance

At a high level, an LLM agent is a **loop** that wraps an LLM with tools, memory, and a controller.

```
          ┌─────────────────────────────────────────┐
          │              USER / ENV                 │
          └───────────────┬─────────────────────────┘
                          │ goal / input
                          ▼
          ┌─────────────────────────────────────────┐
          │           ORCHESTRATOR (loop)           │
          │  - builds prompt   - parses output      │
          │  - routes tools    - manages state      │
          └───┬───────────┬──────────────┬──────────┘
              │           │              │
              ▼           ▼              ▼
        ┌─────────┐  ┌─────────┐   ┌──────────┐
        │   LLM   │  │ MEMORY  │   │  TOOLS   │
        │         │  │ ST / LT │   │ APIs/fs/ │
        └─────────┘  └─────────┘   │ shell/db │
                                   └────┬─────┘
                                        ▼
                                 ┌────────────┐
                                 │ENVIRONMENT │
                                 └────────────┘
```

---

## 3. Layers in detail

### 3.1 Interface layer

The boundary between the agent and whoever is using it.

**Responsibilities**
- Accept goals/inputs in some protocol (chat turns, REST, gRPC, message queue, agent-to-agent protocols like [MCP][mcp]).
- Stream partial outputs back (token streaming, tool-call events, status updates).
- Handle authentication, rate limiting, session identity.
- Expose capabilities/manifests so callers know what the agent can do.

**Design choices**
- Synchronous request/response (chat) vs async job (long-running tasks with polling/webhooks).
- Single-turn vs multi-turn with persistent session IDs.
- Human-in-the-loop hooks: approval gates, clarification requests, interrupt/resume.

---

### 3.2 Orchestration layer (the agent loop)

The control plane. This is where "agent behavior" actually lives — the LLM is stateless; the loop is what gives it agency.

**Per-turn pipeline**

1. **Context assembly** — system prompt + retrieved memory + conversation history + tool schemas + scratchpad.
2. **Token budgeting** — drop, summarize, or chunk to fit the context window.
3. **LLM call** — temperature, stop tokens, tool-use mode, structured-output schema.
4. **Response parsing** — text vs tool call vs structured object; validate against schema.
5. **Action dispatch** — route tool call to executor; enforce permissions.
6. **Observation capture** — append tool result (truncated/summarized if huge).
7. **Termination check** — done? max steps? budget? error? interrupt?
8. **State persistence** — checkpoint so the run can resume.

**Control-flow patterns**

| Pattern | Idea | Best for | Source |
|---|---|---|---|
| **ReAct** | Interleave Thought → Action → Observation | General-purpose, simple | Yao et al., 2022 [[ReAct]][react] |
| **Plan-and-Execute** | LLM plans first, then executes; replans on failure | Long horizons | LangChain pattern; closely related to *Plan-and-Solve* prompting [[Wang et al. 2023]][plan-and-solve] |
| **Reflexion** | Self-critique after each step or at end | Quality-sensitive tasks | Shinn et al., 2023 [[Reflexion]][reflexion] |
| **Tree of Thoughts** | Explore branches, score, pick best | Hard reasoning | Yao et al., 2023 [[ToT]][tot] |
| **Router / dispatcher** | Cheap model classifies, hands off to specialist | Cost optimization | Anthropic, [*Building effective agents*][bea] (the "routing" workflow) |
| **Multi-agent** | Supervisor/worker, debate, blackboard, pipeline | Complex, decomposable work | See [§10 of `sources.md`](./sources.md#9-multi-agent-systems) |

**Failure handling**
- Retries with backoff for transient tool errors.
- Tool-call schema validation → re-prompt with the error.
- Loop detection (same action repeated) → force a different strategy or escalate.
- Budget guards: max tokens, max steps, max wall-clock, max cost.

---

### 3.3 Reasoning layer (the LLM)

**What lives here**
- The base model(s) — possibly multiple (cheap router + strong solver + fast summarizer).
- The system prompt: role, capabilities, constraints, output format, tool-use rules.
- Decoding parameters: temperature, top-p, stop sequences, max tokens.
- Structured output mode (JSON schema, tool-use mode, grammar-constrained decoding).

**Prompt engineering primitives**
- Role/persona, task description, rules, examples (few-shot), output-format spec.
- Chain-of-Thought scaffolding ([Wei et al., 2022][cot]): "think step by step", scratchpad sections.
- Tool-use instructions: when to call, how to format, how to handle errors.
- Self-checks: "before answering, verify X" — formalized as iterative refinement in [Self-Refine][self-refine] (Madaan et al., 2023).

**Model selection trade-offs**
- Strong reasoning vs latency vs cost.
- Long context vs short context + RAG.
- Native tool-use support vs prompt-based parsing.
- Open-weight (self-hosted, private, cheaper at scale) vs hosted frontier models.

---

### 3.4 Memory layer

The model itself has no memory beyond the prompt — everything persistent is engineered. The taxonomy below adapts the cognitive-science split (working / episodic / semantic / procedural) used in Lilian Weng's [*LLM Powered Autonomous Agents*][weng] and in the [MemGPT/Letta][memgpt] framing of long-term memory.

| Type | What it holds | Example |
|---|---|---|
| **Working** | Live context: messages, tool results, scratchpad | The current conversation |
| **Episodic** | Past interactions, often summarized | "On 2026-04-01 we debugged X" |
| **Semantic** | Documents, facts, code as embeddings | Vector database of internal docs |
| **Procedural** | Skills, recipes, cached plans | "How I solved this last time" — see [Voyager][voyager] |
| **User profile** | Stable user facts and preferences | Role, name, prior decisions |

**Memory operations**
- *Write*: extract → classify → dedupe → store.
- *Read*: query → retrieve → rerank → filter → inject.
- *Update / forget*: TTL, supersedence, explicit deletion, contradiction resolution.

**Working-memory management**
- Summarization, truncation, sliding windows, hierarchical compression. The OS-style tiered approach is laid out in the [MemGPT paper][memgpt-paper].
- The orchestrator decides what to keep in the context window vs what to demote to long-term storage.

---

### 3.5 Tool layer

Tools are how the agent acts on the world. Each tool is a typed function with a schema the LLM sees.

**Anatomy of a tool**
- Name + natural-language description (the LLM reads this to decide when to use it).
- Input schema (JSON Schema / Pydantic / function signature).
- Output schema or free-form result.
- Implementation (sync or async).
- Permissions, rate limits, cost, side-effect classification.

**Common categories**
- **Information**: web search, RAG retriever, SQL query, knowledge-graph lookup.
- **Computation**: code interpreter, calculator, data-analysis sandbox.
- **I/O**: file read/write, shell, HTTP, browser automation.
- **Communication**: send email, post to Slack, open a PR, call another agent.
- **Domain-specific**: book a flight, run a deploy, query an internal service.
- **Meta-tools**: ask the user a clarifying question, request approval, spawn a subagent.

**Tool routing strategies**
- All tools always exposed (simple, but pollutes context).
- Filtered/retrieved tools per turn (embed descriptions, retrieve relevant ones).
- Hierarchical: a "category" tool that drills down into sub-tools.
- **Code-as-action**: the LLM writes a code snippet that calls tool functions directly. More expressive and more compositional than JSON tool calls; see [*Executable Code Actions Elicit Better LLM Agents*][codeact] (Wang et al., 2024) and Hugging Face's [smolagents][smolagents].

**Standardization**
- **[Model Context Protocol (MCP)][mcp]** — Anthropic's open spec, released late 2024. Plug tools/servers into any agent.
- **[Agent Skills][agentskills]** — Anthropic's open spec for `SKILL.md`: lazy-loaded prompt-expansion artifacts whose metadata appears in the tool catalog and whose body is fetched on demand. Adopted across Claude Code, Cursor, GitHub Copilot, OpenAI Codex, Gemini CLI, OpenHands, Goose, Letta, and many others. See [`control-plane.md` §3](./control-plane.md#3-skills--the-interesting-case) for the dual-nature mechanics.
- **[OpenAPI][openapi]** specifications can be auto-converted into tool schemas; most frameworks support this.

---

### 3.6 Environment layer

Where tool calls actually do things. The agent's "world".

**Examples**
- Local: filesystem, shell, OS processes, IDE.
- Network: HTTP APIs, databases, message queues.
- Browser: real or headless browser the agent drives.
- Other agents: peers in a multi-agent system.
- Physical: robots, IoT devices.
- Simulated: a sandbox/VM for safe execution.

**Concerns**
- *Sandboxing*: containers, VMs, ephemeral worktrees, read-only mounts.
- *Idempotency*: can a step be safely retried?
- *Reversibility*: can side effects be undone? (delete vs soft-delete, dry-run modes).
- *Determinism*: is the environment reproducible for testing?

---

## 4. Cross-cutting concerns

These touch every layer.

### 4.1 Guardrails / safety
- Input filters: prompt-injection detection, PII scrubbing, jailbreak heuristics. Simon Willison's [prompt-injection series][simonw-pi] is the most comprehensive ongoing reference.
- Output filters: toxicity, secrets leakage, format validation.
- Tool-call policy: allow-lists, per-tool permissions, cost ceilings, human approval for risky actions.
- Sandboxing for untrusted code/file access.
- Constitutional rules: a separate model or rule set that vetoes bad actions — formalized in Anthropic's [Constitutional AI][cai] (Bai et al., 2022).
- Threat surface: see [OWASP Top 10 for LLM Applications][owasp-llm] for the standard list.

### 4.2 Observability
- Structured traces of every LLM call, tool call, retrieval, and decision.
- Token / cost / latency metrics per step and per run.
- Replay: re-run a session against a new model/prompt for regression testing.
- Standards: [OpenTelemetry GenAI semantic conventions][otel-gen-ai] for span shapes; tooling like [LangSmith][langsmith], [Langfuse][langfuse], and [Arize Phoenix][phoenix] sits on top.

### 4.3 Evaluation
- *Offline*: golden datasets, unit tests for tool use, end-to-end task suites such as [SWE-bench][swebench] and [τ-bench][tau-bench].
- *Online*: A/B tests, user feedback signals, success-rate dashboards.
- *LLM-as-judge*: a stronger model grades outputs against rubrics.
- *Trajectory eval*: scoring the whole reasoning trace, not just the final answer — covered in [AgentBench][agentbench] and the [τ-bench][tau-bench] methodology.

### 4.4 State & persistence
- Session store (conversations, scratchpads).
- Checkpoint store (resumable long runs).
- Artifact store (files the agent produced).
- Job queue / scheduler for async and recurring agents.

### 4.5 Security & identity
- Per-user auth flowing into tool calls (the agent acts *as* the user, with their permissions).
- Secret management (no credentials in prompts).
- Audit logs of every action.
- Governance frame: [NIST AI Risk Management Framework][nist-airmf].

### 4.6 Cost & performance
- Caching: prompt caches, tool-result caches, embedding caches.
- Batching parallel tool calls.
- Model cascades: try cheap first, escalate on low confidence.
- Speculative execution: kick off likely tool calls in parallel with reasoning.

---

## 5. The minimal mental model

```python
while not done:
    prompt   = build_context(system, history, memory, tools)
    response = llm(prompt)
    if response.is_tool_call:
        result = run_tool(response.tool, response.args)   # guardrails here
        history.append(response, result)
    else:
        return response.text
```

Slightly less minimal:

```python
state = load_or_init_session(session_id)

while not state.done:
    # 1. Build context
    ctx = assemble_context(
        system_prompt,
        history=compress(state.history),
        memory=retrieve(state.query, k=8),
        tools=select_tools(state.query),
        scratchpad=state.scratchpad,
    )

    # 2. Reason
    resp = llm.call(ctx, tools=tools, response_format=schema)
    trace.log(resp)

    # 3. Act
    if resp.tool_calls:
        for call in resp.tool_calls:
            guardrails.check(call)              # permissions, policy
            result = tools.dispatch(call)        # may run in parallel
            state.history.append(call, result)
    else:
        state.answer = resp.text
        state.done = True

    # 4. Bookkeep
    state.steps += 1
    if state.steps > MAX_STEPS or budget.exceeded():
        state.done = True
    checkpoint(state)

return state.answer
```

Everything fancy in the agent literature — multi-agent systems, RAG variants, planners, reflection — is an elaboration of this loop. For a formal-typed treatment of the same loop, see [`formal-model.md`](./formal-model.md).

---

## 6. Concrete trace — how the layers interact

User asks: *"Find the slowest endpoint in our API last week and open a ticket."*

1. **Interface** receives the message, attaches user identity + session.
2. **Orchestrator** loads session state, retrieves relevant memory ("user is on the platform team", "tickets go in Linear project API").
3. **Reasoning**: LLM produces a plan — (a) query metrics, (b) rank, (c) draft ticket, (d) create it.
4. **Tool call**: `metrics.query(service="api", window="7d", groupby="endpoint")` → guardrails OK → executed → result truncated and appended.
5. **Reasoning**: LLM picks the worst endpoint, drafts ticket text.
6. **Tool call**: `linear.create_issue(...)` → policy says "needs human approval for writes" → orchestrator pauses, asks user via interface.
7. **User approves** → ticket created → result appended.
8. **Reasoning**: LLM produces a final summary with the ticket link.
9. **Memory write**: episodic memory updated ("on 2026-04-07 investigated slow endpoints, opened TICKET-123").
10. **Observability**: trace, tokens, cost, latency stored; success metric incremented.
11. **Interface** streams the final answer back.

---

## 7. TL;DR

**Interface → Orchestrator → (LLM ↔ Memory ↔ Tools) → Environment**, wrapped in guardrails, observability, evaluation, and persistence.

The LLM is stateless. The orchestrator loop, memory, and tools are what give the agent its agency. Everything else is configuration — see [`control-plane.md`](./control-plane.md).

---

## References

The full annotated index lives in [`sources.md`](./sources.md). The references below are cited specifically by this document.

[bea]: https://www.anthropic.com/engineering/building-effective-agents "Anthropic — Building effective agents (Dec 2024)"
[weng]: https://lilianweng.github.io/posts/2023-06-23-agent/ "Lilian Weng — LLM Powered Autonomous Agents (2023)"
[react]: https://arxiv.org/abs/2210.03629 "Yao et al. — ReAct (2022)"
[reflexion]: https://arxiv.org/abs/2303.11366 "Shinn et al. — Reflexion (2023)"
[tot]: https://arxiv.org/abs/2305.10601 "Yao et al. — Tree of Thoughts (2023)"
[plan-and-solve]: https://arxiv.org/abs/2305.04091 "Wang et al. — Plan-and-Solve Prompting (2023)"
[cot]: https://arxiv.org/abs/2201.11903 "Wei et al. — Chain-of-Thought Prompting (2022)"
[self-refine]: https://arxiv.org/abs/2303.17651 "Madaan et al. — Self-Refine (2023)"
[voyager]: https://arxiv.org/abs/2305.16291 "Wang et al. — Voyager (2023)"
[memgpt]: https://github.com/letta-ai/letta "MemGPT / Letta"
[memgpt-paper]: https://arxiv.org/abs/2310.08560 "Packer et al. — MemGPT (2023)"
[codeact]: https://arxiv.org/abs/2402.01030 "Wang et al. — Executable Code Actions Elicit Better LLM Agents (2024)"
[smolagents]: https://huggingface.co/docs/smolagents/index "Hugging Face — smolagents"
[mcp]: https://modelcontextprotocol.io/ "Model Context Protocol"
[agentskills]: https://agentskills.io/ "Agent Skills — open SKILL.md spec"
[openapi]: https://www.openapis.org/ "OpenAPI Specification"
[cai]: https://arxiv.org/abs/2212.08073 "Bai et al. — Constitutional AI (Anthropic, 2022)"
[simonw-pi]: https://simonwillison.net/series/prompt-injection/ "Simon Willison — prompt-injection series"
[owasp-llm]: https://genai.owasp.org/llm-top-10/ "OWASP Top 10 for LLM Applications"
[otel-gen-ai]: https://opentelemetry.io/docs/specs/semconv/gen-ai/ "OpenTelemetry GenAI semantic conventions"
[langsmith]: https://smith.langchain.com/ "LangSmith"
[langfuse]: https://langfuse.com/ "Langfuse"
[phoenix]: https://phoenix.arize.com/ "Arize Phoenix"
[swebench]: https://www.swebench.com/ "SWE-bench"
[tau-bench]: https://github.com/sierra-research/tau-bench "τ-bench"
[agentbench]: https://github.com/THUDM/AgentBench "AgentBench"
[nist-airmf]: https://www.nist.gov/itl/ai-risk-management-framework "NIST AI Risk Management Framework"

### Notes on framing

A few framings used above are the author's teaching devices, not standard literature terms; they're flagged here so the reader doesn't go looking for them in a textbook:

- **"the harness"** for the orchestrator runtime — common in industry usage (Claude Code, Aider, OpenHands), not in academic papers.
- **The five-way memory split** (working / episodic / semantic / procedural / user profile) — adapted from cognitive psychology and from [Weng (2023)][weng]; the user-profile slot is a practical addition not present in classical taxonomies.
- **"Three knobs"** — the *what it knows / how it acts / what it can do* breakdown lives in [`control-plane.md`](./control-plane.md), not here, and is a teaching device specific to these notes.
