# Agent Instructions

This repository includes a portable agent skill (in `skills/esp32/`) for ESP32 family of microcontrollers.

- Keep changes focused and write in plain English. Use short sentences, title case for headings, and Oxford commas.
- Keep shared instructions applicable across boards and toolchains. Verify board-specific claims against the exact schematic, datasheet, or SDK
  version.
- Keep the skill self-contained. Add a supporting file when the task benefits from loading that detail separately.
- Open a physical serial port or flash a board only when the user authorizes the operation. State whether a check used a build, an emulator, or
  physical hardware.
- Keep personal notes and unfiltered logs in ignored local paths such as `tmp/`.
- Validate changes to the skill's frontmatter and run the repository checks shown in the README.
