# Kimi Sysinternals, Part 3: The Conversation Data Model — Blocks, Threads, and Process-as-Data

*Every chat is a tree of messages; every message is a typed block stream; and the tool blocks are the closest thing to a session's true memory.*

---

## The RPC Shape

```
ChatService/ListMessages  {"chat_id": xid, "page_size": 100, "page_token": cursor}
→ {"messages": [...], "nextPageToken": "<base64url cursor>"}   # cursor is EXCLUSIVE
```

Cursors chain cleanly — verified with a 3-page walk. (`ListFeeds` cursors work identically; both accept snake_case `page_token`.)

## The Message Envelope

```json
{"id": "1a08001c-...", "parentId": "...", "role": "user|assistant",
 "status": "MESSAGE_STATUS_COMPLETED", "scenario": "SCENARIO_CHAT",
 "createTime": "...", "isGoal": false, "blocks": [...]}
```

`parentId` chains turns into a thread tree — some messages have no parent (roots). Roles serialize lowercase (`user`, not `ROLE_USER`) — the gateway's JSON mapping strips enum prefixes. IDs are XID-in-UUIDv8 (Part 1), so `createTime` order equals ID order.

## The Block Union

Every block is `{id, parentId, messageId, createTime, <type>: payload}`:

| Type | Payload | What it really is |
|---|---|---|
| `text` | `{content}` | the turn's prose |
| `think` | `{content, summary}` | reasoning + **the model's own TL;DR** |
| `tool` | `{toolCallId, name, args (JSON string), contents[], status}` | **the session's process record** |
| `file` | `{id, meta{sha256, contentType, sizeBytes}, blob{signUrl, previewUrl}, tokenCount}` | attachments w/ time-limited direct URLs |
| `stage` / `multiStage` | `{name: STAGE_NAME_*, status}` | pipeline timeline markers |

## Why Tool Blocks Are the Goldmine

For the `ipython` tool, `args` is `{"code": "..."}` — the exact code — and `contents` is the joined output, **errors included**. A harvested conversation therefore preserves not just conclusions but *process*: every attempt, every `Traceback`, every retry. Months later, the corpus tells you what was tried, not just what worked.

For files, `blob.signUrl` is an **unauthenticated, time-limited** direct download — the only zero-auth byte channel we found in the whole system (Part 5's escrow rides on it).

## Pagination Traps We Hit

1. **XID truncation false-dupes:** comparing IDs cut to 16 chars makes different messages look identical (shared timestamp prefix). Cost us one debugging round.
2. **Cursors are exclusive but sanity-check anyway** — dedupe by full ID on ingest.
3. **`page_size` caps** below what you'd like; plan cursors, not big pages.

## The Harvester's Extraction Rules

Validated over a real 850-conversation corpus:

- `think.content` → thought artifact (index `summary` as its title — free compression tier)
- `tool.args` parsed per tool name → code artifact, output appended
- `text` → text artifact; `file` → metadata + optional signUrl fetch, deduped by sha256
- Idempotent re-sync via content-hash dedupe; XID watermark for incrementals

## Why This Matters

This schema is why "export your chats" was never enough — the value isn't the transcript, it's the typed block stream. A corpus built on blocks understands *code* differently from *thoughts* differently from *files*, and that semantic distinction is what makes downstream recall and dreaming possible.

*Next: Part 4 — the memory system itself: model-written slots, the settings RPC, and the dream switch that was turned off.*
