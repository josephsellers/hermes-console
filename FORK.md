# Athena personal fork (not upstream)

Branch: `joseph/athena-bot-chat-glass`  
Base: Hermes Console **v1.2.10** (`3fd569f`) / gateway contract v7  
Package for Joseph: flavor **`qa`** → `dev.xpetalab.hermesconsole.qa`  
Daily Obtainium 1.2.9 (`dev.xpetalab.hermesconsole`) stays installed.

This is a private glass-swap fork for one Hermes on Athena. It is **not** a
pull-request candidate. Holes below are accepted. Anyone else would need a
real multi-surface contract from Nous.

## What it changes

Stock 1.2.10 opens Bot Chat through `connection.readOnly: true` (composer,
attach, and voice off; write contract deferred). This fork opens the same
pinned Bot Chat as a normal `ChatScreen` so the phone can send.

On Athena that send hitchhikes Desktop's live dashboard session (same
process, `session.resume` reuses the live id). Proven 2026-09-16 on `@tech`
with 1.2.9: photo from Open Chat landed in the Desktop Bot Chat; no
`SESSION_NOT_OWNED`.

1.2.10 already adopts in-flight turns on resume and can fan a second
WebSocket onto the live session. Live view of Desktop generating is that
machinery plus a writable Open Chat — not a new lock, redirect, or mailbox.

## How to run it

**Obtainium (same as the Open Chat QA sitting):** push this branch to
`josephsellers/hermes-console`. GitHub Actions builds flavor `qa` and
replaces pre-release tag `qa-open-chat`. Phone: Obtainium app source
`https://github.com/josephsellers/hermes-console`, **Include prereleases**
on, prefer `hermes-console-qa-open-chat-arm64-v8a.apk`. Package
`dev.xpetalab.hermesconsole.qa` updates in place. Leave official 1.2.9
installed.

Athena has no Flutter SDK. Laptop sideload is the fallback, not the path:

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
2. **Live view is best-effort.** Tokens still do not stream from Desktop
   (`_runTerminal` drops `message.delta`). 1.2.10 QA `@tech` 2026-09-16:
   user line + working spinner were live; the reply needed exit/resume
   because REST was not re-GETed after idle. This fork now fetches REST
   once when remote `working` ends so the durable reply should appear
   without leaving. Not a live token view.
3. **Share sheet still mints a sibling.** Gallery → Hermes Console is
   `source: android-share`, not the pinned Bot Chat. Use **Bots → Open Chat
   → attach** (the proven path).
4. **Desktop tab is the hitchhike host.** If that Bot Chat is not live in
   the dashboard process, the phone becomes the owner. Desktop may not see
   the turn until it resumes. Close-tab / reopen is fine.
5. **Pins are Athena disk state.** Git still strips `chat:`. New bots stay
   pinless until Desktop or CoS writes the pin. Appearance-only vs
   `chat: null` is stock 1.2.10 (#14). This fork does not add a uniqueness
   gate or pick-by-message-count.
6. **No Bot Screen on the phone.** VISION already rejected that.
7. **QA app is a second instance.** Separate pairing, drafts, and Bot Chat
   local pins. Do not treat it as an Obtainium update.
8. **Not robust for anyone else.** No exclusivity UX, no multi-writer
   idempotency, no share-sheet router, no tests of clash, no signed
   production overlay.

## Non-goals (still)

Official `bots.*` API, Telegram, staying on 1.2.9 as the strategy, auto-
picking `Bot Chat #2`, Console as a 2FA surface, patching `/opt/hermes`.
