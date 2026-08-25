# FreeInk SDK change log (Mallorn bring-up)

Local commit trail for the Mallorn board support. Commit hashes are the local
submodule commits; nothing here is pushed upstream.

## `172f34e` — refactor: share the hard-coded X4 ADC ladder, drop the `AdcLadderConfig` profile

- `libs/hardware/InputManager/**`
  - Restored the hard-coded ladder constants that `7b37a37` had replaced:
    `BUTTON_ADC_PIN_1/2` (= 1, 2), `ADC_RANGES_1/2[]` and `ADC_NO_BUTTON`
    (= 3900). `begin()`, `readButtonAdc()`, `getState()` read them again
    instead of `BoardConfig::ACTIVE.adcLadder`.
- `libs/hardware/BoardConfig/include/BoardConfig.h`
  - Removed the `AdcLadderConfig` struct and the `BoardProfile::adcLadder`
    field. The MALLORN profile reads the shared hard-coded ladder.
- Rationale: PIN_REF.md confirms Mallorn uses the same two-pin / six-button
  resistor network as X3/X4/X4 Pro, so the per-board ladder profile is needless
  indirection — see CHANGES.md §4.2.

## `c93b2d3` — feat: add Mallorn device flag, BoardConfig profile + frontlight fields

- `libs/hardware/BoardConfig/include/BoardConfig.h`
  - Added `FREEINK_DEVICE_MALLORN` normalization + coherence/MCU-family wiring
    (ESP32-S3).
  - Added `Board::Mallorn`, `MALLORN` profile (800×480 SSD1677 SPI, X4 ADC ladder,
    native 4-bit SDMMC, brightness/color-select frontlight + active-low
    LED-boost master, ADC battery).
  - Added `FrontlightConfig::gpioColorSelect` and `FrontlightConfig::gpioBoostEnable`.
  - Wired `FREEINK_DEVICE_MALLORN` into driver links (SSD1677/UC8179/UC8279_X4),
    capabilities (FRONTLIGHT, WARMLIGHT, SD_SDMMC), `MAX_FRAMEBUFFER_BYTES`,
    `DEFAULT_DEVICE`, `selectDevice()`, and `isMallorn()`.
  - Did NOT add Mallorn to touch/RTC/gauge/mic caps (no such hardware).

## `7b37a37` — feat: color-select + boost-enable frontlight; probe Mallorn

- `libs/hardware/FrontlightManager/**`
  - `begin()` attaches the color-select pin to `LEDC_CH_WARM` (same timer/freq/
    resolution as the brightness channel) and drives the boost-enable pin.
  - `apply()` gains the brightness/color-select branch: brightness pin carries
    `totalDuty`; color-select pin gets a FLAT duty proportional to coolness,
    independent of brightness. Boost-enable (active-low) tracks whether the
    light is lit.
  - `hasColorTemperature()` also returns true for the color-select topology.
- `libs/hardware/XteinkDetect/src/XteinkDetect.cpp`
  - `FREEINK_XTEINK_DISPLAY_PROBE` includes `FREEINK_DEVICE_MALLORN`.

## Docs

- `docs/mallorn-support.md` — bring-up note with confirmed/pending split.
- `README.md` — device table row, device-flag row, SDMMC cap note, sample-env list.
- `platformio.sample.ini` — `[env:mallorn]`.

## `653eafc` — docs: add Mallorn support + change log; sample env; README tables

(commits the docs/README/sample-ini changes listed above)

## `afad963` — fix: correct InputStyle::XteinkAdcLadder spelling in MALLORN profile

- Fixed the enum spelling in the `MALLORN` profile (`XTeinkAdcLadder` ->
  `XteinkAdcLadder`). Caught by a standalone g++ `-fsyntax-only` pass over the
  header.

## PENDING (must be measured on hardware before merge)

- Display SPI pin sweep (Q1) — pinout from PIN_REF, not yet swept.
- Panel orientation (Q2) — `NO_FLIP` assumed.
- Frontlight topology/polarity (Q3) — brightness/color-select assumed.
- SDMMC always-on + pinout (Q4) — from PIN_REF.
- Power-button pin — GPIO3 (X4 inherited).
- ADC ladder thresholds (Q9) — X4 defaults.
- Battery divider (Q10) — 1:2 per PIN_REF.
- Charge-status pin (Q11) — unassigned until measured.