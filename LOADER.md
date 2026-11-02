# Loader — SkySpoofer V3.exe

Binary: `SkySpoofer V3.exe` (5.38 MB), v4.5.3, .NET, packed with **Enigma Protector** (method names obfuscate to `[obfuscated]`, strings encrypted at rest, `enigmaprotector.com/taggant` markers present). Manifest requests `highestAvailable`, so it self-elevates to admin.

## Download — public, no auth

```
https://pub-0e005041b5744575aa5813ee0bf36378.r2.dev/SkySpoofer.zip            19,525,589 B
https://pub-0e005041b5744575aa5813ee0bf36378.r2.dev/Loader%20[Reseller].zip   18,411,745 B
```

The download page gates this with `user.subscriptions.some(s => s.subscription !== "default")` feeding a `disabled` attribute on the button. The file itself is on a public R2 bucket and returns 200 to anyone, no token.

## Zip contents

```
SkySpoofer V3.exe                    5.38 MB   main loader
SkyFixer/SkyFixer.exe                3.53 MB   cleaner/fixer
HWID Checker/SkyHWID Checker.exe     9.85 MB   HWID checker
HWID Checker/SkyTPM Hash Checker.exe 162 KB    TPM checker
Disable HVCI/DisableVBS_HVCI.bat                VBS/HVCI disabler
HWID Checker/HWIDCHECKER.bat                    HWID check script
Newtonsoft.Json.dll, System.Text.Json.dll, System.IO.Pipelines.dll,
System.Drawing.Common.dll, TaskScheduler.dll, ...   .NET deps
```

The kernel driver is **not** in the zip. The loader pulls offsets at runtime (`SkyOffsetGen.exe` embedded, "kernel offsets now pull directly from our database").

## Embedded resources

```
SkySpoofer_V2.Backend.Embedded.SkyOffsetGen.exe
SkySpoofer_V2.Backend.Embedded.msdia140.dll
SkySpoofer_V2.Sky_logo.png
```

## Runtime flow

Observed by MITM and memory dump:

1. `GET https://dvybxmoxtmqbczgpqgmc.supabase.co/functions/v1/app-version?t=<ts>` — public, UA `SkySpoofer-V2`. Returns `{"version":"4.5.3","download_url":"https://skyspoofer.com/dashboard/download","force_update":true,...}`.
2. Connects to Supabase (`dvybxmoxtmqbczgpqgmc.supabase.co`) and `skyspoofer.com`.
3. Hits a subscription-gated download, gets **401 Unauthorized** without an active sub, and **crashes** — the 401 is never caught. The loader throws `System.AggregateException → System.Net.WebException (401)` and dies instead of showing "no active subscription." That crash is in `error_log.txt` (four entries) and is a plain robustness bug, not a security feature.

## Local API — localhost:1337

The loader runs a local HTTP server that the website's dashboard talks to:

```
handshake      {username, token, twoFactorCode?}
clean          cleaner
cleanstatus    cleaner status
hardwareinfo   device serials
apply          apply spoof
serials        serial profile
spoofstatus    current spoof state
loader-status  ready/progress
```

All traffic to/from it is AES-256-CBC with the hardcoded key from `CREDENTIALS.md`, so the "encryption" of the handshake is recoverable by anyone with the client bundle.

## Embedded payload

The only genuine embedded resources are `SkySpoofer_V2.Backend.Embedded.SkyOffsetGen.exe` and `msdia140.dll`. No API keys ship inside the binary. (An early memory-dump pass pulled the operator's own `KEY=VALUE` environment variables out of the dump and misattributed them as loader config — see `data/loader-config.md` for the correction.)

## How the memory dump was taken

The binary is Enigma-packed and elevated, so static string extraction yields nothing but cert URLs. Instead:

1. Run the loader elevated (it self-elevates via manifest).
2. Wait for it to reach its network calls (strings are decrypted into the managed heap at runtime).
3. `MiniDumpWriteDump` (full memory) from an elevated process — needs admin because the loader runs as admin.
4. Extract ASCII + UTF-16LE strings from the dump.

The decrypted app strings (endpoint URLs, embedded resource names) come out of that dump — but the environment block does too, which is where the operator's unrelated personal keys came from.
