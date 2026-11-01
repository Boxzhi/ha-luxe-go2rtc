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

## Compatibility and migration notes

This add-on intentionally mirrors the important runtime settings of the official AlexxIT go2rtc add-on:

- Host networking is enabled.
- The Home Assistant `/config`, `/media`, and `/ssl` folders are mapped.
- Audio and video device access is enabled.
- Ingress is exposed on port `1984`.
- go2rtc reads `/config/go2rtc.yaml`.

Only one go2rtc instance can run on the host at a time because both instances use the same host-network ports. Stop the official add-on before starting this one.

The add-on slug is different (`luxe_go2rtc`), but the WebRTC integration's default go2rtc endpoint (`http://localhost:1984/`) remains unchanged when host networking is used. The Ingress entry itself will belong to the new add-on.

Do not uninstall the official add-on until the new build has passed real-camera playback, audio, recording, and existing-stream regression tests.
