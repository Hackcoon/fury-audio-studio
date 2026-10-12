# Fury Audio Studio — 6-Week Upstream Update Plan

> Window: 2026-10-12 → 2026-11-23. Purpose: track Dusky Audio Studio
> upstream, port what's worth porting, freeze cleanly at the end.
> Upstream moves fast and the owner has a wave of improvements landing in
> this window — this doc is the routine so nothing valuable slips by and
> nothing half-understood gets merged.
> Rule #0 (non-negotiable): **never merge upstream, always hand-port.**
> The full rename makes merges meaningless and dangerous.

## Baseline (week 0, 2026-10-12)

- Upstream base: `dusklinux/dusky@522cb09` (`user_scripts/audio/dusky_audio_studio/`)
- This repo: pristine snapshot (commit 1) + Fury fork (commit 2)
- Protocol: **v4** (C `FURY_PROTOCOL_VERSION` == Python `PROTOCOL_VERSION` == 4)
- PipeWire nodes: `fury-audio`, `fury-audio-sink`, `fury-audio-capture`,
  `fury-audio-sink-out`, (`fury-audio-monitor` on demand)
- NixOS module: `services.fury-audio-studio`, v0.2.0, live-tested ON/OFF cycle

## Weekly routine (Sundays, ~15 min)

1. Fetch upstream read-only (never checkout, never merge):
   ```sh
   git ls-remote https://github.com/dusklinux/dusky.git main
   ```
   If HEAD moved, pull the range for OUR paths only:
   ```sh
   git clone --depth 50 --filter=blob:none --sparse https://github.com/dusklinux/dusky.git /tmp/dusky-watch
   git -C /tmp/dusky-watch sparse-checkout set --no-cone user_scripts/audio/dusky_audio_studio
   git -C /tmp/dusky-watch log --oneline <last-checked>..HEAD -- user_scripts/audio/dusky_audio_studio/
   ```
2. Classify every commit: **PORT** / **SKIP** / **WATCH** (rubric below).
3. Log it in the tracking table (bottom of this doc) — even "no changes".
4. Port the PORT items immediately (procedure below) or not at all; no
   "port later" pile.

## Triage rubric

**PORT:** DSP bugfixes (denoise quality, latency, xruns, crackle), new FX /
presets / EQ bands, PipeWire/WirePlumber compat fixes, crash/hang fixes,
CPU/RAM reductions, telemetry accuracy.
**SKIP:** Dusky branding, Arch/pacman paths, `~/user_scripts` assumptions,
Dusky-app integrations (rofi/quickpanel hooks), version-gate bumps that
don't change behavior (e.g. "Python 3.15 required" with no code need).
**WATCH:** protocol version bumps, config-file format changes, new IPC
commands, node-name changes — these need coordinated C+Python moves
(see playbook).

## Port procedure (per accepted commit)

1. `git show <upstream-sha> -- <paths>` — read the full diff, understand it.
2. Hand-apply to the Fury files, translating names (`dusky→fury`,
   `ghelper-audio→fury-audio`, `GHA_→FURY_`) as you go. No patch files from
   upstream (paths/names differ — they'd fail anyway).
3. `python3 -m py_compile fury_audio_studio.py`
4. Engine check: `nix-shell -p gcc gnumake pkg-config pipewire.dev rnnoise
   --run "make -C /tmp/fury-build-test"` on a copy (proves the C still
   compiles; the Nix module re-verifies anyway).
5. Commit here with message `Port upstream <short-sha>: <what+why>`
   (keeps provenance greppable: `git log --grep=Port`).
6. Push, then bump `furyRev`+`hash` in the NixOS module, `nixos-rebuild
   build`, run the test matrix, push `fury-nixOS`.
7. Update the tracking table below.

## Test matrix (every port, no exceptions)

- [ ] `fury-audio-studio --help` / `--status` (OFF baseline)
- [ ] `--toggle` ON: `pw-dump` shows all 4 `fury-audio*` nodes + defaults set
- [ ] `--status` shows ON + live VAD/telemetry numbers
- [ ] GUI opens, meters move, footer reads "Fury Mic & Fury Audio"
- [ ] `--toggle` OFF: nodes gone, hardware defaults byte-identical restore
- [ ] No stray processes (`pgrep -f fury_audio_studio` empty)

## Protocol-bump playbook (if upstream leaves v4)

1. Change C (`FURY_PROTOCOL_VERSION`, frame struct) AND Python
   (`PROTOCOL_VERSION`, `HEADER_STRUCT`, telemetry reader) in ONE commit —
   a half-moved pair fails the module's `--protocol-version | grep -qx`
   build gate by design. That gate is the alarm; never bypass it.
2. Old saved configs in `~/.config/fury/studio/` must still load (add a
   migration, don't wipe user settings).
3. Full test matrix + a DSP ON soak (10 min call or recording) before push.

## Week-6 freeze decision (2026-11-23)

Stop tracking. Options: (a) declare divergence — upstream becomes
reference-only, all future work is ours; (b) extend 2 weeks with cause.
Aligns with the Fury distro freeze week (`fury-dusky-plan.md` §0) so the
Arch PKGBUILD and the Nix module freeze on the same source.

## Tracking log

| Date | Upstream range checked | Verdict | Ported (fury commits) |
|---|---|---|---|
| 2026-10-12 | `522cb09` (baseline) | — | base snapshot + fork |
| | | | |
| | | | |
| | | | |
| | | | |
| | | | |
| 2026-11-23 | | FREEZE DECISION | |
