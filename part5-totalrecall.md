# Kimi Sysinternals, Part 5: TotalRecall — The Memory Layer We Built on the Map

*Everything from Parts 1–4 became components: a harvester, a dream engine, a reckoning layer, a controller, a carrier inside a Kimi project, and an auth escrow. This is the architecture and the live results.*

---

## The Thesis

Memory slots are an index, not a memory. The corpus — every conversation's text, code, tool calls, thoughts, files — is the real asset, but it was unreachable: no search, no cross-conversation view, no consolidation. TotalRecall turns the block stream (Part 3) into a long-term, self-growing memory, with the dream system (Part 4's open slot) as its engine.

## The Pipeline

```
harvest (ListFeeds→GetChat→ListMessages, cursor-paginated, checkpointed, idempotent)
   → artifacts (typed: text / thought(+summary) / code(+output) / file(sha256-deduped))
   → index (SQLite FTS5 + correlation edges: shared_file w=1.0, term-overlap cosine)
   → recall (content+title, provenance-ranked) + correlate (cross-conversation link graph)
```

Measured on the real account: 899 conversations registered, per-conversation harvest at ~3.7s, mid-conversation checkpointing so a single 500-artifact chat can grind across timeouts without loss.

## The Dream Engine (neuro-inspired, inspectable)

Phases, per the reactivation / selective-prioritization / memory-updating model:

- **REACTIVATION** — replay each episode; score events: decisions +1, **errors +1.5** (failures are most salient), emphasis +1, discovery +1
- **CONSOLIDATE** — `episode_salience = log(events) × (1 + 0.15·edge_degree) × e^(−age/60)`
- **CO-ACTIVATION** — an episode lights up remote memory (edges + neocortex nodes sharing its project)
- **UPDATING** — the neocortex node **merges in place** (facts deduped, open-state refreshed, activation_count++)
- **FADING** — exponential decay; faded episodes deprioritized, never deleted

Live: 25 episodes distilled into neocortex nodes that genuinely aggregated remote conversations (a `svg` node co-activating 6 conversations; a `content` node holding 9).

## RECKONING — dreams as preemptive thought-reckoning time

A dream scans a conversation, extends its research area against the corpus, and forms an **informed memory**: understanding (merged with the neocortex), momentum (next threads — *including remote ones*), co-relations (weighted, with reasons). Live proof: reckoning the PDF-XSS conversation surfaced a next-move from a *different* conversation (MESCALINE APEX) — cross-conversation momentum, working.

## Multi-Round Dream Dialogues

Dreams became dialogues: round 1 posts the reckoning; round 2 injects **remote-memory snippets** (real corpus hits); round 3 resolves tensions with explicit `ASSUMPTION:` markers; round 4 finalizes a `[STATE]{...}` block + three memory-slot lines. The model's round-4 output contained understanding the extractive reckoning lacked — the dialogue genuinely *reasoned forward* with the corpus as ground truth.

## The Carrier — server-side memory inside a Kimi project

The insight that made the local DB a cache: **a Project is a persistent, conversation-native container.** `TotalRecall Dreams` (use_memory=true) holds one dream-episode conversation per research area, each carrying a `[STATE]` block + model narration, readable from any session on any device. The controller conversation triages briefs (`[ROUTE]{stream, priority, mode}`) and owns a wholesale-rewritten schedule STATE — it even *corrected its own routing* once a new stream existed.

The send protocol that powers it (cracked from a collaborator's captured send script): **Connect-framed bodies** — `0x00 + BE32 length + JSON`; raw JSON gets read as frame headers and dies detail-less. New chat: **omit chat_id entirely** (the server mints it; a client-forged id 404s). Appends: `parent_id = newest message id`.

## The Auth Escrow — sandbox moves without pasting tokens

The last-mile problem: fresh sandboxes need credentials. The bridge uses the one zero-auth byte channel (Part 3's signUrls): encrypt the vault with a passphrase (PBKDF2 200k + HMAC keystream), upload via the multipart file endpoint, hand the next sandbox **a URL + a passphrase**. Zero JWTs cross chat; zero gateway auth needed before the vault exists.

## Why This Matters

The platform gave us the map (Part 1), the keys (Part 2), the schema (Part 3), and an open slot (Part 4). What we added is the layer that makes a decade of conversations *compound* — and every piece of it runs on verified, documented surface.

*Finale: Part 6 — the methodology and the bug ledger: how the recon was actually done, and every trap we fell into so you don't have to.*
