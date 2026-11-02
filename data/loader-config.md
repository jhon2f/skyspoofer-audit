# Loader Backend

Correction from the first pass: the `KEY=VALUE` strings recovered from the `SkySpoofer V3.exe` memory dump (`OPENROUTER_API_KEY`, `ANTHROPIC_*`, `CLICKHOUSE_*`, `DROPBASE_RELEASE_AUTH_KEY`, `MODELHUB_API_KEY`, `CVC_API_KEY`) were the process **environment block**, inherited from the auditing machine's user environment. They belong to the operator's personal AI/data tooling and are **not** part of the loader. They have been removed from this repo.

## What the loader actually embeds

```
SkySpoofer_V2.Backend.Embedded.SkyOffsetGen.exe
SkySpoofer_V2.Backend.Embedded.msdia140.dll
SkySpoofer_V2.Sky_logo.png
```

`SkyOffsetGen.exe` is the offset generator that pairs with the "kernel offsets pull directly from our database" feature — it writes `SkyOffsets.dat` to `%SystemRoot%\System32\SkyOffsets.dat` and symbol data to `%ProgramData%\SkyOffsetGen\syms`. Its server-side auth key is not present in the loader's own strings.

## Genuine loader-only findings

- Download is public: `https://pub-0e005041b5744575aa5813ee0bf36378.r2.dev/SkySpoofer.zip` (19.5 MB) and `Loader [Reseller].zip` (18.4 MB), no auth.
- Version check: `GET /functions/v1/app-version?t=<ts>`, UA `SkySpoofer-V2`, public.
- Local API: `http://localhost:1337/` (handshake, clean, apply, serials, spoofstatus, hardwareinfo, loader-status).
- Packed with Enigma Protector; manifest `highestAvailable`.
- The 401 crash on a no-subscription account is a genuine robustness bug (unhandled `WebException`).
