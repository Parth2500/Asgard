# Asgard suite review

*2026-11-13. Written from the suite docs, each app's design docs, and spot checks of the code.
Where a claim comes from code and not a doc, it says so. Nothing was built or changed.*

## 1. The short version

The suite has two halves built on opposite assumptions, and the hard problems all sit on the line
between them.

| | Account half | Device half |
|---|---|---|
| Apps | Heimdall, Huginn, Norns | Sleipnir, Bragi (Roundelay), Saga |
| "Who are you?" | A user account at an issuer URL | A device that was paired with this home |
| Trust comes from | A public HTTPS certificate on a DNS name | A QR code carrying a key fingerprint |
| Runs | In a browser, any network | Native app or host machine, home Wi-Fi |
| Storage | PostgreSQL / SQL Server, multi-tenant | SQLite, single home |
| Built for | Many tenants, scale-out | One family, one machine |

Three findings matter more than the rest:

1. **The keystone is HTTPS on the home network, not Heimdall's size.** Every browser-based piece
   (Heimdall's login page, Saga's web UI, Huginn, Tafl as a PWA) needs a certificate a browser
   trusts. There is no way to get one for a LAN address without either a real domain or installing
   a certificate on every device. The device half avoids this with fingerprint pinning, which
   browsers cannot do. Until this is decided, Heimdall Lite, Saga login and Bifröst cannot be
   designed.
2. **Suite-level decisions are being made inside single-app specs, and they now contradict each
   other.** Saga's spec says every app requires Heimdall at release. The README says integration is
   never required. Sleipnir's spec says Heimdall is optional and later. All three were written on
   the same day.
3. **Several planned links rest on things that the target app has ruled out or does not have.**
   Huginn excludes bots and will encrypt end to end. Bragi shares a device's system audio and has no
   Linux host. Tafl has no multiplayer. Details in section 5.

## 2. What each app is today

| App | Real state (checked) | Home fit |
|---|---|---|
| **Heimdall** | 146 commits. Phases 0–2 done (OIDC core, DPoP, PAR, logout, conformance). Phase 3 just started (account picker). PostgreSQL only. Refuses plain HTTP. Finds the tenant from the host name. No sign-up, no admin portal, no password reset, no device grant. Users are identified by email. No LICENSE file. | Strong engine, missing everything a household needs to *use* it |
| **Sleipnir** | 36 commits. Engine solid. Daemon with loopback REST, users, roles, devices and hashed tokens built today. Not built: LAN listener, TLS identity, mDNS, pairing, WebSocket events (none of these appear in code). | Best fit. Its design is the model for the device half |
| **Saga** | 0 commits. One design spec. GPL-2.0. | Design only. Its release is gated on two undesigned pieces (below) |
| **Huginn** | 53 commits. Phase 0 done, Phase 1 in progress. Heimdall sign-in works beside passwords. PRODUCT.md still calls it a portfolio demo. SCALING.md targets a million users. Needs PostgreSQL and one HTTPS origin. | Built for the internet, asked to serve a home |
| **Norns** | 4 commits, on hold since January. .NET 9, SQL Server. Two endpoints. Code is still named CoreOps. | Weakest fit (section 5.4) |
| **Bragi** | 35 commits, v0.2.0, not your repo. Noise XX pairing over TCP, per-guest keys, UDP audio. Hosts: Windows and Android only. Pairings are not remembered. | Works, but only for the job it has now |
| **Tafl** | 144 commits, v5.0.0, not your repo. Offline single-player games. Optional accounts on a PHP/MySQL cPanel host. No multiplayer code. | Standalone; suite links are all new work |
| **Ratatoskr** | A handful of files. **Not a git repository.** Still named TuiFileExplorer inside. | No suite role yet |

The Asgard folder itself is not a git repository either, so the README, the vision file and the map
have no history.

## 3. The dependency chain nobody has drawn

```
Saga release
  └─ requires Heimdall login              (Saga spec, owner decision)
       └─ requires Heimdall Lite          (not designed; Heimdall is mid Phase 3)
            ├─ single-tenant mode         (touches decisions D15–D21, DR-006)
            ├─ a way to create family accounts   (no sign-up, no admin UI, no reset today)
            └─ an https issuer on a DNS name     (Heimdall refuses plain HTTP)
                 └─ Bifröst               (not designed; no owner project)
                      └─ a browser-trusted certificate on the LAN   (no decision)
```

Every arrow is an unbuilt piece. The same chain sits under Huginn at home, and under Sleipnir if it
adopts the "Heimdall required at release" rule.

## 4. The three core problems

### 4.1 Trust on the LAN

There are only three ways for a device to trust a home server:

| Method | Works for | Cost |
|---|---|---|
| **Pin a key from a QR code** | Native apps (Sleipnir clients, Bragi, Android apps) | None. No domain, no internet. Sleipnir and Bragi both do this |
| **Real domain, certificate by DNS challenge** (`*.home.example.net` pointing at a LAN address) | Everything, including TV browsers | You own a domain. Renewal needs the internet every few weeks. Some routers block public names that resolve to private addresses |
| **Private certificate authority installed on each device** | Laptops, phone browsers | Manual on every device. Most smart TVs cannot do it. Android apps ignore user-installed authorities by default |

Plain HTTP is the fourth, degraded option: it rules out OIDC (Heimdall's code refuses non-loopback
`http` redirect URIs), secure cookies and service workers. Saga's video playback would still work.

Saga's main screens are TVs, and a TV browser can only use the second row. So the honest choice is:
buy into a real domain for browser clients, or accept that browser clients away from the host get a
no-accounts viewer mode and that signed-in use needs a native app.

### 4.2 Heimdall at home ("the Olympics for a school event")

The weight is not where the docs put it. Saga's spec says Heimdall Lite exists to avoid PostgreSQL.
But:

- PostgreSQL idles small. Huginn needs it anyway, so a home with Huginn already runs it.
- A SQLite provider means a second implementation of Heimdall's most sensitive code. Sixteen
  infrastructure files use PostgreSQL features, nine of them raw SQL (the atomic code-redemption and
  refresh-reuse statements). Each needs a twin, its own tests and its own conformance run.
- Single-tenant is nearly free already: one tenant row and one host name.

What actually blocks a family from using Heimdall is missing from both Lite and full Heimdall:

- No way to create a user except `seed-dev`.
- Users are emails. A child has none.
- No password reset, and a home has no mail server to send one.
- No device grant, so signing in on a TV means typing a password with a remote.
- The HTTPS issuer (4.1).

All of these are also Phase 3 items for cloud Heimdall. Building them first serves both modes.
The SQLite port serves only one, and it is the expensive one.

There are also already **four** places that hold users: Heimdall, Sleipnir's `IdentityService`,
Huginn's password accounts, and Tafl's PHP backend. Sleipnir's is in effect a working "identity
lite" for devices on a LAN.

### 4.3 Three pairing protocols

| | Sleipnir (designed) | Bragi (built) | Saga |
|---|---|---|---|
| Link | `dnm://pair?…` | `mpsp://pair?h=…&fp=…&t=…` | none |
| Channel | HTTPS, pinned SPKI fingerprint | Noise XX over TCP, then UDP | OIDC |
| Credential | 256-bit bearer token per device | Per-guest ChaCha20 key | Session from Heimdall |
| Remembered | Yes, revocable | No, re-scan each time | n/a |
| Discovery | mDNS `_dnm._tcp` + QR | QR only (it can carry Wi-Fi details) | Bifröst, later |

These do not need to become one protocol. Audio over UDP, a REST API and HLS video are different
transports for good reasons, and Bragi is not yours to change. What they share is one ceremony:
*scan a code, check a fingerprint, get a credential*. That ceremony is the thing worth owning once.
Huginn's planned device linking (milestone 2.4) is the same ceremony again.

## 5. Issues waiting further out

### 5.1 Huginn as the family's notification channel
- Decision D10 puts **bots out of scope**. The planned first link is a bot.
- Phase 2 encrypts every chat with MLS. A server webhook cannot post a readable message into an
  encrypted group. The sender has to be a real MLS member with its own keys, which means a headless
  client service. A webhook built now is thrown away at Phase 2.
- Notifications to a closed app use Web Push, which goes through Google, Mozilla or Apple. It does
  not work without the internet.
- Where Huginn runs is undecided ("Hosting: not decided"). At home, a phone away from home gets
  nothing. In the cloud, Norns or Huginn cannot call *into* the home, so the home must hold an
  outbound connection.

### 5.2 Bragi for movie night
Bragi's host shares **the system audio of the device it runs on**, and a Linux host is not
implemented. The home server is Fedora, and Saga does not play audio: it serves files to a browser.
A TV browser cannot be a Bragi host. Guests also play 160 ms late by design, so the video must be
delayed to match. "Saga plus Bragi" therefore needs a new Bragi mode (audio from a stream, clock
shared with a web player). It is a feature in someone else's core, not a link.

### 5.3 Tafl for games night
No multiplayer exists. Tafl's promise is "no accounts, no sign-in". As a PWA it needs HTTPS to
install from the home server (4.1 again). LAN multiplayer, Heimdall sign-in and Huginn invites are
three new features in a repo you contribute to but do not steer.

### 5.4 Norns
The one scenario that uses it ("can I download this?") needs a pending state and an approve button.
Sleipnir's design already leaves room for both. Norns brings SQL Server (heavy, and x86-64 only as
far as I know, which rules out an ARM home server), .NET 9 and a multi-tenant model for one
approval. It is a fine portfolio project. It does not earn a place in the home install.

### 5.5 Spread
- **Databases:** PostgreSQL twice (17 and 18), SQL Server, SQLite twice, MySQL.
- **Runtimes:** .NET 10, .NET 9, Node 24, Rust, Flutter, PHP.
- **Android:** .NET (Sleipnir), Flutter (Bragi), Capacitor (Tafl), native (Huginn epic), undecided (Saga).
- **Licenses:** Saga GPL-2.0, Sleipnir MIT, Bragi Apache-2.0, Heimdall none. Separate processes
  talking over HTTP are fine. Shared *code* is not automatically fine: Apache-2.0 code cannot be
  linked into a GPL-2.0-only program, and Saga's spec already proposes sharing the path-safety rule.
- **Admin models:** Saga allows the admin on the host only. Sleipnir allows paired admin devices.
  Heimdall plans a web admin portal.
- **Service model:** Sleipnir is a per-user daemon that needs no root. Bifröst as described owns
  mDNS, TLS and routing for the machine, which points at a system service. "Each app installs it if
  missing" also means several installers can each try to own its upgrades.

### 5.6 Smaller traps
- **Issuer change.** Apps key users by `(issuer, subject)`. Moving a family from a home Heimdall to
  a cloud tenant changes the issuer and orphans every account unless a migration is planned.
- **Bifröst has two jobs under one name:** a gateway into the home from outside (vision file) and
  the LAN name, certificate and discovery service (Saga spec). They have different threat models.
- **Doc drift.** The README gives streaming to Jellyfin and does not list Saga. Sleipnir's spec
  does the same. The map says seven apps. Huginn and Norns still link `heimdall-idp`. Sleipnir
  still ships `dnm`, `_dnm._tcp` and `dnm://`.

## 6. What I would do

In order. Each step is small and none commits the suite to anything large.

1. **Give the suite its own decision log.** Make `Asgard` a small git repo for suite docs only (not
   a monorepo), and move the suite-level decisions out of Saga's spec into it. Put Ratatoskr under
   git.
2. **Settle the contradiction on "standalone".** My recommendation: keep device pairing as a
   permanent sign-in mode in the device half, and make Heimdall an upgrade that maps onto existing
   users by `(issuer, subject)`. That is Sleipnir's design, and Huginn's `AuthProvider` seam already
   works this way. "Heimdall required at release" puts sections 3 and 4 on every app's critical
   path.
3. **Decide LAN trust (4.1) before designing Heimdall Lite or Bifröst.** Everything else follows
   from it.
4. **Define the shared pairing ceremony as a document, not a service:** one QR payload shape, one
   fingerprint rule, one mDNS record. Sleipnir implements it first, as its LAN mode. Extract
   Bifröst only when a second app needs it.
5. **Reorder Heimdall Lite.** First: first-run setup, admin-created users without email,
   admin-driven reset, and a device grant with a QR code for TVs. Run it at home on PostgreSQL to
   prove it. Add the SQLite provider last, and only if PostgreSQL turns out to be the real pain.
6. **Make the first link test the hard part.** Sleipnir → Huginn proves a webhook, which was never
   in doubt. A better first proof: one phone on the home Wi-Fi signs in to one app through a
   Heimdall running on the home machine. If that works end to end, the rest of the suite is
   plumbing.
7. **Adopt one rule for home ↔ cloud: the home always dials out.** No app outside the house ever
   needs to reach in. That keeps remote access a separate, later decision.
8. **Park honestly.** Norns, the Bragi watch party and Tafl multiplayer stay as ideas, with the
   blockers from section 5 written beside them.

## 7. Decisions that are yours

| # | Question | Why it blocks |
|---|---|---|
| 1 | Is Heimdall required at release, or an optional upgrade? | Sets the critical path for Saga, Sleipnir and Huginn |
| 2 | Real domain for the home, or pinned native apps plus a plain-HTTP viewer mode? | Sets what Bifröst and Heimdall Lite are |
| 3 | Does Huginn run at home, in the cloud, or both? | Sets how events and notifications travel |
| 4 | Is Asgard for your family only, or installable by others? | Decides how much installer, packaging and multi-OS work is justified |
| 5 | Is Norns part of the home install? | Removes or keeps SQL Server and a sixth runtime |
