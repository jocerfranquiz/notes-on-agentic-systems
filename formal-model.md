# A Model for LLM Agents

A formal operational semantics of LLM-based agents — what types they have, how a single tick of computation evolves them, and how multi-agent systems compose. The aim is to write down precisely what a real LLM agent *is*, in a way that exposes the places where the classical agent literature (Wooldridge-style BDI agents, Russell-Norvig agent functions, I/O automata) either bottoms out into hand-waving or quietly collapses into something trivial.

> Companion files:
> - [`agent-architecture.md`](./agent-architecture.md) — the same model in informal prose, organized by layer
> - [`control-plane.md`](./control-plane.md) — what configures the static fields of an agent
> - [`sources.md`](./sources.md) — every source cited here

The notation conventions used throughout (`Δ(X)`, `X*`, sum types, …) and every defined symbol have entries in the [Glossary](#glossary) at the end. Each definition is targeted: it appears only because the term is non-standard, overloaded, or load-bearing for the math.

---

## 1. Motivation: what the classical formalisms hide

The textbook definition of a rational agent is a function

```
action : Percept* → Action
```

mapping the percept history to the next action ([Russell & Norvig, *AIMA*, ch. 2][aima]; [Wooldridge, *MultiAgent Systems*][wooldridge]). Internal state is folded in via

```
next : State × Percept → State
action : State → Action
```

with all the cognitive work living in `next` and `action`. This is clean, type-correct, and — for LLM agents — almost completely empty. Three things break:

1. **The action signature is open.** The classical formalisms assume `Action` is a fixed, finite alphabet, the way I/O automata ([Lynch, *Distributed Algorithms*][lynch]) assume a fixed input/output signature. In an LLM agent, *new tools can be registered at runtime*. The action set grows. No fixed signature can describe the agent's whole lifetime.
2. **`next` is degenerate.** As we'll see in [§5.2](#52-the-wooldridge-collapse), the state-update function in an LLM agent reduces to *append*. All the reasoning has migrated inside the LLM sample in a way the classical type doesn't even have a slot for.
3. **The transition is stochastic.** Sampling from a language model is not a function. The "deterministic agent function" type lies; the honest type is a distribution over actions.

This document writes down a model that takes those three observations seriously.

---

## 2. Preliminaries

### Notation

- `X*` — finite sequences (the Kleene star) over `X`.
- `X + Y` — disjoint union (sum type, tagged union). Elements are written `inj_X(x)` or, when unambiguous, with constructor names like `Reply(text, to)`.
- `X × Y` — Cartesian product (record type).
- `X → Y` — total function.
- `X ⇀ Y` — partial function.
- `Δ(X)` — the set of probability distributions over `X`. A *stochastic function* `f : X → Δ(Y)` is a Markov kernel: it maps each input to a distribution over outputs. Sampling is written `y ~ f(x)`.
- `f[k ↦ v]` — function update: same as `f` except at `k`, where it returns `v`.
- `xs ⊕ ys` — concatenation of sequences. `xs ⊕ x` is shorthand for `xs ⊕ [x]`.

### Base sets (parameters of the model)

| Set | Intended meaning |
|---|---|
| `Token` | Vocabulary symbol of the language model. |
| `AgentId` | Globally unique address for an agent in a multi-agent system. |
| `Name` | Identifier (for tools, agents, fields). |
| `Schema` | A typed schema (JSON Schema, a Pydantic model, …). |
| `Args`, `Result`, `Value`, `Key`, `Query` | Opaque payload sets used in tool calls and memory operations. |
| `Text` | Human-readable string (`⊆ Token*`). |
| `Message` | Inter-agent message payload, defined in [§7](#7-multi-agent-configuration). |

These are the "given" sets of the model. None of them are constructed; they are the carriers we build everything else on. Anything not parametric is defined below.

---

## 3. The agent

An agent is a pair of a static part (fixed at construction) and a dynamic part (mutated under `step`):

```
Agent = Static × Dynamic
```

The split is load-bearing: `Static` is what a constructor takes; `Dynamic` is what a single tick mutates. Most of what people call "agent frameworks" are opinionated ways to populate `Static`.

### 3.1 Static

```
Static = ⟨ id, LLM, Σ, T, σ, halt ⟩

  id    : AgentId                           — globally unique address
  LLM   : Token* → Δ(Token)                 — next-token distribution
  Σ     : Token*                            — system prompt / role / instructions
  T     : Name ⇀ ToolDef                    — tool registry, partial map by name
  σ     : DecodingParams                    — temperature, top-p, max_tokens, …
  halt  : Context × Memory → Bool           — termination predicate
```

with

```
ToolDef = ⟨ name : Name, schema : Schema, impl : Args → (W → Δ(W × (Result + Error))) ⟩
```

Two things to notice:

- `LLM` is the *next-token kernel*, not a one-shot function from prompt to answer. Multi-token generation is the standard autoregressive composition: sample `t₁ ~ LLM(prompt)`, then `t₂ ~ LLM(prompt ⊕ t₁)`, and so on, until a stop token or `max_tokens`. We will write `sample(LLM, prompt; σ) ∈ Δ(Token*)` for that composed kernel.
- `T` is a *partial map* from names to tool definitions, not a flat set. This makes "register new tool" mean "extend `T` at a fresh name", and makes the openness from [§1](#1-motivation-what-the-classical-formalisms-hide) explicit at the type level.
- Each tool's `impl` is **a function on the world** (`W` is the world-state; see [§4](#4-the-environment)). This makes side effects, observability, and stochasticity all type-visible.

### 3.2 Dynamic

```
Dynamic = ⟨ C, M, q, inbox ⟩

  C     : Context  ⊆ Token*                  — working context window
  M     : Memory                             — out-of-context state
  q     : ControlState                       — see below
  inbox : Message*                           — pending messages from other agents
```

with

```
ControlState = { ready, thinking, calling_tool, waiting, halted }
```

`Memory` is an abstract type with three operations:

```
read   : Memory × Key            → Value*
search : Memory × Query × ℕ      → Value*       — top-k
write  : Memory × Key × Value    → Memory
```

Concrete implementations include vector stores, key/value stores, file systems, knowledge graphs, or any composition of those — see the [memory layer in `agent-architecture.md` §3.4](./agent-architecture.md#34-memory-layer).

> **Why split `C` from `M`?** Classical formalisms have a single `State`. Real LLM agents distinguish *what fits in the prompt this turn* (`C`) from *what we know how to retrieve* (`M`), because the prompt has a hard token budget while the store does not. Folding them together loses essential structure (compaction, retrieval, eviction all live in the boundary between `C` and `M`).

---

## 4. The environment

```
Env = ⟨ W, w₀, τ, observe ⟩

  W       : WorldState                       — usually unspecifiable for open worlds
  w₀      : W                                — initial world
  τ       : W × Action → Δ(W × Percept)      — transition + observation
  observe : W × AgentId → Δ(Percept)         — passive observations (events the agent did not cause)
```

Honest note: for real LLM agents `W` is "the file system + the network + the user's mental state + …" — *not formally enumerable*. `τ` is "whatever the tool does." The model is **typed correctly** but its content, at the level of an actual file system or HTTP service, is opaque. This is exactly why the I/O-automaton view broke down: that framework demands an enumerable signature and a specifiable transition, and neither is on offer here.

We keep the type to enable formal reasoning *parameterized* over an environment (e.g., "for all `τ` satisfying property `P`, the agent run satisfies `Q`"), even though we cannot give a closed-form `τ`.

---

## 5. Percepts, actions, and the step function

### 5.1 Percepts and actions are sum types

```
Percept = UserMsg(text : Text)
        + ToolResult(call_id : Name, payload : Result + Error)
        + AgentMsg(from : AgentId, payload : Message)
        + SystemEvent(kind : EventKind)        — timer, interrupt, hook, cancel
```

```
Action = Reply(text : Text, to : AgentId + User)
       + ToolCall(name : Name, args : Args, call_id : Name)
       + MemOp(op : MemoryOp)
       + Spawn(child : Static, task : Text)
       + Send(to : AgentId, payload : Message)
       + Halt(result : Text)
```

with `MemoryOp = Read(Key) + Search(Query, ℕ) + Write(Key, Value)`.

The action set `A` is the disjoint union of the right-hand sides above, parameterized over `T`'s current keys for the `name` slot of `ToolCall`. This makes `A` *explicitly time-varying*: registering a new tool extends `T`, which extends the constructor's parameter range, which extends `A`. The action signature in classical I/O automata is finite and fixed. Here it is not.

### 5.2 The Wooldridge collapse

In the Wooldridge / Russell-Norvig formulation, an agent's internal state is updated by

```
next : State × Percept → State                  (the "cognitive function")
```

In our model the analog is some `U : Context × Action × Percept → Context` that produces the new context from the old context, the action just taken, and the observation that came back. In every real LLM agent

```
U(C, a, o) = C ⊕ render(a) ⊕ render(o)
```

That is, `U` is **just append**. There is no cognition in `U` at all. Every bit of work that the classical literature attributes to `next` has migrated into one place: the LLM sample in step (3) of [§5.3](#53-one-tick) below. We will call this fact the **Wooldridge collapse** and refer back to it whenever the formal type system threatens to mislead us into thinking `U` matters.

The collapse has consequences: any property that is decidable on `next` in the classical model (e.g., "the agent never enters state `s`") becomes a property of *the LLM's sampling distribution conditioned on a context*, which is in general not decidable, not stationary, and not even well-typed across model upgrades.

### 5.3 One tick

The *honest* type of one tick is stochastic in two places — the LLM sample and the tool / environment transition:

```
step : Dynamic × W → Δ(Action × Dynamic × W)
```

Operationally, one tick proceeds as follows. We write `~` for sampling and `←` for deterministic assignment.

```
step(⟨C, M, q, inbox⟩, w):

  1. Drain inbox into context.
       for each msg in inbox:
         C ← C ⊕ render(msg)
       inbox ← ∅

  2. Compose the prompt.
       relevant ← search(M, query_of(C), k)        — retrieve from long-term memory
       prompt   ← Σ ⊕ render(relevant) ⊕ C

  3. Sample from the LLM.
       tokens ~ sample(LLM, prompt; σ)             — Δ(Token*) — STOCHASTIC
       a      ← parse(tokens)                       — into one of the Action variants

  4. Execute the action's side effect.
       case a of
         ToolCall(t, args, _) → (w, r) ~ T(t).impl(args)(w);     o ← ToolResult(_, r)
         Reply(text, to)      → o ← ack
         MemOp(op)            → (M, r) ← apply(M, op);            o ← MemAck(r)
         Spawn(s, task)       → k ← fresh AgentId;                o ← Spawned(k)
         Send(to, payload)    → o ← ack       (delivery happens at the system level)
         Halt(result)         → o ← ⊥;        q ← halted

  5. Update context.
       C ← C ⊕ render(a) ⊕ render(o)                — the Wooldridge-collapsed U

  6. Termination check.
       if halt(C, M):  q ← halted

  7. Return (a, ⟨C, M, q, inbox⟩, w).
```

The randomness is concentrated in steps (3) and (4); the rest is deterministic given those samples. Hence the kernel `step : Dynamic × W → Δ(Action × Dynamic × W)`.

A few points the textbook definitions cannot make:

- **`U` is degenerate** — see [§5.2](#52-the-wooldridge-collapse). Step (5) is the entire `next` function, and it's just append.
- **Memory `M` is separate from context `C`.** Step (2) explicitly retrieves from `M` into the prompt. Classical formalisms have only one `S`.
- **Sampling makes `step` stochastic.** A *run* is therefore a sample path through `Δ`, not a deterministic trajectory.
- **The action is performed before the context is updated.** Steps (4) and (5) form an action–observation pair. This matches the ReAct loop ([Yao et al., 2022][react]) and is the operational atom of every modern LLM agent.

---

## 6. Life cycle

The life cycle of an agent is given by four operations:

```
spawn    : Static × Text → AgentId
            constructs Dynamic with C = render(task), M = M_inherited,
            q = ready, inbox = ∅; registers id in α (see §7).

run      : while q ≠ halted ∧ ¬halt(C, M):
              (a, d', w') ~ step(d, w)
              d, w ← d', w'

shutdown : release tool handles; cancel pending tool calls;
            optionally persist M; deregister from α.

cancel   : external interrupt → SystemEvent(cancel) injected into inbox;
            agent observes it on next step and may Halt gracefully.
```

Two things a real implementation needs that the textbook formalism has no slot for:

- **Suspension.** When an agent is blocked on a tool call or a reply from another agent, `q ← waiting`. The agent yields; the scheduler runs other agents. The classical model has no notion of yielding.
- **Context compaction.** When `|C|` approaches the LLM's context limit, an out-of-band rewrite

  ```
  compact : Context × Memory → Δ(Context × Memory)
  ```

  runs. `compact` is *itself* an LLM call, typically against the same `LLM` with a summarization prompt. This is a **second dynamics on `C`** that runs alongside `step`. The classical agent function has no place to put it.

---

## 7. Multi-agent configuration

Lift to a system of agents in the actor style ([Hewitt et al., 1973][hewitt]; [Agha, *Actors*, 1986][agha]). A configuration of the whole system is

```
Cfg = ⟨ α, μ, w ⟩

  α : AgentId ⇀ AgentState                  — currently live agents
  μ : AgentId ⇀ Message*                    — undelivered messages, per recipient
  w : W                                     — shared world
```

where `AgentState = Static × Dynamic`.

There are two transition rules between configurations.

```
(internal-step)
  α(i) = ⟨s, d⟩            (a, d', w') ~ step(d, w)
  let α' = α[i ↦ ⟨s, d'⟩]
  let (α'', μ') = apply_send_or_spawn(α', μ, a)
  ──────────────────────────────────────────────────────
  ⟨α, μ, w⟩  →  ⟨α'', μ', w'⟩

(deliver)
  μ(j) = m :: rest
  let d_j = dynamic_of(α(j))
  let d_j' = d_j with inbox := d_j.inbox ⊕ AgentMsg(_, m)
  ──────────────────────────────────────────────────────
  ⟨α, μ, w⟩  →  ⟨α[j ↦ ⟨static_of(α(j)), d_j'⟩], μ[j ↦ rest], w⟩
```

The `apply_send_or_spawn` helper handles the side effects of an action that touches the configuration:

```
apply_send_or_spawn(α, μ, a) =
  case a of
    Send(to, p)     → (α,                μ[to ↦ μ(to) ⊕ p])
    Spawn(s, task)  → let k = fresh AgentId
                      let d_k = init_dynamic(s, task)
                      (α[k ↦ ⟨s, d_k⟩], μ[k ↦ ∅])
    Halt(_)         → (α \ {i},          μ)
    _               → (α,                μ)
```

This is the actor-model skeleton, instantiated with LLM agents as actor behaviors. **In practice**, most current frameworks implement only the synchronous fragment: `Send` blocks until the recipient `Reply`s, and there is no `deliver` interleaving with `internal-step`. That is a strict subset of the semantics above.

---

## 8. Run / trace semantics

A **run** of a single agent is a (possibly infinite) sequence

```
(C₀, M₀, w₀) ─(a₀, o₀)→ (C₁, M₁, w₁) ─(a₁, o₁)→ (C₂, M₂, w₂) → …
```

terminating when `halt(Cₙ, Mₙ)` fires or `q = halted`. Because `step` is a Markov kernel, the *semantics* of an agent is a probability distribution over runs, given an initial `Dynamic` and an environment. Concretely, the probability of a finite run prefix is the product of the per-step kernel evaluations:

```
Pr[run = (s₀, a₀, s₁, a₁, …, sₙ)] = ∏_{k=0}^{n-1} step(s_k)(a_k, s_{k+1})
```

(writing `s_k` for the joint state `(d_k, w_k)`).

A **run of a multi-agent system** is a sequence of `Cfg`s connected by `internal-step` and `deliver` transitions. There is no global clock; interleaving is nondeterministic. **Liveness properties** ("agent `i` eventually replies", "every sent message is eventually delivered") require fairness assumptions on the scheduler — exactly as in classical actor semantics.

---

## 9. What this formalism buys, and what it does not

### What it buys

- **A clean separation of concerns**: `Static` vs `Dynamic`, percepts vs messages, context vs memory, agent-step vs message-delivery. Most bugs in real agent frameworks are confusions across these boundaries.
- **A type-correct multi-agent semantics.** You can write down what "two agents communicating" *means*, not just hand-wave it.
- **A place to hang properties you might want to check.** *Policy compliance* — every `ToolCall` satisfies a predicate on its args — is decidable per-step, even though full safety isn't. Per-step decidability is the right granularity for harness-enforced hooks ([`control-plane.md` §5](./control-plane.md#5-three-loading-strategies)).
- **A frame for engineering questions.** Where does compaction fit? Where does cancellation propagate? What does `spawn` inherit from the parent? The model gives each one a slot.

### What it does not buy

- **It does not specify `LLM` or `τ`.** Both are opaque. Any property whose proof goes through the *content* of the LLM (rather than properties of its sampling distribution) is out of reach.
- **It does not give you halting guarantees.** `halt` is a predicate the agent designer supplies; whether it ever fires depends on the LLM and the environment.
- **It does not capture training-time changes.** Fine-tuning, RLHF, prompt optimization (e.g., DSPy) change the underlying `LLM` between runs. The model is single-`LLM`; cross-model dynamics live outside it.
- **It does not formalize "intent" or "belief".** There are no BDI mental states. If you want them, project them out of `C` and `M` post-hoc; they are not primitives here.

---

## 10. Glossary

Targeted definitions of every non-trivial symbol used above. Standard mathematical notation (`∈`, `→`, `×`, `+`) is omitted.

| Symbol / term | Definition |
|---|---|
| `Δ(X)` | The set of probability distributions over `X`. A *kernel* `f : X → Δ(Y)` maps each `x` to a distribution over `Y`; `y ~ f(x)` means "sample `y` from `f(x)`". |
| `X*` | Finite sequences over `X` (Kleene star). Includes the empty sequence. |
| `X ⇀ Y` | Partial function from `X` to `Y`; undefined outside its domain. |
| `f[k ↦ v]` | The function that agrees with `f` everywhere except at `k`, where it returns `v`. |
| `xs ⊕ ys` | Sequence concatenation. |
| `Token` | A vocabulary symbol of the language model. Strings live in `Token*`. |
| `Context` (`C`) | The current working context window: a finite token sequence, `C ⊆ Token*`. Bounded above by the LLM's context limit. |
| `Memory` (`M`) | The out-of-context store: an abstract type carrying `read`, `search`, `write` operations. Concretely a vector DB, KV store, file system, or composition of these. |
| `MemoryOp` | The sum of `Read(Key)`, `Search(Query, ℕ)`, `Write(Key, Value)`. The action type wraps these in `MemOp`. |
| `Static` | The fixed part of an agent: `⟨id, LLM, Σ, T, σ, halt⟩`. What a constructor takes. |
| `Dynamic` | The mutating part of an agent: `⟨C, M, q, inbox⟩`. What `step` evolves. |
| `LLM` | The next-token Markov kernel `Token* → Δ(Token)`. Multi-token generation is its autoregressive composition; we write that as `sample(LLM, prompt; σ)`. |
| `Σ` | System prompt — a fixed token sequence prepended to every LLM call. |
| `T` | Tool registry; a partial map `Name ⇀ ToolDef`. The partiality is essential: registering a new tool extends `T`. |
| `ToolDef` | `⟨name, schema, impl⟩` where `impl : Args → (W → Δ(W × (Result + Error)))` — i.e., a tool acts on the world and may fail. |
| `σ` | Decoding parameters: temperature, top-p, max tokens, stop sequences. |
| `halt` | Termination predicate on `(Context, Memory)`. Supplied by the agent designer. |
| `q` | Control state. Values: `ready`, `thinking`, `calling_tool`, `waiting`, `halted`. Used by the scheduler in [§7](#7-multi-agent-configuration). |
| `inbox` | Per-agent FIFO of pending inter-agent messages. Drained at the start of each `step`. |
| `Percept` | The sum of things the agent observes: `UserMsg`, `ToolResult`, `AgentMsg`, `SystemEvent`. |
| `Action` | The sum of things the agent emits: `Reply`, `ToolCall`, `MemOp`, `Spawn`, `Send`, `Halt`. |
| `Env` | `⟨W, w₀, τ, observe⟩`. The world the agent acts on. `W` is opaque for real systems. |
| `τ` | Environment transition kernel: `W × Action → Δ(W × Percept)`. |
| `step` | Single-tick kernel for one agent: `Dynamic × W → Δ(Action × Dynamic × W)`. |
| `compact` | Out-of-band context-rewrite kernel: `Context × Memory → Δ(Context × Memory)`. Typically itself an LLM call. Runs when `\|C\|` approaches the context limit. |
| `Cfg` | Configuration of a multi-agent system: `⟨α, μ, w⟩`. |
| `α` | Live-agents map: `AgentId ⇀ AgentState`. |
| `μ` | Per-recipient message queues: `AgentId ⇀ Message*`. |
| `(internal-step)` | Configuration rule: one agent takes a `step` and any side effects on `α` and `μ` are applied. |
| `(deliver)` | Configuration rule: one pending message is moved from `μ(j)` into agent `j`'s `inbox`. |
| **Wooldridge collapse** | The observation that the classical state-update function `next : State × Percept → State` reduces to *append* (`U(C, a, o) = C ⊕ render(a) ⊕ render(o)`) for LLM agents. All cognitive content has migrated into the LLM sample. See [§5.2](#52-the-wooldridge-collapse). |
| **Actor model** | A concurrency model where independent actors hold private state and interact only by asynchronous messages. Originated by [Hewitt, Bishop, Steiger (1973)][hewitt]; cleanly formalized by [Agha (1986)][agha]. The multi-agent semantics in [§7](#7-multi-agent-configuration) is an instance. |
| **BDI agent** | Belief-Desire-Intention agent, the dominant pre-LLM agent paradigm; see [Wooldridge][wooldridge]. We do not use BDI mental states here. |

---

## References

The full annotated index lives in [`sources.md`](./sources.md). The references cited specifically by this document:

[wooldridge]: https://www.wiley.com/en-us/An+Introduction+to+MultiAgent+Systems%2C+2nd+Edition-p-9780470519462 "Wooldridge — An Introduction to MultiAgent Systems (2nd ed., 2009)"
[aima]: https://aima.cs.berkeley.edu/ "Russell & Norvig — Artificial Intelligence: A Modern Approach (4th ed., 2020)"
[lynch]: https://www.elsevier.com/books/distributed-algorithms/lynch/978-1-55860-348-6 "Lynch — Distributed Algorithms (1996)"
[hewitt]: https://arxiv.org/abs/1008.1459 "Hewitt — A Universal Modular ACTOR Formalism (1973; later writeup)"
[agha]: https://mitpress.mit.edu/9780262511414/actors/ "Agha — ACTORS: A Model of Concurrent Computation (1986)"
[react]: https://arxiv.org/abs/2210.03629 "Yao et al. — ReAct (2022)"

- **Wooldridge, M.** *An Introduction to MultiAgent Systems* (2nd ed., Wiley, 2009). The standard reference for BDI agents and the classical `next : S × P → S` formalization.
- **Russell, S. & Norvig, P.** *Artificial Intelligence: A Modern Approach* (4th ed., Pearson, 2020). Chapter 2 ("Intelligent Agents") for the percept-history-to-action framing.
- **Lynch, N.** *Distributed Algorithms* (Morgan Kaufmann, 1996). The standard treatment of I/O automata, with which our model deliberately contrasts.
- **Hewitt, C., Bishop, P., Steiger, R.** "A Universal Modular ACTOR Formalism for Artificial Intelligence." IJCAI, 1973. The original actor model.
- **Agha, G.** *ACTORS: A Model of Concurrent Computation in Distributed Systems* (MIT Press, 1986). The cleaner formal treatment of actors.
- **Yao, S. et al.** "[ReAct: Synergizing Reasoning and Acting in Language Models][react]." 2022. The reference for the action–observation atomic step structure used in [§5.3](#53-one-tick).

### Note on external references in the original draft

The original draft mentioned a "Goldberg" reference for the observation that most current agent frameworks implement only the synchronous fragment of the actor model. That attribution could not be verified against any of the sources in [`sources.md`](./sources.md), so the substantive observation has been kept (it is empirically true of LangGraph, AutoGen, CrewAI, OpenAI Agents SDK, and Microsoft Agent Framework as of 2026) but the attribution has been replaced with "in practice". If the original Goldberg source resurfaces, it should go in the references list above.
