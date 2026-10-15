# Igor Loskutov — Profile Knowledge Base

Reusable source-of-truth for CVs, resumes, "tell me about yourself" prompts, applications.
Grounded in git repos (`~/work`), resume (`~/work/resume`), blog (loskutoff.com), testimonials.

> **What may be claimed is governed by `FACTS.md`** in the EI job-search folder
> (`~/Library/Mobile Documents/com~apple~CloudDocs/ei/FACTS.md`). It carries per-project commit
> shares from a codebase audit, plus a list of claims that failed that audit and must never be
> used (React Native at FiddleQuest, "sole builder", GraphQL at an employer, AWS at Monadical,
> design systems, WCAG, Core Web Vitals, A/B testing, analytics, "enterprise client", any lead
> title). This KB holds framing and interview answers; FACTS.md decides what is true.

**Why this doc exists:** job applications repeatedly ask the same kinds of things
(summarize experience, show what you built, link code, state your stack, your story,
your passions, your opinions). This KB holds the validated answers once, so any specific
CV / application can be assembled from facts instead of re-improvised each time.

**The scaffold:** the section headers below come from one representative example of such
an ask — a real application text Igor received, quoted verbatim here so the structure is
self-explanatory. It is a SCAFFOLD ONLY (what categories of info to collect), not a task
to write a reply to it:

> "Send a plain text cover letter. Help us see how you stand out. Summarize your
> experience. Link to 1–5 impressive things you've built and are proud of, link to where
> we can see some of your code, include your portfolio/CV, describe the platforms and
> stacks you're an expert in. How did you start programming? What are you most passionate
> about in technology? What are the most interesting or weird things you've done? What are
> your strongest held professional opinions? Please make your email easy to read."

Status legend: ✅ grounded in artifacts · ❓ needs user input (grilling) · 🔶 inferred, confirm.

> ⚠️ **RULES FOR THIS DOC.** No invented prose. Only factual, validated info.
> Every addition or change must be reviewed by Igor manually before it's trusted.
> 🔶 = inferred/unconfirmed — do NOT treat as fact until Igor signs off.
> Do not write marketing/embellished language here; raw verifiable facts only.

---

## 0. Scope
General reusable KB. NOT a CV or an application for any specific company. The list of
points above is only a scaffold for what to extract. No per-application targeting.

---

## 1. Experience summary ✅
- Senior Full Stack / Frontend. 12+ yrs frontend, 10+ yrs React. Montreal, QC (remote/hybrid).
- **Monadical** (2022–2026) — Senior Full Stack Engineer (title per contract; no lead title). Primary client rep 3+ yrs. Ended Mar 2026 (ROE: A00 shortage of work / end of contract).
- **DesignerAdvantage** (2021–22) — Senior Frontend Engineer (contract). Stabilized a struggling React codebase (state-mgmt/data-flow cleanup + react-query, reducing recurring bugs); Salesforce CRM integration; feature delivery during the client's funding round (contribution, not proven cause). 🔶 **Repo not on this machine — specifics unverified by git; "rescue" framing softened pending repo access.** Client folder reportedly named "james" in `clients/` (absent here).
- **FiddleQuest** (2014–2023) — Lead Dev. React early adopter (2014). 9-yr full-stack platform for violin teachers/students. **More than it looks: a distributed event-driven system across 3 apps** — Meteor web + Phaser/Ink game + native iOS — glued by an AWS **SNS→SQS→DynamoDB→S3** mesh (sole author of the event-mesh files; top author of the game repo). Idempotent SQS consumption, SNS fan-out + HTTP webhooks, DynamoDB badge store bridged into Meteor reactivity via RxJS. Native iOS over Meteor DDP/WebSocket; S3 presigned video upload. Stripe; S3/CloudFront/ElasticTranscoder media. Defensibility **T1**. Deep-dive: STORY.md "FiddleQuest LLC" section.
- **Ethereum DEX (NDA)** (2017–18) — React/Redux trading UI, MetaMask + hardware wallets.
- Long Upwork track record (2015–2020): Java/Lombok, Rails, AngularJS, Node, fleet tracking, Python/Pyramid. Strong testimonials (see §10).
- Education: BSc Applied Informatics, Togliatti State University (2014–18). Oracle Certified Java SE.

## 2. Proud-of projects (candidate set, curate per target) ✅
1. **huly-mcp** (OSS, 2026) — MCP server, stable typed contract over Huly's moving internal API. **Built on Effect** (`effect` ^3.19; `@effect/platform`, `@effect/platform-node`, `@effect/vitest`; 142 src files import `effect`) — core stack, strong T1 corroboration for Effect.ts. 3,467 npm downloads/30d, 30 stars, official MCP registry. 18 releases in 3mo, solo. → real external adoption. **Most capable in its field:** 238 tools / 28 categories; `huly-mcp-watch` sibling repo auto-tracks 22 competitor repos + 12 npm pkgs weekly; competitor roadmap deltas "mostly absorbed" (rivals lead only on workflow-synthesis/distribution extras). github.com/dearlordylord/huly-mcp · npm @firfi/huly-mcp
2. **dnd / rules-as-code** (2026) — Quint (TLA+-derived) spec → Apalache model-check → trace-replay against TS runtime (`quint-connect`) → MCP + XState React app. Article hit HN front page (109 pts, 76 comments). Generalizes to pricing/permissions/claims/workflow. Repos (public): github.com/dearlordylord/5e-quint (core runtime) · github.com/dearlordylord/savage-visual-rules (related). Resume bullets/cover letters should link 5e-quint when citing this project.
3. **Currents / Graphuier** (Monadical) — real-time payment-flow graph viz; reactive dataflow (thi.ng/rstream-graph), d3-force, snail-trace routing algo; 99-lib Nx monorepo; double-entry accounting (Assets=Liabilities invariant).
4. **Reflector** (Monadical, Python) — AI transcription platform; Hatchet DAG durable execution (1,341-line pipeline); Daily.co multitrack audio sync via PyAV; process-fork-safe connections. **Defensibility (see §5b): T2/T3 — Python self-rated basic, no independent depth signal.** Defend Hatchet durable execution + LLM pipeline at *architecture/decision* level (he led the Celery→Hatchet migration). PyAV = real but shallow (approach, not FFmpeg internals); fork-safety/ContextVar = real but NOT deeply defensible — breadth only, don't headline.
5. **Scope3 aiapi** (Monadical, Go) — LLM env-impact; "flow-as-data" StaticGraphNode → auto-generated Mermaid diagrams (code IS the docs); 25+ node inference cascade; go-jet type-safe SQL.
6. **headless-chat** (personal) — Discord-grade real-time chat lib; cursor-based reconciliation, composite message IDs, multi-room, gap recovery; fp-ts/io-ts/ts-pattern.
- **Proudest: Currents** (from Igor). Reason: he made sense of their multi-layer error groups.
  The system had accumulated many issues over time; prior fix attempts were batch jobs that
  *modified the data* to patch problems, sometimes making things worse. He was the one who
  actually diagnosed/fixed the root problem first — where others had only made it worse.
  - ❓ sharpen later: what "multi-layer error groups" concretely were (ledger/accounting data
    corruption from bad batch fixes?) and what the actual fix was.

## 3. Where to see code ✅
- GitHub: github.com/dearlordylord (96 public non-fork repos)
- npm: @firfi/huly-mcp
- **Best FULL-AI-built repo (Igor's call): `huly-mcp`** — point here for "AI-assisted, high quality."
- **Manual (non-AI) code samples — Igor's own caveat: nothing is polished to showcase grade.**
  Assessed candidates:
  - `sstackrr-backend` + `sstackrr-frontend` (2022 Monadical take-home assignment). Best *full-stack
    manual* showcase: Rust GraphQL backend (~1.6k LOC) — minimax + alpha-beta pruning bot, multithreaded,
    Diesel/Postgres; React/TS frontend (~1.3k LOC) with Storybook + Chromatic. Organized, but dated (2022),
    test coverage unverified.
  - `jikan` (~3.6k LOC, 2024) — best showcase of his *signature style*: exercise timers as immutable
    state machines, FP, Nx monorepo (fsm core / adapters / react), time-concern abstracted, some specs.
    But README is TODO-heavy / explicitly unfinished.
  - Other public notables (not deep-assessed): `quint-connect-ts` (8★, real lib), `5e-quint` (31★),
    `shades-of-rust` (5★), `parsers-jamboree` (6★), `react-uuid-hook` (8★).
- RECOMMENDATION: for code quality, lead with `huly-mcp` (AI) + `sstackrr` (manual full-stack) +
  `jikan` (manual FP/state-machine style). Caveat the manual ones as not showcase-polished.

## 4. Portfolio / CV ✅
- Site/blog: loskutoff.com (blog posts + commendations)
- Resume PDFs: ~/work/resume/igor_loskutov_resume_frontend.pdf (+ general, java variants)

## 5. Stacks — expert vs familiar 🔶 (confirm honesty boundaries)
- Expert: React (10+ yr), TypeScript, functional TS (fp-ts, effect, io-ts, immutable.js, ts-pattern), Node.
- **AI coding tools actually used (AUTHORITATIVE — never invent or substitute others):** **Claude Code** (primary), **Codex** (regular), **opencode** (less). **Does NOT use GitHub Copilot — never list Copilot on any CV/cover letter.** AI-assisted-dev credibility leads with the agentic-harness PROJECTS — **huly-mcp (§2.1) and 5e-quint (§2.2)** as his successful agentic-harness builds — NOT the "vibe-code" guide (§11), which is minor/secondary.
- Strong: Nx/monorepo architecture, property-based testing (fast-check), state machines (XState), WebSocket/real-time, Stripe, AWS (Lambda/S3/SQS/DynamoDB).
- Strong (infra/DevOps — primary author across two prod systems, not just exposure):
  - **Currents** (`clients/monadical/infrastructure`, 2023–24): **Terraform** (DigitalOcean + Cloudflare, per-app modules + tfstate envs) + **Kubernetes** (per-app Deployment/Service/Ingress manifests). Git: top overall committer (480), most `.tf` commits of any author (116), 36/37 `kubes/` commits. Plus Helm exposure (crawler2, morpheus EKS).
  - **FiddleQuest** (`clients/darren/fiddlequest`): **Ansible**-provisioned VPS on **DigitalOcean** (15 roles — meteor, node+systemd, mongo, redis, nginx, ssl, kue, backup; staging/prod inventories) + **AWS** media pipeline (S3 us-west-2, CloudFront CDN, DynamoDB, SNS, SQS, ElasticTranscoder, Transloadit). 187 commits, sole author.
  - Net: provisioning (Ansible) → IaC (Terraform) → orchestration (K8s), across **AWS + DigitalOcean**. Honest ceiling: single-cloud-per-system, no managed-EKS/GKE-at-scale or dedicated SRE title.
- Weak (worked with, don't lead): **GCP** — optimized + migrated a Currents service *off* a scalable GCP service; touched Scope3 Cloud Run. No GCP ownership. **No Azure.**
- **Honest self-rating (from Igor):**
  - **Python — basic, eager to expand.** (His framing; keep it. Even though he shipped FastAPI/PyAV/
    Hatchet async work in Reflector, he does not call himself more than basic.)
  - **Go — average.** Wrote it in production (Scope3 aiapi).
  - **Rust — average.** Pre-AI-era enthusiast; dabbled both web (GUIs/WASM) and a game engine
    (bevy), plus a bit of Solana. "Wrote them all" but not expert-level.
- formal methods (Quint/TLA+/Apalache) — actively learning/using, not claiming mastery.
- **Next.js — listed on a resume ONLY when a vacancy explicitly requires it.** Igor actively
  **disdains Next.js and recommends against it**; his preference is **TanStack** (Router/Start/Query) or
  **Effect HTTP** (`@effect/platform` HttpApi). So "Next.js" on a CV is keyword-compliance, not advocacy
  — never frame him as a Next.js enthusiast. (Real exposure: Reflector `www` is Next.js 14; competent,
  not deep — 🔶 confirm how much he personally authored.)

## 5b. Defensibility — real vs defendable (READ before building ANY resume) ✅
Two **independent** axes for every achievement — never collapse them into one rating:
- **Real?** = genuinely his work (authorship/existence).
- **Defensible?** = can he hold up under deep questioning in a senior interview.
- A thing can be **Real but NOT Defensible.**

**Defensibility requires an INDEPENDENT DEPTH SIGNAL** — a talk, a blog post, public OSS with real
users, or a self-rated-strong stack. It is **NOT** established by "code exists in the repo" or
"STORY.md documents it in detail." (STORY.md is reverse-engineered interview *prep* — it proves the
artifact exists, **not** that Igor can recall/defend it.)

**The tell:** scan §11. Every talk/blog is **TS / FP / formal-methods / testing / Rust**. There is
**zero** Python, async, or concurrency content anywhere → that is the hard boundary of deep
defensibility. Anything Python-deep is T2/T3 below regardless of how much code or doc exists.

**Tiers:**
- **T1 — defend anywhere (real + independent depth signal):** React/TS; functional TS
  (fp-ts/io-ts/ts-pattern/immutable/effect); property-based testing (fast-check); XState/state
  machines; Nx/monorepo; MCP authoring (huly-mcp — public, real users); Quint/XState formal methods
  (HN + blog). Backed by talks/blogs/public OSS.
- **T2 — defend at ARCHITECTURE / DECISION level only (real, he made the calls, not deep internals):**
  Reflector backend — Hatchet durable execution + multi-stage LLM pipeline (he led the design +
  Celery→Hatchet migration, so defend the *why*); infra/DevOps — Terraform/K8s/Ansible (primary
  author; defend ops *decisions*, not provider internals); Go (Scope3, self-rated average).
- **T3 — real but NOT deeply defensible → breadth/concept-level only, never a headline:** Python
  language & library *internals* — PyAV/FFmpeg filter graphs, fork-safety/ContextVar, deep asyncio.
  Reason: Python self-rated basic AND no independent depth signal — depth rests on recall he lacks.

**Rule when assembling any CV:** lead with T1; pitch T2 at the decision/architecture altitude; use T3
as breadth only, never as a defended claim. **Do not let artifact density (LOC, STORY.md detail)
promote a T3 item to T1.**

## 6. How I started programming ✅ (from Igor)
- First thing ever: writing macros for a Perl bot for a niche Korean MMORPG.
- Self-taught. (Degree 2014–18 came later / alongside paid work, not the origin.)
- Throughout: consistently driven by "how to make the craft better."
- NOTE: Igor says the "what made me go into it" angle is low-value — do not over-emphasize
  the origin in any output.

## 7. Most passionate about ✅ (from Igor)
Two drivers, in his words:
- **Exploring new frontiers** (e.g. formal methods in real production code, LLM-agent tooling early).
- **Proving he can do better than what the industry has done for decades** — improving the craft itself.
- "Why I stayed in it": the formalism and strictness of program code fits how his mind works —
  something he can do tirelessly, relentlessly, for days, and is good at.
- **What "better than industry" means concretely:** the industry's default *imperativeness and
  mutability* — Igor believes that's what made codebases hit a wall on **feature scalability**.
  His counter-bet is functional / immutable design that keeps scaling as features pile on.
- **Live frontier right now:** executable specs as the source — "can I generate a library in *any*
  language out of it" by writing executable specifications around his TS. (Related to the Quint /
  rules-as-code / quint-connect work; see §2.2.)
- **Relationship of the two drivers:** they are two *separate, independent* goals — not one in
  service of the other. (Frontier exploration ≠ beating the industry; both just true of him.)
- **Evidence the functional/immutable bet pays off (Igor's cited proofs):**
  - Rewrote **Reflector** core logic to **Effect** — "immediately better."
  - **huly-mcp** follows the best practices he established and leads its competitive field:
    the `huly-mcp-watch` repo (sibling to huly-mcp) auto-tracks **22 competitor GitHub repos +
    12 npm packages** weekly; his `competitor-archive/COMPARISON.md` audit puts huly-mcp at
    **238 tools / 28 categories**, with competitor roadmap deltas "mostly absorbed" — rivals only
    lead on workflow-synthesis/distribution extras, not Huly API coverage. (Igor recalled "~15
    competitors"; tracked set is actually 22 repos.)
  - **DesignerAdvantage** (2021–22) — clear earlier case: rescued a failing React codebase by
    fixing state-management / data-flow that was causing cascading bugs. Plus many more.
- **Q4b mechanism (executable specs → any-language lib):** the generation works by *harnessing AI
  coding-agent output* — the executable spec constrains/verifies what the agent produces, so the
  agent can emit the lib in any language and the spec checks it. NOT deterministic codegen; the
  spec is the oracle, the agent is the generator. STATUS: in-flight, unconfirmed — no working
  cross-language generation yet; do not claim it as realized.
- 🔶 supporting evidence (themes across blog/talks): type-driven correctness, parse-don't-validate,
  errors-as-values, immutability, formal methods made practical, making complex domains *visible*.

## 8. Interesting / weird things — DEPRIORITIZED by Igor (low value, skip for now)
- 🔶 UNCONFIRMED candidates only (do NOT use until Igor validates): formal-methods-on-D&D,
  Rust GUIs in WASM survey, Elm→RN multitouch physical-token detection (circumcircle geometry,
  capacitive clustering), Urbit/Hoon dabbling, first-ever Perl-bot macros for a Korean MMORPG (§6).

## 9. Strongest professional opinions ✅ (from Igor — exact stances; captured so far, not exhaustive)
1. **Imperativeness + mutability is the root cause of the feature-scalability wall** (see §7). The
   functional/immutable counter-bet keeps scaling as features pile on.
2. **Naming things matters; explicit over implicit.** Primitive-typed fields are NOT
   self-documenting: `email: string`, `count: number` document nothing. (Pushes toward meaningful,
   domain-named, branded/nominal types.)
3. **System-design / domain perspective is mandatory.** Without domain design, ubiquitous language,
   and anchoring in a defined & documented business language, an app has no good documentation.
4. **Errors as values:** "depends, but often yes." (Not absolute.)
5. **Types are PART of the spec** — a very important part, but not the whole spec. (Do NOT state as
   "types ARE the spec.")
6. **Docs should be generated from / synced with code by best effort.** Refined form: docs are
   effectively dead "when the best effort to generate or sync with code in some other way is not
   done." (Not the absolute "non-generated docs are dead on arrival.")
7. **Recommends AGAINST Next.js.** Disdains it; prefers **TanStack** (Router/Start/Query) or
   **Effect HTTP** (`@effect/platform`). Lists Next.js on a CV only when a posting demands the keyword —
   compliance, not advocacy. (Don't surface this opinion *to* a Next.js-shop interviewer; it's a held
   stance, the CV keyword is pragmatic.)

> CORRECTION LOG — claims the assistant invented and Igor rejected (DO NOT use):
> - "hand-written example-based tests are mostly theater" — fabricated; he does NOT hold this.
> - "most pragmatism is unmanaged tech debt" — fabricated.
> - "types are the spec / if it compiles it's correct" — overstated; see #5.
> - "docs not generated from code are dead on arrival" — overstated; see #6.
> - "uses GitHub Copilot" — FABRICATED (2026-06-12, Toon Boom resume draft). He uses Claude Code + Codex + opencode, NOT Copilot. Authoritative AI-tooling fact now in §5.

OFF RECORD (do NOT publish anywhere — Igor flagged): his most-pushback experiences (generated docs,
branded types, immutability adoption friction). Known publicly only that he *advocates* these; the
friction commentary is private.

## 10. Social proof / testimonials ✅
- "one of the best developers we ever met… broad skill set, solved all tasks" (Yarnee, 2016)
- "surpassed our expectations… one of the most knowledgable developers" (fleet tracking, 2015)
- "entirely reliable, highly skilled, trustworthy… a perfect contract partner" (Darren Jahn, FiddleQuest, 2018)
- "excellent all-rounder… great advisor, architect, full-stack" (TheHireStreet, 2020)

## 11. Talks & writing ✅
- **PROUD-OF articles (Igor's pick — lead with these):** **Property-based testing for temporal graph
  storage** (Monadical) and **TypeScript Validators Jamboree** (Monadical). Plus Model-Based Testing /
  Quint+XState (HN). Feature these in "writing I'm proud of."
- HN front page: Model-Based Testing of UI Logic via Quint + XState (109 pts).
- Monadical blog: TypeScript Validators Jamboree; Property-based testing for temporal graph storage; Vibe-code how to stay in control.
  - **"Monadical" carries no name recognition — don't cite "on the Monadical blog" as if it lends authority.** It reads as a casual in-reference to an unknown company and adds nothing. Either let the hyperlinked title stand on its own, or describe the piece by what it is ("a write-up on staying in control of an AI-assisted codebase"). Name Monadical only where the employer context already makes it relevant, not as a credibility prop.
  - **"How to stay in control of your vibed codebase" — include ONLY in AI-related vacancies.** It's an
    AI-assisted-dev piece; off-topic (and slightly off-brand) for non-AI roles. Drop it from non-AI CVs.
    **Relatively weak as writing** — its value is topical match to AI-native JDs, not prose quality. Lead
    with it only because it's the most on-target AI credential; where a JD allows, prefer the stronger
    pieces (5e-quint / HN front page; 50 Shades of Rust) on merit.
  - **Do NOT claim "2nd most popular post on company blog."** Igor confirms the Jamboree was **not** the
    2nd-most-popular; the ranking is unsubstantiated (no analytics artifact). Cite the articles on their
    own merit, without a popularity ranking.
- loskutoff.com: Negative Space is Misunderstood; **50 Shades of Rust** (r/rust, personal blog — this is
  Igor's actually-popular Rust piece, but it's personal/reddit, not the company blog); D&D Quint posts.
- Montreal meetup talks: Effect.ts (May 2026); Exhaustive Matching with ts-pattern; Property-Based Testing (fast-check).
- Internal talks: Immutability & Side Effects; Property-Based Testing; System Architecture & DDD.

---
## Grilling log (answers folded into sections above)
Session 1 (2026-06-08):
- §6 origin: first code = Perl-bot macros for a niche Korean MMORPG; self-taught; degree later. Origin = low value.
- §7 passion: (1) explore new frontiers, (2) prove he beats decades of industry practice — two separate drivers.
  "Why stayed": code's formalism/strictness fits how he thinks; can do it tirelessly, is good at it.
  Concrete frontier: executable specs → generate lib in any language via AI-agent output + spec-as-oracle (in-flight).
- §7 proofs of functional/immutable bet: Reflector→Effect rewrite "immediately better"; huly-mcp leads 22-repo field; DesignerAdvantage rescue.
- §9 opinions: naming/explicit-over-implicit, primitives aren't self-documenting, domain/ubiquitous-language mandatory,
  errors-as-values (often), types are PART of spec, docs best-effort generated/synced. (4 assistant-invented opinions rejected — see §9 correction log.)
- §5 ratings: Python basic; Go average; Rust average (pre-AI enthusiast: web/WASM, bevy, some Solana).

OPEN / not yet answered:
- §2: proudest = Currents (multi-layer error groups; he fixed root cause first where batch-data
  patches had made it worse). Sharpen: what the errors concretely were + the actual fix.
- §3: ANSWERED — huly-mcp (AI) best; manual = sstackrr (full-stack, 2022) + jikan (FP style, unpolished).
  Open: verify sstackrr test coverage; decide if jikan worth polishing for showcase.
- §7/Q4b: realization status of cross-language spec→lib generation (currently in-flight).
- §9: list not exhaustive — more opinions may exist.
