# Asgard

A family of apps in two groups. One group lives on the internet: an identity provider and a
messenger. The other lives in a home: one service that the family's devices pair with, and the
things that home can do, such as downloading and streaming video. A home can link to the online
group so that its owner hears about it wherever they are.

Named after Asgard, the realm where the gods live: a home, with Heimdall guarding its gate. Every
app takes a Norse name chosen for what it does.

> **Status: early.** The apps are at very different stages, Bifröst is a design only, and none of
> the links between apps is built. Suite-level decisions are in
> [`docs/decisions.md`](docs/decisions.md).

## The two groups

| | Online group | LAN group |
|---|---|---|
| Apps | Heimdall, Huginn | Bifröst and its extensions |
| Runs | Hosted, on a real domain | On one machine at home, and on the family's devices |
| Who you are | A Heimdall account | A device paired with the home |
| Needs the internet | Yes | No, after install |

## The apps

| Folder | Group | What it is | State |
|---|---|---|---|
| [`Heimdall`](Heimdall) | Online | OAuth 2.1 / OpenID Connect provider (.NET 10, PostgreSQL). Sign-in for the online group | Active |
| [`Huginn`](Huginn) | Online | Real-time chat (Angular + NestJS). Also shows notices from linked homes | Active |
| [`Bifrost`](Bifrost) | LAN | The home: pairing, people and devices, discovery, extensions, the link to the online group | [Design](Bifrost/docs/design.md), no code |
| [`Sleipnir`](Sleipnir) | LAN extension | Download manager. Also runs alone on one machine | Active |
| [`Saga`](Saga) | LAN extension | Video streaming for the home. Replaces Jellyfin once it is good enough | Design |
| Files | LAN extension | Device-to-device file sharing, if it is split from Sleipnir | Idea |
| [`Norns`](Norns) | LAN extension, or online | Workflow and approval engine (formerly CoreOps) | On hold |
| [`Ratatoskr`](Ratatoskr) | Beside the LAN group | .NET TUI file explorer (formerly TuiFileExplorer). Touches no network | Kept |
| [`Roundelay`](Roundelay), code name **Bragi** | Separate | LAN audio sync with QR pairing. *Not the owner's repo* | Contributing |
| [`JsGames`](JsGames), code name **Tafl** | Separate | Offline games PWA and Android app. *Not the owner's repo* | Contributing |

## How they fit together

- **Heimdall** signs people in to the online group. No LAN app uses it for sign-in.
- **Bifröst** is installed first at home. It then installs Sleipnir, Saga and the other extensions
  from their GitHub releases. A device pairs once, with Bifröst, and reaches every extension.
- **Extensions** talk to each other through Bifröst's local events. Sleipnir announces a finished
  download, and Saga scans.
- **The link** between the groups is one-way in direction: the home's admin links it to their
  Heimdall account, and from then on the home calls out. It sends notices to Huginn and reads the
  answers. Nothing online can reach into a home, and homes never connect to each other.

Example: a child's tablet asks for a video. The home opens a request and sends the parent a notice
in Huginn. The parent approves, on the phone or at the home machine. The home downloads the video,
Saga adds it to the library, and every paired device can watch it.

## Principles

1. **Each group stands alone.** The online group works without any home. A home works without the internet.
2. **Bifröst first, extensions optional.** Each extension adds one capability. Sleipnir also runs with no Bifröst, on one machine.
3. **The home dials out.** The online group only answers questions a home asked.
4. **Loose coupling.** Apps talk through events and standard protocols (OIDC, HTTP), not shared code or databases.
5. **One thing at a time.** Build the smallest slice that proves a link before the next.

## Conventions

- One flat PascalCase folder per app, named from Norse myth. Folder names are plain ASCII (`Bifrost`). Each app is its own repository. This
  folder's repository holds the suite docs only.
- Roundelay and JsGames keep their repo and folder names because they are not the owner's repos.
  Inside the suite they go by Norse code names: **Bragi** and **Tafl**.
- A decision that affects more than one app goes in [`docs/decisions.md`](docs/decisions.md), not in
  one app's spec.
- Reserved names for future apps: Muninn (photos, notes), Gjallarhorn (alerts), Hliðskjálf
  (dashboard), Mímir (search), Hermod (spare). See [`docs/suite-vision.md`](docs/suite-vision.md).

## Next steps

1. Test that the Android web view enforces a pinned key. The client app depends on it.
2. Build [Bifröst](Bifrost/docs/design.md) by extracting it from Sleipnir, with Sleipnir as the first extension.
3. Start Saga as the second extension.
