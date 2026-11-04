# Luxe go2rtc (HLC7) — 1.9.15-luxe.2

This version upgrades the existing official add-on (slug `fee2ccf4_luxe_go2rtc`) without requiring the standalone Recording RC add-on. It is compatible with the existing `/config/go2rtc.yaml` streams and host-network ports.

## Activate server-side recording

In `/config/go2rtc.yaml`, add alongside existing top-level settings:

```yaml
luxe_recording:
  enabled: true
  secret: "REPLACE_WITH_AT_LEAST_24_RANDOM_CHARACTERS"
  directory: /data/luxe-recordings
```

Generate a secret locally (for example, `openssl rand -hex 24`). Never publish it in screenshots, GitHub or Lovelace dashboard YAML. Restart the add-on after editing. Log should include `Luxe recorder enabled`. The server remains opt-in if the block is absent.

Update the Home Assistant JS resource by replacing `/config/www/luxe-camera-wall.js` with the [single production file](https://github.com/Boxzhi/go2rtc/blob/luxe/hlc7/www/luxe-camera-wall.js) and cache-bust with `?v=1.1.0-luxe.2`. Continue using `type: custom:luxe-camera-wall`. On the first recording action, input the same secret; it is held in card memory only.

Remove the *old resource entry* for `luxe-camera-wall-recording-rc.js`; delete its local file and stop the independent RC add-on only after the new card is verified.

Temporary files are kept on the add-on's own persistent `/data` until the original 15-minute expiry or the next successful same-camera recording. This does **not** delete MP4s saved to the iPhone Files App.

For detailed upgrade and rollback steps, see [full release instructions](https://github.com/Boxzhi/go2rtc/blob/luxe/hlc7/docs/luxe-camera-wall-official-upgrade-1.9.15-luxe.2.md).
