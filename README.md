# Fury Audio Studio

PipeWire voice DSP + GTK3 control window: microphone/speaker RNNoise noise
suppression, 9-band parametric EQ, 16-band robot vocoder, pitch shifter,
autotune, bitcrusher, delay, reverb, and a preset palette (Natural Clean,
Daft Punk, Darth Vader, Chipmunk, Cylon Robot, Kraftwerk, Matrix Agent…).

When ON, apps record from the `Fury Mic` virtual source and hear through the
`Fury Speakers` virtual sink. When OFF, hardware routes directly again and
previous defaults are restored.

## Origin

Fork of **Dusky Audio Studio** by dusklinux (`dusklinux/dusky`,
`user_scripts/audio/dusky_audio_studio/`), snapshotted at upstream commit
`522cb09c4a1a578c3e1fd98b377bd8ad44371fdb` (2026-10-10) — see commit 1, which
is the pristine snapshot. Commit 2 is the Fury rename; everything after is
independent development. Upstream changes are ported by hand, never merged
(the full rename makes merges meaningless). Thank you to the Dusky author
for the DSP work.

## Layout

- `fury_audio_studio.py` — control window (GTK3), CLI (`--gui`, `--toggle`,
  `--on/--off`, `--status`, `--preset`, …), and the daemon server that
  drives the engine over a line-based stdin protocol with binary telemetry
  frames on stdout (protocol v4, see `audio-helper/protocol.h`).
- `audio-helper/` — native DSP engine (`main.c`, `protocol.h`, `Makefile`).
  Builds `fury_audio_dsp`. PipeWire virtual mic/sink + per-direction
  RNNoise, EQ, vocoder, pitch FX, delay, reverb at 48 kHz.

## Dependencies

| | Nix (`pkgs.*`) | Arch |
|---|---|---|
| C build | `gcc`, `gnumake`, `pkg-config`, `pipewire.dev`, `rnnoise` | `gcc`, `make`, `pkgconf`, `pipewire`, `rnnoise` |
| Python GUI | `python3` + `pygobject3`, `pycairo`, `gtk3` | `python`, `python-gobject`, `python-cairo`, `gtk3` |
| Runtime CLIs | `pipewire` (`pw-dump`, `pw-cli`, `pw-metadata`), `wireplumber` (`wpctl`), `libnotify` (`notify-send`) | `pipewire`, `wireplumber`, `libnotify` |

No other dependencies: the Python side is stdlib + GTK3; the C side is
libc/libm/pthreads + PipeWire/SPA + RNNoise.

Build the engine anywhere with:

```sh
make -C audio-helper        # produces audio-helper/fury_audio_dsp
```

`HOST_CFLAGS` and `USE_MOLD` are opt-in (`?=`, default empty/off) so the
same Makefile works on Arch and in sandboxed/pinned builds. Set
`FURY_AUDIO_DSP_BIN` to point the frontend at a prebuilt engine (used by
the NixOS module, where the store is read-only); otherwise it builds
`./audio-helper` in place on first run.

State lives in `~/.config/fury/studio/` (`config.json`, socket, pid files),
cache in `~/.cache/fury-studio/`. GTK theming follows the system theme
(`@theme_*` variables) — no hardcoded palette.

## License

See `LICENSE` (upstream MIT terms, kept verbatim) and `NOTICE` (attribution
+ the one known header discrepancy: `main.c` claims GPL-3.0 while upstream
advertises MIT — treated as GPL-3.0-or-compatible for the engine until
upstream clarifies).
