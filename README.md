# Waveshare ESP32-S3 Touch LCD 2.1 - ESPHome Configuration

This repository contains a heavily refined, battle-tested ESPHome configuration for the **Waveshare ESP32-S3-Touch-LCD-2.1B** display. 

The default ESPHome `st7701s` configurations do not fully support the hardware quirks of this specific 2.1-inch round IPS display, often resulting in PSRAM crashes, inverted colors, or severe vertical bleeding (crosstalk). This configuration has been systematically tuned to solve all of those issues and provides a stable, hardware-accurate foundation for building beautiful LVGL dashboards.

## Features & Fixes Included

- **Native Octal PSRAM Configuration**: The ESP32-S3R8 on this board utilizes internal octal PSRAM. Relying on default `quad` settings triggers `ESP_ERR_NO_MEM` crashes when LVGL attempts to allocate framebuffers. This config maps it correctly.
- **Physical BGR Byte-Swapping Fix**: The physical wiring of the data pins on this board relies on the ESP32's DMA transferring colors in big-endian format. The RGB data pins are explicitly nested here so ESPHome compensates for this byte-swap automatically, preventing a global green/purple tint.
- **Hardware Crosstalk Elimination (INVON)**: Cheap IPS panels often suffer from "bleeding" or "ghosting" when rendering white UI elements on a dark background. This configuration injects the native Waveshare `0x21` (Display Inversion ON) hardware command and couples it with ESPHome `invert_colors: false` to perfectly balance the VCOM cell voltages and eliminate vertical smearing.
- **Tuned Pixel Clock**: The pixel clock is deliberately stabilized at `12MHz` to give the physical traces enough time to discharge between high-contrast pixel boundaries.

## Usage

1. Clone or download this repository.
2. Ensure you have ESPHome installed (or use the Home Assistant ESPHome Add-on).
3. If necessary, adjust the `wifi:` and `api:` blocks in `waveshare_2_1b.yaml` to match your network credentials.
4. Flash the device via USB the first time:
```bash
esphome run waveshare_2_1b.yaml
```
*(Note: If the screen remains blank immediately after the first USB flash completes, unplug the USB cable and plug it back in. The ESP32 soft-reboot does not power-cycle the LCD driver IC, which can cause the display to desync until hard-rebooted).*

## Customization

The configuration includes a basic LVGL Home Assistant dashboard. To modify the UI, edit the `lvgl:` block at the bottom of the yaml file. To hide any final faint traces of ghosting on this specific panel glass, it is highly recommended to use slightly off-white text (e.g. `0xD0D0D0` or `0x888888`) rather than pure white (`0xFFFFFF`) when placing text over pitch-black backgrounds.
