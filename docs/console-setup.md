# Console setup

The titles this project builds are **folder titles** in the format
[ps5-native-app-boilerplate](https://github.com/blackbearreloaded/ps5-native-app-boilerplate) produces: a
fake-signed `eboot.bin`, a clean-room `sce_module/libc.prx` and `sce_sys/param.json`. The console needs a homebrew
environment that can mount and launch such titles. This repository does not jailbreak or configure consoles; follow
each project's own documentation for that.

## What the titles need

| Need | Provided by | Notes |
|------|-------------|-------|
| Kernel exploit / homebrew enabler for your firmware | your jailbreak chain | outside this project's scope |
| Running fake-signed (FSELF) titles | [kstuff-lite](https://github.com/EchoStretch/kstuff-lite) | what the maintainer uses |
| Mounting and registering folder titles from `/data/homebrew` | [ShadowMountPlus](https://github.com/drakmor/ShadowMountPlus) | the loader the boilerplate validated with |
| Uploading titles | an FTP server such as [ftpsrv](https://github.com/ps5-payload-dev/ftpsrv) (port 2121) | `make deploy-probe` uses it |
| Loading ELF payloads (port 9021) | e.g. [elfldr](https://github.com/ps5-payload-dev/elfldr) | not needed by the titles today |

Other homebrew environments (for example etaHEN) may also work but are untested with these titles.

## Tested combinations

Only combinations someone has actually run belong here. Add a row when you confirm one, with the probe log.

| Firmware | Setup | What ran | Result | Reported by |
|----------|-------|----------|--------|-------------|
| 6.02, 12.70 | ShadowMountPlus, ftpsrv | boilerplate hello world (upstream's own testing) | works | boilerplate docs |
| 6.02 | — | ps5-opengl examples (upstream's own testing) | works | ps5-opengl docs |
| 12.40 | kstuff-lite 1.07+, ShadowMountPlus 1.7beta3, ftpsrv | PokeMMO-Prospero probe-2 | title launches; graphics, threads, direct memory pass; crashed in `getaddrinfo` | maintainer, 2026-10-05 |
| 12.40 | same | PokeMMO-Prospero probe-3 | 20 pass / 7 fail / 4 info; one `getaddrinfo` crash, completed on relaunch | maintainer, 2026-10-05 |

## Installing a title

Use the installer that ships with every release (`install.bat` on Windows, `install.command` on macOS,
`install.sh` on Linux; details in the port's `installer/README.md`). It asks for the console's FTP address,
uploads the title to `/data/homebrew/<TITLE_ID>/`, and offers to upload your ROMs to
`/data/homebrew/<TITLE_ID>/roms/`. Developers can use `make deploy-probe PS5_HOST=<console IP>` instead.

After installing, let your title mounter pick the title up, then launch it from the home screen. Close the title
before reinstalling it.

**Reinstalling (ShadowMountPlus):** SMP runs titles from its own copy on a virtual drive, so uploading a new build
over FTP does not replace what launches. Delete the title from the home screen, then wait for SMP to add it
again from `/data/homebrew/<TITLE_ID>/` (the uploaded folder and ROMs stay in place). Expect the title's `/download0`
storage to be wiped by the delete.

**Close every running game or app first.** ShadowMountPlus pauses all scanning while any game or app is running
(its log shows `[GAME] started: <ID>` and then no more scan lines), so a newly installed title only appears once
nothing is running. It rescans every 15 seconds and ignores folders changed in the last 10 seconds. Its log is
`/data/shadowmount/debug.log`; a title it can see but rejects shows up there as `[SKIP] ...`.

Titles are sandboxed. They write only to `/download0`, which is backed by a storage image that FTP cannot
browse. Logs therefore go out over UDP (`tools/udplog.py` in the port).
