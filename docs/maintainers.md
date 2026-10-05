# Maintainer's guide

For whoever keeps PokeMMO-Prospero going next. Everything is pinned and rebuildable; the only secret is a read-only
token CI needs while the port repository is private (see [CI access](#ci-access)).

## How the pieces fit

```
pokemmo-ps5-buildenv (this repo)          PokeMMO-Prospero (the port)
  docker/Dockerfile  ── builds ──▶ image    probe/, loader/ (Phase 2), tools/, docs/
  ps5env             ── runs the image with the port checkout mounted at /work
                                            scripts/build-title.sh
                                              ├─ copies the boilerplate's build system   (/opt/ps5/boilerplate)
                                              ├─ copies ps5-opengl's native-app glue      (/opt/ps5/opengl-src)
                                              ├─ links ps5-opengl                         (/opt/ps5/opengl)
                                              └─ produces build/titles/<ID>/dist/<ID>/ + zip
```

The image exports the paths as `PS5_NATIVE_APP_TEMPLATE`, `PS5_PAYLOAD_SDK`, `PS5_OPENGL_PREFIX`,
`PS5_OPENGL_SOURCE` and `PS5_CLANG`. The port's Makefile refuses to run outside the image (`env-check`).

Title IDs: `PPSA27165` is the probe. `PPSA27166` is reserved for the game title. Keep them stable: changing a
title ID makes the console treat it as a different application with separate storage.

## Routine tasks

### A new PokeMMO client release

```bash
../pokemmo-ps5-buildenv/ps5env make fetch-client   # into private/ (ignored by git, never commit it)
../pokemmo-ps5-buildenv/ps5env make analyze
```

`analyze` exits non-zero and lists what changed if the client now needs something the loader doesn't provide:
new libc/zlib imports, new relocation types, a TLS segment, raw syscalls, new `%fs`/`%gs` accesses, or new
runtime libraries. Teach the loader first, then record the new baseline:

```bash
../pokemmo-ps5-buildenv/ps5env python3 tools/analyze_client.py private/PokeMMO-Client.zip --write-baseline
```

Commit the updated `tools/client-baseline.txt` together with the loader change and mention the client revision
(`revision.txt` in the zip) in the commit message.

### A new console firmware or jailbreak chain

Run the probe on it (see `docs/probe.md` in the PokeMMO-Prospero repository) and add a row to
[console-setup.md](console-setup.md#tested-combinations) and to the port's `docs/plan.md` hardware table, with
the log attached to the commit or an issue.

### Updating pins

The three toolchain pins move together, following ps5-opengl, because ps5-opengl's SDK is built against a specific
boilerplate commit and payload SDK:

1. Pick the new ps5-opengl release. Its SDK archive contains `dependencies.json`; read
   `native_boilerplate.revision` and `native_boilerplate.payload_sdk`.
2. In `docker/Dockerfile` set:
   - `PS5_OPENGL_VERSION` and `PS5_OPENGL_SHA256` (from the release's `ps5-opengl-sdk-<version>.tar.gz.sha256` asset),
   - `BOILERPLATE_COMMIT` to `native_boilerplate.revision`.
   The payload SDK version and hash are pinned inside the boilerplate's `tools/setup-native-dependencies.sh`, so they
   follow the boilerplate commit. Check that its zlib version still matches the pre-seeded
   `zlib-1.3.2.tar.gz` line in the Dockerfile.
3. `ps5env build`, then `ps5env make probe` in the port. `scripts/build-title.sh` patches two spots in the
   boilerplate (the process heap size line and the `--eh-frame-hdr` link line) and copies files from ps5-opengl's
   `native-app/`. If upstream changed those, the script stops with "boilerplate heap hook changed" or "boilerplate
   link step changed". Compare against ps5-opengl's own `tools/build-native-test-app.sh` at the new release and
   mirror what it does.
4. Run the probe on a console before calling the new pins good. A CI build only proves it compiles.
5. Commit with the versions in the message, e.g. `pins: ps5-opengl 1.0.1 -> 1.1.0, boilerplate 4f531c4b -> …`.

## Releases

- Tag the port (`vX.Y.Z`). CI builds the titles and attaches the zips as artifacts.
- Release notes state: buildenv commit, PokeMMO client revision tested, firmware and console setup tested, and what
  is known not to work.
- A release never contains PokeMMO's client, ROMs, or any Sony file. Players fetch the client themselves.

## Rules the project keeps

- **Credit.** Code adapted from PokeMMO-NX keeps its MIT copyright notice in the file header, and the port's
  `CREDITS.md` table is updated in the same commit. Same for any other project's code.
- **License.** GPL-3.0-or-later. The titles link ps5-opengl (GPL), so the port must stay GPL-compatible.
- **Honesty.** "Works" means someone ran it on a console and recorded the result. Untested firmware stays marked untested.
- **AI assistance.** Commits written with AI help carry a `Co-Authored-By` trailer.

## CI access

The image workflow (this repo) builds the port's probe as a smoke test, and the port's workflow pulls the image.
While PokeMMO-Prospero is private, two grants are needed:

1. **Reading the port from this repo's workflow: secret `PROSPERO_READ_TOKEN`.**
   GitHub → Settings → Developer settings → Personal access tokens → Fine-grained tokens → Generate new token.
   Resource owner: the account that owns both repos. Repository access: *Only select repositories* →
   PokeMMO-Prospero. Permissions → Repository permissions → **Contents: Read-only** (Metadata: Read-only is added
   automatically). Pick an expiration and note it. Then in **this** repo: Settings → Secrets and variables →
   Actions → New repository secret, name `PROSPERO_READ_TOKEN`, paste the token.
   When it expires the smoke test fails at "Check out the port"; generate a new one and replace the secret.
2. **Pulling the image from the port's workflow: package access.** After the image workflow has published once:
   your profile → Packages → `pokemmo-ps5-buildenv` → Package settings → Manage Actions access → Add repository →
   PokeMMO-Prospero, role **Read**. (Or make the package public, which also lets anyone use the prebuilt image.)

When the port goes public: delete the secret (the built-in token then reads it) and make the package public.

## Taking over

1. Fork or transfer both repositories (`PokeMMO-Prospero` and `pokemmo-ps5-buildenv`) to the same owner and keep
   their names: the port's CI pulls `ghcr.io/<owner-lowercase>/pokemmo-ps5-buildenv:latest`, and the image workflow checks
   out `<owner>/PokeMMO-Prospero` for its smoke test. If the port repository is private, set up
   [CI access](#ci-access) under the new owner.
2. Run the buildenv `image` workflow once to publish the image under the new owner, and make the package public
   (GitHub → Packages → package settings) so anyone can pull it.
3. Build the probe, run it on your console, and record your setup in the tested-combinations table.
