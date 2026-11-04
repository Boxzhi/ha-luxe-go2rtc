# Changelog

## 1.9.15-luxe.2 — 2026-11-04

- Integrated the Luxe Xiaomi Camera RC4 server-side recording module into the official go2rtc image.
- HLC7 HD video negotiation, CS2 payload fixes, and HEVC/MP4 compatibility retained.
- Up to three concurrent independent MP4 recordings, each up to 30 minutes, with separate temporary 15-minute TTL.
- Server-side status, stop, heartbeat, abort, download and presented-cache APIs; supporting native iOS downloads and multi-file Web Share.
- Official HA iOS App background behavior: abort and discard unfinished recordings with a 3-second frontend heartbeat, a 12-second server lease, and a 2-second cleanup poll.
- The production `luxe-camera-wall.js` v1.1.0 now embeds recording controls in one card (no separate `luxe-camera-wall-recording-rc.js`).
- Retains first-frame UI gating: while video says “正在连接视频...”, screenshot, recording, and native control buttons cannot be displayed or clicked.

**Setup:** Recording is opt-in. After upgrading, add a secure `luxe_recording:` block in `go2rtc.yaml` and upload the latest `luxe-camera-wall.js` to your Lovelace resource folder. See [documentation](https://github.com/Boxzhi/go2rtc/blob/luxe/hlc7/docs/luxe-camera-wall-official-upgrade-1.9.15-luxe.2.md).

## 1.9.15-luxe.1

- Initial stable Luxe fork: HLC7, CS2 and MP4 compatibility patches.
