# Kimi Sysinternals, Part 1: The Gateway — Mapping a Connect-RPC Surface Nobody Published

*Series: Kimi Sysinternals — what we found diving into the internal systems of a production AI chat platform, and what we built with the knowledge.*

---

## The Target

kimi.ai's web app is a Vue 3 SPA. Every server interaction funnels through a single gateway:

```
POST https://www.kimi.ai/apiv2/<package>.<Service>/<Method>
Authorization: Bearer <jwt>
Connect-Protocol-Version: 1
Content-Type: application/connect+json   (streaming) / application/json (unary)
X-Msh-Device-Id / X-Msh-Session-Id / X-Traffic-Id / X-Language / R-Timezone
```

This is **Connect-RPC** (Buf's protocol) over protobuf — not REST, not GraphQL. The web client is generated protobuf-es code. Nobody had published a map of this surface. We made one.

## How We Got the Client (When the CDN Was "Unreachable")

The page renderer strips `<script>` tags; HTML proxies died; web archives timed out. What worked: **a HAR export from a live browser session** — 149 JS chunks, 12.8MB, including the protobuf descriptor layer (`*_pb.js`).

The descriptor layer is the crown jewel: each `*_pb.js` contains a base64 `FileDescriptorProto` blob — the service's entire schema, compiled in. Decoding it requires a varint-aware protobuf walker (30 lines), and it yields the **exact RPC catalog**: package names, service names, method names.

## The Catalog

126+ methods across 19+ services, decoded from the blobs. The load-bearing ones:

| Service | Methods that matter |
|---|---|
| `kimi.gateway.chat.v1.ChatService` | Chat, ResumeChat, ListChats, ListMessages, GetChat, GetChatShareDocx |
| `kimi.gateway.feed.v1.FeedService` | ListFeeds (the conversation list — with project_id filtering) |
| `kimi.gateway.file.v1.FileService` | Upload (presigned multipart) |
| `kimi.gateway.project.v1.ProjectService` | CreateProject, SetChatProject, ListProjectFiles |
| `kimi.usersetting.v1.UserSettingService` | GetUserSetting, UpdateUserSetting (with updateMask paths) |
| `kimi.gateway.config.v1.ConfigService` | GetConfig, GetAvailableModels |

And a full second tier: Search, Swarm, KimiPlus, Context, Tool, Suggest, RemoteControl, Membership, Claw, IM, Skill, Plugin, Notification services.

## The Wall: Cloudflare 1010

Every route — even unauthenticated `/api/config` — returns `error code: 1010` to non-browser User-Agents. The key: **a full browser header set** (UA + Origin + Referer + Accept + the X-Msh family) sails through. Bearer alone authorizes; no cookies needed.

This one discovery shaped everything after: from inside a session runtime, with the right headers, the entire gateway answers.

## The Error Taxonomy (Live-Captured)

| Code | Shape | Lesson |
|---|---|---|
| 401 | `unauthenticated` + `REASON_INVALID_AUTH_TOKEN` | token dead → rotate |
| 400 | `invalid_argument` + buf.validate violations | schema feedback is generous |
| 400 | `...id_kind=xid,uuidv8...` | **IDs are XID-in-UUIDv8** — time-ordered |
| 404 | empty body | unknown RPC |
| 1010 | Cloudflare | send browser headers |

That ID leak is quietly huge: chat/message IDs embed a timestamp in their first 4 bytes (XID epoch 2013-01-01). Conversation IDs sort chronologically — incremental sync becomes a watermark comparison, and ID-prefix comparisons must use more than 16 hex chars (they share timestamp prefixes; truncated comparisons look like false duplicates).

## Why This Matters

A platform's client bundle is its own documentation if you know where to look. The descriptor blobs gave us the whole API; the error taxonomy told us the ID scheme; the headers got us in. Everything in Parts 2–5 rides on this map.

*Tools referenced: HAR export, protobuf varint walker, differential probing. Next: Part 2 — the strangest OAuth-like token system we've ever mapped.*
