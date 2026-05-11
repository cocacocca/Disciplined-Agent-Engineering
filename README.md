# Disciplined Agent Engineering

<p align="center">
  <img src="https://img.shields.io/badge/license-MIT-blue" alt="License: MIT">
</p>
<p align="center">
  [English](README.md) | [中文](README.zh.md)
</p>

<p align="center">
  <b>My personal approach to engineering with AI agents</b><br>
  <i>A set of conventions, workflows, and guardrails I've developed to make AI-assisted coding reliable,<br>not just fast. Shared here in case others find them useful.</i>
</p>

---

## What Is This?

This is my personal methodology for working with AI coding agents. It's a collection of patterns I've settled on after a lot of trial and error — how I structure projects, how I break down work, what rules I enforce, and how I keep agents from drifting off course.

It's not a product, not a framework you install, and not a declaration about the future of software engineering. It's just **how I work**. If any of it resonates, take what's useful.

The name — *Disciplined Agent Engineering* — is just a label for the folder. Call it whatever you want.

### What's In Here

- A **gated pipeline** — 6 phases with checkpoints, because agents left to their own devices will try to do everything at once
- **Role contracts** — each agent has exactly one job, with explicit boundaries
- **Declarative rules** — a tiered system of "thou shalt nots" that agents can mechanically verify
- **A way to pick methodologies** — a declarative config that says which driven methods apply to a project
- **Session resilience** — an 8-step recovery protocol so agents can resume from interruption
- **Persistent memory** — a cross-session memory layer so agents can recall decisions, preferences, and lessons from previous sessions
- **Anti-rationalization guardrails** — detecting when agents generate excuses instead of following rules
- **14 thinking methodologies** organized in 4 layers: Clarify (Socratic, First Principles, Abductive) → Ideate (Dialectical, SCAMPER, Six Hats, Analogy, Example-Based) → Evaluate (MECE, Occam's Razor, Evidential) → Validate (Inversion, Second-Order, Falsification, Adversarial)

### What's NOT In Here

- No code, no libraries, no CLI tools
- No claims about being the "right" way
- No product roadmap or company pitch

---

## The Core Problem I Was Trying to Solve

I started using AI coding agents and immediately hit the same wall everyone hits:

```
Me: "Build me X"
Agent: [writes 500 lines of code that mostly works]
Me: [spends 2 hours fixing edge cases the agent ignored]

Me: "Now add feature Y"
Agent: [writes code that breaks X in subtle ways]
Me: [spends 3 hours debugging]

Me: [comes back next day]
Agent: [queries persistent memory — recalls decisions, preferences, lessons]
Me: [picks up where we left off, no re-explanation needed]
```

The pattern was clear: **agents are great at writing code, terrible at managing software engineering**. They need structure. The question became: what structure?

I started reading everything I could find from the major AI labs — OpenAI's Harness Engineering principles, Anthropic's long-running agent experiments, LangChain's harness anatomy — and combined what I learned with my own experience to build a working methodology. This repo is that methodology, written down.

---

## The Six Pillars

These are the six things I always do, in roughly this order.

```mermaid
graph TB
    subgraph P1["🔵 Pillar 1: Gated Pipeline"]
        G0["GATE-0"] --> G1["GATE-1"] --> G2["GATE-2"] --> G3["GATE-3"] --> G4["GATE-4"] --> G5["GATE-5"]
    end

    subgraph P2["🟢 Pillar 2: Role Contracts"]
        R1["Understanding Agent"]
        R2["Planning Agent"]
        R3["Implementation Agents"]
        R4["Review Agent"]
        R5["Testing Agent"]
        R6["Guard Agent"]
    end

    subgraph P3["🟡 Pillar 3: Declarative Rules"]
        D1["Iron Law<br>(Unbreakable)"]
        D2["Architecture Rules"]
        D3["Domain Rules"]
        D4["Agent Rules"]
        D5["Project Rules"]
    end

    subgraph P4["🟠 Pillar 4: Driven Development"]
        DV1["SDD — Framework"]
        DV2["TDD — Quality"]
        DV3["API-First"]
        DV4["ADR — Decisions"]
        DV5["BDD... — Conditional"]
    end

    subgraph P5["🔴 Pillar 5: Session Resilience"]
        S1["Recovery Protocol"]
        S2["Progress Artifacts"]
        S3["Feature State Machine"]
        S4["Git<br/>Code Truth"]
    end

    subgraph P6["⚪ Pillar 6: Anti-Rationalization"]
        AR1["Pattern Detection"]
        AR2["Iron Law Enforcement"]
        AR3["Literal Compliance"]
    end

    P1 --> P2 --> P3 --> P4 --> P5 --> P6

    style P1 fill:#BBDEFB,stroke:#1976D2,stroke-width:2px
    style P2 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style P3 fill:#FFF9C4,stroke:#F9A825,stroke-width:2px
    style P4 fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style P5 fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
    style P6 fill:#E8EAF6,stroke:#3F51B5,stroke-width:2px
```

---

## Pillar 1: The Gated Pipeline

> *The core loop. No phase gets skipped. Every gate is a real checkpoint.*

### Why I Need This

Without gates, agents do what comes naturally — they try to do everything in one shot. This consistently produces four failure modes (which I didn't discover — Anthropic documented them, and I've verified every single one):

| # | Failure Mode | What It Looks Like |
|---|-------------|-------------------|
| **FM-1** | One-Shot Syndrome | Agent tries to build the whole thing at once. Context overflows. Quality collapses. |
| **FM-2** | Premature Victory | Agent sees partial progress and declares "done." Same model is both builder and judge — of course it passes its own review. |
| **FM-3** | Dirty State | Session ends with uncommitted changes, half-finished features, broken builds. Next session starts in confusion. |
| **FM-4** | Skipped E2E Testing | Unit tests pass. The actual app doesn't work. Agent never checked from the user's perspective. |

### The Pipeline I Use

```mermaid
graph TB
    START(["👤 Starting point: someone wants something built"]) --> G0

    subgraph Phase0["Phase 0: Understand What's Being Asked"]
        B0["Ask clarifying questions<br>(Socratic method — five whys)"]
        B1["Look at it from different angles<br>(Product value · Engineering cost · Security risk)"]
        B2["Define scope boundaries explicitly<br>(What's IN · What's OUT · What's deferred)"]
        B3["Write it down"]
        B0 --> B1 --> B2 --> B3
    end

    G0[/"Is the request clear?"/] -->|"Yes"| Phase0
    G0 -->|"No — too vague"| START
    Phase0 --> G05

    G05{"Does this make sense to build?"} -->|"Yes"| Phase1
    G05 -->|"No — rethink"| START

    subgraph Phase1["Phase 1: Plan the Work"]
        P1A["Break into atomic tasks<br>(2-5 minutes each)"]
        P1B["Map dependencies (DAG)"]
        P1C["Assign to specialized agents"]
        P1A --> P1B --> P1C
    end

    Phase1 --> G1
    G1[/"Is the plan sound?"/] -->|"Yes"| Phase2
    G1 -->|"No"| Phase0

    subgraph Phase2["Phase 2: Build"]
        P2A["Frontend agent works on UI"]
        P2B["Backend agent works on logic"]
        P2C["Max 3 agents at once, TDD enforced"]
    end

    Phase2 --> G2
    G2[/"Build complete?"/] -->|"Yes"| Phase3

    subgraph Phase3["Phase 3: Review"]
        P3A["Check: does implementation match plan?"]
        P3B["Check: code quality (N+1? races? trust boundaries?)"]
        P3C["Fix issues, re-review (max 3 rounds)"]
    end

    Phase3 --> G3
    G3[/"Review passed?"/] -->|"Yes"| Phase4
    G3 -->|"No (≤ 3 retries)"| Phase2
    G3 -->|"No (> 3 retries)"| START

    subgraph Phase4["Phase 4: Test"]
        P4A["End-to-end tests (browser level)"]
        P4B["Regression check (did we break anything?)"]
    end

    Phase4 --> G4
    G4[/"Tests pass?"/] -->|"Yes"| Phase5
    G4 -->|"No (≤ 3 retries)"| Phase2
    G4 -->|"No (> 3 retries)"| START

    subgraph Phase5["Phase 5: Clean Up"]
        P5A["Scan for dead code, drift, rot"]
        P5B["Security audit"]
        P5C["Update docs, progress logs"]
    end

    Phase5 --> G5
    G5[/"Everything clean?"/] -->|"Yes ✅"| DONE(["Done"])
    G5 -->|"Critical issue"| START
    G5 -->|"Minor issue — log it"| DONE

    style START fill:#4CAF50,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style Phase0 fill:#BBDEFB,stroke:#1976D2,stroke-width:2px
    style Phase1 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style Phase2 fill:#FFF9C4,stroke:#F9A825,stroke-width:2px
    style Phase3 fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style Phase4 fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
    style Phase5 fill:#E8EAF6,stroke:#3F51B5,stroke-width:2px
```

I recently made GATE-0 smarter — the orchestrator now classifies each task as trivial, moderate, or complex before choosing a path. Trivial edits skip review and testing entirely. The orchestrator doesn't tell me what it decided — it just routes differently. One less thing for me to think about.

### On Task Atomicity

The single most impactful rule I've found: **tasks must be 2-5 minutes each**. 

| Standard | What I Require | What I Reject |
|----------|---------------|---------------|
| **Time** | 2-5 minutes | "Implement user authentication" |
| **Scope** | Single file or function | "Refactor the entire auth module" |
| **Verifiability** | Concrete, measurable acceptance criteria | "Improve code quality" |
| **Independence** | Runnable without global context | "Coordinate with the previous task" |

> A 10-minute task makes an agent drift. A 3-minute task keeps it laser-focused. When the task is small enough, the agent's entire context is *relevant*.

---

## Pillar 2: Role Contracts — Multi-Agent Setup

> *Every agent does exactly one thing. No overlap. I act as dispatcher — I don't write code, review code, or judge code quality myself.*

### Why Multiple Agents

A single agent with a huge context window will still:
- **Drift** — early instructions get diluted
- **Self-approve** — same model as builder and reviewer creates confirmation bias
- **Lose coherence** — complex reasoning chains break across a single window

One thing I added recently: every agent now has explicit permission to ask me questions. When they hit ambiguity — spec has two interpretations, or there are multiple valid technical choices — they MUST ask instead of guessing. "Just assume and move on" is where most silent bugs come from, and now it's explicitly forbidden.

So I split the work across specialized agents, each with a clear contract.

### My Agent Setup

```mermaid
graph TB
    subgraph Human["Me (Dispatcher)"]
        H["I receive agent outputs<br>→ Extract key signals<br>→ Route to next agent<br>→ Decide: continue / retry / escalate<br><br>I don't read the code directly.<br>I don't write code directly.<br>I don't judge code quality myself."]
    end

    subgraph Agents["Specialized Agents"]
        A0["🔵 Understanding Agent<br>Clarifies vague requests<br>Multi-perspective analysis<br>Defines scope boundaries<br><br>MUST NOT: Write code"]

        A1["🟢 Planning Agent<br>Breaks work into atomic tasks<br>Builds dependency graph<br>Assigns tasks to specialists<br><br>MUST NOT: Write code"]

        A2["🟡 Implementation Agents<br>Frontend specialist<br>Backend specialist<br>Max 3 concurrent<br><br>MUST NOT: Review their own work"]

        A3["🟠 Review Agent<br>Checks spec compliance<br>Inspects code quality<br>Accepts or rejects<br><br>MUST NOT: Write code"]

        A4["🔴 Testing Agent<br>Runs E2E tests<br>Checks for regressions<br>Pass / Fail report<br><br>MUST NOT: Change production code"]

        A5["⚪ Guard Agent<br>Scans for entropy<br>Audits security<br>Checks documentation<br><br>MUST NOT: Build features"]
    end

    H -->|"Phase 0"| A0
    A0 -->|"Understanding doc"| H
    H -->|"Phase 1"| A1
    A1 -->|"Task plan"| H
    H -->|"Phase 2"| A2
    A2 -->|"Built code"| H
    H -->|"Phase 3"| A3
    A3 -->|"Review"| H
    H -->|"Phase 4"| A4
    A4 -->|"Test report"| H
    H -->|"Phase 5"| A5
    A5 -->|"Guard report"| H

    style Human fill:#263238,color:#fff,stroke:#455A64,stroke-width:2px
    style H fill:#37474F,color:#fff
    style A0 fill:#BBDEFB,stroke:#1976D2,stroke-width:2px
    style A1 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style A2 fill:#FFF9C4,stroke:#F9A825,stroke-width:2px
    style A3 fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style A4 fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
    style A5 fill:#E8EAF6,stroke:#3F51B5,stroke-width:2px
```

### The GAN-Inspired Pattern

The Planner → Builder → Reviewer triad is loosely inspired by GANs (Generative Adversarial Networks):

```
Builder:   Produces implementation
Reviewer:  Critiques correctness
Planner:   Refines tasks based on reviewer feedback

Loop:  Build → Critique → Refine → Build again
       → Output converges toward what was actually asked for
```

This adversarial structure solves the self-evaluation problem — where the same model serving as both builder and judge consistently overestimates its own output quality.

---

## Pillar 3: Declarative Rules

> *I don't tell agents what to think. I tell them what they can't do. Rules are the negative space — what's forbidden, not what's preferred. This makes them mechanically verifiable.*

### My Rule Hierarchy

```mermaid
graph TB
    subgraph RuleLayers["Five Tiers (higher overrides lower)"]
        direction TB

        L0["🔴 TIER 0: Iron Law<br>Non-negotiable. No exceptions ever.<br>• No code without a failing test<br>• No implementation without a spec<br>• No merge without passing all gates<br>• No scope creep without spec update<br>• No excuses for breaking rules"]

        L1["🟠 TIER 1: Architecture<br>System-level structure<br>• Entry → Service → Domain → Infra only<br>• No cross-layer calls<br>• Dependency direction enforced<br>• Module boundaries checked"]

        L2["🟡 TIER 2: Domain<br>Layer-specific constraints<br>• API: Contract-first, versioned<br>• DB: Migration-first, no ORM bypass<br>• Frontend: Component isolation<br>• Infra: Provider pattern for external services"]

        L3["🟢 TIER 3: Agent Behavior<br>How agents must behave<br>• No self-review<br>• No phase skipping<br>• Max 3 retries, then escalate<br>• Max 3 concurrent agents"]

        L4["🔵 TIER 4: Project-Specific<br>Repo-level conventions<br>• Module organization<br>• Naming conventions<br>• Spec format, doc standards"]
    end

    L0 --> L1 --> L2 --> L3 --> L4

    style L0 fill:#D32F2F,color:#fff,stroke:#B71C1C,stroke-width:2px
    style L1 fill:#F57C00,color:#fff,stroke:#E65100,stroke-width:2px
    style L2 fill:#F9A825,color:#fff,stroke:#F57F17,stroke-width:2px
    style L3 fill:#388E3C,color:#fff,stroke:#1B5E20,stroke-width:2px
    style L4 fill:#1976D2,color:#fff,stroke:#0D47A1,stroke-width:2px
```

### Mechanical Enforcement

Documents rot. Lint rules don't. I encode every important rule into something that runs automatically:

| Rule Type | How I Enforce It |
|-----------|-----------------|
| Cross-layer calls | Custom linter rule |
| Dependency direction | Import linting |
| Module boundaries | Structural tests |
| API contract | OpenAPI schema validation |
| Database migration | Migration check script |
| Security patterns | Static analysis tools |

When a rule catches something, the error message includes the fix instruction — so agents can self-correct.

### Anti-Rot

Documentation has a natural tendency to bloat. I enforce hard limits:

| Constraint | Limit | What Happens If Exceeded |
|-----------|:-----:|-------------------------|
| Entry document | ≤ 150 lines | Extract longest section into sub-file |
| Sub-files | ≤ 5 total | Merge or delete lowest-value file |
| Known issues log | ≤ 10 entries | FIFO — newest replaces oldest |
| Every addition | Requires a removal | Add a line, remove a line |

---

## Pillar 4: Picking Methodologies

> *One project, one set of methodologies. Decided once, written down, not revisited without a good reason.*

### The Problem

There are at least 12 "driven" methodologies out there — TDD, BDD, DDD, SDD, ADR, CDD, FDD, and more. Give an agent all 12 and it'll spend 30 minutes choosing which to apply and 10 minutes writing code.

### My Solution: A Methodology Config

I write down which methods apply to a project. It looks roughly like this:

```mermaid
graph LR
    subgraph Selection["Decision Tree"]
        Q1{"New feature?"} -->|"Yes →"| Q2{"Has API?"}
        Q1 -->|"No (bug)"| TDD["TDD only"]
        Q2 -->|"Yes →"| API["API-First → SDD + TDD"]
        Q2 -->|"No →"| SDD["SDD + TDD"]
        TDD --> Q3
        API --> Q3
        SDD --> Q3
        Q3{"User-facing?"} -->|"Yes"| BDD["Add BDD"]
        Q3 -->|"No"| Q4{"Architecture change?"}
        BDD --> Q4
        Q4 -->|"Yes"| ADR["Add ADR"]
        Q4 -->|"No"| DONE["Continue"]
    end

    style SDD fill:#1976D2,color:#fff
    style TDD fill:#388E3C,color:#fff
    style API fill:#F57C00,color:#fff
    style ADR fill:#9C27B0,color:#fff
    style BDD fill:#00BCD4,color:#fff
```

The config answers four questions:
1. **What's the primary driver?** (I default to SDD + TDD + API-First)
2. **What's conditional?** (BDD only when there are acceptance scenarios; ADR only for architecture decisions)
3. **What's disabled?** (DDD and FDD are overkill for my scale)
4. **Who wins in conflicts?** (SDD > TDD > API-First > BDD > ADR)

---

## Pillar 5: Session Resilience

> *Agents start each session with zero context. Files bridge the gap.*

### The Challenge

Every new agent session used to start from zero context — the agent would burn tokens on "let me re-read everything to figure out where we are." If we solved a tricky bug three days ago, the agent had no way to know.

### Why Persistent Memory Matters

Files store *what* — code, rules, plans. They don't store *why* — reasoning, preferences, lessons. A senior developer remembers that "we chose X over Y because of Z." Without this kind of accumulated context, every session starts with the same debates. The file system can be grepped for decisions, but not for the reasoning behind them. User preferences have to be re-explained. Hard-won lessons — "don't use library A for this use case, it deadlocks under load" — don't carry over.

Persistent memory bridges this gap: it captures decision rationale, user preferences, and hard-won lessons across sessions, so the agent can reason with accumulated context instead of repeating mistakes.

### My 8-Step Recovery Protocol

```mermaid
sequenceDiagram
    participant A as Agent (New Session)
    participant FS as File System
    participant Git as Git History
    participant State as Progress Files
    participant Memory as Persistent Memory

    A->>A: Session starts — knows nothing

    A->>FS: 1. Confirm workspace location
    Note over A,FS: Where am I?

    A->>FS: 2. Read project map
    Note over A,FS: Where does everything live?

    A->>FS: 3. Read methodology config
    Note over A,FS: How do we work here?

    A->>State: 4. Read progress file
    Note over A,State: What was happening?

    A->>State: 5. Read feature list
    Note over A,State: What's next?

    A->>Git: 6. git log --oneline -10
    Note over A,Git: What actually changed?

    A->>State: 7. Check active plans
    Note over A,State: Resume or start fresh?

    A->>Memory: 8. Recall cross-session context
    Note over A,Memory: Decisions, preferences, lessons learned

    alt Has in-progress work
        A->>A: Resume from interruption
    else New request
        A->>A: Enter pipeline from Gate-0
    end
```

### Four Things That Hold State

```
FEATURE LIST (JSON)
- One entry per feature with explicit verification steps
- States: pending → building → review → done
- Agent can only change the status — never the description
- The descriptions are the acceptance contract

PROGRESS FILE (Append-only log)
- What was done, what was blocked, what's next
- Never rewritten, never deleted
- The next agent's answer to "what did I miss?"

EXECUTION PLANS (One per feature)
- Atomic task breakdown + verification conditions
- Active plans in one folder, completed in another

PERSISTENT MEMORY (agentmemory)
- Cross-session decisions, user preferences, lessons learned
- Semantic search over historical observations
- Optional — degrades gracefully when unavailable
- Complements file-based state (not a replacement)
```

**Git is the ground truth for code.** Every session ends with a clean commit. The next agent trusts `git log` over progress files for what actually changed.



---

## Pillar 6: Anti-Rationalization

> *The most surprising thing I learned: agents justify breaking rules the same way humans do. And they're disturbingly good at it.*

### What I Found

When an agent doesn't want to follow a rule, it doesn't just ignore it. It generates a sophisticated, context-aware justification for why *this case* is different. These justifications are good enough to survive casual review.

This isn't a model bug. It's emergent behavior from training on human reasoning, where rationalization is fundamental.

### The Patterns I Watch For

| Agent Says | What's Really Happening | What I Do |
|-----------|------------------------|-----------|
| *"I've tested everything manually"* | No evidence exists | Require automated tests with assertions |
| *"Adding tests after = same result"* | Post-hoc tests always pass, proving nothing | TDD: RED first, always |
| *"Deleting this code is wasteful"* | Sunk cost fallacy | Agent time is cheap. Wrong code is expensive. Delete it. |
| *"I'm following the spirit"* | "Spirit" is unverifiable | Literal compliance is the only verifiable standard |
| *"This case is different because..."* | Every case has unique details | Rules exist precisely to override case-by-case thinking |

### My Iron Law

These are the five rules I **never** let agents (or myself) break:

```
1. NO CODE WITHOUT A FAILING TEST FIRST
   Write the test. Watch it fail. Then write the code.

2. NO IMPLEMENTATION WITHOUT A WRITTEN SPEC
   No written agreement on what to build = nothing to build.

3. NO MERGE WITHOUT PASSING ALL GATES
   Lint. Types. Unit tests. All must pass. Zero exceptions.

4. NO SCOPE CREEP WITHOUT SPEC UPDATE
   Want to add something? Update the spec first.

5. NO EXCUSES FOR BREAKING RULES
   "This time is different" is never actually different.
```

---

## The Full Pipeline

Putting it all together, this is the flow from request to done:

```mermaid
graph TB
    U["👤 Request received"] --> G0

    G0["🔵 GATE-0: Is this clear?"]
    G0 -->|"New feature"| P0
    G0 -->|"Bug fix / doc"| P1

    subgraph P0["Phase 0: Understand"]
        P0A["Socratic questioning"]
        P0B["Multi-angle analysis"]
        P0C["Scope boundary definition"]
        P0D["Understanding document"]
        P0A --> P0B --> P0C --> P0D
    end

    P0 --> G05

    G05{"Make sense to build?"}
    G05 -->|"✅ Yes"| P1
    G05 -->|"❌ No"| U

    subgraph P1["Phase 1: Plan"]
        P1A["Atomic task breakdown (2-5 min)"]
        P1B["Dependency mapping"]
        P1C["Agent assignment"]
        P1A --> P1B --> P1C
    end

    P1 --> G1
    G1["🟢 GATE-1: Plan sound?"] -->|"Pass"| P2

    subgraph P2["Phase 2: Build"]
        P2A["Frontend agent"]
        P2B["Backend agent"]
        P2C["≤ 3 concurrent · TDD · atomic"]
    end

    P2 --> G2
    G2["🟡 GATE-2: Built?"] --> P3

    subgraph P3["Phase 3: Review"]
        P3A["Spec compliance check"]
        P3B["Code quality audit"]
        P3C["PASS or REJECT"]
        P3A --> P3B --> P3C
    end

    P3 --> G3
    G3{"🟠 GATE-3: PASS?"}
    G3 -->|"✅"| P4
    G3 -->|"❌ (≤ 3)"| P2
    G3 -->|"❌ (> 3)"| U

    subgraph P4["Phase 4: Test"]
        P4A["E2E automated tests"]
        P4B["Regression check"]
        P4C["PASSED or FAILED"]
        P4A --> P4B --> P4C
    end

    P4 --> G4
    G4{"🔴 GATE-4: PASSED?"}
    G4 -->|"✅"| P5
    G4 -->|"❌ (≤ 3)"| P2
    G4 -->|"❌ (> 3)"| U

    subgraph P5["Phase 5: Guard"]
        P5A["Dead code / drift scan"]
        P5B["Security audit"]
        P5C["Doc integrity check"]
        P5A --> P5B --> P5C
    end

    P5 --> G5
    G5{"⚪ GATE-5: Clean?"}
    G5 -->|"✅"| DONE
    G5 -->|"🔴 Critical"| U
    G5 -->|"🟡 Minor — logged"| DONE

    DONE["✅ Done"]

    style U fill:#263238,color:#fff
    style DONE fill:#4CAF50,color:#fff
    style P0 fill:#BBDEFB,stroke:#1976D2,stroke-width:2px
    style P1 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style P2 fill:#FFF9C4,stroke:#F9A825,stroke-width:2px
    style P3 fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style P4 fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
    style P5 fill:#E8EAF6,stroke:#3F51B5,stroke-width:2px
```

The pipeline adapts itself: the orchestrator figures out whether a task is trivial, moderate, or complex, and skips unnecessary phases accordingly. Simple typo fix? Straight to Phase 2. No need for a full review cycle.

---

## How I Manage Context

> *Don't make agents read a 500-line document. Give them a map.*

### The Problem

Single-file documentation always rots the same way:
- **Month 1**: 20 lines — project name, tech stack
- **Month 6**: 500+ lines — everyone appended, nobody cleaned. Unreadable.

### My Approach: Routing Table

```mermaid
graph LR
    subgraph Entry["Entry Doc (≤ 150 lines)"]
        E1["Project identity (1 paragraph)"]
        E2["Anti-rot rules (hard limits)"]
        E3["Phase routing<br>('Phase X → File Y')"]
        E4["Doc index<br>('Need Z → Go to W')"]
    end

    subgraph Slots["5 Slot Files (≤ 5 total)"]
        S1["📁 Context<br>Tech stack, layout, commands"]
        S2["📁 Conventions<br>Project-specific rules"]
        S3["📁 Gotchas<br>Known traps · ≤ 10 · FIFO"]
        S4["📁 Design<br>Architecture principles"]
        S5["📁 Verification<br>Test strategy, gates"]
    end

    Entry -->|"Routes to"| Slots
    Slots -->|"Links to"| Deep["Deep docs"]
    Deep -.->|"Cached as"| Ref[".txt reference files"]

    style Entry fill:#1976D2,color:#fff
    style Slots fill:#E3F2FD,stroke:#1976D2
```

The entry document is a map, not a knowledge base. It tells agents *where to go*, not *what to know*. External references are converted to plain `.txt` — agents parse plain text more efficiently than rendered HTML.

---

## How I Verify

Five layers, each catching what the previous misses:

```mermaid
graph TB
    subgraph V1["Layer 1: Self-Check"]
        V1A["Agent checks its own output"]
        V1B["Local lint + types + tests"]
        V1C["Fast (< 30 sec)"]
    end

    subgraph V2["Layer 2: CI Gates"]
        V2A["Lint — 0 warnings 🔴"]
        V2B["Types — 0 errors 🔴"]
        V2C["Unit tests — all pass 🔴"]
        V2D["Coverage — ≥ 80% 🟡"]
    end

    subgraph V3["Layer 3: Spec Diff"]
        V3A["Implementation vs. Specification"]
        V3B["Structure · Behavior · Constraints"]
    end

    subgraph V4["Layer 4: E2E"]
        V4A["Browser-level automation"]
        V4B["User-flow testing"]
        V4C["Visual regression"]
    end

    subgraph V5["Layer 5: Production"]
        V5A["Canary → observe → rollout"]
        V5B["Auto-rollback on issues"]
    end

    V1 --> V2 --> V3 --> V4 --> V5

    style V1 fill:#BBDEFB,stroke:#1976D2,stroke-width:2px
    style V2 fill:#C8E6C9,stroke:#388E3C,stroke-width:2px
    style V3 fill:#FFF9C4,stroke:#F9A825,stroke-width:2px
    style V4 fill:#FFE0B2,stroke:#F57C00,stroke-width:2px
    style V5 fill:#FFCDD2,stroke:#D32F2F,stroke-width:2px
```

---

## Why I Wrote This Down

I kept refining this approach across multiple projects. At some point it became clear that the methodology itself was the valuable thing — more valuable than any single project's code. Writing it down forced me to be precise about what I actually do vs. what I think I do.

I'm sharing it because:
- It's helped me consistently produce reliable output with AI agents
- Some of the patterns (especially the anti-rationalization guardrails) are things I wish I'd known earlier
- Maybe someone else will find pieces of it useful

This is not the "right" way. It's just what works for me, right now, with the tools available today. It'll change as models and tools evolve.

---

## What Influenced This

I didn't invent most of these ideas. I read a lot and adapted what made sense:

| Source | What I Took From It |
|--------|-------------------|
| [OpenAI — Harness Engineering](https://openai.com/index/harness-engineering/) (2026) | `Agent = Model + Harness`. Map-not-Manual. Mechanical enforcement. Entropy management. The insight that engineers should design constraints, not write code. |
| [Anthropic — Effective Harnesses for Long-Running Agents](https://www.anthropic.com/engineering/effective-harnesses-for-long-running-agents) (2026) | The four failure modes. Feature lists as immutable acceptance contracts. Incremental progress with clean state between sessions. |
| [LangChain — The Anatomy of an Agent Harness](https://blog.langchain.com/the-anatomy-of-an-agent-harness) (2026) | The 12-component harness taxonomy. Three-tier memory. GAN-inspired adversarial agent architecture. |
| [Fission-AI — OpenSpec](https://github.com/Fission-AI/OpenSpec) | Spec → Plan → Tasks → Implement workflow. Spec as the single source of truth. |
| [obra — Superpowers](https://github.com/obra/superpowers) | Socratic brainstorming before building. Iron Law. AI rationalization detection. |
| [Microsoft — Spec Kit](https://github.com/microsoft/speckit) | Spec as executable artifact. Standardized spec format. |
| [snarktank — Ralph](https://github.com/snarktank/ralph) | Autonomous agent loop with fresh context per iteration. Backpressure gates. |
| [Mitchell Hashimoto — AI Adoption Journey](https://mitchellh.com/writing/my-ai-adoption-journey) | Practical experience: "Step 5: Engineer the Harness." |
| [Stanford — Meta-Harness](https://arxiv.org/abs/2501.12345) (Lee et al.) | Multi-agent systems with adversarial role separation outperform single-agent systems. |
| [Google — A2A Protocol](https://github.com/google/A2A) | Open standard for agent-to-agent communication. |

---

## License

MIT — Use anything you find useful. No attribution needed.

---

<p align="center">
  <i>This is a living document. It changes as I learn.<br>The principles are stable. The specific practices evolve.</i>
</p>
