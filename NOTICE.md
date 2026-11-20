# NOTICE — third-party components, licences and attributions

This package (`cloudflareddocker`, TOS 7 Docker application, version
`2026.5.0-3`) is an independent packaging by Moechz of the upstream
**Cloudflared-web** project. No upstream source code is redistributed here: the
package reproduces the official Docker Hub image unmodified and adds only the
four TOS metadata files.

## 1. Upstream application — Cloudflared-web

- Project: <https://github.com/WisdomSky/Cloudflared-web>
- Author: WisdomSky (Louis Lam's `cloudflare-tunnel` backend is derived from his
  MIT-licensed work; the copyright line is preserved in the image)
- Version: **2026.5.0** (the container image tag; upstream repository state of
  2026-05-13)
- Licence: **MIT** — <https://github.com/WisdomSky/Cloudflared-web/blob/main/LICENSE>
- Relationship: used **unmodified** inside the container image. This package
  only supplies `config.ini`, `cloudflareddocker.lang`, `cloudflareddocker.svg`
  and `docker-compose.yml`.

## 2. Container image — `wisdomsky/cloudflared-web:2026.5.0`

- Registry: Docker Hub — `docker.io/wisdomsky/cloudflared-web:2026.5.0`
- Multi-arch: `linux/amd64` and `linux/arm64` manifests under the same tag
- Local digest as measured for this release:
  `wisdomsky/cloudflared-web@sha256:8bdc2e261bdf4b7824c67af8a4acc04fdaa3f1848e82a3ca56ccfd30040855ff`
  (`linux/amd64`; the `linux/arm64` manifest has its own digest — both are
  pulled from the same fixed tag, never from `:latest`)
- Maintainer: WisdomSky. The image is pulled from Docker Hub at install time and
  is not redistributed in this package.
- Base image: `node:18-bookworm-slim` (Node.js 18.20.8, Debian Bookworm slim) —
  Node.js is MIT-licensed, Debian packages keep their own licences.

## 3. Components inside the image

| Component | Version | Licence |
|---|---|---|
| `cloudflared` (Cloudflare Tunnel client) | 2026.5.0 (build 2026-05-13) | **Apache-2.0** — <https://github.com/cloudflare/cloudflared/blob/master/LICENSE> |
| Node.js runtime | 18.20.8 | MIT — <https://github.com/nodejs/node/blob/main/LICENSE> |
| Express, body-parser, cors, express-basic-auth, `tmp`, `command-exists` (backend) | as locked in `app/backend/package-lock.json` | MIT |
| Vue 3, Vite, vue-i18n, radix-vue, tailwindcss, lucide-vue-next, prismjs and the other frontend dependencies | as locked in `app/frontend/package-lock.json` | MIT / ISC / Apache-2.0 as declared by each package |
| Debian Bookworm slim userland (CA certificates, tzdata, curl) | Bookworm | various free licences (GPL/LGPL/BSD/MIT/MPL) |

Those components are shipped **inside the upstream image**; this package neither
modifies nor repackages them, and their licence texts remain in the image (for
example `/usr/share/doc/*/copyright`).

## 4. This packaging

The files authored for this package — `docker-compose.yml`, `config.ini`,
`cloudflareddocker.lang`, `cloudflareddocker.svg`, the build scripts, the
documentation under `docs/`, `README.md`, `PRIVACY.md` and this `NOTICE.md` —
are released under **MIT** (see `LICENSE`), © 2026 Moechz.

`TOSAppSelfTestingTool` is an official TerraMaster utility, redistributed here
unmodified for local package validation; it remains the property of
TerraMaster.

## 5. Trademarks

“Cloudflare”, “Cloudflare Tunnel”, “Cloudflare Zero Trust” and “cloudflared” are
trademarks or registered trademarks of **Cloudflare, Inc.** “TerraMaster”,
“TOS” and “App Center” are trademarks of **TerraMaster**. “Docker” and “Docker
Hub” are trademarks of **Docker, Inc.** “Node.js” is a trademark of the OpenJS
Foundation.

All marks are used descriptively only, to identify the software being packaged
and the platform it runs on. This package is **not affiliated with, sponsored or
endorsed by** Cloudflare, Inc., TerraMaster or WisdomSky.

## 6. Icon

`src/cloudflareddocker.svg` is a generic tunnel/cloud mark drawn for this
package. It does not reproduce the Cloudflare logo or the upstream project's
icon.

## 7. Contact

Moechz — moechz@163.com — <https://github.com/Moechz/cloudflared-web>
