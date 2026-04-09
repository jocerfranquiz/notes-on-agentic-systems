# Sources

A categorized index of every source referenced (or available for citation) across `agent-architecture.md`, `control-plane.md`, `study-guide.md`, and `formal-model.md`.

Each entry: a title and link, the kind of source, and a one-line note on what it is and why it's worth reading.

> Companion files:
> - [`study-guide.md`](./study-guide.md) — curated reading paths
> - [`agent-architecture.md`](./agent-architecture.md) — runtime architecture
> - [`control-plane.md`](./control-plane.md) — configuration plane
> - [`formal-model.md`](./formal-model.md) — formal model

---

## 1. Foundational essays

- **[*Building effective agents*](https://www.anthropic.com/engineering/building-effective-agents)** — Erik Schluntz & Barry Zhang, Anthropic, December 2024. Blog post. The single best practical overview: workflows vs agents, common patterns, when *not* to use an agent. Reference implementations live in the [Claude Cookbooks](https://github.com/anthropics/claude-cookbooks/tree/main/patterns/agents).
- **[*LLM Powered Autonomous Agents*](https://lilianweng.github.io/posts/2023-06-23-agent/)** — Lilian Weng, June 2023. Blog post. The canonical academic-flavored overview: planning, memory, tool use. Origin of the widely-reused diagram of "agent = LLM + memory + planning + tools".
- **[*AI Engineering*](https://www.oreilly.com/library/view/ai-engineering/9781098166298/)** — Chip Huyen, O'Reilly, 2024. Book. The chapter on agents is excellent on evaluation, failure modes, and planning loops; the rest of the book is the best current reference on production AI systems. [Author page](https://huyenchip.com/books/) · [GitHub](https://github.com/chiphuyen/aie-book).

---

## 2. Patterns & techniques — original papers

- **[ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)** — Yao et al., 2022. Paper. Interleave Thought → Action → Observation; the foundation of nearly every modern tool-using agent.
- **[Reflexion: Language Agents with Verbal Reinforcement Learning](https://arxiv.org/abs/2303.11366)** — Shinn et al., 2023. Paper. Self-critique to improve over iterations.
- **[Toolformer: Language Models Can Teach Themselves to Use Tools](https://arxiv.org/abs/2302.04761)** — Schick et al., 2023. Paper. Tool use as a learned skill, via self-supervised data generation.
- **[Tree of Thoughts: Deliberate Problem Solving with Large Language Models](https://arxiv.org/abs/2305.10601)** — Yao et al., 2023. Paper. Explicit search over reasoning branches.
- **[Plan-and-Solve Prompting](https://arxiv.org/abs/2305.04091)** — Wang et al., 2023. Paper. Decompose first, then execute step by step.
- **[Voyager: An Open-Ended Embodied Agent with Large Language Models](https://arxiv.org/abs/2305.16291)** — Wang et al., 2023. Paper. Skill libraries / procedural memory in Minecraft. [Project site](https://voyager.minedojo.org/).
- **[Self-Refine: Iterative Refinement with Self-Feedback](https://arxiv.org/abs/2303.17651)** — Madaan et al., 2023. Paper. Iterative improvement via self-feedback. [Project site](https://selfrefine.info/).
- **[Chain-of-Thought Prompting Elicits Reasoning in Large Language Models](https://arxiv.org/abs/2201.11903)** — Wei et al., 2022. Paper. Step-by-step reasoning prompts.
- **[Generative Agents: Interactive Simulacra of Human Behavior](https://arxiv.org/abs/2304.03442)** — Park et al., 2023. Paper. The famous Smallville simulation; the canonical reference for episodic memory + reflection in LLM agents. [GitHub](https://github.com/joonspk-research/generative_agents).
- **[Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073)** — Bai et al., Anthropic, 2022. Paper. The reference for "constitutional rules / a separate model that vetoes bad actions" — cited in `agent-architecture.md` §4.1.

---

## 3. Protocols & specs

- **[Model Context Protocol (MCP)](https://modelcontextprotocol.io/)** — Anthropic + community, late 2024. Spec + SDKs. The de-facto standard for plugging tool servers into any agent. Start with the [introduction](https://modelcontextprotocol.io/docs/getting-started/intro) and the [main GitHub org](https://github.com/modelcontextprotocol).
- **[AGENTS.md](https://agents.md/)** — Cross-vendor spec, 2024–2025. Spec. A small, opinionated convention for "instructions to coding agents" files. Originated by OpenAI Codex, Amp, Jules (Google), Cursor, and Factory; now stewarded by the Agentic AI Foundation under the Linux Foundation, with implementations across most coding agents (Aider, Goose, Zed, Warp, VS Code, Cognition, JetBrains Junie, GitHub Copilot, and others). [GitHub](https://github.com/agentsmd/agents.md).
- **[Agent Skills](https://agentskills.io/)** — Anthropic + community, 2025. Spec. The cross-vendor open standard for `SKILL.md` — folders of instructions, scripts, and resources that agents discover and load on demand. Originally from Anthropic, now adopted by Claude Code, Cursor, GitHub Copilot, VS Code, Gemini CLI, OpenAI Codex, OpenHands, Goose, Letta, JetBrains Junie, Factory, Amp, Kiro, OpenCode, and many others. [Spec](https://agentskills.io/specification) · [GitHub](https://github.com/agentskills/agentskills) · [example skills](https://github.com/anthropics/skills).
- **[OpenAPI Specification](https://www.openapis.org/)** — OpenAPI Initiative. Spec. Tool schemas can be auto-generated from OpenAPI specs; most agent frameworks support this.

---

## 4. Claude Code & the Anthropic ecosystem

- **[Claude Code documentation](https://code.claude.com/docs/en/overview)** — Anthropic, ongoing. Docs. The official source of truth for hooks, skills, subagents, MCP, settings, slash commands, plugins, output styles, status lines. Mirror at [docs.anthropic.com](https://docs.anthropic.com/en/docs/claude-code/overview); source on [GitHub](https://github.com/anthropics/claude-code).
- **Claude Agent SDK** — Anthropic. Docs. The same primitives Claude Code uses, exposed as an SDK. Reading the SDK docs is the fastest way to understand the harness model.
- **[Claude Cookbooks](https://github.com/anthropics/claude-cookbooks)** — Anthropic. Code. Practical agent recipes; the [`patterns/agents/`](https://github.com/anthropics/claude-cookbooks/tree/main/patterns/agents) folder contains the reference implementations for *Building Effective Agents*.
- **Anthropic Engineering & Research blogs** — [anthropic.com/engineering](https://www.anthropic.com/engineering) and [anthropic.com/research](https://www.anthropic.com/research). Periodic deep dives on agent design, tool use, prompt engineering.
- **[Anthropic Responsible Scaling Policy](https://www.anthropic.com/news/anthropics-responsible-scaling-policy)** — Anthropic. Policy. Model-level safety thinking; the reference for "tiered capability thresholds + safety commitments" framing.

---

## 5. Frameworks

- **[LangGraph](https://docs.langchain.com/oss/python/langgraph/overview)** — LangChain. Framework. Stateful loops, multi-agent topologies, human-in-the-loop. The most rigorous mental model in the open-source space. [Product](https://www.langchain.com/langgraph) · [GitHub](https://github.com/langchain-ai/langgraph).
- **[LlamaIndex Workflows / Agents](https://developers.llamaindex.ai/python/framework/use_cases/agents/)** — LlamaIndex. Framework. Strong on retrieval and memory. See also the [Workflows 1.0 announcement](https://www.llamaindex.ai/blog/announcing-workflows-1-0-a-lightweight-framework-for-agentic-systems).
- **[Pydantic AI](https://ai.pydantic.dev/)** — Pydantic team. Framework. Typed tool calls, clean mental model. [GitHub](https://github.com/pydantic/pydantic-ai).
- **[smolagents](https://huggingface.co/docs/smolagents/index)** — Hugging Face. Framework. Minimal, code-as-action. [GitHub](https://github.com/huggingface/smolagents) · [intro blog](https://huggingface.co/blog/smolagents).
- **[CrewAI](https://docs.crewai.com/)** — CrewAI Inc. Framework. Multi-agent role-based teams. [GitHub](https://github.com/crewaiinc/crewai).
- **[AutoGen](https://microsoft.github.io/autogen/stable/index.html)** — Microsoft Research. Framework (now in maintenance). Conversational multi-agent. [GitHub](https://github.com/microsoft/autogen) · [paper](https://arxiv.org/abs/2308.08155).
- **[Microsoft Agent Framework](https://learn.microsoft.com/en-us/agent-framework/overview/)** — Microsoft. Framework. The successor to AutoGen, enterprise-oriented.
- **[OpenAI Agents SDK](https://openai.github.io/openai-agents-python/)** — OpenAI. Framework. Lightweight handoff-based multi-agent (production successor to [Swarm](https://github.com/openai/swarm)). [GitHub](https://github.com/openai/openai-agents-python).
- **[DSPy](https://dspy.ai/)** — Stanford NLP. Framework. Programmatic prompts as compiled artifacts; optimization over prompts. [GitHub](https://github.com/stanfordnlp/dspy).

---

## 6. Memory & retrieval

- **[GraphRAG](https://microsoft.github.io/graphrag/)** — Microsoft Research, 2024. Framework + paper. Graph-augmented retrieval over a corpus. [GitHub](https://github.com/microsoft/graphrag) · [project page](https://www.microsoft.com/en-us/research/project/graphrag/) · [paper](https://arxiv.org/abs/2404.16130).
- **[MemGPT / Letta](https://github.com/letta-ai/letta)** — Packer et al., UC Berkeley, 2023 → Letta, ongoing. Framework + paper. Tiered memory inspired by OS virtual memory. [Letta docs](https://docs.letta.com/) · [paper](https://arxiv.org/abs/2310.08560).
- **Vector databases**: [Qdrant](https://qdrant.tech/documentation/), [Weaviate](https://docs.weaviate.io/), [pgvector](https://github.com/pgvector/pgvector), [Chroma](https://docs.trychroma.com/).
- **[LlamaIndex retrieval docs](https://developers.llamaindex.ai/python/framework/use_cases/agents/)** — the canonical reference for the modern RAG taxonomy.

---

## 7. Observability & evaluation

### Tracing platforms

- **[Langfuse](https://langfuse.com/)** — Open source, self-hostable tracing. [Docs](https://langfuse.com/docs) · [GitHub](https://github.com/langfuse/langfuse).
- **[LangSmith](https://smith.langchain.com/)** — Hosted, by LangChain. [Docs](https://docs.langchain.com/langsmith/home).
- **[Arize Phoenix](https://phoenix.arize.com/)** — Open source, very conceptual docs. [Docs](https://arize.com/docs/phoenix) · [GitHub](https://github.com/Arize-ai/phoenix).
- **[Helicone](https://www.helicone.ai/)** — Proxy-based observability. [Docs](https://docs.helicone.ai/) · [GitHub](https://github.com/Helicone/helicone).
- **[Braintrust](https://www.braintrust.dev/)** — Eval-focused. [Docs](https://www.braintrust.dev/docs).

### Standards

- **[OpenTelemetry GenAI semantic conventions](https://opentelemetry.io/docs/specs/semconv/gen-ai/)** — OpenTelemetry. Spec. The emerging standard for tracing LLM/agent calls. See the [agent-spans page](https://opentelemetry.io/docs/specs/semconv/gen-ai/gen-ai-agent-spans/) for the per-step schema.

### Benchmarks

- **[`lm-evaluation-harness`](https://github.com/EleutherAI/lm-evaluation-harness)** — EleutherAI. Code. Base-model benchmarks; backend for the HuggingFace Open LLM Leaderboard.
- **[HELM](https://crfm.stanford.edu/helm/)** — Stanford CRFM. Framework. Holistic evaluation. [GitHub](https://github.com/stanford-crfm/helm) · [paper](https://arxiv.org/abs/2211.09110).
- **[AgentBench](https://github.com/THUDM/AgentBench)** — Tsinghua. Benchmark. Multi-environment LLM-as-agent evaluation. [Paper](https://arxiv.org/abs/2308.03688).
- **[SWE-bench](https://www.swebench.com/)** — Princeton + others. Benchmark. Solving real GitHub issues. [GitHub](https://github.com/SWE-bench/SWE-bench).
- **[τ-bench](https://github.com/sierra-research/tau-bench)** — Sierra. Benchmark. Tool-agent-user interaction in real-world domains. [Blog](https://sierra.ai/blog/tau-bench-shaping-development-evaluation-agents) · [τ²-bench](https://github.com/sierra-research/tau2-bench).

---

## 8. Safety, security, and guardrails

- **[OWASP Top 10 for LLM Applications](https://genai.owasp.org/llm-top-10/)** — OWASP. Threat list. The standard reference for LLM-app threats. [Project page](https://owasp.org/www-project-top-10-for-large-language-model-applications/) · [2025 PDF](https://owasp.org/www-project-top-10-for-large-language-model-applications/assets/PDF/OWASP-Top-10-for-LLMs-v2025.pdf).
- **[Simon Willison — prompt injection series](https://simonwillison.net/series/prompt-injection/)** — Simon Willison. Blog. He coined the term and continues the most thorough ongoing coverage. [Main blog](https://simonwillison.net/) · [substack](https://simonw.substack.com/).
- **[NIST AI Risk Management Framework](https://www.nist.gov/itl/ai-risk-management-framework)** — NIST. Framework. The governance reference. [AI RMF 1.0 PDF](https://nvlpubs.nist.gov/nistpubs/ai/nist.ai.100-1.pdf) · [Resource Center](https://airc.nist.gov/airmf-resources/airmf/).
- **[Anthropic Responsible Scaling Policy](https://www.anthropic.com/news/anthropics-responsible-scaling-policy)** — Anthropic. Policy.
- **[Constitutional AI](https://arxiv.org/abs/2212.08073)** — Bai et al., Anthropic, 2022. Paper. Reference for "rule-based vetoing model" patterns.

---

## 9. Multi-agent systems

- **[AutoGen](https://arxiv.org/abs/2308.08155)** — Wu et al., Microsoft Research, 2023. Paper. Multi-agent conversation framework.
- **[MetaGPT](https://github.com/FoundationAgents/MetaGPT)** — Hong et al., 2023. Code + paper. Assembly-line multi-agent. [Paper](https://arxiv.org/abs/2308.00352).
- **[ChatDev](https://github.com/OpenBMB/ChatDev)** — Qian et al., 2023. Code + paper. Software-company-as-multi-agent. [Paper](https://arxiv.org/abs/2307.07924).
- **[Generative Agents](https://github.com/joonspk-research/generative_agents)** — Park et al., 2023. Code + paper. The Smallville simulation. [Paper](https://arxiv.org/abs/2304.03442).

---

## 10. Background — formal models of agents

These back the formal model in `formal-model.md`.

- **Wooldridge — *An Introduction to MultiAgent Systems*** (Wiley, 2nd ed., 2009). Book. The standard reference for the BDI (Belief-Desire-Intention) tradition and the classical `next : S × P → S` formalization of an agent. [Publisher page](https://www.wiley.com/en-us/An+Introduction+to+MultiAgent+Systems%2C+2nd+Edition-p-9780470519462).
- **Russell & Norvig — *Artificial Intelligence: A Modern Approach*** (Pearson, 4th ed., 2020). Book. Chapter 2 ("Intelligent Agents") is the canonical textbook treatment of percepts, actions, environments, and agent functions. [Book site](https://aima.cs.berkeley.edu/).
- **Lynch — *Distributed Algorithms*** (Morgan Kaufmann, 1996). Book. The standard reference for I/O automata. [Publisher page](https://www.elsevier.com/books/distributed-algorithms/lynch/978-1-55860-348-6).
- **Hewitt, Bishop, Steiger — *A Universal Modular ACTOR Formalism for Artificial Intelligence*** (IJCAI, 1973). Paper. The original actor-model paper, basis for the multi-agent configuration semantics in the formal model. [Available on arXiv (Hewitt's later writeup)](https://arxiv.org/abs/1008.1459).
- **Agha — *ACTORS: A Model of Concurrent Computation in Distributed Systems*** (MIT Press, 1986). Book. The cleaner formal treatment of actors. [MIT page](https://mitpress.mit.edu/9780262511414/actors/).

---

## 11. Going deeper — books

- **Chip Huyen — [*AI Engineering*](https://www.oreilly.com/library/view/ai-engineering/9781098166298/)** (O'Reilly, 2024). Production AI systems. [Author page](https://huyenchip.com/books/) · [GitHub](https://github.com/chiphuyen/aie-book).
- **Jay Alammar & Maarten Grootendorst — [*Hands-On Large Language Models*](https://www.llm-book.com/)** (O'Reilly, 2024). Visual, accessible. [O'Reilly](https://www.oreilly.com/library/view/hands-on-large-language/9781098150952/) · [GitHub](https://github.com/HandsOnLLM/Hands-On-Large-Language-Models).
- **Sebastian Raschka — [*Build a Large Language Model (From Scratch)*](https://www.manning.com/books/build-a-large-language-model-from-scratch)** (Manning, 2024). For understanding what's *inside* the LLM. [GitHub](https://github.com/rasbt/LLMs-from-scratch).

---

## 12. Communities & ongoing reading

- **[Anthropic Engineering](https://www.anthropic.com/engineering)** + **[Anthropic Research](https://www.anthropic.com/research)** — best practical writing on agent design.
- **[Simon Willison's blog](https://simonwillison.net/)** — daily LLM news and analysis (the best single feed in the field).
- **[Latent Space podcast](https://www.latent.space/podcast)** — interviews with builders, hosted by swyx and Alessio. [Newsletter](https://www.latent.space/).
- **[LangChain blog](https://blog.langchain.dev/)** + **[LlamaIndex blog](https://www.llamaindex.ai/blog)** — framework-level pattern writeups.
- **arXiv [`cs.CL`](https://arxiv.org/list/cs.CL/recent)** and **[`cs.AI`](https://arxiv.org/list/cs.AI/recent)** — primary research.

---

## 13. A note on version drift

This field churns. Specific tool URLs, framework status, and "best of" lists in this file are accurate as of **2026-04**, but may shift quickly. The papers and books are stable; the framework / tracing / benchmark sections decay fastest. When in doubt, follow the link to the project's own current docs rather than trusting a snapshot in this index.
