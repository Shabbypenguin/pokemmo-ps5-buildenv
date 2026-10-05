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
| 12.40 | kstuff-lite | pokemmo-ps5 probe | **not run yet** | — |

## Installing a title

1. Copy the title folder (for the probe: `PPSA27165/` from `dist/pokemmo-ps5-probe-PPSA27165.zip`) to
   `/data/homebrew/PPSA27165/`, either with `make deploy-probe PS5_HOST=<console IP>` (FTP) or by hand.
2. Let your mounting tool pick it up (see its documentation; a refresh may be needed after the first copy).
3. Launch it from the home screen.

Titles are sandboxed: they write to `/download0` (their own storage, `/user/download/<TITLE_ID>/` over FTP), not
to `/data`.
