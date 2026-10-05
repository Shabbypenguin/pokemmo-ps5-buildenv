# pokemmo-ps5-buildenv

Everything needed to build **[PokeMMO-Prospero](https://github.com/Shabbypenguin/PokeMMO-Prospero)** (the PokeMMO-on-PS5 port) and to keep the project alive
if its maintainer stops: a pinned toolchain image, a wrapper to use it, the console setup it targets, and a
maintainer's guide.

Toolchain versions are pinned in [docker/Dockerfile](docker/Dockerfile), so a build today and a build next year use
the same compilers and SDKs.

## Quick start

Needs an x86-64 Linux machine with Docker and about 4 GB of disk.

```bash
git clone <this-repo-url> pokemmo-ps5-buildenv
git clone https://github.com/Shabbypenguin/PokeMMO-Prospero.git   # side by side
cd PokeMMO-Prospero
../pokemmo-ps5-buildenv/ps5env build           # once, ~10 minutes
../pokemmo-ps5-buildenv/ps5env doctor          # checks Docker, the image and the checkout
../pokemmo-ps5-buildenv/ps5env make probe      # builds dist/pokemmo-prospero-probe-PPSA27165.zip
```

`ps5env` runs `sudo docker` by default. If your user is in the `docker` group: `export DOCKER=docker`.
A prebuilt image is published by CI as `ghcr.io/<owner>/pokemmo-ps5-buildenv:latest`; use it with
`export PS5ENV_IMAGE=ghcr.io/<owner>/pokemmo-ps5-buildenv:latest` and skip `ps5env build`.

## What's in the image

| Path | Component | Pin |
|------|-----------|-----|
| `/opt/ps5/payload-sdk` | [ps5-payload-sdk](https://github.com/ps5-payload-dev/sdk) | v0.42 (SHA-256 checked by the boilerplate) |
| `/opt/ps5/boilerplate` | [ps5-native-app-boilerplate](https://github.com/blackbearreloaded/ps5-native-app-boilerplate) | commit `4f531c4b` |
| `/opt/ps5/opengl` | [ps5-opengl](https://github.com/blackbearreloaded/ps5-opengl) SDK | 1.0.1 (SHA-256 pinned) |
| `/opt/ps5/opengl-src` | ps5-opengl sources bundled with that SDK | same release |
| system | Ubuntu 24.04, Clang/LLD 18, binutils, Python 3 | distribution packages |

The pins are the same versions ps5-opengl uses for its own builds. Do not bump one without the others: see
[docs/maintainers.md](docs/maintainers.md#updating-pins).

## Documentation

| Document | For |
|----------|-----|
| [docs/console-setup.md](docs/console-setup.md) | preparing a PS5 to run the titles |
| [docs/maintainers.md](docs/maintainers.md) | taking over the project: how the pieces fit, routine tasks, releases |
| [docs/troubleshooting.md](docs/troubleshooting.md) | build environment problems |

## Credits

The toolchain is other people's work: the PS5 payload SDK (John Törnblom and contributors), the native-app
boilerplate and ps5-opengl (BlackBearReloaded). The port itself credits PokeMMO-NX (Petit_Prince) and the rest in
its own `CREDITS.md`. Parts of these repositories were written with AI assistance (Anthropic's Claude); such commits
carry a `Co-Authored-By` trailer.

## License

GPL-3.0-or-later ([LICENSE](LICENSE)), matching the port and the projects it builds on.
