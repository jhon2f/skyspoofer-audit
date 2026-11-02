# Discord Identities

Resolved with the leaked bot token (`GET /discord.com/api/v10/users/<id>`, `Authorization: Bot <token>`). The token returns full user objects even though the bot sits in zero guilds.

```
discord_id          username      global_name        clan   linked account
------------------  ------------  -----------------  -----  ---------------------------
1254256648105627658 talinsikesss1 TOXICツ DA CHEATER  TRMP   talin sikes (2 accounts)
1169510343219879948 nassim.1092   sloerkoe.nl😋       YESH   Nassim Kalash
860872744253653002  7hacki        7hacki              ZRM    Jonas Hehl
```

Raw responses (trimmed to the identifying fields):

```json
// 1254256648105627658
{"id":"1254256648105627658","username":"talinsikesss1","discriminator":"0",
 "global_name":"TOXIC\u30c4 DA CHEATER","accent_color":16711680,
 "clan":{"identity_guild_id":"1328528022843691042","tag":"TRMP"}}

// 1169510343219879948
{"id":"1169510343219879948","username":"nassim.1092","discriminator":"0",
 "global_name":"sloerkoe.nl\ud83d\ude0b",
 "clan":{"identity_guild_id":"1006262653951414313","tag":"YESH"}}

// 860872744253653002
{"id":"860872744253653002","username":"7hacki","discriminator":"0",
 "global_name":"7hacki",
 "clan":{"identity_guild_id":"1452090580862505147","tag":"ZRM"}}
```

The bot itself:

```json
// GET /users/@me
{"id":"1402586782683631646","username":"Sky Free Trials","bot":true,
 "verified":true,"mfa_enabled":true,"discriminator":"3544"}
```

Its owner, leaked via `/applications/@me`:

```json
{"id":"1083483399248957522","username":"lolkid2167","global_name":"GameEngine2024","discriminator":"0"}
```
