# Xteink Mallorn

**Bring-up note — SDK profile, NOT yet fully hardware-validated.**

ESP32-S3 (16 MB flash, 8 MB PSRAM) e-reader in the Xteink X4 family. It pairs the
X4 Pro's S3 silicon with the **X4's resistor-ladder button input** (no touch), an
800×480 B/W SSD1677-class panel, **native 4-bit SDMMC**, and a
**brightness/color-select frontlight** with an **active-LOW LED-boost master pin**.
The TP4054 charger reports on the battery ADC.

Distinct from the C3 `XTEINK_X4` (different MCU) and from `XTEINK_X4_PRO` (different
input/touch/frontlight). Its own profile is `BoardConfig::MALLORN`.

The panel controller **may vary by production batch** (SSD1677 / UC8179 / UC8279),
exactly like the X4 Pro; the SDK selects the driver at boot via the display probe
(`FREEINK_XTEINK_DISPLAY_PROBE` includes MALLORN).

Build: `-DFREEINK_DEVICE_MALLORN=1` (see `platformio.sample.ini` `[env:mallorn]`).
`FREEINK_DRIVER_SSD1677`, `FREEINK_DRIVER_UC8179`, `FREEINK_DRIVER_UC8279_X4`,
`FREEINK_CAP_FRONTLIGHT`, `FREEINK_CAP_WARMLIGHT`, and `FREEINK_SD_SDMMC`
auto-enable. The SD path additionally requires `USE_BLOCK_DEVICE_INTERFACE=1` in
the consumer build.

This doc reflects pin data from the board's `PIN_REF` (see the consumer repo's
`docs/PIN_REF.md`). Items marked **Confirmed** come from that reference; items
marked **Pending** still need measurement on hardware.

## Display — SSD1677 / UC8179 / UC8279, 800×480

Pinout from `PIN_REF` (physical pin → io):

| Signal | io | physical pin |
|---|---|---|
| MOSI | io9 | 17 |
| SCK | io10 | 18 |
| CS | io11 | 19 |
| DC | io12 | 20 |
| RST | io13 | 21 |
| BUSY | io14 | 22 |

No display power-enable pin (panel always powered). 20 MHz SPI.

## Input — X4-style ADC resistor ladder

Two ADC groups, same pins as the C3 X4:

| Group | io | Buttons | Ladder |
|---|---|---|---|
| Group 1 | io1 (GPIO1) | Back / Confirm / Left / Right | 100 Ω, 5.1 kΩ, 20 kΩ, 56 kΩ |
| Group 2 | io2 (GPIO2) | Up / Down | 100 Ω, 12 kΩ |

The ADC thresholds are the shared hard-coded `InputManager` bands (X4 defaults:
`ADC_RANGES_1` / `ADC_RANGES_2`, i.e. `{3900, 3100, 2090, 750, INT32_MIN}` /
`{3900, 1120, INT32_MIN}`) — **Pending**: measure real readings per button and
retune the bands.

The power button is GPIO3, active-LOW (X4 inherited) — **Pending**: confirm on
hardware.

## Frontlight — brightness / color-select PWM

| Signal | io | Role |
|---|---|---|
| PWM_LED | io5 (GPIO5) | Brightness PWM (common dimming for both strips) |
| COLOR_SEL | io6 (GPIO6) | Color-select PWM (low = warm, full = cool, mid = blended) |
| LED_ACTIVATE | io17 (GPIO17) | **ACTIVE-LOW** master enable for the LED boost circuit |

Both PWM pins run high-frequency LEDC (10 kHz / 10-bit, active-HIGH) on one shared
timer. `gpioColorSelect` duty is independent of brightness (it selects the tint).
`gpioBoostEnable` (active-low) gates the boost driver; it is driven LOW while the
light is lit and HIGH when off. **Pending**: confirm the color-select pin is really
the brightness/color-select topology vs a dual-string (X4 Pro) pair, and the
polarity of each pin.

## Storage — SD card

Native 4-bit SDMMC (always powered):

| Signal | io | physical pin |
|---|---|---|
| DAT0 | io38 | 31 |
| CLK | io39 | 32 |
| CMD | io40 | 33 |
| DAT3 | io41 | 34 |
| DAT2 | io42 | 35 |
| DAT1 | io48 | 25 |

No power-enable pin (SD always on).

## Battery

- ADC on io4 (GPIO4), 1:2 divider → `batteryDividerMultiplier = 2.0`.
- Charge-status pin (TP4054 CHRG/STDBY) **Pending** (Q11) — currently unassigned.

## USB / RTC

- No USB-detect pin in `PIN_REF` → `usbDetect = PIN_UNASSIGNED` (Q8).
- No on-board RTC.

## Pending / not yet confirmed

- Display SPI pin sweep (Q1) — pinout above is from `PIN_REF`, not yet swept.
- Panel mount orientation (Q2) — profile carries `NO_FLIP`.
- Frontlight topology / polarity (Q3) — brightness/color-select assumed.
- SDMMC sweep + always-on power (Q4) — pinout above is from `PIN_REF`.
- Power-button pin (Q6 area) — GPIO3 inherited from X4.
- ADC ladder thresholds (Q9) — X4 default bands, unmeasured.
- Battery divider (Q10) — 1:2 per `PIN_REF`; multiplier 2.0.
- Charge-status pin (Q11) — TP4054 CHRG unassigned until measured.
</content>
