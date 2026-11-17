# Issues to create

Proposed on 2026-11-15, not yet on GitHub. Create them by the standard in [`issues.md`](issues.md),
then delete this file. Each line is one issue: **title** — type label.

## Bifröst (`Parth2500/Bifrost`)

Milestones are the design's order of work (`Bifrost/docs/design.md` §12), plus `Later`.

**Step 2: Host an extension**
- Run extensions and route `/x/<name>` — feature (built)
- System service with the admin socket — feature (plan 2c)
- Ask the admin, with the desk companion — feature (plan 2d)

**Step 3: LAN**
- Home key and HTTPS on the home network — security (built)
- Pairing by QR code — feature (link and redeem built; the QR image is a screen's job)
- Pairing by a compared 8-digit code — feature (built, commit then reveal)
- Discovery with mDNS — feature (announcing is built through Avahi; browsing from a client is step 4)
- View-only browser fallback — feature (built, off by default)
- polkit prompt for host-only actions — security (built for `identity rotate`; other host-only actions come with their features)
- Audit log for pairing path B — security

**Step 4: Client app**
- Pinning test on every web view engine — testing (S15: Android WebView, WKWebView, WebView2, WebKitGTK)
- Shared .NET client library — feature (built: pairing and the pinned HTTP client; `BifrostClient` reads me, extensions and an extension's address)
- Android app, first cut — feature

**Step 5: Install and update**
- Signed releases with offline keys — security (S14)
- Install, update and roll back extensions — feature
- Bifröst updates itself — feature

**Step 6: Folders and events**
- Home root and per-person folders — feature
- Local events — feature

**Step 7: Online link**
- Link the home to Heimdall — feature
- Notices to Huginn, with answers — feature

**Later**
- Desktop client and browser extensions — feature, epic (S15)
- Apple devices: iPhone, iPad, Safari, Apple TV — feature, epic (S15)
- Huginn courier between devices — feature (S9)

## Sleipnir (`Parth2500/Sleipnir`)

**Under Bifröst**
- Pending downloads ask the admin — feature (blocked by Bifröst "Ask the admin")
- Folders from Bifröst — feature (blocked by Bifröst step 6)
- `sleipnir.download.finished` event — feature (blocked by Bifröst step 6)
- Remove a person's downloads — feature
- Home root unavailable, free space, NTFS file names — feature

**Standalone backend**
- Settings file — feature
- Live events over WebSocket — feature
- OpenAPI document with a drift test — testing
- Download records in SQLite — feature

**Phase 3: TUI as a client**
- `Sleipnir.Client` library — feature
- TUI as a client of the daemon — feature

**Later**
- `sleipnir add` and other commands — feature
- Scheduling — feature
- Windows and macOS services — feature
- Browser extension — feature (S15)
- Engine on Android inside the Bifröst app — feature (S13)

Also: rewrite Sleipnir #1 "Roadmap" to point to the milestones and drop what moved to Bifröst.
