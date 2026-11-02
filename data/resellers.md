# Reseller Database Dump

Full `reseller_stores` dump, public via the anon key. 12 stores.

```
slug              owner_email                  company          balance  keys  revenue  tier  owner_key             custom_domain
----------------  ---------------------------  ---------------  -------  ----  -------  ----  --------------------  --------------------------
cyprus-community  henryholt2026@atomicmail.io  Cyprus Community 14       16    0        none  -                     -
spoefx            alexmarten68@gmail.com       spoefx           2.5      4     0        none  OWNER-G9FR-9ETV-349Q  -
silentlock        talinsikes4@gmail.com        SilentLock       0        0     0        none  -                     -
nassimshop        nassimkalashsss@hotmail.com  nassimshop       0        0     0        none  -                     -
phantom-market    sabanigino241@gmail.com      Phantom-Market   1        16    0        none  OWNER-WV3S-X228-91JM  -
ryze              cooldeadninja2021@gmail.com  Ryze             30       3     0        none  OWNER-98JS-VIDO-J8JE  -
junkie-menu       flixzsmm.shop@gmail.com      JUNKIE MENU      22.5     9     0        none  OWNER-X1FF-IA5S-6LN2  junkiespoofer.com
mattyashop        yuzakirakurotuki@gmail.com   MattyaShop       0        0     0        none  -                     -
spash-spoofer     griffin.tierney71@gmail.com  Spash Spoofer    0        0     0        none  -                     spashspoofer.com
etherix-services  snowj4807@gmail.com          Etherix Services 0        0     0        none  -                     -
bypass-hub        damonsutherlin76@gmail.com   Bypass Hub       0        0     0        none  -                     spoofer.bypasshub.shop
zr-markt          lucabbd46@gmail.com          zr markt         26       1     0        none  OWNER-YU9N-R34W-Q6GD  https://zrmarkt.com/
```

## Owner keys (leaked)

```
OWNER-G9FR-9ETV-349Q   spoefx         alexmarten68@gmail.com
OWNER-WV3S-X228-91JM   phantom-market sabanigino241@gmail.com
OWNER-98JS-VIDO-J8JE   ryze           cooldeadninja2021@gmail.com
OWNER-X1FF-IA5S-6LN2   junkie-menu    flixzsmm.shop@gmail.com
OWNER-YU9N-R34W-Q6GD   zr-markt       lucabbd46@gmail.com
```

## Discord servers

```
junkie-menu  https://discord.gg/junkiemenu
bypass-hub   https://discord.gg/u22YVJmAqT
```

## Extra account surfaced

`silentlock`'s `support_email` is `kathypillans@gmail.com` — a second account belonging to the same person as `talinsikes4@gmail.com` (talin sikes).

## Store schema (full column list)

```
id, user_id, store_slug, company_name, logo_url, primary_color, support_email,
support_discord, is_active, subscription_expires_at, total_keys_generated,
total_revenue, created_at, updated_at, balance, owner_key, owner_key_created_at,
subscription_tier, subscription_started_at, subscription_stripe_id,
monthly_keys_generated, monthly_keys_reset_at, welcome_title, welcome_description,
secondary_color, custom_button_text, banner_url, footer_text, seo_title,
seo_description, dashboard_welcome_title, dashboard_welcome_description, key_mask,
custom_domain
```
