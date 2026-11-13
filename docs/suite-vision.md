# Suite vision

> **Status: ideas, with the structure now decided.** These notes began as a brainstorm on
> 2026-11-13. On 2026-11-14 the owner decided how the suite is shaped: two groups, with Bifröst as
> the home. Those decisions are in [`decisions.md`](decisions.md) and they win over anything here.
> The scenarios and names below are still ideas, not commitments.

## The projects

All live under `~/Projects/Asgard`, one flat PascalCase folder each (convention set 2026-11-13). Roundelay and JsGames keep their repo and folder names because they are not the owner's repos; inside the suite they go by the code names Bragi and Tafl.

| Project | Group | What it is | Role |
|---|---|---|---|
| **Heimdall** (formerly `heimdall-idp`) | Online | Owner's OAuth 2.1 / OpenID Connect identity provider (.NET 10, PostgreSQL) | Sign-in for the online group. One tenant for everyone, open sign-up |
| **Huginn** (formerly `demo-chat-app`) | Online | Real-time chat (Angular + NestJS), signs in with Heimdall | Messaging, and notices from linked homes ("your download is ready", "approve this request") |
| **Bifröst** | LAN | The home service. Design only: [`Bifrost/docs/design.md`](../Bifrost/docs/design.md) | Pairing, people and devices, discovery, extensions, the link to the online group |
| **Sleipnir** | LAN extension | Download manager. Also runs alone on one machine | Moves files into the home |
| **Saga** | LAN extension | Video streaming, borrowing Jellyfin's media code. Design only | Plays what Sleipnir fetched |
| **Norns** (formerly CoreOps) | LAN extension, or online | Workflow engine (on hold) | Approval workflows, e.g. "request a download → admin approves" |
| **Tafl** (code name; repo `JsGames/games`) | Separate | Offline games PWA + Android (Capacitor), optional PHP/MySQL sync backend | Idea: LAN multiplayer. No multiplayer exists today |
| **Bragi** (code name; repo Roundelay) | Separate | LAN audio sync with QR pairing, host fingerprint, per-guest keys (repo under the DarpanVadher GitHub account) | **Prior art for LAN pairing.** Not an extension: its host is the device playing the audio, not the home machine |
| **Jellyfin** (third-party) | Outside | Media server | Streaming until Saga beats it |
| **Ratatoskr** (formerly TuiFileExplorer) | Beside the LAN group | .NET TUI file explorer | Sibling TUI; touches no network |

## Names (Norse theme)

Names are scarce once a theme is set, so they are allocated on purpose (2026-11-13):

| Name | Mythology | Use |
|---|---|---|
| **Heimdall** | Guardian who sees and hears everything | ✅ identity / sign-in (folder renamed from `heimdall-idp`, 2026-11-13) |
| **Huginn** | Odin's raven, "Thought" | ✅ chat (folder renamed from `demo-chat-app`, 2026-11-13) |
| **Sleipnir** | Odin's eight-legged horse, travels between the worlds | ✅ this download manager (folder renamed from "DotNet Download Manager", DNM, 2026-11-13). Eight legs, eight connections |
| **Muninn** | Odin's raven, "Memory" | reserved: photo backup, notes, memories |
| **Bifröst** | The bridge between worlds, guarded by Heimdall | ✅ the home service: what devices pair with, what extensions plug into, and the bridge to the online group (2026-11-14). It is not a gateway into the home from outside |
| **Saga** | Goddess of stories, who drinks with Odin each day | ✅ video streaming (formerly HomeFlix / LocalStream, 2026-11-13) |
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

See [`decisions.md`](decisions.md) for the full entries. In short:

- Two groups: online (Heimdall, Huginn) and LAN (Bifröst and its extensions).
- Heimdall is for the online group only. No LAN app signs in through it. Heimdall Lite is dropped.
- Bifröst is the home. Sleipnir, Saga, file sharing and Norns are extensions it installs.
- Sleipnir still runs alone on one machine.
- The home links to the online group as a client, and always dials out. Homes never connect to each other.
- Devices away from the host use one Bifröst client app with a pinned key.
- Streaming is Saga's job once it beats Jellyfin, not Sleipnir's.

## Scenarios imagined

1. **Movie night.** A tap on a phone makes the home download a video straight into Saga's library.
   Saga scans and the home sends a "ready" notice through Huginn.
   Bragi syncing the audio to headphones needs a new Bragi mode: today it shares the system audio
   of the device it runs on, and it has no Linux host.
2. **"Can I download this?"** The target scenario, in full in [`decisions.md`](decisions.md). A paired
   device asks, Norns in the home opens a request, the admin gets a Huginn notice and approves there
   or on the LAN, Sleipnir downloads, Saga scans.
3. **Phone to PC, no cable.** Share a photo from the phone to the home PC over Wi-Fi via Sleipnir. No
   internet, no account.
4. **Family games night.** Chess or backgammon over the home network, invites through Huginn.
   Tafl has no multiplayer today, so this is all new work in a repo the owner does not steer.
5. **App updates at home.** The home server downloads a new Games APK or puzzle pack once; family
   phones fetch it from the PC over Wi-Fi.
6. **Before a trip.** Sleipnir on the phone pre-downloads shows, maps or game packs on home Wi-Fi.

## What Sleipnir's design leaves room for

- Events other extensions can subscribe to ("download finished", "share received"), sent through Bifröst.
- Per-download destination folders (e.g. Saga's library) and scheduling (start at night).
- An optional "needs approval" state that Norns can drive.

Discovery, pairing, users and roles were also in Sleipnir's design. They now belong to Bifröst.

## Later ideas

- **Huginn on the LAN.** Messages and audio calls between devices in one home with no internet,
  with Bifröst as the courier. The online server stays the only authority (decision S9). This waits
  for end-to-end encryption and the native app.
- **A named HTTPS address for linked homes.** Parked: native clients make it unnecessary.
