# A Serious (and hype-less) Study Guide on Agents and LLMs

A curated set of resources for understanding LLM agent architecture, the control plane, and how to build effective agents, with direct links to every resource.

---

## 1. Recommended path

If you only have a few hours, do these in order:

1. **Anthropic: [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)** *(~1 hour)*
   The single best practical overview from people who ship them.
2. **Lilian Weng: [LLM Powered Autonomous Agents](https://lilianweng.github.io/posts/2023-06-23-agent/)** *(~1 hour)*
   The canonical academic-flavored overview: planning, memory, tool use.
3. **[Model Context Protocol intro](https://modelcontextprotocol.io/docs/getting-started/intro)** + **[Claude Code documentation](https://code.claude.com/docs/en/overview)** *(1–2 hours)*
   The control-plane mental model clicks fast once you've read both.
4. **Skim one framework's "concepts" page**, [LangGraph overview](https://docs.langchain.com/oss/python/langgraph/overview) is the densest *(30 min)*.
5. **Dip into papers** ([ReAct](https://arxiv.org/abs/2210.03629), [Reflexion](https://arxiv.org/abs/2303.11366), …) only when a specific pattern catches your interest.

---

## 2. Foundational essays: read these first

### [*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents)
**Erik Schluntz & Barry Zhang, Anthropic, December 2024.** The best practical overview. Covers workflows vs agents, common patterns (prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer), and (crucially) when *not* to use an agent. The companion code lives in the [Claude Cookbooks agent patterns folder](https://github.com/anthropics/claude-cookbooks/tree/main/patterns/agents).

### [*LLM Powered Autonomous Agents*](https://lilianweng.github.io/posts/2023-06-23-agent/)
**Lilian Weng (OpenAI), June 2023.** The canonical academic-flavored overview: planning, memory, tool use. Still the most-cited single piece in the field. Lives on her blog [Lil'Log](https://lilianweng.github.io/).

### [*AI Engineering*](https://www.oreilly.com/library/view/ai-engineering/9781098166298/) (chapter on Agents)
**Chip Huyen, O'Reilly, 2024.** Excellent on the engineering side: evaluation, failure modes, planning loops. The whole book is worth owning. See also [Chip Huyen's books page](https://huyenchip.com/books/) and the [supporting GitHub repository](https://github.com/chiphuyen/aie-book).

---

## 3. Patterns & techniques: the original papers

| Paper | Year | Key idea |
|---|---|---|
| [**ReAct**](https://arxiv.org/abs/2210.03629) | Yao et al., 2022 | Interleave Thought → Action → Observation |
| [**Reflexion**](https://arxiv.org/abs/2303.11366) | Shinn et al., 2023 | Self-critique to improve over iterations |
| [**Toolformer**](https://arxiv.org/abs/2302.04761) | Schick et al., 2023 | Tool use as a learned skill |
| [**Tree of Thoughts**](https://arxiv.org/abs/2305.10601) | Yao et al., 2023 | Explicit search over reasoning branches |
| [**Plan-and-Solve**](https://arxiv.org/abs/2305.04091) | Wang et al., 2023 | Decompose first, then execute step by step |
| [**Voyager**](https://arxiv.org/abs/2305.16291) | Wang et al., 2023 | Skill libraries / procedural memory in the wild ([project site](https://voyager.minedojo.org/)) |
| [**Self-Refine**](https://arxiv.org/abs/2303.17651) | Madaan et al., 2023 | Iterative improvement via self-feedback ([project site](https://selfrefine.info/)) |
| [**Chain-of-Thought**](https://arxiv.org/abs/2201.11903) | Wei et al., 2022 | Step-by-step reasoning prompts |
| [**Generative Agents**](https://arxiv.org/abs/2304.03442) | Park et al., 2023 | The famous Smallville simulation |

---

## 4. Protocols & specs (the control-plane stuff)

### [Model Context Protocol (MCP)](https://modelcontextprotocol.io/)
Anthropic's open spec for plugging tool servers into any agent. The de-facto standard for tool interoperability. Start with [the introduction](https://modelcontextprotocol.io/docs/getting-started/intro) and the [main GitHub org](https://github.com/modelcontextprotocol).

### [AGENTS.md](https://agents.md/)
Cross-vendor spec for "instructions to coding agents" files. Originated by OpenAI Codex, Amp, Jules (Google), Cursor, and Factory; now stewarded by the Agentic AI Foundation under the Linux Foundation. Implemented across most coding agents. Source on [GitHub](https://github.com/agentsmd/agents.md).

### [Agent Skills](https://agentskills.io/)
Anthropic's open `SKILL.md` standard for lazy-loaded capability bundles. olders of instructions, scripts, and resources that an agent discovers via metadata and loads on demand. Originally a Claude Code feature, now adopted by Cursor, GitHub Copilot, VS Code, Gemini CLI, OpenAI Codex, OpenHands, Goose, Letta, JetBrains Junie, Factory, Amp, and ~20 other tools. Start with the [overview](https://agentskills.io/home), then the [specification](https://agentskills.io/specification). Source on [GitHub](https://github.com/agentskills/agentskills); Anthropic's example skills at [anthropics/skills](https://github.com/anthropics/skills).

### OpenAPI → tool schemas
Tool schemas can be auto-generated from OpenAPI specs. Most frameworks support this directly.

---

## 5. Claude Code & Anthropic ecosystem

### [Claude Code documentation](https://code.claude.com/docs/en/overview)
The official source of truth, updates frequently. Sections on hooks, skills, subagents, MCP, settings, slash commands, plugins, output styles, status lines. The mirror at [docs.anthropic.com/en/docs/claude-code](https://docs.anthropic.com/en/docs/claude-code/overview) also serves the same content. Source on [GitHub](https://github.com/anthropics/claude-code).

### Claude Agent SDK
Same docs site. The SDK exposes the same primitives (tools, hooks, permissions) that Claude Code uses, so reading the SDK docs is one of the fastest ways to understand the harness model.

### [Claude Cookbooks](https://github.com/anthropics/claude-cookbooks)
Practical agent recipes on GitHub (formerly *Anthropic Cookbook*). The [`patterns/agents/`](https://github.com/anthropics/claude-cookbooks/tree/main/patterns/agents) folder contains the reference implementations for *Building Effective Agents* (orchestrator-workers, evaluator-optimizer, etc.).

### Anthropic Engineering blog
Periodic deep dives on agent design, tool use, and prompt engineering. Published under [anthropic.com/engineering](https://www.anthropic.com/engineering) and [anthropic.com/research](https://www.anthropic.com/research).

---

## 6. Frameworks (good for "show me code")

Each framework's docs is essentially an opinionated essay on agent architecture. Read the **concepts** pages, not the API reference.

| Framework | Strength | Links |
|---|---|---|
| **LangGraph** (LangChain) | Stateful loops, multi-agent, human-in-the-loop | [docs](https://docs.langchain.com/oss/python/langgraph/overview) · [product](https://www.langchain.com/langgraph) · [GitHub](https://github.com/langchain-ai/langgraph) |
| **LlamaIndex Workflows / Agents** | Retrieval and memory | [agents docs](https://developers.llamaindex.ai/python/framework/use_cases/agents/) · [Workflows 1.0 announcement](https://www.llamaindex.ai/blog/announcing-workflows-1-0-a-lightweight-framework-for-agentic-systems) |
| **Pydantic AI** | Typed tool calls, clean mental model | [docs](https://ai.pydantic.dev/) · [GitHub](https://github.com/pydantic/pydantic-ai) |
| **smolagents** (Hugging Face) | Minimal, code-as-action | [docs](https://huggingface.co/docs/smolagents/index) · [GitHub](https://github.com/huggingface/smolagents) · [intro blog](https://huggingface.co/blog/smolagents) |
| **CrewAI** | Multi-agent role-based | [docs](https://docs.crewai.com/) · [GitHub](https://github.com/crewaiinc/crewai) |
| **AutoGen** (Microsoft) | Conversational multi-agent (now in maintenance, see Microsoft Agent Framework below) | [docs](https://microsoft.github.io/autogen/stable/index.html) · [GitHub](https://github.com/microsoft/autogen) |
| **Microsoft Agent Framework** | The successor to AutoGen, enterprise-ready | [docs](https://learn.microsoft.com/en-us/agent-framework/overview/) |
| **OpenAI Agents SDK** | Lightweight handoff-based (production successor to Swarm) | [docs](https://openai.github.io/openai-agents-python/) · [GitHub](https://github.com/openai/openai-agents-python) · [original Swarm](https://github.com/openai/swarm) |
| **DSPy** (Stanford) | Programmatic prompts, optimization | [site](https://dspy.ai/) · [GitHub](https://github.com/stanfordnlp/dspy) |

---

## 7. Memory & retrieval

- **[GraphRAG](https://microsoft.github.io/graphrag/)** (Microsoft Research, 2024): graph-augmented retrieval over a corpus. [GitHub](https://github.com/microsoft/graphrag) · [project page](https://www.microsoft.com/en-us/research/project/graphrag/) · [paper](https://arxiv.org/abs/2404.16130).
- **[MemGPT / Letta](https://github.com/letta-ai/letta)**: tiered memory inspired by OS virtual memory. The original [MemGPT paper (Packer et al., 2023)](https://arxiv.org/abs/2310.08560) is the canonical reference; the modern [Letta framework](https://docs.letta.com/) is the production successor.
- **Vector DB docs**: [Qdrant](https://qdrant.tech/documentation/), [Weaviate](https://docs.weaviate.io/), [pgvector](https://github.com/pgvector/pgvector), [Chroma](https://docs.trychroma.com/): each has good intro material.
- For RAG patterns, the [LlamaIndex agents docs](https://developers.llamaindex.ai/python/framework/use_cases/agents/) is the canonical reference.

---

## 8. Observability & evaluation

### Tracing platforms
Each has docs that double as a tutorial on what to instrument:

- **[Langfuse](https://langfuse.com/)**: open source, self-hostable. [Docs](https://langfuse.com/docs) · [GitHub](https://github.com/langfuse/langfuse)
- **[LangSmith](https://smith.langchain.com/)**: hosted, by LangChain. [Docs](https://docs.langchain.com/langsmith/home)
- **[Arize Phoenix](https://phoenix.arize.com/)**: open source, very conceptual docs. [Docs](https://arize.com/docs/phoenix) · [GitHub](https://github.com/Arize-ai/phoenix)
- **[Helicone](https://www.helicone.ai/)**: proxy-based. [Docs](https://docs.helicone.ai/) · [GitHub](https://github.com/Helicone/helicone)
- **[Braintrust](https://www.braintrust.dev/)**: eval-focused. [Docs](https://www.braintrust.dev/docs)

### Standards
- **[OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)**: the emerging standard for tracing LLM/agent calls. See also the [agent-spans page](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/).

### Evaluation frameworks & benchmarks
- **[`lm-evaluation-harness`](https://github.com/EleutherAI/lm-evaluation-harness)** (EleutherAI): base-model benchmarks; backend for the HuggingFace Open LLM Leaderboard.
- **[HELM](https://crfm.stanford.edu/helm/)** (Stanford CRFM): holistic evaluation framework. [GitHub](https://github.com/stanford-crfm/helm) · [paper](https://arxiv.org/abs/2211.09110)
- **[AgentBench](https://github.com/THUDM/AgentBench)** (Tsinghua): multi-environment LLM-as-agent benchmark. [Paper](https://arxiv.org/abs/2308.03688)
- **[SWE-bench](https://www.swebench.com/)**: solving real GitHub issues. [GitHub](https://github.com/SWE-bench/SWE-bench)
- **[τ-bench](https://github.com/sierra-research/tau-bench)** (Sierra): tool-agent-user interaction in real-world domains. [Blog post](https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents) · [τ²-bench](https://github.com/sierra-research/tau2-bench)

---

## 9. Safety, security, and guardrails

- **[OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)**: the standard threat list. [Project page](https://owasp.org/www-project-top-10-for-large-language-model-applications/) · [2025 PDF](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf)
- **Prompt injection**: [Simon Willison's prompt injection series](https://simonwillison.net/series/prompt-injection/) is the most comprehensive ongoing coverage. He coined the term and continues to write about new variants on his [main blog](https://simonwillison.net/) and his [substack](https://simonw.substack.com/).
- **[NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)**: for governance angles. [AI RMF 1.0 PDF](https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf) · [Resource Center](https://airc.nist.gov/airmf-resources/airmf/)
- **Anthropic's Responsible Scaling Policy** model-level safety thinking, published on [anthropic.com](https://www.anthropic.com/).

---

## 10. Multi-agent & emerging directions

- **[AutoGen paper](https://arxiv.org/abs/2308.08155)** (Wu et al., 2023): multi-agent conversation framework
- **[MetaGPT](https://github.com/FoundationAgents/MetaGPT)**: assembly-line multi-agent. [Paper](https://arxiv.org/abs/2308.00352)
- **[ChatDev](https://github.com/OpenBMB/ChatDev)**: software-company-as-multi-agent. [Paper](https://arxiv.org/abs/2307.07924)
- **[Generative Agents](https://github.com/joonspk-research/generative_agents)** (Park et al., 2023): the famous Smallville simulation. [Paper](https://arxiv.org/abs/2304.03442)

---

## 11. Going deeper: books

- **Chip Huyen: [*AI Engineering*](https://www.oreilly.com/library/view/ai-engineering/9781098166298/)** (O'Reilly, 2024): production AI systems. [Author page](https://huyenchip.com/books/) · [GitHub](https://github.com/chiphuyen/aie-book)
- **Jay Alammar & Maarten Grootendorst: [*Hands-On Large Language Models*](https://www.llm-book.com/)** (O'Reilly, 2024): visual, accessible. [O'Reilly](https://www.oreilly.com/library/view/hands-on-large-language/9781098150952/) · [GitHub](https://github.com/HandsOnLLM/Hands-On-Large-Language-Models)
- **Sebastian Raschka: [*Build a Large Language Model (From Scratch)*](https://www.manning.com/books/build-a-large-language-model-from-scratch)** (Manning, 2024): for understanding what's *inside* the LLM. [GitHub](https://github.com/rasbt/LLMs-from-scratch) · [author's books page](https://sebastianraschka.com/books/)

---

## 12. Communities & ongoing reading

- **Anthropic, OpenAI, DeepMind engineering blogs**: best practical writing
  - [Anthropic Engineering](https://www.anthropic.com/engineering) · [Anthropic Research](https://www.anthropic.com/research)
- **[Simon Willison's blog](https://simonwillison.net/)**: daily LLM news and analysis (the best single feed in the field)
- **[Latent Space podcast](https://www.latent.space/podcast)**: interviews with builders, hosted by swyx and Alessio. [Newsletter](https://www.latent.space/)
- **Hacker News `ai` tag**: high-signal discussions
- **[LangChain blog](https://blog.langchain.dev/)**, **[LlamaIndex blog](https://www.llamaindex.ai/blog)**: framework-level pattern writeups
- **arXiv [`cs.CL`](https://arxiv.org/list/cs.CL/recent) and [`cs.AI`](https://arxiv.org/list/cs.AI/recent)**: primary research

---

## 13. By topic: quick reference

| If you want to understand… | Start with |
|---|---|
| What an agent is | [Anthropic *Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents) |
| Planning patterns | [ReAct](https://arxiv.org/abs/2210.03629), [Plan-and-Solve](https://arxiv.org/abs/2305.04091) papers |
| Memory architectures | [Lilian Weng's post](https://lilianweng.github.io/posts/2023-06-23-agent/), [MemGPT/Letta](https://github.com/letta-ai/letta) |
| Tool integration | [MCP docs](https://modelcontextprotocol.io/) |
| Configuration / control plane | [Claude Code docs](https://code.claude.com/docs/en/overview) (hooks, skills, subagents) |
| Multi-agent systems | [LangGraph](https://docs.langchain.com/oss/python/langgraph/overview), [AutoGen](https://microsoft.github.io/autogen/stable/index.html), [MetaGPT](https://github.com/FoundationAgents/MetaGPT) |
| Production tracing | [Arize Phoenix](https://arize.com/docs/phoenix) or [Langfuse](https://langfuse.com/docs) |
| Agent evaluation | [SWE-bench](https://www.swebench.com/), [τ-bench](https://github.com/sierra-research/tau-bench), [AgentBench](https://github.com/THUDM/AgentBench) |
| Prompt injection / safety | [Simon Willison's series](https://simonwillison.net/series/prompt-injection/), [OWASP LLM Top 10](https://genai.owasp.org/llm-top-10/) |
| RAG | [LlamaIndex agents](https://developers.llamaindex.ai/python/framework/use_cases/agents/), [GraphRAG](https://microsoft.github.io/graphrag/) |
| LLMs from the inside | [Sebastian Raschka's book](https://www.manning.com/books/build-a-large-language-model-from-scratch) |

---

## 14. A note on freshness

This field moves fast. Patterns from 2023 may be obsolete; protocols from 2024 may be standard by next quarter. Treat any specific tool or framework recommendation as a snapshot, not gospel. The **concepts** (loop, memory, tools, control plane, three knobs) are stable. The **implementations** churn.

When in doubt: read the official docs of whatever tool you're actually using, then triangulate with one or two of the foundational essays above.
