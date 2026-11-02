# Leaked Credentials

Everything below was recovered from the public client bundle, the public database surface, or the loader's memory. Status reflects a live check made on 2026-11-02.

## Discord

Recovered from `assets/discord-BhuJZqAI.js` — the OAuth + bot stack, shipped to every browser.

```
clientId      1402586782683631646
clientSecret  qypfBf-fAV2UX2A7VzOJYpfoZGH0JejC
  botToken      MTQwMjU4Njc4MjY4MzYzMTY0Ng
                .G0VBHx
                .mHc0wwpB4BXRwlOcQdEK2hChGwkiUjxZ6fDufM   [LIVE]
guildId       1113925627255463947
roleId        1113925627268038717
```

Status:
- **botToken: LIVE.** `GET /users/@me` returns HTTP 200 — bot `Sky Free Trials`, id `1402586782683631646`, `verified: true`, `mfa_enabled: true`.
- **clientSecret: LIVE.** `POST /oauth2/token` (client_credentials) issues an access token with scope `guilds.join guilds.members.read identify guilds`.
- **guildId / roleId: stale.** The bot is in zero guilds (`GET /users/@me/guilds` → `[]`); the guild id returns `Unknown Guild`.

`/applications/@me` with the bot token additionally leaked:

```
verify_key  ede999d97ab80038e5d7a33836b87f7c32b340b25a26b49f635657cc838b1181
redirect    https://skyspoofer.com/discord/callback
OWNER       lolkid2167 / global_name GameEngine2024 (id 1083483399248957522)
```

The bot token doubles as a Discord ID → identity oracle. It resolved the three snowflakes in `data/discord.md` to full profiles (username, global name, avatar, clan tag) despite the bot being guildless.

## Supabase

Anon key (public, in the client bundle):

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImR2eWJ4bW94dG1xYmN6Z3BxZ21jIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NDUzODQzMTMsImV4cCI6MjA2MDk2MDMxM30.jwTLUImKlMZmkGFEYM47U5a4kH9-xFV5VCqWKFFOGfg
```

## Client-side crypto

AES-256-CBC key, hardcoded in `index-BatRCnDp.js` and used for all `{encrypted: ...}` API traffic and the loader's `localhost:1337` handshake:

```
A7F9C3E21D4B8A6F92E5C7D1B0FA43CE6D9A1F3B7C2E8D54A6F0B1C9E3D7F842
```

Format: 16-byte random IV prepended to the ciphertext, then base64 — `base64(IV || AES-256-CBC(payload))`.

## Reseller owner keys

Permanent access keys for five resellers, sitting in the public `reseller_stores.owner_key` column:

```
OWNER-G9FR-9ETV-349Q   spoefx (alexmarten68@gmail.com)
OWNER-WV3S-X228-91JM   phantom-market (sabanigino241@gmail.com)
OWNER-98JS-VIDO-J8JE   ryze (cooldeadninja2021@gmail.com)
OWNER-X1FF-IA5S-6LN2   junkie-menu (flixzsmm.shop@gmail.com)
OWNER-YU9N-R34W-Q6GD   zr-markt (lucabbd46@gmail.com)
```

Not redeemable through `upgrade-license` or `activate-reseller-key` (both reject with "invalid key"). Their only consumer is the reseller white-label loader, which authenticates through the same SkyAuth server-side gate.

---

## Note on the loader memory dump

An earlier pass attributed `OPENROUTER_API_KEY`, `ANTHROPIC_AUTH_TOKEN`/`ANTHROPIC_BASE_URL`, `CLICKHOUSE_*`, `DROPBASE_RELEASE_AUTH_KEY`, `MODELHUB_API_KEY`, and `CVC_API_KEY` to the loader. That was wrong. Those `KEY=VALUE` strings came from the loader process's **environment block** — the binary inherited the auditing machine's user environment, so the operator's personal keys (for unrelated AI/data tooling) appeared verbatim in the dump. They are **not** skyspoofer's and have been removed from this record.

The loader's genuine embedded payload is just `SkyOffsetGen.exe` and `msdia140.dll` (see `LOADER.md`).
> The bot token is split across three lines above (join with dots, no spaces). It is a live Discord secret and GitHub push protection blocks it in single-line form; rotate it before publishing.
