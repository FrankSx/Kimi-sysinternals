# Kimi Sysinternals — A Reverse-Engineering Field Journal

What we found diving into the internal systems of a production AI chat platform (kimi.ai),
and what we built with the map.

## The Series

| Part | Title | Core finding |
|---|---|---|
| 1 | [The Gateway](part1-gateway.md) | Connect-RPC surface mapped from protobuf descriptor blobs; Cloudflare 1010 key; XID-in-UUIDv8 IDs |
| 2 | [The Auth Model](part2-auth.md) | Custom two-token rotation (15-min access / 90-day sliding refresh), live-proven |
| 3 | [The Conversation Data Model](part3-conversations.md) | Typed block stream; tool blocks as process records; signUrls as the zero-auth byte channel |
| 4 | [The Memory System](part4-memory.md) | Model-written slots (memory_space tools); settings RPC; auto_dream present but OFF |
| 5 | [TotalRecall](part5-totalrecall.md) | The memory layer built on the map: dream engine, reckoning, project carrier, controller, auth escrow |
| 6 | [Methodology & Bug Ledger](part6-methodology.md) | How the recon was done; 11 traps and the rules they bought |

## The Companion System

TotalRecall (the subject of Part 5) ships separately: harvest → index → recall →
correlate → dream → reckon → carry. Toolkit, cold-start kit, and tests live in
their own repos/zips.

## Format

- `*.md` — GitHub source
- `*.html` — blog-ready renders (same content, styled)

## License

Use it, share it, learn from it. Cite the series if it saves you a week.

— FrankSx
