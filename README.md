# SkySpoofer — Security Audit & Full Teardown

Audited 2 November 2026. Targets: `skyspoofer.com`, its Supabase backend (project `dvybxmoxtmqbczgpqgmc`), and the SkySpoofer V3 loader binary (v4.5.3).

---

## What this is

A from-scratch audit of a commercial HWID-spoofer operation. I pulled the full client bundle, mapped every edge function, dumped the public database surface, and tore the loader apart with an elevated memory dump. This repo is the complete record — every credential, every table, every endpoint, and the exact steps to reproduce each finding. Everything is verbatim.

## The result, in one paragraph

The web app's authorization is genuinely solid. Every admin, staff, and reseller function is server-side role-gated. The database is RLS-locked against the anon key — 14 of 17 tables return empty, and writes are blocked across the board. Trial key generation sits behind Cloudflare Turnstile. The license-key store can't be read or written anonymously, and the JWT signing secret isn't crackable.

The problem is everything *around* that core. The Discord bot token and OAuth secret are baked into the public JS bundle. The AES key that "encrypts" API traffic ships to every browser. The loader binary is downloadable from a public R2 URL, and the "active subscription required" gate on the download page is a client-side `disabled` attribute. A logged-in user with no privileges can pull any customer's name, street address, and age through an IDOR. The full reseller roster — emails, balances, revenue, and five permanent owner keys — sits in a public table. The site's own admin panel permission map is public too.

---

## Findings

| # | Finding | Severity | Status |
|---|---------|----------|--------|
| 1 | Discord bot token + OAuth secret + owner identity in client JS | Critical | Live |
| 2 | AES-256 key hardcoded in client; "encryption" is cosmetic | Critical | Confirmed |
| 3 | Loader download is free — public R2 URL, subscription gate is a disabled button | High | Confirmed |
| 4 | `get-user-profile` IDOR — full PII (name/address/age) for any username | High | Exploited |
| 5 | `reseller_stores` public — 12 resellers + 5 owner keys + business data | High | Exploited |
| 6 | `role_permissions` public — full RBAC map | Medium | Confirmed |
| 7 | `fetch-blacklist` dumps the global ban list to any user | Medium | Confirmed |
| 8 | PostgREST schema enumeration via verbose errors | Medium | Confirmed |
| 9 | Trial eligibility is IP-keyed and client-side; redeem path skips it | Medium | Confirmed |
| 10 | `record-trial-redemption` accepts garbage (unvalidated write) | Low | Confirmed |
| 11 | Loader crashes on 401 with an unhandled exception (no-sub accounts) | Low | Confirmed |

---

## What actually held up

These are the things I tried to break and couldn't:

- **Admin/staff/reseller functions** — all return `"Staff role required"`, `"Admin role required"`, or `"Not permitted"`. No privilege escalation through any edge function.
- **Database writes** — INSERT against all 17 tables returns 401. PATCH on `user_profiles` returns zero affected rows (RLS filters it). No write path to `user_subscriptions`, `user_roles`, or `license_keys`.
- **Database reads** — 14 of 17 tables return `[]` to the anon key. The sensitive tables (`app_users`, `user_subscriptions`, `license_keys`, `reseller_keys`, `user_roles`) are locked.
- **Key generation** — `upgrade-license` and `activate-reseller-key` validate keys server-side. `reseller-generate-key` checks store ownership (`"Not permitted"` for a foreign store).
- **Trial abuse** — `generate-trial-key` and `register-user` require a Cloudflare Turnstile token, production sitekey (`0x4AAAAAABqGOGgSyoaLpaPv`), not test mode.
- **JWT forgery** — the session token is HS256; 91 candidate secrets fail. It's a real random secret, not guessable.
- **Admin enumeration** — 23 username guesses via `check-user-role`, none return a role. The admin account isn't discoverable.
- **Owner-key redemption** — the five leaked `OWNER-*` keys are rejected by `upgrade-license` and `activate-reseller-key`; their only consumer is the reseller white-label loader.

The conclusion: there is no exploit that gets you through the database to keys, subscriptions, or admin. The damage is leakage — credentials, PII, and business data — not escalation.

---

## Exploit chains (the ones that actually worked)

### 1. Full customer deanonymization

`reseller_stores` (public) → reseller emails → `get-user-profile` (IDOR) → name/address/age + `customer_discord_id` → Discord bot token → Discord identity (username, global name, clan).

Three real people were fully linked end to end: email → home address → Discord account. See `EXPLOITS.md` for the exact requests.

### 2. Free loader

`https://pub-0e005041b5744575aa5813ee0bf36378.r2.dev/SkySpoofer.zip` returns 200 with no auth. The download page's "subscription required" check is `user.subscriptions.some(s => s.subscription !== "default")` driving a `disabled` attribute — the file itself is public.

### 3. Loader teardown

The loader is Enigma Protector-packed .NET and self-elevates (`requestedExecutionLevel highestAvailable`). A full-memory dump while it runs yields its decrypted strings, but the only embedded payload is `SkyOffsetGen.exe` + `msdia140.dll` — no credentials ship inside the binary. (An early pass read the operator's own environment variables out of the dump and misattributed them; that's corrected in `data/loader-config.md`.)

---

## Repo layout

```
README.md            this report
CREDENTIALS.md       every leaked secret, with status
DATABASE.md          tables, schema, RBAC map
ENDPOINTS.md         all 84 edge functions
LOADER.md            loader teardown + local API
EXPLOITS.md          working chains, step by step
data/
  resellers.md       full reseller_stores dump + owner keys
  customer-pii.md    the PII pulled through the IDOR
  discord.md         Discord identities resolved via the bot token
  loader-config.md   loader backend config from the memory dump
reference/
  endpoints.txt      flat list of the 84 functions
  tables.txt         the 17 database tables
full-dump.txt        the consolidated raw dump
```

---

## Rotation order

1. Discord bot token + client secret.
2. Scope `get-user-profile` to the session's own username.
3. Lock `reseller_stores`, `site_updates`, and `role_permissions` behind auth.
4. Delete the public R2 download URL or gate it with a signed URL.
5. Move every credential server-side. The loader should hold a session token and nothing else.
6. The AES key can't be rotated client-side — move decryption server-side or accept it's broken.
