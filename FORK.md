# Athena personal fork (not upstream)

Branch: `joseph/qa-pr-34-35-38-40`  
Base: upstream `main` (after v1.2.10) plus [#34](https://github.com/xP3ta/hermes-console/pull/34),
[#35](https://github.com/xP3ta/hermes-console/pull/35),
[#38](https://github.com/xP3ta/hermes-console/pull/38),
[#40](https://github.com/xP3ta/hermes-console/pull/40).  
Phone: flavor **`qa`** → `dev.xpetalab.hermesconsole.qa` (Obtainium
`qa-open-chat`). Current APK is **`a8247a9`** on this branch. Rollback:
`joseph/athena-bot-chat-glass` (v1.2.10 writable Open Chat + REST-on-idle).
Official Obtainium **1.2.9** stays installed. Do **not** take stock 1.2.10
until #34 ships. Keep this QA APK until Console **1.2.11** includes #38
and #40.

This is a private glass-swap fork for one Hermes on Athena. #34/#35/#38/#40
are the upstream-shaped pieces; the glass branch is not a PR candidate.

## What the stacked APK adds over stock 1.2.10

- **#34** — Bot Chat Open Chat is a writable `ChatScreen`, not
  `buildReadOnlyBotChatDestination`.
- **#35** — resume hidden Bot Chat via `canonical_session` when Desktop
  has not written `ui_meta.hermes-bots.chat`.
- **#38** — while watching another surface, GET REST once more when that
  turn goes idle (durable assistant row). Tokens do not stream.
- **#40** — when this phone owns the turn and the dashboard WebSocket
  drops (`1006`), GET the durable transcript instead of waiting on
  `session.resume` (official gateway has no `turn_idempotency_v1`). Skip
  the 3s full Bot Chat GET while the composer has a draft unless another
  surface owns the live turn.

On Athena, a phone send hitchhikes Desktop's live dashboard session when
Desktop is up (same process, `session.resume` reuses the live id). If
Desktop is quit, the phone is the owner.

Verified 2026-09-16 on `@tech` (#34/#35/#38). #40 is the 2026-09-18
slow-path follow-on (laptop closed, Tailscale → hecate, `ws write slow`
+ `1006`; turns completed on Athena; force-quit REST showed the reply).
Physical pass of this APK is Joseph’s sitting.

## How to run it

Push this branch to `josephsellers/hermes-console`. GitHub Actions builds
flavor `qa`, signs with a **fork-persistent QA key** (repo secrets
`QA_KEYSTORE_*`, not the ephemeral GHA debug cert), and replaces
pre-release tag `qa-open-chat`.

Phone: Obtainium source `https://github.com/josephsellers/hermes-console`,
**Include prereleases** on, prefer
`hermes-console-qa-open-chat-arm64-v8a.apk`. Leave official 1.2.9
installed (title filter `v1\.2\.9`).

Persistent-key installs overlay (`8764000`, then `7a5e368`, then
`a8247a9`). If the signer ever changes again: uninstall Hermes Console
QA, then install once (pairing is lost).

Athena has no Flutter SDK. Laptop sideload is the fallback:

```bash
flutter build apk --profile --flavor qa --split-per-abi \
  --dart-define=HERMES_FLAVOR=qa
```

Open Chat needs live `ui_meta.hermes-bots.chat` pins (Athena already has
them). 1.2.10 Open Chat reads `/p/<profile>/` on `:18794`; multiplex is on.

## Accepted holes

These are known. Joseph will navigate around them. Do not “fix” them in
this fork unless he names that sitting.

1. **Not two live writers.** Sequential glasses. Mid-turn clash is undefined
   (hitchhike, queue, or `SESSION_NOT_OWNED`). No composer lock, no
   `session.redirect`, no takeover, no `bot_live_delivery` for photos.
2. **Not a token stream.** Assistant text appears after the turn is
   durable (REST on idle, or REST after a socket drop), not
   token-by-token. Good enough for this glass-swap; not dual live chat.
3. **Share sheet still mints a sibling.** Gallery → Hermes Console is
   `source: android-share`, not the pinned Bot Chat. Use **Bots → Open Chat
   → attach** (the proven path).
4. **Desktop live runtime is the hitchhike host.** Bot Chat is one pane
   now, but the dashboard still holds a live session for every bot opened
   this sitting. Switching the pane does not drop leases. Quit Desktop or
   disconnect the instance to test pinless discovery. If nothing is live,
   the phone becomes the owner.
5. **Pins are Athena disk state.** Git still strips `chat:`. Keep live
   `chat:` on disk. #35 (`canonical_session`) resumed `@tech` pinless on
   the stacked APK (2026-09-16) and wrote the pin back on first prompt.
   `chat: null` stays a real reset. Do not pick by message count.
6. **No Bot Screen on the phone.** VISION already rejected that.
7. **QA app is a second instance.** Separate pairing, drafts, and Bot Chat
   local pins. Do not treat it as an Obtainium update of official 1.2.9.
8. **Not robust for anyone else.** No exclusivity UX, no multi-writer
   idempotency, no share-sheet router, no tests of clash, no signed
   production overlay.

## Non-goals (still)

Official `bots.*` API, Telegram, staying on 1.2.9 as the strategy, auto-
picking `Bot Chat #2`, Console as a 2FA surface, patching `/opt/hermes`.
