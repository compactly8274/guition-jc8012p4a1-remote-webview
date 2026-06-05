# guition-jc8012p4a1-remote-webview

A Home Assistant remote-display for the Guition JC8012P4A1 10.1" smart screen
(ESP32-P4 main MCU + ESP32-C6 WiFi/BT coprocessor). The panel renders a live
view of your HA dashboard via the [RemoteWebView](https://github.com/strange-v/RemoteWebView)
protocol — all UI logic stays in HA, the panel just streams tiles.

## Hardware

- **Board**: Guition JC8012P4A1 (10.1" 800×1280 IPS MIPI DSI panel)
- **Main MCU**: ESP32-P4NRW32 (dual-core RISC-V, 400 MHz, 32 MB PSRAM, 16 MB Flash)
- **WiFi/BT**: ESP32-C6-MINI-1U-N4 coprocessor over SDIO (`esp32_hosted`)
- **Touch**: GSL3680 capacitive (I2C on GPIO7/8, INT GPIO21, RST GPIO22)
- **Backlight**: GPIO23 via LEDC

## Files

| File | Purpose |
|---|---|
| `guition-jc8012p4a1-remote-webview.yml` | **Primary config** — the working deployable file |
| `ha-pirateweather-forecast-template.yaml` | Optional HA template sensor for a 7-day forecast summary (only used by the LVGL experiment) |
| `experiments/` | Older / non-working configs preserved for reference |

## Deploying the webview config

### 1. Install ESPHome

The standalone CLI works fine:

```bash
pip install esphome
```

Or use the [Home Assistant ESPHome add-on](https://www.home-assistant.io/integrations/esphome/).

### 2. Set up `secrets.yaml`

Create or edit `secrets.yaml` in the same directory you'll run `esphome` from:

```yaml
wifi_ssid: "your-wifi-ssid"
wifi_password: "your-wifi-password"
ap_password: "fallback-ap-password"   # used for the captive-portal fallback AP
ota_password: "your-ota-password"     # used for over-the-air updates
```

### 3. Edit the YAML

Open `guition-jc8012p4a1-remote-webview.yml` and replace the placeholders:

- **`api.encryption.key`** — must be a 32-byte base64 string. Generate one:

  ```powershell
  [Convert]::ToBase64String((1..32 | ForEach-Object { Get-Random -Maximum 256 }) -as [byte[]])
  ```

  The same key must be configured in Home Assistant when the device first
  appears under *Settings → Devices & Services → Discovered*.

- **`rwv_server_host`** in the `substitutions:` block — the IP or hostname
  of the machine running the [RemoteWebView server](https://github.com/strange-v/RemoteWebView).
  This is typically your HA server.

- **`use_address`** in the `wifi:` block — the static IP you want the device
  to have on your network (used for OTA).

The `comment:` field at the top is a free-text description and can be
anything you like.

### 4. Start the RemoteWebView server

The panel can't connect to HA directly; it needs the
[RemoteWebView server](https://github.com/strange-v/RemoteWebView) running
on a machine reachable on your LAN. The default port (8081) is what the
YAML expects.

### 5. Flash

Plug the panel in via USB. Find the COM port in Device Manager
(under *Ports (COM & LPT)*), then:

```bash
esphome run guition-jc8012p4a1-remote-webview.yml --device COM3
```

(Replace `COM3` with the actual port.)

The first flash takes a few minutes (IDF build). After that, OTA updates
work from the HA frontend or via `esphome run --device <ip>`.

## Touch calibration

If touches feel mirrored or rotated wrong, edit the `transform:` block in
the `touchscreen:` section. See the comments in the YAML for the rationale.
Toggling `mirror_x`, `mirror_y`, or `swap_xy` one at a time and re-flashing
is the fastest way to find the right combination for your panel revision.

## Notes

- The 3 AM daily auto-restart in the YAML is intentional — it clears any
  state drift that accumulates over a day of continuous operation (ghost
  touches, websocket stalemates, RAM creep). Remove it if you don't want it.
- `auto_clear_enabled: false` on the display is required when binding
  `remote_webview` (or LVGL) to the display — the framework manages its own
  buffer clears.
- `power_save_mode: NONE` on WiFi is required for the P4+C6 SDIO link —
  power saving causes packet loss during association.

## License

Personal project. Do what you want with it.
