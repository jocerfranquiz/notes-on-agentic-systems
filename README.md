# Notes On Agentic Systems

A small set of notes on LLM agents: how they're built, how they're configured, what they formally are, and where to read more.

---

## The files

| File | What it is |
|---|---|
| [`agent-architecture.md`](./agent-architecture.md) | Runtime architecture of an LLM agent — components, layers, the agent loop, cross-cutting concerns. Informal prose. |
| [`agent-architecture.svg`](./agent-architecture.svg) | Visual diagram of the runtime architecture. Pairs with `agent-architecture.md`. |
| [`control-plane.md`](./control-plane.md) | How configuration files (`AGENTS.md`, hooks, skills, MCP servers, `settings.json`, …) plug into the architecture. |
| [`control-plane.svg`](./control-plane.svg) | Visual diagram of the runtime architecture *with* the configuration plane overlaid. Pairs with `control-plane.md`. |
| [`formal-model.md`](./formal-model.md) | A formal operational semantics of an LLM agent — types, the `step` kernel, multi-agent configurations, the Wooldridge collapse. Math-flavored. |
| [`study-guide.md`](./study-guide.md) | Curated reading paths into the field. Foundational essays, papers, protocols, frameworks, observability, safety. |
| [`sources.md`](./sources.md) | Annotated index of every external source cited across the other files. |

---

## Suggested reading order

1. **[`agent-architecture.md`](./agent-architecture.md)** — start here. The mental model the rest of the notes build on.
2. **[`control-plane.md`](./control-plane.md)** — once you have the architecture, this explains how every config file in a real agent project plugs into it.
3. **[`formal-model.md`](./formal-model.md)** — for the type-theoretic / operational-semantics view of the same thing. Read this when the prose framings start to feel hand-wavy.
4. **[`study-guide.md`](./study-guide.md)** — when you want to go beyond these notes. Has its own internal "if you only have a few hours" recommended path.
5. **[`sources.md`](./sources.md)** — reference, not reading. Look things up here.

---

## How the files relate

```
                       ┌─────────────────────┐
                       │      index.md       │  ← you are here
                       └──────────┬──────────┘
                                  │
        ┌─────────────────────────┼──────────────────────────┐
        ▼                         ▼                          ▼
 ┌───────────────┐        ┌───────────────┐         ┌─────────────────┐
 │ architecture  │ <────> │ control-plane │         │  formal-model   │
 │  (.md + .svg) │        │  (.md + .svg) │         │      (.md)      │
 └───────┬───────┘        └───────┬───────┘         └────────┬────────┘
         │                        │                          │
         └────────────┬───────────┴──────────────────────────┘
                      ▼
              ┌───────────────┐         ┌───────────────┐
              │  study-guide  │ ──────> │    sources    │
              │     (.md)     │         │     (.md)     │
              └───────────────┘         └───────────────┘
```

The three core docs (`agent-architecture`, `control-plane`, `formal-model`) cover the same object from three angles: prose, configuration, and math. `study-guide` and `sources` are the bibliography apparatus around them.
