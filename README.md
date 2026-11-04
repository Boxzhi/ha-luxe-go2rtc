# Luxe go2rtc — official Home Assistant add-on

Home Assistant packaging for Boxzhi's HLC7-enabled go2rtc fork with optional server-side Xiaomi Camera recording.

## Stable version

**1.9.15-luxe.2**. The image is built from the exact merged go2rtc source commit:

`443f0f93222e8ba6013e42b0d0c75def516ae8b3`

Source: [Boxzhi/go2rtc · luxe/hlc7](https://github.com/Boxzhi/go2rtc/tree/luxe/hlc7)

The official image is published to:

`ghcr.io/boxzhi/ha-luxe-go2rtc-amd64:1.9.15-luxe.2`

This repository provides the update of **Luxe go2rtc (HLC7)** from `1.9.15-luxe.1` without changing its add-on slug or the existing host-network ports.

## Installation or upgrade

Install this repository as a Home Assistant add-on repository:

`https://github.com/Boxzhi/ha-luxe-go2rtc`

Existing installations receive the **1.9.15-luxe.2** update once the new container image is published and the add-on repository is refreshed.

After upgrade, **recording is disabled by default until configured**. In `/config/go2rtc.yaml` add:

```yaml
luxe_recording:
  enabled: true
  secret: "LONG_RANDOM_SECRET_AT_LEAST_24_CHARACTERS"
  directory: /data/luxe-recordings
```

Save, restart the add-on, then upload the new single [`luxe-camera-wall.js`](https://github.com/Boxzhi/go2rtc/blob/luxe/hlc7/www/luxe-camera-wall.js) to `/config/www/`. The existing Lovelace `custom:luxe-camera-wall` card type remains unchanged.

Remove the old `luxe-camera-wall-recording-rc.js` resource after verifying the integrated playback/recording UI. **The separate RC add-on is no longer required.**

See [add-on documentation](luxe-go2rtc/DOCS.md) and [changelog](luxe-go2rtc/CHANGELOG.md) for full details.

## Compatibility and rollback

Existing HLC7, CS2, HEVC, audio, video and RTSP fixes are retained. The add-on uses the original `/config/go2rtc.yaml`, host-network `1984 / 8554 / 8555` and the same Ingress path.

Back up `go2rtc.yaml` and `luxe-camera-wall.js` before upgrade. The old `1.9.15-luxe.1` image is independent of the new release and may be used for manual rollback if retained. Do not stop a production stream before the new image build is available.

The earlier standalone Recording POC folder is historical development material, not a requirement for this stable release.
