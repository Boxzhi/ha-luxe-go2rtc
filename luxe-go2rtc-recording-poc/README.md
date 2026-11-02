# Luxe go2rtc Recording POC

**Test add-on only.** This is a separate add-on with its own slug. It does not replace or update the stable `Luxe go2rtc (HLC7)` add-on.

## What it tests

- Native go2rtc MP4 Consumer recording.
- Native browser attachment download through Add-on Ingress.
- Three simultaneous recording sessions and three simultaneous downloads.
- Independent stop and failure handling for each camera.
- Supervisor-authenticated Ingress identity and path-scoped download cookies.
- H.264 / H.265 camera streams with supported camera audio.

The POC is built as a native Go binary and does not install or invoke FFmpeg or GPAC. The existing camera-download test does not use Blob, OPFS, a JavaScript media buffer, a server-side recording directory, or the Home Assistant `/media` folder. A separate client-only OPFS POC is available from the recording page; it writes synthetic test data only to the current browser device and streams it back through a Service Worker and a Dedicated Worker using MessagePort. No File or Blob object is created.

## Isolation from the stable add-on

The POC add-on:

- Uses the separate slug `luxe_go2rtc_recording_poc` and does not mount the stable add-on's `/config` or `/media` directories.
- Binds only the go2rtc API to host port `1985`.
- Disables its RTSP server and WebRTC listener, so it does not collide with the stable add-on's ports `1984 / 8554 / 8555`.
- Reads the existing stable add-on's RTSP streams through `127.0.0.1:8554`.
- Does not mount `/media` and does not modify the stable add-on's config or image.

The stable add-on must remain running while this POC is used.

## Install and test

1. Refresh **Settings → Add-ons → Add-on Store** for the existing repository `https://github.com/Boxzhi/ha-luxe-go2rtc`.
2. Install **Luxe go2rtc Recording POC**. Do not stop or uninstall the stable add-on.
3. Start the POC add-on and open its Ingress panel. The panel opens `recording-poc.html` directly through `ingress_entry: recording-poc.html` (without a leading slash, because Supervisor appends the value to the ingress URL); do not append a path to the `/app/<slug>` Home Assistant panel URL.
4. On the recording page, open **打开 OPFS「停止后下载」独立测试页** to test browser-local OPFS and Service Worker downloads. This test uses synthetic `.bin` files, not camera video.
5. For each camera, click **创建任务**, then **开始下载**. Repeat for all three cameras; the three download responses are independent and may run simultaneously.
6. Use **停止录像** on one row and verify that the other two continue.
7. Test camera MP4 files with `ffprobe`, QuickTime, Safari, and the Home Assistant iOS App.
8. In the OPFS POC, manually remove test files only after confirming the downloaded copies are saved. The Service Worker stream reaching EOF is not a guaranteed OS file-saved callback.

Default stream aliases:

- `jinghuafu_keting_ptz_3k`
- `jinghuafu_keting_fixed_3k`
- `jinghuafu_canting_2k`

These aliases point to the corresponding RTSP stream names on the stable add-on. If those stream names or the stable RTSP port change, update the POC Dockerfile command arguments.

## Build

The image is built from the CI-verified go2rtc commit:

`f9ed2c41c767cbd0adbb46da20f8fd339985c2fe`

The POC workflow publishes only the separate image:

`ghcr.io/boxzhi/ha-luxe-go2rtc-recording-poc-amd64:1.9.15-luxe-poc.6`

If GitHub Packages marks the image private after the first build, change only this POC package to **Public** so Home Assistant can pull it.

**No production Release is created by this POC.**
