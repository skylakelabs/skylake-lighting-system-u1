# Skylake Lighting System for Snapmaker U1

Tested WLED integration for the Skylake LED Top Panel Kit. The stock U1 cavity-light control remains the master switch: off turns WLED off, and on loads WLED preset 1.

This README is the abbreviated procedure for users already comfortable with Fluidd and Klipper configuration. The complete hardware and beginner-focused walkthrough is maintained on the Skylake website.

> [!WARNING]
> This configuration transfers MCU pin `PE15` from the stock purifier to `skylake_24v`. Never let both objects own `PE15`. Back up `printer.cfg` and `moonraker.conf`, make no wiring changes while powered, and do not perform this work during a print.

## Files

| File | Purpose |
| --- | --- |
| [`skylake-wled.cfg`](skylake-wled.cfg) | Enables the top-cover 24 V rail and mirrors the cavity-light state to WLED. |
| [`skylake-moonraker.conf`](skylake-moonraker.conf) | Defines the WLED connection, preset, and 20-LED chain. |
| [`LEDTopPanel-SkylakeLabs-1.7.6.3mf`](models/LEDTopPanel-SkylakeLabs-1.7.6.3mf) | Prepared three-plate project with Snapmaker U1 slicer and support settings. |
| [`SnapmakerU1-LEDTopPanel1.7.6.stl`](models/SnapmakerU1-LEDTopPanel1.7.6.stl) | Raw model geometry for users who prefer to configure slicing manually. |

## Model files

For the recommended print setup, open the 3MF project in a compatible slicer and review its plate, material, and support settings before printing. Use the STL when importing the geometry into another slicer or creating a custom print profile.

Both model files are version **1.7.6** and are stored in the [`models`](models) folder.

## Advanced installation

1. On the U1 touchscreen, enable **Settings → Maintenance → Advanced Mode**. Open Fluidd and back up `printer.cfg` and `moonraker.conf`.
2. Upload `skylake-wled.cfg` and `skylake-moonraker.conf` beside the stock configuration files.
3. In `skylake-moonraker.conf`, replace the example `address` with the WLED controller's fixed IP address. Keep the supplied LED count:

   ```ini
   [wled skylake_lights]
   type: http
   address: 192.168.0.123
   initial_preset: 1
   chain_count: 20
   ```

4. In the stock `[purifier]` section of `printer.cfg`, comment out only the `PE15` assignment:

   ```ini
   [purifier]
   exhaust_pin: !PA8
   inner_pin: !PA9
   # power_enable_pin: PE15
   power_det_pin: PA7
   ```

5. Add this to `printer.cfg`, outside every other configuration section:

   ```ini
   [include skylake-wled.cfg]
   ```

6. Add this to `moonraker.conf`:

   ```ini
   [include skylake-moonraker.conf]
   ```

7. Power-cycle the printer. Confirm all four toolheads are detected, toggle the stock cavity light, verify all 20 LEDs, and test the official top-cover fans and controls. Do not use unattended printing if any stock function behaves unexpectedly.

## WLED setup

1. Power on the printer and join the `WLED-AP` Wi-Fi network using the default password `wled1234`.
2. Open `http://4.3.2.1/`, configure the controller for the local Wi-Fi network, and reconnect your device to that network.
3. In **WLED → Config → LED Preferences**, confirm the LED length is **20**.
4. Note the WLED controller's IP address, put it on the `address:` line in `skylake-moonraker.conf`, save, and restart Moonraker or reboot the printer.
5. Save the desired normal white-light configuration as WLED preset 1.

## Required check after every printer update

A Snapmaker firmware update may replace or remove configuration files. Before using Skylake after any printer update, verify all six items:

- `skylake-wled.cfg` still exists beside `printer.cfg`.
- `skylake-moonraker.conf` still exists beside `moonraker.conf`.
- `[include skylake-wled.cfg]` is still present in `printer.cfg`.
- `[include skylake-moonraker.conf]` is still present in `moonraker.conf`.
- The stock `[purifier]` line `power_enable_pin: PE15` remains commented out before the Skylake include is enabled.
- `skylake-moonraker.conf` still contains the correct WLED IP address and `chain_count: 20`.

Then power-cycle the printer and repeat the functional tests. Never enable `skylake-wled.cfg` while the stock purifier also owns `PE15`.

## Fast recovery

If Klipper does not return to Ready:

1. Remove or comment out `[include skylake-wled.cfg]` in `printer.cfg`.
2. Restore `power_enable_pin: PE15` in the stock `[purifier]` section.
3. Restart the printer.

If Moonraker fails separately, remove or comment out `[include skylake-moonraker.conf]` and restart Moonraker.

## Known log message

Because the stock purifier no longer owns a `power_enable_pin`, changing its fan speed may produce this non-fatal message:

```text
[purifier] power enable pin not exist!
```

The implementation logs the message and returns instead of raising a command error.
