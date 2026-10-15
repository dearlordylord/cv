# Igor Loskutov - Technical Story & Interview Preparation

This document contains the full technical context behind resume bullets, including personal projects, architectural decisions, and deep technical details for interview preparation.

> **Provenance caveat (read before trusting depth):** much of this is reverse-engineered from repo
> inspection, not Igor's recall. Detail density here proves the *artifact exists* — it does NOT prove
> Igor can defend it deeply in an interview. For defensibility tiering (what to lead with vs.
> architecture-level-only vs. breadth-only), see `PROFILE_KB.md` §5b. Applies hardest to the Python /
> Reflector sections: real work, but Python is self-rated basic with no Python/async talks or blogs.
>
> **Claim authority:** `FACTS.md` in the EI job-search folder
> (`~/Library/Mobile Documents/com~apple~CloudDocs/ei/FACTS.md`) governs what may appear on a CV.
> It records per-project commit shares from a codebase audit and a list of claims that failed it.
> Where this document and FACTS.md disagree, FACTS.md wins.

---

## Table of Contents

1. [Monadical Projects (2022-2026)](#monadical-projects)
   - [Currents: Graph-based Payment Distribution & Visualization](#currents)
   - [Reflector: AI Transcription Platform](#reflector)
   - [Scope3 AI: LLM Environmental Impact Analysis](#scope3-ai)
   - [Transcription-Eval: ML Model Evaluation](#transcription-eval)
2. [Personal Projects](#personal-projects)
   - [Headless-Chat: Discord-Grade Real-Time Chat](#headless-chat)
   - [LLM Agent Orchestration (Ralph)](#llm-agent-orchestration)
3. [Technical Talks & Education](#technical-talks)
   - [Property-Based Testing Tech Talk](#property-based-testing)
4. [Cross-Cutting Themes](#themes)

---

## Monadical Projects (2022-2026)

### <a name="currents"></a>Currents: Graph-based Payment Distribution & Visualization

> **Defensibility (§5b): T1 — defend fully.** React/TS, fp-ts/immutable, and domain ownership (he diagnosed the root-cause multi-layer error-group bug others had worsened with batch data-patches). Snail-trace routing algo + accounting invariants are his to defend. Also the source of the infra/DevOps ownership (Terraform/K8s) — defend ops *decisions*.

**Location**: `/Users/firfi/work/clients/monadical/currents`
**Resume Bullet**: "Graph-based Funds Distribution & Visualization"

#### What It Is

Multi-layered payment processing platform for distributing funds through artist/collective networks with real-time graph visualization.

#### Technical Deep Dive

##### **1. Graphuier - Interactive Graph Visualization**

**Location**: `/apps/graphuier`

**What**: React application visualizing payment routing patterns using ForceGraph2D (D3-force-based).

**Visualization Modes**:
- **Tokens View**: Real-time token graph with force-directed layout
- **Payin Trace Simulation**: Step-by-step payment flow visualization
  - Blue nodes: start point
  - Green: filled with funds
  - Gray: faded (no capacity)
  - Teal: split point (multiple outgoing edges)
  - Red: dead-end
- **Tokens JSON**: Interactive form-based graph input

**Why Visualization Matters**:
- Payment routing through 10-50 nodes is cognitively complex
- Stakeholders (artists, collectives) need to understand fund distribution
- Debugging routing algorithm requires visual trace
- URL: https://graphuier-staging.currents.fm/

##### **2. Payment Routing Algorithm - "Snail Trace"**

**Core Concept**: Track payment "packets" as they flow through beneficiary graph.

**Events**:
```
start → fill → split → fade → jump → end
```

**Key Features**:
- **Packet Allocation**: Equal distribution with remainder handling
- **Greed Mechanism**: Each node has capacity requirement (greed value)
- **Full vs Partial Fills**: Node fills completely or passes remainder forward
- **Split Logic**: When node has multiple outbound edges, packet divides equally
- **Depth-First Traversal**: Through graph structure

**Example Flow**:
```
$100 payin → Artist A (greed: $30) [fills $30, passes $70]
           → Collective B (greed: $20) [fills $20, passes $50]
           → [splits 2 ways]
              → Artist C (greed: $15) [fills $15, passes $10]
              → Artist D (greed: $15) [fills $15, passes $10]
```

##### **3. Graph Concepts & Constraints**

**Beneficiary Graph**:
- Nodes: Artists (blogs), Collectives, Settlement accounts
- Edges: BENEFICIARY (collective → artist), SETTLEMENT (artist → bank)
- Constraint: **Flat graph** (no cycles allowed)

**Why Flat?**:
- Prevents infinite loops in routing
- Enforced at write-time: artist channels have exactly 1 outbound edge
- Collective channels: multiple outbound, but all must terminate at artists

**Data Model**:
```typescript
type ContentGraph = {
  nodes: TokenNode[];  // with metadata
  edges: WebbyEdge[];  // relationships
}

type BeneficiaryGraph = {
  channels: Map<ChannelId, Channel>;
  edges: Map<ChannelId, Set<ChannelId>>;  // adjacency list
}

type RoutingResult = {
  snail: RoutingSnail;  // step-by-step trace
  allocations: Map<NodeId, Amount>;  // final distribution
}
```

##### **4. Payment System Architecture**

**Three Layers**:

1. **Alpha System** (deprecated):
   - Direct Firebase/Stripe writes
   - Neo4j graph storage
   - Manual monthly payouts

2. **Beta System** (production):
   - Real-time payin via REST API
   - Synchronous Stripe entity mirroring to Firebase
   - Event-driven payout processing
   - Dual-source: Firebase (master) + Neo4j (queries)

3. **Books - Double-Entry Accounting**:
   - Per-user ledger tracking all movements
   - Books: payins, refunds, pending, payouts, author_queue, deprivations, provider_conversions
   - Prisma + PostgreSQL
   - Invariant: Assets = Liabilities (validated on each transaction)

**Payin Pipeline**:
```
User clicks subscribe/tip
  → Frontend calls POST /api/tip
  → Backend creates Stripe/PayPal charge
  → Charge saved to Firestore
  → (Later) Monthly batch job routes through graph
  → Calculates user balances
  → Issues payouts via Stripe Connect
  → Logs CSV/JSON snapshots for audit
```

**Stripe Integration Details**:
- Customer created on first payin
- Payment methods mirrored to Firestore (performance optimization)
- Two modes: eventual consistency (queue) + synchronous mirroring
- Cached entities reduce API hits by 80%

##### **5. Technical Stack**

**Monorepo Scale**: 99+ libraries organized by domain

**Key Libraries**:
- `token-routing-*`: Payment routing logic
- `stripe-*`: 6 libs for Stripe integration
- `graph-*`: 5 libs for graph data structures
- `firebase-*`: 5 libs for Firebase utilities
- `books`: Double-entry accounting
- `ticketing-*`: 8 libs for ticketing system

**Functional Programming**:
- `fp-ts`: Either, Option, pipe, TaskEither
- `immutable.js`: ImmutableMap, List, Set for structural sharing
- `io-ts`: Runtime validation at boundaries
- `effect/schema`: Schema validation

**Frontend**:
- React + TypeScript + Vite
- `react-force-graph`: ForceGraph2D component
- `react-query`: Data fetching
- Tailwind CSS

**Backend**:
- Express.js REST API
- Prisma ORM
- PostgreSQL (accounting)
- Firebase Firestore (master data)
- Neo4j (graph queries, legacy)

##### **6. Challenging Problems Solved**

1. **Infinite Loop Prevention**:
   - Problem: Cyclic beneficiary graphs cause routing to hang
   - Solution: Flat graph constraint enforced at write-time
   - Validation: Artist channels can only point to settlements

2. **Money Conservation**:
   - Problem: Splits must not lose/create money
   - Solution: Remainder tracking + validation at each step
   - Test: Total input = Total output + Total remainder

3. **Distribution Fairness**:
   - Problem: How to split $100 among 3 artists?
   - Solution: `allocateEqually()` algorithm with remainder distribution
   - Example: $100 / 3 = $33.33 each + $0.01 remainder → first artist gets $33.34

4. **Performance at Scale**:
   - Problem: Stripe API rate limits with 1000s of customers
   - Solution: Firebase cache + queue-based eventual consistency
   - Result: 80% reduction in API calls

5. **Auditability**:
   - Problem: Need to reconstruct any payout months later
   - Solution: CSV/JSON snapshots logged on each payout run
   - Format: Includes routing trace, final balances, Stripe transfer IDs

##### **7. Key Files**

- `/apps/graphuier/src/routes/payin-trace-simulation.tsx` - Visual simulation UI
- `/libs/token-routing-blog-content-graph-routing/` - Routing algorithm
- `/libs/books/` - Double-entry accounting system
- `/libs/stripe-firebase-mirror/` - Stripe entity caching
- `/apps/graph/src/` - Main backend API

#### Interview Talking Points

- **Architecture**: 99-library monorepo demonstrates separation of concerns at scale
- **FP Patterns**: fp-ts for error handling, immutable.js for performance
- **Visualization**: ForceGraph2D makes complex domain accessible to non-technical stakeholders
- **Financial Correctness**: Double-entry accounting + audit trails = production-grade
- **Performance**: Cache strategy reduced Stripe API calls by 80%

---

### <a name="reflector"></a>Reflector: AI Transcription Platform

> **Defensibility (§5b): T2/T3 — Python self-rated basic, no independent depth signal.** Hatchet durable execution + multi-stage LLM pipeline = **T2**: defend at architecture/decision level (he led the Celery→Hatchet migration — defend the *why*), not deep async internals. PyAV multitrack sync = **T3-shallow**: explain the approach (start_time/adelay), not FFmpeg/filter-graph internals. Process-fork-safety/ContextVar = **T3**: real but NOT deeply defensible — breadth/concept-level only, never a headline (dropped from the DNG resume for this reason).

**Location**: `/Users/firfi/work/clients/monadical/reflector`
**Resume Bullet**: "AI Transcription Platform (Reflector)"

#### What It Is

AI-powered meeting transcription platform with real-time speech-to-text, speaker diarization, multilingual translation, and LLM-powered topic detection.

**Key Features**:
- Real-time transcription (Whisper/Parakeet models)
- Speaker diarization (Pyannote 3.1)
- Live translation (Facebook Seamless-M4T)
- Topic detection & summarization (LLM-powered)
- 100% local processing (no vendor lock-in)

#### Technical Deep Dive

##### **1. Hatchet DAG Workflows - Durable Execution**

**Why Hatchet?**:
- Audio processing has long-running tasks (5-10 minutes per recording)
- Need resilience: network failures, GPU timeouts, process crashes
- Traditional Celery: loses state on worker restart
- Hatchet: durable execution with automatic retry & replay

**Architecture**:

```
DailyMultitrackPipeline (main workflow, 1,341 lines)
  ├─ get_recording (fetch from Daily.co)
  ├─ TrackProcessing × N (parallel, one per audio track)
  │   ├─ extract_metadata (PyAV stream.start_time)
  │   ├─ pad_track (adelay filter for sync)
  │   └─ transcribe_track (send to Modal GPU)
  ├─ merge_transcripts (combine all tracks by timestamp)
  ├─ generate_waveform (visualization)
  ├─ TopicChunkProcessing × C (parallel, ceil(words/300))
  │   └─ detect_topics_in_chunk (LLM call)
  ├─ SubjectProcessing × M (parallel, max 6 subjects)
  │   └─ summarize_subject (LLM call)
  └─ finalize (cleanup, Zulip notification)
```

**Key Features**:

1. **Parent-Child Dependencies**:
   ```python
   @hatchet.step(parents=["get_recording"])
   async def extract_metadata(self, context: Context):
       # Runs after get_recording completes
   ```

2. **Dynamic Parallelization**:
   ```python
   # Fan out to N child workflows (one per track)
   bulk_runs = [
       track_workflow.create_bulk_run_item(
           input={"track_url": track.url, "track_index": i}
       )
       for i, track in enumerate(tracks)
   ]
   results = await track_workflow.aio_run_many(bulk_runs)
   ```

3. **Automatic Retry**:
   - Up to 3 retries per task
   - Exponential backoff
   - Persisted state across retries

4. **Execution Timeouts**:
   - `TIMEOUT_SHORT=60s` (API calls)
   - `TIMEOUT_MEDIUM=120s` (LLM calls)
   - `TIMEOUT_AUDIO=300s` (audio processing)
   - `TIMEOUT_HEAVY=600s` (transcription)
   - Dynamic: Mixdown timeout = `300 + (track_count * 60) + (duration / 10)`

5. **WebSocket Progress Broadcasts**:
   - Each step emits events: `STATUS`, `DURATION`, `WAVEFORM`, `TRANSCRIPT`, `TOPIC`, `FINAL_SUMMARY`
   - Broadcast to: `ts:{transcript_id}` (transcript) + `user:{user_id}` (user)
   - Frontend updates in real-time

##### **2. Daily.co Multitrack Audio Synchronization**

**Problem**: Daily.co provides separate WebM files per participant, but they start at different times.

**Example**:
```
Participant A recording: starts at T=0s
Participant B recording: starts at T=5s (joined 5s late)
Participant C recording: starts at T=12s (joined 12s late)
```

**Solution**: PyAV metadata extraction + silence padding

**Step-by-Step**:

1. **Extract Metadata**:
   ```python
   import av
   container = av.open(track_url)
   stream = container.streams.audio[0]
   start_time = stream.start_time  # in stream time base
   start_seconds = float(start_time * stream.time_base)
   ```

2. **Calculate Padding**:
   - Each track's `start_time` indicates when recording began
   - Global sync point = earliest start_time across all tracks
   - Padding needed = `this_track.start_time - global_start_time`

3. **Apply Silence Padding**:
   ```python
   # PyAV filter graph
   filter_graph = av.filter.Graph()

   # adelay: add delay (silence) at beginning
   buffer = filter_graph.add_buffer(template=stream)
   adelay = filter_graph.add("adelay", f"{delay_ms}ms")
   buffersink = filter_graph.add("buffersink")

   buffer.link_to(adelay)
   adelay.link_to(buffersink)

   # Stream frames through filter
   for frame in container.decode(audio=0):
       buffer.push(frame)
       for filtered_frame in buffersink.pull():
           output_container.mux(filtered_frame)
   ```

4. **Mixdown (Optional)**:
   - Use `amix` filter to combine multiple tracks into single audio file
   - Use `aresample` to normalize sample rates
   - Result: Single MP3 with all participants mixed, globally synchronized

**Why This Works**:
- Daily.co bakes timestamp metadata into WebM container
- PyAV reads container-level metadata (not just audio frames)
- Silence padding aligns all tracks to common timeline
- No manual offset calculation needed

**Edge Cases Handled**:
- Skipped frames (participant muted): PyAV handles via PTS (presentation timestamp)
- Different sample rates: `aresample` filter normalizes
- Codec mismatches: Transcode to common format (Opus → PCM → MP3)

##### **3. Process Fork Awareness**

**Problem**: Hatchet uses process forking for task isolation, but:
- Database connection pools are not fork-safe
- S3 clients (aioboto3) share connection state
- Result: Connection pool exhaustion, hangs

**Solution**: Deferred imports + fresh connections per task

**Pattern**:
```python
# BAD: Import at module level
from reflector.db import transcripts_controller

@hatchet.step()
async def process_transcript(self, context: Context):
    # Uses shared DB connection (fork-unsafe)
    await transcripts_controller.update(...)

# GOOD: Import inside task + fresh connection
@hatchet.step()
async def process_transcript(self, context: Context):
    from reflector.db import transcripts_controller

    async with fresh_db_connection():
        # New connection per task (fork-safe)
        await transcripts_controller.update(...)
```

**Context Manager**:
```python
from contextvars import ContextVar

_database_context: ContextVar[Database] = ContextVar("database")

@asynccontextmanager
async def fresh_db_connection():
    db = Database(DATABASE_URL)
    await db.connect()
    token = _database_context.set(db)
    try:
        yield db
    finally:
        await db.disconnect()
        _database_context.reset(token)
```

**Why Context Variables?**:
- Each forked process gets isolated context
- Parent process context not shared with children
- Thread-safe + async-safe

##### **4. Documentation Work**

**Pipeline Documentation** (`/server/docs/daily_pipeline.md`, 422 lines):
- Complete sequence diagrams showing all API calls
- External dependencies: Daily.co API, Modal (GPU), S3, Zulip
- Error handling flowcharts
- Timeout configurations

**Plan Documents**:
- `/PLAN.md` - Hatchet Whereby support (converting Celery → Hatchet)
- `/PLAN_PARALLEL.md` - Parallelization strategy for topic detection
- `/PLAN_WAVEFORM.md` - Waveform visualization implementation

**Inline Comments**:
- `daily_multitrack_pipeline.py` - 200+ lines explaining audio padding timing
- `track_processing.py` - PyAV stream handling and fork-awareness patterns
- All workflows have comprehensive docstrings

##### **5. Tech Stack**

**Backend**:
- FastAPI (REST API)
- Celery (legacy, being migrated to Hatchet)
- Hatchet SDK (DAG workflows)
- PyAV (audio processing)
- SQLAlchemy + Alembic (database)
- aioboto3 (S3 uploads)
- structlog (structured logging)

**ML/AI**:
- Modal (GPU inference platform)
- Whisper (transcription)
- Pyannote 3.1 (diarization)
- OpenAI/Anthropic (topic detection, summarization)

**Frontend**:
- Next.js 14
- Chakra UI
- WebSocket (real-time updates)
- NextAuth.js (authentication)

##### **6. Challenging Problems Solved**

1. **Audio Timing Synchronization**:
   - Problem: 4-8 concurrent WebM tracks with different start times
   - Solution: PyAV metadata extraction + adelay filter padding
   - Result: Transcription timestamps globally synchronized

2. **DAG with Dynamic Parallelism**:
   - Problem: Hatchet requires static DAG, but track count varies
   - Solution: Child workflows spawned dynamically via `aio_run_many()`
   - Result: Scales to any number of tracks

3. **LLM Rate Limiting**:
   - Problem: Topic detection spawns 17-32 LLM calls (ceil(words/300) chunks)
   - Solution: Parallel subject processing + rate limit enforcement (10/sec)
   - Result: 5x speedup vs sequential processing

4. **Process Fork Safety**:
   - Problem: Shared DB connections cause pool exhaustion in Hatchet
   - Solution: Fresh connections per task + ContextVar isolation
   - Result: Reliable execution without connection leaks

5. **Long-Running Task Observability**:
   - Problem: User can't see progress during 5-10 minute processing
   - Solution: WebSocket broadcasts at each step + event log in database
   - Result: Real-time UI updates showing current stage

##### **7. Key Files**

- `/server/reflector/hatchet/workflows/daily_multitrack_pipeline.py` (1,341 lines) - Main DAG
- `/server/reflector/hatchet/workflows/track_processing.py` (229 lines) - Per-track processing
- `/server/reflector/hatchet/workflows/topic_chunk_processing.py` (97 lines) - Topic detection
- `/server/reflector/hatchet/workflows/subject_processing.py` (107 lines) - Subject summarization
- `/server/docs/daily_pipeline.md` - Complete pipeline documentation

#### Interview Talking Points

- **Python Proficiency**: Complex backend with async patterns, context managers, structured logging
- **Audio Processing**: PyAV filter graphs, codec handling, multitrack synchronization
- **Distributed Systems**: Durable execution, automatic retry, process fork isolation
- **Observability**: WebSocket broadcasts, event logging, real-time progress tracking
- **Migration**: Led Celery → Hatchet migration for improved reliability

---

### <a name="scope3-ai"></a>Scope3 AI: LLM Environmental Impact Analysis

> **Defensibility (§5b): T2 — Go self-rated average.** Defend the *design decisions* — "flow-as-data" StaticGraphNode, code-is-the-docs auto-Mermaid, type-safe SQL choice — at architecture level. Not deep Go runtime/ecosystem internals. LLM-integration angle is genuinely on-domain for GenAI roles.

**Location**: `/Users/firfi/work/clients/monadical/aiapi`
**Resume Bullet**: "LLM Environmental Impact Analysis (Scope3)"

#### What It Is

Environmental impact measurement platform for AI/ML workloads. Tracks carbon emissions (CO2e), energy consumption (Wh), and water usage (H2O) for LLM inference calls.

#### Technical Deep Dive

##### **1. "Flow as Data" Pattern - StaticGraphNode**

**Problem**:
- Impact calculation requires complex cascade of fallback inferences
- 25+ decision nodes: "If datacenter PUE unavailable, try region PUE, try country PUE, use default"
- Need to ensure code matches documentation
- Manual diagrams go stale when logic changes

**Solution**: Represent computation flow as explicit data structures

**Core Data Structure**:
```go
type StaticGraphNode struct {
    Id          string
    Description string
    links       []StaticGraphEdge
    linksMap    map[string]StaticGraphEdge
}

type StaticGraphEdge struct {
    Node        StaticGraphNode
    Description string
}
```

**Example**:
```go
var InferPueStaticNode = NewStaticGraphNode(
    "InferPue",
    "PUE Inference",
    []StaticGraphEdge{
        {
            Node:        GetDatacenterPueStaticNode,
            Description: "Datacenter PUE is the highest precision we can get",
        },
        {
            Node:        GetComputeRegionPueStaticNode,
            Description: "Or we try to get its Regional approximation",
        },
        {
            Node:        GetCountryPueStaticNode,
            Description: "Or we try to get its Country approximation",
        },
        {
            Node:        GetDefaultPueStaticNode,
            Description: "Or we use the default PUE",
        },
    },
)
```

**Benefits**:
- Code IS the documentation (single source of truth)
- Type-safe at compile-time
- Verifiable at runtime
- Can auto-generate visualizations

##### **2. Auto-Generated Mermaid Diagrams**

**Why Mermaid?**:
- 25+ node inference cascade too complex for mental model
- Need visual documentation that stays in sync with code
- Solution: Generate diagrams directly from StaticGraphNode structures

**Implementation**:

1. **Generator Tool** (`/tools/impact_params_docs_gen/main.go`):
   ```go
   //go:generate go run ../../tools/impact_params_docs_gen/main.go

   func (node StaticGraphNode) ToMermaid() string {
       diagram := flowchart.NewFlowchart()
       diagram.SetDirection(flowchart.FlowchartDirectionBottomUp)

       // Recursively walk StaticGraphNode tree
       for _, edge := range node.links {
           diagram.AddEdge(node.Id, edge.Node.Id, edge.Description)
           // Recurse into child nodes
           edge.Node.ToMermaid()
       }

       return diagram.String()
   }
   ```

2. **Generated Files**:
   - `/docs/graphs/pue_inference_graph.md` - PUE (Power Usage Effectiveness) inference
   - `/docs/graphs/gridmix_inference_graph.md` - Grid Mix (energy source composition)
   - `/docs/graphs/impact_params_graph.md` - Full combined root graph

3. **Example Output**:
   ````markdown
   ```mermaid
   flowchart BT
       root --> InferPue
       InferPue --> GetDatacenterPue
       InferPue --> GetComputeRegionPue
       InferPue --> GetCountryPue
       InferPue --> GetDefaultPue
       GetDatacenterPue --> InferDatacenter
       GetComputeRegionPue --> InferComputeRegion
       GetCountryPue --> InferCountry
   ```
   ````

**Why Bottom-Up Direction?**:
- Shows dependency flow from root inference down to data sources
- Makes it clear what data needs to flow UP to compute impact
- Matches mental model: "To calculate PUE, I need datacenter data"

##### **3. Code-Documentation Sync Guarantee**

**Runtime Verification**:
```go
func AssertImpactParamsGraphLinks[T any](
    node StaticGraphNode,
    genericStruct T,
    keys interface{}) (T, error) {

    // 1. Get all struct fields via reflection
    structFields := getStructFieldNames(genericStruct)

    // 2. Get all edge IDs from StaticGraphNode
    edgeIds := getEdgeIds(node)

    // 3. Assert: every struct field has corresponding edge
    for _, field := range structFields {
        if !contains(edgeIds, field) {
            return nil, fmt.Errorf("struct field %s missing in graph", field)
        }
    }

    // 4. Assert: every edge has corresponding function
    for _, edgeId := range edgeIds {
        if !hasFunction(genericStruct, edgeId) {
            return nil, fmt.Errorf("graph edge %s missing function", edgeId)
        }
    }

    return genericStruct, nil
}
```

**Catches**:
- Orphaned struct fields (no corresponding graph edge)
- Orphaned graph edges (no corresponding function)
- Ensures code and graph stay in sync

##### **4. Complex Parameter Hierarchy**

**Inference Cascade**:

```
User Request
  ├─ User Location (IP geolocation)
  ├─ Datacenter (provider-specific)
  ├─ Compute Region (us-east-1, eu-west-1, etc.)
  └─ Country (US, DE, etc.)

Impact Calculation
  ├─ PUE (Power Usage Effectiveness)
  │   ├─ Try: Datacenter-specific PUE (highest precision)
  │   ├─ Try: Compute Region average PUE
  │   ├─ Try: Country average PUE
  │   └─ Fallback: Default PUE (1.2)
  │
  ├─ Grid Mix (Energy Source Composition)
  │   ├─ Try: Datacenter-specific grid mix
  │   ├─ Try: Compute Region grid mix
  │   ├─ Try: Country grid mix
  │   └─ Fallback: Global average grid mix
  │
  └─ Model Settings
      ├─ Energy per token (Wh/token)
      ├─ GPU type (A100, V100, etc.)
      └─ Batch size
```

**Example Flow**:
```
Request: "Calculate impact for Anthropic Claude call in us-east-1"
  → Infer Datacenter: Anthropic (AWS us-east-1)
  → Infer PUE: AWS us-east-1 datacenter PUE = 1.15 ✓
  → Infer Grid Mix: Virginia (US) grid mix = 30% coal, 40% gas, 30% renewable ✓
  → Model Settings: Claude-3 Opus = 5.2 Wh/1000 tokens ✓
  → Calculate: 1000 tokens × 5.2 Wh × 1.15 PUE × (0.3×coal_co2 + 0.4×gas_co2 + 0.3×renewable_co2)
  → Result: 8.2 gCO2e, 6.0 Wh, 15 mLH2O
```

##### **5. Type-Safe SQL with go-jet**

**Problem**: String-based SQL is error-prone and not type-checked.

**Solution**: go-jet generates type-safe query builders from database schema.

**Example**:
```go
// Generated types from schema
import (
    "github.com/go-jet/jet/v2/postgres"
    "github.com/scope3ai/aiapi/gen/scope3/public/model"
    "github.com/scope3ai/aiapi/gen/scope3/public/table"
)

// Type-safe query (compile-time checked)
stmt := postgres.
    SELECT(
        table.Datacenter.AllColumns,
        table.Provider.Name.AS("provider_name"),
    ).
    FROM(
        table.Datacenter.
            INNER_JOIN(table.Provider, table.Datacenter.ProviderID.EQ(table.Provider.ID)),
    ).
    WHERE(
        table.Datacenter.Region.EQ(postgres.String("us-east-1")),
    )

var results []model.Datacenter
err := stmt.Query(db, &results)
```

**Benefits**:
- SQL injection impossible (no string concatenation)
- Column typos caught at compile-time
- Join relationships type-checked
- Auto-completion in IDE

##### **6. Tech Stack**

**Backend**:
- Go (Gin web framework)
- PostgreSQL (master data: providers, datacenters, regions)
- ClickHouse (inference logs, high-write-volume time-series)
- go-jet (type-safe SQL)
- fp-go (Either/Option monads for error handling)

**Python SDK** (`scope3ai-py`):
- Supports: OpenAI, Anthropic, Cohere, LiteLLM, HuggingFace, MistralAI, Google GenAI
- Monkey-patches LLM client libraries
- Extracts metadata: model, tokens, provider, datacenter
- Sends to aiapi for impact calculation

**Observability**:
- Structured logging (JSON)
- PostHog analytics integration
- ClickHouse inference log (every call tracked)
- Cloud Run logs → Google Cloud Logging

##### **7. Challenging Problems Solved**

1. **Code-Documentation Drift**:
   - Problem: Manual diagrams become stale
   - Solution: Auto-generate from code via `go generate`
   - Result: Diagrams always match implementation

2. **Correctness at Scale**:
   - Problem: 25+ nodes, easy to miss dependencies
   - Solution: Runtime assertion validates graph completeness
   - Result: Catches orphaned logic immediately

3. **Multiple Inference Strategies**:
   - Problem: 4-5 fallback strategies for PUE, each with different data sources
   - Solution: Channel-based approach emits results as available
   - Result: First available result wins, no blocking

4. **Database Schema Complexity**:
   - Problem: 50+ tables, prone to SQL errors
   - Solution: go-jet code generation from schema
   - Result: Type-safe queries, no runtime SQL errors

##### **8. Key Files**

- `/common/impact_params/graph.go` - StaticGraphNode data structure
- `/common/impact_params/pue_inference.go` - PUE inference logic
- `/common/impact_params/gridmix_inference.go` - Grid mix inference
- `/tools/impact_params_docs_gen/main.go` - Mermaid generator
- `/docs/graphs/*.md` - Auto-generated diagrams
- `/CODE_GUIDE.md` - Design patterns & exhaustiveness checks

#### Interview Talking Points

- **Self-Documenting Code**: Flow-as-data pattern ensures code and docs stay in sync
- **Go Proficiency**: Type-safe SQL, monadic error handling, code generation
- **System Design**: 25+ node inference cascade with fallback strategies
- **Observability**: Auto-generated diagrams make complex logic transparent
- **Production Quality**: ClickHouse for high-volume writes, PostgreSQL for master data

---

### <a name="transcription-eval"></a>Transcription-Eval: ML Model Evaluation

> **Defensibility (§5b): T1/T2 — mostly TS, not the Python it sounds like.** Stack is TypeScript + Node + SQLite/Kysely (Python only a `jiwer` subprocess). Defend the TS architecture, parallel test execution, and cache strategy at **T1**. WER/DER metrics are domain-specific (**T2**) — he implemented DER custom, so explain the concepts, but don't pose as an ASR/ML researcher.

**Location**: `/Users/firfi/work/clients/monadical/transcription-eval`
**Resume Bullet**: "ML Model Evaluation System"

#### What It Is

Automated evaluation framework for comparing transcription quality across different audio input types (multitrack vs single-track).

**Business Goal**: Quantitatively prove Daily.co's raw-tracks (multitrack) produces better transcription than Whereby's mixed audio (single-track).

#### Technical Deep Dive

##### **1. WER/DER Metrics**

**Word Error Rate (WER)**:
- Industry-standard accuracy metric for transcription
- Formula: `WER = (Substitutions + Deletions + Insertions) / Total Words`
- Example: "the cat sat" → "the dog sat" = 1 substitution / 3 words = 33% WER

**Diarization Error Rate (DER)**:
- Measures speaker identification accuracy
- Formula: `DER = (Missed Speech + False Alarm + Speaker Error) / Total Speech Time`
- Example: 60s meeting, 5s missed speaker, 3s wrong speaker = 8/60 = 13% DER

**Automated Calculation**:
- Ground truth stored in database (VTT format with Speaker0/Speaker1 labels)
- Reflector outputs JSONL with transcription + diarization
- Compare: Reflector output vs ground truth
- Result: WER/DER scores for each test case

##### **2. Multitrack Comparison**

**Single-track Mode**:
```bash
python -m reflector.tools.process <audio.mp4> --pipeline file
```
- Input: Single mixed audio file (all participants combined)
- Output: JSONL with transcription + ML-based diarization (Pyannote)

**Multitrack Mode**:
```bash
python -m reflector.tools.process <track1>,<track2>,<track3> --multitrack --pipeline file
```
- Input: Comma-separated paths (one per participant)
- Output: Same JSONL format
- Key difference: Each track is separate, no diarization needed (track index = speaker)

**Why Multitrack Wins**:
- Diarization errors eliminated (track index is ground truth)
- No speaker overlap interference (separate audio streams)
- Cleaner audio per participant (no crosstalk)

##### **3. Scale - Parallel Test Execution**

**Problem**: Evaluating 100+ test cases sequentially takes hours.

**Solution**: Parallel processing

**Implementation**:
```typescript
// Pseudocode
const meetings = await db.select().from(meetings).all();

// Process in batches of 10
const batches = chunk(meetings, 10);

for (const batch of batches) {
  await Promise.all(
    batch.map(async (meeting) => {
      const result = await runEvaluation(meeting);
      await db.insert(results).values(result);
    })
  );
}
```

**Result**: 100 test cases in ~15 minutes (vs 2+ hours sequential)

##### **4. Cache Strategy**

**Problem**: Re-running same evaluation wastes time/money (GPU costs).

**Solution**: Hash-based caching

**Cache Key**:
```typescript
function generateCacheKey(
  meetingId: string,
  cliArgs: Record<string, string>
): string {
  const hash = sha256(`${meetingId}:${JSON.stringify(cliArgs)}`);
  const prefix = meetingId.slice(0, 8);
  return `${hash}-${prefix}`;
}
```

**Why Meeting ID Only?**:
- Meeting ID uniquely identifies input audio + ground truth
- CLI args (--multitrack, --pipeline, model settings) affect processing
- Audio path is redundant (determined by meeting ID)

##### **5. Tech Stack**

**Backend**:
- TypeScript + Node.js
- SQLite + Kysely ORM
- Reflector CLI (Python subprocess)

**Evaluation**:
- Jiwer library (Python) for WER calculation
- Custom DER implementation

**Storage**:
- S3 (audio files from Daily.co recordings)
- SQLite (meetings, ground truth, results)

##### **6. Key Files**

- `/Users/firfi/work/clients/monadical/transcription-eval/PRD.md` - Product requirements
- `/Users/firfi/work/clients/monadical/transcription-eval/.llm-docs/product.md` - Business value

#### Interview Talking Points

- **ML Evaluation**: Industry-standard metrics (WER/DER) for objective quality measurement
- **Automation**: Parallel processing reduces evaluation time by 88%
- **Business Impact**: Data-driven vendor selection (Daily.co vs Whereby)
- **Integration**: TypeScript calling Python CLI, bridging ecosystems

---

## Personal Projects

### <a name="headless-chat"></a>Headless-Chat: Discord-Grade Real-Time Chat

> **Defensibility (§5b): T1 (style) — but PERSONAL project, not shipped at work.** Strong showcase of his signature TS/FP (fp-ts/io-ts/ts-pattern), cursor reconciliation, distributed-ordering. Fully defensible technically. Caveat: frame as a personal library, NOT a production work credential.

**Location**: `/Users/firfi/work/typescript/headless-chat`
**Status**: Personal project (v0.1.0), not in production

#### What It Is

TypeScript-first, framework-agnostic real-time chat library designed as a Discord/Slack-grade alternative to existing solutions.

**Positioning** (from COMPARISON.md):
- Superior to @yext/chat-headless for high-scale real-time applications
- WebSocket-first (vs HTTP/SSE)
- Multi-room support (vs single conversation)
- Sophisticated message reconciliation

#### Technical Deep Dive

##### **1. Cursor-Based Message Reconciliation**

**Problem**: Chat apps must handle:
- Out-of-order message delivery
- Network disconnections
- Reconnection without full resync

**Solution**: Dirty/clean message separation

**Data Model**:
```typescript
type RoomState = {
  clearMessages: ImmutableList<Message>;    // Confirmed from server
  dirtyMessages: ImmutableList<Message>;    // Local, unconfirmed
  cursors: {
    state: 'idle' | 'syncing';
    value: [CursorId, CursorId];            // Range markers
  };
  users: ImmutableSet<UserId>;
};
```

**Reconciliation Flow**:
```
1. User sends message
   → Add to dirtyMessages (local, unconfirmed)
   → Send via WebSocket

2. Server confirms message
   → Move from dirtyMessages to clearMessages
   → Update cursor to message ID

3. User disconnects

4. User reconnects
   → Request range: [LAST_CURSOR, NOW]
   → Server sends missing messages
   → Merge into clearMessages
   → Filter dirtyMessages (remove duplicates)
```

**Key Insight**: Only sync the gap, not everything.

##### **2. Composite Message IDs - Distributed Ordering**

**Problem**:
- Multiple clients send messages concurrently
- Network latency varies
- Messages arrive out-of-order
- Need deterministic ordering

**Solution**: Composite IDs with timestamp + counter

**Format**: `{timestamp}-{counter}`

**Example**:
```
User A sends: 1000-0
User B sends: 1000-1  (same millisecond)
User C sends: 1001-0  (next millisecond)

Server receives: 1000-1, 1001-0, 1000-0 (out of order)
Sorted: 1000-0, 1000-1, 1001-0 (correct order)
```

**Ordering Function**:
```typescript
const messageIdCompositeOrd: Ord<MessageIdComposite> = Ord.tuple(
  Number.Ord,  // timestamp
  Number.Ord   // counter
);
```

**Benefits**:
- Guarantees ordering even with out-of-order delivery
- Prevents collisions (counter breaks ties)
- No coordination needed (client-generated)
- Works across distributed clients

##### **3. Multi-Room Architecture**

**State Structure**:
```typescript
type ChatState = {
  rooms: ImmutableMap<ChatRoomId, RoomState>;  // Multi-room
  online: ImmutableSet<UserId>;                // Global presence
  focusedRoom: Option<ChatRoomId>;             // UI focus
};
```

**Why ImmutableMap?**:
- O(log N) lookups (vs O(N) for array)
- Structural sharing (cheap copies)
- Memory efficient for 1000+ rooms

**Comparison to Competitors**:
- @yext/chat-headless: Single conversation only
- headless-chat: Unlimited concurrent rooms
- Use case: Slack-style workspace with 100+ channels

##### **4. Gap Detection & Recovery**

**Reconciliation Algorithm** (`core.ts` lines 101-184):

```typescript
function reconcile(
  state: RoomState,
  serverMessages: Message[]
): RoomState {
  // 1. Find last clear message
  const lastClearId = state.clearMessages.last()?.id;

  // 2. Detect gap
  const gap = serverMessages.find(msg =>
    msg.id > lastClearId &&
    !state.clearMessages.some(m => m.id === msg.id)
  );

  if (gap) {
    // 3. Request missing range
    requestRange(lastClearId, gap.id);
  }

  // 4. Merge server messages into clearMessages
  const newClearMessages = state.clearMessages
    .concat(serverMessages)
    .sort(messageIdCompositeOrd);

  // 5. Filter dirty messages (remove confirmed)
  const newDirtyMessages = state.dirtyMessages.filter(dirty =>
    !serverMessages.some(server => server.id === dirty.id)
  );

  return {
    ...state,
    clearMessages: newClearMessages,
    dirtyMessages: newDirtyMessages,
  };
}
```

**Edge Cases**:
- Islands of messages with gaps: Warns, doesn't crash
- Duplicate messages: Deduped by ID
- Out-of-order arrival: Sorted by composite ID

##### **5. WebSocket-First Protocol**

**Connection Health**:
```typescript
const config = {
  heartbeat: {
    interval: 3000,  // 3 seconds
    message: JSON.stringify({ type: 'ping' }),
    returnMessage: JSON.stringify({ type: 'pong' }),
  },
  reconnect: true,
  reconnectDelay: 3000,
};
```

**Message Types**:

*Client → Server*:
```typescript
{ type: 'join', roomId }
{ type: 'leave', roomId }
{ type: 'message', roomId, text, messageId, previousMessageId }
{ type: 'ping' }
```

*Server → Client*:
```typescript
{ type: 'message', roomId, from, messageId, text, previousMessageId }
{ type: 'membership', roomId, who, state: 'joined'|'left' }
{ type: 'pong' }
```

**Why WebSocket vs HTTP?**:
- Bidirectional (server can push)
- Lower latency (no HTTP overhead)
- Persistent connection (no reconnect overhead)
- Real-time (instant delivery)

##### **6. Architecture Patterns**

**Functional Programming**:
```typescript
// Pure reducer pattern
export const reduce = (action: ChatAction) => (state: ChatState): ChatState =>
  match(action)
    .with({ type: 'roomActivity' }, ({ activity, roomId }) => {
      // Update room with new activity (pure function)
      return updateRoom(state, roomId, addActivity(activity));
    })
    .with({ type: 'reconcile' }, ({ roomId, messages }) => {
      // Reconcile messages (pure function)
      return updateRoom(state, roomId, reconcile(messages));
    })
    .exhaustive();
```

**Type-Safe Runtime Validation**:
```typescript
// io-ts codec (type + validation)
export const roomActivityMessageCodec = t.intersection([
  t.type({ id: messageIdCodec }),
  roomActivityMessageBaseCodec,
]);

// Decode + validate WebSocket message
pipe(
  rawMessage,
  chatWsOutgoingMessageCodec.decode,
  E.fold(
    (error) => console.error('Invalid message:', error),
    (validated) => dispatch(validated)
  )
);
```

**Pattern Matching**:
```typescript
// ts-pattern exhaustive matching
match(action)
  .with({ type: 'roomActivity' }, handleRoomActivity)
  .with({ type: 'reconcile' }, handleReconcile)
  .with({ type: 'online' }, handleOnline)
  .exhaustive();  // Compile error if cases missing
```

##### **7. Tech Stack**

**Core Libraries**:
- `fp-ts`: Functional programming (Either, Option, pipe)
- `immutable`: Persistent data structures (Map, List, Set)
- `io-ts`: Runtime type validation (codecs)
- `ts-pattern`: Pattern matching (exhaustive)
- `React`: UI framework (hooks)

**Monorepo**:
- `@headless-chat/types`: Shared types + codecs
- `@headless-chat/core`: Pure business logic
- `@headless-chat/react`: React hooks + WebSocket integration

##### **8. Comparison to @yext/chat-headless**

| Feature | headless-chat | @yext/chat-headless |
|---------|---------------|---------------------|
| Protocol | WebSocket | HTTP/SSE |
| Rooms | Multi-room | Single conversation |
| Reconciliation | Cursor-based | Full resync |
| Message IDs | Composite (timestamp-counter) | Simple UUID |
| Gap Recovery | Range query | Full reload |
| Type Safety | io-ts runtime validation | TypeScript only |
| Bundle Size | ~400KB | ~200KB |
| State Management | Immutable.js + reducers | Redux |

**headless-chat wins**: High-scale, multi-room, sophisticated reconciliation
**@yext/chat-headless wins**: Smaller bundle, simpler patterns, production features (analytics, rate limiting)

##### **9. Challenging Problems Solved**

1. **Message Ordering in Distributed Systems**:
   - Problem: Concurrent clients, varying latency
   - Solution: Composite IDs (timestamp-counter)
   - Result: Deterministic ordering without coordination

2. **Reconnection Data Loss**:
   - Problem: Full resync wastes bandwidth
   - Solution: Cursor-based range queries
   - Result: Only fetch missing messages

3. **Multiple Concurrent Rooms**:
   - Problem: Redux-style state gets unwieldy
   - Solution: ImmutableMap + structural sharing
   - Result: O(log N) lookups, memory efficient

4. **Type Safety at Boundaries**:
   - Problem: WebSocket messages are untyped JSON
   - Solution: io-ts decoders validate at runtime
   - Result: No invalid data enters reducer

##### **10. Key Files**

- `/packages/core/src/core.ts` - Reducer + reconciliation logic
- `/packages/types/src/codecs.ts` - io-ts codecs
- `/packages/react/src/useChat.ts` - React hooks
- `/COMPARISON.md` - Detailed comparison with competitors

#### Interview Talking Points

- **Distributed Systems**: Cursor-based reconciliation, gap recovery, composite IDs
- **Functional Programming**: Pure reducers, immutable data, type-safe decoders
- **Real-Time Protocols**: WebSocket-first, heartbeat, reconnection logic
- **Library Design**: Framework-agnostic core, React/Vue adapters
- **Type Safety**: io-ts runtime validation catches errors at boundaries

---

### <a name="llm-agent-orchestration"></a>LLM Agent Orchestration (Ralph)

> **Defensibility (§5b): T2 — personal, not productionized; directly DNG-relevant.** Defend the agent-loop *concept* (generate→score→improve, early 2023–24 agent orchestration) — this is the closest real evidence to DNG's "LLM-based agents / ReACT" ask. Honest framing: it's adjacent prior art, not a shipped production agent; don't claim scale.

**Status**: Personal/experimental project (pre-Cursor/Devin era, 2023-2024)
**Resume Bullet**: "LLM Code Generation System"

#### What It Was

Experimental orchestration system wrapping AI coding agents before tools like Cursor, Devin, or Aider became mainstream. Built when the space was still nascent and these tools didn't exist yet.

**Historical Context**: This was Ralph before Ralph was cool. Built during the early wave of AI coding experiments when wrapping Claude/GPT-4 for code generation was still experimental territory.

#### Technical Deep Dive

##### **System Architecture**

**Core Components**:
1. **Agent Wrapper**: Abstracted interface over multiple AI providers (Claude API, OpenAI API)
2. **Quality Scoring**: Automated evaluation of generated code
3. **Iterative Improvement Loop**: Multi-pass refinement based on scores
4. **Test Generation**: Attempted automated test creation for generated code
5. **Code Review Automation**: Experimental static analysis + LLM review combination

##### **What Made It Interesting (for the time)**

**Problem Solved**: In 2023, no good tools existed for:
- Iteratively refining AI-generated code
- Scoring code quality automatically
- Running multiple improvement passes
- Integrating AI coding into existing workflows

**Approach**:
```typescript
// Simplified architecture
type AgentLoop = {
  generate: (prompt: string) => Promise<Code>;
  score: (code: Code) => Promise<Score>;
  improve: (code: Code, score: Score) => Promise<Code>;
  iterate: (maxIterations: number) => Promise<Code>;
};
```

**Experiments Run**:
1. **Multi-iteration refinement**: Generate → Score → Critique → Regenerate (up to 3-5 iterations)
2. **Automated code review**: LLM identifies issues, suggests fixes, applies fixes
3. **Test generation**: Generate tests based on function signatures and docstrings
4. **Style consistency**: Enforce codebase patterns through few-shot examples

##### **Why It's Notable**

**Timing**: Built before Cursor (late 2023), Devin (early 2024), or other agent-based coding tools were available. This was exploratory work when the space was still wide open.

**Honest Assessment**: Experimental system that showed promise but faced fundamental limitations:
- LLMs in 2023 weren't reliable enough for fully autonomous coding
- Quality scoring was hard to automate without human-in-the-loop
- Cost of multiple iterations was high (pre-caching, pre-batch API)
- Generated code often needed significant manual cleanup

**What Was Learned**:
- Iterative refinement helps but hits diminishing returns after 2-3 passes
- Quality scoring needs domain-specific metrics, generic scores aren't enough
- Human-in-the-loop beats fully autonomous for non-trivial tasks
- Context window limits were a major constraint (pre-100K context models)

##### **Key Insights (Applied to Resume)**

**Resume Framing**: "Built orchestration system wrapping AI coding agents before Cursor/Devin era"

**Why This Works**:
- Establishes you were early in the space
- Shows experimental/R&D mindset
- Honest about it being exploratory ("experiments")
- Provides historical context without claiming it was production-ready

#### Technical Stack

**Core**:
- TypeScript/Node.js
- Claude API, OpenAI API
- Custom prompt engineering
- Static analysis tools (ESLint, TypeScript compiler API)

**Evaluation**:
- Custom scoring metrics (complexity, test coverage, type safety)
- AST analysis for code quality
- LLM-based review prompts

#### Why Not in Production

**Honest reasons**:
1. **Cost**: Multiple LLM calls per iteration made it expensive
2. **Reliability**: Code quality was inconsistent, needed manual review
3. **Timing**: Tools like Cursor/Devin solved this better later
4. **Context limits**: Pre-long-context models made it hard to include full codebase

**But Still Valuable**:
- Showed initiative in emerging space
- Learned LLM limitations hands-on
- Built prompt engineering skills
- Understood agent orchestration patterns

#### Interview Talking Points

- **Early adopter**: Built AI coding tools before they were mainstream
- **Experimental mindset**: Willing to explore unproven territory
- **Practical learning**: Discovered limitations through hands-on work
- **Honest assessment**: Can articulate why experiments didn't go to production
- **Timing awareness**: Knew when better solutions (Cursor) made custom tools obsolete

---

## FiddleQuest LLC (2014–2023) — Distributed Event-Driven Learning Platform

> **Defensibility (§5b): T1 — defend fully.** Igor's own architecture, sole/lead author across all three
> apps: **sole author** of every main-app event-mesh file (`publisher.ts`, `badges/server/watch.ts`,
> `badges/server/storage.ts`, `aws/server.ts`), and **top author of the game repo** (227 commits; next
> author 123). TypeScript/Node + native iOS. The "more data flow than it looks" story is real and his to
> defend end-to-end.

**The deceptive simplicity:** reads like "a violin-teaching web app." It's actually a **distributed,
event-driven system spanning three separately-deployed apps**, glued by an AWS SNS / SQS / DynamoDB / S3
mesh:
1. **Main web app** — Meteor (real-time DDP), TypeScript; Ansible-provisioned VPS on DigitalOcean (managed Mongo).
2. **Game** (`fq-game-ink-phaser`) — separate Phaser + Ink (inklecate) narrative-game service.
3. **iOS teacher app** (`ios-teacher-app/FiddleQuest`) — native Swift/Obj-C, Meteor DDP over WebSocket (PocketSocket).

**Data flow (the part that "seems like less than it is"):**
- **Points/events**: student action fires user-schema hooks → main app **publishes to SNS**
  (`sns.publish({ TopicArn })`, `imports/game/points/queue/server/publisher.ts`).
- **SNS → SQS fan-out**: into a points queue; the **game polls SQS** (`pollPoints`, throttled,
  `ReceiveMessage` max 10, **delete-after-process** = idempotent handling) — `fq-game-ink-phaser/src/features/queue`.
- **Game awards badges → DynamoDB**: `updateItem` into a general table, keyed per-`userId` **plus an
  `'all'` meta key** for global badges (`features/badges/server`).
- **Round-trip back to the web app via SNS webhook**: badge changes notify an **SNS HTTP subscription**
  the main app exposes at `/hooks/badges/sns` (`imports/badges/server/watch.ts`), handling per-user and
  `'all'` events.
- **Web badge store**: reads DynamoDB (`getItem` per user, paginated `scan` with `LastEvaluatedKey`) and
  **bridges DynamoDB → Meteor reactivity via an RxJS observable** (`$dynamoUserBadgesSubject` →
  `Meteor.bindEnvironment`); explicitly handles the DynamoDB string-set "empty set gets deleted" edge case.
- **Media**: iOS gets an **S3 presigned upload permission** from a Meteor method
  (`getStudentVideoS3UploadPermission`) → uploads practice video to **S3** → registers via
  `putS3StudentVideo`. Web side also uses S3/CloudFront + ElasticTranscoder for media.

**Why it's a strong talking point:** three heterogeneous clients (Meteor web, Phaser game, native iOS)
decoupled by an async event mesh — SNS fan-out, idempotent SQS consumption, DynamoDB as shared badge
store, RxJS↔Meteor reactive bridges, and deliberate AWS-edge-case handling. Cross-app eventual
consistency, not the CRUD app the domain implies.

**Honest scope:** small-scale niche music-education product, single-region; the sophistication is in the
*architecture / data-flow design*, not in traffic scale. Cognito appears in deps but is **not** used in
code — don't claim it.

## Technical Talks & Education

### <a name="property-based-testing"></a>Property-Based Testing Tech Talk

> **Defensibility (§5b): T1 — defend fully.** This *is* an independent depth signal (public talk + blog post). The gold standard: where a talk/blog exists, depth is corroborated, not recall-dependent.

**Location**: `/Users/firfi/work/typescript/property-tests-tech-talk`
**Date**: February 2024

#### What It Is

Educational demonstration of property-based testing using TypeScript and fast-check library.

#### Key Concepts

**Properties Demonstrated**:
1. **Involution**: `reverse(reverse(a)) = a` - applying twice returns original
2. **Length Preservation**: `reverse(a).length = a.length`
3. **Reversing Property**: Custom reverse matches native implementation
4. **Element Position**: Each element at index i correctly mapped

**Test Progression**:
```typescript
// 1. Example-based (naive)
expect(reverse([1, 2, 3])).toEqual([3, 2, 1]);

// 2. Property-based (generative)
fc.assert(
  fc.property(fc.array(fc.integer()), (a) => {
    expect(reverse(reverse(a))).toEqual(a);  // Involution
  })
);

// 3. Extracted properties (reusable)
const involution = <T>(a: T[]) =>
  reverse(reverse(a)).isEqual(a);

fc.assert(
  fc.property(fc.array(fc.integer()), involution)
);
```

**Connection to Work**:
- Currents: Property-based testing for payment routing invariants
  - Property: Total input = Total output + Remainder
  - Property: No cycles in graph (flat graph constraint)
- Demonstrates formal verification thinking

#### Tech Stack

- TypeScript + Jest
- fast-check (property-based testing)
- Nx monorepo

---

## Cross-Cutting Themes

### <a name="themes"></a>Technical Patterns Across Projects

#### **1. Functional Programming at Scale**

**Libraries Used**:
- `fp-ts` (TypeScript)
- `fp-go` (Go)
- `immutable.js` (TypeScript)
- `io-ts` (TypeScript)

**Patterns**:
- Pure functions (no side effects)
- Either/Option for error handling (no exceptions)
- Immutable data structures (structural sharing)
- Type-safe decoders (runtime validation)

**Examples**:
- Currents: Payment routing with Either error handling
- Headless-chat: Pure reducers for state management
- Scope3: fp-go monads for fallback strategies

#### **2. Visualization of Complex Concepts**

**Projects**:
- Currents: ForceGraph2D for payment flow
- Scope3: Auto-generated Mermaid diagrams
- Reflector: Waveform visualization for audio

**Why**:
- Complex domains need visual aids
- Stakeholders (non-technical) need to understand
- Debugging benefits from visual traces

#### **3. Self-Documenting Code**

**Patterns**:
- Flow-as-data (Scope3)
- Generated diagrams (Scope3)
- Comprehensive inline docs (Reflector)
- Type-driven development (all projects)

**Benefits**:
- Code and docs stay in sync
- Onboarding easier
- Maintenance clearer

#### **4. Distributed Systems Patterns**

**Concepts**:
- Cursor-based pagination (Headless-chat)
- Gap recovery (Headless-chat)
- Eventual consistency (Currents Stripe caching)
- Durable execution (Reflector Hatchet)
- Automatic retry (Reflector)

**Challenges Solved**:
- Out-of-order delivery
- Network partitions
- Process crashes
- Data loss prevention

#### **5. Python — real surface area, shallow defensible depth**

**Projects**:
- Reflector (FastAPI, PyAV, Hatchet, SQLAlchemy)
- Transcription-Eval (Jiwer, subprocess integration — mostly TS, Python is a subprocess)

**Surface touched** (exposure ≠ depth):
- Async patterns (asyncio, context managers)
- Audio processing (PyAV filter graphs)
- DAG workflows (Hatchet)
- ORM (SQLAlchemy + Alembic migrations)
- Structured logging (structlog)

**Resume Note (§5b governs):** keep **"Python — basic, eager to expand"** (Igor's own rating).
Do NOT upgrade to "intermediate/advanced" on the strength of this code — that is the
artifact-implies-depth fallacy. No Python/async talks or blogs exist, so there is no independent
depth signal. Defend Reflector at *architecture/decision* level (T2); PyAV/fork-safety are
breadth-only (T3). Surface area is wide; deep, recall-defensible Python is not claimed.

---

## Summary: Context for Resume Updates

### What's in Resume

**Updated**:
1. Graph-based Funds Distribution & Visualization ✓
2. AI Transcription Platform (Reflector) ✓ (new)
3. Financial Infrastructure ✓
4. ML Model Evaluation ✓
5. LLM Agent Orchestration ✓
6. LLM Environmental Impact (Scope3) ✓

### What's in STORY.md (Full Detail)

1. **Currents**:
   - Graphuier visualization
   - Snail trace algorithm
   - 99-library monorepo
   - Payment routing deep dive
   - Double-entry accounting
   - Stripe/PayPal integration

2. **Reflector**:
   - Hatchet DAG workflows
   - Daily.co multitrack sync
   - PyAV audio processing
   - Process fork awareness
   - Documentation work

3. **Scope3 AI**:
   - Flow-as-data pattern
   - Auto-generated Mermaid diagrams
   - StaticGraphNode architecture
   - 25+ node inference cascade
   - Type-safe SQL (go-jet)

4. **Transcription-Eval**:
   - WER/DER metrics
   - Multitrack comparison
   - Parallel test execution
   - Cache strategy

5. **Headless-Chat** (personal project):
   - Cursor-based reconciliation
   - Composite message IDs
   - Multi-room architecture
   - Gap detection & recovery
   - WebSocket-first protocol

6. **Property-Based Testing** (tech talk):
   - Educational demonstration
   - fast-check library
   - Involution, length preservation properties

### All Context Preserved

- **Resume**: 6 bullets (Monadical section)
- **STORY.md**: This file - full technical depth
- **CONTRIBUTIONS.txt**: Project list + summaries

Everything from our conversation is now documented. Ready for interviews.
