# Kimi Sysinternals, Part 4: The Memory System — Model-Written Slots, Server Dreams, and the Switch That Was Off

*Kimi's memory isn't a database you query. It's a behavior the model performs — and the settings RPC that controls it was sitting there the whole time.*

---

## How Slots Actually Get Written

The discovery that reshaped our design: **there is no user-facing write API for memory slots.** Slots are written by the *model*, as tool calls, mid-conversation:

- `memory_space_insert` / `memory_space_update` / `memory_space_delete`

These render as `semanticMemory` blocks in the conversation — which means **every memory write lands in the harvestable corpus**. A complete audit trail of what the system chose to remember, diffable over time. (Item schema: `{id, content}` — instruction-style lines, ≤500 chars.)

## The Settings RPC

```
UserSettingService/GetUserSetting     → full settings tree
UserSettingService/UpdateUserSetting  → {"userSetting": {...}, "updateMask": {"paths": [...]}}
```

The memory subtree, live-read from the account:

```json
"memory": {
  "useSemanticMemory": true,
  "useEpisodicMemory": true,
  "userConfigured": true,
  "autoDream": { "enabled": false, "userConfigured": false }
}
```

One toggle per subsystem — and **`autoDream` was OFF**. Also in the tree: `skillEvo.enabled: true` — proof that per-user server-side background jobs already run against accounts. That's the deployment pattern a dream system needs.

## The Memory Types

- **Semantic** — the slots (model-written, instruction-format)
- **Episodic** — event-style memories (the `dreamMemory` util suggests dream-time consolidation writes these)
- **`skillEvo`** — skill evolution, enabled

And `ContextService` exposes Goals — a memory-adjacent store our catalog found but we never needed to touch.

## What This Meant for TotalRecall

Three design consequences, all forced by reality:

1. **Writes go through injection, not API.** TotalRecall distills corpus → emits an injection prompt → the model writes its own slots. The audit trail then harvests those writes back. No private endpoints, no rituals that break when the client updates.
2. **The dream slot was open.** A server-side auto_dream exists but is a black box (and off). Our dream engine (Part 5) is the inspectable version: same neuroscience-inspired phases, plus a journal the user can read.
3. **The per-user job pattern is proven.** skillEvo already does it. Pitching a backend dream worker isn't speculative architecture — it's asking to occupy an existing, enabled pattern.

## The BAN_REASONS Reminder

The client carries explicit account-side states — `banned/suspended/disabled/maintenance` — outside the token lifecycle. No amount of rotation fixes those; they're policy, not auth.

## Why This Matters

Platforms reveal their philosophy in plumbing. Here: memory is *behavioral* (the model writes it), *configurable* (a clean settings RPC), and *background-capable* (skillEvo proves the job pattern) — but the dream layer is opaque and, on this account, dark. That's not a complaint; it's the gap a system like ours is built to fill.

*Next: Part 5 — TotalRecall: the memory architecture we built on top of this map, including server-side dreams living inside a Kimi project, and the token escrow that makes sandbox moves painless.*
