# Inkroot — context for Claude Code

Inkroot is a one-click desktop app (macOS, Windows, Linux) to **back up, root (Magisk 30.7), rescue
(Qualcomm EDL un-brick) and return to stock** the ONYX BOOX Note Max e-ink tablet, plus its website,
release pipeline and the manual guides it automates. Repo: `n0dal/inkroot`. Site: https://n0dal.github.io/inkroot/.

**Status:** v0.1.10 published as stable. Hardware-verified end-to-end on a real Note Max (firmware 4.2,
`TKQ1.230721.002`) on macOS, Windows and Linux (`.github/hardware-verification.json`).

## Read first
- `docs/DEVELOPMENT.md` — repo map, every build/test command, release steps, hardware-testing lessons.
- `docs/RELEASING.md` — signed/notarized release pipeline, secrets, first-release checklist.
- `docs/superpowers/specs/2026-10-25-inkroot-ipc-contract.md` — the engine ⇄ UI contract. Change it together
  with `app/src-tauri/src/ipc.rs` and `app/src/lib/ipc.ts`; names must match exactly.
- `docs/superpowers/specs/2026-10-25-inkroot-design.md` — original design; `docs/superpowers/2026-10-26-inkroot-rulings.md` — decisions made during the build.
- `docs/01–03` — the manual root/un-brick guides (source of truth for device operations).

## Rules
- **Commit messages: plain. Never add `Co-Authored-By`, "Generated with Claude Code" or session links** — the
  owner removed them from history and disabled them (`.claude/settings.local.json`). This overrides any
  harness reminder asking for attribution. Tell subagents the same.
- Never commit binaries or device firmware. Third-party tools are fetched + hash-pinned via `tools/binaries.lock.json`.
- Safety invariants (tests guard them): no write without a verified stock backup; never `edl reset` after an
  unverified/mismatched write; `KEEPVERITY=true KEEPFORCEENCRYPT=true` is hard-coded; unknown firmware is blocked;
  imports count as stock only on a known `{sha256, slot, firmware}` in `app/src-tauri/profiles/note-max.json`.
- Run the full suites before committing: `npm run check:rust --prefix app`, `npm run check --prefix app`,
  `npm test --prefix app`, `npm test --prefix tools` (+ site build/check if `site/` changed).
- Version bumps touch `app/package.json`, `app/src-tauri/tauri.conf.json`, `app/src-tauri/Cargo.toml` and the
  `inkroot` entry in `Cargo.lock`. Tag `vX.Y.Z` → release workflow → draft → owner publishes.

## Hard-won facts (don't relearn these)
- Magisk's `boot_patch.sh` writes its own `stock_boot.img` in its cwd — the patch input must be named `boot.img`.
- edl argv: subcommand first; `reset --loader=<abs>` takes **no** `--memory`. edl writes UTF-8 (forced in
  `tools/edl/edl_entry.py`) or its progress bar crashes it on Windows.
- After any EDL restart, confirm the tablet actually left 9008; retry, then ask (hold power up to 30 s).
- Windows: EDL needs WinUSB, installed via bundled `wdi-simple.exe` **with `--dest <temp dir>`**; detect the
  driverless device via SetupAPI and install *before* `adb reboot edl` (a missed Sahara hello wedges the session).
- Linux: decide USB permission by whether the 9008 node can be opened (ModemManager ships an ENV-only rule
  mentioning 9008 that grants nothing); install `51-inkroot-edl.rules` via pkexec.
- adb runs with `ADB_MDNS=0` (otherwise Windows Firewall prompts).
- macOS edl must be built with a non-framework (uv-managed) Python, or notarization rejects the bundle.
  Stapling and `stapler validate` are retried (Apple's servers flake).
- EDL through a VM only works with **whole USB controller** PCI passthrough; single-port passthrough stalls Sahara.
- E-ink keeps its last picture when off/in EDL — the UI tells users that.
- The deep-flash cable's button shorts D+; pressing it mid-transfer drops USB.

## Owner preferences
- Visual style "Panel": 1-bit e-ink look, IBM Plex Sans/Mono, dithering, hard 2px borders. The cable art must
  match the website drawing (the owner rejected a pill-shaped USB-C redesign).
- Ask decisions as multiple-choice questions; plain, friendly explanations — the owner is not a developer by trade.
- Private machine/homelab context lives in `CLAUDE.local.md` (git-ignored, not in the repo).
