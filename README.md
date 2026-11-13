# Asgard

A family of small, self-hosted apps for one home. Each app works on its own. Together they make
the home network feel like one place: one sign-in, files that arrive where they should, and the
family told when something is ready.

Named after Asgard, the realm where the gods live: a home, with Heimdall guarding its gate. Every
app takes a Norse name chosen for what it does.

> **Status: early.** Several apps are active, one is on hold, and the links between them are mostly
> ideas. The full brainstorm is in [`docs/suite-vision.md`](docs/suite-vision.md).
> Nothing here is a commitment.

## The apps

| Folder | Role | What it is | State |
|---|---|---|---|
| [`Heimdall`](Heimdall) | Identity | OAuth 2.1 / OpenID Connect provider (.NET 10, PostgreSQL, multi-tenant). The suite's single sign-on | Active |
| [`Sleipnir`](Sleipnir) | Files in | Download engine becoming a background service with clients (TUI, web, browser extensions, Android). Also LAN file sharing | Active |
| [`Huginn`](Huginn) | Messages | Real-time chat (Angular + NestJS). Carries family notifications and game invites | Active |
| [`Norns`](Norns) | Workflows | Multi-tenant workflow and approval engine (formerly CoreOps) | On hold |
| [`Ratatoskr`](Ratatoskr) | Files around | .NET TUI file explorer (formerly TuiFileExplorer) | Kept |
| [`Roundelay`](Roundelay), code name **Bragi** | Audio | LAN audio sync with QR pairing. Prior art for LAN pairing. *Not the owner's repo* | Contributing |
| [`JsGames`](JsGames), code name **Tafl** | Games | Offline games PWA and Android app. *Not the owner's repo* | Contributing |

Outside the folder: **Jellyfin** (third-party) handles streaming. Sleipnir drops finished videos
into its library folders.

## How they fit together

- **Heimdall** signs people in. Huginn and Norns plan to use it. Sleipnir starts with its own
  device pairing and adds Heimdall later without changing its user model.
- **Sleipnir** emits events such as "download finished". **Huginn** can turn those into a message.
- **Norns** can hold a download for approval before Sleipnir starts it.
- **Bragi** (Roundelay) already solves "find my home server and pair with it on the Wi-Fi", which several
  apps need.

Example: someone taps a browser extension on a phone, the home server downloads the video into the
Jellyfin library, and Huginn tells the family group it's ready.

## Principles

1. **Each app stands alone.** Integration is optional and adds features. It is never required.
2. **Loose coupling.** Apps talk through events and standard protocols (OIDC, HTTP), not shared code or databases.
3. **Home first.** Everything works on the local network, without an account on someone else's server.
4. **One thing at a time.** Build the smallest link that proves a scenario before the next.

## Conventions

- One flat PascalCase folder per app, named from Norse myth.
- Roundelay and JsGames keep their repo and folder names because they are not the owner's repos. Inside the suite they go by Norse code names: **Bragi** (Roundelay) and **Tafl** (JsGames). Use the code names in suite docs and conversation.
- Reserved names for future apps: Muninn (photos, notes), Bifröst (gateway into the home network),
  Gjallarhorn (alerts), Hliðskjálf (dashboard), Mímir (search), Hermod (spare). See the vision file
  for the reasoning.

## Next step

The first link to build is small: Sleipnir publishes a "download finished" event and Huginn shows it.
