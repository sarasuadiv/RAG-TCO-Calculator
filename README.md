RAG TCO Calculator

Overview
- A single‑page, business‑friendly calculator that estimates the Total Cost of Ownership (TCO) for retrieval‑augmented generation (RAG) systems across four architectures: Long‑Context (no retrieval), Classic RAG, Hybrid RAG, and Agentic RAG (multi‑step with tools).
- Produces annual cost, steady monthly run rate (run‑rate), Month 1 (including one‑time items), cost per conversation, and cost per active user per month.
- Includes a Project Setup & Staffing Breakdown (time & materials) that drives the one‑time “Initial setup ($)” and updates all totals live.

Notation
- t = tokens (e.g., 510t = 510 tokens)
- 1K = 1,000
- 1M = 1,000,000 (also shown as 1e6)
- $/M = dollars per million (tokens or calls)
- $/1K = dollars per thousand (tokens or calls)

How To Use
1) Open final_tco.html in a modern browser (no build required).
2) Pick an architecture preset and adjust the business inputs:
   - Traffic & pattern: conversations/month (or sessions), active users, turns per conversation (or requests per session), cross‑conversation dedupe (H), cache factor, token sizes.
   - Retrieval: ON/OFF, chunks per turn, tokens per chunk, reuse, corpus size (documents, tokens/document).
   - Vendors: choose LLM, and select Vector DB / Infrastructure / Monitoring presets or use Custom rates.
   - Project Setup & Staffing: edit hours × rates by role (PM, Architect, AI Engineer, Project Director) and contingency (%). The mini‑calculator auto‑fills “Initial setup (one‑time, $)”, which updates Month 1 and Annual immediately.
3) Review KPIs, cost tables (Variable, Recurring, One‑time), architecture diagram, and “Assumptions & Sources” for transparent math and sources.

Assumptions (Modeled)
- Vendor pricing: reference snapshots shown in the UI with source and last‑updated. LLM rates are in $/M tokens. Embedding rates are normalized to $/M for math; the panel also shows $/1K for clarity.
- Volume & traffic model: uses conversations/month and turns as a simple proxy for RAG traffic (map to sessions and requests if your workload is not chat). Applies cross‑conversation dedupe H to represent shared templates/caches.
- Prompt cache factor: cached input tokens are billed at a discounted factor (e.g., 0.5) vs fresh tokens.
- Retrieval (RAG): when active, runtime query embeddings and Vector DB reads incur usage costs; Vector DB storage is a monthly recurring; corpus embedding is one‑time when docs × tokens/doc > 0.
- Agent steps: in Agentic RAG, multi‑step orchestration multiplies variable call volume according to the architecture flags.
- Setup (one‑time): “Project Setup & Staffing” is time & materials by role (PM, Architect, AI Engineer, optional Project Director) plus contingency. It only affects Month 1 and Annual.

Calculations
- Conversations/year = convM × 12 (treat as sessions/year for non‑chat)
- Baseline calls/year = convM × 12 × turns × (1 − H) (treat turns as requests/session for non‑chat)
- Effective calls/year = baseline × agentSteps (only if the architecture scales variable or reads with steps)
- Average tokens per call (convAverages):
  - Fresh input = user tokens + new RAG context
  - Cached input = system prompt + reused RAG context
  - Output = base output tokens × reasoning multiplier
- LLM cost per call = (fresh×inputRate + cached×inputRate×cacheFactor + output×outputRate) / 1M (1,000,000)
- Query Embeddings cost (usage, if retrieval ON) = (queryTokens × embeddingPrice/M) / 1M × reads basis
- Initial Embedding (one‑time) = (docs × tokensPerDoc × embeddingPrice/M) / 1M
- Vector DB (custom):
  - Storage (recurring) = (docs / 1M) × storageRatePerM
  - Reads (usage) = (reads per month / 1M) × readsRatePerM
- Reranker (usage) = (reads per month / 1K) × rerankerRatePer1K
- Tools (usage) = (variable calls per month / 1K) × toolsRatePer1K
- Infra & Monitoring (custom): base monthly + (relevant monthly basis / 1M) × per‑million rate (presets provide flat monthly or base+per‑M where applicable)
- Totals:
  - Annual = sum of annual amounts
  - VariableMonthly = sum of monthly amounts tagged “variable”
  - RecurringMonthly = sum of monthly amounts tagged “recurring”
  - Monthly Run‑Rate = VariableMonthly + RecurringMonthly
  - One‑time total = sum of annual amounts tagged “one‑time”
  - Month 1 = Monthly Run‑Rate + One‑time total
  - Per Conversation (annual) = Annual / (convM × 12)
  - Per Active User / Month = Monthly Run‑Rate / users (if users > 0)

Expected UI Behavior
- Live recalculation: every input change updates KPIs, cost tables, architecture diagram, and the “Assumptions & Sources” panel.
- Retrieval gating: if embed query is OFF, chunks are 0, or corpus size is 0, the tool hides Query Embeddings, Vector DB Reads, Reranker, Initial Embedding, and the Vector DB itself (including Storage). Re‑enable retrieval with a corpus to see these lines again. Diagram and costs stay coherent.
- Vendor presets vs Custom:
  - Preset shows a single monthly line (e.g., Vector Database).
  - Custom splits Vector DB into Storage (recurring) and Reads (usage), and exposes Infra/Monitoring base + per‑million knobs.
- Project Setup & Staffing: changing hours/rates/contingency updates the Breakdown Subtotal/Total and auto‑fills the “Initial setup (one‑time, $)” field, affecting Month 1 and Annual immediately. Project Director can remain at 0.
- Assumptions & Sources: shows only active components with formulas, units, and sources; includes a notation primer (t, 1K, 1M/1e6, $/M, $/1K).

Use Cases (One per Architecture)
- Long‑Context (no retrieval)
  - Scenario: summarize long documents pasted directly in the prompt.
  - Expect: LLM variable cost only (+ Infra/Monitoring if turned on). No Vector DB or embeddings. Month 1 = Run‑Rate + Initial setup.
- Classic RAG
  - Scenario: internal FAQs (~1,000 docs × 800 tokens/doc).
  - Expect: Initial Embedding (one‑time), Vector DB Storage (recurring), Vector DB Reads and Query Embeddings (usage), LLM (usage) — when retrieval is active (embed query ON, chunks/tokens > 0) and a corpus is defined. Turning off embed query (or zeroing chunks/tokens) disables retrieval components entirely, removing Reads, Query Embeddings, Initial Embedding and the Vector DB (including Storage). Setting docs=0 removes Storage and Initial Embedding.
- Hybrid RAG
  - Scenario: large knowledge base where prefilter + reranker reduce context.
  - Expect: lower LLM usage (reduced context), Reranker (usage), Vector DB Storage/Reads — when retrieval is active and a corpus exists. Turning off embed query disables these retrieval components. Lowering chunks or increasing reuse reduces LLM and Reads.
- Agentic RAG
  - Scenario: multi‑step orchestration with tool calls.
  - Expect: variable lines (LLM, Query Embeddings, Reads, Reranker, Tools) scale with steps when retrieval is active and a corpus exists; Infra/Monitoring/Storage remain recurring under those conditions.

- Definitions
- Conversation: a session (for chat, a user conversation).
- Turn: one request–response step (for chat, a back‑and‑forth exchange).
- H (dedupe): percent of calls eliminated by shared cache/templates.
- cachedFactor: discount factor for cached input tokens.
- chunks / tokens per chunk: context fragments added per turn; reuse is the fraction that persists.
- reasoning multiplier: scales output tokens for deeper reasoning.
- agentSteps: number of orchestration steps per conversation that multiply variable/reads (per architecture flags).
- Vector DB Storage vs Reads: monthly storage for the corpus vs per‑query reads at runtime.
- Initial Embedding (one‑time) vs Query Embeddings (per turn): corpus embedding upfront vs per‑query embeddings during retrieval.
- Setup (one‑time): hours × rate by role (PM, Architect, AI Engineer, Project Director) plus contingency; Month 1 only.

Defaults & Where To Change Things
- RATES (custom knobs inside final_tco.html):
  - vdb_storage_per_million_vectors, vdb_reads_per_million
  - infra_base_monthly, infra_per_million_calls
  - monitoring_base_monthly, monitoring_per_million_calls
  - reranker_per_thousand, tools_per_thousand_calls
- Vendor pricing: PRICING.llm and PRICING.embeddings in final_tco.html show sources and last‑updated.
- Vector/Infra/Monitoring presets: VENDOR_PRESETS.
- Setup: edit hours/rates/contingency in the Breakdown; it auto‑fills “Initial setup (one‑time, $)”.

Limitations & Disclaimers
- Reference pricing only: confirm with your vendors (region, discounts, and dates vary).
- Embedding units: math normalizes to $/M; if your contract is $/1K, update embedding price accordingly for realistic magnitudes.
- Not modeled: network egress, SSO/SCIM setup, detailed compliance/security programs, premium support, custom UI builds, volume‑tier discounts.
- Rounding: very small monthly lines may display as $0.00–$0.01 due to two‑decimal formatting.
- Context window: the tool warns if average input tokens approach/exceed the model’s context window.
- Intended for directional planning and comparisons, not formal quotes.

Quick Demo Flow
1) Pick the architecture preset aligned to the customer use case.
2) Set conversations/month, active users, turns, and dedupe H.
3) Decide retrieval ON/OFF and set corpus size (docs × tokens/doc); adjust chunks/reuse.
4) Enter setup hours/rates/contingency in the Breakdown; confirm Month 1 impact.
5) Choose vendor presets or custom rates for Vector DB, Infra, Monitoring; toggle Governance/Tools as needed.
6) Explain two anchor numbers: Month 1 (Run‑Rate + one‑time) and Monthly Run‑Rate (operational budget).
