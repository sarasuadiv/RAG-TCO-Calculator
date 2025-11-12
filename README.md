# RAG TCO Calculator

## Overview
- A single-page, business-friendly calculator estimating Total Cost of Ownership (TCO) for retrieval-augmented generation (RAG) systems across four architectures: **Long-Context (no retrieval)**, **Classic RAG**, **Hybrid RAG**, and **Agentic RAG** (multi-step with tools).
- Outputs: annual cost, steady monthly run rate, Month 1 total (including one-time items), cost per conversation, and cost per active user per month.
- Includes a **Project Setup & Staffing Breakdown** (time & materials) driving the one-time “Initial setup ($)” and updating totals live.

## Notation
- `t` = tokens (e.g., 510t = 510 tokens)  
- `1K` = 1,000  
- `1M` = 1,000,000 (also 1e6)  
- `$ / M` = dollars per million (tokens or calls)  
- `$ / 1K` = dollars per thousand (tokens or calls)

## How to Use
1. Open **final_tco.html** in a modern browser (no build required).  
2. Pick an architecture preset and adjust business inputs:
   - **Traffic & pattern:** conversations/month, active users, turns per conversation, cross-conversation dedupe (H), cache factor, token sizes.  
   - **Retrieval:** ON/OFF, chunks per turn, tokens per chunk, reuse, corpus size (docs, tokens/doc).  
   - **Vendors:** choose LLM, select Vector DB / Infrastructure / Monitoring presets or use Custom rates.  
   - **Project Setup & Staffing:** edit hours × rates by role (PM, Architect, AI Engineer, Project Director) + contingency (%). Auto-fills “Initial setup (one-time $)” and updates totals.  
3. Review KPIs, cost tables (Variable, Recurring, One-time), architecture diagram, and **Assumptions & Sources**.

## Assumptions (Modeled)
- **Vendor pricing:** reference snapshots with sources and last-updated. LLM rates in $/M tokens. Embedding rates normalized to $/M; $/1K shown for clarity.  
- **Volume & traffic model:** based on conversations/month × turns; dedupe H represents shared templates/caches.  
- **Prompt cache factor:** cached tokens billed at discount (e.g., 0.5).  
- **Retrieval:** active mode adds query embedding and Vector DB usage/storage; corpus embedding is one-time.  
- **Agent steps:** Agentic RAG multiplies variable calls per architecture flags.  
- **Setup:** time & materials by role + contingency; affects Month 1 and Annual only.

## Calculations
- **Conversations/year** = convM × 12  
- **Baseline calls/year** = convM × 12 × turns × (1 − H)  
- **Effective calls/year** = baseline × agentSteps  
- **Average tokens per call:**
  - Fresh input = user tokens + new RAG context  
  - Cached input = system prompt + reused RAG context  
  - Output = base output tokens × reasoning multiplier  
- **LLM cost per call** = (fresh×inputRate + cached×inputRate×cacheFactor + output×outputRate) / 1M  
- **Query Embeddings (usage)** = (queryTokens×embeddingPrice/M)/1M × reads basis  
- **Initial Embedding (one-time)** = (docs×tokensPerDoc×embeddingPrice/M)/1M  
- **Vector DB:**
  - Storage (recurring) = (docs/1M)×storageRatePerM  
  - Reads (usage) = (reads per month/1M)×readsRatePerM  
- **Reranker (usage)** = (reads per month/1K)×rerankerRatePer1K  
- **Tools (usage)** = (variable calls per month/1K)×toolsRatePer1K  
- **Infra & Monitoring:** base monthly + (monthly basis / 1M)×per-million rate  
- **Totals:**
  - Annual = sum of annual amounts  
  - VariableMonthly = sum of variable monthly items  
  - RecurringMonthly = sum of recurring monthly items  
  - Monthly Run-Rate = VariableMonthly + RecurringMonthly  
  - One-time total = sum of annual one-time items  
  - Month 1 = Monthly Run-Rate + One-time total  
  - Per Conversation = Annual / (convM × 12)  
  - Per Active User / Month = Monthly Run-Rate / users

## Expected UI Behavior
- **Live recalculation:** all inputs update KPIs, tables, diagram, and assumptions panel.  
- **Retrieval gating:** disabling retrieval hides Query Embeddings, Vector DB Reads, Reranker, Initial Embedding, and Vector DB (Storage).  
- **Vendor presets vs Custom:**
  - Preset → single monthly line.  
  - Custom → split Storage/Reads and Infra/Monitoring controls.  
- **Project Setup & Staffing:** edits update Breakdown Subtotal/Total and auto-fill Initial setup.  
- **Assumptions & Sources:** shows active components with formulas, units, and sources.

## Use Cases (One per Architecture)

### Long-Context (no retrieval)
- **Scenario:** summarize long documents in prompt.  
- **Expect:** LLM variable cost only (+ Infra/Monitoring if enabled). No Vector DB or embeddings.

### Classic RAG
- **Scenario:** internal FAQs (~1,000 docs × 800 tokens/doc).  
- **Expect:** Initial Embedding (one-time), Vector DB Storage (recurring), Vector DB Reads & Query Embeddings (usage), LLM (usage).  
- Disabling retrieval removes all retrieval-related costs.

### Hybrid RAG
- **Scenario:** large knowledge base with prefilter + reranker.  
- **Expect:** reduced LLM usage, Reranker (usage), Vector DB Storage/Reads.  
- Disabling retrieval removes these components.

### Agentic RAG
- **Scenario:** multi-step orchestration with tool calls.  
- **Expect:** variable lines (LLM, Query Embeddings, Reads, Reranker, Tools) scale with steps.  
- Infra/Monitoring/Storage remain recurring.

## Definitions
- **Conversation:** session (chat or equivalent).  
- **Turn:** request–response step.  
- **H (dedupe):** percent of calls eliminated by shared cache/templates.  
- **cachedFactor:** discount for cached input tokens.  
- **chunks / tokens per chunk:** context fragments per turn; reuse = fraction persisting.  
- **reasoning multiplier:** scales output tokens for deep reasoning.  
- **agentSteps:** orchestration steps per conversation.  
- **Vector DB Storage vs Reads:** monthly storage vs runtime queries.  
- **Initial Embedding vs Query Embeddings:** upfront vs per-query costs.  
- **Setup (one-time):** hours × rate by role + contingency; Month 1 only.

## Defaults & Where to Change Things
- **Rates (custom in final_tco.html):**
  - `vdb_storage_per_million_vectors`  
  - `vdb_reads_per_million`  
  - `infra_base_monthly`, `infra_per_million_calls`  
  - `monitoring_base_monthly`, `monitoring_per_million_calls`  
  - `reranker_per_thousand`, `tools_per_thousand_calls`
- **Vendor pricing:** `PRICING.llm` and `PRICING.embeddings` (show sources and last-updated).  
- **Presets:** `VENDOR_PRESETS`.  
- **Setup:** edit hours/rates/contingency in Breakdown; auto-fills “Initial setup (one-time $)”.

## Limitations & Disclaimers
- Reference pricing only — verify with vendors.  
- Embedding units normalized to $/M; update if contracts use $/1K.  
- Excludes: network egress, SSO/SCIM, security programs, support, custom UI, discounts.  
- Small lines may round to $0.00–$0.01.  
- Context window warning if input tokens approach model limit.  
- Intended for directional planning, not formal quotes.

## Quick Demo Flow
1. Pick architecture preset for use case.  
2. Set conversations/month, active users, turns, and dedupe H.  
3. Toggle retrieval and set corpus size (docs × tokens/doc); adjust chunks/reuse.  
4. Enter setup hours/rates/contingency; confirm Month 1 impact.  
5. Choose vendor presets or custom rates for Vector DB, Infra, Monitoring; toggle Governance/Tools as needed.  
6. Explain two anchors: **Month 1 = Run-Rate + one-time**, **Monthly Run-Rate = operational budget**.
