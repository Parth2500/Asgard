# Suite vision (parked idea)

> **Status: parked. Not planned, not designed.** These notes record ideas from a brainstorm on
> 2026-11-13 so they can be revisited once the apps below are further along. Nothing here is a
> commitment. Sleipnir's own design only *leaves room* for them.

## The projects

All live under `~/Projects/Asgard`, one flat PascalCase folder each (convention set 2026-11-13). Roundelay and JsGames keep their repo and folder names because they are not the owner's repos; inside the suite they go by the code names Bragi and Tafl.

| Project | What it is | Possible role in a suite |
|---|---|---|
| **Sleipnir** | Download engine becoming a background service with clients (TUI, web UI, browser extensions, Android) | Moves files into the home: downloads, LAN file sharing, app/pack distribution |
| **Heimdall** (formerly `heimdall-idp`) | Owner's OAuth 2.1 / OpenID Connect identity provider (.NET 10, PostgreSQL, multi-tenant) | Single sign-on for the suite. Norns and Huginn already plan to use it, and the games could later |
| **Norns** (formerly CoreOps) | Multi-tenant workflow engine (on hold); signs in with Heimdall | Approval workflows, e.g. "request a large download → admin approves" |
| **Huginn** (formerly `demo-chat-app`) | Real-time chat (Angular + NestJS); Heimdall sign-in planned | Family notifications ("your download is ready"), game invites |
| **Tafl** (code name; repo `JsGames/games`) | Offline games PWA + Android (Capacitor), optional PHP/MySQL sync backend | Family/LAN multiplayer; packs and APK updates served by the home server |
| **Bragi** (code name; repo Roundelay) | LAN audio sync with QR pairing, host fingerprint, per-guest keys (repo under the DarpanVadher GitHub account) | Synced audio for movie nights; **prior art for LAN pairing** |
| **Jellyfin** (third-party) | Media server; replaces the dropped HomeFlix/LocalStream app | Streaming; Sleipnir drops finished videos into its library folders |
| **Ratatoskr** (formerly TuiFileExplorer) | .NET TUI file explorer | Sibling TUI; could share a look |

## Names (Norse theme)

Names are scarce once a theme is set, so they are allocated on purpose (2026-11-13):

| Name | Mythology | Use |
|---|---|---|
| **Heimdall** | Guardian who sees and hears everything | ✅ identity / sign-in (folder renamed from `heimdall-idp`, 2026-11-13) |
| **Huginn** | Odin's raven, "Thought" | ✅ chat (folder renamed from `demo-chat-app`, 2026-11-13) |
| **Sleipnir** | Odin's eight-legged horse, travels between the worlds | ✅ this download manager (folder renamed from "DotNet Download Manager", DNM, 2026-11-13). Eight legs, eight connections |
| **Muninn** | Odin's raven, "Memory" | reserved: photo backup, notes, memories |
| **Bifröst** | The bridge between worlds, guarded by Heimdall | reserved: a gateway into the home network (reverse proxy/VPN) |
| **Gjallarhorn** | Heimdall's horn that alerts the worlds | reserved: notifications / alerts |
| **Norns** | The weavers of fate | ✅ workflow engine (renamed from CoreOps, 2026-11-13). Also covers scheduling |
| **Hliðskjálf** | Odin's seat that sees all nine worlds | reserved: home dashboard / monitoring |
| **Mímir** | The well of wisdom | reserved: knowledge base / search |
| **Asgard** | The realm where the gods live; Heimdall guards its gate | ✅ the whole suite (chosen over Yggdrasil, 2026-11-13: it is a home, and Yggdrasil clashes with an existing mesh-network project). Yggdrasil stays spare, e.g. for an integration layer |
| **Ratatoskr** | Squirrel carrying messages up and down the world tree | ✅ TUI file explorer (renamed from TuiFileExplorer, 2026-11-13). Command alias: `rata` |
| **Bragi** | God of poetry and music | ✅ code name for Roundelay (LAN audio sync). Repo and folder keep the name Roundelay: not the owner's repo |
| **Tafl** | Norse board-game family | ✅ code name for JsGames (games PWA). Repo and folder keep the name JsGames: not the owner's repo |
| **Hermod** | The gods' messenger | spare |

## Decisions so far

- **Heimdall is not used by Sleipnir for now.** It gets a role once the apps are further along.
  Sleipnir's sign-in is designed to be *pluggable*: built-in device pairing first, with an OIDC/Heimdall
  provider added later without changing Sleipnir's user and role model.
- Huginn and Norns use Heimdall because the owner owns them all. The games may adopt it later.
- Streaming is Jellyfin's job, not Sleipnir's.

## Scenarios imagined

1. **Movie night.** A browser-extension click on a phone makes the home server download a video
   straight into the Jellyfin library. Jellyfin rescans and Huginn tells the family group it's ready.
   Bragi (Roundelay) syncs the audio to everyone's headphones.
2. **"Can I download this?"** A family member requests a big download. A Norns workflow sends the
   admin a Huginn notification, and on approval Sleipnir schedules it for the night.
3. **Phone to PC, no cable.** Share a photo from the phone to the home PC over Wi-Fi via Sleipnir. No
   internet, no account.
4. **Family games night.** Chess or backgammon over the home network, invites through Huginn, scores
   synced with one family sign-in (Heimdall, later).
5. **App updates at home.** The home server downloads a new Games APK or puzzle pack once; family
   phones fetch it from the PC over Wi-Fi.
6. **Before a trip.** Sleipnir on the phone pre-downloads shows, maps or game packs on home Wi-Fi.

## What Sleipnir's design leaves room for

- Events other apps can subscribe to ("download finished", "share received").
- Per-download destination folders (e.g. a Jellyfin library) and scheduling (start at night).
- An optional "needs approval" state that an external workflow (Norns) could drive.
- Discovery and pairing as reusable parts, since several suite apps need "find my home server on the Wi-Fi".
- Users and roles that can map onto an external identity (Heimdall `sub`) later.
