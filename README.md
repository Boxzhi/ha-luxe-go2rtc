# Luxe go2rtc Add-on

Home Assistant add-on packaging for the Luxe HLC7-enabled go2rtc build.

## Build source

The container build is pinned to go2rtc fork commit:

`d80b1a584a12dac318baa8316eb6ef975e645649`

Source branch: [Boxzhi/go2rtc `luxe/hlc7`](https://github.com/Boxzhi/go2rtc/tree/luxe/hlc7)

The add-on image is built for `linux/amd64`, matching the current Intel-based Home Assistant host.

## Automatic build

Pushing to `main` or manually running the GitHub Actions workflow builds and publishes:

`ghcr.io/boxzhi/ha-luxe-go2rtc-amd64:1.9.15-luxe.1`

After the first successful build, open the package in GitHub Packages and change its visibility to **Public**. Home Assistant must be able to pull the image without registry credentials.

## Install in Home Assistant

1. Add this repository in **Settings → Add-ons → Add-on Store → ⋮ → Repositories**:
   `https://github.com/Boxzhi/ha-luxe-go2rtc`
2. Wait for the `Luxe go2rtc (HLC7)` add-on to appear.
3. Back up the existing official go2rtc add-on and its configuration.
4. Stop the official go2rtc add-on. Keep it installed and stopped for rollback.
5. Install and start `Luxe go2rtc (HLC7)`.

## Recording download POC

A separate **Luxe go2rtc Recording POC** add-on is provided for validating native browser downloads without replacing the stable add-on.

- Separate add-on slug: `luxe_go2rtc_recording_poc`.
- Uses the CI-verified go2rtc recording-manager commit `b093ef887713e1bd095754aaecae9bf3631d2e7f`.
- Connects to the existing stable add-on's three RTSP stream names through loopback port `8554`.
- Listens only on API port `1985`; its RTSP and WebRTC servers are disabled to avoid port conflicts.
- Opens the separate `/recording-poc.html` page through its own Ingress panel.
- Does not change the stable add-on image, config, or Release.
- Does not write recordings to `/media`, disk, Blob, or OPFS.

See [the POC add-on instructions](luxe-go2rtc-recording-poc/README.md). The POC is a test build, not a production Release.

## Compatibility and migration notes

This add-on intentionally mirrors the important runtime settings of the official AlexxIT go2rtc add-on:

- Host networking is enabled.
- The Home Assistant `/config`, `/media`, and `/ssl` folders are mapped.
- Audio and video device access is enabled.
- Ingress is exposed on port `1984`.
- go2rtc reads `/config/go2rtc.yaml`.

Only one full go2rtc server instance can run on the host at a time because the stable and official instances use the same host-network ports. The isolated Recording POC is an exception: it binds only API port `1985` and disables its RTSP and WebRTC listeners, so it can run alongside the stable add-on.

The add-on slug is different (`luxe_go2rtc`), but the WebRTC integration's default go2rtc endpoint (`http://localhost:1984/`) remains unchanged when host networking is used. The Ingress entry itself will belong to the new add-on.

Do not uninstall the official add-on until the new build has passed real-camera playback, audio, recording, and existing-stream regression tests.
