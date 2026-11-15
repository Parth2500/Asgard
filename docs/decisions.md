# Asgard suite decisions

Decisions that affect more than one app. A decision inside one app belongs in that app's own docs.

Each entry records the decision, why, what it costs, and what it replaces. Change a decision by
adding a new entry that names the one it replaces. Do not edit an old entry.

All entries below were made by the owner in the review conversation of 2026-11-13 and 2026-11-14.
Background: [`suite-review-2026-11-13.md`](suite-review-2026-11-13.md).

---

## S1. Two groups: online and LAN

- **Decision:** the suite has two groups.
  - **Online group:** Heimdall and Huginn. Hosted on the internet by the owner.
  - **LAN group:** Bifröst and its extensions (Sleipnir, Saga, file sharing, Norns). One install per home.
- **Why:** the two halves rest on opposite assumptions (accounts and public certificates against
  paired devices and pinned keys). Forcing one model on both produced Heimdall Lite and the
  certificate problem on the home network.
- **Cost:** there is no single sign-in across both groups. A person is an account online and a set
  of paired devices at home. S7 links the two.
- **Replaces:** the single "family of apps for one home" framing in the README.

## S2. Heimdall serves the online group only

- **Decision:** no LAN app signs people in through Heimdall. Heimdall Lite is dropped.
- **Why:** Heimdall needs HTTPS on a DNS name, PostgreSQL and account management. None of that
  fits a home machine, and a paired device needs none of it.
- **Cost:** Saga and Sleipnir get their people and devices from Bifröst (S4), which must exist first.
- **Replaces:** in Saga's spec: "login through Heimdall at release", "standalone is a development
  rule only", and Heimdall Lite. In Sleipnir's spec: the later OIDC sign-in.

## S3. One Heimdall tenant, open sign-up

- **Decision:** everyone who uses the online group is a user in one Heimdall tenant. A home is not
  a tenant. Sign-up is open. "Who may sign up" is a setting, so it can be closed if traffic demands.
- **Why:** friends in separate tenants could not share one Huginn. Each tenant has its own user pool.
- **Cost:** Heimdall needs sign-up, email verification and password reset before anyone else joins.
  Its threat model already requires them before a public deployment. The owner operates a service
  that holds other people's accounts and messages.

## S4. Bifröst is the home

- **Decision:** Bifröst is the one service installed on the home machine. It owns the home's key,
  the paired devices and people, discovery, local events, the install and update of extensions, and
  the one outbound link to the online group. It owns no downloads, no media and no app screens.
- **Why:** Saga and Sleipnir both need pairing, discovery, TLS and device management. Two copies
  would fight over ports and force a person to pair twice.
- **Cost:** Bifröst is on the critical path for everything on the LAN. It downloads and runs
  programs, so it must verify them.
- **Replaces:** Bifröst as "a gateway into the home network (reverse proxy/VPN)" in the vision file,
  and the "common network service that each app installs if missing" in Saga's spec.
- **Design:** [`Bifrost/docs/design.md`](../Bifrost/docs/design.md).

## S5. Extensions are separate programs installed through Bifröst

- **Decision:** a person installs Bifröst first. Bifröst then installs the latest release of each
  extension from its GitHub repository. Extensions are Sleipnir, Saga, file sharing (if it is split
  from Sleipnir) and Norns. Each runs as its own program. The list is fixed to the owner's
  repositories.
- **Why:** one install order, one place for updates. Separate programs keep a crash in one from
  stopping the others, and keep Saga's GPL-2.0 licence from reaching Bifröst and Sleipnir.
- **Cost:** an extension interface that must stay compatible, because extensions ship on different
  dates.

## S6. Sleipnir keeps a standalone mode

- **Decision:** Sleipnir alone still works on one machine, with no users and no pairing. Anything
  that involves another device needs Bifröst. Saga and file sharing do not run without Bifröst.
- **Why:** the standalone tool exists and works. Bifröst adds devices and people to Sleipnir, and
  Sleipnir adds a download manager to Bifröst.
- **Cost:** Sleipnir has two modes: a local token when alone, Bifröst's identity as an extension.
- **Replaces:** Sleipnir's own users, roles, devices, pairing, LAN listener and discovery. They
  move to Bifröst.

## S7. The home is a client of the online group, and always dials out

- **Decision:** the admin links a home to their Heimdall account once, on the host. From then on
  the home calls the online group. Nothing online connects into a home. The online group can only
  answer a question the home asked. Homes never connect to each other. The only path between two
  homes is a Huginn message between two people.
- **Why:** no domain, port forwarding or public certificate is needed at home. A compromised online
  app can wrongly answer a pending question, but it cannot give the home a new command.
- **Cost:** Huginn needs a notice lane for apps, where a notice can carry actions (S9).

## S8. Native apps are the rule away from the host

- **Decision:** a device other than the host uses the one Bifröst client app, which pins the home's
  key at pairing. The app hosts every extension's screens. A browser is used on the host only,
  through `localhost`. Plain HTTP for other browsers is a fallback that is off by default and
  limited to viewing.
- **Why:** a browser cannot pin a key, and no certificate for a LAN address is trusted by a browser
  without a real domain or a manual install on each device. A TV cannot do the manual install.
- **Cost:** Saga is not usable on a TV until the Android TV client exists. Devices with no client
  (iPhone, non-Android TVs, laptops at first) fall back to plain HTTP or wait.
- **Replaces:** the browser-first viewer plan in Saga's spec, and the idea of a real domain for the home.

## S9. Huginn has one authority

- **Decision:** the online Huginn server is the only authority for message order, read marks and
  membership. Later, the LAN may become a second *delivery path* for messages and calls between
  devices in one home. There is never a second Huginn server.
- **Why:** two servers that accept messages for the same chats must reconcile them, which is the
  hardest problem in messaging.
- **Cost:** LAN delivery waits for end-to-end encryption and the native app. Sign-in, device
  linking and membership changes still need the internet.
- **Also:** notices from apps are not chat messages. Chats will be encrypted and bots stay out of
  scope (Huginn D10), so notices get their own lane.

## S10. Norns is one codebase, installable at home or online

- **Decision:** Norns can be installed as a Bifröst extension or as an online service. Two
  installations never sync. A home's Norns holds that home's requests.
- **Why:** it works with no internet at home, and it avoids a "household" concept in a shared
  online Norns.
- **Cost:** storage that is not tied to SQL Server, and sign-in that accepts either Bifröst's
  identity or Heimdall.

## S11. Bifröst owns the folders and who may see them

- **Decision (2026-11-14):** Bifröst holds the list of folders in the home. Each person has one
  folder, with a `private` and a `public` subfolder. Bifröst decides which one a download goes to.
  When the download finishes, Saga scans only that subfolder. A file under `private` is seen only by
  its owner and the admin. A file under `public` is seen by everyone in the home. Sleipnir alone,
  with no Bifröst, keeps downloading into one folder.
- **The admin sees everything.** The admin runs the home machine and can read the disk, so the
  apps do not pretend otherwise. Hiding a person's files from the admin would need Bifröst to
  encrypt them with a key only that person holds. That is not planned.
- **Why:** Sleipnir writes where Saga reads, so the two must agree on folders and on who may see
  them. One list in Bifröst means the path-safety rule and the visibility rule are each written once.
- **Cost:** Bifröst now holds a little more than identity. The line is that Bifröst knows *folders
  and people*, never what is inside a folder. Saga gets per-person visibility, which its spec had
  left as an open question.
- **Extends:** S4. Answers open question 2 of the Bifröst design.

## S12. Bifröst is a system service, and it asks the admin everywhere at once

- **Decision (2026-11-14):**
  - Bifröst runs as a system service under its own account, always on. Extensions run under it.
  - Signing in on the host means being logged in to the operating system as the admin's account.
    Bifröst has no password of its own. Host-only actions use the operating system's password prompt.
  - Changing the home (settings, extensions, the home key, linking online) is done on the host only.
    Answering a question (a download request, a pairing) can also be done from the admin's paired
    device, and a request can be answered from Huginn.
  - Bifröst has one "ask the admin" facility. It sends a question to the host screen, the admin's
    paired devices and Huginn at the same time. The first answer wins and the others are withdrawn.
  - A plain approve or reject needs no Norns. Norns adds steps, rules and history on top.
- **Why:** the home must answer while nobody is logged in. Sending everywhere avoids guessing where
  the admin is: the client app only reaches the home on the home Wi-Fi, and Huginn reaches the
  admin anywhere.
- **Cost:** installing needs administrator rights once. A small companion program must run in the
  admin's desktop session to show notifications, because a system service cannot. Files belong to
  the service account, and the admin's own account reads them through a group.
- **Replaces:** "one admin, signed in on the host only" in Saga's spec, and "no root to install" in
  Sleipnir's. Sleipnir alone still runs under the person's own account.

## S13. The client app is .NET for Android

- **Decision (2026-11-14):** the one Bifröst client app is built with .NET for Android, as a thin
  shell, without MAUI. It serves Android phones and Android TV.
- **Why:** it has full control over the web view's certificate check, which the pinned connection
  depends on. It keeps the LAN group in one language, and it is the only choice where the phone can
  later run Sleipnir's download engine.
- **Cost:** a larger app and a slower start than Kotlin. Bindings for Android's media library must
  be checked before the native video player is planned.
- **First test:** the web view enforces the pinned key for every request a screen makes.

## S14. Releases are signed by hand with an offline key

- **Decision (2026-11-14):** Bifröst installs a release only when its signature matches a public key
  built into Bifröst. The private key is kept offline, and the owner signs each release of Bifröst
  and of every extension. Bifröst carries two public keys: the one in use, and a backup kept
  somewhere separate.
- **Why:** a key stored in GitHub protects nothing when the GitHub account is taken over. The backup
  key removes the worst failure, a lost key that leaves every home unable to update.
- **Cost:** one manual step per release, per extension. The owner is the only signer.
- **Later, if needed:** an offline root key that approves a short-lived automatic key.

## S15. Every device reaches the home through a small app of ours

- **Decision (2026-11-15):** the family will use Android and iOS phones, Windows, Linux and macOS
  computers, Chrome, Edge, Firefox and Safari, and TVs. Not all at once, but the design must allow
  every one. The same rule holds on each: a device pairs with Bifröst once, **through a small app of
  ours**. The app pins the home's key, keeps the device token in the platform's key store, and shows
  the extensions' screens in the system web view.
- **Browser extensions never talk to the home directly.** They hand the request to our app on the
  same device, through the browser's native messaging. A Sleipnir extension on a laptop that is a
  Bifröst client sends the download to the home this way. With a standalone Sleipnir on that laptop,
  it can also offer "download here". On the host machine the desk companion plays the app's part,
  and reaches Bifröst through the admin socket.
- **Per platform:**

  | Device | Client | Browser extensions |
  |---|---|---|
  | Android phone, Android TV | .NET for Android thin shell (S13). TVs pair by the 6-digit code | — |
  | Windows, Linux, macOS | A small desktop client: pairs, pins, shows the screens in the system web view (WebView2, WebKitGTK, WKWebView) | Chrome, Edge and Firefox: one WebExtension codebase, through native messaging to the desktop client |
  | iPhone, iPad | .NET for iOS, the same thin shell | Safari: packaged inside the app, as Apple requires |
  | Safari on macOS | — | Packaged inside the macOS desktop client |
  | Apple TV | A tvOS app, later. Pairs by code | — |
  | Other TVs (web apps only) | They cannot pin a key, like a browser. View-only, or not supported | — |
  | A browser with no app of ours | The view-only fallback (S8) | — |

- **Order [proposed, owner to confirm]:** Android phone and TV, then the desktop client with the
  Chrome and Firefox extension, then iPhone, iPad and Safari, then Apple TV.
- **Settle now, so later platforms need no redesign:**
  1. One shared .NET client library: pairing, pinning, the token and the API types. The Android, iOS
     and desktop apps all use it. `Bifrost.Core` already has only framework references for this reason.
  2. The pinning test (README, next step 1) runs on every web view engine: Android WebView,
     WKWebView, WebView2 and WebKitGTK. It covers every request a screen makes, video segments and
     WebSockets included.
  3. One local protocol between a browser extension and our app, for every browser.
  4. Every screen stays web-based (Heimdall ADR 001), so no platform needs its own screens.
- **Why:** a browser extension cannot pin a self-signed key (Chrome cannot inspect certificates at
  all), and a browser on another device is view-only (S8). Our app already pins, so routing every
  extension through it solves pinning once, on every platform.
- **Cost:** one thin native app per platform family, and an Apple developer account for iOS and Safari.
- **Open, for later:**
  - Away from home no device reaches the home (S7, design §2). People will expect "send a download
    home from my phone at work". If that is wanted, it would be a request through Huginn that the
    home picks up, never an inbound connection.
  - The other route around pinning, a real HTTPS name for the home, stays parked (design §2).

---

## Target scenario

"Can I download this?" exercises every decision above. Build it last, in slices.

1. A paired family device asks Sleipnir for a video. Sleipnir holds it as pending.
2. Norns, if it is installed, opens a request and applies its rules. Without Norns, Sleipnir asks
   Bifröst for a plain approve or reject.
3. Bifröst asks the admin in three places at once: a notification on the host screen, the client
   app on the admin's phone, and a Huginn notice with Approve and Reject.
4. The first answer wins. An answer from Huginn is collected over the outbound link.
5. Bifröst names the folder: the requester's own. Sleipnir downloads into it on the home machine
   and announces "download finished" with that folder.
6. Saga scans that folder only. Bifröst sends a "ready" notice.
7. If the folder is public, every paired device in the home can stream it from Saga. If it is
   private, only the requester can.

With no internet, the request waits for an approval on the LAN.

## Where each app's docs record these decisions

Updated on 2026-11-14. None of these edits is committed in its repository yet.

| App | Doc | What it now says |
|---|---|---|
| Bifröst | `Bifrost/docs/design.md` | The full design. Every open question answered |
| Saga | `docs/superpowers/specs/2026-11-13-saga-design.md` | A Bifröst extension. No Heimdall, no Heimdall Lite. Viewers use the client app. Visibility by folder |
| Sleipnir | `docs/superpowers/specs/2026-11-13-sleipnir-service-design.md` | Two modes. Users, devices, pairing and the LAN listener moved to Bifröst. Folders from Bifröst. A pending state. The command becomes `sleipnir` |
| Sleipnir | `docs/superpowers/plans/2026-11-13-identity-and-access.md` | Marked as superseded in part |
| Norns | `README.md`, `docs/roadmap.md` | A "Direction" section: one codebase, home or online, storage not tied to SQL Server, two sign-in paths |
| Huginn | `docs/huginn/decisions.md`, `roadmap.md` | D15 (notices from linked homes) and D16 (one authority, LAN delivery later), with two epics |
| Heimdall | `docs/decisions/015-…md`, `README.md`, `CLAUDE.md` | DR-015: account features first in Phase 3, one shared tenant, a home is a public client |

Still stale: `docs/Asgard Map.html`, and Sleipnir's README, user guide and architecture doc, which
describe the users and devices that are built today and will change when the code does.
