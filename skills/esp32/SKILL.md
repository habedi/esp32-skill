---
name: esp32
description: Develop, build, flash, and debug firmware and peripherals for ESP32-family MCUs. Use for board bring-up, GPIO and wiring, ESP-IDF, Arduino-ESP32, PlatformIO, or other ESP32 firmware projects. Do not use for ESP8266, STM32, or general C/C++ questions without ESP32 hardware.
license: Apache-2.0
metadata:
  version: "0.2.0"
---

# ESP32

## Safety Rules

- Open a physical serial port or flash a board only when the user authorizes that operation.
- Opening a serial console can reset some boards through DTR/RTS.
- Confirm which physical board and USB interface a port belongs to, especially after reconnecting or entering download mode.
- On Linux, compare device listings and use `/dev/serial/by-id/` when available, because a name like `/dev/ttyACM0` does not identify the board.
- Check the effective security configuration before flashing or booting a provisioning image.
- A bootloader configured for flash encryption or Secure Boot can burn eFuses on first boot, and flashing can trigger that reset and boot.
- Get explicit authorization before a full-chip erase, overwriting another MCU, or any operation that burns eFuses.
- Preserve existing firmware and data outside the intended update, including firmware on any secondary MCU.

## Identify the Target

- Read the project's build instructions, CI, and configuration to identify the application directory, framework, version, and build target.
- Use the repository's toolchain and wrappers when available.
- Check where the stack selects the target: `platformio.ini` in PlatformIO, chip target and `sdkconfig` in ESP-IDF, or stack-specific board files.
- For hardware work, identify the physical board and module or chip marking, as the build target only records what the firmware expects.
- Verify board and silicon revision, flash size and mode, PSRAM type, and console routing against the physical hardware when relevant.
- Look up board and SoC references in order:
    1. Repository docs and config (`AGENTS.md`, `README.md`, `docs/`, and board or SDK configuration files).
    2. Installed SDK or framework definitions (ESP-IDF `components/soc/` and `docs/en/hw-reference/`, NuttX `boards/`, Arduino-ESP32 `variants/`,
       PlatformIO `boards/`, or `esp-hal/src/soc/`).
    3. Espressif per-target hardware references, chip datasheets, and technical reference manuals for SoC behavior.
    4. Board vendor schematics or wikis for third-party boards, including USB bridges, onboard peripherals, and coprocessor wiring.
- For wiring, use the exact board schematic or pinout to find exposed pins and onboard connections.
- Check module and SoC docs for strapping pins, reserved flash and PSRAM pins, USB and console pins, input-only pins, pulls, and ADC/Wi-Fi conflicts.
- Verify voltage levels, current limits, and common ground before recommending connections.
  ([Espressif hardware design guidelines](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/))
- If the board is unknown, keep pin assignments conditional and proceed with work that does not depend on them.

## Build with the Project's Configuration

- Use headers and examples from the installed SDK version when changing drivers, and reuse project board definitions for pin mappings.
- On a new board or peripheral, start with a known-good example or a small smoke test with one observable result, such as a boot log or sensor read.
- Run the smallest relevant build and tests.
- For ESP-IDF, activate the project's SDK environment and run `idf.py build` in the application directory.
  ([ESP-IDF CLI](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/tools/idf-py.html))
- Check the effective `CONFIG_IDF_TARGET` and `sdkconfig`.
  ([Build configuration](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/build-system.html))
- Update `sdkconfig.defaults` or `sdkconfig.defaults.<target>` when those files drive configuration, and verify `sdkconfig` afterward so stale values
  do not mask changes.
- Preserve local settings before running `idf.py set-target`, because it clears the build directory and regenerates `sdkconfig`.
- For PlatformIO, build the selected environment with `pio run -e ENV`.
  ([PlatformIO CLI](https://docs.platformio.org/en/stable/userguide/cmd_run.html))
- For a standalone Arduino sketch, run `arduino-cli compile --fqbn FQBN SKETCH` with the project's board name and options.
  ([Arduino CLI compile](https://arduino.github.io/arduino-cli/latest/commands/arduino-cli_compile/))

## Flash and Verify on Hardware

- Use the project flasher with an explicit port and the intended build, and check which images and offsets the command will write.
- In ESP-IDF, `idf.py -p PORT flash` writes the bootloader, partition table, and app.
- Use `idf.py -p PORT app-flash` for an app-only update when the app matches the installed bootloader and partition layout.
- Check which app partition the build's write offset targets, and on OTA-enabled boards verify which slot the bootloader will start next.
- In PlatformIO, use `pio run -e ENV -t upload --upload-port PORT` for serial uploads.
- When using `esptool` directly, take image offsets and flash parameters from the matching build artifacts.
- Use the configured console interface and baud rate for a bounded log capture, keeping only one serial reader open at a time and closing it when
  done.
- For ESP-IDF, `idf.py -p PORT monitor` decodes backtraces using the matching build's ELF.
  ([ESP-IDF Monitor](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/tools/idf-monitor.html))
- Check for the expected boot message and a feature-specific result, then report the verification details using the format below.

## Diagnose by Symptom

| Symptom                                             | First checks                                                                                                                                                                                                          |
|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Flasher cannot connect                              | Verify the port, data cable, permissions, and current port owner. Check the board-specific BOOT and RESET procedure if automatic download mode fails.                                                                 |
| Transfer stops or reports corrupt packets           | Check power and cable quality, then try a lower upload baud rate.                                                                                                                                                     |
| Flash succeeds but the console is silent or garbled | Verify the intended console interface (UART bridge, native USB Serial/JTAG, or USB CDC), port after reset, and baud rate for UART.                                                                                    |
| Firmware resets or panics                           | Capture the first boot log and reset reason. Check power for brownouts, decode panics with the matching ELF, and compare flash mode, PSRAM, partition layout, and silicon revision with the effective build settings. |
| Peripheral fails to respond                         | Verify pin mapping, power, common ground, and bus settings against the schematic. Test the smallest transaction or signal before debugging the application.                                                           |

Use the troubleshooting guide for the exact SoC and tool version.
([Espressif flasher troubleshooting](https://docs.espressif.com/projects/esptool/en/latest/esp32/troubleshooting.html))

## Verification Report Format

When reporting a build, simulator run, or hardware check, record:

- **Target:** [Board / SoC / Port, plus whether checked via build, emulator, or physical hardware]
- **Command:** `[Exact command run]`
- **Result:** [Pass or Fail + key log line]
- **Unverified:** [What was only checked at build time or remains untested]

## Example

**User:** Flash my ESP32-S3 on `/dev/ttyACM0` with the new build. **Assistant:**

1. I checked `sdkconfig` and confirmed `CONFIG_IDF_TARGET="esp32s3"`.
2. This command writes the app to `/dev/ttyACM0`: `idf.py -p /dev/ttyACM0 app-flash`.
3. Confirm that `/dev/ttyACM0` is the target board and that you authorize the flash.
