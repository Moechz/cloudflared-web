# Privacy Policy — Cloudflared Web for TerraMaster TOS (`cloudflareddocker`)

**Last updated:** 20 November 2026
**Data controller / contact:** Moechz — moechz@163.com
**Applies to:** version `2026.5.0-3` and later of the TOS application
`cloudflareddocker`, the container image it runs and this repository.
**Public URL:** <https://github.com/Moechz/cloudflared-web/blob/main/PRIVACY.md>
(the same URL is shipped as the App Center *Help* link in `config.ini`, so the
policy opens from inside the application)

## 1. Summary

- The application runs **entirely on your NAS**. The packager does not operate
  any server, relay, telemetry endpoint or analytics service, and never receives
  your data.
- The only persistent state is the Cloudflare tunnel token and the "running"
  flag, kept in the application data folder on your own disk.
- Three parties can be contacted at runtime, and only for the purposes listed in
  §6: **Cloudflare** (the tunnel itself), the **Docker Hub registry** (an
  optional update check) and the **services you choose to publish** through the
  tunnel.
- No account, e-mail address, phone number or usage statistics are collected by
  the packager.

## 2. What data is processed, and where it is stored

| Data | Stored where | Written by |
|---|---|---|
| Cloudflare tunnel token and the "tunnel is running" flag (`config.json`) | `/Volume1/DockerAppData/cloudflareddocker/config/config.json` on your NAS | the web UI when you paste a token or press Start/Stop |
| Tunnel traffic (HTTP requests and responses that pass through the tunnel) | not stored; proxied in memory between Cloudflare and the target service | `cloudflared` |
| Container logs (startup lines, tunnel status, the token echo described in §5) | Docker's log buffer only — `docker logs cloudflareddocker`, no log file on the NAS | the application's stdout/stderr |
| Web UI access | no access log is written by the package; the container's stdout shows only startup and tunnel state | — |

The container is otherwise stateless: `/config` is the only bind mount, and
removing the container does not delete the folder. Nothing is uploaded to the
packager.

## 3. Retention period

- **Tunnel token / configuration:** kept until **you** delete it — there is no
  automatic expiry. Removing the token in the UI, uninstalling the app with
  "delete data", or removing
  `/Volume1/DockerAppData/cloudflareddocker/` erases it.
- **Container logs:** stdout/stderr only; they live in the Docker log buffer and
  follow the daemon's rotation settings (`docker logs` / `docker compose logs`).
  They are destroyed with the container.
- **Tunnel traffic:** not retained at all — it is proxied, not stored.
- **Backups:** if you back up `/Volume1/DockerAppData/cloudflareddocker/`, the
  copy follows your backup tool's retention; the packager keeps no copy.

## 4. Your rights and the deletion channel

| Right | How to exercise it |
|---|---|
| **Access / export** | The complete state is the file `/Volume1/DockerAppData/cloudflareddocker/config/config.json`; copy it, or read it with `docker exec cloudflareddocker cat /config/config.json`. |
| **Correct** | Paste a new token in the web UI (it overwrites the old one), or edit `config.json` while the container is stopped. |
| **Delete** | Press Stop, delete the token in the UI, uninstall the app with "delete data", or `docker compose down` and remove `/Volume1/DockerAppData/cloudflareddocker/`. Deleting a Cloudflare tunnel itself is done in the Cloudflare Zero Trust dashboard. |
| **Object / restrict** | Disable the tunnel (Stop), do not configure public hostnames, or uninstall the app. |
| **Assistance** | E-mail **moechz@163.com** with the subject “cloudflareddocker data request”; a reply follows within 30 days. |

Because the packager holds no copy of your data, access and deletion are
completed locally on your NAS; the e-mail channel exists for questions and for
requests about the repository and the listing records.

## 5. Security measures

- **Non-root.** The container is pinned to `user: "1000:1000"`, runs with
  `cap_drop: [ALL]` and `security_opt: [no-new-privileges:true]`; there is no
  `privileged`, no `cap_add`, no host network and no Docker socket. The upstream
  image metadata declares `USER root`, which is why the review scanner reports a
  root finding; the running process is `uid=1000(node)` — see
  [`docs/ROOT-PRIVILEGE.md`](docs/ROOT-PRIVILEGE.md) for the measured evidence.
- **Least exposure.** Only one port is published (host 8121 → container 14333);
  the image's optional metrics port 60123 is disabled (`METRICS_ENABLE=false`).
- **One write path.** `/config` is the only mount and the only path the
  application writes on the NAS; it lives below
  `/Volume1/DockerAppData/cloudflareddocker/`.
- **Pinned supply chain.** The image is a fixed Docker Hub tag
  (`wisdomsky/cloudflared-web:2026.5.0`, never `:latest`) and the release assets
  are published with their SHA-256 checksums.
- **Nothing hard-coded.** The package contains no password, token or API key;
  the tunnel token is entered by you at runtime. The optional HTTP Basic Auth of
  the upstream UI (`BASIC_AUTH_USER` / `BASIC_AUTH_PASS`) is not enabled by the
  package, so **the web UI is unauthenticated by default** — keep it on the LAN
  or put it behind a TLS reverse proxy. For the same reason the tunnel token
  should be treated as a secret: the upstream UI echoes it into the container
  log when it is saved and passes it to `cloudflared` on the command line, so
  restrict who can run `docker logs` / `docker inspect` on the NAS.

## 6. Third-party APIs and sharing

The packager shares nothing with anyone. The application itself contacts only
the endpoints below, and only as described.

| # | Service / endpoint | When | Data sent | Destination / region |
|---|---|---|---|---|
| 1 | **Cloudflare global network** — `cloudflared tunnel run --token …` connects to the Cloudflare edge (`*.argotunnel.com`, UDP/QUIC or TCP/443) | while the tunnel runs | the tunnel token, and the HTTP requests/responses you publish through the tunnel | Cloudflare, Inc. (US); the edge location is chosen by Cloudflare unless you set `REGION=us` |
| 2 | **Cloudflare Zero Trust dashboard / API** — `dash.cloudflare.com` / `api.cloudflare.com` | when **you** create the tunnel and add public hostnames in a browser | your Cloudflare account session, handled by your browser — the app never talks to this API | Cloudflare, Inc. (US) |
| 3 | **Docker Hub registry API** — `https://registry.hub.docker.com/v2/repositories/wisdomsky/cloudflared-web/tags/` | when the web UI runs its “new version” check | an anonymous HTTPS request; no token, no NAS data, no identifier beyond your IP address | Docker, Inc. (US) |
| 4 | **Docker Hub image pull** — `registry-1.docker.io` (or your configured mirror) | at install time | an anonymous pull of the pinned image | Docker Hub / your mirror |
| 5 | **The services you publish** — any hostname you add in the Cloudflare dashboard | while the tunnel runs | the traffic Cloudflare forwards to that target, addressed from the container | your own network / the target you chose |

There is **no** telemetry, no crash reporting, no update phone-home from the
package, and no Telegram, e-mail or webhook notification: the package does not
send messages anywhere. If you point a tunnel at a service on the NAS itself,
remember that the target must be reachable from the container network — use the
NAS LAN IP (or the Docker bridge gateway), not `127.0.0.1`.

## 7. Children

The application is a self-hosted network utility and is not directed at
children; it does not knowingly process data from children.

## 8. Changes to this policy

Material changes are published in this file, reflected in the “Last updated”
date and noted in `CHANGELOG.md`. The policy ships with every release of the
package and is linked from the App Center help entry.

## 9. Contact

Moechz — **moechz@163.com** — <https://github.com/Moechz/cloudflared-web>
