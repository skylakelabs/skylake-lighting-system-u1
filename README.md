# Skylake Lighting System for Snapmaker U1

Add WLED-controlled lighting to the Snapmaker U1 while keeping the stock cavity-light switch as the master control.

This configuration-only integration keeps the top-cover 24 V rail enabled and mirrors the U1 cavity-light state to WLED. It does not require a Python patch, Klipper extra, slicer change, or custom start/end G-code.

> [!WARNING]
> This is a test configuration for the Snapmaker U1. It reassigns MCU pin `PE15` from the stock purifier to a Klipper output pin. Never allow both objects to own `PE15`, and do not make or test wiring changes while the printer is energized.

## Current behavior

- The top-cover 24 V rail turns on when Klipper configures the MCU.
- The 24 V rail remains on while Klipper is operating.
- The stock cavity-light switch remains the master control.
- Turning the cavity light off turns WLED off.
- Turning the cavity light on loads WLED preset 1.
- WLED is contacted only when the cavity-light state changes.
- The default WLED LED count is 20.
- The installation is reversible.

## Known tradeoff

The stock purifier no longer owns a `power_enable_pin`. Changing its fan speed may produce this non-fatal log message:

```text
[purifier] power enable pin not exist!
```

The current Snapmaker implementation logs the message and returns instead of raising a command error. Official top-cover fan and UI behavior must still be tested before this configuration is treated as production-ready or used for unattended printing.

## Files

| File | Purpose |
| --- | --- |
| [`skylake-wled.cfg`](skylake-wled.cfg) | Keeps the top-cover 24 V rail enabled and mirrors the stock cavity-light state to WLED. |
| [`skylake-moonraker.conf`](skylake-moonraker.conf) | Defines Moonraker's WLED connection, preset, and LED count. |
| [`skylake-installation.txt`](skylake-installation.txt) | Complete installation, testing, recovery, and removal instructions. |
| [`skylake-functionality-plan.txt`](skylake-functionality-plan.txt) | Planned status-aware lighting behavior and future modes. |

## Before installation

1. Save the desired white-light configuration as **WLED preset 1**.
2. Give the WLED controller a static IP address or DHCP reservation.
3. Confirm the lighting, connector, wiring, fuse protection, and load are rated for 24 V and remain within the printer port and power-supply limits.
4. Back up `printer.cfg` and `moonraker.conf` somewhere other than the printer.
5. Do not perform the installation during a print.

## Installation

### 1. Enable Advanced Mode

On the physical U1 touchscreen, open:

```text
Settings > Maintenance > Advanced Mode
```

Accept the warning, enable Advanced Mode, and refresh Fluidd. The configuration-file lock icons should disappear.

### 2. Configure the WLED connection

Open [`skylake-moonraker.conf`](skylake-moonraker.conf) and replace the example address with the WLED controller's fixed address:

```ini
[wled skylake_lights]
type: http
address: 192.168.0.123
initial_preset: 1
chain_count: 20
```

The supplied configuration is set for **20 LEDs**. Change `chain_count` only if the physical installation uses a different number.

### 3. Upload the Skylake files

Upload these files beside `printer.cfg` and `moonraker.conf` in Fluidd:

- `skylake-wled.cfg`
- `skylake-moonraker.conf`

Do not restart yet.

### 4. Transfer ownership of PE15

Find the stock purifier section in `printer.cfg`:

```ini
[purifier]
exhaust_pin: !PA8
inner_pin: !PA9
power_enable_pin: PE15
power_det_pin: PA7
```

Comment out only the `power_enable_pin` line:

```ini
[purifier]
exhaust_pin: !PA8
inner_pin: !PA9
# power_enable_pin: PE15
power_det_pin: PA7
```

Do not alter the other purifier settings. This step is mandatory because `skylake-wled.cfg` assigns `PE15` to `skylake_24v`; Klipper must never assign the same pin to both objects.

### 5. Add the Klipper include

Add this near the other include statements in `printer.cfg`:

```ini
[include skylake-wled.cfg]
```

Before continuing, confirm that:

- `power_enable_pin: PE15` is commented out in the stock purifier section.
- `[include skylake-wled.cfg]` is present.

### 6. Add the Moonraker include

Add this to `moonraker.conf`:

```ini
[include skylake-moonraker.conf]
```

### 7. Restart

1. Restart Moonraker.
2. Perform a Klipper Firmware Restart.
3. Confirm Fluidd returns to **Ready** without a configuration error.

If Klipper reports that `PE15` is used more than once, immediately follow the recovery procedure below.

## Testing

### Test the 24 V rail first

Before connecting the lighting load, use verified connector pinout information and appropriately rated equipment:

1. Boot without the official top cover connected.
2. Verify approximately 24 V between the correct 24 V and GND pins.
3. Wait several minutes and confirm the voltage remains present.
4. Toggle the stock cavity light and confirm the 24 V rail remains present.
5. Perform a Klipper restart and confirm the rail safely cycles and returns.

### Test WLED

After connecting the correctly rated lighting system:

1. Allow WLED to boot and join Wi-Fi.
2. Turn the U1 cavity light off; WLED should turn off within approximately one second.
3. Turn the cavity light on; WLED should load preset 1 within approximately one second.
4. Repeat the cycle several times.

### Test the official top cover

If available, test every stock function under supervision:

1. Confirm genuine-cover detection changes normally.
2. Command both fans off and confirm they stop while the 24 V rail remains active.
3. Command each fan at several speeds.
4. Confirm tachometer and status feedback remain correct.
5. Exercise the stock purifier UI controls and automatic behavior.
6. Review `klippy.log` for unexpected errors beyond the known message above.

Do not proceed to unattended printing if any stock behavior changes materially.

## Fast recovery

If Klipper does not return to Ready:

1. Remove or comment out `[include skylake-wled.cfg]` in `printer.cfg`.
2. Restore `power_enable_pin: PE15` in the stock purifier section.
3. Perform a Firmware Restart.

If Moonraker fails separately, remove or comment out `[include skylake-moonraker.conf]` and restart Moonraker.

## Complete removal

1. Remove `[include skylake-wled.cfg]` from `printer.cfg`.
2. Restore `power_enable_pin: PE15` in the stock purifier section.
3. Remove `[include skylake-moonraker.conf]` from `moonraker.conf`.
4. Restart Moonraker and Klipper.
5. Delete the two Skylake files from the printer if desired.

## Firmware updates

A firmware update may replace `printer.cfg` or `moonraker.conf`. After every update, verify whether the stock purifier owns `PE15` before re-enabling the Skylake include. Never enable `skylake-wled.cfg` while the stock purifier also owns `PE15`.

## Project status

This repository currently contains the minimal cavity-light mirroring version. Planned printer-state effects, toolhead filament-color integration, heating alerts, and other modes are documented in [`skylake-functionality-plan.txt`](skylake-functionality-plan.txt) but are not implemented in the current runtime files.
