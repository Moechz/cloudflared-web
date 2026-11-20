# Root privilege review (S1 / V1) — Cloudflared Web `cloudflareddocker`

**Applies to:** package `cloudflareddocker` version `2026.5.0-3` (review scope:
aarch64; the same package is built for x86_64)
**Review items:** S1 “The service does not run as a non-root user”, V1
“Application runs as root”, T3 “configuration errors (root / missing security
settings)”
**Conclusion up front: the package does **not** need root and does **not** ask
for a root exemption at runtime.** The service is started as `uid 1000`
(`node`) with all capabilities dropped. The root finding comes from the
*upstream image metadata* (`USER root` in the Dockerfile), not from how this
package runs it. If the review tooling insists on an exemption for the image
default, the material required by the Permission Exemption Application Guide
§7.1–§7.2 is collected in §4 below for the platform to decide on — it is not
self-granted here.

## 1. What the reviewer saw

The image `wisdomsky/cloudflared-web:2026.5.0` is built from a Dockerfile that
contains `USER root` (the web UI writes a legacy config to `/root/.cloudflared`
when the “advanced local config” tab is used). Image metadata therefore reports:

```
$ docker image inspect wisdomsky/cloudflared-web:2026.5.0 --format '{{.Config.User}}'
root
```

A scanner that reads image configuration — rather than the compose service
configuration that TOS actually applies — will report “runs as root” for every
package that uses this image, even though the package pins a non-root user.

## 2. What the package actually does

`src/docker-compose.yml` (shipped unchanged in the package):

```yaml
services:
  cloudflareddocker:
    image: wisdomsky/cloudflared-web:2026.5.0
    container_name: cloudflareddocker
    restart: unless-stopped
    user: "1000:1000"
    security_opt:
      - no-new-privileges:true
    cap_drop:
      - ALL
    ports:
      - "8121:14333"
```

- `user: "1000:1000"` overrides the image's `USER root`; Docker starts PID 1 as
  uid 1000, gid 1000.
- `cap_drop: [ALL]` removes every Linux capability (the container keeps none).
- `security_opt: [no-new-privileges:true]` blocks setuid/setgid escalation, so
  the process cannot become root even if a file in the image were setuid.
- No `privileged`, no `cap_add`, no `network_mode: host`, no `docker.sock`, no
  `pid`/`ipc` host namespaces, no device mapping.
- Only one bind mount: `/Volume1/DockerAppData/cloudflareddocker/config:/config`
  (read-write, the application's own data folder).

## 3. Measured evidence (real hardware, this NAS)

The image was started with exactly the constraints above (cap_drop ALL,
no-new-privileges, `--user 1000:1000`, `/config` mounted writable):

```
$ docker exec <container> id
uid=1000(node) gid=1000(node) groups=1000(node)

$ curl -s -o /dev/null -w '%{http_code}' http://127.0.0.1:<port>/
200                                    # web UI serves normally

$ docker logs <container>
STATUS: Init
WEBUI PORT: 14333
WEBUI HOST: 0.0.0.0
```

The two runtime operations the app exists for also work as uid 1000:

```
$ curl -X POST -d '{"token":"…"}' …/token        -> OK!        # token saved to /config/config.json
$ curl -X POST -d '{"start":true}' …/start       -> Started
$ docker logs … | tail
TUNNEL:  Starting cloudflared
TUNNEL:  Stopped cloudflared
Provided Tunnel token is not valid.            # only because the test token was fake
```

`cloudflared` can spawn as uid 1000; there is **no** port below 1024 (the web UI
is 14333, the optional metrics port 60123 is disabled), **no** `NET_ADMIN`
(a tunnel is an outbound QUIC/TCP connection), no raw socket and no host
network. Writes are limited to `/config`.

## 4. Least-privilege plan and the one feature that is root-only upstream

| Item | Setting in this package |
|---|---|
| User / Group | `1000:1000` (`node:node` inside the image) — pinned in compose |
| Capabilities | `cap_drop: [ALL]`, no `cap_add` |
| Escalation | `no-new-privileges:true` |
| Namespaces | bridge network, private PID/IPC, no host network, no docker.sock |
| Writable paths | `/config` → `/Volume1/DockerAppData/cloudflareddocker/config` only |
| Ports | host 8121 → container 14333 (both > 1024) |
| Image | fixed Docker Hub tag `wisdomsky/cloudflared-web:2026.5.0` (never `:latest`) |
| Secrets in package | none; the tunnel token is entered by the user at runtime |

**Root-only upstream feature, intentionally not used:** the upstream UI has an
“advanced” tab that stores a locally-managed tunnel's `config.yml` in
`/root/.cloudflared/`. As uid 1000 that path is not writable, so the endpoint
answers HTTP 400. This package does not advertise that tab: it packages the
**remotely-managed** tunnel workflow (token from the Cloudflare Zero Trust
dashboard, start/stop from the UI), which is fully functional as non-root. The
limitation is stated in `README.md` (§Limitations) and in `PRIVACY.md` (§5), so
the delivered functionality matches the description (review items C9/F5).

## 5. If the platform still requires a root exemption

No exemption is claimed or assumed by this package. Should the review process
treat the image's `USER root` metadata as the deciding factor, the following
inventory is provided so the platform can complete its own §7.1/§7.2 assessment:

- **Service inventory:** one container, `cloudflareddocker`, started by App
  Center from the compose file above (no systemd unit on the host). Functions:
  (a) web UI on 14333, (b) `cloudflared` tunnel child process. Communication:
  inbound LAN 8121 → 14333; outbound HTTPS/QUIC 443 to the Cloudflare edge.
- **Write paths:** `/Volume1/DockerAppData/cloudflareddocker/config` (read-write,
  application data: `config.json` with the tunnel token and start flag). Nothing
  else on the host is mounted read-write; nothing is mounted read-only.
- **Minimal settings:** User/Group `1000:1000`, `CapabilityBoundingSet` empty
  (`cap_drop: ALL`), `NoNewPrivileges=true`, no `privileged`, no host network,
  no docker.sock, single published port.
- **Hardening:** the tunnel is outbound-only; no inbound port is opened by the
  package beyond the web UI; logs are the container's stdout/stderr (audit
  trail); the write path is the allowlist above; backup/rollback = copy or
  delete `/Volume1/DockerAppData/cloudflareddocker/` and reinstall the previous
  package version; the user disable switch is the UI's Stop button or
  uninstalling the app.
- **Risk and mitigation:** the tunnel token is a scoped Cloudflare tunnel
  credential (it cannot administer the Cloudflare account). The upstream UI
  echoes it to the container log and passes it to `cloudflared` via argv, so
  `docker logs` / `docker inspect` access on the NAS should stay root-only —
  this is recorded in `PRIVACY.md` §5. The web UI has no built-in login unless
  `BASIC_AUTH_PASS` is set; the package documents keeping it on the LAN or
  behind a reverse proxy.

## 6. Files involved

- `src/docker-compose.yml` — `user`, `security_opt`, `cap_drop`, comments.
- `src/config.ini` — help link to the privacy policy.
- `PRIVACY.md` §5 — security measures and the root-metadata explanation.
- `README.md` — “Non-root” design note and limitations.
