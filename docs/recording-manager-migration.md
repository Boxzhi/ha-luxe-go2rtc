# Luxe go2rtc — single-add-on recording migration

Development only. **Do not deploy to production yet.**

The Go recording-session API has been added to the Boxzhi/go2rtc
luxe/recording-manager-dev branch. It replaces the legacy recording
management endpoints from custom_components/luxe_xiaomi_camera; no GPAC
sidecar or Home Assistant Core custom integration is required.

## Runtime architecture

- The Add-on serves recording sessions from its native go2rtc MP4 Consumer.
- The Go manager stores only short-lived session metadata in memory.
- MP4 bytes stream directly through authenticated Ingress to the browser.
- The browser writes chunks to OPFS where supported, finalizes the recording,
  and downloads it to the current device.
- The Go Add-on does **not** write recordings to /media or cache complete
  recordings in server memory. No new /media:rw permission is required.
- No recording history or server-side file index is provided.

## Required Add-on settings

Retain the existing 1984 / 8554 / 8555 ports, host networking, Ingress,
homeassistant_api: true, and the existing HLC7 / CS2 / HVC1 build.

The Go manager validates the frontend's Home Assistant Bearer token against
http://homeassistant:8123/api/auth/current. If HAOS networking requires a
different internal Core URL, set it through the go2rtc configuration:

```yaml
luxe_recording:
  enabled: true
  auth_url: http://homeassistant:8123/api/auth/current
```

The manager exposes:
- GET /api/luxe/recordings
- POST /api/luxe/recordings/start
- GET /api/luxe/recordings/{id}
- GET /api/luxe/recordings/{id}/stream
- POST /api/luxe/recordings/{id}/stop

The dashboard must use the Add-on Ingress URL and send the current HA user's
Bearer token. The server accepts only configured go2rtc stream names.

## Fixed recording rules

- Maximum 3 simultaneous sessions.
- At most one active session per stream.
- Server-enforced maximum duration: 30 minutes.
- mp4=all requests supported camera audio independently of live mute state.
- Frontend downloads remain sequential.
- Completed session metadata expires; there is no server-side recording archive.

## Build and release

The current Add-on image 1.9.15-luxe.1 remains unchanged. The Dockerfile
still pins the previously tested HLC7 source commit. After the Go branch CI
passes and the HAOS Ingress/authentication path is verified, update
GO2RTC_REF to the exact verified recording-manager commit, then build a
separate test image. Do not publish a production Release at this stage.
