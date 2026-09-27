# Kimi Sysinternals, Part 2: The Auth Model — A Custom Token Pair in a Connect-RPC World

*Not OAuth2. Not JWT-bearer-simple. A two-token rotation system with its own RPC, its own death rules, and a 15-minute heartbeat.*

---

## The Two Tokens

Both live in localStorage. Both are JWTs (HS512, iss `account`):

- **access_token** — `typ: "access"`, exactly **900 seconds** of life. Claims carry the full identity envelope: `sub` (traffic id), `device_id`, `ssid` (session id), `region`, `membership.level`, `code_membership.level`, `jti`.
- **refresh_token** — `typ: "refresh"`, **sliding ~90-day** expiry (a rotated token we minted carried ~3 months, not the original's full remaining life).

## The Rotation (Live-Proven)

```
POST https://auth.kimi.ai/api/account.gateway.v1.AuthService/RefreshToken
{"refreshToken": "<current refresh token>"}
→ {"accessToken": "<new jwt>", "refreshToken": "<new refresh token>"}   # BOTH rotate
```

Three rules that will bite you:

1. **Rotation kills the old pair instantly.** Two refreshers on one vault = death race. Serialize with a file lock.
2. **The browser also refreshes.** If the browser wins the race, your vault's pair is dead. Re-bootstrap, or accept the vault as the pair's sole owner.
3. **One vault per device.** The pair is bound to `device_id` (and the header family derived from its claims). Don't copy vaults across machines.

We rotated a real pair end-to-end from inside a session: mint → verify via `GetCurrentUser` → the whole lifecycle in one sitting.

## The Client's Own Implementation

The bundle's `refreshAccessToken()` de-dupes concurrent refreshes with a promise singleton and writes both tokens back on success. It also carries a `BAN_REASONS` table — `banned/suspended/disabled/maintenance` are account-side states no rotation can fix.

## The Gateway Access Recipe

With a valid pair, any /apiv2 call needs only:

```python
headers = BROWSER_HEADERS | {
    "Authorization": f"Bearer {access}",
    "X-Msh-Device-Id": claims["device_id"],
    "X-Msh-Session-Id": claims["ssid"],
    "X-Traffic-Id": claims["sub"],
}
```

No cookies. No shield header (the trustdecision fraud SDK adds `x-msh-shield-data` when active, but rotation worked without it in our region).

## The Vault We Built

`token_manager.py`: 0600 file, flock-serialized rotation, `ensure_fresh(min_ttl=120)` returns a ready header dict — the one-liner every harvester/bridge calls before any gateway request. Bootstrap = read the pair from localStorage, write the vault. From then on, the system mints its own access.

## The Honest Caveats

- Pasting tokens into chat is a bad habit with a 15-minute shelf life — Part 5's escrow exists because of it.
- 401 on refresh = pair dead = re-bootstrap. There is no recovery path around that.
- The ~90-day sliding window means even a perfect system must re-touch the vault quarterly.

## Why This Matters

Long-lived automation against a chat platform usually dies on auth. Here, the auth is fully understood, fully implemented, and live-proven — which is what made everything else in this series possible.

*Next: Part 3 — the conversation data model: blocks, threads, and why tool calls are the real memory of a session.*
