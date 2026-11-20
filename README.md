# cloudflared-web-for-tos

> TerraMaster TOS 7 App Center — **Docker application** package for
> [Cloudflared Web](https://github.com/WisdomSky/Cloudflared-web).
>
> Current package: **`2026.5.0-3`** · image `wisdomsky/cloudflared-web:2026.5.0`
> (Docker Hub, unmodified) · architectures `x86_64` + `aarch64` · licence MIT for
> this packaging (see [`NOTICE.md`](NOTICE.md)).

## What this is

A four-file TOS Docker application archive (`cloudflareddocker.tar.gz`) that
installs Cloudflared Web on TOS 7 through the App Center:

- `config.ini` — TOS application metadata (`application_type: docker`, id
  `cloudflareddocker`, help link = the privacy policy)
- `cloudflareddocker.lang` — 23-language superset (TOS requires 14)
- `cloudflareddocker.svg` — application icon
- `docker-compose.yml` — one service, image `wisdomsky/cloudflared-web:2026.5.0`

## What it actually does

Cloudflared Web is a small web UI in front of the Cloudflare Tunnel client:

| Feature | Works in this package |
|---|---|
| Paste / replace the token of a **remotely-managed** Cloudflare tunnel | yes |
| **Start** and **stop** the tunnel with one click | yes |
| Show the saved token and the “running” flag (`/config`) | yes |
| Show the bundled `cloudflared` version | yes (`cloudflared 2026.5.0`) |
| Optional metrics endpoint (`METRICS_ENABLE`, port 60123) | available but **disabled** by default |
| Optional HTTP Basic Auth (`BASIC_AUTH_USER` / `BASIC_AUTH_PASS`) | available but **not enabled** by the package |
| Upstream “advanced local config” tab (writes `/root/.cloudflared`) | **not usable** — root-only upstream, see [Limitations](#limitations) |
| Download a `cloudflared` client binary | **not a feature of this app** (the client is inside the container) |

Public hostnames are created in the **Cloudflare Zero Trust dashboard**, not in
this UI; the UI only runs the tunnel.

## How it works

| | |
|---|---|
| Container image | `wisdomsky/cloudflared-web:2026.5.0` (Docker Hub, used unmodified) |
| Open | `http://<NAS-IP>:8121/` (container port 14333) |
| Runs as | non-root, `user: "1000:1000"`, `cap_drop: [ALL]`, `no-new-privileges` |
| Persistent data | `/config` → `/Volume1/DockerAppData/cloudflareddocker/config` |
| Logs | `docker logs cloudflareddocker` (stdout/stderr) |
| Outbound | HTTPS/QUIC 443 to the Cloudflare edge; optional anonymous update check to `registry.hub.docker.com` |
| Networking | bridge; only 8121 is published |

## Repository layout

| Path | Purpose |
|---|---|
| `tosapp.json` | Application spec: version, image, ports, volumes, descriptions, translations |
| `src/` | The four files that go into the package |
| `src/docker-compose.yml` | Service definition (user, caps, port, volume, healthcheck, `x-app-meta`) |
| `src/config.ini` | App Center metadata (path/port, help, category, platform) |
| `src/cloudflareddocker.lang` | 23-language UI strings, generated from `tosapp.json` (`tosdocker lang`) |
| `src/cloudflareddocker.svg` | Icon (`viewBox="0 0 512 512"`) |
| `scripts/build.sh` | Stage → substitute → verify → deterministic tar.gz + sha256 |
| `scripts/build-dual.sh` | Build x86_64 **and** aarch64, run the official `-c` check on both |
| `tools/verify.py` | Review-standards self-check (layout, fields, ports, user, secrets, …) |
| `test/install-and-verify.sh` | On-device E2E (needs root; not used for review automation) |
| `docs/ROOT-PRIVILEGE.md` | S1/V1 analysis: why the service is non-root, measured evidence, least privilege |
| `docs/REMEDIATION.md` | Code-by-code record of the 2026.5.0-2 review remediation |
| `PRIVACY.md` | Privacy policy (also the App Center help target) |
| `NOTICE.md` | Third-party licences, digests and trademarks |
| `CHANGELOG.md` | User-visible changes per package version |
| `README.md` | This file |

## Design highlights

- **Non-root by construction.** The upstream image declares `USER root`; the
  package overrides it with `user: "1000:1000"` and additionally drops all
  capabilities and sets `no-new-privileges`. Measured on hardware: the container
  reports `uid=1000(node)`, the UI answers HTTP 200 and the tunnel starts. See
  [`docs/ROOT-PRIVILEGE.md`](docs/ROOT-PRIVILEGE.md).
- **Fixed image tag, never `:latest`**, Docker Hub only, no `privileged`, no
  `network_mode: host`, no `cap_add`, no Docker socket.
- **Health check** probing the service on container port 14333.
- **Deterministic build**: GNU tar with uid/gid 0 and zeroed timestamps, gzip
  without a timestamp, plus the review-standards self-check above.
- **One volume, one port.** `/config` is the only persistent path, 8121 the only
  published port.
- **No secrets in the package.** The tunnel token is entered by the user at
  runtime and stored only in the data folder.

## Privacy and data

Everything stays on the NAS. The only persistent state is
`/Volume1/DockerAppData/cloudflareddocker/config/config.json` (tunnel token +
start flag); logs live in the Docker log buffer; tunnel traffic is proxied, not
stored. The app contacts Cloudflare (the tunnel), the Docker Hub registry (an
optional anonymous update check) and the services you choose to publish. No
telemetry, no notifications, no packager-operated server. Full detail, including
the retention period, your rights and the deletion channel, is in
[`PRIVACY.md`](PRIVACY.md).

**Delete your data:** press Stop, delete the token in the UI, uninstall the app
with “delete data”, or remove
`/Volume1/DockerAppData/cloudflareddocker/`. Questions:
**moechz@163.com**.

## Limitations

- **The web UI is unauthenticated by default.** The upstream Basic Auth is
  available (`BASIC_AUTH_USER` / `BASIC_AUTH_PASS`) but the package does not set
  a password, so anyone who can reach port 8121 can change the tunnel token.
  Keep it on the LAN or put it behind a TLS reverse proxy.
- **Bridge networking.** The container is not on the host network (TOS rule), so
  a tunnel ingress rule must point at an address reachable *from the container*:
  the NAS LAN IP or the Docker bridge gateway, **not** `127.0.0.1`. Set the
  hostname → target mapping in the Cloudflare Zero Trust dashboard.
- **The upstream “advanced local config” tab does not work as uid 1000** — it
  writes `/root/.cloudflared/config.yml`, which is root-owned in the image. Use
  the dashboard-managed (remote) tunnel instead; this tab is not part of the
  declared feature set.
- **Token handling.** `cloudflared` receives the token as a command-line
  argument and the upstream UI echoes it into the container log when it is
  saved. Treat `docker logs` / `docker inspect` output as sensitive and keep
  shell access to the NAS restricted.
- **Known CVE follow-up (T1).** The image is pinned to the reviewed tag
  `2026.5.0` (Node 18 base, `cloudflared` 2026.5.0). Upstream builds
  `cloudflared` 2026.11.1 today; the next release upgrades the image tag and
  carries the remediation note, so that change stays reviewable on its own.

## Build

```bash
scripts/build.sh                       # -> out/cloudflareddocker.tar.gz + .sha256
scripts/build.sh 2026.5.0-3            # packaging iteration bump
scripts/build-dual.sh 2026.5.0-3       # both architectures + official -c on each
scripts/build-dual.sh 2026.5.0-3 aarch64
```

The build stages the four files, substitutes the version/platform placeholders,
removes CRLF/BOM/AppleDouble, then runs `tools/verify.py`, a review-standards
self-check (archive layout, JSON validity, required fields, 14+ language
coverage, version consistency, Docker-Hub-only images, fixed tag,
reserved/recommended host ports, non-root user, per-service
healthcheck/TZ/restart, data path below the application data root, never the
`/Volume*` wildcard, comment-free sequence blocks, `x-app-meta` position, no
literal secrets and the SVG size/element limits).

## Local verification

```bash
./TOSAppSelfTestingTool -c out/cloudflareddocker_x86_64.tar.gz   # official spec check
./TOSAppSelfTestingTool -c out/cloudflareddocker_aarch64.tar.gz
bash test/install-and-verify.sh                                  # on-device E2E (needs root)
```

> Docker applications cannot be side-loaded through the App Center's manual
> install page (it accepts `.deb` only); the official self-testing tool's `-i`
> mode or `docker compose` are the only ways to install one for testing.

## Store submission

1. Public GitHub repository (code + documentation only). ✅
2. Release **tag `v<version>`**, assets `<app_id>_x86_64.tar.gz` /
   `<app_id>_aarch64.tar.gz` plus their `.sha256` files.
3. Developer platform → Add Application: ID `cloudflareddocker`, package type
   Docker, repository URL, architecture `x86_64` / `aarch64`.
4. Version Management → Add Version → automated validation → review; the
   submission remarks carry “Rectified per review comments”.

## Licence and branding

This packaging is MIT (see `LICENSE`) and is an independent community submission
by Moechz. The upstream application and its container image are MIT; the bundled
`cloudflared` is Apache-2.0 by Cloudflare, Inc. All trademarks belong to their
owners and are used descriptively only — see [`NOTICE.md`](NOTICE.md). Not
affiliated with, sponsored or endorsed by Cloudflare, WisdomSky or TerraMaster.

---

Packaged for TOS 7 by Moechz. Upstream image used unmodified from Docker Hub.
