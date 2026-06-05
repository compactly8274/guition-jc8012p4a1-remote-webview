# Experiments

This folder contains ESPHome configurations that are not the recommended path.
Kept for reference; not maintained, not deployed.

## guition-jc8012p4a1-lvgl.yml

Native LVGL dashboard (3-column weather/indoor/control layout) running
entirely on-device. **Not working** — purple screen, I2C init error on
GSL3680 touch controller. See [the issue thread on GitHub] for diagnosis
history.

Use [`guition-jc8012p4a1-remote-webview.yml`](../guition-jc8012p4a1-remote-webview.yml)
instead.
