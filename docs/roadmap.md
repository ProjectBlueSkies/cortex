# Roadmap

## Phase 0 — Foundation

**Goal:** Neo4j running locally, schema initialized, Python CRUD layer working.

- [ ] Docker Compose file to run Neo4j locally (`~/.cortex/docker-compose.yml`)
- [ ] Schema initialization script (`src/db/init_schema.py`) — constraints, indexes
- [ ] Python `GraphClient` class wrapping the `neo4j` driver with connection pooling
- [ ] Basic CRUD: `upsert_node()`, `upsert_relationship()`, `get_node()`, `query_layer()`
- [ ] Cursor management: `src/daemon/cursors.py` — read/write `~/.cortex/cursors.json`
- [ ] `.env` setup and config loader
- [ ] Manual test: connect to Neo4j, write a few nodes, verify in Neo4j browser (http://localhost:7474)

**Done when:** Can write a hand-crafted node to Neo4j and retrieve it with Cypher.

---

## Phase 1 — Extraction Pipeline

**Goal:** Haiku reads session transcripts and populates the graph automatically.

- [ ] Diff reader: `src/daemon/diff_reader.py` — scans `~/.claude/projects/*/conversations/*.jsonl` for new turns since cursor
- [ ] Chunker: `src/daemon/chunker.py` — token-bounded batches with overlap
- [ ] Haiku extractor: `src/daemon/extractor.py` — sends chunks to Haiku, parses JSON response
- [ ] Entity resolver: `src/daemon/resolver.py` — exact ID match + fuzzy dedup
- [ ] Graph writer: `src/daemon/writer.py` — Cypher MERGE transaction
- [ ] Main daemon entrypoint: `src/daemon/main.py`
- [ ] Claude Code `Stop` hook configuration
- [ ] Decay pass: `src/maintenance/decay.py`
- [ ] systemd timer for decay (nightly)

**Done when:** Run a Claude Code session, trigger daemon, see nodes appear in Neo4j browser.

---

## Phase 2 — Context Injection

**Goal:** Claude receives relevant memory at the start of every session without manual effort.

- [ ] Context loader: `src/retrieval/context_loader.py` — 4-layer Cypher queries + neighbour expansion
- [ ] Relevance ranker: confidence × recency scoring
- [ ] Markdown formatter: per-layer sections with relationship hints
- [ ] Token budget enforcement
- [ ] Reinforcement-on-read: confidence boost for retrieved nodes
- [ ] Claude Code `PreCompact` hook configuration
- [ ] CLAUDE.md integration — include `~/.cortex/CONTEXT.md`

**Done when:** Start a session and observe that Claude's opening context includes accurate memory from previous sessions.

---

## Phase 3 — Semantic Enhancement

**Goal:** Replace fuzzy string matching with embedding similarity; enable semantic search.

- [ ] Embedding generation: `src/embeddings/embedder.py` using `sentence-transformers` (local, no API cost)
- [ ] Populate `embedding` property on Fact and Skill nodes
- [ ] Neo4j vector index creation (requires Neo4j 5.11+)
- [ ] Update entity resolver to use cosine similarity for deduplication
- [ ] Semantic search query: `src/retrieval/semantic_search.py` — find relevant nodes by query string
- [ ] Update context loader to use semantic search for session-relevant retrieval

**Done when:** Two differently-worded descriptions of the same fact are correctly merged rather than creating duplicates.

---

## Phase 4 — Visualization & Tooling

**Goal:** Make the graph inspectable and provide tools for manual curation.

- [ ] CLI tool: `cortex status` — node counts per layer, recent events, graph stats
- [ ] CLI tool: `cortex search <query>` — semantic search from terminal
- [ ] CLI tool: `cortex forget <node-id>` — manually archive a node
- [ ] CLI tool: `cortex show <node-id>` — print node + neighbours
- [ ] Neo4j Bloom or custom D3.js visualization of the graph (optional)
- [ ] Synthesis trigger: `cortex synthesize` — send a cluster of related Events to Haiku and ask for pattern synthesis → creates `:Synthesis` node

**Done when:** Can inspect, search, and manually curate the graph from the terminal.

---

## Non-Goals (for now)

- Multi-user support — this is a single-user local system
- Cloud sync — all data stays on-device by design
- Real-time streaming extraction — session-end trigger is sufficient
- GUI — Neo4j browser covers visualization needs for Phase 0–3
