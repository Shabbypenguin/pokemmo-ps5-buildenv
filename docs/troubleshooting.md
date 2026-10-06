# Troubleshooting

## `permission denied ... /var/run/docker.sock`

`ps5env` uses `sudo docker` by default. If you set `DOCKER=docker`, your user must be in the `docker` group
(`sudo usermod -aG docker $USER`, then log out and in).

## The image build fails downloading something

Every download is pinned and checksum-verified, so a failure is a network problem, not a silent version change.

- **Docker Hub unreachable** (`failed to resolve source metadata for docker.io/library/ubuntu`): pull
  `ubuntu:24.04` through a mirror you can reach and tag it `ubuntu:24.04` locally, then rebuild.
- **zlib**: the Dockerfile pre-seeds zlib 1.3.2 from its GitHub release because zlib.net is often unreachable; the
  boilerplate's script still verifies its SHA-256.
- **Behind a TLS-intercepting proxy**: put the proxy's CA certificate (`*.crt`) in `docker/extra-ca/` (git-ignored)
  and pass the proxy to the build:
  `PS5ENV_DOCKER_FLAGS="--network host --build-arg HTTPS_PROXY=$HTTPS_PROXY" ./ps5env build`.

## `PokeMMO-Prospero checkout not found`

`ps5env` looks for the port in `$PORT_DIR`, then the current directory, then `../PokeMMO-Prospero` (or `../pokemmo-prospero`) next to
this repository. `cd` into your checkout or set `PORT_DIR`.

## `Not inside the build environment`

Run port Makefile targets through the wrapper: `../pokemmo-ps5-buildenv/ps5env make probe`, not plain `make probe`.

## Files in the port checkout are owned by root

Happens when the container was started as root (e.g. `ps5env shell`). Fix with
`sudo chown -R "$USER": build dist .ccache .home` in the port checkout. Normal `ps5env <command>` runs as your user.

## `boilerplate heap hook changed` / `boilerplate link step changed`

The pinned boilerplate was changed without updating `scripts/build-title.sh`. See
[maintainers.md, Updating pins](maintainers.md#updating-pins).

## The title doesn't appear or won't launch on the console

That's the console environment, not the build:

1. Check that `eboot.bin`, `sce_module/libc.prx` and `sce_sys/param.json` all exist directly in
   `/data/homebrew/<TITLE_ID>/` (not one folder deeper: ShadowMountPlus scans one level by default).
2. **Close any running game or app.** ShadowMountPlus doesn't scan while one is running.
3. Read `/data/shadowmount/debug.log`: no mention of the title means it was never scanned (step 1 or 2); a
   `[SKIP]` line says why it was rejected.

See [console-setup.md](console-setup.md).
