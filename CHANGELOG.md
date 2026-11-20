# Changelog

All notable user-visible changes to the Cloudflared Web TOS application package.

## 2026.5.0-3 — 2026-11-20

Rectified per review comments (review of 2026.5.0-2, aarch64).

### Added
- `PRIVACY.md`: privacy policy with data inventory, retention period, user
  rights and the deletion channel, security measures and the third-party API
  disclosure table (Cloudflare edge, Cloudflare dashboard, Docker Hub update
  check, Docker Hub image pull). Linked from the App Center help entry.
- `NOTICE.md`: third-party licences (Cloudflared-web MIT, `cloudflared`
  Apache-2.0, Node.js and npm dependencies), image digest and trademark notice.
- `docs/REMEDIATION.md`: code-by-code record of the review remediation.
- `docs/ROOT-PRIVILEGE.md`: S1/V1 analysis, measured non-root evidence, least
  privilege table and the platform-side exemption material.

### Changed
- Description and the 23-language `.lang` rewritten to match the delivered
  features (C9/F5): the “download the cloudflared client” claim is gone, the
  Chinese entries no longer describe an unrelated application, and the bridge
  network / DNS target caveat is documented.
- README restructured (features, repository layout, data, limitations, build).
- Container hardened (S1/T3): `cap_drop: [ALL]` and
  `security_opt: [no-new-privileges:true]` on top of `user: "1000:1000"`.

### Notes
- S12: build is deterministic and both `.sha256` assets are regenerated from
  the exact tarball bytes and verified after upload.
- T1: the image stays pinned to the reviewed tag `wisdomsky/cloudflared-web:2026.5.0`
  for this iteration; the upgrade to the current upstream `cloudflared`
  (2026.11.1) is scheduled for the next release together with its CVE
  remediation note.
- T2: no credential is shipped; the tunnel token is user-supplied at runtime.

## 2026.5.0-1 — 2026-11-12

### Added
- First release. Cloudflared Web (`wisdomsky/cloudflared-web:2026.5.0`, Docker Hub official image, unmodified)
  packaged as a TOS 7 Docker application, published on port 8121 and running
  as a non-root user (1000:1000).
- Persistent data is kept under `/Volume1/DockerAppData/cloudflareddocker/`.
