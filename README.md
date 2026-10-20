# Crop mood EOS M slim GUI, with autofocus

Magic Lantern for the **Canon EOS M (firmware 2.0.2)**. It keeps the slim, simple
menu of the [Crop mood EOS M slim GUI](https://github.com/Amit199167/Crop-mood-eosm-slim-gui)
and adds **working autofocus under crop presets**, which the camera normally can't do.

> **Experimental.** It's field-tested on one camera with a few lenses. Magic Lantern runs
> at your own risk, so keep a backup card with a build you trust.

## Why

On the EOS M, Canon switches off Movie Servo AF in every magnified view, and crop presets
need the x5 view. So with a crop preset you normally get no autofocus at all, and Magic
Lantern's own lens commands don't move EF-M lenses on this camera.

This build gets around that in two ways:

1. It uses **Canon's AF at the right moment.** It zooms to x10, where Canon's AF is
   accurate, focuses once, then returns to the preset.
2. **Magic Lantern drives the lens itself.** It sends focus pulses through the camera's
   lens processor (the MPU) and runs its own contrast autofocus at x5, without leaving the
   preset view. That includes during RAW recording.

## Features

All the new settings are in the slim menu.

| Setting | What it does |
|---|---|
| **Shutter zoom → AF** | Half-press: zoom to x10, focus once, return to the preset view. With a manual lens it acts like Shutter zoom ON. |
| **Auto refocus** (OFF / ON / ON + REC) | Watches the picture's sharpness. When the picture goes soft and stays soft, it refocuses. ON + REC also refocuses while recording. This is refocus-on-blur, not smooth tracking. |
| **Refocus method → MPU drive** | Magic Lantern's own contrast AF at x5, by driving the lens directly: a quick local probe first, then a search in both directions if needed. |
| **Auto lens profile** | Learned automatically when you swap lenses or zoom (never while recording). Stored in `ML/SETTINGS/AFLENS.TXT`, with a built-in table for known lenses. Also available: *Profile this lens now* and *Forget lens profiles*. |
| **Focus and Modules tiles** | Added to the slim menu grid. |
| **ISO / shutter / aperture shortcuts while recording** | Fixed. They were always blocked before. |

### Tested lenses

| Lens | Result |
|---|---|
| EF-M 18-55 f/3.5-5.6 IS STM at 18 mm | Solid |
| EF-M 18-55 at 55 mm | 5/5 refocus tests (5.6–10.1 s) |
| EF 50 f/1.8 STM on a 0.71× speedbooster | 5/5 refocus tests (3.3–6.1 s) |

If you test other lenses, please send results (see [Contributing](#contributing)).

## Install

Download the zip from the latest [release](https://github.com/mgojkovic/crop-mood-eosm-slim-gui-af/releases), or build it yourself (below).

With Magic Lantern already on the card, copy **all three of these, from the same build**:

1. `autoexec.bin` to the card root
2. `ML/modules/EOSM_202.sym`
3. `ML/modules/*.mo`

If the `.sym` file doesn't match `autoexec.bin`, modules fail to load or the camera
crashes at boot. A first-time install still needs Canon's firmware-update step, as for
any Magic Lantern build.

## Build

- Toolchain: ARM GCC 5.4 2016q3 (`gcc-arm-none-eabi-5_4-2016q3`). On Apple Silicon it
  runs under Rosetta, and `Makefile.setup.toolchain` finds it automatically.
- Every `modules/*/` folder needs a `module_strings.h`. A clean or failed build deletes
  them, so recreate empty ones before building:

  ```sh
  for d in modules/*/; do [ -f "$d/module_strings.h" ] || printf '' > "$d/module_strings.h"; done
  make -C platform/EOSM.202 zip SHELL=/bin/bash PYTHON=python3
  ```

- `make zip` can exit 0 even when `crop_rec.c` failed to compile, so check the module
  inside the zip against the one you built:
  `unzip -p <zip> ML/modules/crop_rec.mo | md5` against `md5 modules/crop_rec/crop_rec.mo`.
- The zip name includes the build date, so always flash the newest zip.

## How it works, and the research behind it

The design notes, findings and test logs are kept in the repo:

- [doc/EOSM-AF.md](doc/EOSM-AF.md): features, findings, firmware addresses (start here)
- [doc/EOSM-AF-HANDOVER.md](doc/EOSM-AF-HANDOVER.md): current state, open problems, code map
- [doc/EOSM-MPU.md](doc/EOSM-MPU.md), [doc/EOSM-MPU-UART.md](doc/EOSM-MPU-UART.md): how the EOS M lens processor is driven
- [contrib/af_test/](contrib/af_test/), [contrib/af_replay/](contrib/af_replay/): offline AF test harness and simulator, run against real camera logs

The AF code is in [modules/crop_rec/crop_rec.c](modules/crop_rec/crop_rec.c).

## Contributing

Field reports are the most useful thing right now. Please open an issue with:

- the lens and focal length
- what you did, and what happened
- the AF logs from `ML/LOGS/` on the card (logging is on by default)

Pull requests are welcome too.

## Credits and license

- [Magic Lantern](https://www.magiclantern.fm/), and a1ex for `crop_rec`. The original
  Magic Lantern README is in [doc/MAGIC-LANTERN-README.md](doc/MAGIC-LANTERN-README.md).
- The Crop mood builds for the EOS M.
- [Amit199167/Crop-mood-eosm-slim-gui](https://github.com/Amit199167/Crop-mood-eosm-slim-gui),
  for the slim GUI this build starts from.

Licensed under the GNU GPL v2, like Magic Lantern. See [COPYING](COPYING).
