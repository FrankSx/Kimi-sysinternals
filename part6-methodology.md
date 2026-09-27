# Kimi Sysinternals, Part 6: The Methodology and the Bug Ledger — How the Recon Was Actually Done

*Tools, traps, and the honest record. Every hard-won rule we now follow, earned the expensive way.*

---

## The Methodology

1. **Capture beats inference.** A HAR export from a live browser session (149 chunks, 12.8MB) outperformed every attempt to fetch the shell from outside. When the renderer strips scripts, proxies die, and archives time out — capture at the source.
2. **Descriptors are the schema.** Protobuf clients ship their own documentation as base64 `FileDescriptorProto` blobs. A 30-line varint walker + strict TLV parser (not regex) extracted the full RPC catalog — exact service/method names, zero guessing.
3. **Probe error surfaces deliberately.** 401s, 400s, and 404s each carry structured lessons (Part 1's taxonomy). The server *wants* to tell you the schema — buf.validate violations name fields.
4. **Collaborator captures are force multipliers.** The single biggest unlock (Part 5's framed-RPC send) came from a teammate's 2026-07-21 capture with the exact note: "request body is NOT raw JSON — Connect-protocol streaming framed." Three prior turns of enum-guessing collapsed to one line of truth. **Publish your notes.**
5. **Test against live state, then fresh-process verify.** In-session caches (`sys.modules`) produced stale results that looked like code bugs more than once. The raw-dump check caught it every time.

## The Bug Ledger (abridged, chronological)

| Trap | Cost | Rule now |
|---|---|---|
| FTS5 contentless table breaks `snippet()` | one debugging round | standard FTS5 + join column |
| `mesh_node` computes DB paths at import | silent wrong-DB connects | override module attrs on attach |
| Method shadowing (`self._core` vs `_core()`) | confusing tracebacks | name caches `*_cache` |
| SQLite `disk I/O error` after ~200 inserts | three rewrites of connection handling | **check `statvfs` FIRST** — it was a full disk |
| Non-greedy `\[STATE\](\{.*?\})` truncated nested JSON | silently amputated registries | greedy `\{.*\}` to the closing tag |
| Assistant prose dropped when a STATE block coexisted | lost routing replies | extraction is orthogonal to prose |
| Title-matching a controller whose chat got server-renamed | bound to an empty twin | bind by **activity score**, never title |
| XID ids compared at 16 chars | false pagination dupes | full-ID comparisons only |
| Raw JSON to a Connect endpoint | detail-less `invalid_argument` | `0x00 + BE32 + JSON`, always |
| Client-forged chat_id on new chats | 404 `REASON_CHAT_NOT_FOUND` | omit chat_id; server mints |
| Stale `sys.modules` in my own harness | "bugs" that were ghosts | fresh-process verification |

## The Toolchain We Left Behind

- `token_manager.py` — vault, flock-rotation, browser-header set
- `memory_core.py` — harvest / recall / correlate with checkpoints
- `dream.py` — reactivation/consolidation/co-activation/fading + reckoning
- `dream_carrier.py` / `dream_controller.py` — server-side project memory + triage
- `chat_registry.py` — the 899-chat searchable pre-harvest index
- `escrow.py` — signUrl credential bridge
- `tests/smoke.py` — 10-check offline suite, run after every edit

## The Standing Queue

- Full 899-conversation harvest grind (registry makes it on-demand rather than bulk)
- File attachment reference shape (escrow works; in-message file refs still rejected)
- Dream narrator on platform API keys (`api.moonshot.ai` is key-authenticated, separate from the gateway)
- The mXSS Bible's Part VIII (Trusted Types defense half) — the model's own review named it

## Closing

We set out to make the memory slots obsolete and ended up mapping the whole animal: a Connect-RPC gateway with a custom two-token auth, a typed block-stream data model, a behavioral memory system with a dark dream switch — and on top of it, a memory that grows while you sleep, dreams in dialogues, and moves between conversations carrying itself.

The platform was the textbook. We wrote the field notes.

*Stay paranoid. Stay safe.*

— FrankSx
