# Luxe go2rtc — single-add-on recording migration

Development design only. **Do not deploy to production yet.**

The parent Add-on will incorporate the recording management API implemented
inside the `Boxzhi/go2rtc` fork, replacing the old GPAC sidecar and the
`luxe_xiaomi_camera` Home Assistant Core integration.

- Map `/media` with **write** permissions: `media:rw`.
- Store recordings at `/media/luxe_camera_recordings`.
- Retain the current 1984 / 8554 / 8555 ports, three camera sources,
  HLC7 parsing, CS2 fixes and the hvc1 packaging implementation.
- Enable the manager only in the Luxe Add-on build, not all go2rtc builds.
- Authenticate the browser-facing API via HA access-token verification.
- Make the dashboard API route work through an authenticated HA Ingress or
  an equivalent secure endpoint; **do not rely on the unprefixed Add-on slug**.
- Enforce 30-minute max and 3 parallel recordings; sequential downloads.
- Keep the existing `1.9.15-luxe.1` Docker tag/image unchanged until tests
  pass and a new test version is explicitly published.

This branch is only a preparation point for source integration and CI.
