# Athena personal fork (not upstream)

Branch: `joseph/athena-bot-chat-glass`  
Base: Hermes Console **v1.2.10** (`3fd569f`) / gateway contract v7  
Phone: flavor **`qa`** → `dev.xpetalab.hermesconsole.qa` (Obtainium
`qa-open-chat`, currently `1.2.10+9012` / `8764000`)  
Official Obtainium **1.2.9** (`dev.xpetalab.hermesconsole`) stays installed.
Do **not** take stock 1.2.10 — Bot Open Chat is still a viewer there.

This is a private glass-swap fork for one Hermes on Athena. It is **not** a
pull-request candidate. Holes below are accepted. Anyone else would need a
real multi-surface contract from Nous.

## What it changes

Stock 1.2.10 opens Bot Chat through `connection.readOnly: true`. This fork
opens the same pinned Bot Chat as a writable `ChatScreen`.

On Athena, a phone send hitchhikes Desktop's live dashboard session (same
process, `session.resume` reuses the live id). Writer identity is
`(pid, live_session_id)` — no `SESSION_NOT_OWNED`. Photo attach +
`prompt.submit` are Desktop's turn as far as the agent is concerned.

While Open Chat is on screen, Console polls REST during remote `working`
(user line + spinner) and **GETs once more when that turn goes idle**
(durable assistant reply). Tokens do not stream (`_runTerminal` drops
`message.delta`). Verified 2026-09-16 on `@tech`: Desktop chat visible on
the phone, including replies, without leaving the chat.

## How to run it

Push this branch to `josephsellers/hermes-console`. GitHub Actions builds
flavor `qa`, signs with a **fork-persistent QA key** (repo secrets
`QA_KEYSTORE_*`, not the ephemeral GHA debug cert), and replaces
pre-release tag `qa-open-chat`.

Phone: Obtainium source `https://github.com/josephsellers/hermes-console`,
**Include prereleases** on, prefer
`hermes-console-qa-open-chat-arm64-v8a.apk`. Leave official 1.2.9
installed (title filter `v1\.2\.9`).

A GHA debug cert cannot overlay another GHA debug cert
(`failureConflict` / signing-certificate mismatch). The current phone
install is the persistent-key build (`8764000`). Later updates overlay.
If the signer ever changes again: uninstall Hermes Console QA, then
install once (pairing is lost).

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
2. **Not a token stream.** Desktop's assistant text appears after the turn
   is durable (REST on idle), not token-by-token. Good enough for this
   glass-swap; not dual live chat.
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
   local pins. Do not treat it as an Obtainium update of official 1.2.9.
8. **Not robust for anyone else.** No exclusivity UX, no multi-writer
   idempotency, no share-sheet router, no tests of clash, no signed
   production overlay.

## Non-goals (still)

Official `bots.*` API, Telegram, staying on 1.2.9 as the strategy, auto-
picking `Bot Chat #2`, Console as a 2FA surface, patching `/opt/hermes`.
