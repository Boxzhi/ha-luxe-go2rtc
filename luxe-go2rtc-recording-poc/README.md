# Luxe go2rtc Recording POC

**Test add-on only.** This is a separate add-on with its own slug. It does not replace or update the stable `Luxe go2rtc (HLC7)` add-on.

## What it tests

- Native go2rtc MP4 Consumer recording.
- Native browser attachment download through Add-on Ingress.
- HA iOS App WKWebView handoff to its native WKDownload manager (separate synthetic attachment test).
- Three simultaneous recording sessions and three simultaneous downloads.
- Independent stop and failure handling for each camera.
- Supervisor-authenticated Ingress identity and path-scoped download cookies.
- H.264 / H.265 camera streams with supported camera audio.

The POC is built as a native Go binary and does not install or invoke FFmpeg or GPAC. The existing camera-download test does not use Blob, OPFS, a JavaScript media buffer, a server-side recording directory, or the Home Assistant `/media` folder. A separate OPFS POC writes synthetic test data only to the current browser device. Its browser-only path uses Service Worker and Dedicated Worker MessagePort streaming; the HA iOS App integration path instead uses Dedicated Worker HTTP chunks and a bounded in-memory go2rtc pipe. Neither path creates a File or Blob object or persists video on the server.

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
4. For the HA iOS App WKDownload test, open **打开 OPFS → WKDownload 集成测试页**. For the App Store Safari test, open **在 Safari 中测试三路独立下载** from the HA App page; the link opens Safari in a new browser context. Create the A/B/C test files inside Safari itself: HA App WKWebView OPFS and Safari OPFS are isolated and cannot share those files.
5. For each camera, click **创建任务**, then **开始下载**. Repeat for all three cameras; the three download responses are independent and may run simultaneously.
6. Use **停止录像** on one row and verify that the other two continue.
7. Test camera MP4 files with `ffprobe`, QuickTime, Safari, and the Home Assistant iOS App.
8. In the HA App and Safari OPFS POCs, manually remove test files only after confirming the downloaded copies are saved in the corresponding native download manager. A completed stream or CRC check is not itself an operating-system file-saved callback.

Default stream aliases:

- `jinghuafu_keting_ptz_3k`
- `jinghuafu_keting_fixed_3k`
- `jinghuafu_canting_2k`

These aliases point to the corresponding RTSP stream names on the stable add-on. If those stream names or the stable RTSP port change, update the POC Dockerfile command arguments.

## HA iOS App WKDownload POC

Open `/recording-wkdownload-poc.html` from the OPFS POC page. This separate page verifies whether the Home Assistant iOS App's WKWebView hands a user-initiated same-origin HTTP attachment navigation to the app's native WKDownload manager.

- The endpoint is `GET /api/luxe/recording-poc/download` and requires the authenticated Supervisor Ingress identity (or a valid Home Assistant Bearer token for direct requests).
- It generates a deterministic 16 MiB `.bin` payload in 64 KiB chunks and returns `Content-Disposition: attachment`, an exact `Content-Length`, and an `X-Luxe-POC-CRC32` header.
- It uses one bounded 64 KiB scratch buffer per request. It does not create a server-side file, write to NAS or Home Assistant `/media`, or retain the full payload in server memory.
- It does not connect to a camera, call the Recording Manager, or use Blob, OPFS, Service Worker, or a JavaScript media buffer.
- In the HA iOS App, tap **下载 16 MiB 测试文件**. Success means the app's native download manager appears and shows the filename/progress. After completion, use its native share action to save the file to the Files app.
- This test validates only the WKDownload attachment handoff. It does not yet implement or validate OPFS-to-go2rtc streaming.

## OPFS → WKDownload integration POC

Open `/recording-opfs-wkdownload-poc.html` from the OPFS POC page. This is the next-stage integration test for the HA iOS App WKWebView, where Service Worker is unavailable.

- The page scans the current device's OPFS for the synthetic A/B/C files created by `recording-opfs-poc.html`.
- Click **创建中转任务**, then **开始原生下载**. The page first triggers a same-window attachment navigation (matching the WKDownload test already confirmed on the HA iOS App), waits for go2rtc to confirm that the download GET has arrived, and only then starts the OPFS uploader.
- A Dedicated Worker reads the OPFS file in 256 KiB chunks and sends sequential same-origin HTTP POST requests. It does not construct Blob/File objects, Object URLs, or a complete in-memory file.
- go2rtc connects each upload to its own unbuffered `io.Pipe`; the WKDownload attachment GET drains the other end. The pipe applies backpressure: each upload chunk is not acknowledged until the download side consumes it.
- The relay has a three-session limit, independent pipes, strict chunk offsets, exact byte counts, CRC32 verification, authenticated Ingress ownership, and a 10-minute expiry.
- The server does not create a recording file, write to NAS, Home Assistant `/media`, or go2rtc disk. It does not retain the complete video in RAM; only bounded HTTP/pipe buffers are used.
- The POC first tests one 16 MiB synthetic file, then two concurrent files, then all three. The page reports server-side CRC32 and the number of bytes written to the attachment response. Confirm the iOS native download manager also shows **下载已完成** before cleaning OPFS.
- This validates the OPFS-to-WKDownload relay only. It does not yet connect real camera MP4 data or modify the formal Luxe camera UI.

## App Store Safari three-file download POC

Open **在 Safari 中测试三路独立下载** from the HA App recording POC page. The link opens `/recording-opfs-safari-poc.html` in Safari. This is a separate Safari-only test; it does not modify the HA App's WKDownload manager.

- The HA iOS App's WKWebView and Safari use separate browser storage contexts. Create the synthetic A/B/C files inside Safari; files previously written inside the HA App OPFS are not visible to Safari.
- Confirm the page identifies the browser as Safari and reports secure context, OPFS, Dedicated Worker and Service Worker as available.
- Click **三路并发写入** to create three independent 16 MiB synthetic files in Safari OPFS.
- Tap A's **下载文件**, then B's, then C's. Each tap is a separate user gesture. Do not use an automatic multi-click script. If Safari asks whether to allow multiple downloads from this website, allow it.
- Keep the Safari page open and inspect Safari's Downloads list. The acceptance target is three separate files, each completing independently; no ZIP is created.
- The page streams each OPFS file through Service Worker + MessageChannel + Dedicated Worker in bounded 256 KiB chunks. It does not create Blob/File objects, Object URLs, or a whole-file JavaScript buffer. It does not upload video to go2rtc and does not write to NAS or Home Assistant storage.
- After Safari's Downloads list confirms each file is saved, clean up each corresponding OPFS temporary file. A completed Service Worker response is not itself an OS file-saved callback.
- This POC validates synthetic file download only. It does not yet validate real camera MP4, three 30-minute recordings, background execution, or downloads after the device is locked. Those require separate real-device tests.

## Build

The image is built from the CI-verified go2rtc commit:

`7ea764436a1367b337a75f4dbd7c3a9738d2d8c9`

The POC workflow publishes only the separate image:

`ghcr.io/boxzhi/ha-luxe-go2rtc-recording-poc-amd64:1.9.15-luxe-poc.10`

If GitHub Packages marks the image private after the first build, change only this POC package to **Public** so Home Assistant can pull it.

**No production Release is created by this POC.**
