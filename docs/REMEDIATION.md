# Review remediation record — `cloudflareddocker` 2026.5.0-3

Source: TerraMaster App Listing Review On-Hold notice for **Cloudflared Web
2026.5.0-2 (aarch64)** — “App Listing Review On-Hold Notice (Root Privilege
Exemption Available)”, 20 Nov 2026. Scope of this record: **aarch64** (the same
package is also built for x86_64; the x86_64 build was conditionally approved
and is superseded by the same fixes).

Package version: `2026.5.0-2` → **`2026.5.0-3`** (packaging iteration; the
upstream image stays pinned at `wisdomsky/cloudflared-web:2026.5.0`).

| Code | Review item | What was wrong | What changed in 2026.5.0-3 | Where |
|---|---|---|---|---|
| **C1** | Content legality could not be fully verified | Review could not verify content legality; follow-up, no re-submission required | Recorded: the package is a tunnel client, the image is the unmodified official Docker Hub image, and the package itself contains no content library, media or user-generated content. No illegal, violent, pornographic or gambling content is delivered or linked. | this file, §1 |
| **C10** | Repository / directory compliance | README thin, no documented layout, no docs directory | README rewritten (features, layout, data, security, limitations); repository layout table added; `docs/` added with this record and the privilege analysis; `PRIVACY.md` / `NOTICE.md` / `CHANGELOG.md` linked. Repository is public. | `README.md`, `docs/` |
| **C2** | Intellectual property / licence notices | No licence or trademark notice file | `NOTICE.md` added: upstream Cloudflared-web (MIT), `cloudflared` (Apache-2.0), Node.js and the locked npm dependencies, base image, this packaging (MIT) and the Cloudflare / TerraMaster / Docker / Node trademarks (descriptive use only, no affiliation). | `NOTICE.md`, `README.md` §Licence |
| **C3** | Privacy policy missing | No privacy policy at all | `PRIVACY.md` added (controller, data inventory, storage paths, retention, rights, security, third-party APIs, children, changes, contact). | `PRIVACY.md` |
| **C4** | Privacy policy incomplete | – | The policy now covers the five required elements: **retention period** (§3), **user rights** (§4), **security measures** (§5), **third-party sharing** (§6) and the **deletion channel** (§4: UI Stop / delete token, uninstall with “delete data”, remove the data folder, e-mail `moechz@163.com`). | `PRIVACY.md` §3–§6 |
| **C5** | Privacy policy not accessible | Policy not reachable from the app | `config.ini` `help` now points at the public policy: <https://github.com/Moechz/cloudflared-web/blob/main/PRIVACY.md> (App Center “Help” opens it). The same URL is linked from the README, and the file ships in the repository. | `src/config.ini`, `README.md` |
| **C6** | Data collection inconsistent with the declaration | Nothing was declared | The policy declares exactly what happens at runtime: `config/config.json` holds the tunnel token and the start flag; container logs hold startup/tunnel state (and the upstream UI's token echo); traffic is proxied, not stored; the only outbound calls are Cloudflare, the Docker Hub update check and the Docker Hub image pull. | `PRIVACY.md` §2, §6 |
| **C7** | User data rights channel not provided | No channel described | §4 of the policy gives access/export, correction, deletion and an e-mail channel (`moechz@163.com`, 30-day reply), and explains that the packager holds no copy of the data because it all stays on the NAS. | `PRIVACY.md` §4 |
| **C8** | Third-party APIs not disclosed | Undisclosed endpoints | Disclosure table with endpoint, trigger, data sent and destination/region for: Cloudflare global network (`cloudflared tunnel run`, `*.argotunnel.com`, US/edge-chosen), the Cloudflare Zero Trust dashboard/API (browser only, app never calls it), the Docker Hub registry API used by the UI's update check (`registry.hub.docker.com`, anonymous, US), the Docker Hub image pull (`registry-1.docker.io` or mirror) and the services the user publishes. No e-mail/Telegram/webhook notification is used by this package. | `PRIVACY.md` §6 |
| **C9** | Description inconsistent with delivered functionality | The 23-language `.lang` carried the wrong product text (Chinese entries described a habit tracker), and the English text promised a “download the cloudflared client” feature that the UI does not have | `tosapp.json` descriptions rewritten to the actual feature set (paste a remote tunnel token, start/stop, status, config), `.lang` re-rendered for all 23 languages, `zh-cn`/`zh-hk` properly translated (previously an unrelated app's text), `release_note` describes the remediation. | `tosapp.json`, `src/cloudflareddocker.lang` |
| **F5** | Declared functionality not fully usable | See C9; also the bridge network / root-only upstream tab were never documented | Description now matches the delivered features. Measured on hardware as uid 1000: UI serves HTTP 200, token save → `OK!`, start → `cloudflared` spawns, config read-back works. The upstream “advanced local config” tab writes `/root/.cloudflared` and is root-only, so it is **not** part of the declared feature set and is listed as a limitation; the bridge-network ingress caveat (use the NAS LAN IP, not `127.0.0.1`) is documented in `important` and the README. | `README.md` §Limitations, `docs/ROOT-PRIVILEGE.md` §4 |
| **S1** | Service does not run as a non-root user | Reviewer item; the upstream image declares `USER root` | The service already ran as `uid 1000`; it is now pinned and hardened: `user: "1000:1000"`, `cap_drop: [ALL]`, `security_opt: [no-new-privileges:true]`, no privileged/host-network/socket. Measured: `id` → `uid=1000(node)`, UI HTTP 200, tunnel start works. No root exemption is required at runtime; full analysis and the platform-side exemption material (if the scanner keys on image metadata) are in `docs/ROOT-PRIVILEGE.md`. | `src/docker-compose.yml`, `docs/ROOT-PRIVILEGE.md` |
| **S12** | Package checksum inconsistent at runtime | Follow-up item, no re-submission required | The build is deterministic (GNU tar, uid/gid 0, mtime 0, gzip without timestamp) and both `.sha256` assets are regenerated from the exact tarball bytes on every build. Verified for the previous release: the aarch64 asset hashes to `5b9ed127…0dbf6b` and the x86_64 asset to `5f866abb…c62ff8a8`, and the platform version record for the aarch64 submission carries exactly `5b9ed127…0dbf6b`. `config.ini`'s `md5` field stays empty by design — the platform writes it at install time. The 2026.5.0-3 assets are rebuilt and re-verified before upload; the release publishes `<appid>_<arch>.tar.gz` and its matching `.sha256` from the same build. | `scripts/build.sh`, release assets |
| **T1** | High/critical CVE detected | Image 2026.5.0 is old (Node 18 base, `cloudflared` 2026.5.0) | The image stays pinned to the reviewed tag in this packaging-only iteration; the vulnerability report was not attached to the notice, so no component can be singled out here. Upstream has since moved to `cloudflared` 2026.11.1 (upstream Dockerfile, Nov 2026): the next release upgrades the image tag and carries the CVE remediation note, which keeps the change reviewable in isolation. No code in this package adds to the CVE surface (four metadata files, unmodified image). | `README.md` §Known limitations, CHANGELOG |
| **T2** | Hard-coded credentials detected | – | The package contains no credential: no password, token or API key in `config.ini`, the compose file, the `.lang` or the SVG. The tunnel token is supplied by the user at runtime and stored only in `/config/config.json`. Disclosed for transparency: the upstream UI echoes the token into the container log and passes it to `cloudflared` as an argument, so log/`inspect` access must stay restricted (documented in `PRIVACY.md` §5). | `PRIVACY.md` §5 |
| **T3** | Configuration errors detected | Root / missing security settings | `cap_drop: [ALL]` and `no-new-privileges:true` added; the compose keeps a health check, explicit `TZ`, `restart: unless-stopped`, a fixed tag, one published port and one writable data path. The root part of T3 is the same item as S1/V1 and is handled in `docs/ROOT-PRIVILEGE.md`. | `src/docker-compose.yml` |
| **V1** | Application runs as root | Same finding as S1 | See S1 — runtime user is 1000, not root. | `docs/ROOT-PRIVILEGE.md` |

## 1. C1 — content legality statement

The application is a network utility: it packages the official Docker Hub image
of Cloudflared Web, whose only functions are a token form, a start/stop button
and a status display for a Cloudflare Tunnel. The package does not ship, host,
index or link to media libraries, user-generated content, gambling, adult
content or violence, and it does not operate any content service. The upstream
image is used unmodified and is published on Docker Hub by its author.

## 2. Verification evidence for this version

| Gate | Result |
|---|---|
| `scripts/build-dual.sh 2026.5.0-3` | self-check + both architecture packages |
| `TOSAppSelfTestingTool -c out/cloudflareddocker_x86_64.tar.gz` | check passed |
| `TOSAppSelfTestingTool -c out/cloudflareddocker_aarch64.tar.gz` | check passed |
| `python3 stage.py work/cloudflared-web --timeout 300` | isolated compose up, healthy, HTTP, uid 1000, clean teardown |
| Release checksum | `.sha256` regenerated from the exact tarball bytes, verified anonymously after upload |
| Platform submission | aarch64 2026.5.0-3 submitted with “Rectified per review comments” |
