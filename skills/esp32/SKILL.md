---
name: esp32
description: Develop, build, flash, and debug firmware and peripherals for ESP32-family MCUs. Use for board bring-up, GPIO and wiring, ESP-IDF, Arduino-ESP32, PlatformIO, or other ESP32 firmware projects.
license: Apache-2.0
metadata:
  version: "0.1.0"
---

# ESP32

## Identify the Target

- Read the project's build instructions, CI, and configuration. Identify the application directory, framework and version, and build target or board
  profile. Use the repository's toolchain and wrappers when available. PlatformIO selects a board and framework per environment in `platformio.ini`.
  ESP-IDF uses the chip target and project configuration. Other stacks may use their own board definitions.
- For hardware work, identify the actual board and module or chip marking. Verify board and silicon revision, flash size and mode, PSRAM type, and
  console routing when relevant. A build target records what the firmware expects. Verify the physical hardware separately.
- Look up board and SoC references in order. Start with the repository's `AGENTS.md`, `README.md`, `docs/`, and board or SDK configuration files.
  Next, inspect the installed SDK or framework for local SoC and board definitions, such as ESP-IDF `components/soc/` and `docs/en/hw-reference/`,
  NuttX `boards/`, Arduino-ESP32 `variants/`, PlatformIO `boards/`, or `esp-hal/src/soc/`. For external documentation, use Espressif's per-target
  hardware reference, chip datasheets, and technical reference manuals for SoC behavior. Use the board vendor's schematic or wiki for third-party
  boards, including USB bridges, onboard peripherals, and coprocessor wiring.
- For wiring, use the exact board schematic or pinout to find exposed pins and onboard connections. Check the module and SoC documentation for
  strapping levels at reset, reserved flash and PSRAM pins, USB and console pins, input-only pins, pulls, and peripheral restrictions such as Wi-Fi
  sharing ADC resources. Check voltage levels, current limits, and common ground before recommending connections. If the board is unknown, keep pin
  assignments conditional and proceed with work that does not depend on
  them. [Espressif hardware design guides](https://docs.espressif.com/projects/esp-hardware-design-guidelines/en/latest/)

## Build with the Project's Configuration

- Use headers and examples from the installed SDK version when changing drivers. Reuse project board definitions for pin mappings. On a new board or
  peripheral, start with a known-good example or a small smoke test with one observable result, such as a boot log or a successful sensor read.
- Run the smallest relevant build and tests. For plain ESP-IDF, activate the project's SDK environment and run `idf.py build` in the application
  directory. Check the effective `CONFIG_IDF_TARGET` and `sdkconfig`. If `sdkconfig.defaults` or `sdkconfig.defaults.<target>` drives configuration,
  update the appropriate inputs. Existing values in `sdkconfig` can mask changes to defaults, so verify the result after reconfiguration.
  `idf.py set-target` clears the build directory and regenerates `sdkconfig`, so preserve local settings before intentionally switching
  targets. [ESP-IDF CLI](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/tools/idf-py.html) · [Build configuration](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/build-system.html)
- For PlatformIO, build the selected environment with `pio run -e ENV`. For a standalone Arduino sketch, use the project's fully qualified board name
  and board options with `arduino-cli compile --fqbn FQBN SKETCH`. Substitute the project's
  values. [PlatformIO CLI](https://docs.platformio.org/en/stable/userguide/cmd_run.html) · [Arduino CLI compile](https://arduino.github.io/arduino-cli/latest/commands/arduino-cli_compile/)

## Flash and Verify on Hardware

- Open a physical serial port or flash a board only when the user authorizes that operation. Opening
  a console can reset some boards through DTR/RTS. Confirm which physical board and USB interface the port belongs to, especially after reconnecting
  or entering download mode. On Linux, compare device listings and use `/dev/serial/by-id/` when available. A name like `/dev/ttyACM0` alone does not
  establish which board or interface is attached.
- Use the project flasher with an explicit port and the intended build. Check which images and offsets the command will write. In ESP-IDF,
  `idf.py -p PORT flash` can also write the bootloader and partition table. For an app-only update, use `idf.py -p PORT app-flash` when the app
  is compatible with the installed bootloader and partition layout. Before flashing, check which app partition the build's write offset targets.
  On boards using over-the-air updates, verify which partition the bootloader will start next. OTA metadata can select a different slot.
  PlatformIO uses `pio run -e ENV -t upload --upload-port PORT` for serial
  uploads. If using esptool directly, take image offsets and flash parameters from the matching build artifacts. Preserve existing firmware and
  data outside the intended update, including firmware on any secondary MCU.
- Check the effective security configuration before flashing or booting a provisioning image. A bootloader configured to enable flash encryption
  or Secure Boot can burn eFuses on first boot. Flashing can reset the board and start that boot. Get explicit authorization before a full-chip
  erase, overwriting another MCU, or any operation that will burn eFuses, including first-boot provisioning.
- Use the configured console interface and baud rate for a bounded log capture. For plain ESP-IDF, `idf.py -p PORT monitor` can decode backtraces
  using the matching build's ELF. Keep one serial reader at a time and close it when done. Check for the expected boot and a feature-specific result.
  Record the target, command, and evidence, and say what could only be checked at build
  time. [ESP-IDF Monitor](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/api-guides/tools/idf-monitor.html)

## Diagnose by Symptom

| Symptom                                             | First checks                                                                                                                                                                                                          |
|-----------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Flasher cannot connect                              | Verify the port, data cable, permissions, and current port owner. Check the board-specific BOOT and RESET procedure if automatic download mode fails.                                                                 |
| Transfer stops or reports corrupt packets           | Check power and cable quality, then try a lower upload baud rate.                                                                                                                                                     |
| Flash succeeds but the console is silent or garbled | Verify the intended console interface (UART bridge, native USB Serial/JTAG, or USB CDC), port after reset, and baud rate for UART.                                                                                    |
| Firmware resets or panics                           | Capture the first boot log and reset reason. Check power for brownouts, decode panics with the matching ELF, and compare flash mode, PSRAM, partition layout, and silicon revision with the effective build settings. |
| Peripheral fails to respond                         | Verify pin mapping, power, common ground, and bus settings against the schematic. Test the smallest transaction or signal before debugging the application.                                                           |

Use the guide for the exact SoC and tool
version. [Espressif flasher troubleshooting](https://docs.espressif.com/projects/esptool/en/latest/esp32/troubleshooting.html)
